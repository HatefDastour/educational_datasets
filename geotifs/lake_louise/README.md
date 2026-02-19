# Lake Louise GeoTIFF Dataset

This directory contains 6 fundamental raster datasets for Lake Louise (Banff National Park, Alberta, Canada) exported from Google Earth Engine. Perfect for teaching rasterio basics, multi-sensor comparison, and geospatial data processing.

## Study Area

**Lake Louise** - Iconic turquoise glacial lake in Banff National Park:
- **Location**: 51.39°-51.43° N, 116.19°-116.26° W
- **Area**: ~0.8 km² lake + surrounding alpine terrain
- **Elevation**: 1,773m (lake surface) to 3,100m+ (peaks)
- **Key Features**: Glacial sediment (rock flour), Victoria Glacier, alpine vegetation

## Dataset Files

| File | Bands | Sensor | Resolution | Data Range | Purpose |
|------|-------|--------|------------|------------|---------|
| `lake_louise_l9_rgb_nir_2025.tif` | 4 | Landsat 9 | 30m | 0-1 (reflectance) | Multi-band RGB+NIR |
| `lake_louise_s2_rgb_nir_2025.tif` | 4 | Sentinel-2 | 10m | 0-1 (reflectance) | High-res RGB+NIR |
| `lake_louise_srtm_dem.tif` | 1 | SRTM | 30m | 1400-3100m | Elevation |
| `lake_louise_ndvi_l9_2025.tif` | 1 | Landsat 9 | 30m | -1 to 1 | Vegetation index |
| `lake_louise_ndvi_s2_2025.tif` | 1 | Sentinel-2 | 10m | -1 to 1 | High-res vegetation |

**Temporal Coverage**: Summer 2025 (July) - peak glacial melt season

## Band Details

### Multi-band Files (RGB + NIR)
```
Band 1: Blue   (0.45-0.51μm L9 / 0.49μm S2)
Band 2: Green  (0.53-0.59μm L9 / 0.56μm S2) 
Band 3: Red    (0.64-0.67μm L9 / 0.66μm S2)
Band 4: NIR    (0.85-0.88μm L9 / 0.84μm S2)
```
- **Data Type**: Float32
- **Scaling**: Surface reflectance (0-1)

### Single-band Files
```
SRTM: Elevation (meters above sea level)
NDVI: Normalized Difference Vegetation Index (-1 to 1)
```

## Google Earth Engine Script

```javascript
// Lake Louise: 6 Simple Rasters for rasterio Fundamentals
var lakeLouise = ee.Geometry.Polygon([
  [[-116.2609, 51.3925], [-116.1900, 51.3925],
   [-116.1900, 51.4296], [-116.2609, 51.4296],
   [-116.2609, 51.3925]]
]);

// 1. Landsat 9: RGB + NIR (July 2025)
var l9_summer = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(lakeLouise).filterDate('2025-07-01', '2025-07-31')
  .filter(ee.Filter.lt('CLOUD_COVER', 10)).median().clip(lakeLouise);
var l9_scaled = l9_summer.multiply(0.0000275).subtract(0.2)
  .select(['SR_B2','SR_B3','SR_B4','SR_B5']).toFloat(); // B,G,R,NIR

// 2. Sentinel-2: RGB + NIR (10m, July 2025)
var s2_summer = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(lakeLouise).filterDate('2025-07-01', '2025-07-31')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10)).median().clip(lakeLouise);
var s2_scaled = s2_summer.multiply(0.0001).select(['B2','B3','B4','B8']).toFloat();

// 3. SRTM DEM
var srtm = ee.Image('USGS/SRTMGL1_003').select('elevation').clip(lakeLouise).toFloat();

// 4-5. NDVI (L9 & S2)
var ndvi_l9 = l9_summer.normalizedDifference(['SR_B5','SR_B4']).rename('NDVI_L9').clip(lakeLouise).toFloat();
var ndvi_s2 = s2_summer.normalizedDifference(['B8','B4']).rename('NDVI_S2').clip(lakeLouise).toFloat();

// Preview & Export (see full script in repository)
```

## Example Usage: rasterio Fundamentals

### 1. Basic Reading & Inspection
```python
import rasterio
import numpy as np

# Multi-band
with rasterio.open('lake_louise_l9_rgb_nir_2025.tif') as src:
    print(f"Bands: {src.count}, Shape: {src.shape}")
    print(f"CRS: {src.crs}, Bounds: {src.bounds}")
    rgb_nir = src.read()  # (4, height, width)

# Single-band  
with rasterio.open('lake_louise_srtm_dem.tif') as src:
    elevation = src.read(1)  # 2D array
    print(f"Elevation range: {elevation.min():.0f} - {elevation.max():.0f}m")
```

### 2. True Color Composites
```python
import matplotlib.pyplot as plt

def plot_rgb(blue, green, red, title):
    rgb = np.stack([red, green, blue], axis=-1)
    rgb = np.clip(rgb, 0, 0.3) / 0.3
    plt.imshow(rgb)
    plt.title(title)
    plt.axis('off')

with rasterio.open('lake_louise_l9_rgb_nir_2025.tif') as l9, \
     rasterio.open('lake_louise_s2_rgb_nir_2025.tif') as s2:
    
    l9_bands = l9.read()
    s2_bands = s2.read()
    
    fig, axes = plt.subplots(1, 2, figsize=(12, 6))
    plot_rgb(l9_bands[0], l9_bands[1], l9_bands[2], 'Landsat 9 (30m)')
    plot_rgb(s2_bands[0], s2_bands[1], s2_bands[2], 'Sentinel-2 (10m)')
plt.tight_layout()
plt.show()
```

### 3. Multi-Dataset Analysis
```python
# Lake elevation vs vegetation
with rasterio.open('lake_louise_srtm_dem.tif') as dem, \
     rasterio.open('lake_louise_ndvi_s2_2025.tif') as ndvi:
    
    elevation = dem.read(1)
    vegetation = ndvi.read(1)
    
    # Elevation profile of lake shoreline
    lake_mask = elevation < 1800  # Approx lake level
    shore_mask = (elevation > 1800) & (elevation < 2000)
    
    print(f"Lake pixels: {np.sum(lake_mask)}")
    print(f"Shoreline pixels: {np.sum(shore_mask)}")
    print(f"Mean shoreline elevation: {np.mean(elevation[shore_mask]):.0f}m")
    print(f"Mean shoreline NDVI: {np.mean(vegetation[shore_mask]):.3f}")
```

### 4. Sensor Comparison
```python
# Resample S2 to L9 resolution for comparison
from rasterio.warp import reproject, Resampling

l9_profile = rasterio.open('lake_louise_l9_rgb_nir_2025.tif').profile
s2_resampled = np.empty((4, l9_profile['height'], l9_profile['width']), dtype=np.float32)

with rasterio.open('lake_louise_s2_rgb_nir_2025.tif') as s2:
    for i in range(1, 5):
        reproject(
            source=rasterio.band(s2, i),
            destination=s2_resampled[i-1],
            src_transform=s2.transform,
            src_crs=s2.crs,
            dst_transform=l9_profile['transform'],
            dst_crs=l9_profile['crs'],
            resampling=Resampling.bilinear
        )
```

## Learning Objectives

1. **Rasterio basics**: Reading multi/single band, profiles, transforms
2. **Data scaling**: Landsat (0.0000275×-0.2), Sentinel-2 (/10000)
3. **Visualization**: True color, indices, colormaps
4. **Reprojection/Resampling**: Multi-resolution sensor fusion
5. **Masking**: Elevation thresholds, NDVI filtering
6. **Statistics**: Zonal stats, histograms

## Scientific Context

**Lake Louise** glacial turquoise color comes from **rock flour** (finely ground rock from Victoria Glacier). 
- High reflectance in Green (> Red/Blue) = turquoise appearance
- Low NDVI over lake (rock flour), high NDVI in forests
- Elevation gradient: lake (1773m) → tree line (~2200m) → alpine (>2500m)

## License

Public domain: USGS Landsat 9, NASA SRTM, ESA Sentinel-2

**Recommended Citation**:
> Lake Louise educational rasters derived from Landsat 9 Collection 2, Sentinel-2 L2A, SRTMGL1 (2025). Processed via Google Earth Engine.