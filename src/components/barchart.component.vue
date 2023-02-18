<template>
  <div class="shadow">
    <Bar
      :data="config.data"
      :options="config.options"
    />
  </div>
</template>

<script>
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale
} from 'chart.js'
import { Bar } from 'vue-chartjs'

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)
export default {
  name: 'barchart',
  components: {Bar},
  props: ['labels', 'datasets'],
  data() {
    return {
      config: {
        type: 'bar-chart',
        data: {
          labels: this.labels,
          datasets: this.datasets
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
div {
  padding: 16px;
  border-radius: var(--border-radius);
  background: var(--color-main);

  canvas {
    max-width: 100%;
    max-height: 500px;
  }
}
</style>
