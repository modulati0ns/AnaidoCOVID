<template>
  <button type="button" @click="shuffleData">Add data</button>
  <LineChart :chartData="testData" :options="options" />
</template>

<script lang="ts">
import { computed, defineComponent, ref } from "vue";
import { LineChart, useLineChart } from "vue-chart-3";
import { Chart, ChartData, ChartOptions, registerables } from "chart.js";
import "chartjs-adapter-moment";

Chart.register(...registerables);

export default defineComponent({
  name: "App",
  components: { LineChart },
  setup() {
    const dataValues = ref([]);
    const dataLabels = ref([]);
    const toggleLegend = ref(true);

    const testData = computed<ChartData<"doughnut">>(() => ({
      labels: dataLabels.value,
      datasets: [
        {
          data: dataValues.value,
          backgroundColor: [
            "#77CEFF",
            "#0079AF",
            "#123E6B",
            "#97B0C4",
            "#A5C8ED",
          ],
          tension: 0.2,
        },
      ],
    }));

    const options = computed<ChartOptions<"line">>(() => ({
      scales: {
        x: {
          type: "time",
          time: {
            displayFormats: {
              quarter: "MMM YYYY",
            },
          },
        },
      },
      plugins: {
        legend: {
          position: "top",
        },
        title: {
          display: true,
          text: "Chart.js Doughnut Chart",
        },
      },
    }));

    const { lineChartProps, lineChartRef } = useLineChart({
      chartData: testData,
      options,
    });

    let index = ref(20);

    function shuffleData() {
      dataLabels.value.push();
      dataValues.value.push(index.value);
      //   testData._value.labels.push(new Date());
      //   testData._value.datasets[0].push(index.value);

      index.value++;
    }

    setInterval(function () {
      lineChartRef.value.update();
    }, 2000);

    return {
      shuffleData,
      testData,
      options,
      lineChartRef,
      lineChartProps,
    };
  },
  mounted() {
    if (localStorage.data == undefined) {
      localStorage.data = JSON.stringify(this.data);
    } else {
      this.data = localStorage.data;
    }

    let dataProcessed = JSON.parse(this.data);
    // console.log(JSON.parse(this.data));
    console.log(dataProcessed.data);
    var dates = [];
    var temps = [];
    // Obtain dates from dataProcessed Array
    for (var i = 0; i < dataProcessed.data.length; i++) {
      dates.push(dataProcessed.data[i].date);
      temps.push(dataProcessed.data[i].temp);
    }
    this.testData.labels = dates;
    this.testData.datasets[0].data = temps;
  },
});
</script>

<style lang="sass">
 @import '~vuetify/src/styles/main.sass';
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
