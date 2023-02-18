<template>
  <div class="container">
    <div class="tabs">
      <div @click="tab.selectedTab = item" class="tab" :class="{active: tab.selectedTab == item}" v-for="(item,index) in tab.tabs" :key="index">{{item}}</div>
    </div>
    <div class="tab-wrapper">
      <div class="tab" v-if="tab.selectedTab == 'дашборд'">
        <div class="cards" ref="cards" @mouseleave="stopDrag">
          <div class="card-wrapper"
               v-for="(item, index) in cards"
               :key="index" :style="`width: ${item.size.x}px; height: ${item.size.y}px;`">
            <div class="card"
                 :style="`width: ${item.size.x}px; height: ${item.size.y}px;`"
                 @mousedown="mouseStart($event,item)"
                 @mouseup="mouseEnd($event,item)"
                 @mousemove="mouseMove($event,item)"
                 @touchstart="touchStart($event,item)"
                 @touchend="touchEnd($event,item)"
                 @touchmove="touchMove($event,item)">
              <span>{{ item.text }}</span>
              <doughnut v-if="chartdata.labels && chartdata.datasets" :datasets="chartdata.datasets" :labels="chartdata.labels"
                        :options="chartdata.options" />
            </div>
          </div>
        </div>
        <div class="chart-wrapper">
          <linechart v-if="chartdata.labels && chartdata.datasets" :datasets="chartdata.datasets" :labels="chartdata.labels"
                     :options="chartdata.options" class="linechart"/>
        </div>
      </div>
      <div class="tab" v-if="tab.selectedTab == 'редактор кода'">
        <div class="tab-header">
          <p>psql запрос на выборку данных (для продвинутых пользователей)</p>
          <button class="execute">выполнить</button>
        </div>
        <textarea class="code-wrapper" contenteditable="true" v-model="code"/>
      </div>
    </div>
    <div class="chart-wrapper">
      <barchart v-if="chartdata.labels && chartdata.datasets" :datasets="chartdata.datasets" :labels="chartdata.labels"></barchart>
    </div>
  </div>
</template>
<script>
import {gsap,TweenMax} from "gsap"
import linechart from "@/components/linechart.component.vue";
import barchart from "@/components/barchart.component.vue";
import doughnut from "@/components/doughnut.component.vue";
export default {
  name: "home",
  components: {doughnut, barchart, linechart},
  data() {
    return {
      cards: [
        {
          index: 0,
          text: "график 1",
          size:{
            x: 144,
            y: 144
          }
        },
        {
          index: 1,
          text: "график 2",
          size:{
            x: 144,
            y: 144
          }
        },
        {
          index: 2,
          text: "график 3",
          size:{
            x: 144,
            y: 144
          }
        },
        {
          index: 3,
          text: "график 4",
          size:{
            x: 144,
            y: 144
          }
        },
        {
          index: 4,
          text: "график 5",
          size:{
            x: 144,
            y: 144
          }
        },
      ],
      drag: {
        item: undefined,
        interval: undefined,
        el: undefined
      },
      chartdata: {
        delayed: false,
        labels: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12'],
        datasets: [
          {
            label: 'прошлые показатели',
            data: [0, 20, 20, 60, 60, 60, 120, 140, 180, 120, NaN, NaN, NaN],
            borderColor: "#A20DF6",
            backgroundColor: "#A20DF6",
            fill: false,
          }, {
            label: 'предсказано',
            data: [NaN, NaN, NaN, NaN, NaN, NaN, NaN, NaN, NaN, 120, 10, 120, 130],
            borderDash: [5, 5],
            borderColor: "#00C572",
            backgroundColor: "#00C572",
            fill: false,
          }],
        options: {
          tension: 0.4,
          responsive: true,
          maintainAspectRatio: false,
          animation: {
            onComplete: () => {
              this.chartdata.delayed = true;
            },
            delay: (ctx) => {
              let delay = 0;
              if (ctx.type == "data" && ctx.mode == "default" && !this.chartdata.delayed) {
                delay = ctx.dataIndex * 70 + ctx.datasetIndex * 40
              }
              return delay;
            }
          },
          interaction: {
            intersect: false,
          },
          // tooltips: {
          //   mode: 'index'
          // },
          scales: {
            x: {
              display: true,
              title: {
                display: true
              }
            },
            y: {
              display: true,
              title: {
                display: true,
                text: 'Цена'
              },
              suggestedMin: -10,
              suggestedMax: 200
            }
          }
        }
      },
      tab:{
        tabs: [
          "дашборд",
          "редактор кода"
        ],
        selectedTab: "дашборд",
      },
      code: "SELECT current_datetime"
    }
  },
  methods: {
    touchStart(event, item) {
      if (window.mobileAndTabletCheck() && !this.drag.item && !this.drag.el) {
        this.drag.item = item
        let parent = event.target
        for (let i = 0; i < 10; i++) {
          if(parent.classList.contains('card')){
            this.drag.el = parent
            break
          }else{
            parent = parent.parentElement
          }
        }
        if(!this.drag.el) return
        // console.log(event)
        document.body.style.overflowY = 'hidden'
        gsap.to(this.drag.el, {
          top: event.changedTouches[0].pageY+document.body.scrollTop - this.drag.el.offsetHeight / 2,
          left: event.changedTouches[0].pageX+document.body.scrollLeft - this.drag.el.offsetWidth / 2,
          duration: 0
        })
        this.animate(this.drag.el,100)
        this.drag.el.classList.toggle('active', true)
        clearInterval(this.drag.interval)
        this.drag.interval = setInterval(()=>{
          TweenMax.to(this.drag.el, 0.03, {
            rotation: `3`,
            yoyo: true,
            repeat: 1,
          });
        },100)
      }
    },
    touchEnd(event, item) {
      if (window.mobileAndTabletCheck()){
        for (let i = 0; i < this.$refs.cards.querySelectorAll('.card').length; i++) {
          const index = i
          const card = this.$refs.cards.querySelectorAll('.card')[i]
          if (index != this.cards.find(c => c == item).index &&
            event.changedTouches[0].pageX >= card.offsetLeft &&
            event.changedTouches[0].pageX <= card.offsetWidth + card.offsetLeft &&
            event.changedTouches[0].pageY >= card.offsetTop &&
            event.changedTouches[0].pageY <= card.offsetHeight + card.offsetTop) {
            let newIndex = this.cards[index].index
            this.cards[index].index = item.index
            this.cards.map(c=>{
              if(c != this.cards[index] && c.index == item.index){
                c.index = newIndex
              }
              return c
            })
            this.filterByIndex()
            break
          }
        }
        this.stopDrag()
      }
    },
    touchMove(event, item) {
      if (window.mobileAndTabletCheck() && this.drag.item && this.drag.el) {
        // event.target.style.top = `${event.changedTouches[0].pageY - event.target.offsetHeight / 2}px`
        // event.target.style.left = `${event.changedTouches[0].pageX - event.target.offsetWidth / 2}px`
        gsap.to(this.drag.el, {
          top: event.changedTouches[0].pageY+document.body.scrollTop - this.drag.el.offsetHeight / 2,
          left: event.changedTouches[0].pageX+document.body.scrollLeft - this.drag.el.offsetWidth / 2,
          duration: 0
        })
      }
    },
    mouseStart(event, item) {
      if (!window.mobileAndTabletCheck() && !this.drag.item && !this.drag.el) {
        this.drag.item = item
        let parent = event.target
        for (let i = 0; i < 10; i++) {
          if(parent.classList.contains('card')){
            this.drag.el = parent
            break
          }else{
            parent = parent.parentElement
          }
        }
        if(!this.drag.el) return
        console.log(event)
        document.body.style.overflowY = 'hidden'
        gsap.to(this.drag.el, {
          top: event.pageY+document.body.scrollTop - this.drag.el.offsetHeight / 2,
          left: event.pageX+document.body.scrollLeft - this.drag.el.offsetWidth / 2,
          duration: 0
        })
        this.animate(this.drag.el,100)
        this.drag.el.classList.toggle('active', true)
        clearInterval(this.drag.interval)
        this.drag.interval = setInterval(()=>{
          TweenMax.to(this.drag.el, 0.03, {
            rotation: `3`,
            yoyo: true,
            repeat: 1,
          });
        },100)
      }
    },
    mouseEnd(event, item) {
      if (!window.mobileAndTabletCheck()){
        for (let i = 0; i < this.$refs.cards.querySelectorAll('.card').length; i++) {
          const index = i
          const card = this.$refs.cards.querySelectorAll('.card')[i]
          if (index != this.cards.find(c => c == item).index &&
            event.pageX >= card.offsetLeft &&
            event.pageX <= card.offsetWidth + card.offsetLeft &&
            event.pageY >= card.offsetTop &&
            event.pageY <= card.offsetHeight + card.offsetTop) {
            let newIndex = this.cards[index].index
            this.cards[index].index = item.index
            this.cards.map(c=>{
              if(c != this.cards[index] && c.index == item.index){
                c.index = newIndex
              }
              return c
            })
            this.filterByIndex()
            break
          }
        }
        this.stopDrag()
      }
    },
    mouseMove(event, item) {
      if (!window.mobileAndTabletCheck() && this.drag.item && this.drag.el) {
        gsap.to(this.drag.el, {
          top: event.pageY+document.body.scrollTop - this.drag.el.offsetHeight / 2,
          left: event.pageX+document.body.scrollLeft - this.drag.el.offsetWidth / 2,
          duration: 0
        })
        // event.target.style.top = `${event.pageY - event.target.offsetHeight / 2}px`
        // event.target.style.left = `${event.pageX - event.target.offsetWidth / 2}px`
      }
    },
    stopDrag() {
      if (this.drag.item && this.drag.el) {
        this.drag.item = undefined
        this.drag.el = undefined
        clearInterval(this.drag.interval)
        document.body.style.overflowY = 'overlay'
        this.$refs.cards.querySelectorAll('.card').forEach((card, index) => {
          // card.style.top = "auto"
          // card.style.left = "auto"
          card.classList?.toggle('active', false)
          gsap.to(card, {
            top: "auto",
            left: "auto",
            duration: 0.3,
            rotation: 0
          })
        })
      }
    },
    filterByIndex(){
      for (let j = this.cards.length - 1; j > 0; j--) {
        for (let i = 0; i < j; i++) {
          if (this.cards[i].index > this.cards[i + 1].index) {
            let temp = this.cards[i];
            this.cards[i] = this.cards[i + 1];
            this.cards[i + 1] = temp;
          }
        }
      }
      console.log(this.cards)
    },
    vibrate(value) {
      let navigator = window.navigator;
      if ("vibrate" in navigator) {
        navigator.vibrate(value);
      } else if ("oVibrate" in navigator) {
        navigator.oVibrate(value);
      } else if ("mozVibrate" in navigator) {
        navigator.mozVibrate(value);
      } else if ("webkitVibrate" in navigator) {
        navigator.webkitVibrate(value);
      }
    },
    animate(el = document.body, vibration = 0) {
      if (!el.classList.contains("card")) {
        if (!el.parentElement.classList.contains("card")) {
          if (!el.parentElement.parentElement.classList.contains("card")) {
          } else {
            el = el.parentElement.parentElement;
          }
        } else {
          el = el.parentElement;
        }
      }
      this.vibrate(vibration);
      if (vibration == 0) {
        el.style.transform = "none";
      } else {
        TweenMax.to(el, 0.03, {
          rotation: `2`,
          yoyo: true,
          repeat: 1,
        });
      }
    },
  }
}
</script>
<style lang="scss" scoped>
.cards {
  //display: flex;
  //gap: 24px;
  //flex-wrap: wrap;

  .card-wrapper {
    display: inline-block;
    margin: 12px;
    width: 256px;
    height: 256px;
    @media screen and (max-width: 768px) {
      width: 128px;
      height: 128px;
    }
    background: rgba(220, 220, 220, 0.25);
    border-radius: 4px;
  }

  .card {
    width: 256px;
    height: 256px;
    @media screen and (max-width: 768px) {
      width: 128px;
      height: 128px;
    }
    padding: 6px;
    background: #e1e1e1;
    border-radius: 4px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: var(--font-xxl);
    position: absolute;
    color: var(--color-accent);
    cursor: pointer;
    user-select: none;

    &.active {
      z-index: 999;
      box-shadow: 0px 4px 20px rgba(0, 0, 0, 0.03);
    }

    span {
      position: absolute;
      top: 50%;
      left: 50%;
      font-size: var(--font-s);
      color: var(--color-font);
      transform: translate(-50%,-50%);
      pointer-events: none;
      text-align: center;
      width: 80%;
    }
  }
}
.code-wrapper{
  padding: 24px;
  border-radius: var(--border-radius);
  width: 100%;
  min-height: 500px;
  border: 2px solid var(--color-accent-30);
  direction: ltr;
  text-align: left;
  white-space: pre;
  word-spacing: normal;
  -moz-tab-size: 4;
  -o-tab-size: 4;
  tab-size: 4;
  -webkit-hyphens: none;
  -moz-hyphens: none;
  -ms-hyphens: none;
  hyphens: none;
  background: none;
  color: var(--color-font);
  resize: vertical;
  word-break: break-all;
}
.chart-wrapper {
  .linechart {
    height: 350px;
  }
  @media screen and (max-width: 768px) {
    width: 100%;
    overflow-x: scroll;
    overflow-y: hidden;
    border-radius: var(--border-radius);
    .linechart {
      width: 1000px;
    }
  }
}
.tabs{
  display: flex;
  gap: 32px;
  align-items: center;
  padding-bottom: 24px;
  .tab{
    cursor: pointer;
    padding: 8px 16px;
    user-select: none;
    border-radius: var(--border-radius);
    &.active{
      background: var(--color-additional);
      color: var(--color-main);
    }
  }
}
.tab-wrapper{
  .tab{
    display: flex;
    flex-direction: column;
    gap: 24px;
    margin-bottom: 90px;
    .tab-header{
      display: flex;
      gap: 32px;
      flex-wrap: wrap;
      align-items: center;
    }
    button.execute{
      margin-left: auto;
    }
  }
}
</style>
