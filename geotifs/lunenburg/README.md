# Lunenburg, Nova Scotia: Sentinel‑2 Multi‑Band (Summer 2025)

This directory contains a 7‑band Sentinel‑2 surface reflectance GeoTIFF for **Lunenburg, Nova Scotia** (UNESCO World Heritage port town) exported from Google Earth Engine.
The dataset is well suited for teaching multi‑spectral analysis, true/false color visualization, and basic coastal/urban remote sensing.

## Study Area

**Lunenburg, NS AOI** – Historic town, harbor, and surrounding coastal waters on the South Shore of Nova Scotia.

- **Location (WGS84)**: 44.3531°–44.3935° N, 64.3547°–64.2767° W
- **Approximate Area**: Printed in the script as `lunenburg.area().divide(1e6)` km² (roughly 7.8 km × 4 km)
- **Landscape**: Coastal town grid, harbor infrastructure, small vessels, mixed forest, agriculture, and open water
- **Why this AOI?** Compact coastal scene with strong spectral contrast between water, urban surfaces, and vegetation.


## Dataset File

| File | Bands | Sensor | Resolution | Data Range | Purpose |
| :-- | :-- | :-- | :-- | :-- | :-- |
| `lunenburg_s2_summer2025_7bands.tif` | 7 (coastal, blue, green, red, nir, swir1, swir2) | Sentinel‑2 L2A | 10–20 m (exported at 10 m) | 0–1 (surface reflectance) | Multi‑spectral coastal/urban analysis |

- **Data Type**: Float32 (scaled to surface reflectance by dividing DN by 10,000).
- **Temporal Coverage**: Summer 2025 (June–August), low‑cloud median composite.
- **Projection**: EPSG:4326, 10 m export scale (native for RGB/NIR; SWIR bands resampled).


## Google Earth Engine Script

```javascript
// ============================================================
// SENTINEL-2 MULTI-BAND: Lunenburg, Nova Scotia (Summer 2025)
// ============================================================

// Lunenburg, NS (UNESCO World Heritage port town)
var lunenburg = ee.Geometry.Polygon([
  [[-64.3547, 44.3531],  // SW (rounded 4 decimals)
   [-64.2767, 44.3531],  // SE
   [-64.2767, 44.3935],  // NE
   [-64.3547, 44.3935],  // NW
   [-64.3547, 44.3531]]  // Closed
]);

Map.centerObject(lunenburg, 13);
Map.addLayer(lunenburg, {color: 'blue'}, 'Lunenburg AOI');

// Sentinel-2 L2A Surface Reflectance (summer 2025)
var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(lunenburg)
  .filterDate('2025-06-01', '2025-08-31')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 15))
  .map(function(image) {
    // Cloud masking (QA60)
    var qa = image.select('QA60');
    var cloudMask = qa.bitwiseAnd(1 << 10).eq(0);
    var cirrusMask = qa.bitwiseAnd(1 << 11).eq(0);
    return image.updateMask(cloudMask)
      .updateMask(cirrusMask)
      .divide(10000)  // Scale to 0-1 reflectance
      .copyProperties(image, ['system:time_start']);
  });

// Median composite
var composite = s2.median().clip(lunenburg);

// SELECT 7 KEY BANDS (10m/20m resolutions)
var bands = composite.select([
  'B1',   // Coastal Aerosol (60m) - water clarity
  'B2',   // Blue (10m)
  'B3',   // Green (10m) 
  'B4',   // Red (10m)
  'B8',   // NIR (10m) - vegetation
  'B11',  // SWIR1 (20m) - moisture
  'B12'   // SWIR2 (20m) - soil/water
], ['coastal', 'blue', 'green', 'red', 'nir', 'swir1', 'swir2']);

// Visualization
var rgbVis = {bands: ['red', 'green', 'blue'], min: 0, max: 0.3};
var falseColorVis = {bands: ['swir1', 'nir', 'green'], min: 0, max: 0.4};

Map.addLayer(bands, rgbVis, 'S2 True Color (10m)');
Map.addLayer(bands, falseColorVis, 'False Color (SWIR-NIR-Green)');

// Info
print('Images used:', s2.size());
print('Area:', lunenburg.area().divide(1e6), 'km² ~7.8km x 4km');
print('Bands exported:', bands.bandNames());

// SINGLE EXPORT - 7 bands (~80MB at 10m)
Export.image.toDrive({
  image: bands.toFloat(),
  description: 'lunenburg_ns_s2_multiband_2025',
  folder: 'GEE_exports',
  fileNamePrefix: 'lunenburg_s2_summer2025_7bands',
  region: lunenburg,
  scale: 10,  // Native 10m for RGB/NIR
  crs: 'EPSG:4326',
  maxPixels: 1e9,
  fileFormat: 'GeoTIFF'
});
```


## Band Details

The exported GeoTIFF uses descriptive band names:

```text
Band 1: coastal  (B1, Coastal aerosol, ~60 m native)
Band 2: blue     (B2, 10 m)
Band 3: green    (B3, 10 m)
Band 4: red      (B4, 10 m)
Band 5: nir      (B8, 10 m)
Band 6: swir1    (B11, 20 m)
Band 7: swir2    (B12, 20 m)
```

- **Scaling**: All bands are reflectance in the 0–1 range.
- **Physical meaning**:
    - `coastal`: Water clarity, atmospheric scattering.
    - `blue/green/red`: True‑color visualization, shallow water, urban vs vegetation.
    - `nir`: Vegetation vigor and canopy density.
    - `swir1/swir2`: Moisture content, burned areas, soil and built‑up discrimination.


## Example Usage: rasterio and Visualization

### 1. Basic Reading \& Inspection

```python
import rasterio

with rasterio.open('lunenburg_s2_summer2025_7bands.tif') as src:
    print("Bands:", src.count)
    print("Shape (rows, cols):", src.height, src.width)
    print("CRS:", src.crs)
    print("Bounds:", src.bounds)

    data = src.read()  # shape: (7, H, W)
```


### 2. True Color Composite (RGB)

```python
import numpy as np
import matplotlib.pyplot as plt

with rasterio.open('lunenburg_s2_summer2025_7bands.tif') as src:
    bands = src.read()  # (7, H, W)

red   = bands[3]  # red
green = bands[2]  # green
blue  = bands[1]  # blue

rgb = np.stack([red, green, blue], axis=-1)
rgb = np.clip(rgb, 0, 0.3) / 0.3

plt.imshow(rgb)
plt.title("Lunenburg Sentinel-2 True Color (10 m)")
plt.axis("off")
plt.show()
```


### 3. False Color (SWIR–NIR–Green)

```python
swir1 = bands[5]  # swir1
nir   = bands[4]  # nir
green = bands[2]  # green

false_color = np.stack([swir1, nir, green], axis=-1)
false_color = np.clip(false_color, 0, 0.4) / 0.4

plt.imshow(false_color)
plt.title("Lunenburg False Color (SWIR–NIR–Green)")
plt.axis("off")
plt.show()
```


### 4. Simple NDVI from Multi‑Band Stack

```python
nir = bands[4]
red = bands[3]

ndvi = (nir - red) / (nir + red + 1e-6)

print("NDVI min/max:", np.nanmin(ndvi), np.nanmax(ndvi))
```


## Learning Objectives

This multi‑band coastal scene is designed to support:

1. **Multi‑band handling**: Reading, indexing, and naming multiple spectral bands in a single GeoTIFF.
2. **Atmospheric \& cloud masking**: Understanding QA60‑based cloud/cirrus masking in Sentinel‑2 L2A.
3. **Visualization**: Building true‑color and SWIR‑based false‑color composites for coastal/urban analysis.
4. **Spectral reasoning**: Interpreting land cover differences using coastal, NIR, and SWIR bands.
5. **Feature engineering**: Deriving NDVI and other indices for vegetation and moisture characterization.

## License and Citation

All source data come from **ESA Sentinel‑2 Level‑2A Surface Reflectance** products accessed via Google Earth Engine.

**Recommended Citation**:
> Lunenburg, Nova Scotia educational Sentinel‑2 multi‑spectral raster derived from Sentinel‑2 L2A (summer 2025), processed via Google Earth Engine.

