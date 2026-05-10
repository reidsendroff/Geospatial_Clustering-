# Visualization Catalog

This directory holds exported static images from `Project_Plan_Crime_Hotspot_Clustering.ipynb`.
All plots below are generated during notebook execution. Add the `plt.savefig(...)` line
shown for each cell **before** `plt.show()` to export them.

---

## How to Export

Add the following snippet immediately before every `plt.show()` call:

```python
import os
os.makedirs("images", exist_ok=True)
plt.savefig("images/<filename>.png", dpi=150, bbox_inches="tight")
```

---

## Plot Catalog

### 1. `hdbscan_spatial_clusters.png`

**Description:** Scatter plot of 10,000 sampled crime incidents colored by HDBSCAN cluster label.

**Key finding:** Dense clusters emerge around the Loop, Near South Side, Garfield Park, and Englewood.
Noise points (label -1) appear scattered in low-density suburban areas.

**Export line (add to cell 3, before `plt.show()`):**
```python
plt.savefig("images/hdbscan_spatial_clusters.png", dpi=150, bbox_inches="tight")
```

**Embed in README:**
```markdown
![HDBSCAN Spatial Clusters](images/hdbscan_spatial_clusters.png)
```

**Status:** TODO — re-run notebook cell 3 and add savefig line.

---

### 2. `stdbscan_spatiotemporal_clusters.png`

**Description:** Scatter plot colored by ST-DBSCAN cluster labels, which incorporate both geographic
position and time (scaled so 3 days ~ 1 spatial unit).

**Key finding:** Cluster boundaries shift compared to pure spatial HDBSCAN, revealing crime
patterns that are geographically proximate but temporally separated.

**Export line (add to cell 4, before `plt.show()`):**
```python
plt.savefig("images/stdbscan_spatiotemporal_clusters.png", dpi=150, bbox_inches="tight")
```

**Embed in README:**
```markdown
![ST-DBSCAN Spatio-Temporal Clusters](images/stdbscan_spatiotemporal_clusters.png)
```

**Status:** TODO — re-run notebook cell 4 and add savefig line.

---

### 3. `kde_density_heatmap.png`

**Description:** Scatter plot colored by continuous KDE density score (Gaussian kernel, bandwidth=0.01).
Rendered with the `'hot'` colormap to mimic a heatmap.

**Key finding:** KDE reveals a smooth risk surface, useful as a baseline against which the
discrete clustering approaches can be compared.

**Export line (add to cell 5, before `plt.show()`):**
```python
plt.savefig("images/kde_density_heatmap.png", dpi=150, bbox_inches="tight")
```

**Embed in README:**
```markdown
![KDE Crime Density Heatmap](images/kde_density_heatmap.png)
```

**Status:** TODO — re-run notebook cell 5 and add savefig line.

---

## Image Guidelines

- Format: PNG
- DPI: 150 minimum (use 300 for print-quality)
- Size: aim for figures saved at `figsize=(8, 6)` or larger
- Colormap consistency: spatial scatter plots use `tab10`; density plots use `hot`
- Axis labels and titles are already set in the notebook cells

---

## Interactive Map

The Folium heatmap (`crime_hotspot_heatmap.html`) is excluded from this directory and from
version control (see `.gitignore`). To share it:

- Upload to GitHub Pages or a static host, or
- Export a screenshot with your browser's developer tools and save as `images/folium_heatmap_screenshot.png`
