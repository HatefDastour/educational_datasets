# Mark Twain Lake GeoTIFF Datasets

This directory contains geospatial datasets for Mark Twain Lake in Northeast Missouri, downloaded from Google Earth Engine for educational purposes focused on raster fundamentals and basic remote sensing analysis.

## Study Area

Mark Twain Lake is a reservoir in Northeast Missouri with the following specifications:
- **Longitude**: -91.947° to -91.620° W
- **Latitude**: 39.388° to 39.589° N
- **Coordinate Reference System**: EPSG:32615 (WGS 84 / UTM zone 15N)
- **Spatial Resolution**: 30 meters
- **Area Coverage**: Approximately 33 km × 22 km
- **Key Features**: Large reservoir, surrounding forests, agricultural areas

## Datasets

This collection includes three complementary raster datasets designed for teaching raster data fundamentals:

### 1. Landsat 9 RGB + NIR (Multi-band)

**File:** `mark_twain_l9_rgb_nir_2025.tif`

**Description:** Cloud-free median composite with four scaled surface reflectance bands from July 2025, ideal for teaching multi-band raster operations.

**Source:** USGS Landsat 9 Level-2, Collection 2, Tier 1 (LANDSAT/LC09/C02/T1_L2)

**Temporal Coverage:** July 1-31, 2025 (Peak summer season)

**Bands Included (in order):**
1. Band 2 (Blue): 0.45-0.51 μm
2. Band 3 (Green): 0.53-0.59 μm
3. Band 4 (Red): 0.64-0.67 μm
4. Band 5 (NIR): 0.85-0.88 μm

**Data Characteristics:**
- **Value Range**: 0.0 - ~0.5 (scaled surface reflectance)
- **Data Type**: Float32
- **Processing**: Cloud cover < 10%, USGS scaling applied (DN × 0.0000275 - 0.2)

### 2. SRTM Digital Elevation Model (Single-band)

**File:** `mark_twain_srtm_dem.tif`

**Description:** Single-band elevation raster from SRTM, useful for teaching terrain analysis and single-band operations.

**Source:** NASA/USGS SRTM Global 1 arc second (USGS/SRTMGL1_003)

**Data Characteristics:**
- **Value Range**: 150-300 meters elevation
- **Data Type**: Float32
- **Band**: 1 (Elevation)

### 3. NDVI - Normalized Difference Vegetation Index (Single-band)

**File:** `mark_twain_ndvi_2025.tif`

**Description:** Pre-calculated NDVI from July 2025 Landsat 9 data, demonstrating a derived single-band product.

**Formula:** `NDVI = (NIR - Red) / (NIR + Red)`

**Data Characteristics:**
- **Value Range**: -0.2 to 0.8 (normalized index)
- **Data Type**: Float32
- **Interpretation**:
  - < 0: Water bodies
  - 0.0-0.2: Bare soil, urban areas
  - 0.2-0.4: Sparse vegetation, grasslands
  - 0.4-0.6: Moderate vegetation
  - > 0.6: Dense vegetation, healthy forests

## Google Earth Engine Code

```javascript
// Practice 5.02: Mark Twain Lake Raster Basics
// Exports 3 simple rasters for rasterio fundamentals

// MARK TWAIN LAKE: Northeast Missouri reservoir
var markTwain = ee.Geometry.Rectangle([-91.947, 39.388, -91.620, 39.589]);

// 1. Landsat 9 Surface Reflectance (Multi-band RGB + NIR)
var l9_summer = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(markTwain)
  .filterDate('2025-07-01', '2025-07-31')
  .filter(ee.Filter.lt('CLOUD_COVER', 10))
  .median()
  .clip(markTwain);

// Scale SR data (USGS standard: multiply by 0.0000275 - 0.2)
var l9_scaled = l9_summer.multiply(0.0000275).subtract(0.2)
  .select(['SR_B2','SR_B3','SR_B4','SR_B5'])  // Blue,Green,Red,NIR
  .toFloat();

// 2. SRTM DEM (Single-band elevation)
var srtm = ee.Image('USGS/SRTMGL1_003')
  .select('elevation')
  .clip(markTwain)
  .toFloat();

// 3. NDVI (Single-band derived product)
var ndvi = l9_summer.normalizedDifference(['SR_B5','SR_B4']).rename('NDVI')
  .clip(markTwain)
  .toFloat();

// VISUAL PREVIEW
Map.centerObject(markTwain, 11);
Map.addLayer(l9_scaled, {bands:['SR_B4','SR_B3','SR_B2'], min:0, max:0.3}, 'L9 RGB');
Map.addLayer(srtm, {min:150, max:300, palette:['blue','green','yellow','red']}, 'SRTM');
Map.addLayer(ndvi, {min:-0.2, max:0.8, palette:['red','yellow','green']}, 'NDVI');

// EXPORT TO GOOGLE DRIVE (Tasks → Run after preview)
Export.image.toDrive({
  image: l9_scaled, scale:30, region:markTwain, maxPixels:1e9,
  description: 'mark_twain_l9_rgb_nir_2025', folder: 'GEE_exports'
});
Export.image.toDrive({
  image: srtm, scale:30, region:markTwain, maxPixels:1e9,
  description: 'mark_twain_srtm_dem', folder: 'GEE_exports'
});
Export.image.toDrive({
  image: ndvi, scale:30, region:markTwain, maxPixels:1e9,
  description: 'mark_twain_ndvi_2025', folder: 'GEE_exports'
});
```

## Dataset Summary

| File | Bands | Data Type | Value Range | Size (approx) | Use Case |
|------|-------|-----------|-------------|---------------|----------|
| `mark_twain_l9_rgb_nir_2025.tif` | 4 (BGRN) | Float32 | 0.0-0.5 | ~25 MB | Multi-band operations, RGB visualization |
| `mark_twain_srtm_dem.tif` | 1 (Elev) | Float32 | 150-300 m | ~3 MB | Terrain analysis, single-band operations |
| `mark_twain_ndvi_2025.tif` | 1 (NDVI) | Float32 | -0.2-0.8 | ~3 MB | Vegetation analysis, derived products |

## Educational Use

These datasets are specifically designed for teaching:

### Rasterio Fundamentals
1. **Reading single vs. multi-band rasters**
2. **Understanding raster metadata** (CRS, transform, bounds)
3. **Band indexing and slicing**
4. **Data type handling** (Float32 vs. Int16)
5. **Visualization with matplotlib**

### Basic Operations
1. **Band math** (calculate indices from multi-band data)
2. **Masking and thresholding** (water extraction using NDVI)
3. **Reprojection and resampling**
4. **Subsetting and cropping**
5. **Statistics calculation** (mean, std, histograms)

### Analysis Workflows
1. **Water body delineation** (using NDVI < 0)
2. **Hillshade generation** (from DEM)
3. **False color composites** (NIR-R-G)
4. **Elevation profiling** (transect extraction)
5. **Multi-dataset integration** (overlaying DEM with optical data)

## Example Usage in Python

### Example 1: Reading and Visualizing All Datasets

```python
import rasterio
import matplotlib.pyplot as plt
import numpy as np
from rasterio.plot import show

# Read multi-band RGB+NIR
with rasterio.open('mark_twain_l9_rgb_nir_2025.tif') as src:
    rgb_nir = src.read()  # Shape: (4, height, width)
    print(f"RGB+NIR: {src.count} bands, {src.dtypes[0]}, CRS: {src.crs}")

# Read single-band DEM
with rasterio.open('mark_twain_srtm_dem.tif') as src:
    dem = src.read(1)  # Shape: (height, width)
    print(f"DEM: {src.count} band, {src.dtypes[0]}, Range: {dem.min():.1f}-{dem.max():.1f}m")

# Read single-band NDVI
with rasterio.open('mark_twain_ndvi_2025.tif') as src:
    ndvi = src.read(1)
    print(f"NDVI: {src.count} band, {src.dtypes[0]}, Range: {ndvi.min():.2f}-{ndvi.max():.2f}")

# Visualize all three
fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# True color (bands are in B-G-R-NIR order, indices 2,1,0)
rgb = np.dstack([rgb_nir[2], rgb_nir[1], rgb_nir[0]])  # Red, Green, Blue
rgb = np.clip(rgb, 0, 0.3) / 0.3  # Normalize for display
axes[0].imshow(rgb)
axes[0].set_title('Landsat 9 True Color (July 2025)', fontsize=12)
axes[0].axis('off')

# DEM
im1 = axes[1].imshow(dem, cmap='terrain', vmin=150, vmax=300)
axes[1].set_title('SRTM Elevation (m)', fontsize=12)
axes[1].axis('off')
plt.colorbar(im1, ax=axes[1], fraction=0.046, label='Elevation (m)')

# NDVI
im2 = axes[2].imshow(ndvi, cmap='RdYlGn', vmin=-0.2, vmax=0.8)
axes[2].set_title('NDVI (Vegetation Index)', fontsize=12)
axes[2].axis('off')
plt.colorbar(im2, ax=axes[2], fraction=0.046, label='NDVI')

plt.suptitle('Mark Twain Lake - Multi-Dataset Overview', fontsize=16, y=0.98)
plt.tight_layout()
plt.show()
```

### Example 2: Water Body Extraction from NDVI

```python
import rasterio
import matplotlib.pyplot as plt
import numpy as np

# Read NDVI
with rasterio.open('mark_twain_ndvi_2025.tif') as src:
    ndvi = src.read(1)
    profile = src.profile

# Extract water bodies (NDVI < 0)
water_mask = (ndvi < 0).astype(np.uint8)

# Calculate water area
pixel_area = 30 * 30  # 30m resolution
water_pixels = np.sum(water_mask)
water_area_km2 = (water_pixels * pixel_area) / 1e6

print(f"Water body area: {water_area_km2:.2f} km²")
print(f"Water pixels: {water_pixels:,}")

# Visualize
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

axes[0].imshow(ndvi, cmap='RdYlGn', vmin=-0.2, vmax=0.8)
axes[0].set_title('NDVI', fontsize=13)
axes[0].axis('off')

axes[1].imshow(water_mask, cmap='Blues_r')
axes[1].set_title(f'Water Body Mask\nArea: {water_area_km2:.2f} km²', fontsize=13)
axes[1].axis('off')

plt.tight_layout()
plt.show()

# Optional: Export water mask
profile.update(dtype=rasterio.uint8, count=1, nodata=255)
with rasterio.open('mark_twain_water_mask.tif', 'w', **profile) as dst:
    dst.write(water_mask, 1)
```

### Example 3: Creating False Color Composite

```python
import rasterio
import matplotlib.pyplot as plt
import numpy as np

# Read RGB+NIR data
with rasterio.open('mark_twain_l9_rgb_nir_2025.tif') as src:
    bands = src.read()  # Shape: (4, height, width)
    # Band order: Blue(0), Green(1), Red(2), NIR(3)

# Create different composites
composites = {
    'True Color (R-G-B)': np.dstack([bands[2], bands[1], bands[0]]),
    'False Color (NIR-R-G)': np.dstack([bands[3], bands[2], bands[1]]),
    'False Color (NIR-G-B)': np.dstack([bands[3], bands[1], bands[0]]),
    'Color Infrared (R-NIR-G)': np.dstack([bands[2], bands[3], bands[1]])
}

fig, axes = plt.subplots(2, 2, figsize=(14, 12))
axes = axes.flatten()

for idx, (title, composite) in enumerate(composites.items()):
    # Normalize for display
    comp_norm = np.clip(composite, 0, 0.3) / 0.3
    axes[idx].imshow(comp_norm)
    axes[idx].set_title(title, fontsize=12)
    axes[idx].axis('off')

plt.suptitle('Mark Twain Lake - Band Combinations', fontsize=16)
plt.tight_layout()
plt.show()
```

### Example 4: Terrain Analysis with DEM

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt
from scipy.ndimage import sobel

# Read DEM
with rasterio.open('mark_twain_srtm_dem.tif') as src:
    dem = src.read(1)
    transform = src.transform

# Calculate slope (gradient magnitude)
slope_x = sobel(dem, axis=1) / (8 * transform[0])  # dx
slope_y = sobel(dem, axis=0) / (8 * abs(transform[4]))  # dy
slope = np.sqrt(slope_x**2 + slope_y**2)
slope_degrees = np.degrees(np.arctan(slope))

# Calculate aspect (gradient direction)
aspect = np.degrees(np.arctan2(-slope_y, slope_x))
aspect = (90 - aspect) % 360  # Convert to compass bearing

# Simple hillshade
azimuth = 315  # Light from NW
altitude = 45
slope_rad = np.radians(slope_degrees)
aspect_rad = np.radians(aspect)
shaded = np.sin(np.radians(altitude)) * np.sin(slope_rad) + \
         np.cos(np.radians(altitude)) * np.cos(slope_rad) * \
         np.cos(np.radians(azimuth) - aspect_rad)

# Visualize
fig, axes = plt.subplots(2, 2, figsize=(14, 12))

axes[0, 0].imshow(dem, cmap='terrain')
axes[0, 0].set_title('Elevation (m)', fontsize=12)
axes[0, 0].axis('off')

im1 = axes[0, 1].imshow(slope_degrees, cmap='YlOrRd', vmin=0, vmax=15)
axes[0, 1].set_title('Slope (degrees)', fontsize=12)
axes[0, 1].axis('off')
plt.colorbar(im1, ax=axes[0, 1], fraction=0.046)

im2 = axes[1, 0].imshow(aspect, cmap='hsv', vmin=0, vmax=360)
axes[1, 0].set_title('Aspect (degrees)', fontsize=12)
axes[1, 0].axis('off')
plt.colorbar(im2, ax=axes[1, 0], fraction=0.046)

axes[1, 1].imshow(shaded, cmap='gray')
axes[1, 1].set_title('Hillshade', fontsize=12)
axes[1, 1].axis('off')

plt.suptitle('Mark Twain Lake - Terrain Analysis', fontsize=16)
plt.tight_layout()
plt.show()
```

### Example 5: Extracting Elevation Profiles

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read DEM
with rasterio.open('mark_twain_srtm_dem.tif') as src:
    dem = src.read(1)
    
# Extract horizontal profile (middle row)
mid_row = dem.shape[0] // 2
profile_horizontal = dem[mid_row, :]

# Extract vertical profile (middle column)
mid_col = dem.shape[1] // 2
profile_vertical = dem[:, mid_col]

# Calculate distance arrays (in km)
dist_h = np.arange(len(profile_horizontal)) * 0.03  # 30m pixels
dist_v = np.arange(len(profile_vertical)) * 0.03

# Visualize
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Show DEM with profile lines
im = axes[0, 0].imshow(dem, cmap='terrain')
axes[0, 0].axhline(mid_row, color='red', linewidth=2, label='Horizontal')
axes[0, 0].axvline(mid_col, color='blue', linewidth=2, label='Vertical')
axes[0, 0].set_title('DEM with Profile Lines', fontsize=12)
axes[0, 0].legend()
axes[0, 0].axis('off')
plt.colorbar(im, ax=axes[0, 0], fraction=0.046, label='Elevation (m)')

# Horizontal profile
axes[0, 1].plot(dist_h, profile_horizontal, 'r-', linewidth=2)
axes[0, 1].fill_between(dist_h, profile_horizontal, alpha=0.3, color='red')
axes[0, 1].set_xlabel('Distance (km)', fontsize=11)
axes[0, 1].set_ylabel('Elevation (m)', fontsize=11)
axes[0, 1].set_title('Horizontal Profile (West to East)', fontsize=12)
axes[0, 1].grid(True, alpha=0.3)

# Vertical profile
axes[1, 0].plot(dist_v, profile_vertical, 'b-', linewidth=2)
axes[1, 0].fill_between(dist_v, profile_vertical, alpha=0.3, color='blue')
axes[1, 0].set_xlabel('Distance (km)', fontsize=11)
axes[1, 0].set_ylabel('Elevation (m)', fontsize=11)
axes[1, 0].set_title('Vertical Profile (North to South)', fontsize=12)
axes[1, 0].grid(True, alpha=0.3)

# Elevation histogram
axes[1, 1].hist(dem.flatten(), bins=50, color='green', alpha=0.7, edgecolor='black')
axes[1, 1].set_xlabel('Elevation (m)', fontsize=11)
axes[1, 1].set_ylabel('Frequency', fontsize=11)
axes[1, 1].set_title('Elevation Distribution', fontsize=12)
axes[1, 1].grid(True, alpha=0.3, axis='y')

plt.suptitle('Mark Twain Lake - Elevation Analysis', fontsize=16)
plt.tight_layout()
plt.show()
```

## Study Area Features

Mark Twain Lake includes diverse features ideal for educational analysis:
- **Large reservoir**: Clear water body for water detection exercises
- **Surrounding forests**: Healthy vegetation with high NDVI values
- **Agricultural fields**: Varied vegetation patterns
- **Terrain variation**: Rolling hills and valleys (150-300m elevation)
- **Mixed land use**: Natural and human-modified landscapes

## License

These datasets were downloaded from Google Earth Engine for educational purposes. The original data is in the public domain:
- [USGS Landsat 9](https://www.usgs.gov/landsat-missions/landsat-9) - Public Domain
- [USGS SRTM](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1) - Public Domain

## Citation

If you use these datasets in your research or educational materials, please cite the original data sources:

**Landsat 9:**
> U.S. Geological Survey, 2025, Landsat 9 Level-2, Collection 2, Tier 1: U.S. Geological Survey data release, https://doi.org/10.5066/P9OGBGM6

**SRTM:**
> NASA Shuttle Radar Topography Mission (SRTM) (2013). Shuttle Radar Topography Mission (SRTM) Global. Distributed by OpenTopography. https://doi.org/10.5069/G9445JDF

## Contact

For questions or issues related to these datasets, please open an issue in the main repository.
