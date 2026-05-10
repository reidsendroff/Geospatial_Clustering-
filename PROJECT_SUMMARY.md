# Project Summary — Crime Hotspot Clustering: Geospatial Analysis of Chicago Crime Data

## Concise Summary

This project applies three density-based clustering algorithms — HDBSCAN, ST-DBSCAN, and
Kernel Density Estimation — to real-time Chicago crime data to identify spatial and
spatio-temporal crime hotspots. Pulling 100,000 records live from the City of Chicago Open
Data Portal, the pipeline clusters 10,000 incidents, evaluates predictive accuracy using the
Predictive Accuracy Index (PAI@5%), and renders results as an interactive Folium heatmap.
The KDE-based hotspot model captured 16.23% of future crimes within the top 5% of predicted
risk zones — more than three times the rate expected by random chance.

---

## Resume Bullets

- Applied HDBSCAN, ST-DBSCAN, and KDE to 100k Chicago crime records (Socrata API) to
  identify spatial and spatio-temporal crime hotspots, achieving PAI@5% = 16.23% on a
  held-out 30-day validation window of 19,762 incidents.
- Built a reproducible Python/Jupyter geospatial analysis pipeline integrating scikit-learn,
  hdbscan, and folium; implemented spatio-temporal feature scaling from first principles to
  simulate ST-DBSCAN using standard DBSCAN primitives.
- Designed and computed PAI@5% evaluation metric to benchmark KDE hotspot predictions against
  real future crime locations, demonstrating 3x lift over random baseline across 19k
  validation records.

---

## Technical Explanation

The project compares three density-based approaches for geospatial hotspot detection on
Chicago Police Department incident data. HDBSCAN (min\_cluster\_size=50, min\_samples=10) is
applied directly to latitude/longitude coordinates, discovering clusters of variable density
without requiring a pre-specified cluster count. ST-DBSCAN is simulated by constructing a
three-dimensional feature space: spatially normalized latitude/longitude plus a temporal axis
scaled so that three days of separation equals one unit of spatial distance (one standard
deviation). Standard DBSCAN (eps=0.5, min\_samples=10) is then applied to this combined
space, approximating the spatio-temporal neighborhood logic of true ST-DBSCAN. KDE
(Gaussian kernel, bandwidth=0.01) provides a continuous risk surface baseline. Predictive
performance is evaluated with PAI@5%: the KDE identifies top-5% risk zones, and the fraction
of crimes from the subsequent 30-day window falling in those zones is measured. The result —
16.23% of 19,762 future crimes captured in 5% of the study area — demonstrates meaningful
predictive signal.

---

## Interview Version

In this project, I built a geospatial crime hotspot detection pipeline entirely from publicly
available data. I pulled 100,000 recent Chicago crime records live from the city's open data
API and applied three complementary clustering algorithms: HDBSCAN for robust spatial
clustering that handles noise naturally, a spatio-temporal variant of DBSCAN that I
implemented by engineering a combined space-time feature space with calibrated scaling, and
Kernel Density Estimation as a continuous baseline. The most interesting challenge was the
ST-DBSCAN approximation — the `st-dbscan` library had environment conflicts, so I encoded
the temporal dimension as a normalized third axis and tuned the scaling factor so three days
of time separated incidents roughly as much as one standard deviation in space. I then
evaluated the KDE model with PAI@5%, a metric borrowed from predictive policing research:
16.23% of crimes in the following month fell within the top 5% of predicted risk areas,
which is about three times better than random.

---

## Why This Project Stands Out

Most introductory clustering projects apply k-means to toy datasets. This project:

- Uses live government data via a REST API rather than a static CSV
- Compares three methodologically distinct density estimators side-by-side
- Implements a spatio-temporal feature engineering workaround from scratch
- Applies a domain-specific evaluation metric (PAI) rather than generic silhouette scores
- Produces an interactive web map as a deliverable, not just static plots

---

## Key Skills Demonstrated

**Geospatial / Domain:**
- Density-based spatial clustering (HDBSCAN, DBSCAN)
- Spatio-temporal feature engineering and distance scaling
- Kernel Density Estimation for risk surface modeling
- Predictive Accuracy Index (PAI) evaluation

**Programming & Tools:**
- Python (pandas, numpy, scikit-learn, hdbscan, folium, matplotlib)
- Socrata REST API data ingestion
- Jupyter Notebook workflow

**Statistical / Mathematical:**
- Gaussian kernel density estimation
- Feature normalization and dimensional scaling
- Hold-out temporal validation design

---

## Project Outcomes

**PAI@5% = 16.23%** — future crimes captured in top-5% risk zones (30-day validation, 19,762 records)

**3x+ lift** over random baseline (random expectation = 5%)

**100k records** ingested live from City of Chicago Socrata API

**3 algorithms** compared: HDBSCAN, ST-DBSCAN (approximated), KDE

---

## Context

Geospatial crime analysis / density-based clustering independent project.
Data: City of Chicago Open Data Portal (public domain).
Language: Python 3. Primary libraries: scikit-learn, hdbscan, folium.
