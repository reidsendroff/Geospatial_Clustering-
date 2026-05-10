# Visualization Catalog

## Social Preview Image

GitHub shows a 1280×640 px card when the repo URL is shared on LinkedIn, Slack, or Twitter.
The default is a generic GitHub tile. Run the script below once (after exporting the 3 plot
images) to generate a custom card that puts the headline result front and center.

Add this as a new cell at the end of the notebook, or run it as a standalone script:

```python
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches

fig, ax = plt.subplots(figsize=(12.8, 6.4), facecolor="#0d1117")
ax.set_xlim(0, 12.8)
ax.set_ylim(0, 6.4)
ax.axis("off")

# Title
ax.text(6.4, 4.5, "Crime Hotspot Clustering",
        ha="center", va="center", fontsize=40, color="white", fontweight="bold")

# Subtitle
ax.text(6.4, 3.65, "HDBSCAN · ST-DBSCAN · KDE  |  Chicago Crime Data  |  Python",
        ha="center", va="center", fontsize=18, color="#8b949e")

# Stat chips
for x, label in zip([2.5, 6.4, 10.3],
                    ["100k incidents", "PAI@5% = 16.23%", "3.25x lift"]):
    ax.add_patch(mpatches.FancyBboxPatch(
        (x - 1.5, 2.1), 3.0, 0.9,
        boxstyle="round,pad=0.1", facecolor="#21262d", edgecolor="#30363d"))
    ax.text(x, 2.55, label, ha="center", va="center",
            fontsize=15, color="#58a6ff", fontweight="bold")

import os
os.makedirs("images", exist_ok=True)
plt.tight_layout(pad=0)
plt.savefig("images/social_preview.png", dpi=100, bbox_inches="tight",
            facecolor="#0d1117")
plt.show()
print("Saved: images/social_preview.png")
```

After running, upload the image at:
**github.com/reidsendroff/Geospatial_Clustering- → Settings → Social preview → Edit → Upload image**

The `gh` CLI does not support binary uploads — browser upload is required.

---

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
