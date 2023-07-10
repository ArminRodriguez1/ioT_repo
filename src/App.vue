<template>
  <div class="dashboard">
    <h1>Dashboard del Barril con Agua</h1>

    <!-- Gráfico de nivel de agua -->
    <div class="chart-container">
      <canvas ref="waterLevelChart"></canvas>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js';
import axios from 'axios';

class Authentication {
  constructor(username, password) {
    this.username = username;
    this.password = password;
    this.accessToken = '';
  }

  async authenticate() {
    try {
      const response = await axios.post(
        'http://iot.ceisufro.cl:8080/api/auth/login',
        {
          username: this.username,
          password: this.password
        },
        {
          headers: {
            'Content-Type': 'application/json',
            Accept: 'application/json'
          }
        }
      );
      if (response.status === 200 && response.data?.token) {
        this.accessToken = response.data.token;
        console.log('Authentication successful. Access token:', this.accessToken);
      } else {
        console.error('Authentication failed');
      }
    } catch (error) {
      console.error('Authentication failed:', error);
    }
  }
}

class DataFetcher {
  constructor(accessToken) {
    this.accessToken = accessToken;
  }

  async fetchData() {
    try {
      const response = await axios.get(
        'http://iot.ceisufro.cl:8080/api/plugins/telemetry/DEVICE/4c278870-162e-11ee-b199-3d650e5455ce/values/timeseries?key=distance',
        {
          headers: {
            'Content-Type': 'application/json',
            'X-Authorization': `Bearer ${this.accessToken}`
          }
        }
      );

      if (response.status === 200 && response.data?.distance?.[0]?.value) {
        const value = response.data.distance[0].value;
        console.log('Valor obtenido:', value);
        return value;
      }
    } catch (err) {
      console.error(err);
    }
  }
}

class WaterLevelChart {
  constructor(chartElement) {
    this.chartElement = chartElement;
    this.waterLevelChart = null;
  }

  createChart() {
    const waterLevelChartCtx = this.chartElement.getContext('2d');
    this.waterLevelChart = new Chart(waterLevelChartCtx, {
      type: 'doughnut',
      data: {
        labels: ['Distancia'],
        datasets: [
          {
            data: [0, 10],
            backgroundColor: ['#36A2EB', '#F0F0F0']
          }
        ]
      },
      options: {
        cutout: '70%',
        rotation: -Math.PI,
        circumference: Math.PI,
        plugins: {
          legend: {
            display: false
          }
        }
      }
    });
  }

  updateChart(value) {
    this.waterLevelChart.data.datasets[0].data[0] = value;
    this.waterLevelChart.data.datasets[0].data[1] = 10 - value;
    this.waterLevelChart.update();
  }
}

export default {
  name: 'DashboardView',
  data() {
    return {
      waterLevelChart: null
    };
  },
  mounted() {
    const authentication = new Authentication('o.morales01@ufromail.cl', 'harry2012');
    authentication.authenticate().then(() => {
      const dataFetcher = new DataFetcher(authentication.accessToken);
      this.waterLevelChart = new WaterLevelChart(this.$refs.waterLevelChart);
      this.waterLevelChart.createChart();

      setInterval(async () => {
        const value = await dataFetcher.fetchData();
        this.waterLevelChart.updateChart(value);
      }, 3000);
    });
  }
};
</script>

<style scoped>
.chart-container {
  width: 300px;
  height: 200px;
  margin: 20px;
}
</style>
