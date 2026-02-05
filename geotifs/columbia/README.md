# Columbia, Missouri GeoTIFF Dataset

This directory contains Landsat 9 multispectral data for Columbia, Missouri, downloaded from Google Earth Engine for educational purposes.

## Study Area

The dataset covers the Columbia, Missouri metropolitan area with the following specifications:
- **Longitude**: -92.416° to -92.207° W
- **Latitude**: 38.880° to 39.019° N
- **Coordinate Reference System**: EPSG:32615 (WGS 84 / UTM zone 15N)
- **Spatial Resolution**: 30 meters
- **Area Coverage**: Approximately 20 km × 15 km

## Dataset

### Landsat 9 Multispectral Composite (Unscaled)

**File:** `landsat9_como_30m_unscaled_all_bands.tif`

**Description:** Cloud-masked median composite with original Digital Number (DN) values for all seven optical bands from summer 2025.

**Source:** USGS Landsat 9 Level-2, Collection 2, Tier 1 (LANDSAT/LC09/C02/T1_L2)

**Temporal Coverage:** June 1, 2025 - August 31, 2025 (Summer season)

**Bands Included (in order):**
1. Band 1 (Coastal Aerosol): 0.43-0.45 μm
2. Band 2 (Blue): 0.45-0.51 μm
3. Band 3 (Green): 0.53-0.59 μm
4. Band 4 (Red): 0.64-0.67 μm
5. Band 5 (NIR): 0.85-0.88 μm
6. Band 6 (SWIR 1): 1.57-1.65 μm
7. Band 7 (SWIR 2): 2.11-2.29 μm

**Data Characteristics:**
- **Value Range**: 0 - ~30,000 (DN values)
- **No Data Value**: -9999
- **Data Type**: Int16
- **Scaling Formula**: To convert to surface reflectance: `SR = DN × 0.0000275 - 0.2`

**Processing:**
- Cloud cover filter: < 50%
- Cloud and shadow masking using QA_PIXEL band (bits 3 and 4)
- Median composite to reduce noise and fill gaps
- Original DN values preserved (not scaled)

**Google Earth Engine Code:**
```javascript
// 1. Define rectangular AOI and visualization
var rect = ee.Geometry.Rectangle([-92.416, 38.880, -92.207, 39.019], null, false);
Map.centerObject(rect, 11);
Map.addLayer(rect, {color: 'red'}, 'Study Area');

// 2. Temporal window - Summer 2025
var start = '2025-06-01';
var end   = '2025-08-31';

// 3. Load Landsat 9 SR (UNSCALED integers) + basic cloud/shadow mask
var l9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(rect)
  .filterDate(start, end)
  .filter(ee.Filter.lt('CLOUD_COVER', 50))
  .map(function(img) {
    // NO scaling - keep DN integers
    // Simple cloud/shadow mask (bits 3=cloud, 4=shadow)
    var qa = img.select('QA_PIXEL');
    var mask = qa.bitwiseAnd(1 << 3).eq(0).and(qa.bitwiseAnd(1 << 4).eq(0));
    return img.updateMask(mask);
  });

// 4. Median composite (unscaled)
var comp = l9.median().clip(rect);

// 5. True-color vis (adjust for unscaled DN ~0-30000)
Map.addLayer(comp.select(['SR_B4','SR_B3','SR_B2']), 
             {min: 0, max: 30000, gamma: 1.2}, 'True Color (Unscaled)');

// Export ALL spectral bands (unscaled DN, NoData=-9999)
Export.image.toDrive({
  image: comp.select([
    'SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 
    'SR_B6', 'SR_B7'
  ]),
  description: 'landsat9_como_30m_unscaled_all_bands',
  folder: 'GEE_exports',
  fileNamePrefix: 'landsat9_como_30m_unscaled_all_bands',
  region: rect,
  scale: 30,
  crs: 'EPSG:32615',
  maxPixels: 1e13
});
```

## Usage

This dataset is ideal for:
- Urban land cover classification and mapping
- Vegetation analysis in mixed urban-rural environments
- Water body detection (Missouri River visibility)
- Supervised and unsupervised machine learning training
- Spectral index calculation (NDVI, NDBI, NDWI, etc.)
- Urban heat island studies
- Change detection and temporal analysis
- Educational demonstrations of multispectral analysis

## Data Format

| File | Bands | Data Type | Value Range | No Data |
|------|-------|-----------|-------------|----------|
| `landsat9_como_30m_unscaled_all_bands.tif` | 7 (B1-B7) | Int16 | 0-30000 (DN) | -9999 |

## Example Usage in Python

### Basic Data Loading and Visualization

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.colors import LinearSegmentedColormap

# Read unscaled Landsat data
with rasterio.open('landsat9_como_30m_unscaled_all_bands.tif') as src:
    bands = src.read()  # Shape: (7, height, width)
    profile = src.profile
    nodata = -9999
    
# Mask no-data values
bands = np.where(bands == nodata, np.nan, bands)

# Convert DN values to surface reflectance
sr = bands.astype(np.float32) * 0.0000275 - 0.2

# Create true color composite (R, G, B)
rgb = np.dstack([sr[3], sr[2], sr[1]])  # Red, Green, Blue
rgb = np.clip(rgb, 0, 0.3) / 0.3  # Normalize for display

# Visualize
plt.figure(figsize=(12, 10))
plt.imshow(rgb)
plt.title('Columbia, MO - Landsat 9 True Color (Summer 2025)', fontsize=14)
plt.axis('off')
plt.tight_layout()
plt.show()
```

### Calculate Vegetation and Urban Indices

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read and convert to surface reflectance
with rasterio.open('landsat9_como_30m_unscaled_all_bands.tif') as src:
    bands = src.read().astype(np.float32)
    
# Mask no-data and convert to SR
bands = np.where(bands == -9999, np.nan, bands)
sr = bands * 0.0000275 - 0.2

# Extract bands
blue = sr[1]    # Band 2
green = sr[2]   # Band 3
red = sr[3]     # Band 4
nir = sr[4]     # Band 5
swir1 = sr[5]   # Band 6

# Calculate indices
# NDVI - Normalized Difference Vegetation Index (vegetation health)
ndvi = (nir - red) / (nir + red + 1e-8)

# NDBI - Normalized Difference Built-up Index (urban areas)
ndbi = (swir1 - nir) / (swir1 + nir + 1e-8)

# NDWI - Normalized Difference Water Index (water bodies)
ndwi = (green - nir) / (green + nir + 1e-8)

# Visualize indices
fig, axes = plt.subplots(2, 2, figsize=(14, 12))

# True color
rgb = np.dstack([red, green, blue])
rgb = np.clip(rgb, 0, 0.3) / 0.3
axes[0, 0].imshow(rgb)
axes[0, 0].set_title('True Color', fontsize=12)
axes[0, 0].axis('off')

# NDVI
im1 = axes[0, 1].imshow(ndvi, cmap='RdYlGn', vmin=-0.3, vmax=0.8)
axes[0, 1].set_title('NDVI (Vegetation)', fontsize=12)
axes[0, 1].axis('off')
plt.colorbar(im1, ax=axes[0, 1], fraction=0.046)

# NDBI
im2 = axes[1, 0].imshow(ndbi, cmap='RdYlBu_r', vmin=-0.3, vmax=0.3)
axes[1, 0].set_title('NDBI (Built-up)', fontsize=12)
axes[1, 0].axis('off')
plt.colorbar(im2, ax=axes[1, 0], fraction=0.046)

# NDWI
im3 = axes[1, 1].imshow(ndwi, cmap='Blues', vmin=-0.3, vmax=0.5)
axes[1, 1].set_title('NDWI (Water)', fontsize=12)
axes[1, 1].axis('off')
plt.colorbar(im3, ax=axes[1, 1], fraction=0.046)

plt.suptitle('Columbia, MO - Spectral Indices Analysis', fontsize=16, y=0.98)
plt.tight_layout()
plt.show()
```

### Simple Land Cover Classification

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans

# Read and convert to surface reflectance
with rasterio.open('landsat9_como_30m_unscaled_all_bands.tif') as src:
    bands = src.read().astype(np.float32)
    height, width = src.height, src.width
    profile = src.profile

# Mask and convert
bands = np.where(bands == -9999, np.nan, bands)
sr = bands * 0.0000275 - 0.2

# Reshape for clustering (pixels x bands)
sr_reshaped = sr.reshape(7, -1).T

# Remove NaN pixels
valid_mask = ~np.isnan(sr_reshaped).any(axis=1)
sr_valid = sr_reshaped[valid_mask]

# Perform K-means clustering (5 classes)
kmeans = KMeans(n_clusters=5, random_state=42, n_init=10)
labels_valid = kmeans.fit_predict(sr_valid)

# Reconstruct full image
labels = np.full(sr_reshaped.shape[0], np.nan)
labels[valid_mask] = labels_valid
labels_2d = labels.reshape(height, width)

# Visualize classification
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# True color
rgb = np.dstack([sr[3], sr[2], sr[1]])
rgb = np.clip(rgb, 0, 0.3) / 0.3
axes[0].imshow(rgb)
axes[0].set_title('True Color Composite', fontsize=14)
axes[0].axis('off')

# Classification
cmap = plt.cm.get_cmap('tab10', 5)
im = axes[1].imshow(labels_2d, cmap=cmap, interpolation='nearest')
axes[1].set_title('Unsupervised Classification (5 Classes)', fontsize=14)
axes[1].axis('off')
cbar = plt.colorbar(im, ax=axes[1], fraction=0.046)
cbar.set_label('Class', fontsize=12)

plt.suptitle('Columbia, MO - Land Cover Classification', fontsize=16)
plt.tight_layout()
plt.show()

# Optional: Export classification
profile.update(dtype=rasterio.uint8, count=1, nodata=255)
with rasterio.open('columbia_classification.tif', 'w', **profile) as dst:
    dst.write(np.nan_to_num(labels_2d, nan=255).astype(np.uint8), 1)
```

### Create False Color Composites

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

with rasterio.open('landsat9_como_30m_unscaled_all_bands.tif') as src:
    bands = src.read().astype(np.float32)

# Mask and convert to SR
bands = np.where(bands == -9999, np.nan, bands)
sr = bands * 0.0000275 - 0.2

# Define different composites
composites = {
    'True Color (R-G-B)': [sr[3], sr[2], sr[1]],
    'False Color (NIR-R-G)': [sr[4], sr[3], sr[2]],
    'Agriculture (SWIR1-NIR-Blue)': [sr[5], sr[4], sr[1]],
    'Urban (SWIR2-SWIR1-R)': [sr[6], sr[5], sr[3]]
}

fig, axes = plt.subplots(2, 2, figsize=(16, 14))
axes = axes.flatten()

for idx, (title, bands_combo) in enumerate(composites.items()):
    composite = np.dstack(bands_combo)
    composite = np.clip(composite, 0, 0.4) / 0.4
    axes[idx].imshow(composite)
    axes[idx].set_title(title, fontsize=13)
    axes[idx].axis('off')

plt.suptitle('Columbia, MO - Band Combination Composites', fontsize=16)
plt.tight_layout()
plt.show()
```

## Educational Applications

### Recommended Exercises for Students

1. **Spectral Signature Analysis**: Extract and plot spectral profiles for different land cover types (water, vegetation, urban, bare soil)

2. **Threshold-based Classification**: Use NDVI or NDWI thresholds to create binary masks for vegetation or water

3. **Principal Component Analysis (PCA)**: Perform PCA on all 7 bands to reduce dimensionality

4. **Texture Analysis**: Calculate GLCM (Gray Level Co-occurrence Matrix) features for urban feature extraction

5. **Comparison with Historical Data**: Compare this summer 2025 dataset with other seasons or years

6. **Campus Mapping**: Identify and delineate the University of Missouri campus using spectral characteristics

## Study Area Features

The Columbia dataset includes diverse land cover types:
- **Urban areas**: Downtown Columbia, residential neighborhoods, commercial zones
- **University of Missouri campus**: Large institutional complex with mixed land use
- **Vegetation**: Parks, forests, agricultural fields
- **Water bodies**: Portions of the Missouri River, streams, and lakes
- **Transportation**: Major highways (I-70, US-63) and road networks

## License

This dataset was downloaded from Google Earth Engine for educational purposes. The original data is in the public domain:
- [USGS Landsat 9](https://www.usgs.gov/landsat-missions/landsat-9) - Public Domain

## Citation

If you use this dataset in your research or educational materials, please cite the original data source:

> U.S. Geological Survey, 2025, Landsat 9 Level-2, Collection 2, Tier 1: U.S. Geological Survey data release, https://doi.org/10.5066/P9OGBGM6

## Contact

For questions or issues related to this dataset, please open an issue in the main repository.
