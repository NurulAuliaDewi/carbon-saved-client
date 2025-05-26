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
      <!-- Header Section -->
      <div class="sub-title mb-5">
        <h4>Detail Information {{ athleteName }}</h4>
      </div>
      
      <!-- Period Filter -->
      <div class="filter-chart-section my-5">
        <div class="d-flex justify-content-end align-items-center">
          <b-dropdown 
            id="dropdown-1" 
            size="md" 
            text="Filter by Period" 
            class="m-md-2 button-primary"
          >
            <b-dropdown-item @click="updatePeriodAthlete('day')">Daily</b-dropdown-item>
            <b-dropdown-item @click="updatePeriodAthlete('month')">Monthly</b-dropdown-item>
            <b-dropdown-item @click="updatePeriodAthlete('year')">Yearly</b-dropdown-item>
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
      
      <br><br>
      
      <!-- Loading Message -->
      <div v-if="loading" class="alert alert-info">Loading activities...</div>
      
      <!-- Error Message -->
      <div v-if="error" class="alert alert-danger">{{ error }}</div>
      
      <!-- No Activities Message -->
      <div v-if="!loading && activities.length === 0" class="alert alert-warning">
        No activities found for this athlete.
      </div>
      
      <br>
      
      <!-- Table of Activities -->
      <div v-if="activities.length > 0" class="table-responsive table-container">
        <table class="table">
          <thead>
            <tr>
              <th scope="col">No</th>
              <th scope="col">Commuter</th>
              <th scope="col">Route Name</th>
              <th scope="col">Distance (km)</th>
              <th scope="col">Moving Time</th>
              <th scope="col">Activity Date</th>
              <th scope="col">Elevation Gain</th>
              <th scope="col">Carbon Saving (kg CO<sub>2</sub>)</th>
            </tr>
          </thead>
          <tbody>
            <tr 
              v-for="(activity, index) in activities" 
              :key="activity.id"
            >
              <td>{{ index + 1 }}</td>
              <td>{{ activity.athlete_firstname }} {{ activity.athlete_lastname }}</td>
              <td>{{ activity.activity_name }}</td>
              <td>{{ formatDistance(activity.distance) }}</td>
              <td>{{ formatTime(activity.moving_time) }}</td>
              <td>{{ new Date(activity.created_at).toLocaleDateString() }}</td>
              <td>{{ activity.total_elevation_gain }}</td>
              <td>{{ formatCarbonSaving(activity.carbon_saving) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

// Constants
const API_BASE_URL = 'http://localhost:5000';

export default {
  name: "DetailView",
  
  data() {
    return {
      // Data state
      activities: [],
      loading: false,
      error: null,
      totalDistance: 0,
      totalMovingTime: 0,
      totalCarbonSaving: 0,
      period: 'day',
      athleteName: '',
    };
  },

  computed: {
    // Summary cards configuration
    summaryCards() {
      return [
        {
          id: 'distance',
          title: 'Total Distance',
          value: `${this.totalDistance} KM`,
          icon: require('../assets/map.svg'),
          alt: 'distance',
          color: '#2A67AD'
        },
        {
          id: 'time',
          title: 'Total Moving Time',
          value: this.totalMovingTime,
          icon: require('../assets/clock.svg'),
          alt: 'time',
          color: '#F1CB39'
        },
        {
          id: 'carbon',
          title: 'Total Carbon Saving',
          value: `${this.formatCarbonSaving(this.totalCarbonSaving)} kg CO₂`,
          icon: require('../assets/carbon.svg'),
          alt: 'carbon',
          color: '#00887A'
        }
      ];
    }
  },

  mounted() {
    this.loadActivities();
    this.fetchDataAthlete();
  },

  methods: {
    // Formatting utilities
    formatDistance(value) {
      return new Intl.NumberFormat('id-ID', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(value / 1000);
    },

    formatTime(seconds) {
      const hours = Math.floor(seconds / 3600);
      const minutes = Math.floor((seconds % 3600) / 60);
      const secs = seconds % 60;
      return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    },

    formatCarbonSaving(value) {
      return new Intl.NumberFormat('id-ID', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(value);
    },

    // Period filtering
    filterActivitiesByPeriod() {
      const currentDate = new Date();
      return this.activities.filter(activity => {
        const activityDate = new Date(activity.date);
        const timeDifference = currentDate - activityDate;
        const twentyFourHours = 24 * 60 * 60 * 1000;

        if (this.period === 'day') {
          return timeDifference <= twentyFourHours;
        } else if (this.period === 'month') {
          return activityDate.getMonth() === currentDate.getMonth() && 
                 activityDate.getFullYear() === currentDate.getFullYear();
        } else if (this.period === 'year') {
          return activityDate.getFullYear() === currentDate.getFullYear();
        }
        return true;
      });
    },

    // API calls
    async loadActivities() {
      this.loading = true;
      this.error = null;

      try {
        const athleteId = parseInt(this.$route.params.id, 10);
        const response = await axios.get(`${API_BASE_URL}/athlete/${athleteId}`);
        
        if (response.data && response.data.data) {
          const athleteActivities = response.data.data;
          
          // Set athlete name from first activity
          if (athleteActivities.length > 0) {
            this.athleteName = athleteActivities[0].athlete_firstname + ' ' + athleteActivities[0].athlete_lastname;
          }
          
          // Store activities and filter by period
          this.activities = athleteActivities;
          this.activities = this.filterActivitiesByPeriod();
        } else {
          this.error = "No activities found for this athlete.";
          console.error("Data aktivitas tidak ditemukan.");
        }
      } catch (error) {
        this.error = "Error fetching activities. Please try again later.";
        console.error("Error fetching activities:", error);
      } finally {
        this.loading = false;
      }
    },

    async fetchDataAthlete() {
      try {
        console.log("Fetching data from API...");
        const response = await axios.get(`${API_BASE_URL}/total-athlete?period=${this.period}`);
        
        if (response.data.status === 200) {
          const apiData = response.data.data;
          const athleteId = parseInt(this.$route.params.id, 10);
          const athleteData = apiData.find(item => item.id_athlete === athleteId);
          
          if (athleteData) {
            const carbonSaving = parseFloat(athleteData.total_carbon_saving);
            this.totalDistance = this.formatDistance(athleteData.total_distance);
            this.totalMovingTime = this.formatTime(athleteData.total_time);
            this.totalCarbonSaving = carbonSaving.toFixed(2);
          } else {
            console.error("Athlete data not found for ID:", athleteId);
          }
        } else {
          console.error("Failed to fetch data from API. Status not 200.");
        }
      } catch (error) {
        console.error("Error saat memanggil API:", error);
      }
    },

    // Event handlers
    updatePeriodAthlete(period) {
      this.period = period;
      this.fetchDataAthlete();
      this.loadActivities();
    }
  },
};
</script>

<style scoped>
/* Button Styles */
.button-primary, 
#dropdown-1__BV_toggle_ {
  background-color: #2A67AD !important;
  border-color: #4f91dc !important;
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

/* Responsive Styles */
@media (max-width: 574px) {
  .table th, 
  .table td {
    font-size: 12px;
    padding: 5px;
  }
}
</style>