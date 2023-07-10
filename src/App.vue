<template>
  <div class="dashboard">
    <nav class="navbar">
      <ul class="navbar-nav">
        <li class="nav-item">
          <a class="nav-tittle">Dashboard</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Inicio</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Acerca de</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Contacto</a>
        </li>
      </ul>
    </nav>
    <!-- Gráfico de nivel de agua -->
    <div class="chart-container">
      <canvas ref="waterLevelChart"></canvas>
    </div>
    <div class="card">
  <div class="card-body">
    <h5 class="card-title">Temperatura</h5>
    <p class="card-text">10°C</p>
  </div>
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
      type: 'bar',
      data: {
        labels: ['Distancia'],
        datasets: [
          {
            data: [0, 200],
            backgroundColor: [
              'rgba(0, 0, 255, 0.5)', 
              'rgba(255, 255, 255, 0.2)'
            ],
            borderColor: [
              'rgba(0, 0, 255, 1)', 
              'rgba(255, 255, 255, 1)' 
            ],
            borderWidth: 1
          }
        ]
      },
      options: {
        indexAxis: 'y', 
        scales: {
          y: {
            min: 0,
            max: 200,
            ticks: {
              stepSize: 10 
            }
          }
        },
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
    this.waterLevelChart.data.datasets[0].data[1] = 150 - value;
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
/* Estilos de la barra de navegación */
.navbar {
  background-color: #f8f8f8;
  padding: 10px;
}

.chart-container {
  width: 600px;
  height: 300px;
  margin: 20px;
}

.navbar-nav {
  display: flex;
  list-style: none;
  padding: 0;
  margin: 0;
}

.nav-item {
  margin-right: 10px;
}

.nav-link {
  color: #333;
  text-decoration: none;
  padding: 5px 10px;
}

.nav-link:hover {
  background-color: #333;
  color: #fff;
}

.nav-title {
  font-size: 24px;
  font-weight: bold;
  color: blue;
}
.card {
  width: 200px;
  margin: 20px;
  background-color: #f8f8f8;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 10px;
}

.card-title {
  font-size: 18px;
  font-weight: bold;
}

.card-text {
  font-size: 16px;
  color: #333;
}
</style>