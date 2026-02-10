# Utah Lake GeoTIFF Dataset

This directory contains Landsat 9 multispectral data for Utah Lake, downloaded from Google Earth Engine for educational purposes focused on water quality monitoring and large lake analysis.

## Study Area

Utah Lake is a large, shallow freshwater lake in north-central Utah with the following specifications:
- **Longitude**: -111.95° to -111.64° W
- **Latitude**: 40.01° to 40.37° N
- **Coordinate Reference System**: EPSG:4326 (WGS 84)
- **Spatial Resolution**: 30 meters
- **Area Coverage**: Approximately 31 km × 40 km
- **Key Features**: Large shallow lake, high turbidity, surrounded by urban areas (Provo-Orem)

## Dataset

### Landsat 9 Summer 2024 Composite (8 bands)

**File:** `utah_lake_l9_summer_2024.tif`

**Description:** Cloud-masked median composite with scaled surface reflectance for seven optical bands plus thermal band from summer 2024, ideal for water quality analysis.

**Source:** USGS Landsat 9 Level-2, Collection 2, Tier 1 (LANDSAT/LC09/C02/T1_L2)

**Temporal Coverage:** June 1 - August 31, 2024 (Summer season)

**Bands Included (in order):**
1. Coastal Aerosol (SR_B1): 0.43-0.45 μm
2. Blue (SR_B2): 0.45-0.51 μm
3. Green (SR_B3): 0.53-0.59 μm
4. Red (SR_B4): 0.64-0.67 μm
5. NIR (SR_B5): 0.85-0.88 μm
6. SWIR1 (SR_B6): 1.57-1.65 μm
7. SWIR2 (SR_B7): 2.11-2.29 μm
8. Thermal (ST_B10): 10.6-11.19 μm

**Data Characteristics:**
- **Optical Bands (1-7)**: 0.0 - ~0.5 (scaled surface reflectance)
- **Thermal Band (8)**: 149 - ~320 Kelvin (scaled brightness temperature)
- **Data Type**: Float32
- **Processing**: 
  - Cloud cover < 20%
  - Cloud and saturation masking using QA_PIXEL and QA_RADSAT
  - Optical scaling: DN × 0.0000275 - 0.2
  - Thermal scaling: DN × 0.00341802 + 149.0
  - Median composite

## Google Earth Engine Code

```javascript
// ============================================================
// GOOGLE EARTH ENGINE SCRIPT 1: LANDSAT 9 SUMMER 2024
// ============================================================
// Run this in GEE Code Editor: https://code.earthengine.google.com/

// Study area: Utah Lake
var utahLake = ee.Geometry.Rectangle([
  -111.95, 40.37,  // west, south
  -111.64, 40.01   // east, north
]);

Map.centerObject(utahLake, 11);
Map.addLayer(utahLake, {color: 'red'}, 'Utah Lake Boundary');

// Date range: Summer 2024 (June-August)
var startDate = '2024-06-01';
var endDate = '2024-08-31';

// Load Landsat 9 Collection 2 Surface Reflectance
var landsat9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
  .filterBounds(utahLake)
  .filterDate(startDate, endDate)
  .filter(ee.Filter.lt('CLOUD_COVER', 20));

print('Landsat 9 scenes available:', landsat9.size());

// Cloud masking function for Landsat Collection 2
function maskL9sr(image) {
  var qaMask = image.select('QA_PIXEL').bitwiseAnd(parseInt('11111', 2)).eq(0);
  var saturationMask = image.select('QA_RADSAT').eq(0);
  
  // Apply scaling factors
  var opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
  var thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0);
  
  return image.addBands(opticalBands, null, true)
      .addBands(thermalBands, null, true)
      .updateMask(qaMask)
      .updateMask(saturationMask);
}

// Apply cloud mask and create median composite
var l9Composite = landsat9
  .map(maskL9sr)
  .median()
  .clip(utahLake);

// Select and rename bands for clarity
var l9Export = l9Composite.select(
  ['SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7', 'ST_B10'],
  ['coastal', 'blue', 'green', 'red', 'nir', 'swir1', 'swir2', 'thermal']
);

// Visualization
var rgbVis = {
  bands: ['red', 'green', 'blue'],
  min: 0,
  max: 0.3
};

Map.addLayer(l9Export, rgbVis, 'Landsat 9 RGB Composite');

// False color for water quality (SWIR1, NIR, Green)
var waterVis = {
  bands: ['swir1', 'nir', 'green'],
  min: 0,
  max: 0.3
};

Map.addLayer(l9Export, waterVis, 'Water Quality False Color');

// Export to Google Drive
Export.image.toDrive({
  image: l9Export.toFloat(),
  description: 'utah_lake_landsat9_summer2024',
  folder: 'GEE_exports',
  fileNamePrefix: 'utah_lake_l9_summer_2024',
  region: utahLake,
  scale: 30,
  crs: 'EPSG:4326',
  maxPixels: 1e9,
  fileFormat: 'GeoTIFF'
});
```

## Dataset Summary

| File | Bands | Data Type | Band Types | Size (approx) | Primary Use |
|------|-------|-----------|------------|---------------|-------------|
| `utah_lake_l9_summer_2024.tif` | 8 | Float32 | 7 optical + 1 thermal | ~40 MB | Water quality, thermal analysis |

## Educational Use

This dataset is specifically designed for teaching:

### Water Quality Analysis
1. **Turbidity estimation** using visible bands
2. **Chlorophyll-a detection** using NIR-Red ratios
3. **Suspended sediment mapping** using SWIR bands
4. **Water surface temperature** from thermal band
5. **Algal bloom detection** using band combinations

### Thermal Remote Sensing
1. **Land-water temperature contrast**
2. **Urban heat island effects** (Provo-Orem cities)
3. **Thermal-optical data fusion**
4. **Surface temperature spatial patterns**

### Spectral Analysis
1. **Water quality indices** (NDWI, turbidity indices)
2. **False color composites** for water features
3. **Coastal aerosol band applications**
4. **Multi-band classification** of water vs. land

## Example Usage in Python

### Example 1: Loading and Visualizing All Bands

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read all 8 bands
with rasterio.open('utah_lake_l9_summer_2024.tif') as src:
    bands = src.read()  # Shape: (8, height, width)
    band_names = ['Coastal', 'Blue', 'Green', 'Red', 'NIR', 'SWIR1', 'SWIR2', 'Thermal']
    
    print(f"Bands: {src.count}")
    print(f"Size: {src.width} x {src.height}")
    print(f"CRS: {src.crs}")
    print(f"Bounds: {src.bounds}")

# Extract individual bands
coastal, blue, green, red, nir, swir1, swir2, thermal = bands

# Create visualization
fig, axes = plt.subplots(2, 4, figsize=(18, 10))
axes = axes.flatten()

for idx, (band, name) in enumerate(zip(bands, band_names)):
    if name == 'Thermal':
        # Thermal band in Kelvin
        im = axes[idx].imshow(band, cmap='hot', vmin=290, vmax=310)
        axes[idx].set_title(f'{name}\n(Kelvin)', fontsize=11)
    else:
        # Optical bands
        im = axes[idx].imshow(band, cmap='gray', vmin=0, vmax=0.3)
        axes[idx].set_title(f'{name}\n(Reflectance)', fontsize=11)
    
    axes[idx].axis('off')
    plt.colorbar(im, ax=axes[idx], fraction=0.046)

plt.suptitle('Utah Lake - Landsat 9 All Bands (Summer 2024)', fontsize=16)
plt.tight_layout()
plt.show()
```

### Example 2: True Color and Water Quality False Color

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read bands
with rasterio.open('utah_lake_l9_summer_2024.tif') as src:
    bands = src.read()

# Extract bands (0-indexed: coastal=0, blue=1, green=2, red=3, nir=4, swir1=5, swir2=6, thermal=7)
blue, green, red = bands[1], bands[2], bands[3]
nir, swir1 = bands[4], bands[5]

# True color composite
rgb = np.dstack([red, green, blue])
rgb = np.clip(rgb, 0, 0.3) / 0.3  # Normalize

# False color for water quality (SWIR1-NIR-Green)
water_quality = np.dstack([swir1, nir, green])
water_quality = np.clip(water_quality, 0, 0.3) / 0.3

# False color for vegetation (NIR-Red-Green)
vegetation = np.dstack([nir, red, green])
vegetation = np.clip(vegetation, 0, 0.4) / 0.4

fig, axes = plt.subplots(1, 3, figsize=(18, 6))

axes[0].imshow(rgb)
axes[0].set_title('True Color (Red-Green-Blue)', fontsize=13)
axes[0].axis('off')

axes[1].imshow(water_quality)
axes[1].set_title('Water Quality (SWIR1-NIR-Green)\nTurbidity and Sediment', fontsize=13)
axes[1].axis('off')

axes[2].imshow(vegetation)
axes[2].set_title('Vegetation (NIR-Red-Green)\nLand-Water Contrast', fontsize=13)
axes[2].axis('off')

plt.suptitle('Utah Lake - Band Combinations', fontsize=16)
plt.tight_layout()
plt.show()
```

### Example 3: Water Quality Indices

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read bands
with rasterio.open('utah_lake_l9_summer_2024.tif') as src:
    bands = src.read()

# Extract bands
green, red, nir, swir1 = bands[2], bands[3], bands[4], bands[5]

# Calculate water quality indices
# NDWI - Normalized Difference Water Index (water body delineation)
ndwi = (green - nir) / (green + nir + 1e-8)

# MNDWI - Modified NDWI (better for turbid water)
mndwi = (green - swir1) / (green + swir1 + 1e-8)

# Simple turbidity proxy (Red/Green ratio)
turbidity = red / (green + 1e-8)

# NDVI for surrounding land
ndvi = (nir - red) / (nir + red + 1e-8)

# Create water mask (MNDWI > 0)
water_mask = mndwi > 0

fig, axes = plt.subplots(2, 3, figsize=(16, 11))

# True color
rgb = np.dstack([red, green, bands[1]])
rgb = np.clip(rgb, 0, 0.3) / 0.3
axes[0, 0].imshow(rgb)
axes[0, 0].set_title('True Color', fontsize=12)
axes[0, 0].axis('off')

# NDWI
im1 = axes[0, 1].imshow(ndwi, cmap='Blues', vmin=-0.5, vmax=0.5)
axes[0, 1].set_title('NDWI (Water Index)', fontsize=12)
axes[0, 1].axis('off')
plt.colorbar(im1, ax=axes[0, 1], fraction=0.046)

# MNDWI
im2 = axes[0, 2].imshow(mndwi, cmap='RdYlBu', vmin=-0.5, vmax=0.5)
axes[0, 2].set_title('MNDWI (Modified Water Index)', fontsize=12)
axes[0, 2].axis('off')
plt.colorbar(im2, ax=axes[0, 2], fraction=0.046)

# Turbidity proxy
turbidity_masked = np.where(water_mask, turbidity, np.nan)
im3 = axes[1, 0].imshow(turbidity_masked, cmap='YlOrBr', vmin=0.5, vmax=1.5)
axes[1, 0].set_title('Turbidity Proxy (Red/Green)\n(Water only)', fontsize=12)
axes[1, 0].axis('off')
plt.colorbar(im3, ax=axes[1, 0], fraction=0.046)

# NDVI
im4 = axes[1, 1].imshow(ndvi, cmap='RdYlGn', vmin=-0.2, vmax=0.6)
axes[1, 1].set_title('NDVI (Vegetation)', fontsize=12)
axes[1, 1].axis('off')
plt.colorbar(im4, ax=axes[1, 1], fraction=0.046)

# Water mask
axes[1, 2].imshow(water_mask, cmap='Blues_r')
axes[1, 2].set_title('Water Body Mask', fontsize=12)
axes[1, 2].axis('off')

plt.suptitle('Utah Lake - Water Quality Analysis', fontsize=16)
plt.tight_layout()
plt.show()

# Print statistics
water_turbidity = turbidity[water_mask]
print(f"Water area: {np.sum(water_mask) * 900 / 1e6:.2f} km²")  # 30m pixels
print(f"Mean turbidity: {np.nanmean(water_turbidity):.3f}")
print(f"Turbidity range: {np.nanmin(water_turbidity):.3f} - {np.nanmax(water_turbidity):.3f}")
```

### Example 4: Thermal Analysis

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read bands
with rasterio.open('utah_lake_l9_summer_2024.tif') as src:
    bands = src.read()

# Extract thermal and optical bands
thermal_k = bands[7]  # Kelvin
green, nir, swir1 = bands[2], bands[4], bands[5]

# Convert Kelvin to Celsius
thermal_c = thermal_k - 273.15

# Create water mask using MNDWI
mndwi = (green - swir1) / (green + swir1 + 1e-8)
water_mask = mndwi > 0
land_mask = ~water_mask

# Separate water and land temperatures
water_temp = np.where(water_mask, thermal_c, np.nan)
land_temp = np.where(land_mask, thermal_c, np.nan)

# Calculate statistics
water_mean = np.nanmean(water_temp)
land_mean = np.nanmean(land_temp)
temp_diff = land_mean - water_mean

print(f"Water temperature: {water_mean:.2f}°C (±{np.nanstd(water_temp):.2f})")
print(f"Land temperature: {land_mean:.2f}°C (±{np.nanstd(land_temp):.2f})")
print(f"Land-water contrast: {temp_diff:.2f}°C")

# Visualize
fig, axes = plt.subplots(2, 2, figsize=(14, 12))

# True color
rgb = np.dstack([bands[3], green, bands[1]])
rgb = np.clip(rgb, 0, 0.3) / 0.3
axes[0, 0].imshow(rgb)
axes[0, 0].set_title('True Color', fontsize=13)
axes[0, 0].axis('off')

# Full thermal
im1 = axes[0, 1].imshow(thermal_c, cmap='hot', vmin=15, vmax=40)
axes[0, 1].set_title('Surface Temperature (°C)', fontsize=13)
axes[0, 1].axis('off')
plt.colorbar(im1, ax=axes[0, 1], fraction=0.046, label='°C')

# Water temperature only
im2 = axes[1, 0].imshow(water_temp, cmap='coolwarm', vmin=18, vmax=26)
axes[1, 0].set_title(f'Water Temperature\nMean: {water_mean:.2f}°C', fontsize=13)
axes[1, 0].axis('off')
plt.colorbar(im2, ax=axes[1, 0], fraction=0.046, label='°C')

# Land temperature only
im3 = axes[1, 1].imshow(land_temp, cmap='hot', vmin=25, vmax=40)
axes[1, 1].set_title(f'Land Temperature\nMean: {land_mean:.2f}°C', fontsize=13)
axes[1, 1].axis('off')
plt.colorbar(im3, ax=axes[1, 1], fraction=0.046, label='°C')

plt.suptitle('Utah Lake - Thermal Analysis (Summer 2024)', fontsize=16)
plt.tight_layout()
plt.show()

# Temperature histogram
plt.figure(figsize=(10, 6))
plt.hist(water_temp[~np.isnan(water_temp)], bins=50, alpha=0.6, label='Water', color='blue')
plt.hist(land_temp[~np.isnan(land_temp)], bins=50, alpha=0.6, label='Land', color='brown')
plt.xlabel('Temperature (°C)', fontsize=12)
plt.ylabel('Frequency', fontsize=12)
plt.title('Temperature Distribution: Water vs. Land', fontsize=14)
plt.legend(fontsize=11)
plt.grid(True, alpha=0.3)
plt.show()
```

### Example 5: Coastal Aerosol Band Application

```python
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Read bands
with rasterio.open('utah_lake_l9_summer_2024.tif') as src:
    bands = src.read()

# Extract bands
coastal, blue, green, red, nir = bands[0], bands[1], bands[2], bands[3], bands[4]

# Coastal aerosol applications
# 1. Haze detection (coastal aerosol is sensitive to atmospheric scattering)
haze_index = coastal / (blue + 1e-8)

# 2. Shallow water penetration (coastal aerosol penetrates water better)
shallow_water = np.dstack([coastal, blue, green])
shallow_water = np.clip(shallow_water, 0, 0.2) / 0.2

# 3. Atmospheric correction check
atmosphere_diff = coastal - blue

fig, axes = plt.subplots(2, 2, figsize=(14, 12))

# Coastal aerosol band
im1 = axes[0, 0].imshow(coastal, cmap='gray', vmin=0, vmax=0.15)
axes[0, 0].set_title('Coastal Aerosol Band', fontsize=12)
axes[0, 0].axis('off')
plt.colorbar(im1, ax=axes[0, 0], fraction=0.046)

# Blue band for comparison
im2 = axes[0, 1].imshow(blue, cmap='gray', vmin=0, vmax=0.15)
axes[0, 1].set_title('Blue Band', fontsize=12)
axes[0, 1].axis('off')
plt.colorbar(im2, ax=axes[0, 1], fraction=0.046)

# Haze index
im3 = axes[1, 0].imshow(haze_index, cmap='RdYlBu_r', vmin=0.8, vmax=1.2)
axes[1, 0].set_title('Haze Index (Coastal/Blue)', fontsize=12)
axes[1, 0].axis('off')
plt.colorbar(im3, ax=axes[1, 0], fraction=0.046)

# Shallow water composite
axes[1, 1].imshow(shallow_water)
axes[1, 1].set_title('Shallow Water (Coastal-Blue-Green)', fontsize=12)
axes[1, 1].axis('off')

plt.suptitle('Utah Lake - Coastal Aerosol Band Applications', fontsize=16)
plt.tight_layout()
plt.show()
```

## Study Area Context

**Utah Lake** is one of the largest natural freshwater lakes in the western United States:
- **Characteristics**: Very shallow (average depth ~3m), high turbidity, eutrophic
- **Water Quality Issues**: Algal blooms, high sediment load, phosphorus enrichment
- **Surroundings**: Provo-Orem metropolitan area (urban influence), agricultural land
- **Research Value**: Excellent for studying turbid water optics, urban-lake interactions, thermal dynamics

**Unique Features for Remote Sensing:**
- High turbidity makes it ideal for testing water quality algorithms
- Clear thermal contrast between water and land
- Visible algal blooms during summer
- Mixing of urban, agricultural, and natural landscapes

## Recommended Exercises

1. **Compare turbidity patterns** across the lake using different band combinations
2. **Map thermal gradients** and relate to water depth (shallower areas may be warmer)
3. **Detect algal blooms** using NIR-Red ratios and coastal aerosol band
4. **Classify water quality zones** based on spectral characteristics
5. **Analyze urban heat island effect** in Provo-Orem using thermal band
6. **Create time series** by processing multiple dates to track seasonal changes

## License

This dataset was downloaded from Google Earth Engine for educational purposes. The original data is in the public domain:
- [USGS Landsat 9](https://www.usgs.gov/landsat-missions/landsat-9) - Public Domain

## Citation

If you use this dataset in your research or educational materials, please cite the original data source:

> U.S. Geological Survey, 2024, Landsat 9 Level-2, Collection 2, Tier 1: U.S. Geological Survey data release, https://doi.org/10.5066/P9OGBGM6

## Contact

For questions or issues related to this dataset, please open an issue in the main repository.
