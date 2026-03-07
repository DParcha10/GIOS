# Large-Scale Environmental Pattern Monitoring with Sentinel-2 + Landsat

This repository is a portfolio-ready framework for building and presenting a geospatial data analysis project that:

1. Pulls **ESA Sentinel-2** and **USGS Landsat** imagery through STAC-compatible APIs.
2. Computes environmental indicators (e.g., NDVI, NDWI, NBR) over large areas and long periods.
3. Produces **time-series geospatial outputs** (maps + statistics).
4. Serves **interactive dashboard experiences** for both technical and executive audiences.

---

## 1) Recommended architecture

```text
           +--------------------+
           |  AOI + Time config |
           +---------+----------+
                     |
                     v
 +-------------------+------------------+
 | STAC Search + Download               |
 | - Sentinel-2 (AWS Earth Search)      |
 | - Landsat Collection 2 (USGS/AWS)    |
 +-------------------+------------------+
                     |
                     v
 +-------------------+------------------+
 | Harmonize + Preprocess               |
 | - Reproject / resample / cloud mask  |
 | - Align spatial grid + metadata      |
 +-------------------+------------------+
                     |
                     v
 +-------------------+------------------+
 | Index + Aggregation                  |
 | - NDVI/NDWI/NBR                      |
 | - Monthly/seasonal composites        |
 | - Pixel and zonal summaries          |
 +-------------------+------------------+
                     |
                     v
 +---------+--------------------+--------+
 | Outputs |                    |         |
 | - COGs  | - Parquet/CSV      | Dashboard
 | - PNGs  | - GeoJSON summaries| + maps
 +---------+--------------------+--------+
```

---

## 2) What is included here

- `src/pipeline.py`: end-to-end ingest + index generation + monthly statistics.
- `src/dashboard.py`: compact operational dashboard (Streamlit + Plotly).
- `src/portfolio_dashboard.py`: full portfolio app with KPIs, trend analysis, seasonality heatmap, and raster gallery.
- `config/project_config.yaml`: central configuration for AOI, date range, and outputs.
- `requirements.txt`: baseline Python dependencies.

This scaffold is intentionally minimal so you can adapt it for your region, policies, and infrastructure.

---

## 3) Setup

### Create environment

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Configure project

Edit `config/project_config.yaml`:
- `aoi_geojson`: path to your AOI polygon.
- `start_date`, `end_date`: time window.
- `cloud_cover_max`: cloud threshold.
- `output_dir`: where outputs are stored.

---

## 4) Run pipeline

```bash
python -m src.pipeline --config config/project_config.yaml
```

Outputs will include:
- per-scene NDVI rasters,
- monthly NDVI composites,
- `monthly_ndvi_summary.csv` for dashboards and external analysis.

---

## 5) Run dashboards

### Operational dashboard
```bash
streamlit run src/dashboard.py
```

### Portfolio dashboard
```bash
streamlit run src/portfolio_dashboard.py
```

Portfolio dashboard provides:
- KPI summary strip for quick storytelling,
- temporal trend chart + rolling trend,
- seasonality heatmap by month/year,
- raster gallery with monthly and scene-level previews,
- downloadable summary table.

---

## 6) Suggested analysis roadmap

1. **Pilot AOI** (single region, 1–2 years) to validate data quality.
2. **Cloud/shadow strategy** tuned for your climate regime.
3. **Harmonization checks** between Sentinel-2 and Landsat reflectance products.
4. **Temporal smoothing** (rolling median, STL decomposition).
5. **Pattern detection**:
   - trend slope by pixel,
   - anomaly maps (z-score by month),
   - abrupt change detection (CUSUM/BFAST-like logic).
6. **Scale out** via chunked AOIs + cloud object storage + workflow orchestration (Prefect/Airflow).

---

## 7) Practical notes for your use case

- For continental-scale analysis, write COG outputs and avoid loading full rasters in memory.
- Keep all outputs in EPSG:4326 or a fixed equal-area CRS depending on metric type.
- If combining sensors, document and test differences in spectral response and revisit frequency.
- Store derived time-series tables in Parquet for fast filtering in dashboards.

---

## 8) Next enhancements

- Add land cover masks (e.g., ESA WorldCover) to stratify trends by biome.
- Add precipitation/temperature covariates (ERA5, CHIRPS) for attribution.
- Add alerting rules for low-vegetation anomalies and burned-area spikes.
- Deploy dashboard with authentication (Streamlit Community Cloud / Docker / internal VM).

If you want, I can also help you create:
- a **production-ready folder structure**,
- **automated monthly runs**,
- and **an executive dashboard layout** tailored to your stakeholders.


See `docs/PORTFOLIO_GUIDE.md` for how to package this into a polished project portfolio.


## 9) Shareable portfolio link (GitHub Pages)

A static portfolio site is included in `site/` and auto-publishes to the `gh-pages` branch using `.github/workflows/deploy-portfolio.yml`.

### Correct shareable URL
```
https://<your-github-username>.github.io/GIOS/
```

### Common mistake
- `https://github.com/<your-github-username>/GIOS` → repository page (not live dashboard)
- `https://<your-github-username>.github.io/GIOS/` → live portfolio dashboard

### Publish steps
1. Push this repo to GitHub.
2. Run the **Deploy Portfolio (gh-pages branch)** workflow once (branch: `main`) to create/update `gh-pages`.
3. In **Settings → Pages**, set **Source = Deploy from a branch**.
4. Select branch **gh-pages** and folder **/(root)**, then save.
5. Open `https://<your-github-username>.github.io/GIOS/` and share it.

To refresh visuals with your latest pipeline data, replace `site/monthly_ndvi_summary.csv` with `outputs/monthly_ndvi_summary.csv`, then commit and push.


### Troubleshooting quick notes
- The **Runners** tab can be empty; use **Actions → Workflows** instead.
- `gh-pages` may not appear in Pages settings until after the first successful run of **Deploy Portfolio (gh-pages branch)**.
