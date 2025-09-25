<template>
  <div class="weather-component">
    <header class="wc-header">
      <div>
        <h1 class="wc-title">Weather Forecast</h1>
        <p class="wc-subtitle">Latest data from the server</p>
      </div>
      <div class="wc-actions">
        <button class="btn"
                @click="fetchData"
                :disabled="loading"
                aria-label="Refresh forecast"
                title="Refresh">
          <span v-if="!loading" class="btn-icon" aria-hidden="true">⟳</span>
          <span v-else class="spinner" aria-hidden="true"></span>
          <span class="btn-label">{{ loading ? 'Refreshing...' : 'Refresh' }}</span>
        </button>
      </div>
    </header>

    <div v-if="error" class="alert">
      <strong>Couldn’t load data.</strong> {{ error }}
    </div>

    <div v-if="loading && !post" class="skeleton">
      <div class="row" v-for="i in 5" :key="i"></div>
    </div>

    <div v-if="post && post.length" class="content card">
      <table class="table">
        <thead>
          <tr>
            <th>Date</th>
            <th>Temp. (°C)</th>
            <th>Temp. (°F)</th>
            <th>Summary</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="forecast in post" :key="forecast.date">
            <td>{{ formatDate(forecast.date) }}</td>
            <td>{{ forecast.temperatureC }}</td>
            <td>{{ forecast.temperatureF }}</td>
            <td>{{ forecast.summary }}</td>
          </tr>
        </tbody>
      </table>
      <footer class="meta">
        <span>Last updated: {{ lastUpdated ? formatDateTime(lastUpdated) : '—' }}</span>
      </footer>
    </div>

    <div v-else-if="!loading && !error" class="empty card">
      <p>No forecast data available.</p>
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent } from 'vue';

  type Forecast = {
    date: string;
    temperatureC: number;
    temperatureF: number;
    summary: string;
  };

  interface Data {
    loading: boolean;
    post: Forecast[] | null;
    error: string | null;
    lastUpdated: string | null;
  }

  export default defineComponent({
    name: 'WeatherForecast',
    data(): Data {
      return {
        loading: false,
        post: null,
        error: null,
        lastUpdated: null,
      };
    },
    async created() {
      await this.fetchData();
    },
    watch: {
      $route: 'fetchData',
    },
    methods: {
      formatDate(iso: string) {
        const d = new Date(iso);
        return isNaN(d.getTime()) ? iso : d.toLocaleDateString();
      },
      formatDateTime(iso: string) {
        const d = new Date(iso);
        return isNaN(d.getTime()) ? iso : d.toLocaleString();
      },
      async fetchData() {
        this.loading = true;
        this.error = null;

        try {
          const response = await fetch('weatherforecast', { cache: 'no-store' });
          if (!response.ok) {
            throw new Error(`${response.status} ${response.statusText}`);
          }
          const json = (await response.json()) as Forecast[];
          this.post = Array.isArray(json) ? json : [];
          this.lastUpdated = new Date().toISOString();
        } catch (e: any) {
          this.error = e?.message ?? 'Unknown error';
          this.post = null;
        } finally {
          this.loading = false;
        }
      },
    },
  });
</script>

<style scoped>
  :root {
    --bg: #0b0f1a;
    --surface: #121826;
    --surface-2: #0f1522;
    --border: #263042;
    --text: #e6edf7;
    --muted: #9fb0c3;
    --primary: #3b82f6;
    --primary-700: #2563eb;
    --ring: rgba(59, 130, 246, 0.35);
    --shadow: rgba(0, 0, 0, 0.25);
  }

  .weather-component {
    max-width: 960px;
    margin: 2rem auto;
    padding: 0 1rem;
    color: var(--text);
  }

  .wc-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    margin-bottom: 1rem;
  }

  .wc-title {
    margin: 0;
    font-size: 1.5rem;
    letter-spacing: 0.2px;
  }

  .wc-subtitle {
    margin: 0.25rem 0 0;
    color: var(--muted);
    font-size: 0.95rem;
  }

  .wc-actions {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  /* Button */
  .btn {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.55rem 0.9rem;
    border-radius: 0.75rem;
    border: 1px solid var(--border);
    background: linear-gradient(180deg, var(--primary) 0%, var(--primary-700) 100%);
    color: white;
    font-weight: 600;
    cursor: pointer;
    box-shadow: 0 6px 18px var(--ring), 0 2px 6px var(--shadow);
    transition: transform 120ms ease, filter 150ms ease, box-shadow 150ms ease, opacity 150ms ease;
    user-select: none;
  }

    .btn:hover:not(:disabled) {
      transform: translateY(-1px);
      filter: brightness(1.02);
    }

    .btn:active:not(:disabled) {
      transform: translateY(0);
      filter: brightness(0.98);
    }

    .btn:focus-visible {
      outline: none;
      box-shadow: 0 0 0 3px var(--ring);
    }

    .btn:disabled {
      opacity: 0.7;
      cursor: not-allowed;
    }

  .btn-icon {
    font-size: 1rem;
    line-height: 1;
  }

  .btn-label {
    line-height: 1;
  }

  /* Spinner */
  .spinner {
    width: 1rem;
    height: 1rem;
    border: 2px solid rgba(255,255,255,0.45);
    border-top-color: white;
    border-radius: 50%;
    display: inline-block;
    animation: spin 0.9s linear infinite;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  /* Cards & Surfaces */
  .card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 1rem;
    box-shadow: 0 10px 24px var(--shadow);
    overflow: hidden;
  }

  /* Table */
  .table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
  }

    .table thead th {
      text-align: left;
      background: var(--surface-2);
      color: var(--muted);
      font-weight: 700;
      letter-spacing: 0.3px;
      padding: 0.9rem 1rem;
      border-bottom: 1px solid var(--border);
    }

    .table tbody td {
      padding: 0.85rem 1rem;
      border-bottom: 1px solid var(--border);
    }

    .table tbody tr:hover {
      background: rgba(59, 130, 246, 0.06);
    }

  /* Metadata */
  .meta {
    display: flex;
    justify-content: flex-end;
    padding: 0.75rem 1rem;
    color: var(--muted);
    border-top: 1px solid var(--border);
  }

  /* Alerts */
  .alert {
    background: #2a1f1f;
    color: #ffd7d7;
    border: 1px solid #5f2b2b;
    padding: 0.75rem 1rem;
    border-radius: 0.75rem;
    margin-bottom: 1rem;
  }

  /* Empty & Skeleton */
  .empty {
    text-align: center;
    padding: 2rem;
    color: var(--muted);
  }

  .skeleton {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 1rem;
    padding: 0.75rem 1rem;
  }

    .skeleton .row {
      height: 16px;
      margin: 12px 0;
      background: linear-gradient(90deg, #1b2434 0%, #252f42 50%, #1b2434 100%);
      background-size: 200% 100%;
      border-radius: 8px;
      animation: shimmer 1.2s ease-in-out infinite;
    }

  @keyframes shimmer {
    0% {
      background-position: 200% 0;
    }

    100% {
      background-position: -200% 0;
    }
  }

  @media (max-width: 640px) {
    .wc-header {
      flex-direction: column;
      align-items: flex-start;
    }

    .table thead {
      display: none;
    }

    .table, .table tbody, .table tr, .table td {
      display: block;
      width: 100%;
    }

      .table tr {
        border-bottom: 1px solid var(--border);
      }

      .table td {
        border: none;
        padding: 0.6rem 0.8rem;
      }

        .table td::before {
          content: attr(data-label);
          display: block;
          color: var(--muted);
          font-size: 0.8rem;
          margin-bottom: 0.25rem;
        }
  }
</style>
