<template>
  <div>
    <div>
      <canvas ref="chartCanvas" width="800" height="400"></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Ref, Vue, Watch } from "vue-property-decorator";
import Chart from "chart.js/auto";

@Component({ components: {} })
export default class ScaleChart extends Vue {
  barChart!: any;
  @Prop({ default: [12, 11, 10] })
  data!: Array<number>;

  @Ref()
  chartCanvas!: HTMLCanvasElement;

  @Watch("data")
  onDataChanged(newVal: Array<number>): void {
    console.log("data ScaleChart changed");
    this.setupChart();
  }

  mounted(): void {
    this.setupChart();
  }

  setupChart(): void {
    this.setupBarChart();
  }

  setupBarChart(): void {
    this.barChart = new Chart(this.chartCanvas, {
      type: "bar",
      data: {
        labels: Object.keys(this.data),
        datasets: [
          {
            // label: '# of Votes',
            data: this.data,
            backgroundColor: ["rgba(255, 99, 132, 0.2)"],
            borderColor: ["rgba(255, 99, 132, 1)"],
            borderWidth: 1
          }
        ]
      },
      options: {
        scales: {
          y: {
            beginAtZero: true
          }
        }
      }
    });
  }
}
</script>

<style scoped></style>
