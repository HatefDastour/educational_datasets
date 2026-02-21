# Lunenburg, Nova Scotia: Sentinel‑2 + SRTM Multi‑Sensor (Summer 2025)

This directory contains an 8‑band **multi‑sensor GeoTIFF** for **Lunenburg, Nova Scotia** that combines Sentinel‑2 surface reflectance with SRTM elevation, plus a separate SRTM DEM export. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003)
The dataset is designed for teaching multi‑spectral coastal analysis, terrain‑aware modeling, and rasterio workflows with mixed‑resolution data.

## Study Area

**Lunenburg, NS AOI** – Historic UNESCO port town, harbor, and surrounding coastal waters on Nova Scotia’s South Shore.

- **Location (WGS84)**: 44.3531°–44.3935° N, 64.3547°–64.2767° W  
- **Approximate Area**: Reported in the script via `lunenburg.area().divide(1e6)` km²  
- **Landscape**: Low‑relief coastal town and harbor fringed by mixed forest, agricultural fields, and open water. [elevation](https://elevation.city/ca/3lxit)
- **Why this AOI?** Compact coastal scene with strong spectral contrast and modest relief for integrating reflectance and topography.

## Dataset Files

| File | Bands | Sensor(s) | Resolution | Data Range | Purpose |
|------|-------|-----------|-----------|-----------|---------|
| `lunenburg_s2_srtm_summer2025.tif` | 8 (7 optical + 1 elevation) | Sentinel‑2 L2A + SRTMGL1 | 10 m export (SRTM resampled from 30 m) | 0–1 (optical), meters (elevation) | Multi‑sensor coastal & terrain analysis |
| `lunenburg_srtm_30m.tif` | 1 (elevation) | SRTMGL1 V3 | 30 m | Meters above sea level | Stand‑alone DEM for terrain studies |

- **Source products**:  
  - Sentinel‑2 L2A Surface Reflectance (COPERNICUS/S2_SR_HARMONIZED), reflectance scaled by 1/10,000. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)
  - SRTMGL1_003 elevation at ~30 m, single `elevation` band in meters. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003)
- **Data Type**: Float32 in exports (`toFloat()` on optical and DEM bands).  
- **Projection**: EPSG:4326, 10 m export scale for the multi‑sensor stack, 30 m for the separate SRTM DEM.

## Google Earth Engine Script

```javascript
// ============================================================
// SENTINEL-2 + SRTM: Lunenburg, Nova Scotia (Summer 2025)
// ============================================================

var lunenburg = ee.Geometry.Polygon([
  [[-64.3547, 44.3531], [-64.2767, 44.3531],
   [-64.2767, 44.3935], [-64.3547, 44.3935],
   [-64.3547, 44.3531]]
]);

Map.centerObject(lunenburg, 13);
Map.addLayer(lunenburg, {color: 'blue'}, 'Lunenburg AOI');

// === SENTINEL-2 (7 optical bands) ===
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

var s2_composite = s2.median().clip(lunenburg);

// === SRTM DEM (elevation band 8) ===
var srtm = ee.Image('USGS/SRTMGL1_003')
  .select('elevation')
  .clip(lunenburg);

// === COMBINE: 7 S2 bands + 1 SRTM = 8 bands total ===
var multiSensor = s2_composite.select([
  'B1', 'B2', 'B3', 'B4', 'B8', 'B11', 'B12'
], ['coastal', 'blue', 'green', 'red', 'nir', 'swir1', 'swir2'])
  .addBands(srtm.rename('elevation'));

// Visualization
var rgbVis = {bands: ['red', 'green', 'blue'], min: 0, max: 0.3};
var demVis = {min: 0, max: 80, palette: ['blue', 'green', 'yellow', 'red']};

Map.addLayer(multiSensor, rgbVis, 'S2 True Color (10m)');
Map.addLayer(multiSensor.select('elevation'), demVis, 'SRTM Elevation');

// Info
print('S2 images:', s2.size());
print('SRTM elevation range:', multiSensor.select('elevation').reduceRegion({
  reducer: ee.Reducer.minMax(), 
  geometry: lunenburg, 
  scale: 30, maxPixels: 1e6
}));
print('8 bands:', multiSensor.bandNames());
print('Area:', lunenburg.area().divide(1e6), 'km²');

// EXPORT 8-BAND MULTI-SENSOR (~85MB)
Export.image.toDrive({
  image: multiSensor.toFloat(),
  description: 'lunenburg_s2_srtm_8band_2025',
  folder: 'GEE_exports',
  fileNamePrefix: 'lunenburg_s2_srtm_summer2025',
  region: lunenburg,
  scale: 10,  // 10m (resamples SRTM from 30m)
  crs: 'EPSG:4326',
  maxPixels: 1e9,
  fileFormat: 'GeoTIFF'
});

// SRTM SEPARATE EXPORT ===
Export.image.toDrive({
  image: srtm.toFloat(),
  description: 'lunenburg_srtm',
  folder: 'GEE_exports',
  fileNamePrefix: 'lunenburg_srtm_30m',
  region: lunenburg,
  scale: 30,  // Native SRTM resolution
  crs: 'EPSG:4326',
  maxPixels: 1e8,
  fileFormat: 'GeoTIFF'
});
```

## Band Details

### 8‑Band Multi‑Sensor Stack (`lunenburg_s2_srtm_summer2025.tif`)

```text
Band 1: coastal    (Sentinel‑2 B1, coastal aerosol, reflectance 0–1)
Band 2: blue       (Sentinel‑2 B2, 10 m, reflectance 0–1)
Band 3: green      (Sentinel‑2 B3, 10 m, reflectance 0–1)
Band 4: red        (Sentinel‑2 B4, 10 m, reflectance 0–1)
Band 5: nir        (Sentinel‑2 B8, 10 m, reflectance 0–1)
Band 6: swir1      (Sentinel‑2 B11, 20 m native, resampled)
Band 7: swir2      (Sentinel‑2 B12, 20 m native, resampled)
Band 8: elevation  (SRTMGL1_003, 30 m native, resampled; meters above sea level)
```

- **Optical bands**: Sentinel‑2 Level‑2A surface reflectance, atmospherically corrected and scaled by 1/10,000 in the source dataset. [docs.sentinel-hub](https://docs.sentinel-hub.com/api/latest/data/sentinel-2-l2a/)
- **Elevation band**: SRTM `elevation` band is a DEM in meters at ~30 m resolution before resampling. [earthdata.nasa](https://www.earthdata.nasa.gov/data/catalog/lpcloud-srtmgl1-003)

### Separate DEM (`lunenburg_srtm_30m.tif`)

- Single‑band SRTM DEM at 30 m (`elevation` in meters). [developers.google](https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003)
- Suitable as a reference for comparing resampling effects against the 10 m elevation band in the multi‑sensor stack.

## Example Usage: rasterio, NDVI, and Terrain

### 1. Reading the 8‑Band Stack

```python
import rasterio

with rasterio.open('lunenburg_s2_srtm_summer2025.tif') as src:
    print("Bands:", src.count)
    print("Shape:", src.height, src.width)
    print("CRS:", src.crs)
    print("Bounds:", src.bounds)
    data = src.read()  # (8, H, W)

```

### 2. NDVI and Elevation Relationship

```python
import numpy as np

ndvi = (nir - red) / (nir + red + 1e-6)

print("NDVI range:", np.nanmin(ndvi), np.nanmax(ndvi))
print("Elevation range:", np.nanmin(elevation), np.nanmax(elevation))
```

You can then explore slope‑dependent vegetation patterns, harbor vs upland contrasts, or elevation‑conditioned indices.

### 4. Working with Native SRTM

```python
with rasterio.open('lunenburg_srtm_30m.tif') as dem_src:
    dem = dem_src.read(1)
    print("Native SRTM resolution:", dem_src.res)
    print("DEM min/max:", dem.min(), dem.max())
```

This enables comparison between native 30 m SRTM and the resampled 10 m elevation band in the multi‑sensor stack.

## Learning Objectives

This multi‑sensor coastal scene supports:

1. **Multi‑sensor integration**: Combining Sentinel‑2 reflectance with SRTM elevation into a single 8‑band product. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)
2. **Cloud‑masked composites**: Building low‑cloud seasonal composites using QA60 masks. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)
3. **Topography‑aware analysis**: Examining how elevation modulates land cover patterns (e.g., vegetation, settlement). [en-ca.topographic-map](https://en-ca.topographic-map.com/map-xhstj/Lunenburg-County/)
4. **Resampling concepts**: Understanding implications of resampling a 30 m DEM to a 10 m grid. [developers.google](https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003)
5. **Rasterio practice**: Reading multi‑band stacks, deriving indices (NDVI), and visualizing both imagery and terrain.

## License and Citation

Source data:

- **Sentinel‑2 L2A Surface Reflectance** (COPERNICUS/S2_SR_HARMONIZED). [developers.google](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)
- **SRTMGL1_003** Shuttle Radar Topography Mission 1‑arc‑second global DEM. [earthdata.nasa](https://www.earthdata.nasa.gov/data/catalog/lpcloud-srtmgl1-003)

**Recommended Citation**:  
> Lunenburg, Nova Scotia multi‑sensor (Sentinel‑2 + SRTM) educational raster derived from Sentinel‑2 L2A (summer 2025) and SRTMGL1 V3 elevation, processed via Google Earth Engine.