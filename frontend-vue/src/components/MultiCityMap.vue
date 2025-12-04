<template>
  <div class="map-container">
    <section class="map-section">
      <!-- HEADER -->
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Multi-City Geographic Analysis</h2>
          <p class="section-subtitle">
            Compare ERA5 bias patterns across multiple urban stations with interactive mapping
          </p>
        </div>
      </div>

      <!-- CITY & YEAR SELECTOR -->
      <div class="selector-panel">
        <div class="selector-grid">
          <!-- City Selection -->
          <div class="selector-group">
            <div class="selector-header">
              <h3 class="selector-title">Select Cities</h3>
              <span class="selector-hint">Hold Ctrl/Cmd for multiple</span>
            </div>
            
            <div class="city-list-wrapper">
              <div class="city-list">
                <label 
                  v-for="city in availableCities" 
                  :key="city"
                  class="city-checkbox"
                  :class="{ selected: selectedCities.includes(city) }"
                >
                  <input 
                    type="checkbox" 
                    :value="city"
                    v-model="selectedCities"
                    class="checkbox-input"
                  />
                  <span class="checkbox-custom"></span>
                  <span class="city-name">{{ city }}</span>
                  <span v-if="selectedCities.includes(city)" class="selected-indicator"></span>
                </label>
              </div>
            </div>

            <div class="selection-summary">
              <span class="summary-text">
                {{ selectedCities.length }} {{ selectedCities.length === 1 ? 'city' : 'cities' }} selected
              </span>
              <button 
                v-if="selectedCities.length > 0"
                class="btn-clear-selection"
                @click="selectedCities = []"
              >
                Clear All
              </button>
            </div>
          </div>

          <!-- Year & Action -->
          <div class="selector-group">
            <div class="year-selector">
              <h3 class="selector-title">Analysis Year</h3>
              <div class="year-input-group">
                <input 
                  v-model.number="year" 
                  type="number" 
                  class="year-input" 
                  min="2020" 
                  max="2024"
                  placeholder="YYYY"
                />
                <span class="year-range">2020 - 2024</span>
              </div>
            </div>

            <div class="action-group">
              <button 
                class="btn-generate"
                :class="{ disabled: loading || selectedCities.length === 0 }"
                @click="loadMap" 
                :disabled="loading || selectedCities.length === 0"
              >
                <span v-if="!loading" class="btn-icon">🗺</span>
                <span v-if="loading" class="btn-spinner"></span>
                {{ loading ? 'Generating Map...' : 'Generate Map' }}
              </button>

              <p class="action-hint" v-if="selectedCities.length === 0">
                Please select at least one city to continue
              </p>
            </div>
          </div>
        </div>
      </div>

      <!-- LOADING STATE -->
      <transition name="fade">
        <div v-if="loading" class="loading-panel">
          <div class="loading-content">
            <div class="spinner-large"></div>
            <h3 class="loading-title">Building Geographic Analysis</h3>
            <p class="loading-text">
              Processing {{ selectedCities.length }} {{ selectedCities.length === 1 ? 'city' : 'cities' }} for year {{ year }}
            </p>
          </div>
        </div>
      </transition>

      <!-- ERROR STATE -->
      <transition name="fade">
        <div v-if="error && !loading" class="error-panel">
          <div class="error-icon">⚠</div>
          <div class="error-content">
            <h3 class="error-title">Map Generation Failed</h3>
            <p class="error-message">{{ error }}</p>
            <button class="btn-retry" @click="loadMap">
              Retry
            </button>
          </div>
        </div>
      </transition>

      <!-- MAP RESULTS -->
      <transition name="slide-up">
        <div v-show="mapData && !loading" class="results-container">
          
          <!-- Summary Stats -->
          <div class="summary-stats">
            <div class="summary-card primary">
              <div class="summary-icon cities-icon"></div>
              <div class="summary-content">
                <div class="summary-value">{{ mapData?.cities_count || 0 }}</div>
                <div class="summary-label">Cities Analyzed</div>
              </div>
            </div>

            <div class="summary-card secondary">
              <div class="summary-icon stations-icon"></div>
              <div class="summary-content">
                <div class="summary-value">{{ mapData?.stations_count || 0 }}</div>
                <div class="summary-label">Weather Stations</div>
              </div>
            </div>

            <div class="summary-card tertiary">
              <div class="summary-icon year-icon"></div>
              <div class="summary-content">
                <div class="summary-value">{{ year }}</div>
                <div class="summary-label">Analysis Year</div>
              </div>
            </div>

            <div class="summary-card accent" v-if="mapData?.map_points">
              <div class="summary-icon bias-icon"></div>
              <div class="summary-content">
                <div class="summary-value">
                  {{ calculateAverageBias() }}°C
                </div>
                <div class="summary-label">Average Bias</div>
              </div>
            </div>
          </div>

          <!-- Interactive Map -->
          <div class="map-visualization">
            <div class="visualization-header">
              <h3 class="visualization-title">Geographic Distribution</h3>
              <p class="visualization-subtitle">
                Marker size and color indicate bias magnitude and direction
              </p>
            </div>
            <div class="map-wrapper">
              <div ref="mapPlot" class="map-canvas"></div>
            </div>
            <div class="map-legend">
              <div class="legend-item">
                <div class="legend-color cold"></div>
                <span class="legend-text">Cold Bias (ERA5 < Observed)</span>
              </div>
              <div class="legend-item">
                <div class="legend-color neutral"></div>
                <span class="legend-text">Neutral</span>
              </div>
              <div class="legend-item">
                <div class="legend-color hot"></div>
                <span class="legend-text">Hot Bias (ERA5 > Observed)</span>
              </div>
            </div>
          </div>

          <!-- Station Data Table -->
          <div class="data-section">
            <div class="data-header">
              <h3 class="data-title">Station Bias Summary</h3>
              <button class="btn-export-small" @click="exportTableData">
                Export CSV
              </button>
            </div>

            <div class="table-wrapper">
              <table class="data-table">
                <thead>
                  <tr>
                    <th class="sortable" @click="sortTable('city')">
                      City
                      <span class="sort-icon">↕</span>
                    </th>
                    <th class="sortable" @click="sortTable('station_name')">
                      Station
                      <span class="sort-icon">↕</span>
                    </th>
                    <th>Country</th>
                    <th class="numeric sortable" @click="sortTable('mean_bias')">
                      Mean Bias (°C)
                      <span class="sort-icon">↕</span>
                    </th>
                    <th class="numeric">Std Bias (°C)</th>
                    <th class="numeric">Max Bias (°C)</th>
                    <th class="numeric sortable" @click="sortTable('mean_temp')">
                      Mean Temp (°C)
                      <span class="sort-icon">↕</span>
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr 
                    v-for="point in sortedMapPoints" 
                    :key="point.station_id"
                    class="data-row"
                  >
                    <td class="city-cell">
                      <span class="city-badge">{{ point.city }}</span>
                    </td>
                    <td class="station-cell">{{ point.station_name }}</td>
                    <td class="country-cell">{{ point.country }}</td>
                    <td class="numeric bias-cell" :class="getBiasClass(point.mean_bias)">
                      <span class="bias-value">{{ point.mean_bias.toFixed(2) }}</span>
                      <div class="bias-bar" :style="getBiasBarStyle(point.mean_bias)"></div>
                    </td>
                    <td class="numeric">{{ point.std_bias.toFixed(2) }}</td>
                    <td class="numeric">{{ point.max_bias.toFixed(2) }}</td>
                    <td class="numeric temp-cell">{{ point.mean_temp.toFixed(1) }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

        </div>
      </transition>

      <!-- EMPTY STATE -->
      <div v-if="!mapData && !loading && !error" class="empty-state">
        <div class="empty-icon"></div>
        <h3 class="empty-title">No Map Data</h3>
        <p class="empty-text">
          Select one or more cities and click "Generate Map" to visualize geographic bias patterns
        </p>
      </div>

    </section>
  </div>
</template>

<script>
import { ref, nextTick, computed } from 'vue';
import axios from 'axios';
import Plotly from 'plotly.js-dist-min';

const API_BASE = 'http://localhost:5000/api';

export default {
  name: 'MultiCityMap',
  props: {
    availableCities: {
      type: Array,
      required: true
    }
  },
  setup() {
    const selectedCities = ref([]);
    const year = ref(2022);
    const loading = ref(false);
    const error = ref(null);
    const mapData = ref(null);
    const mapPlot = ref(null);
    const sortKey = ref('');
    const sortOrder = ref('asc');

    const sortedMapPoints = computed(() => {
      if (!mapData.value?.map_points) return [];
      
      const points = [...mapData.value.map_points];
      
      if (sortKey.value) {
        points.sort((a, b) => {
          const aVal = a[sortKey.value];
          const bVal = b[sortKey.value];
          
          if (typeof aVal === 'string') {
            return sortOrder.value === 'asc' 
              ? aVal.localeCompare(bVal)
              : bVal.localeCompare(aVal);
          }
          
          return sortOrder.value === 'asc' 
            ? aVal - bVal
            : bVal - aVal;
        });
      }
      
      return points;
    });

    const calculateAverageBias = () => {
      if (!mapData.value?.map_points) return '0.00';
      const avg = mapData.value.map_points.reduce((sum, p) => sum + p.mean_bias, 0) / mapData.value.map_points.length;
      return avg.toFixed(2);
    };

    const sortTable = (key) => {
      if (sortKey.value === key) {
        sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc';
      } else {
        sortKey.value = key;
        sortOrder.value = 'asc';
      }
    };

    const loadMap = async () => {
      try {
        loading.value = true;
        error.value = null;

        const res = await axios.post(`${API_BASE}/multi_city/map_data`, {
          cities: selectedCities.value,
          year: year.value
        });

        mapData.value = res.data;
        await nextTick();
        setTimeout(() => {
          renderMap();
        }, 100);
      } catch (err) {
        error.value = err.response?.data?.error || err.message;
      } finally {
        loading.value = false;
      }
    };

    const renderMap = () => {
      if (!mapPlot.value || !mapData.value) {
        console.error('Map plot ref or data not available');
        return;
      }

      const points = mapData.value.map_points;

      // Apply theme
      const style = getComputedStyle(document.documentElement);
      const textColor = style.getPropertyValue("--color-text").trim();
      const bgColor = style.getPropertyValue("--color-bg-tertiary").trim();

      const trace = {
        type: 'scattermapbox',
        lat: points.map(p => p.lat),
        lon: points.map(p => p.lon),
        mode: 'markers',
        marker: {
          size: points.map(p => Math.abs(p.mean_bias) * 4 + 8),
          color: points.map(p => p.mean_bias),
          colorscale: 'RdBu',
          reversescale: true,
          cmin: -3,
          cmax: 3,
          colorbar: {
            title: {
              text: 'Bias (°C)',
              font: { color: textColor }
            },
            thickness: 20,
            len: 0.7,
            tickfont: { color: textColor },
            outlinewidth: 0
          },
          line: {
            color: '#ffffff',
            width: 2
          }
        },
        text: points.map(p => 
          `<b>${p.station_name}</b><br>` +
          `City: ${p.city}<br>` +
          `Country: ${p.country}<br>` +
          `Mean Bias: ${p.mean_bias.toFixed(2)}°C<br>` +
          `Mean Temp: ${p.mean_temp.toFixed(1)}°C<br>` +
          `Max Bias: ${p.max_bias.toFixed(2)}°C`
        ),
        hoverinfo: 'text',
        hovertemplate: '%{text}<extra></extra>'
      };

      const centerLat = points.reduce((sum, p) => sum + p.lat, 0) / points.length;
      const centerLon = points.reduce((sum, p) => sum + p.lon, 0) / points.length;

      const layout = {
        mapbox: {
          style: 'open-street-map',
          center: { lat: centerLat, lon: centerLon },
          zoom: points.length === 1 ? 8 : 4
        },
        margin: { t: 0, b: 0, l: 0, r: 0 },
        height: 600,
        paper_bgcolor: bgColor,
        font: { color: textColor }
      };

      Plotly.newPlot(mapPlot.value, [trace], layout, {
        responsive: true,
        displaylogo: false,
        scrollZoom: true
      });
    };

    const getBiasClass = (bias) => {
      if (bias > 1) return 'bias-hot';
      if (bias < -1) return 'bias-cold';
      return 'bias-neutral';
    };

    const getBiasBarStyle = (bias) => {
      const absWidth = Math.min(Math.abs(bias) / 3 * 100, 100);
      const color = bias > 1 ? '#ef4444' : bias < -1 ? '#3b82f6' : '#94a3b8';
      return {
        width: absWidth + '%',
        background: color
      };
    };

    const exportTableData = () => {
      if (!mapData.value?.map_points) return;
      
      const headers = ['City', 'Station', 'Country', 'Mean Bias (°C)', 'Std Bias (°C)', 'Max Bias (°C)', 'Mean Temp (°C)'];
      const rows = mapData.value.map_points.map(p => [
        p.city,
        p.station_name,
        p.country,
        p.mean_bias.toFixed(2),
        p.std_bias.toFixed(2),
        p.max_bias.toFixed(2),
        p.mean_temp.toFixed(1)
      ]);
      
      const csv = [headers, ...rows].map(row => row.join(',')).join('\n');
      const blob = new Blob([csv], { type: 'text/csv' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `multi_city_bias_${year.value}.csv`;
      link.click();
      URL.revokeObjectURL(url);
    };

    return {
      selectedCities,
      year,
      loading,
      error,
      mapData,
      mapPlot,
      sortedMapPoints,
      loadMap,
      getBiasClass,
      getBiasBarStyle,
      calculateAverageBias,
      sortTable,
      exportTableData
    };
  }
};
</script>

<style scoped>
/* ===== CONTAINER ===== */
.map-container {
  max-width: 1400px;
  margin: 0 auto;
}

.map-section {
  background: var(--color-bg-secondary);
  border: 1px solid var(--color-border);
  border-radius: 12px;
  padding: 2rem;
  box-shadow: var(--shadow-card);
}

/* ===== HEADER ===== */
.section-header {
  margin-bottom: 2rem;
  padding-bottom: 1.5rem;
  border-bottom: 2px solid var(--color-border);
}

.section-title-group {
  max-width: 800px;
}

.section-title {
  font-size: 1.75rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 0.5rem 0;
  line-height: 1.3;
}

.section-subtitle {
  font-size: 1rem;
  color: var(--color-text-secondary);
  margin: 0;
  line-height: 1.6;
}

/* ===== SELECTOR PANEL ===== */
.selector-panel {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  padding: 2rem;
  margin-bottom: 2rem;
}

.selector-grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 2rem;
}

.selector-group {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.selector-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.selector-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0;
}

.selector-hint {
  font-size: 0.8rem;
  color: var(--color-text-tertiary);
  font-style: italic;
}

/* ===== CITY LIST ===== */
.city-list-wrapper {
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 0.5rem;
  max-height: 400px;
  overflow-y: auto;
}

.city-list {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.city-checkbox {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1rem;
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.city-checkbox:hover {
  background: var(--color-bg-secondary);
  border-color: var(--color-primary);
}

.city-checkbox.selected {
  background: var(--color-primary-light);
  border-color: var(--color-primary);
}

.checkbox-input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
}

.checkbox-custom {
  width: 20px;
  height: 20px;
  border: 2px solid var(--color-border);
  border-radius: 4px;
  flex-shrink: 0;
  transition: all 0.2s ease;
  position: relative;
}

.city-checkbox.selected .checkbox-custom {
  background: var(--color-primary);
  border-color: var(--color-primary);
}

.city-checkbox.selected .checkbox-custom::after {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 0.875rem;
  font-weight: 700;
}

.city-name {
  flex: 1;
  font-size: 0.95rem;
  font-weight: 500;
  color: var(--color-text);
}

.selected-indicator {
  width: 8px;
  height: 8px;
  background: var(--color-primary);
  border-radius: 50%;
  animation: pulse 2s infinite;
}

.selection-summary {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  background: var(--color-bg);
  border-radius: 6px;
  border: 1px solid var(--color-border);
}

.summary-text {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--color-text);
}

.btn-clear-selection {
  padding: 0.375rem 0.75rem;
  background: transparent;
  border: 1px solid var(--color-border);
  border-radius: 4px;
  color: var(--color-text-secondary);
  font-size: 0.8rem;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-clear-selection:hover {
  background: var(--color-error-light);
  border-color: var(--color-error);
  color: var(--color-error);
}

/* ===== YEAR SELECTOR ===== */
.year-selector {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.year-input-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.year-input {
  width: 100%;
  padding: 0.875rem 1rem;
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  color: var(--color-text);
  font-size: 1.25rem;
  font-weight: 600;
  text-align: center;
  font-variant-numeric: tabular-nums;
}

.year-input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--color-primary-light);
}

.year-range {
  font-size: 0.8rem;
  color: var(--color-text-tertiary);
  text-align: center;
}

/* ===== ACTION GROUP ===== */
.action-group {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-top: auto;
}

.btn-generate {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  width: 100%;
  padding: 1rem 1.5rem;
  background: var(--color-primary);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-generate:hover:not(.disabled) {
  background: var(--color-primary-hover);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(59, 130, 246, 0.4);
}

.btn-generate.disabled {
  background: var(--color-bg-tertiary);
  color: var(--color-text-tertiary);
  cursor: not-allowed;
  opacity: 0.6;
}

.btn-icon {
  font-size: 1.25rem;
}

.btn-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid white;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.action-hint {
  font-size: 0.85rem;
  color: var(--color-text-tertiary);
  text-align: center;
  margin: 0;
  font-style: italic;
}

/* ===== LOADING & ERROR ===== */
.loading-panel {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  padding: 3rem 2rem;
  margin: 2rem 0;
  text-align: center;
}

.loading-content {
  max-width: 400px;
  margin: 0 auto;
}

.spinner-large {
  width: 64px;
  height: 64px;
  border: 4px solid var(--color-border);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 1.5rem;
}

.loading-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 0.5rem 0;
}

.loading-text {
  font-size: 0.95rem;
  color: var(--color-text-secondary);
  margin: 0;
}

.error-panel {
  display: flex;
  gap: 1.5rem;
  background: var(--color-error-light);
  border: 1px solid var(--color-error);
  border-radius: 10px;
  padding: 1.5rem;
  margin: 2rem 0;
}

.error-icon {
  font-size: 2.5rem;
  flex-shrink: 0;
}

.error-content {
  flex: 1;
}

.error-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 0.5rem 0;
}

.error-message {
  font-size: 0.95rem;
  color: var(--color-text-secondary);
  margin: 0 0 1rem 0;
}

.btn-retry {
  padding: 0.5rem 1rem;
  background: var(--color-error);
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 0.875rem;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-retry:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.3);
}

/* ===== RESULTS ===== */
.results-container {
  margin-top: 2rem;
}

/* ===== SUMMARY STATS ===== */
.summary-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.25rem;
  margin-bottom: 2rem;
}

.summary-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.5rem;
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  transition: all 0.2s ease;
}

.summary-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-hover);
}

.summary-icon {
  width: 48px;
  height: 48px;
  border-radius: 8px;
  flex-shrink: 0;
}

.cities-icon {
  background: linear-gradient(135deg, #3b82f6, #2563eb);
}

.stations-icon {
  background: linear-gradient(135deg, #8b5cf6, #7c3aed);
}

.year-icon {
  background: linear-gradient(135deg, #10b981, #059669);
}

.bias-icon {
  background: linear-gradient(135deg, #f59e0b, #d97706);
}

.summary-content {
  flex: 1;
}

.summary-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--color-text);
  line-height: 1;
  font-variant-numeric: tabular-nums;
}

.summary-label {
  font-size: 0.85rem;
  color: var(--color-text-secondary);
  margin-top: 0.375rem;
}

/* ===== MAP VISUALIZATION ===== */
.map-visualization {
  margin-bottom: 2rem;
}

.visualization-header {
  margin-bottom: 1rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--color-border);
}

.visualization-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 0.375rem 0;
}

.visualization-subtitle {
  font-size: 0.875rem;
  color: var(--color-text-secondary);
  margin: 0;
}

.map-wrapper {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.map-canvas {
  width: 100%;
  height: 600px;
  border-radius: 6px;
}

.map-legend {
  display: flex;
  justify-content: center;
  gap: 2rem;
  padding: 1rem;
  background: var(--color-bg-tertiary);
  border-radius: 6px;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.legend-color {
  width: 24px;
  height: 24px;
  border-radius: 4px;
  border: 2px solid var(--color-border);
}

.legend-color.cold {
  background: #3b82f6;
}

.legend-color.neutral {
  background: #94a3b8;
}

.legend-color.hot {
  background: #ef4444;
}

.legend-text {
  font-size: 0.875rem;
  color: var(--color-text-secondary);
  font-weight: 500;
}

/* ===== DATA TABLE ===== */
.data-section {
  margin-top: 2rem;
}

.data-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--color-border);
}

.data-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0;
}

.btn-export-small {
  padding: 0.5rem 1rem;
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  color: var(--color-text);
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-export-small:hover {
  background: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}

.table-wrapper {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  overflow-x: auto;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.data-table thead {
  background: var(--color-bg);
  border-bottom: 2px solid var(--color-border);
}

.data-table th {
  padding: 1rem;
  text-align: left;
  font-weight: 600;
  font-size: 0.85rem;
  color: var(--color-text);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  white-space: nowrap;
}

.data-table th.numeric {
  text-align: right;
}

.data-table th.sortable {
  cursor: pointer;
  user-select: none;
  transition: color 0.2s;
}

.data-table th.sortable:hover {
  color: var(--color-primary);
}

.sort-icon {
  margin-left: 0.25rem;
  opacity: 0.5;
  font-size: 0.75rem;
}

.data-table td {
  padding: 0.875rem 1rem;
  border-top: 1px solid var(--color-border);
  color: var(--color-text);
}

.data-table td.numeric {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

.data-row {
  transition: background-color 0.15s ease;
}

.data-row:hover {
  background-color: var(--color-bg);
}

.city-badge {
  display: inline-block;
  padding: 0.25rem 0.625rem;
  background: var(--color-primary-light);
  color: var(--color-primary);
  border-radius: 4px;
  font-size: 0.85rem;
  font-weight: 600;
}

.bias-cell {
  position: relative;
  font-weight: 600;
}

.bias-value {
  position: relative;
  z-index: 1;
}

.bias-bar {
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  height: 70%;
  border-radius: 0 4px 4px 0;
  opacity: 0.2;
  transition: width 0.3s ease;
}

.bias-hot {
  color: #ef4444;
}

.bias-cold {
  color: #3b82f6;
}

.bias-neutral {
  color: var(--color-text-secondary);
}

/* ===== EMPTY STATE ===== */
.empty-state {
  text-align: center;
  padding: 4rem 2rem;
}

.empty-icon {
  width: 80px;
  height: 80px;
  margin: 0 auto 1.5rem;
  background: linear-gradient(135deg, var(--color-bg-tertiary), var(--color-border));
  border-radius: 50%;
}

.empty-title {
  font-size: 1.375rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 0.75rem 0;
}

.empty-text {
  font-size: 1rem;
  color: var(--color-text-secondary);
  max-width: 500px;
  margin: 0 auto;
  line-height: 1.6;
}

/* ===== TRANSITIONS ===== */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: all 0.4s ease;
}

.slide-up-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

/* ===== RESPONSIVE ===== */
@media (max-width: 1024px) {
  .selector-grid {
    grid-template-columns: 1fr;
  }
  
  .summary-stats {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .map-section {
    padding: 1.5rem;
  }
  
  .selector-panel {
    padding: 1.5rem;
  }
  
  .section-title {
    font-size: 1.5rem;
  }
  
  .summary-stats {
    grid-template-columns: 1fr;
  }
  
  .map-canvas {
    height: 400px;
  }
  
  .map-legend {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.75rem;
  }
  
  .data-table {
    font-size: 0.8rem;
  }
  
  .data-table th,
  .data-table td {
    padding: 0.625rem 0.75rem;
  }
}

@media (max-width: 480px) {
  .map-section {
    padding: 1rem;
  }
  
  .selector-panel {
    padding: 1rem;
  }
  
  .map-canvas {
    height: 350px;
  }
  
  .city-checkbox {
    padding: 0.625rem 0.875rem;
  }
  
  .summary-card {
    padding: 1rem;
  }
  
  .summary-icon {
    width: 40px;
    height: 40px;
  }
  
  .summary-value {
    font-size: 1.5rem;
  }
}
</style>
