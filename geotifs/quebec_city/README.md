# Quebec City MODIS NDVI: 2025 Growing Season

This repository provides a processed **MODIS Terra NDVI GeoTIFF** covering the Quebec City and St. Lawrence River region. The data is a median composite of the **2025 growing season**, derived from the MOD13Q1 16-day product at 250m resolution.

It is designed as a lightweight dataset for teaching **vegetation phenology**, **regional raster analysis**, and **Python-based remote sensing workflows** (e.g., `rasterio`, `numpy`).

---

## 📍 Study Area: Quebec City, QC

The Area of Interest (AOI) encompasses the urban corridor of Quebec City, the St. Lawrence River, and surrounding agricultural and forested lands.

* **Coordinates (WGS84):** 46.36°–47.09° N, 71.84°–70.63° W
* **Landscape:** A diverse mix of high-density urban zones (Old Quebec), suburban sprawl, fertile St. Lawrence lowlands, and boreal forested uplands.
* **Why this area?** It provides a perfect "natural laboratory" for studying urban heat islands, rural-urban gradients, and riverine ecosystem dynamics using coarse-resolution sensors.

---

## 📊 Dataset Specifications

| Attribute | Details |
| --- | --- |
| **File Name** | `quebec_city_modis_ndvi_summer2025.tif` |
| **Product** | MODIS/Terra Vegetation Indices (MOD13Q1.061) |
| **Temporal Range** | May 1, 2025 – Sept 30, 2025 (Median Composite) |
| **Spatial Res** | 250 meters |
| **Projection** | EPSG:4326 (WGS 84) |
| **Data Type** | Float32 |
| **Value Range** | -0.2 to 1.0 (NDVI units) |

> **Technical Note:** While MOD13Q1 raw data is stored as 16-bit integers with a 0.0001 scale factor, this GeoTIFF has been pre-scaled to physical NDVI values for ease of use.

---

## 🛠️ Google Earth Engine Pipeline

The dataset was generated using the following GEE logic. You can use this snippet to replicate or adjust the temporal window.

```javascript
// Define the Quebec City AOI
var quebecCity = ee.Geometry.Rectangle([-71.8455, 46.3683, -70.6315, 47.0968]);

// Process MODIS NDVI
var modisNDVI = ee.ImageCollection('MODIS/061/MOD13Q1')
  .filterBounds(quebecCity)
  .filterDate('2025-05-01', '2025-09-30')
  .select('NDVI')
  .median()             // Create seasonal composite
  .multiply(0.0001)     // Apply MODIS scale factor
  .clip(quebecCity);

// Export as Float32 GeoTIFF
Export.image.toDrive({
  image: modisNDVI.toFloat(),
  description: 'quebec_modis_ndvi_2025',
  scale: 250,
  region: quebecCity,
  fileFormat: 'GeoTIFF'
});

```

---

## 🐍 Quick Start: Python Analysis

### 1. Load and Inspect

```python
import rasterio
import matplotlib.pyplot as plt

with rasterio.open('quebec_city_modis_ndvi_summer2025.tif') as src:
    ndvi = src.read(1)
    
    plt.figure(figsize=(10, 6))
    plt.imshow(ndvi, cmap='RdYlGn', vmin=0, vmax=0.9)
    plt.colorbar(label='NDVI Value')
    plt.title('Quebec City NDVI (Summer 2025)')
    plt.show()

```

### 2. Interpretation Guide

| NDVI Range | Land Cover Type |
| --- | --- |
| **< 0.1** | Water, snow, or barren rock |
| **0.1 – 0.3** | Urban areas, sparse vegetation, or bare soil |
| **0.3 – 0.6** | Grasslands, shrubs, or agricultural crops |
| **> 0.6** | Dense temperate/boreal forest |

---

## 🎓 Learning Objectives

This repository supports several key remote sensing competencies:

1. **Bit-Depth & Scaling:** Understanding why satellite products require scale factors.
2. **Temporal Aggregation:** The benefits of median compositing to remove cloud artifacts.
3. **Spatial Analysis:** Calculating "greenness" statistics for urban planning.
4. **Python GIS:** Gaining proficiency in `rasterio` and `matplotlib` for geospatial viz.

---

## 📜 License and Citation

**Data Source:** NASA LP DAAC MODIS/Terra Vegetation Indices 16-Day L3 Global 250m (MOD13Q1).

**Citation:**

> Quebec City MODIS Terra NDVI Dataset (2025). Derived from MOD13Q1 Collection 061 via Google Earth Engine. Distributed via GitHub.