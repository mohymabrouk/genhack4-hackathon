<template>
  <div class="knn-container">
    <section class="knn-section">
      <!-- HEADER -->
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">K-Nearest Neighbors Climate Similarity</h2>
          <p class="section-subtitle">
            Discover cities with similar climate patterns using advanced K-NN clustering on standardized temperature and bias features
          </p>
        </div>
      </div>

      <!-- CONFIGURATION PANEL -->
      <div class="config-panel">
        <h3 class="config-title">Analysis Configuration</h3>
        
        <div class="config-grid">
          <!-- City Selection -->
          <div class="config-group city-selection-group">
            <div class="config-header">
              <label class="config-label">Select Cities for Analysis</label>
              <span class="selection-count">
                {{ selectedCities.length }} selected
              </span>
            </div>
            
            <div class="city-checkbox-list">
              <label 
                v-for="city in availableCities" 
                :key="city"
                class="city-checkbox-item"
                :class="{ selected: selectedCities.includes(city) }"
              >
                <input 
                  type="checkbox" 
                  :value="city"
                  v-model="selectedCities"
                  class="checkbox-input"
                />
                <span class="checkbox-custom"></span>
                <span class="city-label">{{ city }}</span>
              </label>
            </div>

            <div class="selection-actions">
              <button 
                v-if="selectedCities.length > 0"
                class="btn-clear"
                @click="selectedCities = []"
              >
                Clear Selection
              </button>
              <button 
                class="btn-select-all"
                @click="selectAll"
              >
                Select All
              </button>
            </div>
          </div>

          <!-- Parameters -->
          <div class="config-group parameters-group">
            <div class="parameter-item">
              <label class="config-label">Analysis Year</label>
              <div class="year-selector">
                <input 
                  v-model.number="year" 
                  type="number" 
                  class="year-input" 
                  min="2020" 
                  max="2024"
                />
                <span class="year-hint">Range: 2020-2024</span>
              </div>
            </div>

            <div class="parameter-item">
  <label class="config-label">Number of Clusters</label>
  <div class="k-selector">
    <input 
      v-model.number="n_clusters" 
      type="range" 
      class="k-slider" 
      min="2" 
      :max="Math.min(10, selectedCities.length)" 
    />
    <div class="k-value-display">{{ n_clusters }}</div>
  </div>
  <div class="k-labels">
    <span>2</span>
    <span>6</span>
    <span>10</span>
  </div>
</div>


            <div class="action-container">
              <button 
                class="btn-run-analysis"
                :class="{ disabled: loading || selectedCities.length < 2 }"
                @click="runKNN" 
                :disabled="loading || selectedCities.length < 2"
              >
                <span v-if="!loading" class="btn-icon">🔬</span>
                <span v-if="loading" class="btn-spinner"></span>
                {{ loading ? 'Computing Similarities...' : 'Run KNN Analysis' }}
              </button>

              <p v-if="selectedCities.length < 2" class="requirement-hint">
                Minimum 2 cities required for analysis
              </p>
              <p v-else class="ready-hint">
                Ready to analyze {{ selectedCities.length }} cities
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
            <h3 class="loading-title">Computing Climate Similarities</h3>
            <p class="loading-text">
              Analyzing {{ selectedCities.length }} cities using K-NN with k={{ k }}
            </p>
          </div>
        </div>
      </transition>

      <!-- ERROR STATE -->
      <transition name="fade">
        <div v-if="error && !loading" class="error-panel">
          <div class="error-icon">⚠</div>
          <div class="error-content">
            <h3 class="error-title">Analysis Failed</h3>
            <p class="error-message">{{ error }}</p>
            <button class="btn-retry" @click="runKNN">
              Retry Analysis
            </button>
          </div>
        </div>
      </transition>

      <!-- RESULTS -->
      <transition name="slide-up">
        <div v-show="results && !loading" class="results-wrapper">
          
          <!-- Analysis Summary -->
          <div class="analysis-summary">
            <div class="summary-card">
              <div class="summary-header">
                <h3 class="summary-title">Analysis Overview</h3>
              </div>
              <div class="summary-grid">
                <div class="summary-item">
                  <span class="summary-label">Method</span>
                  <span class="summary-value">{{ results?.method || 'K-NN' }}</span>

                </div>
                <div class="summary-item">
                  <span class="summary-label">Features Used</span>
                  <span class="summary-value">{{ results?.features_used?.length || 0 }}</span>

                </div>
                <div class="summary-item">
                  <span class="summary-label">Clusters Detected</span>
                  <span class="summary-value">{{ Object.keys(clusterGroups).length }}</span>
                </div>
                <div class="summary-item">
                  <span class="summary-label">Cities Analyzed</span>
                  <span class="summary-value">{{ selectedCities.length }}</span>
                </div>
              </div>
              
              <div class="features-list">
                <h4 class="features-title">Climate Features</h4>
                <div class="feature-tags">
                  <span 
                  v-for="feature in results?.features_used || []" 
                    :key="feature"
                    class="feature-tag"
                  >
                    {{ formatFeatureName(feature) }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <!-- Cluster Legend -->
          <div class="clusters-section">
            <h3 class="section-subtitle">Climate Cluster Classification</h3>
            <div class="cluster-legend-grid">
              <div 
                v-for="(cities, clusterName) in clusterGroups" 
                :key="clusterName"
                class="cluster-badge-card"
                :style="{ borderColor: getClusterColor(clusterName) }"
              >
                <div 
                  class="cluster-color-indicator" 
                  :style="{ background: getClusterColor(clusterName) }"
                ></div>
                <div class="cluster-info">
                  <h4 class="cluster-name">{{ clusterName }}</h4>
                  <p class="cluster-count">{{ cities.length }} {{ cities.length === 1 ? 'city' : 'cities' }}</p>
                  <div class="cluster-cities">
                    <span 
                      v-for="city in cities.slice(0, 3)" 
                      :key="city"
                      class="cluster-city-tag"
                    >
                      {{ city }}
                    </span>
                    <span v-if="cities.length > 3" class="more-cities">
                      +{{ cities.length - 3 }} more
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Geographic Map -->
          <div class="map-section">
            <div class="map-header">
              <h3 class="section-subtitle">Geographic Distribution</h3>
              <p class="map-description">
                Cities are color-coded by climate cluster. Hover for detailed statistics.
              </p>
            </div>
            <div class="map-wrapper">
              <div ref="mapPlot" class="map-canvas"></div>
            </div>
          </div>

          <!-- City Analysis Cards -->
          <div class="city-analysis-section">
            <h3 class="section-subtitle">Detailed City Analysis</h3>
            
            <div class="knn-cards-grid">
              <div 
                v-for="result in results?.knn_results || []" 
                :key="result.city" 
                class="knn-analysis-card"
              >
                <!-- Card Header -->
                <div 
                  class="card-header-bar"
                  :style="{ background: `linear-gradient(135deg, ${getClusterColor(result.cluster_label)}22, ${getClusterColor(result.cluster_label)}44)` }"
                >
                  <div class="city-title-group">
                    <h3 class="city-name">{{ result.city }}</h3>
                    <span 
                      class="cluster-badge" 
                      :style="{ 
                        background: getClusterColor(result.cluster_label),
                        color: '#fff'
                      }"
                    >
                      {{ result.cluster_label }}
                    </span>
                  </div>
                  <div class="data-points-info">
                    {{ result.data_points || 'N/A' }} data points
                  </div>
                </div>

                <!-- Climate Features -->
                <div class="features-section">
                  <h4 class="subsection-title">Climate Metrics</h4>
                  <div class="metrics-grid">
                    <div 
                      v-for="(value, key) in result.features" 
                      :key="key" 
                      class="metric-card"
                    >
                      <div class="metric-icon" :class="`icon-${key}`"></div>
                      <div class="metric-content">
                        <span class="metric-label">{{ formatFeatureName(key) }}</span>
                        <span class="metric-value">{{ formatMetricValue(key, value) }}</span>
                      </div>
                    </div>
                  </div>
                </div>

                <!-- Nearest Neighbors -->
                <div class="neighbors-section">
                  <h4 class="subsection-title">Most Similar Cities</h4>
                  <div class="neighbors-list">
                    <div 
                      v-for="(neighbor, idx) in result.nearest_neighbors" 
                      :key="idx"
                      class="neighbor-card"
                    >
                      <div class="neighbor-rank-badge">{{ idx + 1 }}</div>
                      
                      <div class="neighbor-details">
                        <div class="neighbor-header">
                          <span class="neighbor-city">{{ neighbor.city }}</span>
                          <span 
                            class="neighbor-cluster"
                            :style="{ 
                              background: getClusterColor(neighbor.cluster_label) + '33',
                              color: getClusterColor(neighbor.cluster_label),
                              border: `1px solid ${getClusterColor(neighbor.cluster_label)}`
                            }"
                          >
                            {{ neighbor.cluster_label }}
                          </span>
                        </div>
                        
                        <div class="neighbor-metrics">
                          <div class="neighbor-metric">
                            <span class="metric-icon-small">📏</span>
                            <span class="metric-text">Distance: {{ neighbor.distance.toFixed(3) }}</span>
                          </div>
                          <div class="neighbor-metric highlight">
                            <span class="metric-icon-small">✓</span>
                            <span class="metric-text">Similarity: {{ (neighbor.similarity_score * 100).toFixed(1) }}%</span>
                          </div>
                        </div>
                      </div>

                      <div class="similarity-indicator">
                        <div 
                          class="similarity-bar"
                          :style="{ 
                            width: (neighbor.similarity_score * 100) + '%',
                            background: getClusterColor(neighbor.cluster_label)
                          }"
                        ></div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

        </div>
      </transition>

      <!-- EMPTY STATE -->
      <div v-if="!results && !loading && !error" class="empty-state">
        <div class="empty-icon"></div>
        <h3 class="empty-title">No Analysis Results</h3>
        <p class="empty-text">
          Configure your analysis parameters above and click "Run KNN Analysis" to discover climate similarities across cities
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

// Cluster colors
const CLUSTER_COLORS = {
  'Faithful': '#22c55e',
  'Heat Trap': '#ef4444',
  'Chaotic': '#fbbf24',
  'Moderate': '#3b82f6',
  'Cool': '#06b6d4',
  'Stable': '#8b5cf6'
};

export default {
  name: 'KNNAnalysis',
  props: {
    availableCities: {
      type: Array,
      required: true
    }
  },
  setup(props) {
  const selectedCities = ref([]);
  const year = ref(2022);
  const k = ref(3);
  const n_clusters = ref(3);  // ADD THIS LINE
  const loading = ref(false);
  const error = ref(null);
  const results = ref(null);
  const heatmapPlot = ref(null);
  const mapPlot = ref(null);


  const clusterGroups = computed(() => {
  if (!results.value?.knn_results) return {};
  const groups = {};
  
  results.value.knn_results.forEach(result => {
    const cluster = result.cluster_label || 'Unclustered';  // Backend provides this
    if (!groups[cluster]) {
      groups[cluster] = [];
    }
    groups[cluster].push(result.city);
  });
  
  return groups;
});



    const selectAll = () => {
      selectedCities.value = [...props.availableCities];
    };

    const getClusterColor = (clusterName) => {
  // If it's "Cluster 1", "Cluster 2", etc from backend
  if (clusterName.startsWith('Cluster ')) {
    const colors = ['#22c55e', '#3b82f6', '#ef4444', '#fbbf24', '#8b5cf6', '#06b6d4', '#ec4899', '#f97316'];
    const num = parseInt(clusterName.split(' ')[1]) - 1;
    return colors[num % colors.length];
  }
  
  // Fallback for named clusters
  return CLUSTER_COLORS[clusterName] || '#94a3b8';
};


    const formatFeatureName = (key) => {
      const names = {
        mean_bias: 'Mean Bias',
        std_bias: 'Volatility',
        mean_temp: 'Mean Temp',
        temp_range: 'Temp Range',
        p95_temp: '95th Percentile',
        max_bias: 'Max Bias',
        min_bias: 'Min Bias'
      };
      return names[key] || key.replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase());
    };

    const formatMetricValue = (key, value) => {
      if (value === null || value === undefined) return 'N/A';
      
      if (key.includes('temp') || key.includes('bias')) {
        return value.toFixed(2) + '°C';
      }
      
      return value.toFixed(2);
    };

    




    const runKNN = async () => {
  try {
    loading.value = true;
    error.value = null;
    
    const res = await axios.post(`${API_BASE}/multi_city/knn_analysis`, {
      cities: selectedCities.value,
      year: year.value,
      k: k.value,
      n_clusters: n_clusters.value,  // ADD THIS
      clustering_method: "kmeans"
    });
    
    results.value = res.data;
    
    // REMOVE OR COMMENT OUT assignClusters() since backend does it now
    // if (results.value.knn_results) {
    //   assignClusters();
    // }
    
    await nextTick();
    setTimeout(() => {
      renderMap();
      renderHeatmap();
    }, 100);
  } catch (err) {
    error.value = err.response?.data?.error || err.message;
  } finally {
    loading.value = false;
  }
};



    const assignClusters = () => {
      results.value.knn_results.forEach(result => {
        if (!result.cluster_label) {
          const bias = result.features.mean_bias || 0;
          const temp = result.features.mean_temp || 0;
          const volatility = result.features.std_bias || 0;
          
          if (Math.abs(bias) < 0.5 && volatility < 1) {
            result.cluster_label = 'Faithful';
          } else if (bias > 1.5 || temp > 25) {
            result.cluster_label = 'Heat Trap';
          } else if (volatility > 2) {
            result.cluster_label = 'Chaotic';
          } else if (temp < 15) {
            result.cluster_label = 'Cool';
          } else if (volatility < 1) {
            result.cluster_label = 'Stable';
          } else {
            result.cluster_label = 'Moderate';
          }
        }
      });
      
      results.value.knn_results.forEach(result => {
        result.nearest_neighbors.forEach(neighbor => {
          const neighborData = results.value.knn_results.find(r => r.city === neighbor.city);
          if (neighborData) {
            neighbor.cluster_label = neighborData.cluster_label;
          }
        });
      });
    };

    const renderMap = () => {
      if (!mapPlot.value || !results.value) {
        console.error('Map plot ref or results not available');
        return;
      }

      const style = getComputedStyle(document.documentElement);
      const textColor = style.getPropertyValue("--color-text").trim();
      const bgColor = style.getPropertyValue("--color-bg-tertiary").trim();

      const traces = [];
      const knnResults = results.value.knn_results;

      Object.keys(clusterGroups.value).forEach(clusterName => {
        const clusterCities = clusterGroups.value[clusterName];
        const clusterData = knnResults.filter(r => clusterCities.includes(r.city));
        
        const lats = [];
        const lons = [];
        const texts = [];
        const hovers = [];
        
        clusterData.forEach(result => {
          if (result.lat && result.lon) {
            lats.push(result.lat);
            lons.push(result.lon);
            texts.push(result.city);
            hovers.push(
              `<b>${result.city}</b><br>` +
              `Cluster: ${clusterName}<br>` +
              `Mean Bias: ${result.features.mean_bias?.toFixed(2) || 'N/A'}°C<br>` +
              `Mean Temp: ${result.features.mean_temp?.toFixed(1) || 'N/A'}°C<br>` +
              `Volatility: ${result.features.std_bias?.toFixed(2) || 'N/A'}°C`
            );
          }
        });
        
        if (lats.length > 0) {
          traces.push({
            type: 'scattermapbox',
            lat: lats,
            lon: lons,
            mode: 'markers+text',
            name: clusterName,
            text: texts,
            hovertext: hovers,
            hoverinfo: 'text',
            textposition: 'top center',
            textfont: {
              color: getClusterColor(clusterName),
              size: 10,
              family: 'Inter, sans-serif',
              weight: 'bold'
            },
            marker: {
              size: 28,
              color: getClusterColor(clusterName),
              opacity: 0.9,
              line: {
                color: '#ffffff',
                width: 3
              }
            }
          });
        }
      });

      const allLats = [];
      const allLons = [];
      
      knnResults.forEach(r => {
        if (r.lat && r.lon) {
          allLats.push(r.lat);
          allLons.push(r.lon);
        }
      });
      
      if (allLats.length === 0) return;
      
      const centerLat = allLats.reduce((a, b) => a + b, 0) / allLats.length;
      const centerLon = allLons.reduce((a, b) => a + b, 0) / allLons.length;

      const layout = {
        mapbox: {
          style: 'open-street-map',
          center: { lat: centerLat, lon: centerLon },
          zoom: 4
        },
        margin: { t: 0, b: 0, l: 0, r: 0 },
        height: 600,
        paper_bgcolor: bgColor,
        plot_bgcolor: bgColor,
        font: { color: textColor },
        showlegend: true,
        legend: {
          x: 0.02,
          y: 0.98,
          bgcolor: 'rgba(10, 10, 10, 0.92)',
          bordercolor: 'rgba(148, 163, 184, 0.3)',
          borderwidth: 1,
          font: { size: 11, color: textColor }
        }
      };

      Plotly.newPlot(mapPlot.value, traces, layout, {
        responsive: true,
        displaylogo: false,
        scrollZoom: true
      });
    };

    const renderHeatmap = () => {
      if (!heatmapPlot.value || !results.value) return;

      const style = getComputedStyle(document.documentElement);
      const textColor = style.getPropertyValue("--color-text").trim();
      const bgColor = style.getPropertyValue("--color-bg-tertiary").trim();

      const cities = results.value.knn_results.map(r => r.city);
      const n = cities.length;
      
      const distanceMatrix = Array(n).fill(0).map(() => Array(n).fill(0));
      
      results.value.knn_results.forEach((result, i) => {
        result.nearest_neighbors.forEach(neighbor => {
          const j = cities.indexOf(neighbor.city);
          if (j !== -1) {
            distanceMatrix[i][j] = neighbor.distance;
            distanceMatrix[j][i] = neighbor.distance;
          }
        });
      });

      const trace = {
        type: 'heatmap',
        x: cities,
        y: cities,
        z: distanceMatrix,
        colorscale: 'Viridis',
        reversescale: true,
        colorbar: { 
          title: { text: 'Distance', font: { color: textColor } },
          tickfont: { color: textColor }
        }
      };

      const layout = {
        xaxis: { 
          tickangle: -45,
          tickfont: { color: textColor }
        },
        yaxis: { 
          tickfont: { color: textColor }
        },
        height: 600,
        paper_bgcolor: bgColor,
        plot_bgcolor: bgColor,
        font: { color: textColor }
      };

      Plotly.newPlot(heatmapPlot.value, [trace], layout, {
        responsive: true,
        displaylogo: false
      });
    };

    return {
  selectedCities,
  year,
  k,
  n_clusters,  // ADD THIS
  loading,
  error,
  results,
  heatmapPlot,
  mapPlot,
  clusterGroups,
  runKNN,
  selectAll,
  formatFeatureName,
  formatMetricValue,
  getClusterColor
};

  }
};
</script>

<style scoped>
/* ===== CONTAINER ===== */
.knn-container {
  max-width: 1600px;
  margin: 0 auto;
}

.knn-section {
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
  max-width: 900px;
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

/* ===== CONFIG PANEL ===== */
.config-panel {
  background: linear-gradient(135deg, var(--color-bg-tertiary), var(--color-bg-secondary));
  border: 1px solid var(--color-border);
  border-radius: 12px;
  padding: 2rem;
  margin-bottom: 2rem;
}

.config-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 1.5rem 0;
}

.config-grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 2rem;
}

.config-group {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.config-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.config-label {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--color-text);
  margin-bottom: 0.75rem;
  display: block;
}

.selection-count {
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--color-primary);
  padding: 0.25rem 0.75rem;
  background: var(--color-primary-light);
  border-radius: 12px;
}

/* ===== CITY CHECKBOX LIST ===== */
.city-checkbox-list {
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 0.75rem;
  max-height: 300px;
  overflow-y: auto;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.5rem;
}

.city-checkbox-item {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.625rem 0.875rem;
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.city-checkbox-item:hover {
  background: var(--color-bg-secondary);
  border-color: var(--color-primary);
}

.city-checkbox-item.selected {
  background: var(--color-primary-light);
  border-color: var(--color-primary);
}

.checkbox-input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
}

.checkbox-custom {
  width: 18px;
  height: 18px;
  border: 2px solid var(--color-border);
  border-radius: 4px;
  flex-shrink: 0;
  transition: all 0.2s ease;
  position: relative;
}

.city-checkbox-item.selected .checkbox-custom {
  background: var(--color-primary);
  border-color: var(--color-primary);
}

.city-checkbox-item.selected .checkbox-custom::after {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
}

.city-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--color-text);
}

.selection-actions {
  display: flex;
  gap: 0.75rem;
}

.btn-clear,
.btn-select-all {
  flex: 1;
  padding: 0.625rem 1rem;
  border: 1px solid var(--color-border);
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-clear {
  background: var(--color-error-light);
  color: var(--color-error);
  border-color: var(--color-error);
}

.btn-clear:hover {
  background: var(--color-error);
  color: white;
}

.btn-select-all {
  background: var(--color-bg-tertiary);
  color: var(--color-text);
}

.btn-select-all:hover {
  background: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}

/* ===== PARAMETERS ===== */
.parameters-group {
  background: var(--color-bg);
  border-radius: 8px;
  padding: 1.5rem;
  border: 1px solid var(--color-border);
}

.parameter-item {
  margin-bottom: 1.5rem;
}

.year-selector {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.year-input {
  width: 100%;
  padding: 0.875rem 1rem;
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  color: var(--color-text);
  font-size: 1.125rem;
  font-weight: 600;
  text-align: center;
}

.year-input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--color-primary-light);
}

.year-hint {
  font-size: 0.8rem;
  color: var(--color-text-tertiary);
  text-align: center;
}

.k-selector {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 0.5rem;
}

.k-slider {
  flex: 1;
  height: 6px;
  border-radius: 3px;
  background: var(--color-bg-tertiary);
  outline: none;
  -webkit-appearance: none;
}

.k-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: var(--color-primary);
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.4);
}

.k-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: var(--color-primary);
  cursor: pointer;
  border: none;
}

.k-value-display {
  min-width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--color-primary);
  color: white;
  font-weight: 700;
  font-size: 1.125rem;
  border-radius: 8px;
}

.k-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: var(--color-text-tertiary);
}

.action-container {
  margin-top: auto;
}

.btn-run-analysis {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
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

.btn-run-analysis:hover:not(.disabled) {
  background: var(--color-primary-hover);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(59, 130, 246, 0.4);
}

.btn-run-analysis.disabled {
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

.requirement-hint {
  margin-top: 0.75rem;
  font-size: 0.85rem;
  color: var(--color-error);
  text-align: center;
  font-style: italic;
}

.ready-hint {
  margin-top: 0.75rem;
  font-size: 0.85rem;
  color: var(--color-success);
  text-align: center;
  font-weight: 500;
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
.results-wrapper {
  margin-top: 2rem;
}

/* Analysis Summary */
.analysis-summary {
  margin-bottom: 2rem;
}

.summary-card {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  padding: 1.5rem;
}

.summary-header {
  margin-bottom: 1.25rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--color-border);
}

.summary-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.summary-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.summary-label {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--color-text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.summary-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--color-text);
  font-variant-numeric: tabular-nums;
}

.features-list {
  padding-top: 1rem;
  border-top: 1px solid var(--color-border);
}

.features-title {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--color-text-secondary);
  margin: 0 0 0.75rem 0;
}

.feature-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.feature-tag {
  padding: 0.375rem 0.75rem;
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 500;
  color: var(--color-text);
}

/* Cluster Legend */
.clusters-section {
  margin-bottom: 2rem;
}

.cluster-legend-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
}

.cluster-badge-card {
  display: flex;
  gap: 1rem;
  padding: 1.25rem;
  background: var(--color-bg-tertiary);
  border: 2px solid var(--color-border);
  border-radius: 8px;
  border-left-width: 4px;
  transition: all 0.2s ease;
}

.cluster-badge-card:hover {
  transform: translateX(4px);
  box-shadow: var(--shadow-hover);
}

.cluster-color-indicator {
  width: 12px;
  border-radius: 6px;
  flex-shrink: 0;
}

.cluster-info {
  flex: 1;
}

.cluster-name {
  font-size: 1rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0 0 0.375rem 0;
}

.cluster-count {
  font-size: 0.85rem;
  color: var(--color-text-secondary);
  margin: 0 0 0.75rem 0;
}

.cluster-cities {
  display: flex;
  flex-wrap: wrap;
  gap: 0.375rem;
}

.cluster-city-tag {
  padding: 0.25rem 0.5rem;
  background: var(--color-bg);
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 500;
  color: var(--color-text);
}

.more-cities {
  padding: 0.25rem 0.5rem;
  background: var(--color-bg);
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 500;
  color: var(--color-text-tertiary);
  font-style: italic;
}

/* Map Section */
.map-section {
  margin-bottom: 2rem;
}

.map-header {
  margin-bottom: 1rem;
}

.map-description {
  font-size: 0.9rem;
  color: var(--color-text-secondary);
  margin: 0.5rem 0 0 0;
}

.map-wrapper {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 1rem;
}

.map-canvas {
  width: 100%;
  height: 600px;
  border-radius: 6px;
}

/* City Analysis Cards */
.city-analysis-section {
  margin-bottom: 2rem;
}

.knn-cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
  gap: 2rem;
  margin-top: 1.5rem;
}

.knn-analysis-card {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  overflow: hidden;
  transition: all 0.2s ease;
}

.knn-analysis-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-hover);
}

.card-header-bar {
  padding: 1.5rem;
  border-bottom: 1px solid var(--color-border);
}

.city-title-group {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 0.5rem;
}

.city-name {
  font-size: 1.375rem;
  font-weight: 600;
  color: var(--color-text);
  margin: 0;
}

.cluster-badge {
  padding: 0.375rem 0.875rem;
  border-radius: 12px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.data-points-info {
  font-size: 0.85rem;
  color: var(--color-text-secondary);
  font-style: italic;
}

/* Features Section */
.features-section {
  padding: 1.5rem;
  border-bottom: 1px solid var(--color-border);
}

.subsection-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--color-text-secondary);
  margin: 0 0 1rem 0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.metrics-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.75rem;
}

.metric-card {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.875rem;
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 6px;
}

.metric-icon {
  width: 32px;
  height: 32px;
  border-radius: 6px;
  flex-shrink: 0;
  background: linear-gradient(135deg, var(--color-primary), var(--color-primary-hover));
}

.metric-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.metric-label {
  font-size: 0.75rem;
  color: var(--color-text-secondary);
  font-weight: 500;
}

.metric-value {
  font-size: 1rem;
  font-weight: 700;
  color: var(--color-text);
  font-variant-numeric: tabular-nums;
}

/* Neighbors Section */
.neighbors-section {
  padding: 1.5rem;
}

.neighbors-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.neighbor-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.neighbor-card:hover {
  border-color: var(--color-primary);
}

.neighbor-rank-badge {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--color-primary);
  color: white;
  border-radius: 50%;
  font-weight: 700;
  font-size: 0.9rem;
  flex-shrink: 0;
}

.neighbor-details {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.neighbor-header {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.neighbor-city {
  font-size: 1rem;
  font-weight: 600;
  color: var(--color-text);
}

.neighbor-cluster {
  padding: 0.25rem 0.625rem;
  border-radius: 8px;
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
}

.neighbor-metrics {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.neighbor-metric {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 0.8rem;
  color: var(--color-text-secondary);
}

.neighbor-metric.highlight {
  color: var(--color-success);
  font-weight: 600;
}

.metric-icon-small {
  font-size: 0.875rem;
}

.similarity-indicator {
  width: 60px;
  height: 8px;
  background: var(--color-bg-tertiary);
  border-radius: 4px;
  overflow: hidden;
  flex-shrink: 0;
}

.similarity-bar {
  height: 100%;
  transition: width 0.3s ease;
  border-radius: 4px;
}

/* Heatmap Section */
.heatmap-section {
  margin-top: 2rem;
}

.heatmap-header {
  margin-bottom: 1rem;
}

.heatmap-description {
  font-size: 0.9rem;
  color: var(--color-text-secondary);
  margin: 0.5rem 0 0 0;
}

.heatmap-wrapper {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 1rem;
}

.heatmap-canvas {
  width: 100%;
  height: 600px;
  border-radius: 6px;
}

/* Empty State */
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
  max-width: 600px;
  margin: 0 auto;
  line-height: 1.6;
}

/* Transitions */
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

/* Responsive */
@media (max-width: 1200px) {
  .config-grid {
    grid-template-columns: 1fr;
  }
  
  .summary-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .knn-cards-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .knn-section {
    padding: 1.5rem;
  }
  
  .config-panel {
    padding: 1.5rem;
  }
  
  .section-title {
    font-size: 1.5rem;
  }
  
  .city-checkbox-list {
    grid-template-columns: 1fr;
  }
  
  .summary-grid {
    grid-template-columns: 1fr;
  }
  
  .cluster-legend-grid {
    grid-template-columns: 1fr;
  }
  
  .metrics-grid {
    grid-template-columns: 1fr;
  }
  
  .map-canvas,
  .heatmap-canvas {
    height: 400px;
  }
}

@media (max-width: 480px) {
  .knn-section {
    padding: 1rem;
  }
  
  .config-panel {
    padding: 1rem;
  }
  
  .section-title {
    font-size: 1.25rem;
  }
  
  .map-canvas,
  .heatmap-canvas {
    height: 350px;
  }
}
</style>
