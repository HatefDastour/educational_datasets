# GeoTIFF Datasets Collection

This directory contains curated geospatial raster datasets in GeoTIFF format for educational purposes in remote sensing, geospatial analysis, and machine learning applications.

## Available Datasets

### Missouri Study Areas

#### [Columbia, MO](columbia/) - Urban and Mixed-Use
- **Location**: Columbia metropolitan area, University of Missouri campus
- **Data**: Landsat 9 Multispectral (7 bands, Summer 2025, unscaled DN values)
- **File**: `landsat9_como_30m_unscaled_all_bands.tif`
- **Best For**: Urban analysis, land cover classification, campus mapping

#### [Lake of the Ozarks](lake_of_the_ozarks/) - Rural and Natural
- **Location**: Lake of the Ozarks region, Central Missouri
- **Data**: 
  - SRTM 30m Digital Elevation Model
  - Landsat 9 RGB Composite (2023, scaled)
  - Landsat 9 Multispectral (7 bands, 2023, unscaled)
- **Files**: 
  - `srtm_lake_ozarks_30m_composite.tif`
  - `landsat9_lake_ozarks_30m_rgb.tif`
  - `landsat9_lake_ozarks_30m_unscaled.tif`
- **Best For**: Terrain analysis, forest monitoring, multispectral analysis

#### [Mark Twain Lake](mark_twain_lake/) - Reservoir and Mixed
- **Location**: Mark Twain Lake, Northeast Missouri
- **Data**: Three complementary rasters
  - Landsat 9 RGB + NIR (4 bands, July 2025, scaled)
  - SRTM Digital Elevation Model
  - NDVI (pre-calculated)
- **Files**: 
  - `mark_twain_l9_rgb_nir_2025.tif`
  - `mark_twain_srtm_dem.tif`
  - `mark_twain_ndvi_2025.tif`
- **Best For**: Rasterio fundamentals, water body extraction, terrain analysis

### Utah Study Area

#### [Utah Lake](utah_lake/) - Large Freshwater Lake
- **Location**: Utah Lake, north-central Utah
- **Data**: Landsat 9 (8 bands: 7 optical + thermal, Summer 2024, scaled)
- **File**: `utah_lake_l9_summer_2024.tif`
- **Best For**: Water quality analysis, thermal remote sensing, turbidity monitoring

## Dataset Comparison

| Location | Type | Temporal | Datasets | Resolution | Key Features |
|----------|------|----------|----------|------------|-------------|
| **Columbia** | Urban | Summer 2025 | L9 (7 bands) | 30m | Campus, urban areas, agriculture |
| **Lake of the Ozarks** | Rural | Growing 2023 | DEM + L9 (3 & 7 bands) | 30m | Terrain, water, forests |
| **Mark Twain Lake** | Reservoir | July 2025 | DEM + L9 (4 bands) + NDVI | 30m | Large reservoir, varied terrain |
| **Utah Lake** | Water body | Summer 2024 | L9 (8 bands w/ thermal) | 30m | Turbid water, thermal analysis |

## Learning Progression

### Beginner Level
**Start with: Mark Twain Lake**
- Learn rasterio basics (reading files, band operations)
- Understand single vs. multi-band rasters
- Practice simple visualizations
- Perform water body extraction

### Intermediate Level
**Progress to: Lake of the Ozarks**
- Terrain analysis (slope, aspect, hillshade)
- Spectral index calculations (NDVI, NDWI)
- False color composites
- Multi-dataset integration

### Advanced Level
**Option 1: Columbia** (Urban/ML Focus)
- Urban land cover classification
- Machine learning applications
- Campus feature extraction
- Advanced spectral analysis

**Option 2: Utah Lake** (Water Quality Focus)
- Water quality monitoring
- Thermal remote sensing
- Turbidity and sediment mapping
- Multi-band water analysis

## Technical Specifications

### Common Properties
- **Format**: GeoTIFF
- **Spatial Resolution**: 30 meters
- **Data Type**: Float32 or Int16
- **Source**: USGS Landsat 9 & NASA SRTM
- **Processing Platform**: Google Earth Engine

### Coordinate Reference Systems
- **Missouri datasets**: EPSG:32615 (WGS 84 / UTM zone 15N)
- **Utah Lake**: EPSG:4326 (WGS 84 Geographic)

## File Naming Convention

```
[sensor/source]_[location]_[resolution]_[type/bands]_[year].tif
```

**Examples:**
- `landsat9_como_30m_unscaled_all_bands.tif`
- `srtm_lake_ozarks_30m_composite.tif`
- `mark_twain_ndvi_2025.tif`
- `utah_lake_l9_summer_2024.tif`

## Data Sources

All datasets are derived from public domain sources:

- **Landsat 9**: USGS Landsat 9 Level-2, Collection 2, Tier 1 Surface Reflectance
  - Optical bands scaled: DN × 0.0000275 - 0.2
  - Thermal bands scaled: DN × 0.00341802 + 149.0 (Kelvin)
  - [USGS Landsat Missions](https://www.usgs.gov/landsat-missions/landsat-9)

- **SRTM**: NASA Shuttle Radar Topography Mission Global 1 arc-second
  - Void-filled elevation data
  - [USGS SRTM](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1)

- **Processing**: Google Earth Engine Data Catalog
  - Cloud masking, compositing, and export
  - [Google Earth Engine](https://earthengine.google.com/)

## Getting Started

### Prerequisites

```python
# Required Python packages
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Optional but recommended
from scipy.ndimage import sobel  # Terrain analysis
from sklearn.cluster import KMeans  # Classification
import geopandas as gpd  # Vector operations
```

### Basic Usage Example

```python
import rasterio
import matplotlib.pyplot as plt

# Read a GeoTIFF
with rasterio.open('path/to/dataset.tif') as src:
    # Get metadata
    print(f"Bands: {src.count}")
    print(f"CRS: {src.crs}")
    print(f"Bounds: {src.bounds}")
    
    # Read data
    data = src.read()  # All bands
    band1 = src.read(1)  # First band only
    
    # Visualize
    plt.imshow(band1, cmap='gray')
    plt.colorbar()
    plt.show()
```

## Educational Applications

### By Dataset

**Mark Twain Lake** - Introductory remote sensing courses
- Week 1-2: Raster data fundamentals
- Week 3: Water body detection
- Week 4: Basic terrain analysis

**Lake of the Ozarks** - Intermediate GIS/remote sensing
- Multi-band analysis and spectral indices
- Terrain derivatives (slope, aspect)
- Data integration workflows

**Columbia** - Advanced topics and capstone projects
- Urban remote sensing
- Machine learning classification
- Feature extraction and mapping

**Utah Lake** - Water resources and environmental courses
- Water quality monitoring
- Thermal remote sensing
- Environmental change detection

### Cross-Dataset Comparisons

1. **Urban vs. Rural**: Compare Columbia with Lake of the Ozarks
2. **Water Bodies**: Analyze Mark Twain Lake vs. Utah Lake (clear vs. turbid)
3. **Temporal**: Compare 2023, 2024, and 2025 datasets
4. **Geographic**: Missouri vs. Utah regional differences

## Documentation

Each dataset directory contains:
- **README.md**: Detailed documentation
- **GEE Scripts**: Complete Google Earth Engine code for reproduction
- **Python Examples**: 3-5 working code examples
- **Use Cases**: Specific applications and exercises
- **Metadata**: Full band descriptions and specifications

## Citation

When using these datasets, please cite the original sources:

**Landsat 9:**
> U.S. Geological Survey, 2023-2025, Landsat 9 Level-2, Collection 2, Tier 1: U.S. Geological Survey data release, https://doi.org/10.5066/P9OGBGM6

**SRTM:**
> NASA Shuttle Radar Topography Mission (SRTM) (2013). Shuttle Radar Topography Mission (SRTM) Global. Distributed by OpenTopography. https://doi.org/10.5069/G9445JDF

## License

All datasets are derived from public domain sources and are freely available for educational, research, and commercial use.

## Contributing

For questions, suggestions, or to report issues, please open an issue in the main repository.

## About

These datasets are curated and maintained by Dr. Hatef Dastour, Assistant Teaching Professor of Data Science and Analytics at the University of Missouri, for educational and research purposes in geospatial AI and environmental data science.
