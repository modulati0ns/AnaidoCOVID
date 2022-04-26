<template>
  <v-app>
    <v-main style="width: 100rem" class="justify-center pa-6 m-3">
      <v-container align="center">
        <v-card class="justify-center text-center pa-6 m-3">
          <v-containter class="spacing-playground p-6 m-3" fluid>
            <v-row class="justify-center">
              <h1>Anaida Covid</h1>
            </v-row>
            <v-row>
              <v-col>
                <v-btn @click="selectJavi">Javi</v-btn>
              </v-col>
              <v-col>
                <v-btn @click="selectAnaida">Anaida</v-btn>
              </v-col>
            </v-row>
            <v-spacer />
            <v-row>
              <vue3-chart-js ref="chartRef" v-bind="{ ...lineChart }" />
            </v-row>
            <v-template
              v-if="selectedUser == 'Javi' || selectedUser == 'Anaida'"
            >
              <v-row>
                <v-col cols="10">
                  <v-text-field
                    v-model="temperatureValueToAdd"
                    label="Temperatura actual"
                    required
                  ></v-text-field>
                </v-col>
                <v-col cols="2">
                  <v-btn @click="addTemperature" size="x-large">Añadir</v-btn>
                </v-col>
              </v-row>
            </v-template>
          </v-containter>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script script lang="ts">
import { reactive, onMounted, ref } from "vue";
import Vue3ChartJs from "@j-t-mcc/vue3-chartjs";
import zoomPlugin from "chartjs-plugin-zoom";
import dataLabels from "chartjs-plugin-datalabels";
import "chartjs-adapter-moment";
import { initializeApp } from "firebase/app";
import { getDatabase, ref as fbRef, child, get, set } from "firebase/database";
Vue3ChartJs.registerGlobalPlugins([zoomPlugin]);
export default {
  name: "App",
  components: {
    Vue3ChartJs,
  },
  setup() {
    // Variables de Firebase
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
    initializeApp(firebaseConfig);
    const dbRef = fbRef(getDatabase());

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
                enabled: false,
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

    const users = ["Javi", "Anaida"];
    const selectedUser = ref(null);
    const temperatureValueToAdd = ref(null);

    onMounted(() => {
      console.log("beforeMount hook!");
      setInterval(function () {
        chartRef.value.update(250);
      }, 1000);
    });

    const rawData = reactive([]);

    return {
      rawData,
      temperatureValueToAdd,
      selectedUser,
      users,
      lineChart,
      chartRef,
      dbRef,
    };
  },
  methods: {
    selectUserFuncion() {
      this.rawData = [];
      console.log(this.selectedUser);
      let user = "";
      if (this.selectedUser == "Javi") {
        user = "dataJavi";
      } else {
        user = "dataAnaida";
      }
      var dates = [];
      var temps = [];
      get(child(this.dbRef, user))
        .then((snapshot) => {
          if (snapshot.exists()) {
            // Obtain dates from dataProcessed Array
            for (var i = 0; i < snapshot.val().length; i++) {
              dates.push(snapshot.val()[i].date);
              temps.push(snapshot.val()[i].temp);
              this.rawData.push({
                date: snapshot.val()[i].date,
                temp: snapshot.val()[i].temp,
              });
            }
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
    selectAnaida() {
      this.selectedUser = "Anaida";
      this.selectUserFuncion();
    },
    selectJavi() {
      this.selectedUser = "Javi";
      console.log(this.selectedUser == "Javi");
      this.selectUserFuncion();
    },
    addTemperature() {
      let user = "";
      if (this.selectedUser == "Javi") {
        user = "dataJavi";
      } else {
        user = "dataAnaida";
      }

      console.log(this.rawData);
      this.rawData.push({
        date: new Date().toISOString().toString(),
        temp: this.temperatureValueToAdd,
      });
      console.log(this.rawData);
      const db = getDatabase();
      set(fbRef(db, user), this.rawData);

      this.selectUserFuncion();
    },
  },
};
</script>
