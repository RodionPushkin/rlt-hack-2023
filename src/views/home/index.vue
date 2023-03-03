<template>
  <div>
    <form class="search" @submit.prevent>
      <div class="input">
        <input ref="search" placeholder=" " type="text" @change="load">
        <label>поиск</label>
      </div>
      <button @click="load">найти</button>
    </form>
    <div class="filters">
      <select v-for="(filter,index) in filters" :key="index" :ref="filter.field_name" v-model="filter.selected"
              @change="load">
        <option :value="filter.field" selected>{{ filter.field }}</option>
        <option v-for="(value,value_index) in filter.values" :key="filter.field+value_index" :value="value">
          {{ value }}
        </option>
      </select>
      <div class="input">
        <input ref="datefrom" placeholder=" " type="date" @change="load">
        <label>дата от</label>
      </div>
      <div class="input">
        <input ref="dateto" placeholder=" " type="date" @change="load">
        <label>дата до</label>
      </div>
    </div>
    <div ref="cards" class="cards" @mouseleave="stopDrag">
      <div v-for="(item, index) in cards"
           :key="index+item.type"
           :style="`width: ${item.size?.x ? item.size.x : ''}; height: ${item.size?.y ? item.size.y : ''};`"
           class="card-wrapper">
        <div :style="`width: ${item.size?.x ? item.size.x : ''}; height: ${item.size?.y ? item.size.y : ''};`"
             class="card"
             @mousedown="mouseStart($event,item)"
             @mousemove="mouseMove($event,item)"
             @mouseup="mouseEnd($event,item)"
             @touchend="touchEnd($event,item)"
             @touchmove="touchMove($event,item)"
             @touchstart="touchStart($event,item)">
          <!--          {{item.index}}-->
          <!--          <i>{{item.index}}</i>-->
          <span :class="{info: item.type == 'info'}" class="title">{{ item.title }}</span>
          <p v-if="item.type == 'info'">{{ item.value }}</p>
          <div v-if="item.type == 'doughnut' || item.type == 'linechart' || item.type == 'barchart'"
               class="chart-wrapper">
            <doughnut v-if="chartdata.labels && chartdata.datasets && item.type == 'doughnut'"
                      :datasets="chartdata.datasets" :labels="chartdata.labels"
                      :options="chartdata.options"/>
            <linechart v-if="chartdata.labels && chartdata.datasets && item.type == 'linechart'"
                       :datasets="chartdata.datasets2" :labels="chartdata.labels"
                       :options="chartdata.options" class="linechart"/>
            <barchart v-if="chartdata.labels && chartdata.datasets && item.type == 'barchart'"
                      :datasets="chartdata.datasets3" :labels="chartdata.labels"></barchart>
          </div>
        </div>
      </div>
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
      loadTimer: 0,
      cards: [
        {
          index: 0,
          title: "кол-во продаж",
          type: "linechart",
        },
        {
          index: 1,
          title: "сум. кол-во продаж",
          type: "info",
          value: 140
        },
        {
          index: 2,
          title: "сред. кол-во продаж",
          type: "info",
          value: 46
        },
        {
          index: 3,
          title: "прибыль",
          type: "barchart",
        },
        {
          index: 4,
          title: "сум. прибыль",
          type: "info",
          value: 50
        },
        {
          index: 5,
          title: "сред. прибыль",
          type: "info",
          value: 34
        },
        {
          index: 6,
          title: "эффективность продаж",
          type: "linechart"
        },
        {
          index: 7,
          title: "сум. эффек-ность. продаж",
          type: "info",
          value: 86
        },
        {
          index: 8,
          title: "сред. эффек-ность. продаж",
          type: "info",
          value: 64
        },
        {
          index: 9,
          title: "окупаемость",
          type: "info",
          value: 40
        },
        {
          index: 10,
          title: "средний НДС",
          type: "info",
          value: 10
        },
        {
          index: 11,
          title: "сред. стоимость клиента",
          type: "info",
          value: 1000
        },
        {
          index: 12,
          title: "сред. стоимость перехода",
          type: "info",
          value: 800
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
        datasets2: [
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
        datasets3: [
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
      filters: [
        {
          field: "источник",
          field_name: "source",
          values: [],
          selected: "источник"
        },
        {
          field: "менеджер",
          field_name: "manager",
          values: [],
          selected: "менеджер"
        },
        {
          field: "тип рекламы",
          field_name: "typeofad",
          values: [],
          selected: "тип рекламы"
        },
        {
          field: "по дням",
          field_name: "bydate",
          values: ["по неделям", "по месяцам", "по годам"],
          selected: "по дням"
        },
      ]
    }
  },
  mounted() {
    this.$api.get('source').then(res => {
      // console.log(res)
      for (let i = 0; i < this.filters.length; i++) {
        if (this.filters[i].field_name == "source") {
          res.forEach(item => {
            this.filters[i].values.push(item.title)
          })
        }
      }
    })
    this.$api.get('medium').then(res => {
      console.log(res)
    })
    this.$api.get('tarif').then(res => {
      console.log(res)
    })
    this.$api.get('seller').then(res => {
      // console.log(res)
      for (let i = 0; i < this.filters.length; i++) {
        if (this.filters[i].field_name == "manager") {
          res.forEach(item => {
            this.filters[i].values.push(item.name)
          })
        }
      }
    })
    // this.$api.get('/tarif').then(res=>{
    //   console.log(res)
    // })
  },
  methods: {
    load() {
      // console.log(this.filters.length)
      clearTimeout(this.timer);
      // console.log(this.$refs)
      this.timer = setTimeout(() => {
        let query = '/data?'
        // console.log(this.$refs.search,this.$refs.datefrom,this.$refs.dateto)
        query += `search=${this.$refs.search.value}`
        query += `&datefrom=${this.$refs.datefrom.value}`
        query += `&dateto=${this.$refs.dateto.value}`
        for (let i = 0; i < this.filters.length; i++) {
          query += `&${this.filters[i].field_name}=${this.filters[i].selected}`
        }
        this.$api.get(query).then(res => {
          console.log(res)
        })
      }, 700);
    },
    touchStart(event, item) {
      if (window.mobileAndTabletCheck() && !this.drag.item && !this.drag.el) {
        this.drag.item = item
        let parent = event.target
        for (let i = 0; i < 10; i++) {
          if (parent.classList.contains('card')) {
            this.drag.el = parent
            break
          } else {
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
            event.pageX + document.body.scrollLeft >= card.offsetLeft &&
            event.pageX + document.body.scrollLeft <= card.offsetWidth + card.offsetLeft &&
            event.pageY + document.body.scrollTop >= card.offsetTop &&
            event.pageY + document.body.scrollTop <= card.offsetHeight + card.offsetTop) {
            let newIndex = this.cards[index].index
            this.cards[index].index = item.index
            this.cards.map(c => {
              if (c != this.cards[index] && c.index == item.index) {
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
::-webkit-calendar-picker-indicator {
  filter: invert(1);
}

.cards {
  display: flex;
  flex-wrap: wrap;
  //align-items: flex-start;
  //display: grid;
  //align-items: auto;
  .card-wrapper {
    display: inline-flex;
    vertical-align: top;
    text-align: justify;
    margin: 12px;
    width: calc(33.3vw - 24px);
    min-height: 256px;
    @media screen and (max-width: 768px) {
      width: calc(50vw - 24px);
    }
    background: rgba(220, 220, 220, 0.25);
    border-radius: 4px;
  }

  .card {
    width: calc(33.3vw - 24px);
    height: 256px;
    @media screen and (max-width: 768px) {
      width: calc(50vw - 24px);
    }
    padding: 6px;
    background: #e1e1e1;
    border-radius: 4px;
    font-size: var(--font-xxl);
    position: absolute;
    color: var(--color-accent);
    cursor: pointer;
    user-select: none;

    > * {
      pointer-events: none;
    }

    &.active {
      z-index: 999;
      box-shadow: 0px 4px 20px rgba(0, 0, 0, 0.03);
    }

    .title {
      position: absolute;
      left: 12px;
      top: 12px;
      font-size: var(--font-s);
      color: var(--color-font);

      &.info {
        font-size: calc(var(--font-xl) * 1.2);
        top: auto;
        bottom: 12px;
      }
    }

    p {
      position: absolute;
      top: 50%;
      left: 50%;
      font-family: "Rothek";
      font-size: calc(var(--font-xl) * 4);
      transform: translate(-50%, -50%);
    }

    i {
      position: absolute;
    }
  }
}
.chart-wrapper {
  height: 100%;

  > * {
    height: 100%;
  }

  @media screen and (max-width: 768px) {
    width: 100%;
    overflow-x: scroll;
    overflow-y: hidden;
    border-radius: var(--border-radius);
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

    button.execute {
      margin-left: auto;
    }
  }
}

.filters {
  margin: 12px;
  display: flex;
  flex-wrap: wrap;
  gap: 32px;
}

.search {
  display: flex;
  gap: 24px;
  margin: 24px 12px;

  .input {
    margin-top: 0;
    width: 100%;
  }
}
</style>
