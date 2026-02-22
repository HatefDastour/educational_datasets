# Albuquerque, NM: MODIS NDVI + LST + SRTM (July 2025)

This repository contains a multi-layer geospatial dataset for **Albuquerque, New Mexico**, featuring MODIS-derived vegetation and temperature indices paired with SRTM elevation data. The data was processed and exported via Google Earth Engine to support teaching multi-scale remote sensing, urban heat island analysis, and terrain-aware environmental modeling.

## Study Area: Albuquerque Basin

The Area of Interest (AOI) covers the urban core of Albuquerque, the fertile Rio Grande Valley, and the dramatic elevation rise of the Sandia Mountains.

* **Location (WGS84):** 34.95°–35.35° N, 106.80°–106.20° W
* **Landscape:** A high-desert environment featuring a city-suburban interface, irrigated river-valley agriculture, riparian corridors (the "Bosque"), and arid foothills transitioning into montane forests.
* **Scientific Value:** This region is ideal for studying the "Inverse Urban Heat Island" effect common in arid cities, where irrigated urban greenery can be cooler than surrounding dry shrublands.

---

## Dataset Specifications

| File | Product | Resolution | Data Range | Purpose |
| --- | --- | --- | --- | --- |
| `abq_modis_ndvi_july2025.tif` | MOD13Q1 (NDVI) | 250 m | 0.0 to 1.0 | Vegetation health/vigor |
| `abq_modis_lst_july2025.tif` | MOD11A2 (LST) | 1 km | Celsius (°C) | Surface temperature patterns |
| `abq_srtm_elevation.tif` | SRTMGL1 (DEM) | 30 m | Meters (a.s.l.) | Topographic context |

### Technical Details

* **NDVI Scaling:** Derived from MOD13Q1 16-day composites. Scaled from raw integers to physical units (0–1) using a  multiplier.
* **LST Processing:** Derived from MOD11A2 8-day composites. Converted from Kelvin to Celsius using: .
* **Elevation:** SRTM 1-arc second (~30 m) void-filled elevation data.
* **Format:** All files are exported as **Float32 GeoTIFFs** in **EPSG:4326** (WGS84).

---

## Google Earth Engine Implementation

```javascript
// Albuquerque, NM AOI (Rio Grande Valley & Sandia Mountains)
var albuquerque = ee.Geometry.Rectangle([-106.80, 34.95, -106.20, 35.35]);

// 1. MODIS NDVI (250m)
var modisNDVI = ee.ImageCollection("MODIS/061/MOD13Q1")
  .filterDate('2025-07-01', '2025-07-31')
  .select('NDVI').median()
  .multiply(0.0001).clip(albuquerque);

// 2. MODIS LST Day (1km)
var modisLST = ee.ImageCollection("MODIS/061/MOD11A2")
  .filterDate('2025-07-01', '2025-07-31')
  .select('LST_Day_1km').mean()
  .multiply(0.02).subtract(273.15).clip(albuquerque);

// 3. SRTM DEM (30m)
var srtm = ee.Image("USGS/SRTMGL1_003").select('elevation').clip(albuquerque);

// Export Example (NDVI)
Export.image.toDrive({
  image: modisNDVI.toFloat(),
  description: 'albuquerque_modis_ndvi_july2025',
  scale: 250,
  region: albuquerque,
  fileFormat: 'GeoTIFF'
});

```

---

## Python Analysis Examples

### NDVI vs. Land Surface Temperature

Using `rasterio` and `matplotlib`, you can explore the correlation between vegetation density and surface cooling.

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Load datasets
with rasterio.open('albuquerque_modis_ndvi_july2025.tif') as n_src, \
     rasterio.open('albuquerque_modis_lst_day_july2025.tif') as l_src:
    ndvi = n_src.read(1)
    lst = l_src.read(1)

# Visualization of the NDVI-LST Relationship
plt.scatter(ndvi.flatten(), lst.flatten(), alpha=0.1, s=1)
plt.xlabel("NDVI")
plt.ylabel("Daytime LST (°C)")
plt.title("Vegetation vs. Surface Temp: Albuquerque (July 2025)")
plt.show()

```

---

## Learning Objectives

1. **Resolution Mismatch:** Learn to handle multi-sensor fusion where resolutions vary significantly (30 m, 250 m, and 1 km).
2. **Arid Phenology:** Observe how the Rio Grande riparian corridor maintains high NDVI while surrounding areas remain dormant.
3. **Lapse Rates:** Use the SRTM layer to calculate the adiabatic lapse rate (temperature drop per 1000 m of elevation gain) across the Sandia Mountains.
4. **Scaling Factors:** Practice converting raw satellite Digital Numbers (DN) into meaningful physical units (Celsius, Index units).

---

## Citation

**Data Sources:**

* MODIS NDVI: NASA LP DAAC (MOD13Q1.061)
* MODIS LST: NASA LP DAAC (MOD11A2.061)
* SRTM: NASA/USGS (SRTMGL1_003)

**Recommended Citation:**

> Albuquerque, New Mexico Multi-Sensor Educational Dataset (July 2025). Derived from MODIS Terra and SRTMGL1 products via Google Earth Engine.