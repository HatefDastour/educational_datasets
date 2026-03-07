# Utah Lake GeoTIFF Dataset

This directory contains multispectral satellite data for Utah Lake, including Landsat 9, Landsat time series (2020-2024), and Sentinel-2 monthly composites from Google Earth Engine. Designed for educational purposes in water quality monitoring, temporal analysis, and large lake studies.

## Study Area

Utah Lake is a large, shallow freshwater lake in north-central Utah:
- **Longitude**: -111.95° to -111.64° W
- **Latitude**: 40.01° to 40.37° N
- **Coordinate Reference System**: EPSG:4326 (WGS 84)
- **Spatial Resolution**: 30m (Landsat), 10m (Sentinel-2)
- **Area Coverage**: Approximately 31 km × 40 km
- **Key Features**: Shallow (avg. 3m depth), high turbidity, eutrophic, urban surroundings (Provo-Orem)

## Datasets Overview

### Landsat 9 Summer 2024 Composite (8 bands)
**File**: `utah_lake_l9_summer_2024.tif`  
**Description**: Cloud-masked median composite (June-Aug 2024) with 7 optical + 1 thermal band.  
**Bands**: Coastal, Blue, Green, Red, NIR, SWIR1, SWIR2, Thermal  
**Size**: ~40 MB | **Use**: Water quality, thermal analysis 

### Landsat Annual Summer Composites (2020-2024)
**Files**: `utah_lake_landsat_summer_YYYY.tif` (5 files)  
**Description**: Annual summer median composites (L8/L9 merged, June-Aug each year).  
**Bands**: Same 8 as Landsat 9  
**Size**: ~40 MB each | **Use**: Multi-year trends, change detection

### Sentinel-2 Monthly Composites (Apr-Oct 2024)
**Files**: `utah_lake_s2_2024_30m_MM.tif` (7 files, MM=04-10)  
**Description**: 30m monthly medians during growing season, cloud-masked.  
**Bands**: Coastal (B1), Blue (B2), Green (B3), Red (B4), NIR (B8), SWIR1 (B11), SWIR2 (B12)  
**Size**: ~40 MB each | **Use**: High-res temporal analysis, fine-scale features

## Dataset Summary

| Dataset | Files | Bands | Resolution | Temporal Coverage | Size (per file) | Primary Use |
|---------|-------|-------|------------|-------------------|-----------------|-------------|
| Landsat 9 Summer 2024 | 1 | 8 | 30m | Jun-Aug 2024 | ~40 MB | Baseline water quality |
| Landsat Time Series | 5 | 8 | 30m | Summers 2020-2024 | ~40 MB | Trend analysis |
| Sentinel-2 Monthly | 7 | 7 | 10m | Apr-Oct 2024 | ~150 MB | High-res phenology |

## Google Earth Engine Scripts

### Script 1: Landsat 9 Summer 2024

```javascript
// ============================================================
// GOOGLE EARTH ENGINE SCRIPT 1: LANDSAT 9 SUMMER 2024
// ============================================================

var utahLake = ee.Geometry.Rectangle([
  -111.95, 40.37,  // west, south
  -111.64, 40.01   // east, north
]);

Map.centerObject(utahLake, 11);

function maskLandsatSR(image) {
  var qaMask = image.select('QA_PIXEL').bitwiseAnd(parseInt('11111', 2)).eq(0);
  var saturationMask = image.select('QA_RADSAT').eq(0);
  var opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
  var thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0);
  return image.addBands(opticalBands, null, true)
      .addBands(thermalBands, null, true)
      .updateMask(qaMask)
      .updateMask(saturationMask);
}

var l9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(utahLake)
  .filterDate('2024-06-01', '2024-08-31')
  .filter(ee.Filter.lt('CLOUD_COVER', 20))
  .map(maskLandsatSR)
  .median()
  .clip(utahLake);

var rgbVis = {bands: ['SR_B4', 'SR_B3', 'SR_B2'], min: 0, max: 0.3};
Map.addLayer(l9, rgbVis, 'L9 Summer 2024');

var exportImg = l9.select(
  ['SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7', 'ST_B10'],
  ['coastal', 'blue', 'green', 'red', 'nir', 'swir1', 'swir2', 'thermal']
);

Export.image.toDrive({
  image: exportImg.toFloat(),
  description: 'utah_lake_l9_summer_2024',
  folder: 'earth_engine_exports',
  fileNamePrefix: 'utah_lake_l9_summer_2024',
  region: utahLake,
  scale: 30,
  crs: 'EPSG:4326',
  maxPixels: 1e9,
  fileFormat: 'GeoTIFF'
});
```

### Script 2: Sentinel-2 Multi-Temporal (Apr-Oct 2024)
```javascript
// ============================================================
// GOOGLE EARTH ENGINE SCRIPT 2: SENTINEL-2 MULTI-TEMPORAL
// ============================================================

var utahLake = ee.Geometry.Rectangle([
  -111.95, 40.37,  // west, south
  -111.64, 40.01   // east, north
]);

Map.centerObject(utahLake, 11);

function maskS2clouds(image) {
  var qa = image.select('QA60');
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
  return image.updateMask(mask)
      .divide(10000)
      .copyProperties(image, ['system:time_start']);
}

function createMonthlyComposite(year, month) {
  var startDate = ee.Date.fromYMD(year, month, 1);
  var endDate = startDate.advance(1, 'month');
  var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
    .filterBounds(utahLake)
    .filterDate(startDate, endDate)
    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
    .map(maskS2clouds)
    .median()
    .clip(utahLake);
  return s2.set('month', month).set('year', year);
}

var months = [4, 5, 6, 7, 8, 9, 10];
var year = 2024;
var monthlyImages = months.map(function(month) {
  return createMonthlyComposite(year, month);
});
var collection = ee.ImageCollection.fromImages(monthlyImages);

var s2Vis = {bands: ['B4', 'B3', 'B2'], min: 0, max: 0.3};
months.forEach(function(month) {
  var monthName = ee.List(['', 'Jan', 'Feb', 'Mar', 'Apr', 'May', 
                           'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'])
                    .get(month);
  var img = ee.Image(collection.filter(ee.Filter.eq('month', month)).first());
  Map.addLayer(img, s2Vis, 'S2 ' + monthName + ' 2024', false);
});

months.forEach(function(month) {
  var monthStr = ee.Number(month).format('%02d').getInfo();
  var img = ee.Image(collection.filter(ee.Filter.eq('month', month)).first());
  var exportImg = img.select(
    ['B1', 'B2', 'B3', 'B4', 'B8', 'B11', 'B12'],
    ['coastal', 'blue', 'green', 'red', 'nir', 'swir1', 'swir2']
  );
  Export.image.toDrive({
    image: exportImg.toFloat(),
    description: 'utah_lake_s2_2024_' + monthStr,
    folder: 'earth_engine_exports',
    fileNamePrefix: 'utah_lake_s2_2024_' + monthStr,
    region: utahLake,
    scale: 10,
    crs: 'EPSG:4326',
    maxPixels: 1e9,
    fileFormat: 'GeoTIFF'
  });
});
print('Export tasks created for', months.length, 'months');
```

### Script 3: Landsat Time Series (2020-2024)
```javascript
// ============================================================
// GOOGLE EARTH ENGINE SCRIPT 3: LANDSAT TIME SERIES 2020-2024
// ============================================================

var utahLake = ee.Geometry.Rectangle([
  -111.95, 40.37,  // west, south
  -111.64, 40.01   // east, north
]);

Map.centerObject(utahLake, 11);

function maskLandsatSR(image) {
  var qaMask = image.select('QA_PIXEL').bitwiseAnd(parseInt('11111', 2)).eq(0);
  var saturationMask = image.select('QA_RADSAT').eq(0);
  var opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
  var thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0);
  return image.addBands(opticalBands, null, true)
      .addBands(thermalBands, null, true)
      .updateMask(qaMask)
      .updateMask(saturationMask);
}

function createAnnualSummer(year) {
  var startDate = ee.Date.fromYMD(year, 6, 1);
  var endDate = ee.Date.fromYMD(year, 8, 31);
  var l8 = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2');
  var l9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2');
  var landsat = l8.merge(l9)
    .filterBounds(utahLake)
    .filterDate(startDate, endDate)
    .filter(ee.Filter.lt('CLOUD_COVER', 20))
    .map(maskLandsatSR)
    .median()
    .clip(utahLake);
  return landsat.set('year', year);
}

var years = [2020, 2021, 2022, 2023, 2024];
var annualImages = years.map(function(year) {
  return createAnnualSummer(year);
});
var timeSeries = ee.ImageCollection.fromImages(annualImages);

var rgbVis = {bands: ['SR_B4', 'SR_B3', 'SR_B2'], min: 0, max: 0.3};
years.forEach(function(year) {
  var img = ee.Image(timeSeries.filter(ee.Filter.eq('year', year)).first());
  Map.addLayer(img, rgbVis, 'Summer ' + year, false);
});

years.forEach(function(year) {
  var img = ee.Image(timeSeries.filter(ee.Filter.eq('year', year)).first());
  var exportImg = img.select(
    ['SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7', 'ST_B10'],
    ['coastal', 'blue', 'green', 'red', 'nir', 'swir1', 'swir2', 'thermal']
  );
  Export.image.toDrive({
    image: exportImg.toFloat(),
    description: 'utah_lake_landsat_summer_' + year,
    folder: 'earth_engine_exports',
    fileNamePrefix: 'utah_lake_landsat_summer_' + year,
    region: utahLake,
    scale: 30,
    crs: 'EPSG:4326',
    maxPixels: 1e9,
    fileFormat: 'GeoTIFF'
  });
});
print('Export tasks created for', years.length, 'years');
```

## Educational Applications

- **Temporal Analysis**: Track turbidity/algal bloom trends (Landsat 2020-2024)
- **High-Resolution Monitoring**: Monthly phenology at 10m (Sentinel-2 2024)
- **Multi-Sensor Fusion**: Combine Landsat thermal + Sentinel-2 optical
- **Change Detection**: Urban expansion, water level fluctuations
- **Algorithm Testing**: Validate water quality indices across sensors/years

## Example Usage Extensions

Add to existing Python examples:

```python
# Time series analysis example
years = ['2020', '2021', '2022', '2023', '2024']
turbidities = []
for year in years:
    with rasterio.open(f'utah_lake_landsat_summer_{year}.tif') as src:
        bands = src.read()
        green, red = bands[2], bands[3]
        turbidity = np.nanmean(red / (green + 1e-8))
        turbidities.append(turbidity)
        print(f"{year} mean turbidity: {turbidity:.3f}")

# Plot trend
plt.plot(years, turbidities, marker='o')
plt.title('Utah Lake Turbidity Trend (2020-2024)')
plt.ylabel('Red/Green Ratio')
plt.xticks(rotation=45)
plt.grid(True, alpha=0.3)
plt.show()
```

## License & Citation

Public domain (USGS Landsat, ESA Sentinel-2). Cite original sources:
- Landsat: [DOI:10.5066/P9OGBGM6](https://doi.org/10.5066/P9OGBGM6)
- Sentinel-2: Copernicus Sentinel data (2024)