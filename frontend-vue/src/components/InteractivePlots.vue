
<template>
  <div class="dashboard-container">
    <!-- Control panel -->
    <div class="control-panel">
      <div class="control-group">
        <label>Start Date:</label>
        <input type="date" v-model="startDate" />
      </div>
      <div class="control-group">
        <label>End Date:</label>
        <input type="date" v-model="endDate" />
      </div>
      <button
        class="btn btn-primary"
        @click="applyFilters"
        :disabled="isGlobalLoading"
      >
        {{ isGlobalLoading ? 'Loading...' : 'Apply Date Filter' }}
      </button>
    </div>

    <h2 class="section-title">Interactive Data Exploration</h2>

    <div class="charts-grid">
      <!-- Time Series -->
      <div class="chart-card full-width">
        <div class="card-header">
          <h3>Temperature Time Series</h3>
          <button @click="loadChart('timeseries')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.timeseries" class="spinner"></div>
          <canvas ref="timeseriesCanvas"></canvas>
        </div>
      </div>

      <!-- Bias -->
      <div class="chart-card full-width">
        <div class="card-header">
          <h3>Bias Time Series (ERA5 - TX)</h3>
          <button @click="loadChart('diff_timeseries')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.diff_timeseries" class="spinner"></div>
          <canvas ref="diffCanvas"></canvas>
        </div>
      </div>

      <!-- Scatter TX -->
      <div class="chart-card">
        <div class="card-header">
          <h3>Scatter: ERA5 vs TX</h3>
          <button @click="loadChart('scatter_tx')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.scatter_tx" class="spinner"></div>
          <canvas ref="scatterTX"></canvas>
        </div>
      </div>

      <!-- Scatter TN -->
      <div class="chart-card">
        <div class="card-header">
          <h3>Scatter: ERA5 vs TN</h3>
          <button @click="loadChart('scatter_tn')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.scatter_tn" class="spinner"></div>
          <canvas ref="scatterTN"></canvas>
        </div>
      </div>

      <!-- Bias Histogram - Plotly div -->
      <div class="chart-card">
        <div class="card-header">
          <h3>Bias Distribution</h3>
          <button @click="loadBiasHistogram()" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.bias_histogram" class="spinner"></div>
          <div ref="biasHistogramCanvas" style="width:100%;height:100%;"></div>
        </div>
      </div>

      <!-- NDVI Correlation -->
      <div class="chart-card">
        <div class="card-header">
          <h3>NDVI vs Temperature</h3>
          <button @click="loadNDVICorrelation()" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.ndvi_correlation" class="spinner"></div>
          <canvas ref="ndviCanvas"></canvas>
        </div>
      </div>

      <!-- Seasonal Means -->
      <div class="chart-card">
        <div class="card-header">
          <h3>Seasonal Climate</h3>
          <button @click="loadChart('seasonal_means')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.seasonal_means" class="spinner"></div>
          <canvas ref="seasonalMeansCanvas"></canvas>
        </div>
      </div>

      <!-- Bias vs Quantile -->
      <div class="chart-card">
        <div class="card-header">
          <h3>Bias vs Quantile</h3>
          <button @click="loadChart('bias_quantiles')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.bias_quantiles" class="spinner"></div>
          <canvas ref="biasCanvas"></canvas>
        </div>
      </div>

      <!-- Temperature Regime Bias -->
      <div class="chart-card">
        <div class="card-header">
          <h3>Bias by Temperature Regime</h3>
          <button @click="loadChart('temp_regime_bias')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.temp_regime_bias" class="spinner"></div>
          <canvas ref="tempRegimeCanvas"></canvas>
        </div>
      </div>

      <!-- Correlation Matrix -->
      <div class="chart-card full-width">
        <div class="card-header">
          <h3>Correlation Matrix</h3>
          <button @click="loadChart('correlation_matrix')" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container heatmap">
          <div v-if="loading.correlation_matrix" class="spinner"></div>
          <div ref="heatmapContainer" class="heatmap-box"></div>
        </div>
      </div>



      <!-- UHI Summer Night -->
      <div class="chart-card">
        <div class="card-header">
          <h3>UHI Summer Nights</h3>
          <button @click="loadUHIChart()" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.uhi" class="spinner"></div>
          <canvas ref="uhiCanvas"></canvas>
        </div>
      </div>

      <!-- Temperature Comparison - Plotly div -->
      <div class="chart-card">
        <div class="card-header">
          <h3>ERA5 vs Station Comparison</h3>
          <button @click="loadTempComparison()" class="btn-icon">↻</button>
        </div>
        <div class="canvas-container">
          <div v-if="loading.temp_comparison" class="spinner"></div>
          <div ref="tempComparisonCanvas" style="width:100%;height:100%;"></div>
        </div>
      </div>
    </div>

  </div>
</template>






<script>
import { ref, reactive, nextTick, computed, onMounted, watch } from "vue";
import axios from "axios";
import Plotly from "plotly.js-dist-min";
import { Chart, registerables } from "chart.js";
import 'chartjs-adapter-date-fns';  // ← ADD THIS LINE
Chart.register(...registerables);

const API_BASE = "http://localhost:5000/api";

export default {
  name: "InteractivePlots",
  props: {
    city: String,
  },
  setup(props) {
    const startDate = ref(null);
    const endDate = ref(null);

    // CHART REFS
    const timeseriesCanvas = ref(null);
    const seasonalCanvas = ref(null);
    const biasCanvas = ref(null);
    const heatmapContainer = ref(null);
    const scatterTX = ref(null);
    const scatterTN = ref(null);
    const diffCanvas = ref(null);
    // New chart refs
    const biasHistogramCanvas = ref(null); // Plotly div
    const ndviCanvas = ref(null);
    const seasonalMeansCanvas = ref(null);
    const tempRegimeCanvas = ref(null);
    const heatwaveCanvas = ref(null); // Plotly div
    const uhiCanvas = ref(null);
    const tempComparisonCanvas = ref(null); // Plotly div

    // ML REFS
    const xgboostPlot = ref(null);
    const quantilePlot = ref(null);
    const clusteringPlot = ref(null);
    const uhiPlot = ref(null);
    const shapPlot = ref(null);
    const biasCorrectionPlot = ref(null);

    const chartsData = reactive({});

    const loading = reactive({
      timeseries: false,
      seasonal_uhi: false,
      seasonal_means: false,
      bias_quantiles: false,
      correlation_matrix: false,
      scatter_tx: false,
      scatter_tn: false,
      diff_timeseries: false,
      temp_regime_bias: false,
      bias_histogram: false,
      ndvi_correlation: false,
      heatwave: false,
      uhi: false,
      temp_comparison: false,
    });

    const mlLoading = reactive({
      xgboost: false,
      quantile: false,
      clustering: false,
      uhi_proxy: false,
      shap: false,
      qdm: false,
      variance: false,
      ml: false,
      hybrid: false,
    });

    const mlData = reactive({
      xgboost: null,
      quantile: null,
      clustering: null,
      uhi_proxy: null,
      shap: null,
      bias_correction: null,
    });

    const chartInstances = {
      timeseries: null,
      seasonal: null,
      seasonalMeans: null,
      bias: null,
      scatterTX: null,
      scatterTN: null,
      diff: null,
      biasHistogram: null,
      ndvi: null,
      tempRegime: null,
      heatwave: null,
      uhi: null,
      tempComparison: null,
    };

    const isGlobalLoading = computed(() =>
      Object.values(loading).some((v) => v)
    );

    const getParams = () => {
      const p = {};
      if (startDate.value) p.start = startDate.value;
      if (endDate.value) p.end = endDate.value;
      return p;
    };

    // ---------- FETCH DATA ----------
    const fetchChartData = async (type) => {
      const res = await axios.get(
        `${API_BASE}/data/${props.city}/chart/${type}`,
        { params: getParams() }
      );
      chartsData[type] = res.data;
    };

    const fetchPlotData = async (type) => {
      const res = await axios.get(
        `${API_BASE}/plot/${props.city}/${type}`,
        { params: getParams() }
      );
      chartsData[type] = res.data.plot;
    };

    const loadChart = async (type) => {
      loading[type] = true;
      try {
        await fetchChartData(type);
        await nextTick();
        renderChart(type);
      } catch (e) {
        console.error(`[ERROR] Failed to load chart ${type}:`, e);
      } finally {
        loading[type] = false;
      }
    };

    const loadPlot = async (type) => {
      loading[type] = true;
      try {
        await fetchPlotData(type);
        await nextTick();
        renderChart(type);
      } catch (e) {
        console.error(`[ERROR] Failed to load plot ${type}:`, e);
      } finally {
        loading[type] = false;
      }
    };

    
    const loadAllCharts = async () => {
  await nextTick();
  
  await Promise.all([
    loadChart("timeseries"),
    loadChart("diff_timeseries"),
    loadChart("scatter_tx"),
    loadChart("scatter_tn"),
    loadChart("bias_quantiles"),
    loadChart("correlation_matrix"),
    loadChart("temp_regime_bias"),
    loadChart("seasonal_means"),
  ]);

  await Promise.all([
    loadBiasHistogram(),
    loadNDVICorrelation().catch(err => {
      console.warn("NDVI data not available for this city");
    }),
    loadHeatwaveChart(),
    loadUHIChart().catch(err => {
      console.warn("UHI data not available for this city");
    }),
    loadTempComparison(),
  ]);
};


    onMounted(() => {
      if (props.city) {
        loadAllCharts();
      }
    });

    watch(
      () => props.city,
      async () => {
        // Clear data
        Object.keys(chartsData).forEach((k) => delete chartsData[k]);

        // Destroy Chart.js instances safely
        Object.entries(chartInstances).forEach(([key, instance]) => {
          if (instance && typeof instance.destroy === "function") {
            instance.destroy();
            chartInstances[key] = null;
          }
        });

        await nextTick();

        if (props.city) {
          loadAllCharts();
        }
      }
    );

    const applyFilters = () => loadAllCharts();

    // ======================================================
    //     ML MODEL RUNNER
    // ======================================================
    const runML = async (modelType) => {
      try {
        mlLoading[modelType] = true;
        mlData[modelType] = null;
        const res = await axios.get(
          `${API_BASE}/analyze/${props.city}/${modelType}`
        );
        mlData[modelType] = res.data;
        mlLoading[modelType] = false;
        await nextTick();
        const cfg = { responsive: true, displaylogo: false };
        if (modelType === "xgboost" && xgboostPlot.value)
          Plotly.newPlot(
            xgboostPlot.value,
            res.data.plot.data,
            res.data.plot.layout,
            cfg
          );
        if (modelType === "quantile" && quantilePlot.value)
          Plotly.newPlot(
            quantilePlot.value,
            res.data.plot.data,
            res.data.plot.layout,
            cfg
          );
        if (modelType === "clustering" && clusteringPlot.value)
          Plotly.newPlot(
            clusteringPlot.value,
            res.data.plot.data,
            res.data.plot.layout,
            cfg
          );
        if (modelType === "uhi_proxy" && uhiPlot.value)
          Plotly.newPlot(
            uhiPlot.value,
            res.data.plot.data,
            res.data.plot.layout,
            cfg
          );
        if (modelType === "shap" && shapPlot.value)
          Plotly.newPlot(
            shapPlot.value,
            res.data.plot.data,
            res.data.plot.layout,
            cfg
          );
      } catch (err) {
        console.error(err);
        mlLoading[modelType] = false;
      }
    };

    const runBiasCorrection = async (modelType) => {
      try {
        mlLoading[modelType] = true;
        mlData.bias_correction = null;
        let endpoint;
        switch (modelType) {
          case "qdm":
            endpoint = "bias_correction_qdm";
            break;
          case "variance":
            endpoint = "bias_correction_variance";
            break;
          case "ml":
            endpoint = "bias_correction_ml";
            break;
          case "hybrid":
            endpoint = "bias_correction_hybrid";
            break;
        }
        const res = await axios.get(
          `${API_BASE}/analyze/${props.city}/${endpoint}`
        );
        mlData.bias_correction = res.data;
        mlLoading[modelType] = false;
        await nextTick();
        const cfg = { responsive: true, displaylogo: false };
        if (biasCorrectionPlot.value) {
          Plotly.newPlot(
            biasCorrectionPlot.value,
            res.data.plot.data,
            res.data.plot.layout,
            cfg
          );
        }
      } catch (err) {
        console.error(err);
        mlLoading[modelType] = false;
      }
    };

    // ======================================================
    //       CHART RENDERERS
    // ======================================================
    const renderChart = (type) => {
      switch (type) {
        case "timeseries":
          createTimeSeries();
          break;
        case "seasonal_uhi":
          createSeasonal();
          break;
        case "seasonal_means":
          createSeasonalMeans();
          break;
        case "bias_quantiles":
          createBias();
          break;
        case "correlation_matrix":
          createHeatmap();
          break;
        case "scatter_tx":
          createScatterTX();
          break;
        case "scatter_tn":
          createScatterTN();
          break;
        case "diff_timeseries":
          createDiff();
          break;
        case "temp_regime_bias":
          createTempRegime();
          break;
      }
    };

    const ensureCanvasSize = (canvasRef) => {
      if (!canvasRef?.value) return null;
      const canvas = canvasRef.value;
      const parent = canvas.parentElement;
      if (parent) {
        canvas.width = parent.clientWidth;
        canvas.height = 320;
      }
      return canvas;
    };

    const createTimeSeries = () => {
      const d = chartsData.timeseries;
      if (!d || !timeseriesCanvas.value) return;
      if (chartInstances.timeseries) chartInstances.timeseries.destroy();

      const canvas = ensureCanvasSize(timeseriesCanvas);
      const datasets = [];

      if (d.tx && d.tx.length) {
        datasets.push({
          label: "Station TX",
          data: d.tx,
          borderColor: "#E74C3C",
          backgroundColor: "rgba(231, 76, 60, 0.1)",
          borderWidth: 2,
          pointRadius: 0,
          fill: false,
        });
      }

      if (d.era5 && d.era5.length) {
        datasets.push({
          label: "ERA5",
          data: d.era5,
          borderColor: "#3498DB",
          backgroundColor: "rgba(52, 152, 219, 0.1)",
          borderWidth: 2,
          pointRadius: 0,
          fill: false,
        });
      }

      chartInstances.timeseries = new Chart(canvas, {
        type: "line",
        data: { labels: d.dates, datasets },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: { position: "top" },
          },
          scales: {
            x: {
              type: "time",
              time: { unit: "month" },
            },
            y: {
              title: { display: true, text: "Temperature (°C)" },
            },
          },
        },
      });
    };

    const createDiff = () => {
      const d = chartsData.diff_timeseries;
      if (!d || !diffCanvas.value) return;
      if (chartInstances.diff) chartInstances.diff.destroy();

      const canvas = ensureCanvasSize(diffCanvas);

      chartInstances.diff = new Chart(canvas, {
        type: "line",
        data: {
          labels: d.dates,
          datasets: [
            {
              label: "Bias (ERA5 - TX)",
              data: d.diff,
              borderColor: "#9B59B6",
              backgroundColor: "rgba(155, 89, 182, 0.2)",
              borderWidth: 2,
              pointRadius: 0,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: { position: "top" },
          },
          scales: {
            x: {
              type: "time",
              time: { unit: "month" },
            },
            y: {
              title: { display: true, text: "Bias (°C)" },
            },
          },
        },
      });
    };

    const createScatterTX = () => {
      const d = chartsData.scatter_tx;
      if (!d || !scatterTX.value) return;
      if (chartInstances.scatterTX) chartInstances.scatterTX.destroy();

      const canvas = ensureCanvasSize(scatterTX);

      chartInstances.scatterTX = new Chart(canvas, {
        type: "scatter",
        data: {
          datasets: [
            {
              label: "ERA5 vs TX",
              data: d.tx.map((x, i) => ({ x, y: d.era5[i] })),
              backgroundColor: "rgba(231, 76, 60, 0.6)",
              pointRadius: 3,
              pointHoverRadius: 5,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: { position: "top" },
          },
          scales: {
            x: {
              title: { display: true, text: "Station TX (°C)" },
            },
            y: {
              title: { display: true, text: "ERA5 (°C)" },
            },
          },
        },
      });
    };

    const createScatterTN = () => {
      const d = chartsData.scatter_tn;
      if (!d || !scatterTN.value) return;
      if (chartInstances.scatterTN) chartInstances.scatterTN.destroy();

      const canvas = ensureCanvasSize(scatterTN);

      chartInstances.scatterTN = new Chart(canvas, {
        type: "scatter",
        data: {
          datasets: [
            {
              label: "ERA5 vs TN",
              data: d.tn.map((x, i) => ({ x, y: d.era5[i] })),
              backgroundColor: "rgba(46, 204, 113, 0.6)",
              pointRadius: 3,
              pointHoverRadius: 5,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: { position: "top" },
          },
          scales: {
            x: {
              title: { display: true, text: "Station TN (°C)" },
            },
            y: {
              title: { display: true, text: "ERA5 (°C)" },
            },
          },
        },
      });
    };

    const createSeasonal = () => {
      const d = chartsData.seasonal_uhi;
      if (!d || !seasonalCanvas.value) return;
      if (chartInstances.seasonal) chartInstances.seasonal.destroy();

      const canvas = ensureCanvasSize(seasonalCanvas);

      chartInstances.seasonal = new Chart(canvas, {
        type: "bar",
        data: {
          labels: d.labels || ["Winter", "Spring", "Summer", "Autumn"],
          datasets: [
            {
              label: "Mean UHI",
              data: d.mean || [0, 0, 0, 0],
              backgroundColor: "#3498DB",
            },
            {
              label: "P90",
              data: d.p90 || [0, 0, 0, 0],
              backgroundColor: "#9B59B6",
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              beginAtZero: true,
              title: { display: true, text: "UHI Intensity (°C)" },
            },
          },
        },
      });
    };

    const createSeasonalMeans = () => {
      const d = chartsData.seasonal_means;
      if (!d || !seasonalMeansCanvas.value) return;
      if (chartInstances.seasonalMeans) chartInstances.seasonalMeans.destroy();

      const canvas = ensureCanvasSize(seasonalMeansCanvas);

      const labels = d.seasons || ["winter", "spring", "summer", "autumn"];
      const temps = d.temperatures || [];
      const precip = d.precipitation || [];

      chartInstances.seasonalMeans = new Chart(canvas, {
        type: "bar",
        data: {
          labels: labels.map((s) => s.charAt(0).toUpperCase() + s.slice(1)),
          datasets: [
            {
              label: "Mean Temperature",
              data: temps,
              backgroundColor: "#E74C3C",
              yAxisID: "y",
            },
            {
              label: "Precipitation",
              data: precip,
              backgroundColor: "#3498DB",
              yAxisID: "y1",
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              type: "linear",
              position: "left",
              title: { display: true, text: "Temperature (°C)" },
            },
            y1: {
              type: "linear",
              position: "right",
              title: { display: true, text: "Precipitation (mm)" },
              grid: { drawOnChartArea: false },
            },
          },
          plugins: {
            legend: { position: "top" },
          },
        },
      });
    };

    const createBias = () => {
      const d = chartsData.bias_quantiles;
      if (!d || !biasCanvas.value) return;
      if (chartInstances.bias) chartInstances.bias.destroy();

      const canvas = ensureCanvasSize(biasCanvas);

      chartInstances.bias = new Chart(canvas, {
        type: "line",
        data: {
          labels: d.percentiles,
          datasets: [
            {
              label: "Mean Bias",
              data: d.bias,
              borderColor: "#E67E22",
              backgroundColor: "rgba(230, 126, 34, 0.1)",
              borderWidth: 3,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              title: { display: true, text: "TX Percentile (%)" },
            },
            y: {
              title: { display: true, text: "Mean Bias (°C)" },
            },
          },
        },
      });
    };

    const createTempRegime = () => {
      const d = chartsData.temp_regime_bias;
      if (!d || !tempRegimeCanvas.value) return;
      if (chartInstances.tempRegime) chartInstances.tempRegime.destroy();

      const canvas = ensureCanvasSize(tempRegimeCanvas);

      const labels = d.labels || [];
      const biases = d.biases || [];
      const rmses = d.rmses || [];

      chartInstances.tempRegime = new Chart(canvas, {
        type: "bar",
        data: {
          labels,
          datasets: [
            {
              label: "Mean Bias",
              data: biases,
              backgroundColor: "#3498DB",
            },
            {
              label: "RMSE",
              data: rmses,
              backgroundColor: "#E74C3C",
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              beginAtZero: true,
              title: { display: true, text: "°C" },
            },
          },
          plugins: {
            legend: { position: "top" },
          },
        },
      });
    };

    const createHeatmap = () => {
      const d = chartsData.correlation_matrix;
      if (!d || !heatmapContainer.value) return;

      const labels = Object.keys(d);
      const z = labels.map((r) => labels.map((c) => d[r][c] || 0));

      Plotly.newPlot(
        heatmapContainer.value,
        [
          {
            type: "heatmap",
            x: labels,
            y: labels,
            z,
            colorscale: "RdBu",
            zmin: -1,
            zmax: 1,
            hoverongaps: false,
            colorbar: { title: "Correlation" },
          },
        ],
        {
          title: "Variable Correlation Matrix",
          margin: { t: 40, b: 40, l: 100, r: 40 },
          height: 500,
        }
      );
    };

    // ======================================================
    //       ADDITIONAL PLOTS
    // ======================================================
    const loadBiasHistogram = async () => {
      loading.bias_histogram = true;
      try {
        await fetchPlotData("bias_histogram");
        await nextTick();
        createBiasHistogram();
      } catch (e) {
        console.error(e);
      } finally {
        loading.bias_histogram = false;
      }
    };

    const createBiasHistogram = () => {
      const plot = chartsData.bias_histogram;
      if (!plot || !biasHistogramCanvas.value) return;

      Plotly.newPlot(
        biasHistogramCanvas.value,
        plot.data,
        plot.layout,
        { responsive: true, displaylogo: false }
      );
    };

    const loadNDVICorrelation = async () => {
      loading.ndvi_correlation = true;
      try {
        await fetchPlotData("ndvi_correlation");
        await nextTick();
        createNDVICorrelation();
      } catch (e) {
        console.error(e);
      } finally {
        loading.ndvi_correlation = false;
      }
    };

    const createNDVICorrelation = () => {
      const d = chartsData.ndvi_correlation;
      if (!d || !ndviCanvas.value) return;
      if (chartInstances.ndvi) chartInstances.ndvi.destroy();

      const canvas = ensureCanvasSize(ndviCanvas);

      const scatterData = d.data[0];
      const trendData = d.data[1];

      chartInstances.ndvi = new Chart(canvas, {
        type: "scatter",
        data: {
          datasets: [
            {
              label: scatterData.name,
              data: scatterData.x.map((x, i) => ({ x, y: scatterData.y[i] })),
              backgroundColor: "rgba(46, 204, 113, 0.6)",
              pointRadius: 3,
            },
            {
              label: trendData.name,
              data: trendData.x.map((x, i) => ({ x, y: trendData.y[i] })),
              borderColor: "#E74C3C",
              backgroundColor: "transparent",
              type: "line",
              borderWidth: 2,
              pointRadius: 0,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              title: { display: true, text: "NDVI" },
            },
            y: {
              title: { display: true, text: "Temperature (°C)" },
            },
          },
        },
      });
    };

    const loadHeatwaveChart = async () => {
      loading.heatwave = true;
      try {
        await fetchPlotData("heatwaves");
        await nextTick();
        createHeatwaveChart();
      } catch (e) {
        console.error(e);
      } finally {
        loading.heatwave = false;
      }
    };

  


    const createHeatwaveChart = () => {
  const plot = chartsData.heatwaves;
  if (!plot || !heatwaveCanvas.value) {
    console.error("Missing plot data or canvas element for heatwave chart");
    return;
  }

  console.log("Heatwave plot data:", plot); // Debug log

  try {
    Plotly.newPlot(
      heatwaveCanvas.value,
      plot.data,
      plot.layout,
      { 
        responsive: true, 
        displaylogo: false,
        modeBarButtonsToRemove: ['sendDataToCloud']
      }
    );
  } catch (error) {
    console.error("Error creating heatwave chart:", error);
  }
};


    const loadUHIChart = async () => {
      loading.uhi = true;
      try {
        await fetchPlotData("seasonal_uhi");
        await nextTick();
        createUHIChart();
      } catch (e) {
        console.error(e);
      } finally {
        loading.uhi = false;
      }
    };

    const createUHIChart = () => {
      const d = chartsData.seasonal_uhi;
      if (!d || !uhiCanvas.value) return;
      if (chartInstances.uhi) chartInstances.uhi.destroy();

      const canvas = ensureCanvasSize(uhiCanvas);

      chartInstances.uhi = new Chart(canvas, {
        type: "bar",
        data: {
          labels: ["Winter", "Spring", "Summer", "Autumn"],
          datasets: [
            {
              label: "Mean UHI",
              data: [2.1, 2.3, 2.5, 2.2], // placeholder until backend provides real values
              backgroundColor: "#3498DB",
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              beginAtZero: true,
              title: { display: true, text: "UHI Intensity (°C)" },
            },
          },
        },
      });
    };

    const loadTempComparison = async () => {
      loading.temp_comparison = true;
      try {
        await fetchPlotData("temp_comparison");
        await nextTick();
        createTempComparison();
      } catch (e) {
        console.error(e);
      } finally {
        loading.temp_comparison = false;
      }
    };

    const createTempComparison = () => {
      const plot = chartsData.temp_comparison;
      if (!plot || !tempComparisonCanvas.value) return;

      Plotly.newPlot(
        tempComparisonCanvas.value,
        plot.data,
        plot.layout,
        { responsive: true, displaylogo: false }
      );
    };

    return {
      startDate,
      endDate,
      applyFilters,
      isGlobalLoading,
      loading,
      chartsData,
      timeseriesCanvas,
      seasonalCanvas,
      biasCanvas,
      heatmapContainer,
      scatterTX,
      scatterTN,
      diffCanvas,
      biasHistogramCanvas,
      ndviCanvas,
      seasonalMeansCanvas,
      tempRegimeCanvas,
      heatwaveCanvas,
      uhiCanvas,
      tempComparisonCanvas,
      runML,
      runBiasCorrection,
      mlLoading,
      mlData,
      xgboostPlot,
      quantilePlot,
      clusteringPlot,
      uhiPlot,
      shapPlot,
      biasCorrectionPlot,
      loadChart,
      loadBiasHistogram,
      loadNDVICorrelation,
      loadHeatwaveChart,
      loadUHIChart,
      loadTempComparison,
    };
  },
};
</script>


<style scoped>
/* ------------------------------
   GLOBAL CONTAINER
------------------------------ */
.dashboard-container {
  max-width: 1500px;
  margin: 0 auto;
  padding: 24px;
  font-family: 'Segoe UI', sans-serif;
  color: var(--color-text);
  transition: background 0.25s ease, color 0.25s ease;
}













.canvas-container canvas {
  width: 100% !important;
  height: 320px !important;
  display: block !important;
}

/* For Plotly divs */
.canvas-container > div[ref] {
  min-height: 320px !important;
}



/* ------------------------------
   CONTROL PANEL
------------------------------ */
.control-panel {
  background: var(--color-bg-secondary);
  padding: 18px;
  border-radius: 10px;
  display: flex;
  gap: 24px;
  align-items: flex-end;
  border: 1px solid var(--color-border);
  margin-bottom: 32px;
  flex-wrap: wrap;
}

.control-group label {
  display: block;
  font-size: 0.9em;
  font-weight: 600;
  margin-bottom: 6px;
  color: var(--color-text);
}

.control-group input {
  padding: 9px;
  border: 1px solid var(--color-border);
  border-radius: 5px;
  background: var(--color-bg-tertiary);
  color: var(--color-text);
  width: 180px;
  transition: 0.2s ease;
}

.control-group input:focus {
  outline: none;
  border-color: var(--color-primary);
  background: var(--color-bg);
}

/* ------------------------------
   SECTION TITLES
------------------------------ */
.section-title {
  font-size: 1.5em;
  color: var(--color-text);
  border-bottom: 2px solid var(--color-border);
  padding-bottom: 10px;
  margin-bottom: 18px;
  margin-top: 5px;
}

.section-subtitle {
  color: var(--color-text-secondary);
  margin-top: -10px;
  margin-bottom: 25px;
}

/* ------------------------------
   CHART GRID
------------------------------ */
.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(520px, 1fr));
  gap: 22px;
}

.chart-card {
  background: var(--color-bg-secondary);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  padding: 18px;
  box-shadow: var(--shadow-card);
  transition: 0.2s ease;
}

.chart-card:hover {
  border-color: var(--color-border-secondary);
  transform: translateY(-1px);
  box-shadow: var(--shadow-hover);
}

.full-width {
  grid-column: 1 / -1;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  padding-bottom: 5px;
  border-bottom: 1px solid var(--color-border);
}

.card-header h3 {
  margin: 0;
  font-size: 1.1em;
  color: var(--color-text);
  font-weight: 600;
}

.canvas-container {
  position: relative;
  height: 320px;
  width: 100%;
}

.heatmap-box {
  width: 100%;
  height: 100%;
  min-height: 400px;
}

/* ------------------------------
   BUTTONS
------------------------------ */
.btn {
  padding: 8px 18px;
  border: 1px solid var(--color-border);
  border-radius: 5px;
  cursor: pointer;
  background: var(--color-bg-secondary);
  color: var(--color-text);
  transition: 0.2s ease;
  font-weight: 500;
}

.btn:hover:not(:disabled) {
  border-color: var(--color-border-secondary);
  background: var(--color-bg-tertiary);
  transform: translateY(-1px);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-primary {
  background: var(--color-primary);
  color: var(--color-bg);
  border-color: var(--color-primary);
}

.btn-primary:hover:not(:disabled) {
  background: var(--color-primary-hover);
  border-color: var(--color-primary-hover);
}

.btn-run {
  background: var(--color-success);
  color: var(--color-bg);
  padding: 6px 14px;
  border-radius: 5px;
  font-size: 0.9em;
  transition: 0.2s ease;
  border: none;
  font-weight: 500;
}

.btn-run:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-1px);
}

.btn-run-small {
  background: var(--color-info);
  color: var(--color-bg);
  padding: 4px 10px;
  border-radius: 4px;
  font-size: 0.8em;
  transition: 0.2s ease;
  border: none;
  margin-right: 5px;
}

.btn-run-small:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-1px);
}

.model-buttons {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.btn-icon {
  background: none;
  border: none;
  font-size: 1.35em;
  cursor: pointer;
  color: var(--color-text-secondary);
  transition: 0.2s ease;
  padding: 4px;
  border-radius: 4px;
}

.btn-icon:hover {
  color: var(--color-primary);
  background: var(--color-bg-tertiary);
}

/* ------------------------------
   SPINNER
------------------------------ */
.spinner {
  position: absolute;
  top: 45%;
  left: 48%;
  width: 32px;
  height: 32px;
  border: 3px solid var(--color-border);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* ------------------------------
   ML GRID
------------------------------ */
.ml-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(420px, 1fr));
  gap: 28px;
  margin-top: 20px;
}

.ml-card {
  background: var(--color-bg-secondary);
  border: 1px solid var(--color-border);
  border-radius: 10px;
  box-shadow: var(--shadow-card);
  overflow: hidden;
  transition: 0.2s ease;
}

.ml-card:hover {
  border-color: var(--color-border-secondary);
  transform: translateY(-1px);
}

.ml-body {
  padding: 18px;
  background: var(--color-bg-tertiary);
  min-height: 250px;
}

.ml-plot-box {
  width: 100%;
  height: 360px !important;
  min-height: 360px !important;
  border: 1px solid var(--color-border);
  border-radius: 6px;
  background: var(--color-bg);
  margin-bottom: 14px;
}

/* ------------------------------
   STATS
------------------------------ */
.stats-panel {
  width: 100%;
  background: var(--color-bg-secondary);
  color: var(--color-info);
  padding: 10px;
  border-radius: 6px;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 0.85em;
  border: 1px solid var(--color-border);
  max-height: 120px;
  overflow-y: auto;
}

.stat-line {
  margin-bottom: 4px;
  color: var(--color-text-secondary);
  line-height: 1.4;
}

.placeholder-text {
  color: var(--color-text-tertiary);
  text-align: center;
  margin-top: 40px;
  font-style: italic;
  padding: 20px;
}

.loader-box {
  text-align: center;
  margin-top: 50px;
  font-weight: bold;
  color: var(--color-info);
  padding: 20px;
}

/* ------------------------------
   RESPONSIVE FIXES
------------------------------ */
@media (max-width: 1024px) {
  .charts-grid {
    grid-template-columns: 1fr;
  }
  .ml-grid {
    grid-template-columns: 1fr;
  }
  .control-panel {
    flex-direction: column;
    align-items: stretch;
  }
  .control-group input {
    width: 100%;
  }
}

@media (max-width: 768px) {
  .dashboard-container {
    padding: 16px;
  }
  
  .charts-grid {
    gap: 16px;
  }
  
  .chart-card {
    padding: 14px;
  }
  
  .canvas-container {
    height: 280px;
  }
  
  .ml-plot-box {
    height: 300px !important;
  }
}

@media (max-width: 480px) {
  .section-title {
    font-size: 1.3em;
  }
  
  .card-header h3 {
    font-size: 1em;
  }
  
  .ml-card.full-width {
    margin-left: -8px;
    margin-right: -8px;
    border-radius: 0;
  }
}
</style>