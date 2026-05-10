# Crime Hotspot Clustering: Geospatial Analysis of Chicago Crime Data

> Identifying spatial and spatio-temporal crime hotspots using HDBSCAN, ST-DBSCAN, and KDE on 100,000 live Chicago PD records — with a PAI@5% of 16.23%.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3%2B-orange)
![hdbscan](https://img.shields.io/badge/hdbscan-0.8.33%2B-blue)
![folium](https://img.shields.io/badge/folium-0.14%2B-green)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange)

---

## Overview

This project applies three density-based clustering algorithms to real-time Chicago Police
Department incident data to detect spatial and spatio-temporal crime hotspots across the
city. Rather than working from a static CSV, all data is fetched live at runtime from the
City of Chicago Open Data Portal (Socrata API), so results reflect current conditions.

The core of the work lies in comparing two fundamentally different clustering strategies:
HDBSCAN, which treats crime geography as a manifold and extracts clusters of arbitrary shape
and variable density without requiring a pre-specified count; and a spatio-temporal DBSCAN
that encodes time as a calibrated third axis alongside latitude and longitude, allowing the
algorithm to find incidents that are both geographically proximate and temporally
co-occurring. Kernel Density Estimation provides a continuous risk surface that bridges the
two, and its output drives the final predictive evaluation.

What makes this implementation distinctive is the ST-DBSCAN feature engineering: rather than
relying on the `st-dbscan` library (which has dependency conflicts), the temporal dimension
is encoded directly into the feature matrix with a scaling factor chosen so that three days
of elapsed time equals one standard deviation of spatial distance. This first-principles
approach makes the underlying mechanics of spatio-temporal clustering explicit. The KDE
model is then evaluated using the Predictive Accuracy Index (PAI@5%), a metric from
predictive policing research, which yielded 16.23% of future crimes captured in the top 5%
of predicted risk zones — more than three times random chance.

---

## What This Project Demonstrates

- Density-based spatial and spatio-temporal clustering on real geographic coordinate data
- Spatio-temporal feature engineering from first principles (calibrated space-time distance scaling)
- Kernel Density Estimation as a continuous risk surface estimator
- Predictive evaluation using the PAI (Predictive Accuracy Index) metric
- Live API data ingestion and preprocessing (Socrata REST, pandas)
- Interactive geospatial visualization with Folium
- Comparison of three methodologically distinct density estimators side-by-side

---

## Methods Used

| Method | Type | Use Case |
|---|---|---|
| HDBSCAN | Hierarchical density-based clustering | Spatial crime cluster discovery, noise-tolerant |
| ST-DBSCAN (approximated) | Spatio-temporal density-based clustering | Clusters events proximate in both space and time |
| KDE (Gaussian) | Non-parametric density estimation | Continuous risk surface; drives PAI evaluation |
| Folium HeatMap | Interactive visualization | Web-renderable crime density overlay |

---

## Datasets / Inputs

**Source:** City of Chicago Open Data Portal — "Crimes - 2001 to Present"

| Property | Value |
|---|---|
| API endpoint | `https://data.cityofchicago.org/resource/ijzp-q8t2.csv` |
| Records fetched | 100,000 (most recent, via `$limit=100000`) |
| After 2023-01-01 filter | 99,924 records |
| Working sample | 10,000 (random, `random_state=42`) |
| Columns | 22 (id, date, block, primary\_type, arrest, latitude, longitude, ...) |
| License | Public domain (City of Chicago) |
| Local storage | None — fetched at runtime; no data files committed |

Key columns used: `latitude`, `longitude`, `date`. The dataset covers incident type,
location description, arrest status, ward, community area, and FBI NIBRS codes.
See [data/DATA_SOURCES.md](data/DATA_SOURCES.md) for full schema and reproducibility notes.

**Preprocessing:** Drop rows with null latitude, longitude, or date; parse date strings to
datetime; filter to 2023-present; downsample to 10,000 records for memory efficiency.

---

## Key Technical Steps

```
Chicago Crime API (Socrata)
          |
          v
  Load 100k records
  (ijzp-q8t2, $limit=100000)
          |
          v
  Filter & clean
  (drop NA lat/lon/date, filter >= 2023-01-01)
  99,924 records remaining
          |
          v
  Random sample 10,000 records (random_state=42)
          |
     _____|_____
    |           |
    v           v
 HDBSCAN    ST-DBSCAN
 (spatial)  (space + time)
    |           |
    |___________|
          |
          v
  KDE density estimation
  (Gaussian, bandwidth=0.01)
          |
     _____|_____
    |           |
    v           v
 PAI@5%     Folium
 evaluation  heatmap
```

### Step 1 — Load and Clean

```python
url = "https://data.cityofchicago.org/resource/ijzp-q8t2.csv?$limit=100000"
df = pd.read_csv(url)
df = df.dropna(subset=['latitude', 'longitude', 'date'])
df['date'] = pd.to_datetime(df['date'])
recent_df = df[df['date'] >= '2023-01-01']
```

### Step 2 — Subsample

10,000 records are drawn with `random_state=42` to keep HDBSCAN and DBSCAN runtimes
manageable while preserving geographic representativeness.

### Step 3 — HDBSCAN Spatial Clustering

```python
clusterer = hdbscan.HDBSCAN(min_cluster_size=50, min_samples=10)
sample_df['hdbscan'] = clusterer.fit_predict(X)
```

HDBSCAN builds a hierarchy of density-connected components and extracts flat clusters
at the level of highest stability. Points that cannot be assigned to any stable cluster
receive label `-1` (noise). Parameters: `min_cluster_size=50`, `min_samples=10`.

### Step 4 — ST-DBSCAN (Spatio-Temporal)

Temporal distance is encoded as a third feature axis, scaled so that 3 days of elapsed
time equals one standard deviation of spatial distance:

```python
time_scaled = (time - time.mean()) / (60 * 60 * 24 * 3)  # 3 days = 1 unit
coords_time = np.hstack([space_scaled, time_scaled])
stdbscan = DBSCAN(eps=0.5, min_samples=10).fit(coords_time)
```

This approximates the neighborhood logic of true ST-DBSCAN without requiring a
separate library, and makes the space-time weighting decision explicit.

### Step 5 — KDE Baseline

A Gaussian KDE (bandwidth=0.01 decimal degrees, roughly 1.1 km at Chicago's latitude)
is fit to the spatial coordinates and evaluated at every sample point to produce a
continuous density score:

```python
kde = KernelDensity(bandwidth=0.01, kernel='gaussian').fit(coords)
densities = np.exp(kde.score_samples(coords))
```

### Step 6 — Predictive Evaluation (PAI@5%)

The Predictive Accuracy Index quantifies how efficiently a hotspot model concentrates
future crime predictions:

```
PAI@5% = (fraction of future crimes falling in predicted hotspots)
          / (fraction of total area covered by hotspots)
```

Here, the top 5% of KDE-scored points define the hotspot zone. Future crimes are drawn
from the last 30 days of the full 100k dataset (held out from training):

```
Validation window: 2025-09-11 → 2025-10-11
Validation records: 19,762
PAI@5% = 16.23%
```

A random model would capture 5% of future crimes in 5% of the area (PAI = 1.0).
This model achieves PAI = 3.25x, capturing 16.23% in that same 5% of risk area.

### Step 7 — Interactive Folium Heatmap

KDE density scores weight each point in a Folium `HeatMap` layer rendered on a
CartoDB dark-matter basemap, saved as `crime_hotspot_heatmap.html`.

---

## Results and Interpretation

| Metric | Value |
|---|---|
| Records loaded | 99,924 |
| Working sample | 10,000 |
| Validation window | 2025-09-11 → 2025-10-11 |
| Validation records | 19,762 |
| **PAI@5%** | **16.23%** |
| Lift over random | ~3.25x |

**HDBSCAN cluster observations:**
- High-density clusters correspond to the Loop, Near South Side, Garfield Park, and Englewood
- Low-density and suburban areas produce noise points (label -1)
- Cluster boundaries align with known Chicago neighborhood boundaries

**ST-DBSCAN observations:**
- Cluster assignments shift relative to HDBSCAN — spatially adjacent incidents from
  different time windows are separated, revealing episodic rather than persistent hotspots

**KDE density surface:**
- Produces a smooth gradient from high-risk corridors to low-risk suburban fringe
- Provides a continuous score usable for threshold-based alerting (e.g., top-10% zones)

> TODO: Export cluster count tables from notebook (number of HDBSCAN clusters, noise
> fraction, ST-DBSCAN cluster count) and add to the table above.

---

## Example Visualizations

The following plots are generated by the notebook. Export them by adding `plt.savefig(...)`
before each `plt.show()` call — see [images/README.md](images/README.md) for exact lines.

| Plot | File | Status |
|---|---|---|
| HDBSCAN spatial clusters | `images/hdbscan_spatial_clusters.png` | TODO — export from cell 3 |
| ST-DBSCAN spatio-temporal clusters | `images/stdbscan_spatiotemporal_clusters.png` | TODO — export from cell 4 |
| KDE density heatmap (static) | `images/kde_density_heatmap.png` | TODO — export from cell 5 |
| Folium interactive heatmap | `crime_hotspot_heatmap.html` | Generated at runtime (not committed) |

---

## Repository Structure

```
Geospatial-Clustering/
├── Project_Plan_Crime_Hotspot_Clustering.ipynb   # Main analysis notebook (7 sections)
├── Density_Based_Clustering_Geospatial_Analysis (2).pptx  # Slide deck
├── Project_Plan_Crime_Hotspot_Clustering.pptx    # Project plan slides
├── README.md                                      # This file
├── PROJECT_SUMMARY.md                             # Portfolio elevator pitch + resume bullets
├── requirements.txt                               # Pinned dependencies
├── .gitignore                                     # Python/Jupyter ignores
├── data/
│   └── DATA_SOURCES.md                            # API source, schema, reproducibility notes
└── images/
    └── README.md                                  # Visualization catalog + export instructions
```

---

## How to Run

### Prerequisites

- Python 3.10+
- Internet connection (data is fetched live from the Chicago API)

### Installation

```bash
git clone https://github.com/reidsendroff/Geospatial_Clustering-.git
cd Geospatial_Clustering-

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
pip install jupyter
```

### Run

```bash
jupyter notebook Project_Plan_Crime_Hotspot_Clustering.ipynb
```

Run all cells in order (Kernel > Restart & Run All). The first cell fetches data from the
API and may take 15–30 seconds depending on connection speed.

The interactive heatmap is saved to `crime_hotspot_heatmap.html` in the working directory.
Open it in any browser.

---

## Skills Demonstrated

**Mathematical / Statistical:**
- Kernel Density Estimation (Gaussian kernel, bandwidth selection)
- Density-based clustering (HDBSCAN, DBSCAN, ST-DBSCAN)
- Spatio-temporal distance scaling and feature engineering
- Predictive Accuracy Index (PAI) evaluation metric
- Hold-out temporal validation design

**Programming & Tools:**
- Python: pandas, numpy, scikit-learn, hdbscan, folium, matplotlib
- Socrata REST API data ingestion
- Jupyter Notebook workflow and cell-level documentation
- Git / GitHub version control

**Geospatial / Domain:**
- Geographic coordinate systems (WGS84 lat/lon)
- Urban crime data structure and IUCR / FBI NIBRS classification
- Interactive web map construction (Folium, CartoDB tiles)

---

## Project Context

Geospatial crime analysis and density-based clustering independent study project.
Data: City of Chicago Open Data Portal (public domain, updated daily).
All analysis is performed on publicly available, anonymized incident records.

---

## Why This Matters

Crime hotspot prediction is one of the most direct applications of geospatial machine
learning. Police departments in major cities allocate patrol resources partly on the basis
of predicted hotspot locations; a model that concentrates 16% of future incidents in 5% of
the predicted area provides a meaningful operational signal. Beyond policing, the same
density-based clustering pipeline applies to any event-point dataset: earthquake aftershock
mapping, disease outbreak surveillance, traffic incident prediction, or retail demand
forecasting. Density-based methods are specifically well-suited to these domains because
real-world spatial events rarely form the spherical, uniform-density clusters that k-means
assumes.

---

## Future Improvements

- [ ] Export static plot images to `images/` directory (add `plt.savefig(...)` to cells 3, 4, 5)
- [ ] Add HDBSCAN cluster count summary table (number of clusters, noise fraction)
- [ ] Add ST-DBSCAN cluster count and temporal span analysis per cluster
- [ ] Tune KDE bandwidth using cross-validated log-likelihood
- [ ] Compare PAI@5% across all three methods (HDBSCAN, ST-DBSCAN, KDE) rather than KDE alone
- [ ] Add crime type breakdown within each cluster (THEFT vs. BATTERY vs. ASSAULT)
- [ ] Extend to haversine distance metric (more accurate at city scale than Euclidean lat/lon)
- [ ] Export Folium map screenshot for embedding in README
- [ ] Add time-of-day analysis (night vs. day crime distribution within clusters)
- [ ] Publish interactive heatmap via GitHub Pages

---

## Author

Reid Sendroff
GitHub: [reidsendroff](https://github.com/reidsendroff)

---

*Three density estimators, one city, one question: where will crime happen next?*
