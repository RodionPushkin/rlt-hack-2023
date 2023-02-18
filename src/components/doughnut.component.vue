<template>
  <div class="shadow">
    <Doughnut
      :data="config.data"
      :options="config.options"
    />
  </div>
</template>

<script>
import { Chart as ChartJS, ArcElement, Tooltip, Legend } from 'chart.js'
import { Doughnut } from 'vue-chartjs'

ChartJS.register(ArcElement, Tooltip, Legend)
export default {
  name: 'doughnut',
  components: {Doughnut},
  props: ['labels', 'datasets'],
  data() {
    return {
      config: {
        type: 'doughnut',
        data: {
          // labels: this.labels,
          datasets: this.datasets
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
div {
  //padding: 16px;
  //border-radius: var(--border-radius);
  //background: var(--color-main);

  canvas {
    max-width: 100%;
    aspect-ratio: 1/1;
  }
}
</style>
