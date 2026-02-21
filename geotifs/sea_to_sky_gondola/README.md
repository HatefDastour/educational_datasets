# Sea to Sky Gondola: Landsat 8 Red \& NIR (Summer 2025)

This directory contains separate **Red** and **Near-Infrared (NIR)** surface reflectance GeoTIFFs for the Sea to Sky Gondola area near Squamish, British Columbia, exported from Google Earth Engine.

These rasters are ideal for teaching rasterio basics, vegetation indices (NDVI), and band-by-band analysis using a clean, small-area subset.

## Study Area

**Sea to Sky Gondola AOI** – Coastal mountain terrain above Howe Sound near Squamish, BC.

- **Location (WGS84)**: 49.6577°–49.6817° N, 123.1678°–123.1165° W
- **Approximate Area**: Printed in the script using `seaToSky.area().divide(1e6)` (km²)
- **Landscape**: Steep forested slopes, rock outcrops, gondola corridor, and infrastructure platforms
- **Why this AOI?** Compact, visually interpretable terrain with strong red–NIR contrast between vegetation, rock, and built features.


## Dataset Files

| File | Bands | Sensor | Resolution | Data Range | Purpose |
| :-- | :-- | :-- | :-- | :-- | :-- |
| `seatosky_l8_red_summer2025.tif` | 1 (Red, SR_B4) | Landsat 8 | 30 m | ~0–1 (surface reflectance) | Red band for RGB/indices |
| `seatosky_l8_nir_summer2025.tif` | 1 (NIR, SR_B5) | Landsat 8 | 30 m | ~0–1 (surface reflectance) | NIR band for NDVI/vegetation |

- **Data Type**: Float32 reflectance (scaled in script using Landsat Collection 2 coefficients).
- **Temporal Coverage**: Summer 2025 (June–August), low cloud, median composite.
- **Projection**: EPSG:4326, 30 m nominal pixel size.


## Google Earth Engine Script

```javascript
// ============================================================
// LANDSAT 8: SEPARATE RED & NIR - Sea to Sky Gondola (Exact Coords)
// ============================================================

var seaToSky = ee.Geometry.Polygon([
  [[-123.1678, 49.6577],  // Point 0
   [-123.1165, 49.6577],  // Point 1  
   [-123.1165, 49.6817],  // Point 2
   [-123.1678, 49.6817],  // Point 3
   [-123.1678, 49.6577]]  // Point 4 (closed)
]);

Map.centerObject(seaToSky, 14);
Map.addLayer(seaToSky, {color: 'red', fillColor: '00000000'}, 'Sea to Sky AOI');

// Landsat 8 summer 2025 - low cloud cover
var landsat8 = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
  .filterBounds(seaToSky)
  .filterDate('2025-06-01', '2025-08-31')
  .filter(ee.Filter.lt('CLOUD_COVER', 15))
  .map(function(image) {
    var qaMask = image.select('QA_PIXEL').bitwiseAnd(parseInt('11111', 2)).eq(0);
    var satMask = image.select('QA_RADSAT').eq(0);
    var optical = image.select('SR_B.*').multiply(0.0000275).add(-0.2);
    return image.addBands(optical, null, true)
      .updateMask(qaMask)
      .updateMask(satMask;
  });

var composite = landsat8.median().clip(seaToSky);

// SINGLE BAND EXPORTS
var redBand = composite.select(['SR_B4']).rename('red');
var nirBand = composite.select(['SR_B5']).rename('nir');

// Visualization
Map.addLayer(redBand, {min: 0, max: 0.4, gamma: 1.2}, 'Red (SR_B4)');
Map.addLayer(nirBand, {min: 0, max: 0.5, gamma: 1.1}, 'NIR (SR_B5)');

// NDVI preview
var ndvi = nirBand.subtract(redBand).divide(nirBand.add(redBand)).rename('NDVI');
Map.addLayer(ndvi, {min: -0.2, max: 0.8, palette: ['red', 'yellow', 'green']}, 'NDVI');

// Info
print('Area size:', seaToSky.area().divide(1e6), 'km²');
print('Scenes found:', landsat8.size());
print('AOI bounds:', seaToSky.bounds().coordinates());

// EXPORT SEPARATELY (~1.5MB each)
Export.image.toDrive({
  image: redBand.toFloat(),
  description: 'seatosky_l8_red_2025',
  folder: 'GEE_exports',
  fileNamePrefix: 'seatosky_l8_red_summer2025',
  region: seaToSky, scale: 30, crs: 'EPSG:4326',
  maxPixels: 1e8, fileFormat: 'GeoTIFF'
});

Export.image.toDrive({
  image: nirBand.toFloat(),
  description: 'seatosky_l8_nir_2025', 
  folder: 'GEE_exports',
  fileNamePrefix: 'seatosky_l8_nir_summer2025',
  region: seaToSky, scale: 30, crs: 'EPSG:4326',
  maxPixels: 1e8, fileFormat: 'GeoTIFF'
});
```


## Example Usage: rasterio Basics

### 1. Reading and Inspecting the Bands

```python
import rasterio

with rasterio.open('seatosky_l8_red_summer2025.tif') as red_src:
    red = red_src.read(1)
    print("Red band shape:", red.shape)
    print("CRS:", red_src.crs)
    print("Bounds:", red_src.bounds)

with rasterio.open('seatosky_l8_nir_summer2025.tif') as nir_src:
    nir = nir_src.read(1)
    print("NIR band shape:", nir.shape)
```


### 2. Quick NDVI from Separate Files

```python
import numpy as np

# Assume red, nir read as above
ndvi = (nir - red) / (nir + red + 1e-6)

print("NDVI range:", np.nanmin(ndvi), np.nanmax(ndvi))
```


### 3. Stacking to a Two-Band Raster

```python
import rasterio
from rasterio.transform import Affine

with rasterio.open('seatosky_l8_red_summer2025.tif') as red_src, \
     rasterio.open('seatosky_l8_nir_summer2025.tif') as nir_src:
    
    profile = red_src.profile
    profile.update(count=2)

    red = red_src.read(1)
    nir = nir_src.read(1)

    with rasterio.open('seatosky_l8_red_nir_stack_2025.tif', 'w', **profile) as dst:
        dst.write(red, 1)
        dst.write(nir, 2)
```


## Learning Objectives

This mini-dataset is designed to support:

1. **Rasterio basics**: Opening single-band GeoTIFFs, reading arrays, inspecting metadata.
2. **Radiometric scaling**: Understanding Landsat Collection 2 reflectance scaling (`0.0000275 × + (−0.2)`).
3. **Vegetation indices**: Implementing NDVI from separate red and NIR rasters.
4. **Masking \& quality**: Discussing QA masks (`QA_PIXEL`, `QA_RADSAT`) and their role in cleaning scenes.
5. **Small-area exports**: Using clipped AOIs for lightweight teaching datasets.

## License and Citation

All source data come from **USGS Landsat 8 Collection 2 Level-2** products accessed via Google Earth Engine.

**Recommended Citation**:
> Sea to Sky Gondola educational red and NIR rasters derived from Landsat 8 Collection 2 Level-2 (summer 2025), processed via Google Earth Engine.

