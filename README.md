# Educational Datasets Repository

This repository serves as a centralized collection of curated datasets for educational purposes in data science, geospatial analysis, and machine learning applications. The datasets are organized by type and include comprehensive documentation to facilitate learning and research.

## Repository Structure

Currently, this repository contains the following dataset categories:

### GeoTIFFs

Geospatial raster datasets in GeoTIFF format for remote sensing and spatial analysis applications.

#### Missouri Study Areas

- **[Columbia, MO](geotifs/columbia/)** - Urban and mixed-use area
  - **Location**: Columbia metropolitan area, University of Missouri campus
  - **Data**: Landsat 9 Multispectral (7 bands, Summer 2025, unscaled DN values)
  - **Use Cases**: Urban analysis, land cover classification, campus mapping, spectral indices
  - **File**: `landsat9_como_30m_unscaled_all_bands.tif`

- **[Lake of the Ozarks](geotifs/lake_of_the_ozarks/)** - Rural and natural landscape
  - **Location**: Lake of the Ozarks region, Central Missouri
  - **Data**: 
    - SRTM 30m Digital Elevation Model
    - Landsat 9 RGB Composite (2023 growing season, scaled)
    - Landsat 9 Multispectral Composite (7 bands, 2023 growing season, unscaled)
  - **Use Cases**: Terrain analysis, water body detection, forest monitoring, multispectral analysis
  - **Files**: 
    - `srtm_lake_ozarks_30m_composite.tif`
    - `landsat9_lake_ozarks_30m_rgb.tif`
    - `landsat9_lake_ozarks_30m_unscaled.tif`

- **[Mark Twain Lake](geotifs/mark_twain_lake/)** - Reservoir and mixed landscape
  - **Location**: Mark Twain Lake, Northeast Missouri
  - **Data**: Three complementary rasters for teaching raster fundamentals
    - Landsat 9 RGB + NIR (4 bands, July 2025, scaled)
    - SRTM Digital Elevation Model (single-band)
    - NDVI - Pre-calculated vegetation index (single-band)
  - **Use Cases**: Rasterio fundamentals, water body extraction, terrain analysis, vegetation monitoring
  - **Files**: 
    - `mark_twain_l9_rgb_nir_2025.tif`
    - `mark_twain_srtm_dem.tif`
    - `mark_twain_ndvi_2025.tif`

## Dataset Comparison

| Location | Area Type | Temporal Coverage | Datasets | Key Features | Best For |
|----------|-----------|-------------------|----------|--------------|----------|
| **Columbia** | Urban/Mixed | Summer 2025 | Landsat 9 (7 bands) | Campus, urban areas, agriculture | Urban analysis, ML classification |
| **Lake of the Ozarks** | Rural/Natural | Growing season 2023 | DEM + Landsat 9 (3 & 7 bands) | Terrain, water bodies, forests | Advanced multispectral, terrain |
| **Mark Twain Lake** | Reservoir/Mixed | July 2025 | DEM + Landsat 9 (4 bands) + NDVI | Large reservoir, varied terrain | Rasterio basics, water extraction |

## Educational Use

These datasets are designed to support:

### Beginner Level
- **Mark Twain Lake**: Start here for rasterio fundamentals (reading files, band operations, visualization)
- Basic raster operations and single vs. multi-band data
- Water body extraction and simple NDVI analysis

### Intermediate Level
- **Lake of the Ozarks**: Terrain analysis and multispectral processing
- Spectral index calculation (NDVI, NDWI, band ratios)
- False color composites and custom visualizations
- Integration of elevation with optical data

### Advanced Level
- **Columbia**: Urban analysis and machine learning applications
- Land cover classification (supervised and unsupervised)
- Urban feature extraction and campus mapping
- Multi-temporal change detection
- Custom spectral analysis workflows

### Cross-Dataset Applications
- Comparing urban vs. rural environments (Columbia vs. Lake of the Ozarks)
- Water body analysis across different reservoirs (Mark Twain Lake vs. Lake of the Ozarks)
- Terrain influence on land cover patterns
- Seasonal and temporal comparison (2023 vs. 2025)

## Data Sources

Datasets are sourced from reputable public repositories including:
- **NASA/USGS Landsat Program**: Landsat 9 Level-2 Surface Reflectance (Collection 2, Tier 1)
- **NASA SRTM**: Shuttle Radar Topography Mission Digital Elevation Model
- **Google Earth Engine Data Catalog**: Processing and export platform

All data is in the public domain. Each dataset directory contains detailed documentation about data provenance, processing steps, and licensing information.

## Getting Started

### For Students
1. **Start with Mark Twain Lake** - Learn rasterio basics with simple, complementary datasets
2. **Progress to Lake of the Ozarks** - Practice terrain analysis and multispectral operations
3. **Advance to Columbia** - Apply skills to urban analysis and machine learning

### For Instructors
Each dataset directory contains:
- Detailed README with metadata and specifications
- Google Earth Engine scripts for data reproduction
- Multiple Python examples with complete code
- Suggested exercises and learning objectives
- Expected outputs and visualizations

### Technical Requirements
```python
# Recommended Python packages
import rasterio
import numpy as np
import matplotlib.pyplot as plt
from scipy.ndimage import sobel  # For terrain analysis
from sklearn.cluster import KMeans  # For classification
```

## File Naming Convention

All files follow a consistent naming pattern:
```
[sensor/source]_[location]_[resolution]_[type/bands]_[year].tif
```

Examples:
- `landsat9_como_30m_unscaled_all_bands.tif`
- `srtm_lake_ozarks_30m_composite.tif`
- `mark_twain_ndvi_2025.tif`

## Repository Organization

```
educational_datasets/
├── README.md (this file)
└── geotifs/
    ├── columbia/
    │   ├── README.md
    │   └── landsat9_como_30m_unscaled_all_bands.tif
    ├── lake_of_the_ozarks/
    │   ├── README.md
    │   ├── srtm_lake_ozarks_30m_composite.tif
    │   ├── landsat9_lake_ozarks_30m_rgb.tif
    │   └── landsat9_lake_ozarks_30m_unscaled.tif
    └── mark_twain_lake/
        ├── README.md
        ├── mark_twain_l9_rgb_nir_2025.tif
        ├── mark_twain_srtm_dem.tif
        └── mark_twain_ndvi_2025.tif
```

## Contributing

This repository is maintained for educational purposes. For questions, suggestions, or issues, please open an issue or contact the repository maintainer.

## Future Additions

Planned dataset categories and expansions:
- Additional Missouri locations (Kansas City, St. Louis metropolitan areas)
- Multi-temporal datasets for change detection analysis
- Sentinel-2 high-resolution optical data (10m)
- Time series data for phenology studies
- Point cloud datasets (LiDAR)
- Vector geospatial data (shapefiles, GeoJSON)
- Labeled training datasets for supervised learning
- Climate and environmental monitoring data
- Hyperspectral imagery samples

## License

Individual datasets may have different licenses based on their original sources. Please refer to the README in each dataset directory for specific licensing information and usage restrictions. 

**All Landsat and SRTM data used in this repository is in the public domain** and freely available for educational, research, and commercial use.

## Citation

If you use these datasets in your research or publications, please cite the original data sources:

**Landsat 9:**
> U.S. Geological Survey, 2023-2025, Landsat 9 Level-2, Collection 2, Tier 1: U.S. Geological Survey data release, https://doi.org/10.5066/P9OGBGM6

**SRTM:**
> NASA Shuttle Radar Topography Mission (SRTM) (2013). Shuttle Radar Topography Mission (SRTM) Global. Distributed by OpenTopography. https://doi.org/10.5069/G9445JDF

## About

This repository is maintained by **Dr. Hatef Dastour**, Assistant Teaching Professor of Data Science and Analytics at the University of Missouri, for educational and research purposes.

### Contact
- **Institution**: University of Missouri
- **Department**: Data Science and Analytics
- **Focus**: Geospatial AI, Remote Sensing, Environmental Data Science

For questions about datasets or educational use, please open an issue in this repository.
