<template>
  <div class="metrics-container">
    <!-- ===== ERA5 vs STATION COMPARISON ===== -->
    <section class="metrics-section">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">ERA5 vs Station Temperature Comparison</h2>
          <p class="section-subtitle">Statistical validation of ERA5 reanalysis against ground observations</p>
        </div>
        <button class="btn-toggle" @click="toggleSection('era5')">
          <span class="toggle-icon" :class="{ open: openSections.era5 }">▼</span>
          {{ openSections.era5 ? 'Collapse' : 'Expand' }}
        </button>
      </div>
      
      <div class="comparison-grid">
        <!-- TX CARD -->
        <div class="metric-card" v-if="metrics.t2m_vs_TX">
          <div class="metric-card-header">
            <h3 class="metric-title">Maximum Temperature (TX)</h3>
            <span class="metric-badge">Primary</span>
          </div>
          <div class="metric-stats">
            <div class="stat-row stat-highlight">
              <span class="stat-label">Mean Bias</span>
              <span class="stat-value" :class="getBiasClass(metrics.t2m_vs_TX.mean_bias)">
                {{ fmt(metrics.t2m_vs_TX.mean_bias, 2) }} °C
              </span>
            </div>
            <div class="stat-row">
              <span class="stat-label">RMSE</span>
              <span class="stat-value">{{ fmt(metrics.t2m_vs_TX.rmse, 2) }} °C</span>
            </div>
            <div class="stat-row">
              <span class="stat-label">MAE</span>
              <span class="stat-value">{{ fmt(metrics.t2m_vs_TX.mae, 2) }} °C</span>
            </div>
            <div class="stat-row stat-divider">
              <span class="stat-label">Correlation</span>
              <span class="stat-value correlation" :class="getCorrelationClass(metrics.t2m_vs_TX.corr)">
                {{ fmt(metrics.t2m_vs_TX.corr, 3) }}
                <span class="correlation-bar" :style="{ width: (Math.abs(metrics.t2m_vs_TX.corr) * 100) + '%' }"></span>
              </span>
            </div>
            <div class="stat-row stat-meta">
              <span class="stat-label">Sample Size</span>
              <span class="stat-value-small">{{ formatNumber(metrics.t2m_vs_TX.n) }} days</span>
            </div>
          </div>
        </div>

        <!-- TN CARD -->
        <div class="metric-card" v-if="metrics.t2m_vs_TN">
          <div class="metric-card-header">
            <h3 class="metric-title">Minimum Temperature (TN)</h3>
            <span class="metric-badge metric-badge-secondary">Secondary</span>
          </div>
          <div class="metric-stats">
            <div class="stat-row stat-highlight">
              <span class="stat-label">Mean Bias</span>
              <span class="stat-value" :class="getBiasClass(metrics.t2m_vs_TN.mean_bias)">
                {{ fmt(metrics.t2m_vs_TN.mean_bias, 2) }} °C
              </span>
            </div>
            <div class="stat-row">
              <span class="stat-label">RMSE</span>
              <span class="stat-value">{{ fmt(metrics.t2m_vs_TN.rmse, 2) }} °C</span>
            </div>
            <div class="stat-row">
              <span class="stat-label">MAE</span>
              <span class="stat-value">{{ fmt(metrics.t2m_vs_TN.mae, 2) }} °C</span>
            </div>
            <div class="stat-row stat-divider">
              <span class="stat-label">Correlation</span>
              <span class="stat-value correlation" :class="getCorrelationClass(metrics.t2m_vs_TN.corr)">
                {{ fmt(metrics.t2m_vs_TN.corr, 3) }}
                <span class="correlation-bar" :style="{ width: (Math.abs(metrics.t2m_vs_TN.corr) * 100) + '%' }"></span>
              </span>
            </div>
            <div class="stat-row stat-meta">
              <span class="stat-label">Sample Size</span>
              <span class="stat-value-small">{{ formatNumber(metrics.t2m_vs_TN.n) }} days</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Extended Details -->
      <transition name="expand">
        <div v-if="openSections.era5" class="detail-panel">
          <h4 class="detail-title">Detailed Performance Metrics</h4>
          <div class="detail-grid">
            <div class="detail-item">
              <span class="detail-label">TX: Hot-day Bias</span>
              <span class="detail-value" :class="getBiasClass(metrics.t2m_vs_TX.hot_bias)">
                {{ fmt(metrics.t2m_vs_TX.hot_bias, 2) }} °C
              </span>
            </div>
            <div class="detail-item">
              <span class="detail-label">TX: Hot-day RMSE</span>
              <span class="detail-value">{{ fmt(metrics.t2m_vs_TX.hot_rmse, 2) }} °C</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">KS Statistic (TX)</span>
              <span class="detail-value">{{ fmt(metrics.t2m_vs_TX.ks_stat, 4) }}</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">KS p-value (TX)</span>
              <span class="detail-value" :class="getPValueClass(metrics.t2m_vs_TX.ks_p)">
                {{ fmt(metrics.t2m_vs_TX.ks_p, 4) }}
              </span>
            </div>
          </div>
        </div>
      </transition>
    </section>

    <!-- ===== URBAN HEAT ISLAND (UHI) ===== -->
    <section class="metrics-section" v-if="metrics.uhi_distance_summer_night">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Urban Heat Island (UHI) Analysis</h2>
          <p class="section-subtitle">Summer night UHI intensity based on distance gradient method</p>
        </div>
        <button class="btn-toggle" @click="toggleSection('uhi')">
          <span class="toggle-icon" :class="{ open: openSections.uhi }">▼</span>
          {{ openSections.uhi ? 'Collapse' : 'Expand' }}
        </button>
      </div>
      
      <div class="uhi-summary">
        <div class="uhi-stat-card">
          <div class="uhi-stat-icon mean-icon"></div>
          <div class="uhi-stat-content">
            <span class="uhi-stat-value">{{ fmt(metrics.uhi_distance_summer_night.mean, 2) }} °C</span>
            <span class="uhi-stat-label">Mean UHI Intensity</span>
          </div>
        </div>
        <div class="uhi-stat-card">
          <div class="uhi-stat-icon median-icon"></div>
          <div class="uhi-stat-content">
            <span class="uhi-stat-value">{{ fmt(metrics.uhi_distance_summer_night.median, 2) }} °C</span>
            <span class="uhi-stat-label">Median UHI</span>
          </div>
        </div>
        <div class="uhi-stat-card">
          <div class="uhi-stat-icon std-icon"></div>
          <div class="uhi-stat-content">
            <span class="uhi-stat-value">{{ fmt(metrics.uhi_distance_summer_night.std, 2) }} °C</span>
            <span class="uhi-stat-label">Standard Deviation</span>
          </div>
        </div>
        <div class="uhi-stat-card">
          <div class="uhi-stat-icon range-icon"></div>
          <div class="uhi-stat-content">
            <span class="uhi-stat-value">{{ fmt(metrics.uhi_distance_summer_night.max, 2) }} °C</span>
            <span class="uhi-stat-label">Maximum UHI</span>
          </div>
        </div>
      </div>

      <div class="uhi-meta">
        <span class="meta-info">
          Based on {{ metrics.uhi_distance_summer_night.n_days }} days of observations 
          ({{ metrics.uhi_distance_summer_night.season }}, {{ metrics.uhi_distance_summer_night.conditions }})
        </span>
      </div>

      <!-- Extended UHI Details -->
      <transition name="expand">
        <div v-if="openSections.uhi" class="detail-section">
          <div class="detail-columns">
            <div class="detail-column">
              <h4 class="detail-subtitle">UHI Percentiles</h4>
              <div class="detail-list">
                <div class="detail-list-item">
                  <span class="list-label">10th Percentile</span>
                  <span class="list-value">{{ fmt(metrics.uhi_distance_summer_night.p10, 2) }} °C</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">90th Percentile</span>
                  <span class="list-value">{{ fmt(metrics.uhi_distance_summer_night.p90, 2) }} °C</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Minimum UHI</span>
                  <span class="list-value">{{ fmt(metrics.uhi_distance_summer_night.min, 2) }} °C</span>
                </div>
              </div>
            </div>
            <div class="detail-column">
              <h4 class="detail-subtitle">Distance Parameters</h4>
              <div class="detail-list">
                <div class="detail-list-item">
                  <span class="list-label">Urban Max Radius</span>
                  <span class="list-value">{{ fmt(metrics.uhi_distance_summer_night.urban_max_km, 1) }} km</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Rural Min Radius</span>
                  <span class="list-value">{{ fmt(metrics.uhi_distance_summer_night.rural_min_km, 1) }} km</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </section>

    <!-- ===== VEGETATION & NDVI ===== -->
    <section class="metrics-section" v-if="metrics.ndvi_surface_cooling">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Vegetation Cooling Effect</h2>
          <p class="section-subtitle">NDVI impact on surface temperature</p>
        </div>
        <button class="btn-toggle" @click="toggleSection('ndvi')">
          <span class="toggle-icon" :class="{ open: openSections.ndvi }">▼</span>
          {{ openSections.ndvi ? 'Collapse' : 'Expand' }}
        </button>
      </div>
      
      <div class="ndvi-summary">
        <div class="ndvi-stat-card">
          <div class="ndvi-stat-icon cooling-icon"></div>
          <div class="ndvi-stat-content">
            <span class="ndvi-stat-value">{{ fmt(metrics.ndvi_surface_cooling.elasticity_surface_C_per_NDVI, 2) }} °C</span>
            <span class="ndvi-stat-label">   Cooling per NDVI unit</span>
          </div>
        </div>
        <div class="ndvi-stat-card">
          <div class="ndvi-stat-icon correlation-icon"></div>
          <div class="ndvi-stat-content">
            <span class="ndvi-stat-value" :class="getCorrelationClass(metrics.ndvi_surface_cooling.corr_surface_deseas)">
              {{ fmt(metrics.ndvi_surface_cooling.corr_surface_deseas, 3) }}
            </span>
            <span class="ndvi-stat-label">  Correlation (deseasonalized)</span>
          </div>
        </div>
      </div>

      <div class="uhi-meta">
        <span class="meta-info">Based on {{ metrics.ndvi_surface_cooling.n_days }} days of observations</span>
      </div>
    </section>

    <!-- ===== EXTREME EVENTS ===== -->
    <section class="metrics-section">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Extreme Weather Events</h2>
          <p class="section-subtitle">Heatwave, cold spell, and precipitation extremes</p>
        </div>
        <button class="btn-toggle" @click="toggleSection('extremes')">
          <span class="toggle-icon" :class="{ open: openSections.extremes }">▼</span>
          {{ openSections.extremes ? 'Collapse' : 'Expand' }}
        </button>
      </div>
      
      <div class="extremes-grid">
        <!-- Heatwaves -->
        <div class="extreme-card heatwave-card" v-if="metrics.heatwaves">
          <div class="extreme-header">
            <div class="extreme-icon hot-icon"></div>
            <h3 class="extreme-title">Heatwaves</h3>
          </div>
          <div class="extreme-stats">
            <div class="extreme-stat">
              <span class="extreme-value">{{ metrics.heatwaves.count }}</span>
              <span class="extreme-label">Events</span>
            </div>
            <div class="extreme-stat">
              <span class="extreme-value">{{ fmt(metrics.heatwaves.mean_length, 1) }}</span>
              <span class="extreme-label">Avg Duration (days)</span>
            </div>
            <div class="extreme-stat">
              <span class="extreme-value">{{ metrics.heatwaves.max_length }}</span>
              <span class="extreme-label">Max Duration (days)</span>
            </div>
          </div>
        </div>

        <!-- Cold Spells -->
        <div class="extreme-card coldspell-card" v-if="metrics.cold_spells">
          <div class="extreme-header">
            <div class="extreme-icon cold-icon"></div>
            <h3 class="extreme-title">Cold Spells</h3>
          </div>
          <div class="extreme-stats">
            <div class="extreme-stat">
              <span class="extreme-value">{{ metrics.cold_spells.count }}</span>
              <span class="extreme-label">Events</span>
            </div>
            <div class="extreme-stat">
              <span class="extreme-value">{{ fmt(metrics.cold_spells.mean_length, 1) }}</span>
              <span class="extreme-label">Avg Duration (days)</span>
            </div>
            <div class="extreme-stat">
              <span class="extreme-value">{{ metrics.cold_spells.max_length }}</span>
              <span class="extreme-label">Max Duration (days)</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Extended Details -->
      <transition name="expand">
        <div v-if="openSections.extremes" class="detail-section">
          <div class="detail-columns">
            <!-- Heatwave Bias -->
            <div class="detail-column" v-if="metrics.heatwave_bias">
              <h4 class="detail-subtitle">ERA5 Performance on Heatwaves</h4>
              <div class="detail-list">
                <div class="detail-list-item">
                  <span class="list-label">Temperature Threshold (90th)</span>
                  <span class="list-value">{{ fmt(metrics.heatwave_bias.threshold_90th, 1) }} °C</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Mean Bias</span>
                  <span class="list-value" :class="getBiasClass(metrics.heatwave_bias.mean_bias)">
                    {{ fmt(metrics.heatwave_bias.mean_bias, 2) }} °C
                  </span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">RMSE</span>
                  <span class="list-value">{{ fmt(metrics.heatwave_bias.rmse, 2) }} °C</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Max Underestimate</span>
                  <span class="list-value warning">{{ fmt(metrics.heatwave_bias.max_underestimate, 2) }} °C</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Heatwave Days</span>
                  <span class="list-value">{{ metrics.heatwave_bias.count }}</span>
                </div>
              </div>
            </div>

            <!-- Precipitation Extremes -->
            <div class="detail-column" v-if="metrics.precip_extremes">
              <h4 class="detail-subtitle">Precipitation Extremes</h4>
              <div class="detail-list">
                <div class="detail-list-item">
                  <span class="list-label">Top 1% Threshold</span>
                  <span class="list-value">{{ fmt(metrics.precip_extremes.top1pct_threshold, 1) }} mm</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Extreme Day Count</span>
                  <span class="list-value">{{ metrics.precip_extremes.extreme_day_count }} days</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Mean on Extreme Days</span>
                  <span class="list-value">{{ fmt(metrics.precip_extremes.extreme_day_mean, 1) }} mm</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </section>

    <!-- ===== SEASONAL CLIMATE ===== -->
    <section class="metrics-section" v-if="metrics.seasonal_means">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Seasonal Climate Summary</h2>
          <p class="section-subtitle">Temperature and precipitation by season</p>
        </div>
      </div>
      
      <div class="data-table-wrapper">
        <table class="data-table seasonal-table">
          <thead>
            <tr>
              <th>Season</th>
              <th>Mean Temperature (°C)</th>
              <th>Total Precipitation (mm)</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(data, season) in metrics.seasonal_means" :key="season">
              <td class="season-name">{{ season }}</td>
              <td class="numeric-cell">{{ fmt(data.temp_mean, 1) }}</td>
              <td class="numeric-cell">{{ fmt(data.precip_sum, 0) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- ===== METEOROLOGICAL REGIMES ===== -->
    <section class="metrics-section">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Meteorological Regimes</h2>
          <p class="section-subtitle">Temperature patterns by weather conditions</p>
        </div>
        <button class="btn-toggle" @click="toggleSection('regimes')">
          <span class="toggle-icon" :class="{ open: openSections.regimes }">▼</span>
          {{ openSections.regimes ? 'Collapse' : 'Expand' }}
        </button>
      </div>
      
      <div class="regime-summary">
        <!-- Wind Regime -->
        <div class="regime-card" v-if="metrics.regime_metrics">
          <h4 class="regime-title">Wind Regime Impact</h4>
          <div class="regime-comparison">
            <div class="regime-item">
              <span class="regime-label">Calm Days</span>
              <span class="regime-value">{{ fmt(metrics.regime_metrics.calm_mean_temp, 1) }} °C</span>
            </div>
            <div class="regime-divider">vs</div>
            <div class="regime-item">
              <span class="regime-label">Windy Days</span>
              <span class="regime-value">{{ fmt(metrics.regime_metrics.windy_mean_temp, 1) }} °C</span>
            </div>
          </div>
          <div class="regime-difference">
            <span class="diff-label">Temperature Difference:</span>
            <span class="diff-value">{{ fmt(metrics.regime_metrics.difference, 2) }} °C</span>
          </div>
        </div>

        <!-- Wind-Heat Relationship -->
        <div class="regime-card" v-if="metrics.wind_heat_relation">
          <h4 class="regime-title">Wind–Heat Relationship</h4>
          <div class="regime-stats">
            <div class="regime-stat-item">
              <span class="stat-label-small">Temperature–Wind Correlation</span>
              <span class="stat-value-large" :class="getCorrelationClass(metrics.wind_heat_relation.corr_temp_wind)">
                {{ fmt(metrics.wind_heat_relation.corr_temp_wind, 3) }}
              </span>
            </div>
            <div class="regime-stat-item">
              <span class="stat-label-small">Mean Wind Speed (Hot Days)</span>
              <span class="stat-value-large">
                {{ fmt(metrics.wind_heat_relation.mean_hotday_wind, 2) }} m/s
              </span>
            </div>
          </div>
        </div>
      </div>

      <!-- Extended Details -->
      <transition name="expand">
        <div v-if="openSections.regimes" class="detail-section">
          <div class="detail-columns">
            <!-- Wet/Dry Stats -->
            <div class="detail-column" v-if="metrics.wet_dry_stats">
              <h4 class="detail-subtitle">Precipitation Regime</h4>
              <div class="detail-list">
                <div class="detail-list-item">
                  <span class="list-label">Wet Days</span>
                  <span class="list-value">{{ metrics.wet_dry_stats.wet_count }} days</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Dry Days</span>
                  <span class="list-value">{{ metrics.wet_dry_stats.dry_count }} days</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Wet Day Mean TX</span>
                  <span class="list-value">{{ fmt(metrics.wet_dry_stats.wet_temp_mean, 1) }} °C</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Dry Day Mean TX</span>
                  <span class="list-value">{{ fmt(metrics.wet_dry_stats.dry_temp_mean, 1) }} °C</span>
                </div>
              </div>
            </div>

            <!-- Temperature Variability -->
            <div class="detail-column" v-if="metrics.temp_variability">
              <h4 class="detail-subtitle">Temperature Variability</h4>
              <div class="detail-list">
                <div class="detail-list-item">
                  <span class="list-label">Standard Deviation</span>
                  <span class="list-value">{{ fmt(metrics.temp_variability.std_temp, 2) }} °C</span>
                </div>
                <div class="detail-list-item">
                  <span class="list-label">Interquartile Range</span>
                  <span class="list-value">{{ fmt(metrics.temp_variability.iqr_temp, 2) }} °C</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </section>

    <!-- ===== TEMPERATURE REGIME BIAS ===== -->
    <section class="metrics-section" v-if="metrics.temp_regime_bias">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Bias by Temperature Regime</h2>
          <p class="section-subtitle">ERA5 performance across different temperature ranges</p>
        </div>
      </div>
      
      <div class="data-table-wrapper">
        <table class="data-table regime-table">
          <thead>
            <tr>
              <th>Temperature Regime</th>
              <th>Mean Bias (°C)</th>
              <th>RMSE (°C)</th>
              <th>Sample Size</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(data, regime) in metrics.temp_regime_bias" :key="regime">
              <td class="regime-name">{{ formatRegimeName(regime) }}</td>
              <td class="numeric-cell" :class="getBiasClass(data.mean_bias)">
                {{ fmt(data.mean_bias, 2) }}
              </td>
              <td class="numeric-cell">{{ fmt(data.rmse, 2) }}</td>
              <td class="numeric-cell">{{ formatNumber(data.count) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- ===== BIAS DISTRIBUTION & STRUCTURE ===== -->
    <section class="metrics-section">
      <div class="section-header">
        <div class="section-title-group">
          <h2 class="section-title">Bias Distribution & Structure</h2>
          <p class="section-subtitle">Statistical properties of ERA5 bias</p>
        </div>
        <button class="btn-toggle" @click="toggleSection('bias')">
          <span class="toggle-icon" :class="{ open: openSections.bias }">▼</span>
          {{ openSections.bias ? 'Collapse' : 'Expand' }}
        </button>
      </div>
      
      <div class="bias-summary-grid">
        <!-- Bias Distribution -->
        <div class="bias-stat-card" v-if="metrics.bias_distribution">
          <h4 class="bias-stat-title">Distribution Parameters</h4>
          <div class="bias-stat-list">
            <div class="bias-stat-row">
              <span class="bias-label">Mean</span>
              <span class="bias-value" :class="getBiasClass(metrics.bias_distribution.mean)">
                {{ fmt(metrics.bias_distribution.mean, 2) }} °C
              </span>
            </div>
            <div class="bias-stat-row">
              <span class="bias-label">Std Dev</span>
              <span class="bias-value">{{ fmt(metrics.bias_distribution.std, 2) }} °C</span>
            </div>
            <div class="bias-stat-row">
              <span class="bias-label">Skewness</span>
              <span class="bias-value">{{ fmt(metrics.bias_distribution.skew, 3) }}</span>
            </div>
            <div class="bias-stat-row">
              <span class="bias-label">Kurtosis</span>
              <span class="bias-value">{{ fmt(metrics.bias_distribution.kurtosis, 3) }}</span>
            </div>
          </div>
        </div>

        <!-- Bias Stability -->
        <div class="bias-stat-card" v-if="metrics.bias_stability">
          <h4 class="bias-stat-title">Temporal Stability</h4>
          <div class="stability-metric">
            <span class="stability-label">30-Day Rolling Bias Std Dev</span>
            <span class="stability-value">{{ fmt(metrics.bias_stability.rolling_bias_std, 3) }} °C</span>
            <p class="stability-note">Lower values indicate more consistent bias patterns</p>
          </div>
        </div>
      </div>

      <!-- Extended Details -->
      <transition name="expand">
        <div v-if="openSections.bias" class="detail-section">
          <!-- Bias Quantiles -->
          <div v-if="metrics.bias_quantiles" class="quantile-section">
            <h4 class="detail-subtitle">Bias by Temperature Percentile</h4>
            <div class="data-table-wrapper">
              <table class="data-table quantile-table">
                <thead>
                  <tr>
                    <th>Percentile Range</th>
                    <th>Mean Bias (°C)</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(value, key) in metrics.bias_quantiles" :key="key">
                    <td class="percentile-cell">{{ key.replace('-', '-') }}%</td>
                    <td class="numeric-cell" :class="getBiasClass(value)">
                      {{ fmt(value, 2) }}
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <!-- Correlation Matrix -->
          <div v-if="metrics.correlation_matrix" class="correlation-section">
            <div class="correlation-header">
              <h4 class="detail-subtitle">Variable Correlation Matrix</h4>
              <button class="btn-toggle-small" @click="toggleSection('correlation')">
                {{ openSections.correlation ? 'Hide Matrix' : 'Show Matrix' }}
              </button>
            </div>
            <transition name="expand">
              <div v-if="openSections.correlation" class="correlation-matrix-wrapper">
                <div class="data-table-wrapper">
                  <table class="data-table correlation-table">
                    <thead>
                      <tr>
                        <th class="matrix-header"></th>
                        <th v-for="(colVal, colKey) in metrics.correlation_matrix" :key="colKey" class="matrix-header">
                          {{ formatVariableName(colKey) }}
                        </th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(row, rowKey) in metrics.correlation_matrix" :key="rowKey">
                        <td class="matrix-row-header">{{ formatVariableName(rowKey) }}</td>
                        <td v-for="(val, colKey) in row" :key="colKey" class="correlation-cell"
                          :class="getHeatmapClass(val)"
                          :title="`${formatVariableName(rowKey)} vs ${formatVariableName(colKey)}: ${fmt(val, 3)}`">
                          {{ fmt(val, 2) }}
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </transition>
          </div>
        </div>
      </transition>
    </section>
  </div>
</template>

<script>
export default {
  name: 'MetricsOverview',
  props: {
    metrics: {
      type: Object,
      required: true,
      default: () => ({})
    },
    city: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      openSections: {
        era5: false,
        uhi: false,
        extremes: false,
        ndvi: false,
        regimes: false,
        bias: false,
        correlation: false
      }
    }
  },
  methods: {
    toggleSection(section) {
      this.openSections[section] = !this.openSections[section]
    },
    
    fmt(value, digits = 2) {
      if (value === null || value === undefined || isNaN(value)) {
        return '—'
      }
      return Number(value).toFixed(digits)
    },
    
    formatNumber(num) {
      if (num === null || num === undefined) return '—'
      return num.toLocaleString()
    },
    
    getBiasClass(bias) {
      if (bias === null || bias === undefined || isNaN(bias)) return ''
      if (bias > 1) return 'bias-positive'
      if (bias < -1) return 'bias-negative'
      return 'bias-neutral'
    },
    
    getCorrelationClass(corr) {
      if (corr === null || corr === undefined || isNaN(corr)) return ''
      const absCorr = Math.abs(corr)
      if (absCorr >= 0.8) return 'corr-strong'
      if (absCorr >= 0.5) return 'corr-moderate'
      if (absCorr >= 0.3) return 'corr-weak'
      return 'corr-very-weak'
    },
    
    getPValueClass(pValue) {
      if (pValue === null || pValue === undefined || isNaN(pValue)) return ''
      if (pValue < 0.001) return 'p-significant'
      if (pValue < 0.01) return 'p-very-significant'
      if (pValue < 0.05) return 'p-significant'
      return 'p-not-significant'
    },
    
    getHeatmapClass(value) {
      if (value === null || value === undefined || isNaN(value)) return ''
      if (value >= 0.8) return 'heat-very-strong'
      if (value >= 0.5) return 'heat-strong'
      if (value >= 0.3) return 'heat-moderate'
      if (value >= 0.1) return 'heat-weak'
      if (value >= -0.1) return 'heat-neutral'
      if (value >= -0.3) return 'heat-negative-weak'
      if (value >= -0.5) return 'heat-negative-moderate'
      if (value >= -0.8) return 'heat-negative-strong'
      return 'heat-negative-very-strong'
    },
    
    formatRegimeName(regime) {
      const names = {
        'below_0': 'Below 0°C',
        '0_10': '0-10°C',
        '10_20': '10-20°C',
        '20_30': '20-30°C',
        'above_30': 'Above 30°C'
      }
      return names[regime] || regime.replace('_', '-').replace('_', '-')
    },
    
    formatVariableName(variable) {
      const names = {
        'ERA5_t2m_max_C': 'ERA5 Tmax',
        'TX_C_city_mean': 'Station TX',
        'TN_C_city_mean': 'Station TN',
        'RR_mm_city_mean': 'Precipitation',
        'PP_hPa_city_mean': 'Pressure',
        'FG_ms_city_mean': 'Wind Speed',
        'NDVI_city': 'NDVI'
      }
      return names[variable] || variable
    }
  }
}
</script>

<style scoped>
/* ===== CONTAINER ===== */
.metrics-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

/* ===== SECTION STRUCTURE ===== */
.metrics-section {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 2rem;
  margin-bottom: 2rem;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  transition: all 0.3s ease;
}

.metrics-section:hover {
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 2rem;
  padding-bottom: 1.5rem;
  border-bottom: 2px solid #e2e8f0;
}

.section-title-group {
  flex: 1;
}

.section-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #1a202c;
  margin: 0 0 0.5rem 0;
  line-height: 1.3;
}

.section-subtitle {
  font-size: 0.95rem;
  color: #718096;
  margin: 0;
  font-weight: 400;
}

/* ===== TOGGLE BUTTON ===== */
.btn-toggle {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.625rem 1rem;
  background: #f7fafc;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  color: #4a5568;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.btn-toggle:hover {
  background: #edf2f7;
  border-color: #4299e1;
  color: #2d3748;
}

.toggle-icon {
  display: inline-block;
  transition: transform 0.3s ease;
  font-size: 0.75rem;
  color: #718096;
}

.toggle-icon.open {
  transform: rotate(180deg);
}

.btn-toggle-small {
  padding: 0.375rem 0.75rem;
  font-size: 0.8rem;
  background: #f7fafc;
  border: 1px solid #e2e8f0;
  border-radius: 4px;
  color: #4a5568;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-toggle-small:hover {
  background: #4299e1;
  color: white;
  border-color: #4299e1;
}

/* ===== COMPARISON GRID ===== */
.comparison-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.metric-card {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1.5rem;
  transition: all 0.2s ease;
}

.metric-card:hover {
  border-color: #4299e1;
  box-shadow: 0 4px 6px -1px rgba(66, 153, 225, 0.1), 0 2px 4px -1px rgba(66, 153, 225, 0.06);
  transform: translateY(-2px);
}

.metric-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.25rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #e2e8f0;
}

.metric-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: #1a202c;
  margin: 0;
}

.metric-badge {
  padding: 0.25rem 0.625rem;
  background: #4299e1;
  color: white;
  border-radius: 4px;
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.metric-badge-secondary {
  background: #38b2ac;
}

.metric-badge-tertiary {
  background: #a0aec0;
}

/* ===== METRIC STATS ===== */
.metric-stats {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.stat-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.625rem 0;
}

.stat-row.stat-highlight {
  padding: 0.875rem 0;
  border-bottom: 1px solid #e2e8f0;
}

.stat-row.stat-divider {
  padding-top: 1rem;
  border-top: 1px solid #e2e8f0;
}

.stat-row.stat-meta {
  padding-top: 0.5rem;
  border-top: 1px dashed #e2e8f0;
}

.stat-label {
  font-size: 0.9rem;
  color: #718096;
  font-weight: 500;
}

.stat-value {
  font-size: 1rem;
  font-weight: 600;
  color: #1a202c;
  font-variant-numeric: tabular-nums;
}

.stat-value-small {
  font-size: 0.85rem;
  color: #718096;
}

/* ===== BIAS COLOR CODING ===== */
.bias-positive {
  color: #e53e3e;
  font-weight: 700;
}

.bias-negative {
  color: #3182ce;
  font-weight: 700;
}

.bias-neutral {
  color: #38a169;
}

/* ===== CORRELATION INDICATORS ===== */
.stat-value.correlation {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.25rem;
  position: relative;
}

.correlation-bar {
  height: 3px;
  background: #4299e1;
  border-radius: 2px;
  transition: width 0.3s ease;
}

.corr-strong {
  color: #38a169;
  font-weight: 700;
}

.corr-moderate {
  color: #3182ce;
  font-weight: 600;
}

.corr-weak {
  color: #d69e2e;
}

.corr-very-weak {
  color: #a0aec0;
}

/* ===== DETAIL PANEL ===== */
.detail-panel {
  background: #f7fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1.5rem;
  margin-top: 1.5rem;
}

.detail-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: #1a202c;
  margin: 0 0 1.25rem 0;
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.detail-item {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  padding: 0.75rem;
  background: white;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.detail-label {
  font-size: 0.8rem;
  color: #718096;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.detail-value {
  font-size: 1.125rem;
  font-weight: 600;
  color: #1a202c;
  font-variant-numeric: tabular-nums;
}

.detail-value.warning {
  color: #dd6b20;
  font-weight: 700;
}

/* ===== UHI SECTION ===== */
.uhi-summary {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.uhi-stat-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.25rem;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.uhi-stat-card:hover {
  border-color: #4299e1;
  transform: translateY(-2px);
  box-shadow: 0 4px 6px -1px rgba(66, 153, 225, 0.1);
}

.uhi-stat-icon {
  width: 48px;
  height: 48px;
  border-radius: 8px;
  flex-shrink: 0;
}

.mean-icon {
  background: linear-gradient(135deg, #4299e1, #3182ce);
}

.median-icon {
  background: linear-gradient(135deg, #805ad5, #6b46c1);
}

.std-icon {
  background: linear-gradient(135deg, #38b2ac, #319795);
}

.range-icon {
  background: linear-gradient(135deg, #ed8936, #dd6b20);
}

.cooling-icon {
  background: linear-gradient(135deg, #48bb78, #38a169);
}

.correlation-icon {
  background: linear-gradient(135deg, #3182ce, #2c5282);
}

.uhi-stat-content {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  flex: 1;
}

.uhi-stat-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #1a202c;
  line-height: 1;
  font-variant-numeric: tabular-nums;
}

.uhi-stat-label {
  font-size: 0.8rem;
  color: #718096;
  font-weight: 500;
}

.uhi-meta {
  text-align: center;
  padding: 0.75rem;
  background: #f7fafc;
  border-radius: 6px;
  border: 1px dashed #e2e8f0;
  margin-top: 1rem;
}

.meta-info {
  font-size: 0.875rem;
  color: #718096;
  font-style: italic;
}

/* ===== NDVI SUMMARY ===== */
.ndvi-summary {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.ndvi-stat-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.25rem;

  border: 1px solid #9ae6b4;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.ndvi-stat-card:hover {
  border-color: #48bb78;
  transform: translateY(-2px);
  box-shadow: 0 4px 6px -1px rgba(72, 187, 120, 0.1);
}

/* ===== DETAIL SECTIONS ===== */
.detail-section {
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #e2e8f0;
}

.detail-columns {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.detail-column {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.detail-subtitle {
  font-size: 1rem;
  font-weight: 600;
  color: #1a202c;
  margin: 0 0 1rem 0;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #e2e8f0;
}

.detail-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.detail-list-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.625rem 0.875rem;
  background: white;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.detail-list-item.highlight {
  background: #ebf8ff;
  border-color: #4299e1;
}

.list-label {
  font-size: 0.875rem;
  color: #718096;
  font-weight: 500;
}

.list-value {
  font-size: 0.9rem;
  font-weight: 600;
  color: #1a202c;
  font-variant-numeric: tabular-nums;
}

/* ===== DATA TABLES ===== */
.data-table-wrapper {
  width: 100%;
  overflow-x: auto;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
  background: white;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.data-table thead {
  background: #f7fafc;
  border-bottom: 2px solid #e2e8f0;
}

.data-table th {
  padding: 0.875rem 1rem;
  text-align: left;
  font-weight: 600;
  font-size: 0.85rem;
  color: #4a5568;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.data-table td {
  padding: 0.75rem 1rem;
  border-top: 1px solid #e2e8f0;
  color: #4a5568;
}

.data-table tbody tr {
  transition: background-color 0.15s ease;
}

.data-table tbody tr:hover {
  background-color: #f7fafc;
}

.season-name {
  text-transform: capitalize;
  font-weight: 600;
  color: #1a202c;
}

.regime-name {
  font-weight: 600;
  color: #1a202c;
}

.numeric-cell {
  text-align: right;
  font-variant-numeric: tabular-nums;
  font-weight: 500;
}

.percentile-cell {
  font-weight: 500;
}

/* ===== EXTREMES SECTION ===== */
.extremes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.extreme-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1.5rem;
  transition: all 0.2s ease;
}

.extreme-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

.heatwave-card {
  border-left: 4px solid #e53e3e;
}

.coldspell-card {
  border-left: 4px solid #3182ce;
}

.extreme-header {
  display: flex;
  align-items: center;
  gap: 0.875rem;
  margin-bottom: 1.25rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #e2e8f0;
}

.extreme-icon {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  flex-shrink: 0;
}

.hot-icon {
  background: linear-gradient(135deg, #fc8181, #f56565);
}

.cold-icon {
  background: linear-gradient(135deg, #90cdf4, #4299e1);
}

.extreme-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: #1a202c;
  margin: 0;
}

.extreme-stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.extreme-stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.375rem;
  text-align: center;
}

.extreme-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: #1a202c;
  line-height: 1;
  font-variant-numeric: tabular-nums;
}

.extreme-label {
  font-size: 0.75rem;
  color: #718096;
  font-weight: 500;
}

/* ===== REGIME SECTION ===== */
.regime-summary {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.regime-card {
  padding: 1.5rem;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
}

.regime-title {
  font-size: 1rem;
  font-weight: 600;
  color: #1a202c;
  margin: 0 0 1.25rem 0;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid #e2e8f0;
}

.regime-comparison {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 1rem;
  align-items: center;
  margin-bottom: 1rem;
}

.regime-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem;
  background: white;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.regime-label {
  font-size: 0.85rem;
  color: #718096;
  font-weight: 500;
}

.regime-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #1a202c;
  font-variant-numeric: tabular-nums;
}

.regime-divider {
  font-size: 0.9rem;
  font-weight: 600;
  color: #a0aec0;
}

.regime-difference {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.875rem 1rem;
  background: #ebf8ff;
  border-radius: 6px;
  border: 1px solid #4299e1;
}

.diff-label {
  font-size: 0.9rem;
  font-weight: 500;
  color: #2d3748;
}

.diff-value {
  font-size: 1.125rem;
  font-weight: 700;
  color: #2b6cb0;
  font-variant-numeric: tabular-nums;
}

.regime-stats {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.regime-stat-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.875rem 1rem;
  background: white;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.stat-label-small {
  font-size: 0.85rem;
  color: #718096;
  font-weight: 500;
}

.stat-value-large {
  font-size: 1.25rem;
  font-weight: 700;
  color: #1a202c;
  font-variant-numeric: tabular-nums;
}

/* ===== BIAS DISTRIBUTION ===== */
.bias-summary-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.bias-stat-card {
  padding: 1.5rem;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
}

.bias-stat-title {
  font-size: 1rem;
  font-weight: 600;
  color: #1a202c;
  margin: 0 0 1.25rem 0;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid #e2e8f0;
}

.bias-stat-list {
  display: flex;
  flex-direction: column;
  gap: 0.875rem;
}

.bias-stat-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.625rem 0.875rem;
  background: white;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.bias-label {
  font-size: 0.875rem;
  color: #718096;
  font-weight: 500;
}

.bias-value {
  font-size: 1rem;
  font-weight: 600;
  color: #1a202c;
  font-variant-numeric: tabular-nums;
}

.stability-metric {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  padding: 1rem;
  background: white;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.stability-label {
  font-size: 0.875rem;
  color: #718096;
  font-weight: 500;
}

.stability-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: #1a202c;
  font-variant-numeric: tabular-nums;
}

.stability-note {
  font-size: 0.8rem;
  color: #a0aec0;
  font-style: italic;
  margin: 0;
}

/* ===== CORRELATION MATRIX ===== */
.correlation-section {
  margin-top: 2rem;
}

.correlation-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.correlation-matrix-wrapper {
  margin-top: 1rem;
}

.correlation-table {
  font-size: 0.8rem;
}

.matrix-header {
  font-size: 0.75rem;
  padding: 0.625rem 0.5rem !important;
  text-align: center !important;
  vertical-align: middle;
  font-weight: 600;
  background: #f7fafc;
}

.matrix-row-header {
  font-weight: 600;
  background: #f7fafc;
  position: sticky;
  left: 0;
  z-index: 1;
  border-right: 1px solid #e2e8f0;
}

.correlation-cell {
  text-align: center !important;
  font-weight: 600;
  font-variant-numeric: tabular-nums;
  cursor: help;
  transition: all 0.15s ease;
  min-width: 50px;
  padding: 0.5rem !important;
}

.correlation-cell:hover {
  transform: scale(1.1);
  box-shadow: 0 0 0 2px #4299e1;
  z-index: 10;
  position: relative;
}

/* Heatmap Colors */
.heat-very-strong {
  background: #22543d;
  color: white;
}

.heat-strong {
  background: #38a169;
  color: white;
}

.heat-moderate {
  background: #68d391;
  color: #1a202c;
}

.heat-weak {
  background: #c6f6d5;
  color: #1a202c;
}

.heat-neutral {
  background: #f7fafc;
  color: #4a5568;
}

.heat-negative-weak {
  background: #fed7d7;
  color: #1a202c;
}

.heat-negative-moderate {
  background: #fc8181;
  color: white;
}

.heat-negative-strong {
  background: #e53e3e;
  color: white;
}

.heat-negative-very-strong {
  background: #742a2a;
  color: white;
}

/* P-value colors */
.p-significant {
  color: #38a169;
  font-weight: 600;
}

.p-very-significant {
  color: #22543d;
  font-weight: 700;
}

.p-not-significant {
  color: #a0aec0;
}

/* ===== TRANSITIONS ===== */
.expand-enter-active,
.expand-leave-active {
  transition: all 0.3s ease;
  overflow: hidden;
}

.expand-enter-from,
.expand-leave-to {
  opacity: 0;
  max-height: 0;
}

.expand-enter-to,
.expand-leave-from {
  opacity: 1;
  max-height: 2000px;
}

/* ===== RESPONSIVE ===== */
@media (max-width: 1024px) {
  .comparison-grid,
  .extremes-grid,
  .regime-summary,
  .bias-summary-grid {
    grid-template-columns: 1fr;
  }
  
  .detail-columns {
    grid-template-columns: 1fr;
  }
  
  .extreme-stats {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 768px) {
  .metrics-section {
    padding: 1.5rem;
  }
  
  .section-header {
    flex-direction: column;
    align-items: stretch;
    gap: 1rem;
  }
  
  .btn-toggle {
    width: 100%;
    justify-content: center;
  }
  
  .uhi-summary,
  .ndvi-summary {
    grid-template-columns: 1fr;
  }
  
  .regime-comparison {
    grid-template-columns: 1fr;
  }
  
  .regime-divider {
    transform: rotate(90deg);
    padding: 0.5rem;
  }
  
  .data-table {
    font-size: 0.8rem;
  }
  
  .data-table th,
  .data-table td {
    padding: 0.5rem 0.625rem;
  }
  
  .correlation-table {
    font-size: 0.7rem;
  }
}

@media (max-width: 480px) {
  .metrics-section {
    padding: 1rem;
  }
  
  .section-title {
    font-size: 1.25rem;
  }
  
  .metric-card {
    padding: 1rem;
  }
  
  .uhi-stat-value,
  .ndvi-stat-value {
    font-size: 1.25rem;
  }
  
  .extreme-value {
    font-size: 1.5rem;
  }
  
  .extreme-stats {
    grid-template-columns: 1fr;
    gap: 0.5rem;
  }
}
</style>