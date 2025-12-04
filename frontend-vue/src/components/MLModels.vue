<template>
  <div class="ml-models-container">
    <!-- HEADER -->
    <div class="section">
      <div class="card">
        <div class="card-header">
          <div class="header-text">
            <h2 class="card-title">Machine Learning Models Performance</h2>
            <p class="header-subtitle">
              Advanced bias correction, UHI prediction, downscaling, and uncertainty quantification
            </p>
          </div>
          <button @click="loadAllData" class="btn btn-primary btn-large" :disabled="loading">
            <span v-if="!loading">Refresh Data</span>
            <span v-else>Loading...</span>
          </button>
        </div>
      </div>
    </div>

    <!-- LOADING STATE -->
    <div v-if="loading" class="loading-panel">
      <div class="spinner-large"></div>
      <p>Loading ML models and predictions...</p>
    </div>

    <!-- ERROR STATE -->
    <div v-if="error && !loading" class="error-panel">
      <div class="error-icon">⚠</div>
      <div class="error-content">
        <h3 class="error-title">Failed to Load Models</h3>
        <p class="error-message">{{ error }}</p>
        <button class="btn btn-primary" @click="loadAllData">Retry</button>
      </div>
    </div>

    <!-- MAIN CONTENT -->
    <div v-if="!loading && !error && chartData">
      
      <!-- MODEL PERFORMANCE SUMMARY -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Model Performance Summary</h3>
          <div class="performance-grid">
            
            <!-- XGBoost Bias Correction -->
            <div class="performance-card" v-if="performance.XGBoost_Bias_Correction">
              <div class="perf-header">
                <h4>XGBoost Bias Correction</h4>
                <span class="model-badge">TX Max Temp</span>
              </div>
              <div class="perf-metrics">
                <div class="metric-row">
                  <span class="metric-label">Test RMSE:</span>
                  <span class="metric-value">{{ performance.XGBoost_Bias_Correction.test_rmse?.toFixed(3) || 'N/A' }}°C</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">Test R²:</span>
                  <span class="metric-value">{{ performance.XGBoost_Bias_Correction.test_r2?.toFixed(3) || 'N/A' }}</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">Overfitting Gap:</span>
                  <span class="metric-value" :class="getOverfitClass(performance.XGBoost_Bias_Correction.overfitting_gap)">
                    {{ performance.XGBoost_Bias_Correction.overfitting_gap?.toFixed(3) || 'N/A' }}
                  </span>
                </div>
              </div>
              <div class="perf-params">
                <strong>Best Parameters:</strong>
                <div class="param-tags">
                  <span class="param-tag" v-if="bestParams.XGBoost_Bias_Correction">
                    n_estimators: {{ bestParams.XGBoost_Bias_Correction.params.n_estimators }}
                  </span>
                  <span class="param-tag" v-if="bestParams.XGBoost_Bias_Correction">
                    max_depth: {{ bestParams.XGBoost_Bias_Correction.params.max_depth }}
                  </span>
                  <span class="param-tag" v-if="bestParams.XGBoost_Bias_Correction">
                    lr: {{ bestParams.XGBoost_Bias_Correction.params.learning_rate }}
                  </span>
                </div>
              </div>
            </div>

            <!-- Random Forest UHI -->
            <div class="performance-card" v-if="performance.RandomForest_UHI">
              <div class="perf-header">
                <h4>Random Forest UHI</h4>
                <span class="model-badge">Urban Heat Island</span>
              </div>
              <div class="perf-metrics">
                <div class="metric-row">
                  <span class="metric-label">Test RMSE:</span>
                  <span class="metric-value">{{ performance.RandomForest_UHI.test_rmse?.toFixed(3) || 'N/A' }}°C</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">Test R²:</span>
                  <span class="metric-value">{{ performance.RandomForest_UHI.test_r2?.toFixed(3) || 'N/A' }}</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">CV Std:</span>
                  <span class="metric-value">{{ bestParams.RandomForest_UHI?.cv_std_r2?.toFixed(3) || 'N/A' }}</span>
                </div>
              </div>
              <div class="perf-params">
                <strong>Best Parameters:</strong>
                <div class="param-tags">
                  <span class="param-tag" v-if="bestParams.RandomForest_UHI">
                    n_estimators: {{ bestParams.RandomForest_UHI.params.n_estimators }}
                  </span>
                  <span class="param-tag" v-if="bestParams.RandomForest_UHI">
                    max_depth: {{ bestParams.RandomForest_UHI.params.max_depth }}
                  </span>
                </div>
              </div>
            </div>

            <!-- Ridge Downscaling -->
            <div class="performance-card" v-if="performance.Ridge_Downscaling">
              <div class="perf-header">
                <h4>Ridge Downscaling</h4>
                <span class="model-badge">TN Min Temp</span>
              </div>
              <div class="perf-metrics">
                <div class="metric-row">
                  <span class="metric-label">Model RMSE:</span>
                  <span class="metric-value">{{ performance.Ridge_Downscaling.model_rmse?.toFixed(3) || 'N/A' }}°C</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">Test R²:</span>
                  <span class="metric-value">{{ performance.Ridge_Downscaling.test_r2?.toFixed(3) || 'N/A' }}</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">Improvement:</span>
                  <span class="metric-value success">{{ performance.Ridge_Downscaling.improvement_pct?.toFixed(1) || 'N/A' }}%</span>
                </div>
              </div>
              <div class="perf-params">
                <strong>Best Parameters:</strong>
                <div class="param-tags">
                  <span class="param-tag" v-if="bestParams.Ridge_Downscaling">
                    alpha: {{ bestParams.Ridge_Downscaling.params.alpha }}
                  </span>
                  <span class="param-tag" v-if="bestParams.Ridge_Downscaling">
                    solver: {{ bestParams.Ridge_Downscaling.params.solver }}
                  </span>
                </div>
              </div>
            </div>

            <!-- Quantile Regression -->
            <div class="performance-card" v-if="performance.Quantile_Regression">
              <div class="perf-header">
                <h4>Quantile Regression</h4>
                <span class="model-badge">Uncertainty</span>
              </div>
              <div class="perf-metrics">
                <div class="metric-row">
                  <span class="metric-label">Test RMSE:</span>
                  <span class="metric-value">{{ performance.Quantile_Regression.test_rmse?.toFixed(3) || 'N/A' }}°C</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">80% Coverage:</span>
                  <span class="metric-value">{{ (performance.Quantile_Regression.test_interval_metrics?.coverage_80 * 100)?.toFixed(1) || 'N/A' }}%</span>
                </div>
                <div class="metric-row">
                  <span class="metric-label">50% Coverage:</span>
                  <span class="metric-value">{{ (performance.Quantile_Regression.test_interval_metrics?.coverage_50 * 100)?.toFixed(1) || 'N/A' }}%</span>
                </div>
              </div>
              <div class="perf-params">
                <strong>Best Parameters:</strong>
                <div class="param-tags">
                  <span class="param-tag" v-if="bestParams.Quantile_Regression">
                    n_estimators: {{ bestParams.Quantile_Regression.params.n_estimators }}
                  </span>
                  <span class="param-tag" v-if="bestParams.Quantile_Regression">
                    max_depth: {{ bestParams.Quantile_Regression.params.max_depth }}
                  </span>
                </div>
              </div>
            </div>

          </div>
        </div>
      </div>

      <!-- DATE RANGE FILTER -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Time Series Filters</h3>
          <div class="filter-grid">
            <div class="filter-item">
              <label class="filter-label">Start Date</label>
              <input type="date" v-model="startDate" class="date-input" />
            </div>
            <div class="filter-item">
              <label class="filter-label">End Date</label>
              <input type="date" v-model="endDate" class="date-input" />
            </div>
            <div class="filter-item">
              <button @click="applyDateFilter" class="btn btn-primary">Apply Filter</button>
              <button @click="clearDateFilter" class="btn btn-secondary">Clear</button>
            </div>
          </div>
        </div>
      </div>

      <!-- MODEL VISUALIZATIONS -->
      
      <!-- 1. XGBoost Bias Correction -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Model 1: XGBoost Bias Correction (TX Max Temp)</h3>
          <div ref="biasPlot" class="plot-container"></div>
        </div>
      </div>

      <!-- 2. Random Forest UHI -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Model 2: Random Forest UHI Prediction</h3>
          <div ref="uhiPlot" class="plot-container"></div>
        </div>
      </div>

      <!-- 3. Ridge Downscaling -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Model 3: Ridge Downscaling (TN Min Temp)</h3>
          <div ref="downscalingPlot" class="plot-container"></div>
        </div>
      </div>

      <!-- 4. Quantile Regression -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Model 4: Quantile Regression with Uncertainty</h3>
          <div ref="quantilePlot" class="plot-container"></div>
        </div>
      </div>

      <!-- PERFORMANCE COMPARISON CHARTS -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Model Performance Comparison</h3>
          <div class="comparison-grid">
            <div ref="rmseComparisonPlot" class="plot-container-small"></div>
            <div ref="r2ComparisonPlot" class="plot-container-small"></div>
          </div>
        </div>
      </div>

      <!-- OVERFITTING ANALYSIS -->
      <div class="section">
        <div class="card">
          <h3 class="card-title">Overfitting Analysis</h3>
          <div ref="overfittingPlot" class="plot-container"></div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import { ref, onMounted, nextTick, watch } from 'vue';
import axios from 'axios';
import Plotly from 'plotly.js-dist-min';

const API_BASE = 'http://localhost:5000/api';

export default {
  name: 'MLModelsVisualization',
  props: {
    city: {
      type: String,
      required: true
    }
  },
  setup(props) {
    const loading = ref(false);
    const error = ref(null);
    const chartData = ref(null);
    const performance = ref({});
    const bestParams = ref({});
    const startDate = ref('');
    const endDate = ref('');

    // Plot refs
    const biasPlot = ref(null);
    const uhiPlot = ref(null);
    const downscalingPlot = ref(null);
    const quantilePlot = ref(null);
    const rmseComparisonPlot = ref(null);
    const r2ComparisonPlot = ref(null);
    const overfittingPlot = ref(null);

    const getOverfitClass = (gap) => {
      if (!gap) return '';
      if (gap < 0.15) return 'success';
      if (gap < 0.25) return 'warning';
      return 'danger';
    };

    const loadAllData = async () => {
      try {
        loading.value = true;
        error.value = null;

        console.log('[INFO] Loading ML models data for', props.city);

        // Load comparison chart data (merged predictions + prepared data)
        const chartRes = await axios.get(`${API_BASE}/models/${props.city}/comparison_chart`);
        chartData.value = chartRes.data;
        console.log('[INFO] Chart data loaded:', chartData.value);

        // Load performance metrics
        try {
          const perfRes = await axios.get(`${API_BASE}/models/${props.city}/performance`);
          performance.value = perfRes.data;
          console.log('[INFO] Performance loaded:', performance.value);
        } catch (err) {
          console.warn('[WARN] Performance metrics not available:', err);
        }

        // Load best parameters
        try {
          const paramsRes = await axios.get(`${API_BASE}/models/${props.city}/best_parameters`);
          bestParams.value = paramsRes.data;
          console.log('[INFO] Best params loaded:', bestParams.value);
        } catch (err) {
          console.warn('[WARN] Best parameters not available:', err);
        }

        // Wait for DOM to update
        await nextTick();
        
        // Add small delay to ensure refs are ready
        setTimeout(() => {
          console.log('[INFO] Rendering plots...');
          renderAllPlots();
        }, 100);

      } catch (err) {
        console.error('[ERROR] Error loading ML models:', err);
        error.value = err.response?.data?.error || err.message;
      } finally {
        loading.value = false;
      }
    };

    const applyDateFilter = async () => {
      if (!startDate.value && !endDate.value) return;
      
      try {
        loading.value = true;
        const params = {};
        if (startDate.value) params.start_date = startDate.value;
        if (endDate.value) params.end_date = endDate.value;

        const chartRes = await axios.get(`${API_BASE}/models/${props.city}/comparison_chart`, { params });
        chartData.value = chartRes.data;

        await nextTick();
        setTimeout(() => renderAllPlots(), 100);
      } catch (err) {
        error.value = err.response?.data?.error || err.message;
      } finally {
        loading.value = false;
      }
    };

    const clearDateFilter = () => {
      startDate.value = '';
      endDate.value = '';
      loadAllData();
    };

    const renderAllPlots = () => {
      console.log('[INFO] renderAllPlots called');
      console.log('[INFO] chartData exists:', !!chartData.value);
      console.log('[INFO] biasPlot ref:', biasPlot.value);
      
      if (!chartData.value) {
        console.error('[ERROR] No chart data available');
        return;
      }
      
      // Check if we have data
      if (!chartData.value.dates || chartData.value.dates.length === 0) {
        console.error('[ERROR] No dates in chart data');
        return;
      }
      
      console.log('[INFO] Data points:', chartData.value.dates.length);
      
      renderBiasCorrection();
      renderUHI();
      renderDownscaling();
      renderQuantile();
      
      if (performance.value && Object.keys(performance.value).length > 0) {
        renderRMSEComparison();
        renderR2Comparison();
        renderOverfittingAnalysis();
      }
    };

    const renderBiasCorrection = () => {
      if (!biasPlot.value) {
        console.error('[ERROR] biasPlot ref not available');
        return;
      }
      
      if (!chartData.value.models.bias_correction) {
        console.warn('[WARN] No bias correction data');
        return;
      }

      console.log('[INFO] Rendering bias correction plot');

      const data = chartData.value;
      const bias = data.models.bias_correction;

      const traces = [
        {
          type: 'scatter',
          mode: 'lines',
          name: 'Station Obs (Ground Truth)',
          x: data.dates,
          y: bias.ground_truth,
          line: { color: '#000000', width: 2 }
        },
        {
          type: 'scatter',
          mode: 'lines',
          name: 'Raw ERA5',
          x: data.dates,
          y: bias.raw_era5,
          line: { color: '#95a5a6', width: 1, dash: 'dash' },
          opacity: 0.6
        },
        {
          type: 'scatter',
          mode: 'lines',
          name: 'XGBoost Corrected',
          x: data.dates,
          y: bias.prediction,
          line: { color: '#e74c3c', width: 2 }
        }
      ];

      const layout = {
        xaxis: { title: 'Date' },
        yaxis: { title: 'Temperature (°C)' },
        hovermode: 'x unified',
        showlegend: true,
        legend: { x: 0.02, y: 0.98 },
        margin: { t: 20, b: 50, l: 60, r: 20 },
        autosize: true
      };

      const config = {
        responsive: true,
        displaylogo: false,
        displayModeBar: true
      };

      try {
        Plotly.newPlot(biasPlot.value, traces, layout, config);
        console.log('[SUCCESS] Bias correction plot rendered');
      } catch (err) {
        console.error('[ERROR] Failed to render bias plot:', err);
      }
    };

    const renderUHI = () => {
      if (!uhiPlot.value || !chartData.value.models.uhi) {
        console.warn('[WARN] UHI plot not available');
        return;
      }

      console.log('[INFO] Rendering UHI plot');

      const data = chartData.value;
      const uhi = data.models.uhi;

      if (!uhi.prediction || uhi.prediction.length === 0) {
        console.warn('[WARN] No UHI prediction data');
        return;
      }

      const meanUHI = uhi.prediction.reduce((a, b) => a + b, 0) / uhi.prediction.length;

      const traces = [
        {
          type: 'scatter',
          mode: 'lines',
          name: 'UHI Prediction',
          x: data.dates,
          y: uhi.prediction,
          line: { color: '#3498db', width: 2 }
        },
        {
          type: 'scatter',
          mode: 'lines',
          name: `Mean UHI (${meanUHI.toFixed(2)}°C)`,
          x: [data.dates[0], data.dates[data.dates.length - 1]],
          y: [meanUHI, meanUHI],
          line: { color: '#95a5a6', width: 1, dash: 'dot' }
        }
      ];

      const layout = {
        xaxis: { title: 'Date' },
        yaxis: { title: 'UHI Intensity (°C)' },
        hovermode: 'x unified',
        showlegend: true,
        legend: { x: 0.02, y: 0.98 },
        margin: { t: 20, b: 50, l: 60, r: 20 },
        autosize: true
      };

      Plotly.newPlot(uhiPlot.value, traces, layout, { responsive: true, displaylogo: false });
      console.log('[SUCCESS] UHI plot rendered');
    };

    const renderDownscaling = () => {
      if (!downscalingPlot.value || !chartData.value.models.downscaling) {
        console.warn('[WARN] Downscaling plot not available');
        return;
      }

      console.log('[INFO] Rendering downscaling plot');

      const data = chartData.value;
      const down = data.models.downscaling;

      const traces = [
        {
          type: 'scatter',
          mode: 'lines',
          name: 'Station Obs (Ground Truth)',
          x: data.dates,
          y: down.ground_truth,
          line: { color: '#000000', width: 2 }
        },
        {
          type: 'scatter',
          mode: 'lines',
          name: 'Raw ERA5',
          x: data.dates,
          y: down.raw_era5,
          line: { color: '#95a5a6', width: 1, dash: 'dash' },
          opacity: 0.6
        },
        {
          type: 'scatter',
          mode: 'lines',
          name: 'Ridge Downscaled',
          x: data.dates,
          y: down.prediction,
          line: { color: '#2ecc71', width: 2 }
        }
      ];

      const layout = {
        xaxis: { title: 'Date' },
        yaxis: { title: 'Temperature (°C)' },
        hovermode: 'x unified',
        showlegend: true,
        legend: { x: 0.02, y: 0.98 },
        margin: { t: 20, b: 50, l: 60, r: 20 },
        autosize: true
      };

      Plotly.newPlot(downscalingPlot.value, traces, layout, { responsive: true, displaylogo: false });
      console.log('[SUCCESS] Downscaling plot rendered');
    };

    const renderQuantile = () => {
      if (!quantilePlot.value || !chartData.value.models.quantile) {
        console.warn('[WARN] Quantile plot not available');
        return;
      }

      console.log('[INFO] Rendering quantile plot');

      const data = chartData.value;
      const quant = data.models.quantile;

      const traces = [
        {
          type: 'scatter',
          mode: 'lines',
          name: 'Station Obs (Ground Truth)',
          x: data.dates,
          y: quant.ground_truth,
          line: { color: '#000000', width: 2 }
        },
        {
          type: 'scatter',
          mode: 'lines',
          name: 'Quantile Median (q50)',
          x: data.dates,
          y: quant.median,
          line: { color: '#9b59b6', width: 2 }
        }
      ];

      // Add prediction intervals if available
      if (quant.q90 && quant.q10 && quant.q90.length > 0 && quant.q10.length > 0) {
        traces.push({
          type: 'scatter',
          mode: 'lines',
          name: '80% Interval (q10-q90)',
          x: data.dates.concat(data.dates.slice().reverse()),
          y: quant.q90.concat(quant.q10.slice().reverse()),
          fill: 'toself',
          fillcolor: 'rgba(155, 89, 182, 0.2)',
          line: { color: 'transparent' },
          showlegend: true,
          hoverinfo: 'skip'
        });
      }

      const layout = {
        xaxis: { title: 'Date' },
        yaxis: { title: 'Temperature (°C)' },
        hovermode: 'x unified',
        showlegend: true,
        legend: { x: 0.02, y: 0.98 },
        margin: { t: 20, b: 50, l: 60, r: 20 },
        autosize: true
      };

      Plotly.newPlot(quantilePlot.value, traces, layout, { responsive: true, displaylogo: false });
      console.log('[SUCCESS] Quantile plot rendered');
    };

    const renderRMSEComparison = () => {
      if (!rmseComparisonPlot.value) return;

      console.log('[INFO] Rendering RMSE comparison');

      const models = ['XGBoost\nBias', 'RF\nUHI', 'Ridge\nDownscale', 'Quantile\nRegression'];
      const trainRMSE = [
        performance.value.XGBoost_Bias_Correction?.train_rmse || 0,
        performance.value.RandomForest_UHI?.train_rmse || 0,
        performance.value.Ridge_Downscaling?.train_rmse || 0,
        performance.value.Quantile_Regression?.train_rmse || 0
      ];
      const testRMSE = [
        performance.value.XGBoost_Bias_Correction?.test_rmse || 0,
        performance.value.RandomForest_UHI?.test_rmse || 0,
        performance.value.Ridge_Downscaling?.test_rmse || 0,
        performance.value.Quantile_Regression?.test_rmse || 0
      ];

      const traces = [
        {
          type: 'bar',
          name: 'Train RMSE',
          x: models,
          y: trainRMSE,
          marker: { color: '#3498db' }
        },
        {
          type: 'bar',
          name: 'Test RMSE',
          x: models,
          y: testRMSE,
          marker: { color: '#e74c3c' }
        }
      ];

      const layout = {
        title: 'RMSE Comparison',
        xaxis: { title: '' },
        yaxis: { title: 'RMSE (°C)' },
        barmode: 'group',
        showlegend: true,
        margin: { t: 40, b: 50, l: 60, r: 20 },
        autosize: true
      };

      Plotly.newPlot(rmseComparisonPlot.value, traces, layout, { responsive: true, displaylogo: false });
    };

    const renderR2Comparison = () => {
      if (!r2ComparisonPlot.value) return;

      console.log('[INFO] Rendering R² comparison');

      const models = ['XGBoost\nBias', 'RF\nUHI', 'Ridge\nDownscale', 'Quantile\nRegression'];
      const trainR2 = [
        performance.value.XGBoost_Bias_Correction?.train_r2 || 0,
        performance.value.RandomForest_UHI?.train_r2 || 0,
        performance.value.Ridge_Downscaling?.train_r2 || 0,
        performance.value.Quantile_Regression?.train_r2 || 0
      ];
      const testR2 = [
        performance.value.XGBoost_Bias_Correction?.test_r2 || 0,
        performance.value.RandomForest_UHI?.test_r2 || 0,
        performance.value.Ridge_Downscaling?.test_r2 || 0,
        performance.value.Quantile_Regression?.test_r2 || 0
      ];

      const traces = [
        {
          type: 'bar',
          name: 'Train R²',
          x: models,
          y: trainR2,
          marker: { color: '#2ecc71' }
        },
        {
          type: 'bar',
          name: 'Test R²',
          x: models,
          y: testR2,
          marker: { color: '#f39c12' }
        }
      ];

      const layout = {
        title: 'R² Comparison',
        xaxis: { title: '' },
        yaxis: { title: 'R² Score', range: [0, 1] },
        barmode: 'group',
        showlegend: true,
        margin: { t: 40, b: 50, l: 60, r: 20 },
        autosize: true,
        shapes: [{
          type: 'line',
          x0: -0.5,
          x1: 3.5,
          y0: 0.7,
          y1: 0.7,
          line: { color: 'red', width: 2, dash: 'dash' }
        }]
      };

      Plotly.newPlot(r2ComparisonPlot.value, traces, layout, { responsive: true, displaylogo: false });
    };

    const renderOverfittingAnalysis = () => {
      if (!overfittingPlot.value) return;

      console.log('[INFO] Rendering overfitting analysis');

      const models = ['XGBoost Bias', 'RF UHI', 'Ridge Downscale', 'Quantile Regression'];
      const gaps = [
        performance.value.XGBoost_Bias_Correction?.overfitting_gap || 0,
        (performance.value.RandomForest_UHI?.train_r2 || 0) - (performance.value.RandomForest_UHI?.test_r2 || 0),
        (performance.value.Ridge_Downscaling?.train_r2 || 0) - (performance.value.Ridge_Downscaling?.test_r2 || 0),
        (performance.value.Quantile_Regression?.train_r2 || 0) - (performance.value.Quantile_Regression?.test_r2 || 0)
      ];

      const colors = gaps.map(g => g < 0.15 ? '#2ecc71' : g < 0.25 ? '#f39c12' : '#e74c3c');

      const trace = {
        type: 'bar',
        x: models,
        y: gaps,
        marker: { color: colors },
        text: gaps.map(g => g.toFixed(3)),
        textposition: 'outside'
      };

      const layout = {
        title: 'Overfitting Analysis (Train R² - Test R²)',
        xaxis: { title: '' },
        yaxis: { title: 'Overfitting Gap' },
        showlegend: false,
        margin: { t: 40, b: 80, l: 60, r: 20 },
        autosize: true,
        shapes: [
          {
            type: 'line',
            x0: -0.5,
            x1: 3.5,
            y0: 0.15,
            y1: 0.15,
            line: { color: '#f39c12', width: 2, dash: 'dash' }
          },
          {
            type: 'line',
            x0: -0.5,
            x1: 3.5,
            y0: 0.25,
            y1: 0.25,
            line: { color: '#e74c3c', width: 2, dash: 'dash' }
          }
        ]
      };

      Plotly.newPlot(overfittingPlot.value, [trace], layout, { responsive: true, displaylogo: false });
    };

    onMounted(() => {
      console.log('[INFO] Component mounted, loading data...');
      loadAllData();
    });

    watch(() => props.city, () => {
      console.log('[INFO] City changed, reloading data...');
      loadAllData();
    });

    return {
      loading,
      error,
      chartData,
      performance,
      bestParams,
      startDate,
      endDate,
      biasPlot,
      uhiPlot,
      downscalingPlot,
      quantilePlot,
      rmseComparisonPlot,
      r2ComparisonPlot,
      overfittingPlot,
      loadAllData,
      applyDateFilter,
      clearDateFilter,
      getOverfitClass
    };
  }
};
</script>


<style scoped>
.ml-models-container {
  width: 100%;
}

.header-text {
  flex: 1;
}

.header-subtitle {
  color: var(--color-text-secondary);
  font-size: 0.95rem;
  margin-top: 0.5rem;
}

.btn-large {
  padding: 0.875rem 1.75rem;
  font-size: 1rem;
  white-space: nowrap;
}

/* Performance Grid */
.performance-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.performance-card {
  background: var(--color-bg-tertiary);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 1.5rem;
  transition: all 0.2s ease;
}

.performance-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-hover);
}

.perf-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.75rem;
  border-bottom: 2px solid var(--color-border);
}

.perf-header h4 {
  margin: 0;
  font-size: 1.125rem;
  color: var(--color-text);
}

.model-badge {
  background: var(--color-primary);
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 12px;
  font-size: 0.75rem;
  font-weight: 600;
}

.perf-metrics {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.metric-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.metric-label {
  font-size: 0.875rem;
  color: var(--color-text-secondary);
}

.metric-value {
  font-size: 1rem;
  font-weight: 600;
  color: var(--color-text);
}

.metric-value.success {
  color: #2ecc71;
}

.metric-value.warning {
  color: #f39c12;
}

.metric-value.danger {
  color: #e74c3c;
}

.perf-params {
  padding-top: 1rem;
  border-top: 1px solid var(--color-border);
}

.param-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.param-tag {
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-family: 'Courier New', monospace;
}

/* Filters */
.filter-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.filter-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.filter-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--color-text);
}

.date-input {
  padding: 0.625rem;
  border: 1px solid var(--color-border);
  border-radius: 4px;
  background: var(--color-bg-tertiary);
  color: var(--color-text);
  font-size: 0.875rem;
}

/* Plots */
.plot-container {
  width: 100%;
  height: 500px;
  margin-top: 1rem;
}

.plot-container-small {
  width: 100%;
  height: 400px;
}

.comparison-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  margin-top: 1rem;
}

/* Loading/Error States */
.loading-panel {
  text-align: center;
  padding: 4rem 2rem;
}

.spinner-large {
  width: 60px;
  height: 60px;
  border: 4px solid var(--color-border);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 1rem;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error-panel {
  text-align: center;
  padding: 4rem 2rem;
}

.error-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.error-title {
  font-size: 1.5rem;
  margin-bottom: 0.5rem;
}

.error-message {
  color: var(--color-text-secondary);
  margin-bottom: 2rem;
}

@media (max-width: 768px) {
  .performance-grid {
    grid-template-columns: 1fr;
  }

  .comparison-grid {
    grid-template-columns: 1fr;
  }

  .filter-grid {
    grid-template-columns: 1fr;
  }
}
</style>
