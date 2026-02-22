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

// Print info
print('Banff Town Area:', banffTown.area().divide(1e6), 'km² (~8km x 6km)');
print('S2 scenes:', s2.size(), 'L9 scenes:', l9.size());
print('Elevation:', srtm
```