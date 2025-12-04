<template>
  <div class="section">
    <div class="card">
      <!-- HEADER WITH ACTION -->
      <div class="card-header">
        <div class="header-text">
          <h2 class="card-title">Station Discovery & Metadata</h2>
          <p class="header-subtitle">Search and explore {{ pagination.total_stations?.toLocaleString() || 'thousands of' }} weather stations across Europe</p>
        </div>

      </div>

      <!-- SEARCH FILTERS - PROMINENT -->
      <div class="search-section">
        <h3 class="search-title">Filter Stations</h3>
        <div class="search-grid">
          <div class="search-input-wrapper">
            <label class="input-label">
              <span class="label-icon"></span>
              Station Name
            </label>
            <input 
              v-model="filters.name" 
              class="input input-large" 
              placeholder="Type station name (e.g., BERLIN)"
              @keyup.enter="searchStations"
            />
          </div>
          
          <div class="search-input-wrapper">
            <label class="input-label">
              <span class="label-icon"></span>
              Country Code
            </label>
            <input 
              v-model="filters.country" 
              class="input input-large" 
              placeholder="2-letter code (e.g., DE)"
              @keyup.enter="searchStations"
            />
          </div>
          
          <div class="search-input-wrapper">
            <label class="input-label">
              <span class="label-icon"></span>
              Variable Type
            </label>
            <select v-model="filters.has_variable" class="input input-large">
              <option value="">All Variables</option>
              <option value="TX">TX (Max Temp)</option>
              <option value="TN">TN (Min Temp)</option>
              <option value="RR">RR (Precipitation)</option>
              <option value="PP">PP (Pressure)</option>
              <option value="FG">FG (Wind)</option>
            </select>
          </div>
          
          <div class="search-button-wrapper">
            <button 
              class="btn btn-primary btn-search" 
              @click="searchStations" 
              :disabled="searchLoading"
            >
              <span v-if="!searchLoading">Search</span>
              <span v-else>Searching...</span>
            </button>
            <button 
              v-if="filters.name || filters.country || filters.has_variable"
              class="btn btn-secondary btn-clear" 
              @click="clearFilters"
            >
              ✖ Clear
            </button>
          </div>
        </div>
      </div>

      <!-- SUMMARY STATS -->
      <div v-if="summary" class="stats-section">
        <div class="stat-card stat-primary">
          <div class="stat-icon"></div>
          <div class="stat-content">
            <div class="stat-value">{{ summary.total_stations.toLocaleString() }}</div>
            <div class="stat-label">Total Stations</div>
          </div>
        </div>
        <div class="stat-card stat-secondary">
          <div class="stat-icon"></div>
          <div class="stat-content">
            <div class="stat-value">{{ summary.countries_covered }}</div>
            <div class="stat-label">Countries</div>
          </div>
        </div>
        <div class="stat-card stat-tertiary">
          <div class="stat-icon"></div>
          <div class="stat-content">
            <div class="stat-value">{{ summary.stations_with_metadata }}</div>
            <div class="stat-label">With Metadata</div>
          </div>
        </div>
        <div class="stat-card stat-accent">
          <div class="stat-icon"></div>
          <div class="stat-content">
            <div class="stat-value">{{ summary.variables_coverage?.TX || 0 }}</div>
            <div class="stat-label">TX Stations</div>
          </div>
        </div>
      </div>

      <!-- STATION CARDS -->
      <div v-if="stations.length" class="stations-section">
        <div class="results-header">
          <h3 class="results-title">
            Found {{ pagination.total_stations }} station{{ pagination.total_stations !== 1 ? 's' : '' }}
          </h3>
          <div class="pagination-info">
            Page {{ pagination.page }} of {{ pagination.total_pages }}
          </div>
        </div>

        <!-- CARD GRID -->
        <div class="station-grid">
          <div 
            v-for="station in stations" 
            :key="station.STAID"
            class="station-card"
          >
            <!-- STATION HEADER -->
            <div class="station-header">
              <div class="station-id-badge">
                <span class="badge-label">STAID</span>
                <span class="badge-value">{{ station.STAID }}</span>
              </div>
              <div class="station-country">
                <span class="country-flag"></span>
                <span class="country-code">{{ station.country }}</span>
              </div>
            </div>

            <!-- STATION NAME -->
            <h4 class="station-name">{{ station.name }}</h4>

            <!-- STATION INFO -->
            <div class="station-info">
              <div class="info-row">
                <span class="info-icon"></span>
                <span class="info-text">{{ formatCoordinates(station) }}</span>
              </div>
              <div class="info-row">
                <span class="info-icon"></span>
                <span class="info-text">
                  {{ station.elevation_m ? station.elevation_m + ' m' : 'Elevation N/A' }}
                </span>
              </div>
            </div>


            <!-- ACTIONS -->
            <div class="station-actions">
              <button 
                class="btn btn-outline btn-action" 
                @click="viewDetails(station.STAID)"
                title="View detailed information"
              >
                Details
              </button>
              <button 
                class="btn btn-primary btn-action" 
                @click="showInMap(station)"
                title="Show station location on map"
              >
                Show Map
              </button>
            </div>
          </div>
        </div>

        <!-- PAGINATION -->
        <div class="pagination-controls">
          <button 
            class="btn btn-pagination" 
            @click="changePage(pagination.page - 1)" 
            :disabled="pagination.page === 1"
          >
            ← Previous
          </button>
          
          <div class="pagination-pages">
            <button
              v-for="page in visiblePages"
              :key="page"
              class="btn btn-page"
              :class="{ active: page === pagination.page }"
              @click="changePage(page)"
            >
              {{ page }}
            </button>
          </div>

          <button 
            class="btn btn-pagination" 
            @click="changePage(pagination.page + 1)" 
            :disabled="pagination.page >= pagination.total_pages"
          >
            Next →
          </button>
        </div>
      </div>

      <!-- EMPTY STATE -->
      <div v-else-if="!loading && !searchLoading" class="empty-state">
        <div class="empty-icon"></div>
        <h3 class="empty-title">No stations found</h3>
        <p class="empty-text">Try adjusting your search filters or click "Scan All Stations" to load data.</p>
        <button class="btn btn-primary" @click="discover">
          Load Stations
        </button>
      </div>

      <!-- LOADING STATE -->
      <div v-if="loading || searchLoading" class="loading">
        <div class="spinner"></div>
        <span>{{ loading ? 'Discovering stations...' : 'Searching...' }}</span>
      </div>

      <!-- ERROR -->
      <div v-if="error" class="error">
        <strong>Error:</strong> {{ error }}
      </div>
    </div>

    <!-- STATION DETAIL MODAL -->
    <div v-if="selectedStation" class="modal-overlay" @click="selectedStation = null">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <div>
            <h3>Station {{ selectedStation.STAID }} - {{ selectedStation.metadata?.name }}</h3>
            <div class="modal-meta">
              <span>{{ selectedStation.metadata?.country }}</span>
              <span>•</span>
              <span>{{ selectedStation.metadata?.elevation_m }}m elevation</span>
            </div>
          </div>
          <button class="btn-close" @click="selectedStation = null">×</button>
        </div>
        <div class="modal-body">
          <div class="detail-grid">
            <div class="detail-item">
              <span class="detail-label">Coordinates (DMS)</span>
              <span class="detail-value">
                {{ selectedStation.metadata?.latitude_dms }}, {{ selectedStation.metadata?.longitude_dms }}
              </span>
            </div>
            <div class="detail-item">
              <span class="detail-label">Coordinates (Decimal)</span>
              <span class="detail-value">
                {{ selectedStation.metadata?.latitude_dd?.toFixed(4) }}°, 
                {{ selectedStation.metadata?.longitude_dd?.toFixed(4) }}°
              </span>
            </div>
          </div>

          <h4 class="section-subtitle">Data Availability</h4>
          <div class="table-container">
            <table>
              <thead>
                <tr>
                  <th>Variable</th>
                  <th>Status</th>
                  <th>Records</th>
                  <th>Completeness</th>
                  <th>Date Range</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(data, var_name) in selectedStation.variables" :key="var_name">
                  <td><code>{{ var_name }}</code></td>
                  <td>
                    <span :class="data.file_exists ? 'status-badge status-yes' : 'status-badge status-no'">
                      {{ data.file_exists ? '✓ Available' : '✗ Missing' }}
                    </span>
                  </td>
                  <td>{{ data.total_records?.toLocaleString() || '—' }}</td>
                  <td>
                    <span v-if="data.completeness_pct" class="completeness-badge">
                      {{ data.completeness_pct }}%
                    </span>
                    <span v-else>—</span>
                  </td>
                  <td>
                    <span v-if="data.date_start && data.date_end" class="date-range">
                      {{ data.date_start }} → {{ data.date_end }}
                    </span>
                    <span v-else>—</span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <div class="modal-actions">
            <button class="btn btn-primary" @click="showInMapFromModal()">
              Show on Map
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- MAP POPUP MODAL -->
    <div v-if="mapStation" class="modal-overlay" @click="mapStation = null">
      <div class="modal-content modal-map" @click.stop>
        <div class="modal-header">
          <div>
            <h3>{{ mapStation.name }}</h3>
            <div class="station-meta">
              <span class="meta-chip">{{ mapStation.country }}</span>
              <span class="meta-chip">{{ mapStation.staid }}</span>
              <span v-if="mapStation.elevation" class="meta-chip">
                {{ mapStation.elevation }}m
              </span>
              <span class="meta-chip">
                {{ mapStation.lat.toFixed(4) }}°, {{ mapStation.lon.toFixed(4) }}°
              </span>
            </div>
          </div>
          <button class="btn-close" @click="mapStation = null">×</button>
        </div>
        <div class="modal-body modal-body-map">
          <div ref="mapPlot" class="map-plot"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, nextTick, watch, computed } from 'vue';
import axios from 'axios';
import Plotly from 'plotly.js-dist-min';

const API_BASE = 'http://localhost:5000/api';

export default {
  name: 'StationDiscovery',
  setup() {
    const loading = ref(false);
    const searchLoading = ref(false);
    const error = ref(null);
    const summary = ref(null);
    const stations = ref([]);
    const pagination = ref({});
    const selectedStation = ref(null);
    const mapStation = ref(null);
    const mapPlot = ref(null);

    const filters = ref({
      name: '',
      country: '',
      has_variable: ''
    });

    // Computed: visible page numbers for pagination
    const visiblePages = computed(() => {
      const current = pagination.value.page || 1;
      const total = pagination.value.total_pages || 1;
      const pages = [];
      
      // Show max 7 pages
      let start = Math.max(1, current - 3);
      let end = Math.min(total, current + 3);
      
      for (let i = start; i <= end; i++) {
        pages.push(i);
      }
      
      return pages;
    });

    const clearFilters = () => {
      filters.value = { name: '', country: '', has_variable: '' };
      searchStations();
    };

    const formatCoordinates = (station) => {
      if (!station.latitude_dms || !station.longitude_dms) return 'Coordinates unavailable';
      return `${station.latitude_dms}, ${station.longitude_dms}`;
    };

    const discover = async () => {
      try {
        loading.value = true;
        error.value = null;
        const res = await axios.get(`${API_BASE}/stations/discover`, {
          params: { per_page: 100, scan_dates: false }
        });
        summary.value = res.data.summary;
        stations.value = res.data.stations;
        pagination.value = res.data.pagination;
      } catch (err) {
        error.value = err.response?.data?.error || err.message;
      } finally {
        loading.value = false;
      }
    };

    const searchStations = async () => {
      try {
        searchLoading.value = true;
        error.value = null;
        const params = { per_page: 50 };
        if (filters.value.name) params.name = filters.value.name;
        if (filters.value.country) params.country = filters.value.country;
        if (filters.value.has_variable) params.has_variable = filters.value.has_variable;

        const res = await axios.get(`${API_BASE}/stations/search`, { params });
        stations.value = res.data.results;
        pagination.value = res.data.pagination;
      } catch (err) {
        error.value = err.response?.data?.error || err.message;
      } finally {
        searchLoading.value = false;
      }
    };

    const changePage = async (page) => {
      pagination.value.page = page;
      await searchStations();
      window.scrollTo({ top: 0, behavior: 'smooth' });
    };

    const viewDetails = async (staid) => {
      try {
        const res = await axios.get(`${API_BASE}/stations/${staid}`);
        selectedStation.value = res.data;
      } catch (err) {
        error.value = err.response?.data?.error || err.message;
      }
    };

    const showInMap = (station) => {
      const lat = parseDMS(station.latitude_dms);
      const lon = parseDMS(station.longitude_dms);
      
      if (lat && lon) {
        mapStation.value = {
          staid: station.STAID,
          name: station.name,
          country: station.country,
          lat,
          lon,
          elevation: station.elevation_m
        };
        nextTick(() => renderMap());
      } else {
        alert('Unable to parse coordinates for this station');
      }
    };

    const showInMapFromModal = () => {
      if (selectedStation.value?.metadata) {
        const metadata = selectedStation.value.metadata;
        mapStation.value = {
          staid: selectedStation.value.STAID,
          name: metadata.name,
          country: metadata.country,
          lat: metadata.latitude_dd,
          lon: metadata.longitude_dd,
          elevation: metadata.elevation_m
        };
        selectedStation.value = null;
        nextTick(() => renderMap());
      }
    };

    const renderMap = () => {
      if (!mapPlot.value || !mapStation.value) return;

      const station = mapStation.value;

      const trace = {
        type: 'scattermapbox',
        lat: [station.lat],
        lon: [station.lon],
        mode: 'markers',
        marker: {
          size: 20,
          color: '#E74C3C',
          symbol: 'marker'
        },
        text: [
          `<b>${station.name}</b><br>` +
          `Country: ${station.country}<br>` +
          `STAID: ${station.staid}<br>` +
          `Elevation: ${station.elevation || 'N/A'}m<br>` +
          `Coordinates: ${station.lat.toFixed(4)}°, ${station.lon.toFixed(4)}°`
        ],
        hoverinfo: 'text'
      };

      const layout = {
        mapbox: {
          style: 'open-street-map',
          center: {
            lat: station.lat,
            lon: station.lon
          },
          zoom: 10
        },
        margin: { t: 0, b: 0, l: 0, r: 0 },
        height: 600,
        paper_bgcolor: 'transparent',
        plot_bgcolor: 'transparent'
      };

      Plotly.newPlot(mapPlot.value, [trace], layout, {
        responsive: true,
        displaylogo: false,
        scrollZoom: true
      });
    };

    const parseDMS = (dms) => {
      if (!dms) return null;
      const parts = dms.split(':');
      if (parts.length !== 3) return null;
      
      const sign = parts[0].startsWith('+') ? 1 : -1;
      const degrees = Math.abs(parseFloat(parts[0]));
      const minutes = parseFloat(parts[1]);
      const seconds = parseFloat(parts[2]);
      
      return sign * (degrees + minutes / 60 + seconds / 3600);
    };

    watch(mapStation, (newVal) => {
      if (newVal) {
        nextTick(() => renderMap());
      }
    });

    return {
      loading,
      searchLoading,
      error,
      summary,
      stations,
      pagination,
      filters,
      selectedStation,
      mapStation,
      mapPlot,
      visiblePages,
      discover,
      searchStations,
      changePage,
      viewDetails,
      showInMap,
      showInMapFromModal,
      clearFilters,
      formatCoordinates
    };
  }
};
</script>

<style scoped>
/* ===== HEADER ===== */
.header-text {
  flex: 1;
}

.header-subtitle {
  color: var(--color-text-secondary);
  font-size: 0.95rem;
  margin-top: 0.5rem;
  margin-bottom: 0;
}

.btn-large {
  padding: 0.875rem 1.75rem;
  font-size: 1rem;
  white-space: nowrap;
}

/* ===== SEARCH SECTION ===== */
.search-section {
  background: linear-gradient(135deg, var(--color-bg-secondary) 0%, var(--color-bg-tertiary) 100%);
  padding: 2rem;
  border-radius: 12px;
  margin-bottom: 2.5rem;
  border: 1px solid var(--color-border);
}

.search-title {
  font-size: 1.125rem;
  margin-bottom: 1.25rem;
  color: var(--color-text);
  font-weight: 600;
}

.search-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.25rem;
  align-items: end;
}

.search-input-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.label-icon {
  margin-right: 0.375rem;
}

.input-large {
  padding: 0.75rem 1rem;
  font-size: 0.95rem;
}

.search-button-wrapper {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.btn-search {
  flex: 1;
  min-width: 120px;
}

.btn-secondary {
  background: var(--color-bg);
  color: var(--color-text);
}

.btn-secondary:hover {
  background: var(--color-bg-tertiary);
}

.btn-clear {
  padding: 0.75rem 1.25rem;
}

/* ===== STATS SECTION ===== */
.stats-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.25rem;
  margin-bottom: 2.5rem;
}

.stat-card {
  background: var(--color-bg-tertiary);
  padding: 1.5rem;
  border-radius: 10px;
  border: 1px solid var(--color-border);
  display: flex;
  align-items: center;
  gap: 1rem;
  transition: all 0.2s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-hover);
}

.stat-icon {
  font-size: 2.5rem;
  line-height: 1;
}

.stat-content {
  flex: 1;
}

.stat-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--color-text);
  line-height: 1.2;
}

.stat-label {
  font-size: 0.875rem;
  color: var(--color-text-secondary);
  margin-top: 0.25rem;
}

/* ===== RESULTS HEADER ===== */
.results-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 2px solid var(--color-border);
}

.results-title {
  font-size: 1.25rem;
  color: var(--color-text);
  margin: 0;
  font-weight: 600;
}

.pagination-info {
  background: var(--color-bg-tertiary);
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-size: 0.9rem;
  font-weight: 500;
  color: var(--color-text-secondary);
}

/* ===== STATION CARDS ===== */
.station-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.station-card {
  background: var(--color-bg-secondary);
  border: 1px solid var(--color-border);
  border-radius: 12px;
  padding: 1.5rem;
  transition: all 0.2s ease;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.station-card:hover {
  border-color: var(--color-primary);
  box-shadow: var(--shadow-hover);
  transform: translateY(-4px);
}

/* Station Header */
.station-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.station-id-badge {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: var(--color-bg-tertiary);
  padding: 0.375rem 0.75rem;
  border-radius: 6px;
  border: 1px solid var(--color-border);
}

.badge-label {
  font-size: 0.7rem;
  color: var(--color-text-tertiary);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.badge-value {
  font-family: 'Courier New', monospace;
  font-weight: 700;
  color: var(--color-primary);
  font-size: 0.9rem;
}

.station-country {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  font-weight: 600;
  color: var(--color-text);
}

.country-flag {
  font-size: 1.25rem;
}

.country-code {
  font-size: 0.9rem;
}

/* Station Name */
.station-name {
  font-size: 1.125rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0;
  line-height: 1.3;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

/* Station Info */
.station-info {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.info-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.875rem;
  color: var(--color-text-secondary);
}

.info-icon {
  font-size: 1rem;
  flex-shrink: 0;
}

.info-text {
  line-height: 1.4;
}

/* Variables */
.station-variables {
  padding-top: 0.75rem;
  border-top: 1px solid var(--color-border);
}

.variables-label {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--color-text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  display: block;
  margin-bottom: 0.5rem;
}

.variable-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 0.375rem;
}

.variable-chip {
  background: var(--color-primary);
  color: white;
  padding: 0.25rem 0.625rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.3px;
}

.no-variables {
  color: var(--color-text-tertiary);
  font-size: 0.875rem;
  font-style: italic;
}

/* Actions */
.station-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.75rem;
  margin-top: auto;
}

.btn-action {
  font-size: 0.875rem;
  padding: 0.625rem 1rem;
}

.btn-outline {
  background: transparent;
  border: 1px solid var(--color-border);
  color: var(--color-text);
}

.btn-outline:hover {
  background: var(--color-bg-tertiary);
  border-color: var(--color-primary);
  color: var(--color-primary);
}

/* ===== PAGINATION ===== */
.pagination-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  padding: 2rem 0;
  flex-wrap: wrap;
}

.btn-pagination {
  padding: 0.625rem 1.25rem;
  font-weight: 500;
}

.pagination-pages {
  display: flex;
  gap: 0.5rem;
}

.btn-page {
  min-width: 40px;
  padding: 0.625rem;
  font-weight: 500;
  background: var(--color-bg-tertiary);
}

.btn-page.active {
  background: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}

/* ===== EMPTY STATE ===== */
.empty-state {
  text-align: center;
  padding: 4rem 2rem;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  opacity: 0.5;
}

.empty-title {
  font-size: 1.5rem;
  color: var(--color-text);
  margin-bottom: 0.5rem;
}

.empty-text {
  color: var(--color-text-secondary);
  margin-bottom: 2rem;
}

/* ===== MODAL IMPROVEMENTS ===== */
.modal-meta {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  margin-top: 0.5rem;
  font-size: 0.9rem;
  color: var(--color-text-secondary);
}

.section-subtitle {
  font-size: 1.125rem;
  margin: 1.5rem 0 1rem 0;
  color: var(--color-text);
  font-weight: 600;
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  background: var(--color-bg-tertiary);
  padding: 1.25rem;
  border-radius: 8px;
  margin-bottom: 1.5rem;
}

.detail-item {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.detail-label {
  font-size: 0.8rem;
  color: var(--color-text-secondary);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.detail-value {
  font-size: 0.95rem;
  color: var(--color-text);
  font-weight: 500;
}

.status-badge {
  padding: 0.25rem 0.625rem;
  border-radius: 4px;
  font-size: 0.8rem;
  font-weight: 600;
}

.status-yes {
  background: var(--color-success-light);
  color: var(--color-success);
  border: 1px solid var(--color-success);
}

.status-no {
  background: var(--color-bg-tertiary);
  color: var(--color-text-tertiary);
  border: 1px solid var(--color-border);
}

.completeness-badge {
  background: var(--color-primary-light);
  color: var(--color-primary);
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.8rem;
  font-weight: 600;
}

.date-range {
  font-family: 'Courier New', monospace;
  font-size: 0.85rem;
}

.modal-actions {
  margin-top: 1.5rem;
  text-align: center;
}

.meta-chip {
  background: var(--color-bg-tertiary);
  padding: 0.375rem 0.75rem;
  border-radius: 6px;
  font-size: 0.85rem;
  border: 1px solid var(--color-border);
  white-space: nowrap;
}

/* ===== MODAL BASE ===== */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.75);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.2s ease;
  backdrop-filter: blur(4px);
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal-content {
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 12px;
  max-width: 900px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s ease;
  box-shadow: var(--shadow-modal);
}

@keyframes slideUp {
  from { 
    opacity: 0;
    transform: translateY(30px); 
  }
  to { 
    opacity: 1;
    transform: translateY(0); 
  }
}

.modal-map {
  max-width: 1200px;
  width: 95%;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 1.5rem;
  border-bottom: 1px solid var(--color-border);
  background: var(--color-bg-secondary);
  border-radius: 12px 12px 0 0;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.5rem;
  color: var(--color-text);
}

.station-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin-top: 0.75rem;
}

.btn-close {
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: var(--color-text-secondary);
  transition: all 0.2s;
  line-height: 1;
  padding: 0;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  border-radius: 6px;
}

.btn-close:hover {
  color: var(--color-error);
  background: var(--color-error-light);
}

.modal-body {
  padding: 1.5rem;
}

.modal-body-map {
  padding: 0;
}

.map-plot {
  width: 100%;
  height: 600px;
  border-radius: 0 0 12px 12px;
}

/* ===== RESPONSIVE ===== */
@media (max-width: 768px) {
  .search-grid {
    grid-template-columns: 1fr;
  }
  
  .search-button-wrapper {
    grid-column: 1;
  }
  
  .btn-search,
  .btn-clear {
    width: 100%;
  }
  
  .station-grid {
    grid-template-columns: 1fr;
  }
  
  .results-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.75rem;
  }
  
  .pagination-controls {
    flex-direction: column;
    gap: 1.5rem;
  }
  
  .station-meta {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .map-plot {
    height: 400px;
  }
}

@media (max-width: 480px) {
  .stat-card {
    flex-direction: column;
    text-align: center;
  }
  
  .station-actions {
    grid-template-columns: 1fr;
  }
}
</style>
