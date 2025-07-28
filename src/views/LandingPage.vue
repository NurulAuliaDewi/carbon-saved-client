<template>
  <div class="landing">
    <div class="bg-animation">
      <div class="floating-shape shape-1"></div>
      <div class="floating-shape shape-2"></div>
      <div class="floating-shape shape-3"></div>
      <div class="floating-shape shape-4"></div>
    </div>

    <div class="hero-content">
      <div class="logo-container">
        <div class="logo">
          <i class="fas fa-bicycle"></i>
        </div>
        <div class="logo-glow"></div>
      </div>

      <h1 class="main-title">
        <span class="gradient-text">Bike to Work </span>
        <span class="highlight">Tracker</span>
      </h1>
      
      <p class="subtitle">
        <span class="typing-text">Track Your Journey, Save The Earth</span>
      </p>
      
      <p class="desc">
        Monitor your daily bike commuting activities, track your carbon savings, 
        and contribute to a <span class="highlight-green">greener future</span> with Bike to Work.
      </p>

      <div class="summary-section">
        <h2 class="summary-title">Summary Overview</h2>
        
        <div class="period-selector">
          <button class="period-btn active" @click="changePeriod('month', $event)">Monthly</button>
          <button class="period-btn" @click="changePeriod('semester', $event)">Semester</button>
        </div>

        <div class="month-selector" v-if="currentPeriod === 'month'">
          <select v-model="selectedMonth" @change="onMonthChange" class="month-dropdown">
            <option v-for="month in availableMonths" :key="month.value" :value="month.value">
              {{ month.label }}
            </option>
          </select>
        </div>

        <div class="semester-selector" v-if="currentPeriod === 'semester'">
          <select v-model="selectedSemester" @change="onSemesterChange" class="semester-dropdown">
            <option value="1">Semester 1 (Jan - Jun {{ currentYear }})</option>
            <option value="2">Semester 2 (Jul - Dec {{ currentYear }})</option>
          </select>
        </div>

        <div class="summary-grid">
          <div class="summary-card" v-for="(stat, index) in stats" :key="index">
            <div class="card-icon">
              <i :class="stat.icon"></i>
            </div>
            <div class="card-value">
              <span v-if="loading" class="loading-spinner">⟳</span>
              <span v-else>{{ stat.value }}</span>
            </div>
            <div class="card-label">{{ stat.label }}</div>
            <div class="card-subtitle">{{ stat.subtitle }}</div>
          </div>
        </div>

        <div class="activity-chart">
          <h3 class="chart-title">Weekly Activity Frequency</h3>
          <div class="chart-bar" v-for="activity in activities" :key="activity.label">
            <div class="bar-label">{{ activity.label }}</div>
            <div class="bar-container">
              <div class="bar-fill" :style="`width: ${activity.percentage}%`"></div>
            </div>
            <div class="bar-value">{{ activity.count }}</div>
          </div>
        </div>

        <div class="top-athletes-section">
          <h3 class="section-title">🏆 Top 20 Carbon Savers</h3>
          <div class="athletes-container">
            <div v-if="loadingAthletes" class="loading-container">
              <div class="loading-spinner-large">⟳</div>
              <p>Loading top athletes...</p>
            </div>
            <div v-else class="athletes-grid">
              <div 
                v-for="(athlete, index) in topAthletes" 
                :key="athlete.id || index"
                class="athlete-card"
                :class="{ 'top-three': index < 3 }"
              >
                <div class="athlete-rank">
                  <span v-if="index === 0" class="crown gold">👑</span>
                  <span v-else-if="index === 1" class="crown silver">🥈</span>
                  <span v-else-if="index === 2" class="crown bronze">🥉</span>
                  <span v-else class="rank-number">#{{ index + 1 }}</span>
                </div>
                <div class="athlete-info">
                  <div class="athlete-name">{{ athlete.name || 'Anonymous' }}</div>
                  <div class="athlete-stats">
                    <div class="stat-item">
                      <i class="fas fa-leaf"></i>
                      <span class="carbon-saved">{{ formatCarbon(athlete.carbon_saved) }}</span>
                    </div>
                    <div class="stat-item">
                      <i class="fas fa-route"></i>
                      <span class="distance">{{ formatDistance(athlete.total_distance) }}</span>
                    </div>
                  </div>
                </div>
                <div class="athlete-badge" v-if="index < 3">
                  <i class="fas fa-star"></i>
                </div>
              </div>
            </div>
            <div v-if="!loadingAthletes && topAthletes.length === 0" class="no-data">
              <i class="fas fa-info-circle"></i>
              <p>No athlete data available for this period</p>
            </div>
          </div>
        </div>
      </div>

      <button class="cta" @click="goToDashboard">
        <span class="cta-text">
          <i class="fas fa-chart-line"></i> 
          View Dashboard
        </span>
        <div class="cta-ripple"></div>
      </button>
    </div>

    <div class="scroll-indicator">
      <div class="scroll-dot"></div>
      <div class="scroll-dot"></div>
      <div class="scroll-dot"></div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'LandingPage',
  data() {
    return {
      loading: true,
      loadingAthletes: true,
      currentPeriod: 'month',
      currentYear: new Date().getFullYear(),
      selectedMonth: null, 
      selectedSemester: '1',
      availableMonths: [
        { value: '1', label: 'January 2024' },
        { value: '2', label: 'February 2024' },
        { value: '3', label: 'March 2024' },
        { value: '4', label: 'April 2024' },
        { value: '5', label: 'May 2024' },
        { value: '6', label: 'June 2024' },
        { value: '7', label: 'July 2024' },
        { value: '8', label: 'August 2024' },
        { value: '9', label: 'September 2024' },
        { value: '10', label: 'October 2024' },
        { value: '11', label: 'November 2024' },
        { value: '12', label: 'December 2024' }
      ],
      stats: [
        { icon: 'fas fa-users', value: '-', label: 'Total Participants', subtitle: 'Active cyclists', key: 'total_participants' },
        { icon: 'fas fa-bicycle', value: '-', label: 'Total Rides', subtitle: 'Cycling activities', key: 'total_activities' },
        { icon: 'fas fa-route', value: '-', label: 'Total Distance', subtitle: 'Kilometers covered', key: 'total_distance' },
        { icon: 'fas fa-clock', value: '-', label: 'Total Time', subtitle: 'Hours spent cycling', key: 'total_time' },
        { icon: 'fas fa-leaf', value: '-', label: 'CO₂ Saved', subtitle: 'Kilograms of emissions avoided', key: 'total_carbon_saved' },
        { icon: 'fas fa-chart-line', value: '-', label: 'Daily Average', subtitle: 'Kilometers per day', key: 'daily_average' }
      ],
      activities: [
        { label: 'Daily Cyclists', percentage: 0, count: '-', key: 'every_day' },
        { label: '4-6 Days/Week', percentage: 0, count: '-', key: '4_to_6_days' },
        { label: '2-3 Days/Week', percentage: 0, count: '-', key: '3_days' },
        { label: '1-2 Days/Week', percentage: 0, count: '-', key: 'less_than_3_days' }
      ],
      topAthletes: [],
      apiBaseUrl: process.env.VUE_APP_API_URL || 'http://localhost:5000'
    }
  },
  async mounted() {
    const now = new Date();
    this.selectedMonth = (now.getMonth() + 1).toString();
    
    await this.fetchAvailableMonths();
    await this.fetchSummaryData();
    await this.fetchTopAthletes();
  },
  methods: {
    formatNumber(num) {
      if (!num || num === null || num === undefined || isNaN(num)) return '-';
      const numValue = typeof num === 'string' ? parseFloat(num) : num;
      return new Intl.NumberFormat('id-ID', {
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
      }).format(Math.round(numValue));
    },

    formatNumberWithDecimals(num, decimals = 1) {
      if (!num || num === null || num === undefined || isNaN(num)) return '-';
      const numValue = typeof num === 'string' ? parseFloat(num) : num;
      return new Intl.NumberFormat('id-ID', {
        minimumFractionDigits: decimals,
        maximumFractionDigits: decimals
      }).format(numValue);
    },

    formatDistance(distance) {
      if (!distance || distance === null || distance === undefined) return '0 km';
      const distanceValue = typeof distance === 'string' ? parseFloat(distance) : distance;
      const kmValue = distanceValue / 1000;
      if (kmValue >= 1000) {
        return `${this.formatNumber(kmValue)} km`;
      } else {
        return `${this.formatNumberWithDecimals(kmValue, 1)} km`;
      }
    },

    formatCarbon(carbon) {
      if (!carbon || carbon === null || carbon === undefined) return '0 kg';
      const carbonValue = typeof carbon === 'string' ? parseFloat(carbon) : carbon;
      
      // Langsung format tanpa konversi karena data sudah dalam kg CO₂
      if (carbonValue >= 1000) {
        return `${this.formatNumber(carbonValue)} kg`;
      } else {
        return `${this.formatNumberWithDecimals(carbonValue, 1)} kg`;
      }
    },

    formatTimeHours(seconds) {
      if (!seconds || seconds === null || seconds === undefined) return '0 hrs';
      const secondsValue = typeof seconds === 'string' ? parseInt(seconds) : seconds;
      const hours = Math.round(secondsValue / 3600);
      return `${this.formatNumber(hours)} hrs`;
    },


    // ===== API METHODS =====
    async fetchAvailableMonths() {
      try {
        const response = await axios.get(`${this.apiBaseUrl}/available-periods`);
        
        if (response.data && response.data.status === 200 && response.data.data.months) {
          const monthNames = [
            'January', 'February', 'March', 'April', 'May', 'June',
            'July', 'August', 'September', 'October', 'November', 'December'
          ];
          
          this.availableMonths = response.data.data.months.map(item => ({
            value: item.month.toString(),
            label: `${monthNames[item.month - 1]} ${item.year}`,
            year: item.year
          }));
          
          if (this.availableMonths.length > 0) {
            const currentMonthAvailable = this.availableMonths.find(m => m.value === this.selectedMonth);
            if (!currentMonthAvailable) {
              this.selectedMonth = this.availableMonths[0].value;
            }
          }
        }
      } catch (error) {
        console.error('Error fetching available months:', error);
      }
    },

    async fetchTopAthletes() {
      try {
        this.loadingAthletes = true;
        let params = {};
        if (this.currentPeriod === 'month') {
          const selectedMonthData = this.availableMonths.find(m => m.value === this.selectedMonth);
          const month = parseInt(this.selectedMonth);
          const year = selectedMonthData ? selectedMonthData.year : this.currentYear;
          
          params = {
            period: 'month',
            month: month,
            year: year
          };
        } else if (this.currentPeriod === 'semester') {
          params = {
            period: 'year',
            year: this.currentYear
          };
        }
        
        const response = await axios.get(`${this.apiBaseUrl}/total-athlete`, { params });        
        console.log('Top athletes API response:', response.data);
        
        if (response.data && response.data.status === 200) {
          let athleteData = response.data.data || [];
          if (this.currentPeriod === 'semester') {
            console.log('Semester filtering - showing yearly data as fallback');
          }

          this.topAthletes = athleteData.map(athlete => ({
            id: athlete.id_athlete,
            name: athlete.athlete_firstname && athlete.athlete_lastname ? 
                  `${athlete.athlete_firstname} ${athlete.athlete_lastname}` : 
                  (athlete.athlete_firstname || athlete.athlete_lastname || 'Anonymous'),
            carbon_saved: athlete.total_carbon_saving,
            total_distance: athlete.total_distance,
            total_activities: athlete.total_activity,
            total_time: athlete.total_time
          })).slice(0, 20);
          
        } else {
          console.error('Invalid API response for top athletes:', response.data);
          this.topAthletes = [];
        }
        
      } catch (error) {
        console.error('Error fetching top athletes:', error);
        this.topAthletes = [];
      } finally {
        this.loadingAthletes = false;
      }
    },

    async fetchSummaryData() {
      try {
        this.loading = true;
        const now = new Date();
        const currentMonth = now.getMonth() + 1;
        const currentYear = now.getFullYear();
        console.log('Current month:', currentMonth, 'Current year:', currentYear);
        let apiUrl = `${this.apiBaseUrl}/summary-stats`;
        let params = {};
        if (this.currentPeriod === 'month') {
          const selectedMonthData = this.availableMonths.find(m => m.value === this.selectedMonth);
          const month = parseInt(this.selectedMonth);
          const year = selectedMonthData ? selectedMonthData.year : currentYear;
          
          params = {
            period: 'month',
            month: month,
            year: year
          };
        } else if (this.currentPeriod === 'semester') {
          params = {
            period: 'semester',
            year: this.currentYear,
            semester: this.selectedSemester
          };
        }
        
        console.log('Fetching data from:', apiUrl, 'with params:', params);
        console.log('Current period:', this.currentPeriod);
        console.log('Expected API call for semester: period=semester&year=2025&semester=1');
        const response = await axios.get(apiUrl, { params });
        console.log('API Response:', response.data);
        if (response.data && response.data.status === 200) {
          this.updateStatsFromAPI(response.data.data);
        } else {
          console.error('Invalid API response:', response.data);
        }
        
      } catch (error) {
        console.error('Error fetching summary data:', error);
      } finally {
        this.loading = false;
      }
    },
    
    updateStatsFromAPI(data) {
      console.log('Raw API data:', data);

      const currentDate = new Date();
      const daysInMonth = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0).getDate();
      const daysInPeriod = this.currentPeriod === 'month' ? daysInMonth : 180;
      const totalDistanceValue = typeof data.total_distance === 'string' ? 
        parseFloat(data.total_distance) : data.total_distance;
      const dailyAvgKm = totalDistanceValue ? (totalDistanceValue / 1000 / daysInPeriod) : 0;
    
      this.stats.forEach(stat => {
        try {
          switch(stat.key) {
            case 'total_participants':
              stat.value = this.formatNumber(data.total_participants);
              break;
            case 'total_activities':
              stat.value = this.formatNumber(data.total_activities);
              break;
            case 'total_distance':
              stat.value = this.formatDistance(data.total_distance);
              break;
            case 'total_time':
              stat.value = this.formatTimeHours(data.total_time);
              break;
            case 'total_carbon_saved':
              stat.value = this.formatCarbon(data.total_carbon_saved);
              break;
            case 'daily_average':
              if (dailyAvgKm >= 1000) {
                stat.value = `${this.formatNumber(dailyAvgKm)} km`;
              } else {
                stat.value = `${this.formatNumberWithDecimals(dailyAvgKm, 1)} km`;
              }
              break;
          }
          console.log(`Updated ${stat.key}:`, stat.value);
        } catch (error) {
          console.error(`Error updating ${stat.key}:`, error);
          stat.value = '-';
        }
      });

      if (data.frequency_stats) {
        console.log('Frequency stats:', data.frequency_stats);
        const freqStats = data.frequency_stats;
        const total = Object.values(freqStats).reduce((sum, val) => {
          const numVal = typeof val === 'string' ? parseInt(val) : val;
          return sum + (isNaN(numVal) ? 0 : numVal);
        }, 0);
        
        console.log('Total frequency:', total);
        this.activities.forEach(activity => {
          try {
            const count = freqStats[activity.key] || 0;
            const countValue = typeof count === 'string' ? parseInt(count) : count;
            const percentage = total > 0 ? Math.round((countValue / total) * 100) : 0;
            activity.count = this.formatNumber(countValue);
            activity.percentage = Math.max(percentage, 5);
          } catch (error) {
            console.error(`Error updating activity ${activity.key}:`, error);
            activity.count = '0';
            activity.percentage = 0;
          }
        });
      } else {
        console.log('No frequency stats found in API response');
      }
      
      this.$forceUpdate();
    },

    // ===== EVENT HANDLERS =====
    async onSemesterChange() {
      await this.fetchSummaryData();
      await this.fetchTopAthletes();
    },

    async onMonthChange() {
      await this.fetchSummaryData();
      await this.fetchTopAthletes();
    },

    async changePeriod(period, event) {
      document.querySelectorAll('.period-btn').forEach(btn => btn.classList.remove('active'));
      event.target.classList.add('active');
      this.currentPeriod = period;
      await this.fetchSummaryData();
      await this.fetchTopAthletes();
    },
    
    goToDashboard() {
      const button = event.currentTarget;
      const ripple = button.querySelector('.cta-ripple');
      ripple.style.animation = 'ripple 0.6s ease-out';
      setTimeout(() => {
        ripple.style.animation = '';
      }, 600);

      document.body.style.transition = 'opacity 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94)';
      document.body.style.opacity = '0.8';
      
      setTimeout(() => {
        this.$router.push('/Dashboard');
        document.body.style.opacity = '1';
      }, 500);
    }
  }
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.landing {
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #224d7e 0%, #0c2540 50%, #133a5a 100%);
  color: #ffffff;
  text-align: center;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow-x: hidden;
}

/* Animated Background */
.bg-animation {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
}

.floating-shape {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  animation: float 6s ease-in-out infinite;
}

.shape-1 { width: 80px; height: 80px; top: 10%; left: 10%; animation-delay: 0s; }
.shape-2 { width: 120px; height: 120px; top: 60%; right: 10%; animation-delay: 2s; }
.shape-3 { width: 60px; height: 60px; bottom: 30%; left: 20%; animation-delay: 4s; }
.shape-4 { width: 100px; height: 100px; top: 30%; right: 30%; animation-delay: 1s; }

@keyframes float {
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  50% { transform: translateY(-20px) rotate(180deg); }
}

/* Hero Content */
.hero-content {
  position: relative;
  z-index: 1;
  animation: fadeInUp 1s ease-out forwards;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}

.logo-container {
  position: relative;
  margin-bottom: 2rem;
}

.logo {
  width: 120px;
  height: 120px;
  background: #18cf55;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  animation: logoFloat 3s ease-in-out infinite;
  z-index: 2;
  position: relative;
  box-shadow: 0 20px 40px rgba(24, 207, 85, 0.3);
}

.logo i {
  font-size: 3rem;
  color: white;
}

.logo-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 150px;
  height: 150px;
  background: radial-gradient(circle, rgba(24, 207, 85, 0.3) 0%, transparent 70%);
  border-radius: 50%;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes logoFloat {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

@keyframes pulse {
  0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.6; }
  50% { transform: translate(-50%, -50%) scale(1.1); opacity: 0.8; }
}

.main-title {
  font-size: 3.5rem;
  font-weight: 900;
  margin-bottom: 1rem;
  line-height: 1.2;
}

.gradient-text {
  background: linear-gradient(45deg, #18cf55, #18cf55);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.highlight {
  color: #ffffff;
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.5);
}

.highlight-green {
  color: #18cf55;
  font-weight: 600;
}

.subtitle {
  font-size: 1.8rem;
  margin-bottom: 1.5rem;
  color: #e0e0e0;
  min-height: 2.5rem;
}

.typing-text {
  display: inline-block;
  border-right: 2px solid #60a5fa;
  animation: typing 3s steps(30, end) 1 forwards, blink 0.8s step-end 5;
}

@keyframes typing {
  from { width: 0; }
  to { width: 100%; }
}

@keyframes blink {
  50% { border-color: transparent; }
}

.desc {
  max-width: 600px;
  margin: 0 auto 3rem;
  font-size: 1.1rem;
  line-height: 1.6;
  color: #e0e0e0;
  opacity: 0.9;
}

/* Summary Section */
.summary-section {
  margin: 4rem 0;
}

.summary-title {
  font-size: 2.5rem;
  font-weight: 800;
  margin-bottom: 3rem;
  background: linear-gradient(45deg, #60a5fa, #18cf55);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.section-title {
  font-size: 2rem;
  font-weight: 700;
  margin: 4rem 0 2rem;
  background: linear-gradient(45deg, #ffd700, #ff6b35);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.period-selector {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 2rem;
}

.period-btn {
  padding: 0.8rem 2rem;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 30px;
  color: #e0e0e0;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 600;
  backdrop-filter: blur(10px);
}

.period-btn.active {
  background: linear-gradient(45deg, #18cf55, #149e42);
  border-color: #18cf55;
  color: white;
  box-shadow: 0 10px 30px rgba(24, 207, 85, 0.3);
  transform: translateY(-2px);
}

.period-btn:not(.active):hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-2px);
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  max-width: 1200px;
  margin: 0 auto 3rem;
}

.summary-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  padding: 2rem;
  transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  animation: cardSlideIn 0.8s ease-out forwards;
}

.summary-card:nth-child(n) {
  animation-delay: calc(0.1s * var(--index, 1));
}

@keyframes cardSlideIn {
  from { opacity: 0; transform: translateY(50px); }
  to { opacity: 1; transform: translateY(0); }
}

.summary-card:hover {
  transform: translateY(-10px) scale(1.02);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.3);
}

.card-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
  background: linear-gradient(45deg, #60a5fa, #18cf55);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.card-value {
  font-size: 2.5rem;
  font-weight: 800;
  color: #18cf55;
  margin-bottom: 0.5rem;
  text-shadow: 0 5px 15px rgba(24, 207, 85, 0.3);
}

.card-label {
  font-size: 1.1rem;
  color: #e0e0e0;
  opacity: 0.9;
  font-weight: 500;
}

.card-subtitle {
  font-size: 0.9rem;
  color: #b0b0b0;
  margin-top: 0.5rem;
}

/* Activity Chart */
.activity-chart {
  max-width: 600px;
  margin: 0 auto;
}

.chart-title {
  font-size: 1.5rem;
  margin-bottom: 2rem;
  color: #e0e0e0;
}

.chart-bar {
  display: flex;
  align-items: center;
  margin-bottom: 1rem;
  gap: 1rem;
}

.bar-label {
  width: 140px;
  text-align: left;
  font-size: 0.9rem;
  color: #e0e0e0;
}

.bar-container {
  flex: 1;
  height: 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  background: linear-gradient(45deg, #18cf55, #149e42);
  border-radius: 10px;
  transition: width 0.8s ease;
}

.bar-value {
  width: 40px;
  text-align: right;
  font-weight: 600;
  color: #18cf55;
}

/* Top Athletes Section */
.top-athletes-section {
  margin-top: 4rem;
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
}

.athletes-container {
  width: 100%;
}

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 3rem;
  color: #e0e0e0;
}

.loading-spinner-large {
  font-size: 3rem;
  color: #18cf55;
  animation: spin 1s linear infinite;
  margin-bottom: 1rem;
}

.athletes-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1.5rem;
  margin-top: 2rem;
  max-width: 800px;
  margin-left: auto;
  margin-right: auto;
}

.athlete-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  padding: 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  position: relative;
  overflow: hidden;
}

.athlete-card:hover {
  transform: translateY(-5px) scale(1.02);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
}

.athlete-card.top-three {
  border: 2px solid #ffd700;
  background: linear-gradient(135deg, rgba(255, 215, 0, 0.1), rgba(255, 255, 255, 0.1));
}

.athlete-card.top-three:nth-child(1) {
  border-color: #ffd700;
  box-shadow: 0 0 30px rgba(255, 215, 0, 0.3);
}

.athlete-card.top-three:nth-child(2) {
  border-color: #c0c0c0;
  box-shadow: 0 0 30px rgba(192, 192, 192, 0.3);
}

.athlete-card.top-three:nth-child(3) {
  border-color: #cd7f32;
  box-shadow: 0 0 30px rgba(205, 127, 50, 0.3);
}

.athlete-rank {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  flex-shrink: 0;
}

.crown {
  font-size: 1.8rem;
}

.crown.gold { filter: drop-shadow(0 0 10px #ffd700); }
.crown.silver { filter: drop-shadow(0 0 10px #c0c0c0); }
.crown.bronze { filter: drop-shadow(0 0 10px #cd7f32); }

.rank-number {
  font-size: 1.2rem;
  font-weight: 700;
  color: #18cf55;
}

.athlete-info {
  flex: 1;
}

.athlete-name {
  font-size: 1.1rem;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 0.5rem;
  text-align: left;
}

.athlete-stats {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.9rem;
  color: #e0e0e0;
}

.stat-item i {
  color: #18cf55;
  width: 16px;
}

.carbon-saved {
  color: #18cf55;
  font-weight: 600;
}

.distance {
  color: #60a5fa;
  font-weight: 600;
}

.athlete-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  color: #ffd700;
  font-size: 1.2rem;
  animation: sparkle 2s ease-in-out infinite;
}

@keyframes sparkle {
  0%, 100% { transform: scale(1) rotate(0deg); opacity: 0.8; }
  50% { transform: scale(1.2) rotate(180deg); opacity: 1; }
}

.no-data {
  text-align: center;
  padding: 3rem;
  color: #b0b0b0;
}

.no-data i {
  font-size: 3rem;
  margin-bottom: 1rem;
  opacity: 0.5;
}

/* CTA Button */
.cta {
  background: linear-gradient(45deg, #149e42, #18cf55);
  color: #ffffff;
  font-weight: 700;
  padding: 1.2rem 3rem;
  border-radius: 50px;
  border: none;
  font-size: 1.1rem;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  box-shadow: 0 15px 35px rgba(24, 207, 85, 0.3);
  margin-top: 2rem;
}

.cta:hover {
  transform: translateY(-5px);
  box-shadow: 0 20px 50px rgba(24, 207, 85, 0.4);
}

.cta-ripple {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
  transform: translate(-50%, -50%);
}

@keyframes ripple {
  to {
    width: 300px;
    height: 300px;
    opacity: 0;
  }
}

/* Scroll Indicator */
.scroll-indicator {
  position: fixed;
  right: 2rem;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.scroll-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
  transition: background 0.3s ease;
}

.scroll-dot:first-child {
  background: #18cf55;
}

/* Semester Selector */
.semester-selector {
  margin-bottom: 2rem;
  animation: fadeIn 0.3s ease-out;
}

.semester-dropdown {
  padding: 0.8rem 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  color: #e0e0e0;
  font-size: 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  min-width: 250px;
}

.semester-dropdown:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-2px);
}

.semester-dropdown:focus {
  outline: none;
  border-color: #18cf55;
  box-shadow: 0 0 0 3px rgba(24, 207, 85, 0.2);
}

.semester-dropdown option {
  background: #133a5a;
  color: #e0e0e0;
  padding: 0.5rem;
}

/* Month Selector */
.month-selector {
  margin-bottom: 2rem;
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

.month-dropdown {
  padding: 0.8rem 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  color: #e0e0e0;
  font-size: 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  min-width: 200px;
}

.month-dropdown:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-2px);
}

.month-dropdown:focus {
  outline: none;
  border-color: #18cf55;
  box-shadow: 0 0 0 3px rgba(24, 207, 85, 0.2);
}

.month-dropdown option {
  background: #133a5a;
  color: #e0e0e0;
  padding: 0.5rem;
}

.loading-spinner {
  display: inline-block;
  animation: spin 1s linear infinite;
  font-size: 1.5rem;
  color: #18cf55;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@media (max-width: 768px) {
  .main-title { font-size: 2.5rem; }
  .subtitle { font-size: 1.3rem; }
  .summary-grid { grid-template-columns: 1fr; gap: 1.5rem; }
  .athletes-grid { 
    grid-template-columns: 1fr; 
    gap: 1rem; 
    max-width: 500px;
    margin: 2rem auto 0;
  }
  .period-selector { flex-direction: column; align-items: center; }
  .bar-label { width: 120px; font-size: 0.9rem; }
  .scroll-indicator { display: none; }
  .athlete-stats { flex-direction: column; gap: 0.5rem; }
}

@media (max-width: 480px) {
  .landing { padding: 1rem; }
  .main-title { font-size: 2rem; }
  .summary-card { padding: 1.5rem; }
  .athletes-grid {
    grid-template-columns: 1fr;
    max-width: 100%;
  }
  .athlete-card { 
    padding: 1rem; 
    flex-direction: column; 
    text-align: center;
    min-width: 280px;
  }
  .athlete-rank { margin-bottom: 1rem; }
  .chart-bar { flex-direction: column; gap: 0.5rem; text-align: center; }
  .bar-label { width: auto; }
}
</style>