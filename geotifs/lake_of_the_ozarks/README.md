# Lake of the Ozarks GeoTIFF Datasets

This directory contains geospatial datasets for a study area near Lake of the Ozarks in Missouri, downloaded from Google Earth Engine for educational purposes.

## Study Area

All datasets cover a rectangular area of interest (AOI) in Missouri with the following specifications:
- **Longitude**: -92.945° to -92.455° W
- **Latitude**: 37.970° to 38.336° N
- **Coordinate Reference System**: EPSG:32615 (WGS 84 / UTM zone 15N)
- **Spatial Resolution**: 30 meters

## Datasets

### 1. SRTM 30m Digital Elevation Model (DEM)

**File:** `srtm_lake_ozarks_30m_composite.tif`

**Description:** Elevation data derived from the Shuttle Radar Topography Mission (SRTM) void-filled dataset.

**Source:** NASA/USGS SRTM Global 1 arc second (USGS/SRTMGL1_003)

**Data Range:** 200-400 meters elevation

**Google Earth Engine Code:**
```javascript
// 1. Define the rectangular AOI
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
  description: 'SRTM_Lake_Ozarks_30m',
  folder: 'GEE_exports',
  fileNamePrefix: 'srtm_lake_ozarks_30m_composite',
  region: rect,
  scale: 30,
  crs: 'EPSG:32615',
  maxPixels: 1e13
});
```

### 2. Landsat 9 RGB Composite (Scaled)

**File:** `landsat9_lake_ozarks_30m_rgb.tif`

**Description:** Cloud-masked median composite of Landsat 9 surface reflectance data from the 2023 growing season with scaled reflectance values (0.0-1.0 range).

**Source:** USGS Landsat 9 Level-2, Collection 2, Tier 1 (LANDSAT/LC09/C02/T1_L2)

**Temporal Coverage:** April 1, 2023 - October 31, 2023

**Bands Included:**
- Band 4 (Red): 0.64-0.67 μm
- Band 3 (Green): 0.53-0.59 μm
- Band 2 (Blue): 0.45-0.51 μm

**Processing:**
- Cloud cover filter: < 50%
- Surface reflectance scaling: Values multiplied by 0.0000275 and offset by -0.2
- Cloud and shadow masking using QA_PIXEL band (bits 3 and 4)
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

// 5. Visualize scaled RGB composite
Map.addLayer(comp, {bands: ['SR_B4', 'SR_B3', 'SR_B2'], min: 0, max: 0.3}, 'Landsat 9 RGB (Scaled)');

// 6. Export RGB bands (scaled) to Drive
Export.image.toDrive({
  image: comp.select(['SR_B4', 'SR_B3', 'SR_B2']),
  description: 'Landsat9_Lake_Ozarks_RGB_Scaled',
  folder: 'GEE_exports',
  fileNamePrefix: 'landsat9_lake_ozarks_30m_rgb',
  region: rect,
  scale: 30,
  crs: 'EPSG:32615',
  maxPixels: 1e13
});
```

### 3. Landsat 9 Multispectral Composite (Unscaled)

**File:** `landsat9_lake_ozarks_30m_unscaled.tif`

**Description:** Cloud-masked median composite with original Digital Number (DN) values for all optical bands, suitable for custom processing workflows.

**Source:** USGS Landsat 9 Level-2, Collection 2, Tier 1 (LANDSAT/LC09/C02/T1_L2)

**Temporal Coverage:** April 1, 2023 - October 31, 2023

**Bands Included (in order):**
1. Band 1 (Coastal Aerosol): 0.43-0.45 μm
2. Band 2 (Blue): 0.45-0.51 μm
3. Band 3 (Green): 0.53-0.59 μm
4. Band 4 (Red): 0.64-0.67 μm
5. Band 5 (NIR): 0.85-0.88 μm
6. Band 6 (SWIR 1): 1.57-1.65 μm
7. Band 7 (SWIR 2): 2.11-2.29 μm

**Value Range:** 7,273 - 65,535 (DN values with offset)

**Scaling Formula:** To convert to surface reflectance: `SR = DN × 0.0000275 - 0.2`

**Processing:**
- Cloud cover filter: < 50%
- Cloud and shadow masking using QA_PIXEL band
- Median composite
- Original DN values preserved (not scaled)

**Google Earth Engine Code:**
```javascript
// 1. Define rectangular AOI
var rect = ee.Geometry.Rectangle([-92.945, 37.970, -92.455, 38.336], null, false);
Map.centerObject(rect, 11);
Map.addLayer(rect, {color: 'red'}, 'Study Area');

// 2. Temporal window
var start = '2023-04-01';
var end   = '2023-10-31';

// 3. Load Landsat 9 SR with cloud masking (keep original DN values)
var l9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(rect)
  .filterDate(start, end)
  .filter(ee.Filter.lt('CLOUD_COVER', 50))
  .map(function(img) {
    // Cloud/shadow mask using QA_PIXEL (bits 3=cloud, 4=shadow)
    var qa = img.select('QA_PIXEL');
    var cloud = qa.bitwiseAnd(1 << 3).eq(0).and(qa.bitwiseAnd(1 << 4).eq(0));
    // Apply mask but keep original DN values
    return img.select('SR_B[1-7]').updateMask(cloud);
  });

// 4. Median composite (unscaled)
var comp_unscaled = l9.median().clip(rect);

// 5. Visualize (apply scaling only for display)
var scaled_for_display = comp_unscaled.multiply(0.0000275).add(-0.2);
Map.addLayer(scaled_for_display, {bands: ['SR_B4', 'SR_B3', 'SR_B2'], min: 0, max: 0.3}, 'Landsat 9 RGB (Display)');

// 6. Export all 7 optical bands (unscaled DN values) to Drive
Export.image.toDrive({
  image: comp_unscaled,
  description: 'Landsat9_Lake_Ozarks_Unscaled',
  folder: 'GEE_exports',
  fileNamePrefix: 'landsat9_lake_ozarks_30m_unscaled',
  region: rect,
  scale: 30,
  crs: 'EPSG:32615',
  maxPixels: 1e13
});
```

## File Naming Convention

Files follow this pattern: `[sensor]_[location]_[resolution]_[type].tif`

- **sensor**: Data source (e.g., srtm, landsat9)
- **location**: Geographic area (e.g., lake_ozarks)
- **resolution**: Spatial resolution in meters (e.g., 30m)
- **type**: Data characteristics (e.g., composite, rgb, unscaled)

## Usage

These datasets are intended for educational purposes, including:
- Remote sensing analysis and interpretation
- Geospatial data processing tutorials
- Machine learning applications in environmental science
- Terrain analysis and 3D visualization
- Multispectral image analysis and classification
- Spectral index calculation (NDVI, NDWI, etc.)
- Custom band ratio analysis
- GIS workflow demonstrations

## Data Format

All files are provided as GeoTIFF format with the following specifications:

| File | Bands | Data Type | Value Range | File Size |
|------|-------|-----------|-------------|----------|
| `srtm_lake_ozarks_30m_composite.tif` | 1 (Elevation) | Int16 | 200-400 m | ~1.6 MB |
| `landsat9_lake_ozarks_30m_rgb.tif` | 3 (R, G, B) | Float32 | 0.0-1.0 (scaled) | ~48 MB |
| `landsat9_lake_ozarks_30m_unscaled.tif` | 7 (B1-B7) | UInt16 | 7273-65535 (DN) | ~15 MB |

## Example Usage in Python

### Basic Visualization

```python
import rasterio
import matplotlib.pyplot as plt
import numpy as np

# Read SRTM elevation data
with rasterio.open('srtm_lake_ozarks_30m_composite.tif') as src:
    elevation = src.read(1)
    
# Read Landsat RGB composite (scaled)
with rasterio.open('landsat9_lake_ozarks_30m_rgb.tif') as src:
    rgb = src.read([1, 2, 3])  # Read all three bands
    rgb = np.transpose(rgb, (1, 2, 0))  # Reorder for plotting
    rgb = np.clip(rgb, 0, 0.3) / 0.3  # Normalize for display

# Visualize
fig, axes = plt.subplots(1, 2, figsize=(14, 5))
axes[0].imshow(elevation, cmap='terrain')
axes[0].set_title('SRTM Elevation (m)')
axes[0].axis('off')
axes[1].imshow(rgb)
axes[1].set_title('Landsat 9 RGB Composite (2023)')
axes[1].axis('off')
plt.tight_layout()
plt.show()
```

### Working with Unscaled Data

```python
import rasterio
import numpy as np

# Read unscaled Landsat data
with rasterio.open('landsat9_lake_ozarks_30m_unscaled.tif') as src:
    bands = src.read()  # Shape: (7, height, width)
    profile = src.profile
    
# Convert DN values to surface reflectance
sr = bands.astype(np.float32) * 0.0000275 - 0.2

# Extract specific bands (0-indexed)
blue = sr[1]    # Band 2
green = sr[2]   # Band 3
red = sr[3]     # Band 4
nir = sr[4]     # Band 5

# Calculate NDVI (Normalized Difference Vegetation Index)
ndvi = (nir - red) / (nir + red + 1e-8)

# Calculate NDWI (Normalized Difference Water Index)
ndwi = (green - nir) / (green + nir + 1e-8)

# Visualize indices
fig, axes = plt.subplots(1, 2, figsize=(14, 5))
axes[0].imshow(ndvi, cmap='RdYlGn', vmin=-0.5, vmax=0.8)
axes[0].set_title('NDVI (Vegetation Index)')
axes[0].axis('off')
axes[1].imshow(ndwi, cmap='Blues', vmin=-0.5, vmax=0.5)
axes[1].set_title('NDWI (Water Index)')
axes[1].axis('off')
plt.tight_layout()
plt.show()
```

### Creating Custom Band Combinations

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

with rasterio.open('landsat9_lake_ozarks_30m_unscaled.tif') as src:
    bands = src.read()
    
# Convert to surface reflectance
sr = bands.astype(np.float32) * 0.0000275 - 0.2

# False color composite (NIR, Red, Green) for vegetation analysis
false_color = np.dstack([sr[4], sr[3], sr[2]])  # NIR, R, G
false_color = np.clip(false_color, 0, 0.4) / 0.4

# SWIR composite (SWIR2, SWIR1, Red) for geology/moisture
swir_composite = np.dstack([sr[6], sr[5], sr[3]])  # SWIR2, SWIR1, R
swir_composite = np.clip(swir_composite, 0, 0.4) / 0.4

fig, axes = plt.subplots(1, 2, figsize=(14, 5))
axes[0].imshow(false_color)
axes[0].set_title('False Color (NIR-R-G)')
axes[0].axis('off')
axes[1].imshow(swir_composite)
axes[1].set_title('SWIR Composite')
axes[1].axis('off')
plt.tight_layout()
plt.show()
```

## License

These datasets were downloaded from Google Earth Engine for educational purposes. Please refer to the original data sources for licensing information:
- [USGS SRTM](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1) - Public Domain
- [USGS Landsat 9](https://www.usgs.gov/landsat-missions/landsat-9) - Public Domain

## Citation

If you use these datasets in your research or educational materials, please cite the original data sources:

**SRTM:**
> NASA Shuttle Radar Topography Mission (SRTM) (2013). Shuttle Radar Topography Mission (SRTM) Global. Distributed by OpenTopography. https://doi.org/10.5069/G9445JDF

**Landsat 9:**
> U.S. Geological Survey, 2023, Landsat 9 Level-2, Collection 2, Tier 1: U.S. Geological Survey data release, https://doi.org/10.5066/P9OGBGM6

## Suggestions and Best Practices

### When to Use Each Dataset

- **RGB Scaled (`landsat9_lake_ozarks_30m_rgb.tif`)**: 
  - Quick visualization and display
  - True-color image analysis
  - Teaching basic remote sensing concepts
  - Smaller file size for demos

- **Unscaled Multispectral (`landsat9_lake_ozarks_30m_unscaled.tif`)**:
  - Spectral index calculations (NDVI, NDWI, EVI, etc.)
  - Custom band ratio analysis
  - Machine learning feature engineering
  - Advanced multispectral analysis
  - Preserves full radiometric resolution

- **SRTM DEM (`srtm_lake_ozarks_30m_composite.tif`)**:
  - Terrain analysis (slope, aspect, hillshade)
  - Watershed delineation
  - Viewshed analysis
  - 3D visualization
  - Integration with multispectral data for topographic correction

### Additional Analysis Ideas

1. **Land Cover Classification**: Use unscaled data to train supervised classifiers
2. **Water Body Extraction**: Apply NDWI thresholding to identify Lake of the Ozarks
3. **Vegetation Monitoring**: Calculate NDVI to assess forest and agricultural areas
4. **Topographic Analysis**: Combine DEM with optical data for slope-based stratification
5. **Time Series Analysis**: Compare with other Landsat collections for change detection

## Contact

For questions or issues related to these datasets, please open an issue in the main repository.
