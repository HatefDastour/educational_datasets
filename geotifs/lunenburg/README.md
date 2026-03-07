# Lunenburg, Nova Scotia: Sentinel‑2 + SRTM (Multi‑dataset, Summer 2025)

This directory contains a **Sentinel‑2 7‑band surface reflectance GeoTIFF** and a separate **SRTM 30 m DEM** for **Lunenburg, Nova Scotia**, exported from Google Earth Engine. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)
The pair is ideal for teaching multi‑spectral coastal analysis, terrain‑aware modeling, and rasterio workflows with separate optical and elevation datasets. [earthdata.nasa](https://www.earthdata.nasa.gov/data/instruments/sentinel-2-msi)

## Study Area

**Lunenburg, NS AOI** – Historic UNESCO port town, harbor, and surrounding coastal waters on Nova Scotia’s South Shore. [en.wikipedia](https://en.wikipedia.org/wiki/Lunenburg,_Nova_Scotia)

- **Location (WGS84)**: 44.3531°–44.3935° N, 64.3547°–64.2767° W  
- **Approximate Area**: Reported in the script via `lunenburg.area().divide(1e6)` km²  
- **Landscape**: Low‑relief coastal town, harbor, mixed forest, agriculture, and open Atlantic inlets. [en-ca.topographic-map](https://en-ca.topographic-map.com/place-l6c8zs/Lunenburg-County/)
- **Why this AOI?** Compact coastal scene with strong land–water–urban contrasts and modest topographic variation suitable for DEM integration. [en-ca.topographic-map](https://en-ca.topographic-map.com/place-l6c8zs/Lunenburg-County/)

## Dataset Files

| File | Bands | Sensor | Resolution | Data Range | Purpose |
|------|-------|--------|------------|------------|---------|
| `lunenburg_s2_summer2025.tif` | 7 (coastal, blue, green, red, nir, swir1, swir2) | Sentinel‑2 L2A | 10 m (export) | 0–1 reflectance | Multi‑spectral coastal/urban analysis |
| `lunenburg_srtm_30m.tif` | 1 (elevation) | SRTMGL1 V3 | 30 m | Meters above sea level | Terrain and topographic context |

- **Sentinel‑2 source**: COPERNICUS/S2_SR_HARMONIZED Level‑2A surface reflectance, scaled by 1/10,000 in the raw product and explicitly divided by 10,000 in the script. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_HARMONIZED)
- **SRTM source**: USGS/NASA SRTMGL1_003 global 1‑arc‑second (~30 m) DEM, `elevation` band in meters. [earthdata.nasa](https://www.earthdata.nasa.gov/data/catalog/lpcloud-srtmgl1-003)
- **Data Type**: Both exports cast to Float32 with `.toFloat()` for downstream processing. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)
- **Projection**: EPSG:4326, 10 m export scale for Sentinel‑2, 30 m for SRTM.

## Google Earth Engine Script

```javascript
// ============================================================
// SENTINEL-2 + SRTM: Lunenburg, Nova Scotia (Multi-dataset)
// ============================================================

var lunenburg = ee.Geometry.Polygon([
  [[-64.3547, 44.3531],
   [-64.2767, 44.3531],
   [-64.2767, 44.3935],
   [-64.3547, 44.3935],
   [-64.3547, 44.3531]]
]);

Map.centerObject(lunenburg, 13);
Map.addLayer(lunenburg, {color: 'blue'}, 'Lunenburg AOI');

// === SENTINEL-2 (Summer 2025) ===
var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(lunenburg)
  .filterDate('2025-06-01', '2025-08-31')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 15))
  .map(function(image) {
    var qa = image.select('QA60');
    var cloudMask = qa.bitwiseAnd(1 << 10).eq(0);
    var cirrusMask = qa.bitwiseAnd(1 << 11).eq(0);
    return image.updateMask(cloudMask)
      .updateMask(cirrusMask)
      .divide(10000)
      .copyProperties(image, ['system:time_start']);
  });

var s2Composite = s2.median().clip(lunenburg);

// === SRTM DEM (30m global elevation) ===
var srtm = ee.Image('USGS/SRTMGL1_003')
  .select('elevation')
  .clip(lunenburg);

// === VISUALIZATION ===
var rgbVis = {bands: ['B4', 'B3', 'B2'], min: 0, max: 0.3};
Map.addLayer(s2Composite, rgbVis, 'S2 True Color');
Map.addLayer(srtm, {min: 0, max: 80, palette: ['blue', 'green', 'yellow', 'red']}, 'SRTM Elevation');

// Info
print('S2 images:', s2.size());
print('Elevation range:', srtm.reduceRegion({
  reducer: ee.Reducer.minMax(),
  geometry: lunenburg,
  scale: 30
}));
print('Area:', lunenburg.area().divide(1e6), 'km²');

// 1. S2 7-bands
var s2Bands = s2Composite.select(['B1','B2','B3','B4','B8','B11','B12'], 
                                ['coastal','blue','green','red','nir','swir1','swir2']);

Export.image.toDrive({
  image: s2Bands.toFloat(),
  description: 'lunenburg_s2_7bands_2025',
  folder: 'GEE_exports',
  fileNamePrefix: 'lunenburg_s2_summer2025',
  region: lunenburg, scale: 10, crs: 'EPSG:4326',
  maxPixels: 1e9, fileFormat: 'GeoTIFF'
});

// 2. SRTM Elevation
Export.image.toDrive({
  image: srtm.toFloat(),
  description: 'lunenburg_srtm_dem',
  folder: 'GEE_exports',
  fileNamePrefix: 'lunenburg_srtm_30m',
  region: lunenburg, scale: 30, crs: 'EPSG:4326',
  maxPixels: 1e8, fileFormat: 'GeoTIFF'
});
```

## Band Details

### Sentinel‑2 7‑Band Stack (`lunenburg_s2_summer2025.tif`)

```text
Band 1: coastal  (B1, coastal aerosol, 60 m native; water clarity / haze)
Band 2: blue     (B2, 10 m; shallow water, true‑color blue)
Band 3: green    (B3, 10 m; vegetation, true‑color green)
Band 4: red      (B4, 10 m; vegetation/chlorophyll, true‑color red)
Band 5: nir      (B8, 10 m; vegetation vigor, biomass)
Band 6: swir1    (B11, 20 m native; moisture, burned areas)
Band 7: swir2    (B12, 20 m native; soil, geology, dry surfaces)
```

- All bands are surface reflectance scaled to 0–1 via division by 10,000. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_HARMONIZED)
- Native Sentinel‑2 pixel sizes: 10 m (B2, B3, B4, B8), 20 m (B11, B12), 60 m (B1), resampled to 10 m at export. [earthdata.nasa](https://www.earthdata.nasa.gov/data/instruments/sentinel-2-msi)

### SRTM DEM (`lunenburg_srtm_30m.tif`)

- Single `elevation` band from SRTMGL1_003, units in meters above sea level, ~30 m resolution. [usgs](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1)
- Coverage includes global land between 60° N and 56° S, including the Lunenburg region. [earthdata.nasa](https://www.earthdata.nasa.gov/data/catalog/lpcloud-srtmgl1n-003)

## Example Usage: rasterio and Analysis

### 1. Reading Sentinel‑2 7‑Band Raster

```python
import rasterio

with rasterio.open('lunenburg_s2_summer2025.tif') as src:
    print("Bands:", src.count)
    print("Shape:", src.height, src.width)
    print("CRS:", src.crs)
    print("Bounds:", src.bounds)
    s2 = src.read()  # (7, H, W)

coastal = s2[0]
blue    = s2[1]
green   = s2[2]
red     = s2[3]
nir     = s2[4]
swir1   = s2[5]
swir2   = s2[6]
```

### 2. True Color Composite

```python
import numpy as np
import matplotlib.pyplot as plt

rgb = np.stack([red, green, blue], axis=-1)
rgb = np.clip(rgb, 0, 0.3) / 0.3

plt.imshow(rgb)
plt.title("Lunenburg Sentinel-2 True Color (10 m)")
plt.axis("off")
plt.show()
```

### 3. NDVI from Sentinel‑2

```python
ndvi = (nir - red) / (nir + red + 1e-6)
print("NDVI min/max:", np.nanmin(ndvi), np.nanmax(ndvi))
```

### 4. Working with SRTM DEM

```python
with rasterio.open('lunenburg_srtm_30m.tif') as dem_src:
    dem = dem_src.read(1)
    print("DEM resolution:", dem_src.res)
    print("DEM min/max:", dem.min(), dem.max())
```

This lets you relate vegetation patterns (NDVI) to elevation or derive terrain attributes (slope/aspect) from the DEM. [earthdata.nasa](https://www.earthdata.nasa.gov/data/catalog/lpcloud-srtmgl1-003)

## Learning Objectives

This multi‑dataset Lunenburg package supports: [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)

1. **Multi‑band Sentinel‑2 handling**: Reading, indexing, and visualizing a 7‑band reflectance stack.  
2. **Cloud‑aware compositing**: Applying QA60‑based masks and seasonal median compositing. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_HARMONIZED)
3. **Coastal remote sensing**: Leveraging coastal, visible, NIR, and SWIR bands for water, urban, and vegetation discrimination. [earthdata.nasa](https://www.earthdata.nasa.gov/data/instruments/sentinel-2-msi)
4. **Terrain context**: Integrating an external SRTM DEM for elevation‑aware interpretation. [earthdata.nasa](https://www.earthdata.nasa.gov/data/catalog/lpcloud-srtmgl1-003)
5. **Rasterio workflows**: Managing separate optical and elevation rasters in reproducible Python analyses.

## License and Citation

Source data:

- **Sentinel‑2 L2A Surface Reflectance** (COPERNICUS/S2_SR_HARMONIZED). [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)
- **SRTMGL1_003** Shuttle Radar Topography Mission global 1‑arc‑second DEM. [usgs](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1)

**Recommended Citation**:  
> Lunenburg, Nova Scotia Sentinel‑2 + SRTM educational rasters derived from Sentinel‑2 L2A (summer 2025) and SRTMGL1 V3 elevation, processed via Google Earth Engine.