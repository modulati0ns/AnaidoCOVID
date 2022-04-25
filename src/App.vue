<template>
  <button type="button" @click="shuffleData">Add data</button>
  <div style="width: 70rem">
    <vue3-chart-js ref="chartRef" v-bind="{ ...lineChart }" />
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, ref } from "vue";
import Vue3ChartJs from "@j-t-mcc/vue3-chartjs";
import zoomPlugin from "chartjs-plugin-zoom";
import dataLabels from "chartjs-plugin-datalabels";
import "chartjs-adapter-moment";

import { initializeApp } from "firebase/app";
import { getDatabase, ref as fbRef, child, get } from "firebase/database";

Vue3ChartJs.registerGlobalPlugins([zoomPlugin]);

export default {
  name: "App",
  components: {
    Vue3ChartJs,
  },
  setup() {
    const lineChart = {
      type: "line",
      // locally registered and available for this chart
      plugins: [dataLabels],
      data: {
        labels: [],
        datasets: [
          {
            label: "Temperatura corporal",
            data: [],
            fill: false,
            borderColor: "#41B883",
            backgroundColor: "black",
          },
        ],
      },
      options: {
        scales: {
          x: {
            type: "time",
            time: {
              displayFormats: {
                quarter: "MMM YYYY",
              },
            },
          },
          y: {
            min: 36,
            max: 39,
          },
        },
        plugins: {
          zoom: {
            zoom: {
              wheel: {
                enabled: true,
              },
              pinch: {
                enabled: true,
              },
              mode: "y",
            },
          },
          // datalabels: {
          //   backgroundColor: function (context) {
          //     return context.dataset.backgroundColor;
          //   },
          //   borderRadius: 4,
          //   color: "white",
          //   font: {
          //     weight: "bold",
          //   },
          //   formatter: Math.round,
          //   padding: 6,
          // },
        },
      },
    };

    const chartRef = ref(null);

    setInterval(function () {
      chartRef.value.update(250);
    }, 1000);

    return {
      lineChart,
      chartRef,
    };
  },
  mounted() {
    const firebaseConfig = {
      apiKey: "AIzaSyAab2bG-GFO9tuzF4mSEqTt0vwa4cTr458",
      authDomain: "anaidacovid.firebaseapp.com",
      databaseURL: "https://anaidacovid-default-rtdb.firebaseio.com",
      projectId: "anaidacovid",
      storageBucket: "anaidacovid.appspot.com",
      messagingSenderId: "54511310540",
      appId: "1:54511310540:web:2ba52244709f6dd3c25469",
    };

    // Initialize Firebase
    const app = initializeApp(firebaseConfig);
    const dbRef = fbRef(getDatabase());
    const user = "dataJavi";

    var dates = [];
    var temps = [];

    get(child(dbRef, user))
      .then((snapshot) => {
        if (snapshot.exists()) {
          // Obtain dates from dataProcessed Array
          for (var i = 0; i < snapshot.val().length; i++) {
            dates.push(snapshot.val()[i].date);
            temps.push(snapshot.val()[i].temp);
          }
          console.log(snapshot.val());
        } else {
          console.log("No data available");
        }
      })
      .catch((error) => {
        console.error(error);
      });

    this.lineChart.data.labels = dates;
    this.lineChart.data.datasets[0].data = temps;
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
