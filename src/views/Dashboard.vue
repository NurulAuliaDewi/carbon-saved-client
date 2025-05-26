<template>
  <div>
    <!-- Navbar -->
    <nav class="navbar">
      <div class="container-banner">
        <img src="../assets/logob2w.png" alt="Logo">
      </div>
      <div class="container-fluid">
        <span class="navbar-brand mb-0 h2" style="color: #ffffff">DASHBOARD</span>
      </div>
    </nav>
    
    <br>
    
    <div class="content">
      <!-- Bike Commuting Data Section -->
      <div class="sub-title mb-5">
        <h4>Bike Commuting Data</h4>
      </div>
      
      <!-- Period Filter for Charts -->
      <div class="filter-chart-section my-5">
        <div class="d-flex justify-content-end align-items-center">
          <b-dropdown 
            id="dropdown-chart-filter" 
            size="md" 
            text="Filter by Period" 
            class="m-md-2 button-primary"
          >
            <b-dropdown-item @click="updatePeriodAll('day')">Daily</b-dropdown-item>
            <b-dropdown-item @click="updatePeriodAll('month')">Monthly</b-dropdown-item>
            <b-dropdown-item @click="updatePeriodAll('year')">Yearly</b-dropdown-item>
          </b-dropdown>
        </div>
      </div>
      
      <!-- Summary Cards -->
      <div class="row">
        <div 
          v-for="card in summaryCards" 
          :key="card.id" 
          class="col-md-4"
        >
          <div class="card text-center">
            <div 
              class="card-header" 
              :style="{ backgroundColor: card.color, color: '#FFFFFF' }"
            >
              <h5 class="card-title">{{ card.title }}</h5>
            </div>
            <div class="card-body">
              <div class="d-flex align-items-center justify-content-center">
                <img 
                  :src="card.icon" 
                  :alt="card.alt" 
                  class="card-img-left" 
                  style="width: 40px; height: 40px; margin-right: 10px;"
                >
                <div class="card-divider"></div>
                <h4 class="card-text" :style="{ color: '#002147' }">
                  {{ card.value }}
                </h4>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <br>
      
      <!-- Charts Section -->
      <div class="row row-cols-1 row-cols-md-2 g-4">
        <div 
          v-for="chart in chartInfo" 
          :key="chart.id" 
          class="col"
        >
          <div class="card">
            <div class="card-body">
              <div :id="chart.id" class="chart-container"></div>
            </div>
          </div>
        </div>
      </div>
      
      <br><br>
      
      <!-- Summary Commuting Section -->
      <div class="sub-title mb-5">
        <h4>Summary Commuting</h4>
      </div>
      
      <!-- Search & Filter -->
      <div class="search-filter-container">
        <!-- Search -->
        <div class="search-container">
          <label class="sr-only text-white" for="search-input">Search</label>
          <b-input-group class="button-primary rounded w-100">
            <b-form-input 
              id="search-input"
              v-model="searchQuery" 
              class="button-primary" 
              placeholder="Search by Username"
            />
            <b-input-group-append class="button-primary">
              <span class="input-group-text button-primary rounded-right">
                <i class="fa-solid fa-search"></i>
              </span>
            </b-input-group-append>
          </b-input-group>
        </div>
        
        <!-- Filter -->
        <div class="filter-container">
          <b-dropdown 
            id="dropdown-table-filter" 
            size="md" 
            text="Filter by Period" 
            class="button-primary mb-2 mb-md-0 mr-2"
          >
            <b-dropdown-item @click="updatePeriod('day')">Daily</b-dropdown-item>
            <b-dropdown-item @click="updatePeriod('month')">Monthly</b-dropdown-item>
            <b-dropdown-item @click="updatePeriod('year')">Yearly</b-dropdown-item>
          </b-dropdown>
        </div>
      </div>
      
      <br>

      <!-- Athletes Table -->
      <div class="table-responsive table-container">
        <table class="table">
          <thead>
            <tr>
              <th scope="col">No</th>
              <th scope="col">Commuter</th>
              <th scope="col">Distance (KM)</th>
              <th scope="col">Moving Time</th>
              <th scope="col">Carbon Saving (kg CO<sub>2</sub>)</th>
            </tr>
          </thead>
          <tbody>
            <tr 
              v-for="(athlete, index) in filteredAthletes" 
              :key="athlete.id_athlete" 
              @click="goToDetailPage(athlete.id_athlete)" 
              class="table-row-clickable"
            >
              <td>{{ index + 1 }}</td>
              <td>{{ athlete.fullName }}</td>
              <td>{{ athlete.distance }}</td>
              <td>{{ athlete.movingTime }}</td>
              <td>{{ athlete.carbonSaving }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import ApexCharts from 'apexcharts';
import axios from 'axios';

// Constants
const API_BASE_URL = 'http://localhost:5000';
const DISTANCE_THRESHOLDS = {
  day: { short: 15, mid: 30 },
  month: { short: 300, mid: 500 },
  year: { short: 3600, mid: 6000 }
};

export default {
  name: 'HomeView',
  
  data() {
    return {
      // Data state
      athletes: [],
      totalDistanceAll: 0,
      totalMovingTimeAll: 0,
      totalCarbonSavingAll: 0,
      
      // UI state
      period: 'day',
      searchQuery: '',
      
      // Chart configuration
      chartInstances: [],
      chartInfo: [
        { id: 'chart1', type: 'line' },
        { id: 'chart2', type: 'bar' },
        { id: 'chart3', type: 'donut' },
        { id: 'chart4', type: 'area' },
      ],
    };
  },

  computed: {
    // Summary cards configuration
    summaryCards() {
      return [
        {
          id: 'distance',
          title: 'Total Distance',
          value: `${this.formatDistanceCard(this.totalDistanceAll)} KM`,
          icon: require('../assets/map.svg'),
          alt: 'distance',
          color: '#2A67AD'
        },
        {
          id: 'time',
          title: 'Total Moving Time',
          value: this.formatTime(this.totalMovingTimeAll),
          icon: require('../assets/clock.svg'),
          alt: 'time',
          color: '#F1CB39'
        },
        {
          id: 'carbon',
          title: 'Total Carbon Saving',
          value: `${this.formatCarbonSaving(this.totalCarbonSavingAll)} kg CO₂`,
          icon: require('../assets/carbon.svg'),
          alt: 'carbon',
          color: '#00887A'
        }
      ];
    },

    // Filtered athletes based on search query
    filteredAthletes() {
      if (!this.searchQuery.trim()) {
        return this.athletes;
      }
      
      const query = this.searchQuery.toLowerCase();
      return this.athletes.filter(athlete => 
        athlete.fullName.toLowerCase().includes(query)
      );
    },

    // Current period label for charts
    periodLabel() {
      return this.period.charAt(0).toUpperCase() + this.period.slice(1);
    },

    // Distance classification thresholds
    distanceThresholds() {
      return DISTANCE_THRESHOLDS[this.period];
    }
  },

  async mounted() {
    console.log("Component successfully mounted");
    try {
      await Promise.all([
        this.fetchDataAll(),
        this.fetchData()
      ]);
    } catch (error) {
      console.error("Error during component initialization:", error);
    }
  },

  beforeDestroy() {
    this.destroyCharts();
  },

  methods: {
    // Chart management
    destroyCharts() {
      this.chartInstances.forEach(chart => {
        try {
          chart.destroy();
        } catch (error) {
          console.warn("Error destroying chart:", error);
        }
      });
      this.chartInstances = [];
    },

    // Formatting utilities
    formatDistance(value) {
      return (value / 1000).toFixed(2);
    },

    formatDistanceCard(value) {
      return new Intl.NumberFormat('id-ID', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(value / 1000);
    },

    formatTime(seconds) {
      if (!seconds || seconds < 0) return '00:00:00';
      
      const hours = Math.floor(seconds / 3600);
      const minutes = Math.floor((seconds % 3600) / 60);
      const secs = Math.floor(seconds % 60);
      
      return [hours, minutes, secs]
        .map(unit => unit.toString().padStart(2, '0'))
        .join(':');
    },

    formatCarbonSaving(value) {
      return new Intl.NumberFormat('id-ID', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(value || 0);
    },

    // Navigation
    goToDetailPage(id) {
      this.$router.push({ path: `/detail/${id}` });
    },

    // Distance classification
    groupDistances(distances) {
      const { short: shortLimit, mid: midLimit } = this.distanceThresholds;
      const result = { short: 0, mid: 0, long: 0 };

      distances.forEach(distance => {
        if (distance > 0) {
          if (distance <= shortLimit) {
            result.short++;
          } else if (distance <= midLimit) {
            result.mid++;
          } else {
            result.long++;
          }
        }
      });

      return result;
    },

    // Period update handlers
    async updatePeriodAll(period) {
      this.period = period;
      try {
        await Promise.all([
          this.fetchDataAll(),
          this.fetchData()
        ]);
      } catch (error) {
        console.error("Error updating period for all data:", error);
      }
    },

    async updatePeriod(period) {
      this.period = period;
      try {
        await Promise.all([
          this.fetchData(),
          this.fetchDataAll()
        ]);
      } catch (error) {
        console.error("Error updating period:", error);
      }
    },

    // API calls
    async fetchDataAll() {
      try {
        const response = await axios.get(`${API_BASE_URL}/total`, {
          params: { period: this.period }
        });

        if (response.status === 200 && response.data.status === 200) {
          const apiData = response.data.data;
          const firstItem = apiData[0];
          
          if (firstItem) {
            this.totalDistanceAll = firstItem.total_distance || 0;
            this.totalMovingTimeAll = firstItem.total_time || 0;
            this.totalCarbonSavingAll = firstItem.total_carbon_saving || 0;
          } else {
            console.warn("No data found in API response");
            this.resetTotalData();
          }
        } else {
          console.error("Failed to fetch data from API, status:", response.status);
          this.resetTotalData();
        }
      } catch (error) {
        console.error("Error fetching total data:", error);
        this.resetTotalData();
      }
    },

    async fetchData() {
      try {
        this.athletes = [];
        
        const response = await axios.get(`${API_BASE_URL}/total-athlete`, {
          params: { period: this.period }
        });

        if (response.data.status === 200) {
          const apiData = response.data.data;
          this.athletes = this.processAthleteData(apiData);
          this.updateCharts(apiData);
        } else {
          console.error("Failed to fetch athlete data from API");
        }
      } catch (error) {
        console.error("Error fetching athlete data:", error);
      }
    },

    // Data processing
    processAthleteData(apiData) {
      return apiData.map(item => ({
        fullName: `${item.athlete_firstname || ''} ${item.athlete_lastname || ''}`.trim(),
        distance: this.formatDistanceCard(item.total_distance || 0),
        carbonSaving: parseFloat(item.total_carbon_saving || 0).toFixed(2),
        movingTime: this.formatTime(item.total_time || 0),
        id_athlete: item.id_athlete,
        activityTotal: item.total_activity_all || 0
      }));
    },

    resetTotalData() {
      this.totalDistanceAll = 0;
      this.totalMovingTimeAll = 0;
      this.totalCarbonSavingAll = 0;
    },

    // Chart management
    updateCharts(apiData) {
      this.destroyCharts();

      if (!apiData || apiData.length === 0) {
        console.warn("No data available for charts");
        return;
      }

      const chartData = this.prepareChartData(apiData);
      const chartOptions = this.getChartOptions(chartData);

      this.$nextTick(() => {
        this.renderCharts(chartOptions);
      });
    },

    prepareChartData(apiData) {
      // Calculate totals
      const totalActivity = apiData.reduce((sum, item) => 
        sum + parseFloat(item.total_activity || 0), 0
      );

      // Get top 10 by distance and carbon saving
      const top10ByDistance = [...apiData]
        .sort((a, b) => (b.total_distance || 0) - (a.total_distance || 0))
        .slice(0, 10);

      const top10ByCarbonSaving = [...apiData]
        .sort((a, b) => (b.total_carbon_saving || 0) - (a.total_carbon_saving || 0))
        .slice(0, 10);

      // Prepare data arrays
      const distances = apiData.map(item => this.formatDistance(item.total_distance || 0));
      const namesByDistance = top10ByDistance.map(item => 
        `${item.athlete_firstname || ''} ${item.athlete_lastname || ''}`.trim()
      );
      const totalDistances = top10ByDistance.map(item => 
        this.formatDistance(item.total_distance || 0)
      );
      const namesByCarbonSaving = top10ByCarbonSaving.map(item => 
        `${item.athlete_firstname || ''} ${item.athlete_lastname || ''}`.trim()
      );
      const totalCarbonSavings = top10ByCarbonSaving.map(item => {
        const carbonSaving = parseFloat(item.total_carbon_saving || 0);
        return carbonSaving.toFixed(2);
      });

      return {
        totalActivity,
        distances,
        namesByDistance,
        totalDistances,
        namesByCarbonSaving,
        totalCarbonSavings
      };
    },

    getChartOptions(data) {
      const { short: shortLimit, mid: midLimit } = this.distanceThresholds;
      const distanceGroups = this.groupDistances(data.distances);
      
      // Labels for distance classification
      const shortLabel = `≤ ${shortLimit} km`;
      const midLabel = `≤ ${midLimit} km`;
      const longLabel = `> ${midLimit} km`;

      return {
        donut: {
          chart: { type: 'donut' },
          series: [distanceGroups.short, distanceGroups.mid, distanceGroups.long],
          labels: [`Short (${shortLabel})`, `Mid (${midLabel})`, `Long (${longLabel})`],
          title: {
            text: `Total Range Distance per-${this.periodLabel}`,
            align: 'center',
            style: { fontSize: '11px', fontWeight: 'bold' },
          },
        },
        line: {
          chart: { type: 'line' },
          series: [{ name: 'Total Distance', data: data.totalDistances }],
          xaxis: { categories: data.namesByDistance },
          yaxis: { title: { text: 'Total Distance' } },
          title: {
            text: `Total Distance Top 10 Athletes\nper-${this.periodLabel}`,
            align: 'center',
            floating: true,
            margin: 60,
            style: { fontSize: '10px', fontWeight: 'bold' },
          },
        },
        bar: {
          chart: { type: 'bar' },
          series: [{ name: 'Total Activity', data: [data.totalActivity] }],
          xaxis: { categories: ['Total Activity'] },
          title: {
            text: `Total Activity per-${this.periodLabel}`,
            align: 'center',
            style: { fontSize: '11px', fontWeight: 'bold' },
          },
        },
        area: {
          chart: { type: 'area' },
          series: [{ name: 'Total Carbon Saving', data: data.totalCarbonSavings }],
          xaxis: { categories: data.namesByCarbonSaving },
          yaxis: { title: { text: 'Total Carbon Saving' } },
          title: {
            text: `Total Carbon Saved Top 10 Athletes\nper-${this.periodLabel}`,
            align: 'center',
            floating: true,
            margin: 60,
            style: { fontSize: '10px', fontWeight: 'bold' },
          },
        },
      };
    },

    renderCharts(chartOptions) {
      this.chartInfo.forEach(chart => {
        const chartElement = document.querySelector(`#${chart.id}`);
        if (!chartElement) {
          console.error(`Chart element with ID #${chart.id} not found`);
          return;
        }

        try {
          const chartInstance = new ApexCharts(chartElement, chartOptions[chart.type]);
          chartInstance.render();
          this.chartInstances.push(chartInstance);
        } catch (error) {
          console.error(`Error rendering chart with ID #${chart.id}:`, error);
        }
      });
    },
  },
};
</script>

<style scoped>
/* Button Styles */
.button-primary, 
#dropdown-chart-filter__BV_toggle_,
#dropdown-table-filter__BV_toggle_ {
  background-color: #2A67AD !important;
  border-color: #4f91dc !important;
  color: #b3cce8 !important;
}

.button-primary::placeholder {
  color: #b3cce8 !important;
}

/* Navigation Styles */
.container-banner img {
  width: 180px;
  height: auto;
}

.container-fluid {
  background-color: #002147;
  color: #b3cce8;
  display: flex;
  justify-content: center;
}

/* Content Styles */
.sub-title h4 {
  color: #002147;
  font-weight: bold;
}

.content {
  margin: 0 20px;
}

/* Card Styles */
.card-divider {
  border-left: 2px solid #002147;
  height: 40px;
  margin: 0 10px;
}

/* Chart Styles */
.chart-container {
  width: 100%;
  max-width: 400px;
  height: 300px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.chart-container .apexcharts-canvas {
  max-width: 100% !important;
  height: auto !important;
}

/* Search and Filter Styles */
.search-filter-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: space-between;
  align-items: center;
}

.search-container {
  flex: 1 1 0%;
  min-width: 0;
  max-width: 350px;
}

.filter-container {
  flex: 0 1 200px;
  min-width: 0;
}

/* Table Styles */
.table-container {
  max-height: 300px;
}

.table {
  border-collapse: collapse;
}

.table th, 
.table td {
  padding: 8px;
  text-align: left;
  border: 1px solid #ddd;
}

.table thead th {
  background-color: #C2D1E2;
  color: #002147;
  font-weight: bold;
  position: sticky;
  top: 0;
  z-index: 1;
}

.table-row-clickable {
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.table-row-clickable:hover {
  background-color: #f8f9fa;
}

/* Responsive Styles */
@media (max-width: 574px) {
  .chart-container {
    max-width: 350px;
    height: auto;
  }

  .table th, 
  .table td {
    font-size: 12px;
    padding: 5px;
  }

  .search-filter-container {
    justify-content: flex-start;
  }

  .search-container,
  .filter-container {
    flex-basis: 100%;
    margin-bottom: 10px;
  }
}
</style>