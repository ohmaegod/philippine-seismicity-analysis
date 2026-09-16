# 🌋 Philippine Seismicity Analysis & Near-Term Risk Scoring (1900–2025)

An end-to-end data science pipeline analyzing over 125 years of Philippine earthquake records. This project merges and reconciles multi-source catalog data (USGS & PHIVOLCS), performs spatial cluster detection using DBSCAN, and constructs a localized, multi-metric near-term seismic risk model across Philippine regions.

---

## 📌 Project Overview

* **Multi-Source Catalog Reconciliation:** Deduplicated and standardized historical earthquake data (1900–2025) combining USGS and PHIVOLCS datasets.
* **Spatial Clustering:** Applied DBSCAN (Density-Based Spatial Clustering of Applications with Noise) to identify high-density seismic active zones across the Philippine archipelago.
* **Near-Term Risk Scoring:** Built a composite regional risk scoring system integrating event frequency, magnitude distributions, energy release rate (Gutenberg-Richter relations), and temporal Recency.
* **Interactive Visualizations:** Mapped spatial cluster density, depth profiles, and regional risk indices.

---

## 📊 Key Features & Analysis Steps

1. **Data Ingestion & Cleaning**
   * Combined historical records from USGS and PHIVOLCS.
   * Standardized timestamp formatting, coordinate systems, depth indicators, and magnitude metrics ($M_w$, $M_s$, $M_l$).
   * Handled duplicate entries using spatial-temporal windowing thresholds.

2. **Exploratory Data Analysis (EDA)**
   * Historical earthquake frequency trends over the last 125 years.
   * Depth vs. Magnitude relationship analysis along major Philippine fault systems (e.g., Philippine Fault Zone, Manila Trench, Philippine Trench).

3. **Spatial Clustering (DBSCAN)**
   * Grouped localized epicenters into distinct seismic active clusters using haversine metric distance thresholds ($\epsilon$) and minimum sample density ($min\_samples$).

4. **Regional Risk Evaluation Modeling**
   * Formulated a normalized Risk Index based on:
     * **Event Frequency:** Historical count of events $M \ge 4.0$.
     * **Max/Mean Magnitude:** Severity weighting of seismic activity.
     * **Energy Release:** Estimated cumulative seismic energy ($E \propto 10^{1.5M}$).
     * **Temporal Recency:** Exponential decay weighting prioritizing recent activity within specific regions.

---

## 🛠️ Tech Stack

* **Language:** Python 3.x
* **Data Processing:** Pandas, NumPy
* **Spatial & Clustering:** SciPy, Scikit-learn (DBSCAN), GeoPandas
* **Visualization:** Matplotlib, Seaborn, Folium / Plotly
* **Environment:** Jupyter Notebook
