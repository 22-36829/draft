# 📊 Forecasting System Overview 

## 🏗️ System Architecture

### Backend
- **Flask API** (`backend/app.py`) with JWT auth (24h expiry) and 401 handling
- **Forecasting core** (`backend/forecasting_service.py`) using SARIMAX + ensemble
- **PostgreSQL** for historical data, trained models, accuracy history

### Frontend
- **React** (`frontend/src/pages/manager/Forecasting.js`)
- **Chart.js + chartjs-plugin-zoom** for interactive charts
- **Light-only theme** (dark mode removed)

---

## 🔄 End-to-end Flow

### 1) Data → Clean series
```
historical_sales_daily → join product prices/costs → fill missing days →
outlier clipping (IQR) → smoothing (rolling) → stationarity check
```

### 2) Training → Model store
```
best SARIMAX params (proven combos + validation) →
fit model → walk-forward metrics (MAE, RMSE, true Accuracy%) →
save into forecasting_models + accuracy history
```

### 3) Forecasts → UI
```
load historical → fit best params → ensemble predict (with CI) →
serve via API → render in main chart + analyzer
```

---

## 🔧 APIs (key)
- `GET /api/forecasting/predictions` — forecasts for a target (returns values, dates, CI, accuracy)
- `GET /api/forecasting/models` — trained models list (fixed to query DB)
- 401 handling: frontend clears token and redirects to `/login` on expired/invalid JWT

---

## 🎛️ UI/UX Changes and Behavior
- **Light-only theme**: removed dark theme and toggle; unified colors for readability
- **Separate containers** for Forecast and Volume charts; aligned sizing; no extra right spacing
- **Volume chart**: fixed visibility, correct alignment, subtle grid; independent small bar chart
- **Technical analysis section**: reduced spacing; non-overlapping RSI and MACD panels
- **Date labels**: show only dates (no "China Standard Time"); tooltips and ticks use local date formatting
- **Zoom**: drag-to-zoom enabled; wheel/pinch disabled; zoom persists until user clicks Reset (outside fullscreen)
- **Fullscreen**: added for forecast chart; reset button removed inside fullscreen modal; ESC closes
- **Pagination**: product list with First/Prev/Next/Last; resets to page 1 on filter changes
- **Redundant buttons removed**: non-functional analysis tabs/category filters deleted
- **Consistent icons**: dashboard/sections use unified bar chart icon where applicable

---

## 📈 Charts & Indicators
- **Main Forecast chart**: historical sales, SMA trend, revenue/profit overlays, dashed forecast, CI-aware
- **Volume chart**: compact bar chart below main chart (aligned X-axis)
- **RSI & MACD**: dedicated panels with guides (RSI 70/30 bands, MACD zero line)

---

## 🧠 Chart Analyzer (for non-technical users)
- Uses current chart target and selected timeframe
- Explains: current condition, demand level, profit margin, forecast reliability
- Timeframe-aware revenue/profit math using real `avg_daily_sales`, `unit_price`, `cost_price`
- Adds explanations for **Volume**, **RSI**, **MACD** and a combined summary
- Outputs recommendations and immediate action items; updates when timeframe changes

---

## 📊 Accuracy & Reliability
- Removed any artificial boosting/capping; reports true accuracy
- Accuracy computed via walk-forward validation; smoothed, but not inflated
- Backend fix: `/api/forecasting/models` now returns real DB rows, eliminating "Models: 0" issue

---

## 🔐 Auth & Resilience
- JWT lifetime extended to 24h
- Frontend auto-logout on 401 (expired/invalid token)

---

## 🔎 Feature Checklist
- **Forecasting**: SARIMAX + ensemble, confidence intervals, timeframe-aware views
- **Analyzer**: human-friendly insights with Volume/RSI/MACD explanations
- **Charts**: separated forecast/volume, fullscreen, persistent drag-zoom
- **UI**: light-only theme, spacing/alignment fixes, cleaned analysis filters
- **Pagination**: first/last page controls; client-side search/sort
- **APIs**: predictions/models working; accuracy sourced from trained models

---

## 📐 Data Flow (high level)
```
DB (historical + product) ─▶ preprocessing ─▶ param search + fit ─▶
accuracy (walk-forward) ─▶ model store ─▶ predict (ensemble) ─▶ UI charts + analyzer
```

---

## 🗺️ What changed since the previous summary
- Dark theme removed; standardized light theme
- Models endpoint fixed to return real data
- Volume chart visibility/overlap resolved; separate containers and sizing
- Extra line removed; grid/ticks tuned; right padding normalized
- Date labels cleaned (no time zone suffix); tooltips show dates only
- Fullscreen added; reset removed in fullscreen; ESC to close
- Drag-zoom persists until explicit reset (outside fullscreen)
- Analyzer enhanced with Volume/RSI/MACD explanations and timeframe-aware math
- Product list pagination improved with first/last controls; redundant tabs/filters removed

---

This revised overview reflects the light-only UI, fixed APIs, true accuracy reporting, improved charting layout, persistent zoom behavior, enriched analyzer, and streamlined filtering/pagination.
