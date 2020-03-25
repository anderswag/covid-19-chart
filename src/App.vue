<template>
  <div id="app">
    <time-series-chart :chart-data="datacollection" :options="options" />
  </div>
</template>

<script>
import TimeSeriesChart from "./components/TimeSeriesChart";

const STORAGE_KEY_DATA = "data";
const STORAGE_KEY_TIMESTAMP = "timestamp";
const ONE_HOUR = 60 * 60 * 1000;

export default {
  components: {
    TimeSeriesChart
  },
  data() {
    return {
      datacollection: null,
      options: {
        scales: {
          yAxes: [
            {
              ticks: {
                beginAtZero: true
              },
              gridLines: {
                display: true
              }
            }
          ],
          xAxes: [
            {
              gridLines: {
                display: false
              }
            }
          ]
        },
        legend: {
          display: true
        },
        responsive: true,
        maintainAspectRatio: false
      }
    };
  },
  mounted() {
    this.fillData();
  },
  methods: {
    fillData() {
      let collection = {
        labels: [],
        datasets: [
          {
            label: "Active",
            backgroundColor: "#d13e57",
            data: []
          }
        ]
      };
      const localStorageData = localStorage.getItem(STORAGE_KEY_DATA);
      const localStorageTimestamp = localStorage.getItem(STORAGE_KEY_TIMESTAMP);

      // Fetch if the data is 1 hours old
      const shouldFetch =
        !localStorageData ||
        Date.now() - JSON.parse(localStorageTimestamp) > ONE_HOUR;

      if (shouldFetch) {
        fetch("https://pomber.github.io/covid19/timeseries.json")
          .then(response => response.json())
          .then(data => {
            console.log("FETCHED DATA");
            data["Canada"].forEach(({ date, confirmed, deaths, recovered }) => {
              collection.labels.push(date);
              collection.datasets[0][STORAGE_KEY_DATA].push(
                confirmed - deaths - recovered
              );
            });
          })
          .finally(() => {
            this.datacollection = collection;
            localStorage.setItem(STORAGE_KEY_DATA, JSON.stringify(collection));
            localStorage.setItem(
              STORAGE_KEY_TIMESTAMP,
              JSON.stringify(Date.now())
            );
          });
      } else {
        this.datacollection = JSON.parse(localStorageData);
      }
    }
  }
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
