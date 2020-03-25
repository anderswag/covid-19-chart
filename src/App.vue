<template>
  <div id="app">
    <time-series-chart :chart-data="datacollection" :options="options" />
  </div>
</template>

<script>
import TimeSeriesChart from "./components/TimeSeriesChart";
const STORAGE_KEY_DATA = "data";

export default {
  components: {
    TimeSeriesChart
  },
  data() {
    return {
      datacollection: null,
      options: {
        responsive: true
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

      if (!localStorageData) {
        fetch("https://pomber.github.io/covid19/timeseries.json")
          .then(response => response.json())
          .then(data => {
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
