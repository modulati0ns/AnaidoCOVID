<template>
  <v-app style="background: rgb(var(--v-theme-surface-variant))">
    <v-main style="align-items: flex-start" class="pa-6 m-3">
      <v-container style="max-width: 90rem" align="center">
        <v-card class="justify-center text-center pa-9 m-3">
          <v-containter class="spacing-playground p-8 m-3" fluid>
            <v-row class="justify-center">
              <h1>Anaida Covid</h1>
            </v-row>
            <v-row>
              <v-col>
                <v-btn @click="selectJavi"><h2>Javi</h2></v-btn>
              </v-col>
              <v-col>
                <v-btn @click="selectAnaida"><h2>Anaida</h2></v-btn>
              </v-col>
            </v-row>
            <v-spacer />
            <v-row class="wrapper">
              <vue3-chart-js ref="chartRef" v-bind="{ ...lineChart }" />
            </v-row>
            <v-template v-if="selectedUser == 'Javi' || selectedUser == 'Anaida'">
              <v-row class="mt-8">
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

<script setup>
import { reactive, onMounted, ref } from "vue";
import Vue3ChartJs from "@j-t-mcc/vue3-chartjs";
import zoomPlugin from "chartjs-plugin-zoom";
import "chartjs-adapter-moment";
import { initializeApp } from "firebase/app";
import { getDatabase, ref as fbRef, child, get, set } from "firebase/database";
Vue3ChartJs.registerGlobalPlugins([zoomPlugin]);

// Variables de Firebase
const firebaseConfig = {
  apiKey: "AIzaSyCAma9PeZlkqCzkch9wT6NisdWMIPTJ-oU",
  authDomain: "anaidacovid-18bbc.firebaseapp.com",
  projectId: "anaidacovid-18bbc",
  storageBucket: "anaidacovid-18bbc.appspot.com",
  messagingSenderId: "779192310492",
  appId: "1:779192310492:web:0e6d044590e77bab34b219",
};

// Initialize Firebase
initializeApp(firebaseConfig);
const dbRef = fbRef(getDatabase());

const lineChart = {
  type: "line",
  // locally registered and available for this chart
  data: {
    labels: [],
    datasets: [
      {
        label: "Temperatura corporal",
        data: [],
        fill: false,
        borderColor: "#41B883",
        backgroundColor: "black",
        tension: 0.4,
      },
    ],
  },
  options: {
    scales: {
      x: {
        type: "time",
        time: {
          unit: "hour",
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
    },
  },
};

const chartRef = ref(null);

let selectedUser = ref(null);
let temperatureValueToAdd = ref(null);
let rawData = reactive([]);

onMounted(() => {
  setInterval(function () {
    chartRef.value.update(250);
  }, 1000);
});

function selectUserFuncion() {
  rawData = [];

  var user = "";
  if (selectedUser.value == "Javi") {
    user = "dataJavi";
  } else {
    user = "dataAnaida";
  }
  var dates = [];
  var temps = [];
  get(child(dbRef, user))
    .then((snapshot) => {
      if (snapshot.exists()) {
        // Obtain dates from dataProcessed Array
        for (var i = 0; i < snapshot.val().length; i++) {
          dates.push(snapshot.val()[i].date);
          temps.push(snapshot.val()[i].temp);
          rawData.push({
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
  lineChart.data.labels = dates;
  lineChart.data.datasets[0].data = temps;
}

function selectAnaida() {
  selectedUser.value = "Anaida";
  selectUserFuncion();
}

function selectJavi() {
  selectedUser.value = "Javi";
  selectUserFuncion();
}

function addTemperature() {
  let user = "";
  if (selectedUser.value == "Javi") {
    user = "dataJavi";
  } else {
    user = "dataAnaida";
  }

  rawData.push({
    date: new Date().toISOString().toString(),
    temp: temperatureValueToAdd.value,
  });

  const db = getDatabase();
  set(fbRef(db, user), rawData);

  selectUserFuncion();
}
</script>
