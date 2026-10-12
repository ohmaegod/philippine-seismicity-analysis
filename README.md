# 🌋 Philippine Seismicity Analysis & Near-Term Risk Scoring (1900–2025)

An end-to-end data science pipeline analyzing over 125 years of Philippine earthquake records. This project merges and reconciles multi-source catalog data (USGS & PHIVOLCS), performs spatial cluster detection using DBSCAN, and constructs a localized, multi-metric near-term seismic risk model across Philippine regions.

> **Scope note, stated up front:** this is *not* an attempt to predict the time, location, or magnitude of an individual earthquake. Nobody can currently do that. The well-posed question this answers is: *given a region's recent seismic activity, how likely is it to experience a significant (M≥5) event in the following month?* That is a relative risk score, not a forecast.

**Independent extension of undergraduate thesis.** The underlying thesis — *A Machine Learning Approach to Classifying Earthquake-Prone Regions and Magnitude Occurrences in the Philippines* (Molina & Peralta, Mapúa Institute of Technology, October 2025) — compared five supervised classifiers to produce regional magnitude probability distributions. This repository is my own later work on the same data: it adds spatial clustering, reframes the target as near-term relative risk, and validates against subsequently observed 2026 data. The thesis itself is co-authored and is not published here.

---

## 📊 Results

### Catalog reconciliation

The two sources turned out to be **complementary, not redundant** — deduplication on time *and* space found under 1% overlap. PHIVOLCS's local network catches small events USGS's global network isn't dense enough to register. Cleaning also surfaced that USGS uses 12-hour am/pm time with inconsistent day-first/month-first ordering, leaving **~9,700 genuinely ambiguous rows**, resolved with a documented default rather than silently.

### Spatial clustering

DBSCAN with a **haversine** metric (lat/long are not a flat grid — Euclidean distance would have distorted cluster shapes) and grid-searched `eps`/`min_samples` produced **45 spatial clusters**. Checked against known geology, these track the Mindanao trench system and the eastern Philippine Trench almost exactly. That correspondence is the actual validation — the method recovered structure that matches independently known fault geography.

### Near-term risk model

| Metric | Value |
|---|---|
| ROC-AUC | **≈ 0.78** |
| Base rate (M≥5 in following month) | 32% |
| Precision at default threshold | 0.45 – 0.50 |
| Dominant feature | Recent event frequency (3–12 month windows) |

An AUC of 0.78 against a 32% base rate is a real, modest signal — elevated recent activity correlates with near-term risk, which is consistent with aftershock and swarm behaviour. It is **not** a prediction system: a meaningful share of high-risk flags are not followed by an M≥5 event that month. The honest conclusion is a useful relative-risk signal for prioritising attention.

The number is trustworthy specifically because the train/test split is **time-based, not random**. A random split over monthly data lets neighbouring months leak the future into training and reports a better figure that means nothing.

### Magnitude forecasting — a negative result

Training was cut off at **November 2025** so that December 2025 and January 2026 would be genuinely unseen. Real PHIVOLCS bulletin data for those months was supplied afterwards, purely for evaluation.

| Method | Mean-magnitude MAE |
|---|---|
| Holt's linear trend (alpha, beta grid-searched per region) | 1.59 |
| Naive rolling-average baseline | **0.30** |

**The naive baseline won decisively.** The mechanism is traceable: training data included a period of elevated activity in several regions, Holt's trend term extrapolated that elevation forward, and January reverted toward baseline instead. The baseline, having no trend term to overshoot with, absorbed the same period into a longer average and landed closer to reality.

This result is in the repository rather than omitted from it. Including a persistence baseline at all is what made the failure visible instead of hidden behind a single unquestioned forecast.

---

## 📌 Pipeline

1. **Data ingestion & cleaning** — combined USGS and PHIVOLCS records; standardised timestamps, coordinates, depth, and magnitude scales (Mw, Ms, Ml); deduplicated via spatial-temporal windowing.
2. **Exploratory analysis** — frequency trends across 125 years; depth vs magnitude along the Philippine Fault Zone, Manila Trench, and Philippine Trench.
3. **Spatial clustering** — DBSCAN on haversine distance to identify dense seismic zones.
4. **Risk scoring** — regional index combining event frequency (M≥4.0), max/mean magnitude, estimated energy release, and exponential recency weighting.
5. **Forecasting** — Holt's linear trend exponential smoothing implemented from scratch in NumPy, with per-region parameter grid search, evaluated against a naive persistence baseline.

### One decision worth calling out

The model is restricted to **2015 onward**. The exploratory analysis showed PHIVOLCS reporting was far sparser before roughly 2017, when the network densified. Training across that boundary would teach the model to conflate *"the network wasn't watching"* with *"nothing happened"* — a subtle but serious label-leakage problem. The same boundary explains an apparent drop in average magnitude around 2016–17, which is a reporting-density artifact rather than a change in real seismicity.

---

## ⚠️ Known limitations

1. Date ordering is genuinely ambiguous for ~9,700 USGS rows; resolved with a documented default assumption.
2. Deduplication uses a distance/time heuristic rather than authoritative event IDs — sufficient to demonstrate the approach, not a substitute for real record linkage.
3. Clustering parameters were tuned on a subsample for speed and should be re-run on the full dataset before firm conclusions are drawn.
4. The risk model is validated only on 2015+, where reporting is dense and consistent. It has not been tested on earlier or structurally different data.
5. The forecast comparison rests on a single held-out window (Dec 2025 – Jan 2026). One month is evidence, not proof.

---

## 🛠️ Tech stack

Python 3.x · pandas · NumPy · scikit-learn (DBSCAN) · SciPy · GeoPandas · Matplotlib · Seaborn · Jupyter

---

## ▶️ Reproducing

```bash
pip install -r requirements.txt
jupyter notebook philippine-seismicity-analysis.ipynb
```

Run cells in order. The notebook is self-contained apart from the held-out evaluation data.

### Data

| File | Source |
|---|---|
| `earthquakes_1900-2025_cleaned.csv` | Merged PHIVOLCS + USGS catalogs, 1900–2025 |
| `thefuture.csv` | PHIVOLCS bulletin data, 15 Dec 2025 – 31 Jan 2026 — held out, used only for evaluation |
| `gadm41_PHL_1.shp` | GADM Philippine administrative boundaries |

---

## Author

**Aruen Mae Molina** — BS Data Science, Mapúa Institute of Technology
github.com/ohmaegod
