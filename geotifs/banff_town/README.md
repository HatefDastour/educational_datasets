# Banff Townsite Multi‑Sensor: Sentinel‑2 + Landsat 9 + SRTM (Summer 2025)

This repository contains three complementary raster datasets for **Banff Townsite** (Bow Valley, Alberta) from Sentinel‑2, Landsat 9, and SRTM, exported as separate GeoTIFFs from Google Earth Engine.
The collection is perfect for teaching multi‑sensor analysis, urban–river–mountain interactions, and rasterio workflows in a compact alpine town setting.

## Study Area

**Banff Town AOI** – Banff townsite, hotels, Bow River corridor, and surrounding forested slopes at the base of Sulphur Mountain.

- **Location (WGS84)**: 51.16°–51.22° N, 115.60°–115.52° W (compact rectangular AOI).  
- **Approximate Area**: Printed by the script as `banffTown.area().divide(1e6)` km² (~8 km × 6 km).  
- **Landscape**: Tourist town, winding Bow River, mixed conifer forest, alpine meadows, and dramatic elevation gradient.
- **Why this AOI?** Small, visually compelling scene with urban, river, forest, and topography in a single ~10 km² area ideal for high‑resolution analysis.

## Dataset Files

The script exports **three separate GeoTIFFs**, one per sensor/product.

| File | Bands | Sensor / Product | Resolution | Data Range | Purpose |
|------|-------|------------------|-----------|-----------|---------|
| `banff_town_s2_summer2025.tif` | 7 (B1,B2,B3,B4,B8,B11,B12) | Sentinel‑2 L2A SR | 10 m | 0–1 reflectance | High‑res multispectral town & river |
| `banff_town_l9_summer2025.tif` | 5 (SR_B2,SR_B3,SR_B4,SR_B5,ST_B10) | Landsat 9 L2 | 30 m | Reflectance (0–1), LST (K) | RGB+NIR+thermal comparison |
| `banff_town_srtm_elevation.tif` | 1 (elevation) | SRTMGL1 DEM | 30 m | Meters above sea level | Alpine terrain context |

- **Sentinel‑2 L2A**: Surface reflectance bands scaled by 1/10,000 and then divided by 10,000 to give 0–1 reflectance.
- **Landsat 9 L2**: Optical SR bands scaled with `0.0000275 × DN + (−0.2)`; ST_B10 converted to land surface temperature in Kelvin via `0.00341802 × DN + 149.0`.
- **SRTMGL1_003**: Global 1‑arc‑second (~30 m) DEM, `elevation` in meters. 

## Google Earth Engine Workflow

```javascript
// ============================================================
// BANFF TOWN MULTI-SENSOR: S2 + L9 + SRTM (2025 Summer)
// ============================================================

// Banff townsite only (Bow Valley, hotels, Bow River)
var banffTown = ee.Geometry.Polygon([
  [[-115.60, 51.16],   // SW (Bow Valley)
   [-115.52, 51.16],   // SE (Canmore direction)
   [-115.52, 51.22],   // NE (Banff Ave)
   [-115.60, 51.22],   // NW (Sulphur Mountain base)
   [-115.60, 51.16]]   // Closed
]);

Map.centerObject(banffTown, 14);
Map.addLayer(banffTown, {color: 'darkgreen'}, 'Banff Town AOI');

// === 1. SENTINEL-2 L2A (10m, 7 bands) ===
var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(banffTown).filterDate('2025-06-01', '2025-08-31')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 15))
  .map(function(img) {
    var qa = img.select('QA60');
    return img.updateMask(qa.bitwiseAnd(1<<10).eq(0).and(qa.bitwiseAnd(1<<11).eq(0)))
      .divide(10000).copyProperties(img, ['system:time_start']);
  });
var s2Comp = s2.median().clip(banffTown);

// === 2. LANDSAT 9 L2A (30m, RGB+NIR+Thermal) ===
var l9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(banffTown).filterDate('2025-06-01', '2025-08-31')
  .filter(ee.Filter.lt('CLOUD_COVER', 15))
  .map(function(img) {
    var qaMask = img.select('QA_PIXEL').bitwiseAnd(parseInt('11111',2)).eq(0);
    var optical = img.select('SR_B.*').multiply(0.0000275).add(-0.2);
    var thermal = img.select('ST_B10').multiply(0.00341802).add(149.0);
    return img.addBands(optical, null, true).addBands(thermal, null, true)
      .updateMask(qaMask);
  });
var l9Comp = l9.median().clip(banffTown);

// === 3. SRTM DEM (30m) ===
var srtm = ee.Image('USGS/SRTMGL1_003').select('elevation').clip(banffTown);

// Urban + river visualization (perfect for small town)
var s2rgb = {bands: ['B4','B3','B2'], min: 0, max: 0.25};
var riverVis = {bands: ['B11','B8','B2'], min: 0, max: 0.35};  // SWIR-NIR-Blue

Map.addLayer(s2Comp, s2rgb, 'S2 True Color (10m)');
Map.addLayer(s2Comp, riverVis, 'S2 River Index (Bow River)');
Map.addLayer(l9Comp, s2rgb, 'Landsat 9 (30m)');
Map.addLayer(srtm, {min: 1350, max: 1550, palette: ['#0066CC','#00AA66','#FFCC00']}, 'SRTM Elevation');

print('AOI area (km²):', banffTown.area().divide(1e6));
print('S2 scenes used:', s2.size());
print('L9 scenes used:', l9.size());

// === EXPORTS ===
// 1. Sentinel-2 (7 bands, 10 m)
var s2Export = s2Comp.select(
  ['B1','B2','B3','B4','B8','B11','B12'],
  ['coastal','blue','green','red','nir','swir1','swir2']
);
Export.image.toDrive({
  image: s2Export.toFloat(),
  description: 'banff_town_s2_summer2025',
  folder: 'GEE_exports',
  fileNamePrefix: 'banff_town_s2_summer2025',
  region: banffTown, scale: 10, crs: 'EPSG:4326',
  maxPixels: 1e9, fileFormat: 'GeoTIFF'
});

// 2. Landsat 9 (5 bands, 30 m)
var l9Export = l9Comp.select(
  ['SR_B2','SR_B3','SR_B4','SR_B5','ST_B10'],
  ['blue','green','red','nir','thermal']
);
Export.image.toDrive({
  image: l9Export.toFloat(),
  description: 'banff_town_l9_summer2025',
  folder: 'GEE_exports',
  fileNamePrefix: 'banff_town_l9_summer2025',
  region: banffTown, scale: 30, crs: 'EPSG:4326',
  maxPixels: 1e9, fileFormat: 'GeoTIFF'
});

// 3. SRTM DEM (30 m)
Export.image.toDrive({
  image: srtm.toFloat(),
  description: 'banff_town_srtm_elevation',
  folder: 'GEE_exports',
  fileNamePrefix: 'banff_town_srtm_elevation',
  region: banffTown, scale: 30, crs: 'EPSG:4326',
  maxPixels: 1e8, fileFormat: 'GeoTIFF'
});
```

## Band Details

### Sentinel‑2 7‑Band Stack (`banff_town_s2_summer2025.tif`)

```text
Band 1: coastal  (B1, 60 m native → resampled 10 m; coastal aerosol / haze)
Band 2: blue     (B2, 10 m; true-colour blue, shallow water)
Band 3: green    (B3, 10 m; vegetation, true-colour green)
Band 4: red      (B4, 10 m; vegetation/chlorophyll, true-colour red)
Band 5: nir      (B8, 10 m; vegetation vigour, biomass)
Band 6: swir1    (B11, 20 m native → resampled 10 m; moisture, burned areas)
Band 7: swir2    (B12, 20 m native → resampled 10 m; soil, geology)
```
All bands are surface reflectance scaled to 0–1.

### Landsat 9 5‑Band Stack (`banff_town_l9_summer2025.tif`)

```text
Band 1: blue     (SR_B2, 30 m; 0.45–0.51 µm)
Band 2: green    (SR_B3, 30 m; 0.53–0.59 µm)
Band 3: red      (SR_B4, 30 m; 0.64–0.67 µm)
Band 4: nir      (SR_B5, 30 m; 0.85–0.88 µm)
Band 5: thermal  (ST_B10, 30 m; Land Surface Temperature in Kelvin)
```
Optical bands: surface reflectance 0–1; thermal: Kelvin (~270–310 K for summer).

### SRTM DEM (`banff_town_srtm_elevation.tif`)

Single `elevation` band in metres above sea level (~1,380–2,500 m across the AOI).

## Python Analysis Examples

### 1. Reading and Displaying All Three Datasets

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read Sentinel-2 true color
with rasterio.open('banff_town_s2_summer2025.tif') as s2_src:
    s2 = s2_src.read()  # (7, H, W)
    print(f"S2 bands: {s2_src.count}, shape: {s2_src.shape}, CRS: {s2_src.crs}")

# Read Landsat 9
with rasterio.open('banff_town_l9_summer2025.tif') as l9_src:
    l9 = l9_src.read()  # (5, H, W)
    print(f"L9 bands: {l9_src.count}, shape: {l9_src.shape}")

# Read SRTM DEM
with rasterio.open('banff_town_srtm_elevation.tif') as srtm_src:
    dem = srtm_src.read(1)  # (H, W)
    print(f"DEM elevation range: {dem.min():.0f}–{dem.max():.0f} m")

# Visualize
fig, axes = plt.subplots(1, 3, figsize=(18, 6))

# S2 true color (red=B4, green=B3, blue=B2 → indices 3,2,1)
s2_rgb = np.stack([s2[3], s2[2], s2[1]], axis=-1)
s2_rgb = np.clip(s2_rgb, 0, 0.25) / 0.25
axes[0].imshow(s2_rgb)
axes[0].set_title('Sentinel-2 True Color (10 m)', fontsize=12)
axes[0].axis('off')

# L9 true color (red=B4, green=B3, blue=B2 → indices 2,1,0)
l9_rgb = np.stack([l9[2], l9[1], l9[0]], axis=-1)
l9_rgb = np.clip(l9_rgb, 0, 0.25) / 0.25
axes[1].imshow(l9_rgb)
axes[1].set_title('Landsat 9 True Color (30 m)', fontsize=12)
axes[1].axis('off')

# DEM
im = axes[2].imshow(dem, cmap='terrain', vmin=1350, vmax=2500)
axes[2].set_title('SRTM Elevation (m)', fontsize=12)
axes[2].axis('off')
plt.colorbar(im, ax=axes[2], fraction=0.046, label='Elevation (m)')

plt.suptitle('Banff Town – Multi-Sensor Overview', fontsize=16)
plt.tight_layout()
plt.show()
```

### 2. NDVI Comparison: Sentinel-2 vs. Landsat 9

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

with rasterio.open('banff_town_s2_summer2025.tif') as s2_src:
    s2 = s2_src.read()

with rasterio.open('banff_town_l9_summer2025.tif') as l9_src:
    l9 = l9_src.read()

# S2 NDVI: NIR=band5(idx4), Red=band4(idx3)
s2_nir, s2_red = s2[4], s2[3]
ndvi_s2 = (s2_nir - s2_red) / (s2_nir + s2_red + 1e-6)

# L9 NDVI: NIR=band4(idx3), Red=band3(idx2)
l9_nir, l9_red = l9[3], l9[2]
ndvi_l9 = (l9_nir - l9_red) / (l9_nir + l9_red + 1e-6)

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

im0 = axes[0].imshow(ndvi_s2, cmap='RdYlGn', vmin=-0.2, vmax=0.8)
axes[0].set_title('NDVI – Sentinel-2 (10 m)', fontsize=12)
axes[0].axis('off')
plt.colorbar(im0, ax=axes[0], fraction=0.046)

im1 = axes[1].imshow(ndvi_l9, cmap='RdYlGn', vmin=-0.2, vmax=0.8)
axes[1].set_title('NDVI – Landsat 9 (30 m)', fontsize=12)
axes[1].axis('off')
plt.colorbar(im1, ax=axes[1], fraction=0.046)

plt.suptitle('Banff Town – NDVI Sensor Comparison', fontsize=16)
plt.tight_layout()
plt.show()

print(f"S2 NDVI  mean: {np.nanmean(ndvi_s2):.3f}")
print(f"L9 NDVI  mean: {np.nanmean(ndvi_l9):.3f}")
```

### 3. Thermal Band Analysis (Land Surface Temperature)

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

with rasterio.open('banff_town_l9_summer2025.tif') as src:
    l9 = src.read()

# Band 5 (index 4) is thermal in Kelvin
lst_k = l9[4]
lst_c = lst_k - 273.15  # Convert to Celsius

# Mask fill values (≈ 0 K)
lst_c = np.where(lst_k < 100, np.nan, lst_c)

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

im0 = axes[0].imshow(lst_c, cmap='RdYlBu_r', vmin=5, vmax=35)
axes[0].set_title('Land Surface Temperature (°C)', fontsize=12)
axes[0].axis('off')
plt.colorbar(im0, ax=axes[0], fraction=0.046, label='°C')

# LST histogram
axes[1].hist(lst_c[~np.isnan(lst_c)].flatten(), bins=50,
             color='coral', edgecolor='black', alpha=0.8)
axes[1].set_xlabel('Temperature (°C)', fontsize=11)
axes[1].set_ylabel('Frequency', fontsize=11)
axes[1].set_title('LST Distribution – Banff Town', fontsize=12)
axes[1].grid(True, alpha=0.3)

plt.suptitle('Banff Town – Landsat 9 Thermal Analysis', fontsize=16)
plt.tight_layout()
plt.show()

print(f"Mean LST: {np.nanmean(lst_c):.1f} °C")
print(f"Min  LST: {np.nanmin(lst_c):.1f} °C")
print(f"Max  LST: {np.nanmax(lst_c):.1f} °C")
```

## Learning Objectives

1. **Multi-sensor handling**: Read and compare Sentinel-2 (10 m) and Landsat 9 (30 m) rasters for the same scene.
2. **Resolution differences**: Observe how spatial resolution affects feature visibility (town streets, river banks).
3. **Spectral indices**: Calculate NDVI from both sensors and compare results.
4. **Thermal remote sensing**: Extract and interpret Land Surface Temperature from Landsat 9 ST_B10.
5. **DEM integration**: Combine elevation context with optical data (e.g., elevation vs. NDVI profile).
6. **Rasterio workflows**: Practice multi-band reading, scaling, and visualisation in Python.

## License and Citation

Source data (all public domain):
- **Sentinel-2 L2A**: ESA Copernicus Programme via Google Earth Engine (`COPERNICUS/S2_SR_HARMONIZED`)
- **Landsat 9 L2**: USGS Landsat 9 Level-2, Collection 2, Tier 1 (`LANDSAT/LC09/C02/T1_L2`)
- **SRTMGL1_003**: NASA Shuttle Radar Topography Mission global 1-arc-second DEM

**Recommended Citation:**
> Banff Townsite Multi-Sensor Educational Dataset (Summer 2025). Derived from Sentinel-2 L2A, Landsat 9 Collection 2, and SRTMGL1 products via Google Earth Engine.