# Datasets

This repository contains geospatial datasets for educational purposes, primarily focused on a study area in Missouri.

## Study Area

All datasets cover a rectangular area of interest (AOI) in Missouri with the following coordinates:
- Longitude: -92.945° to -92.455° W
- Latitude: 37.970° to 38.336° N
- Coordinate Reference System: EPSG:32615 (WGS 84 / UTM zone 15N)
- Spatial Resolution: 30 meters

## Datasets

### 1. SRTM 30m Digital Elevation Model (DEM)

**File:** `SRTM_30m_Composite.tif`

**Description:** Elevation data derived from the Shuttle Radar Topography Mission (SRTM) void-filled dataset.

**Source:** NASA/USGS SRTM Global 1 arc second (USGS/SRTMGL1_003)

**Data Range:** 200-400 meters elevation

**Google Earth Engine Code:**
```javascript
// 1. Define the same rectangular AOI
var rect = ee.Geometry.Rectangle([-92.945, 37.970, -92.455, 38.336], null, false);
Map.centerObject(rect, 11);
Map.addLayer(rect, {color: 'red'}, 'Study Area');

// 2. Load SRTM 30m DEM (NASA/USGS void-filled)
var srtm = ee.Image('USGS/SRTMGL1_003').clip(rect);

// 3. Visualize on map
Map.addLayer(srtm, {min: 200, max: 400, palette: ['blue', 'green', 'yellow', 'orange', 'red']}, 'SRTM 30m');

// 4. Export SRTM to Drive (single-band elevation GeoTIFF)
Export.image.toDrive({
  image: srtm,
  description: 'SRTM_30m_Composite',
  folder: 'GEE_exports',
  fileNamePrefix: 'SRTM_30m_Composite',
  region: rect,
  scale: 30,
  crs: 'EPSG:32615',
  maxPixels: 1e13
});
```

### 2. Landsat 9 RGB Composite

**File:** `Landsat9_RGB_Composite.tif`

**Description:** Cloud-masked median composite of Landsat 9 surface reflectance data from the 2023 growing season.

**Source:** USGS Landsat 9 Level-2, Collection 2, Tier 1 (LANDSAT/LC09/C02/T1_L2)

**Temporal Coverage:** April 1, 2023 - October 31, 2023

**Bands Included:**
- Band 4 (Red): 0.64-0.67 μm
- Band 3 (Green): 0.53-0.59 μm
- Band 2 (Blue): 0.45-0.51 μm

**Processing:**
- Cloud cover filter: < 50%
- Surface reflectance scaling: Values multiplied by 0.0000275 and offset by -0.2
- Cloud and shadow masking using QA_PIXEL band
- Median composite to reduce noise and fill gaps

**Google Earth Engine Code:**
```javascript
// 1. Define rectangular AOI and visualization
var rect = ee.Geometry.Rectangle([-92.945, 37.970, -92.455, 38.336], null, false);
Map.centerObject(rect, 11);
Map.addLayer(rect, {color: 'red'}, 'Study Area');

// 2. Temporal window
var start = '2023-04-01';
var end   = '2023-10-31';

// 3. Load Landsat 9 SR, scale bands, and apply robust mask
var l9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(rect)
  .filterDate(start, end)
  .filter(ee.Filter.lt('CLOUD_COVER', 50))  // prefilter
  .map(function(img) {
    // Scale SR bands: multiply by 0.0000275, add -0.2
    var optical = img.select('SR_B.*').multiply(0.0000275).add(-0.2);
    // Cloud/shadow mask using QA_PIXEL (bits 3=cloud, 4=shadow)
    var qa = img.select('QA_PIXEL');
    var cloud = qa.bitwiseAnd(1 << 3).eq(0).and(qa.bitwiseAnd(1 << 4).eq(0));
    return img.addBands(optical, null, true).updateMask(cloud);
  });

// 4. Median composite
var comp = l9.median().clip(rect);

// Export RGB bands only to Drive
Export.image.toDrive({
  image: comp.select(['SR_B4', 'SR_B3', 'SR_B2']),
  description: 'Landsat9_RGB_Composite',
  folder: 'GEE_exports',
  fileNamePrefix: 'Landsat9_RGB_Composite',
  region: rect,
  scale: 30,
  crs: 'EPSG:32615',
  maxPixels: 1e13
});
```

## Usage

These datasets are intended for educational purposes, including:
- Remote sensing analysis
- Geospatial data processing tutorials
- Machine learning applications in environmental science
- Terrain analysis and visualization
- Multispectral image analysis

## Data Format

All files are provided as GeoTIFF format with the following specifications:
- Projection: UTM Zone 15N (EPSG:32615)
- Spatial Resolution: 30 meters
- Data Type: Float32 (surface reflectance) or Int16 (elevation)

## License

These datasets were downloaded from Google Earth Engine for educational purposes. Please refer to the original data sources for licensing information:
- [USGS SRTM](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1)
- [USGS Landsat 9](https://www.usgs.gov/landsat-missions/landsat-9)

## Contact

For questions or issues related to these datasets, please open an issue in this repository.
