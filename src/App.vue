<template>
  <div class="chart-wrapper">
    <h3 v-if="isLoaded">{{ title }}</h3>
    <Pie v-if="isLoaded" :data="chartData" :options="options" />
    <div class="totalMW-wrapper" v-if="isLoaded">
      <h2>Total Megawatts</h2>
      <h1>{{ totalWatts }}</h1>
    </div>
  </div>
</template>

<script lang="ts">
import { Pie } from 'vue-chartjs'
import { Chart as ChartJS, ArcElement, Tooltip, Legend } from 'chart.js'

ChartJS.register(ArcElement, Tooltip, Legend)

export default {
  name: 'App',
  components: {
    Pie
  },
  data() {
    return {
      isLoaded: false,
      chartData: { datasets: [] },
      options: {},
      title: null,
      totalWatts: null
    }
  },
  async mounted() {
    this.isLoaded = false;

    try {
      const response = await fetch("https://api.misoenergy.org/MISORTWDDataBroker/DataBrokerServices.asmx?messageType=getfuelmix&returnType=json", {
        method: 'GET',
        headers: { 'Cookie': 'ASP.NET_SessionId=iagkb2ip3a3sxdwgytuq5jba' },
        redirect: 'follow'
      });

      const jsonData = await response.json();

      this.title = jsonData.RefId;
      this.totalWatts = parseInt(jsonData.TotalMW).toLocaleString();
      this.options = {
        responsive: true,
        maintainAspectRatio: false,
        elements: { arc: { borderWidth: 0 } },
        plugins: {
          legend: {
            position: 'right',
            labels: {
              color: '#333',
              boxWidth: 12, boxHeight: 12,
              font: { size: 16, weight: 600 }
            },

          }
        }
      };
      this.chartData = {
        labels: jsonData.Fuel.Type.map(({ FUEL_CATEGORY }) => FUEL_CATEGORY),
        datasets: [
          {
            backgroundColor: ['#595A5E', '#3274BE', '#D97331', '#83B12E', '#9D9EA0', '#F5C451',],
            data: jsonData.Fuel.Type.map(({ ACT }) => parseInt(ACT)),
          }
        ]
      };
      this.isLoaded = true;
    } catch (error) {
      console.log('error: ', error);
    }
  }
}
</script>

<style scoped>
.chart-wrapper {
  font-family: 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;
  color: #333;
  position: relative;
  width: 800px;
  height: 600px;
  margin: auto;
}

.totalMW-wrapper {
  position: absolute;
  top: 4rem;
  right: 0px;
}

.totalMW-wrapper h2 {
  margin-top: 0px;
  margin-bottom: 0.5rem;
  font-size: 2rem;
}

.totalMW-wrapper h1 {
  margin: 0px;
  text-align: right;
  font-size: 3rem;
}
</style>