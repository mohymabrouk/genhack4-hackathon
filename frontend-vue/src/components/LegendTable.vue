<template>
  <div class="section">
    <div class="card legend-card">
      <div class="card-header centered-header">
        <h2 class="card-title">Dataset Legend & Variable Guide</h2>
      </div>

      <!-- CATEGORY BLOCK -->
      <div
        v-for="group in groupedLegend"
        :key="group.title"
        class="legend-group"
      >
        <button class="legend-header" @click="toggle(group)">
          <span class="icon">{{ group.icon }}</span>
          <span class="title">{{ group.title }}</span>
          <span class="chevron">{{ group.open ? "▲" : "▼" }}</span>
        </button>

        <transition name="expand">
          <div v-show="group.open" class="legend-content">
            
            <!-- Responsive Grid -->
            <div class="legend-grid">
              <div
                v-for="item in group.items"
                :key="item.key"
                class="legend-item"
              >
                <div class="item-header">
                  <code class="var-name">{{ item.key }}</code>
                  <span class="pretty-name">{{ item.name }}</span>
                </div>

                <p class="description">{{ item.description }}</p>

                <div v-if="item.unit" class="unit">
                  <strong>Unit:</strong> {{ item.unit }}
                </div>
              </div>
            </div>

          </div>
        </transition>
      </div>

      <!-- DOWNLOAD -->
      <div class="download">
        <a
          :href="`http://localhost:5000/api/download/${city}/features`"
          class="btn btn-primary"
          download
        >
          Download Raw CSV
        </a>
      </div>
    </div>
  </div>
</template>



<script>
export default {
  name: "LegendTable",
  props: { city: String },

  data() {
    return {
      legend: this.buildLegend(),
      groups: [
        { title: "Core Observed Climate Variables", key: "core", open: false },
        { title: "ERA5 Reanalysis Variables", key: "era5", open: false },
        { title: "Anomaly Fields", key: "anomalies", open: false },
        { title: "Urban Heat Island Metrics", key: "uhi", open: false },
        { title: "Extreme Events",key: "extremes", open: false },
        { title: "Vegetation & NDVI Metrics",  key: "ndvi", open: false },
        { title: "Precipitation & Wet/Dry Metrics",key: "precip", open: false },
        { title: "Wind & Weather Regimes", key: "regimes", open: false },
        { title: "Bias & Diagnostics", key: "bias", open: false },
        { title: "Machine Learning Models",  key: "ml", open: false },
      ]
    };
  },

  computed: {
    groupedLegend() {
      return this.groups.map(g => ({
        ...g,
        items: this.legend[g.key]
      }));
    }
  },

  methods: {
    toggle(group) {
      const target = this.groups.find(g => g.title === group.title);
      target.open = !target.open;
    },


    buildLegend() {
      return {
        core: [
          { key: "TX_C_city_mean", name: "Max Temperature (°C)", description: "Daily maximum temperature from stations.", unit: "°C" },
          { key: "TN_C_city_mean", name: "Min Temperature (°C)", description: "Daily minimum temperature.", unit: "°C" },
          { key: "TG_C_city_mean", name: "Mean Temperature (°C)", description: "Daily average temperature.", unit: "°C" },
          { key: "RR_mm_city_mean", name: "Precipitation (mm)", description: "Daily rainfall.", unit: "mm" },
          { key: "PP_hPa_city_mean", name: "Surface Pressure", description: "Atmospheric pressure.", unit: "hPa" },
          { key: "FG_ms_city_mean", name: "Wind Speed", description: "Daily wind speed.", unit: "m/s" },
          { key: "NDVI_city", name: "Vegetation Index (NDVI)", description: "Green vegetation index.", unit: "NDVI" }
        ],

        era5: [
          { key: "ERA5_t2m_C", name: "ERA5 Temperature (°C)", description: "ERA5 2m temperature.", unit: "°C" },
          { key: "ERA5_tp_m", name: "ERA5 Precipitation", description: "ERA5 total precipitation.", unit: "m" },
          { key: "ERA5_u10_ms", name: "ERA5 Wind U", description: "East–west wind.", unit: "m/s" },
          { key: "ERA5_v10_ms", name: "ERA5 Wind V", description: "North–south wind.", unit: "m/s" }
        ],

        anomalies: [
          { key: "ERA5_t2m_C_anom", name: "ERA5 Temp Anomaly", description: "Deviation from climatology." },
          { key: "TX_C_city_mean_anom", name: "Max Temp Anomaly", description: "Station max temperature anomaly." },
          { key: "TN_C_city_mean_anom", name: "Min Temp Anomaly", description: "Station min temperature anomaly." },
          { key: "TG_C_city_mean_anom", name: "Mean Temp Anomaly", description: "Average temp anomaly." },
          { key: "NDVI_city_anom", name: "NDVI Anomaly", description: "Vegetation anomaly." }
        ],

        uhi: [
          { key: "uhi.mean", name: "Mean UHI", description: "Urban–rural temperature difference." },
          { key: "uhi.p90", name: "Peak UHI (P90)", description: "UHI during hottest days." },
          { key: "UHI_day_night", name: "Daytime UHI", description: "UHI computed from TX differences." },
          { key: "seasonal_uhi", name: "Seasonal UHI", description: "UHI by season." },
          { key: "uhi_wind_conditioned", name: "Wind-adjusted UHI", description: "Effect of ventilation on UHI." }
        ],

        extremes: [
          { key: "heatwaves", name: "Heatwaves", description: "Count and duration of heatwave events." },
          { key: "cold_spells", name: "Cold Spells", description: "Cold spell statistics." },
          { key: "precip_extremes", name: "Extreme Rainfall", description: "99th percentile precipitation." }
        ],

        ndvi: [
          { key: "ndvi_temp", name: "NDVI–Temp Correlation", description: "Vegetation/temperature relationship." },
          { key: "ndvi_temp_elasticity", name: "Temperature Elasticity to NDVI", description: "How NDVI affects temperature." },
          { key: "bias_by_ndvi_bin", name: "Bias by NDVI Class", description: "Bias in built-up vs green zones." }
        ],

        precip: [
          { key: "wet_dry_stats", name: "Wet vs Dry Days", description: "Temperature differences on wet/dry days." }
        ],

        regimes: [
          { key: "wind_heat_relation", name: "Wind–Heat Relation", description: "Correlation between wind and heat." },
          { key: "regime_metrics", name: "Calm/Windy Regimes", description: "How wind regimes affect temperature." },
          { key: "temp_regime_bias", name: "Temperature Regime Bias", description: "ERA5 bias across temperature bins." }
        ],

        bias: [
          { key: "t2m_vs_TX", name: "ERA5 vs Station Bias", description: "Bias, RMSE, correlation, KS test." },
          { key: "bias_distribution", name: "Bias Distribution Shape", description: "Mean, std, skew, kurtosis." },
          { key: "bias_quantiles", name: "Bias by Temperature Percentile", description: "Bias across TX deciles." },
          { key: "bias_stability", name: "Bias Stability", description: "30-day rolling bias variation." }
        ],

        ml: [
          { key: "xgboost", name: "XGBoost Bias Model", description: "Learns non-linear bias corrections." },
          { key: "quantile", name: "Quantile Regression", description: "Predicts 90th percentile extremes." },
          { key: "clustering", name: "Weather Regime Clustering", description: "K-means atmospheric regimes." },
          { key: "uhi_proxy", name: "UHI Driver Model", description: "Feature importance for UHI." },
          { key: "shap", name: "SHAP Explainability", description: "Breakdown of model decisions." }
        ]
      }
    }
  }
}
</script>


<style scoped>
/* Limit width - prevents full page stretching */
.legend-card {
  max-width: 900px;
  margin: 0 auto;
  padding: 1rem 1.5rem;
}

/* Centered title */
.centered-header {
  text-align: center;
  margin-bottom: 1.5rem;
}

.legend-group {
  padding: 0.5rem 0;
}

/* Compact accordion header */
.legend-header {
  color: var(--color-text);
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  
  padding: 0.7rem 1rem;
  background: var(--color-bg-secondary);

  border: 1px solid var(--color-border);
  border-radius: 8px;

  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;

  transition: background 0.2s ease, border-color 0.2s ease;
}

.legend-header:hover {
  background: var(--color-bg-tertiary);
}

/* Animation */
.expand-enter-active,
.expand-leave-active {
  transition: all 0.25s ease;
}
.expand-enter-from,
.expand-leave-to {
  opacity: 0;
  max-height: 0;
}

/* Responsive Grid for items */
.legend-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 0.75rem;
  margin-top: 0.8rem;
}

@media (min-width: 700px) {
  .legend-grid {
    grid-template-columns: 1fr 1fr;
  }
}

/* Item card */
.legend-item {
  padding: 0.8rem;
  background: var(--color-bg-tertiary);
  border-radius: 6px;
  border: 1px solid var(--color-border);
  box-shadow: 0 2px 4px rgb(0 0 0 / 5%);
}

.item-header {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  align-items: center;
}

.var-name {
  background: var(--color-bg-secondary);
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 0.78rem;
  border: 1px solid var(--color-border);
}

.pretty-name {
  font-weight: 600;
}

.description {
  font-size: 0.9rem;
  margin-top: 0.3rem;
  color: var(--color-text-secondary);
}

.unit {
  margin-top: 0.3rem;
  font-size: 0.85rem;
  color: var(--color-info);
}

.download {
  margin-top: 2rem;
  text-align: center;
}
</style>
