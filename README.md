# Educational Datasets Repository

> A curated collection of high-quality datasets for teaching and learning data science, geospatial analysis, machine learning, and environmental science.

This repository serves as a comprehensive resource for educators, students, and researchers seeking well-documented, ready-to-use datasets for educational purposes. All datasets include detailed documentation, processing scripts, and practical code examples.

## 📊 Available Dataset Categories

### [GeoTIFFs](geotifs/) - Geospatial Raster Data
Remote sensing and terrain datasets in GeoTIFF format, including:
- **Landsat 8/9 multispectral imagery** (optical and thermal bands)
- **Sentinel-2 multispectral imagery** (10 m high-resolution optical bands)
- **MODIS products** (NDVI 250 m, Land Surface Temperature 1 km)
- **SRTM Digital Elevation Models** (terrain data at 30 m)
- **Derived products** (NDVI, water indices, composites)

**Current Locations:**

| Region | Location | Datasets | Highlights |
|--------|----------|----------|------------|
| **USA – Missouri** | [Columbia](geotifs/columbia/) | Landsat 9 (7 bands, unscaled) | Urban/campus analysis, ML classification |
| **USA – Missouri** | [Lake of the Ozarks](geotifs/lake_of_the_ozarks/) | SRTM + Landsat 9 (RGB & 7 bands) | Terrain, forest, multispectral |
| **USA – Missouri** | [Mark Twain Lake](geotifs/mark_twain_lake/) | Landsat 9 RGB+NIR + SRTM + NDVI | Beginner rasterio fundamentals |
| **USA – Utah** | [Utah Lake](geotifs/utah_lake/) | Landsat 9 (8 bands) + time series + Sentinel-2 monthly | Water quality, thermal, temporal analysis |
| **USA – New Mexico** | [Albuquerque](geotifs/albuquerque/) | MODIS NDVI + LST + SRTM | Urban heat island, multi-scale sensors |
| **Canada – Alberta** | [Banff Town](geotifs/banff_town/) | Sentinel-2 + Landsat 9 + SRTM | Alpine multi-sensor comparison |
| **Canada – Alberta** | [Lake Louise](geotifs/lake_louise/) | Landsat 9 + Sentinel-2 + SRTM + NDVI | Glacial lake, rasterio basics |
| **Canada – Nova Scotia** | [Lunenburg](geotifs/lunenburg/) | Sentinel-2 (7 bands) + SRTM | Coastal analysis, maritime environment |
| **Canada – British Columbia** | [Sea to Sky Gondola](geotifs/sea_to_sky_gondola/) | Landsat 8 Red + NIR | NDVI basics, single-band workflows |
| **Canada – Quebec** | [Quebec City](geotifs/quebec_city/) | MODIS NDVI | Vegetation phenology, regional analysis |

[**→ Browse GeoTIFF Datasets**](geotifs/)

### 🔜 Coming Soon
Additional dataset categories planned:
- Vector geospatial data (shapefiles, GeoJSON)
- Point cloud data (LiDAR)
- Labeled training datasets for supervised learning
- Climate and environmental monitoring data
- Tabular datasets for statistical analysis

## 🎯 Key Features

### ✅ Comprehensive Documentation
Each dataset includes:
- **Detailed README** with metadata, specifications, and context
- **Processing scripts** (Google Earth Engine, Python)
- **Working code examples** (3-5 per dataset)
- **Use cases and applications**
- **Suggested exercises** for students

### ✅ Educational Focus
- **Progressive difficulty**: Beginner → Intermediate → Advanced
- **Real-world applications**: Urban analysis, water quality, terrain modeling, coastal remote sensing
- **Reproducible workflows**: Complete scripts for data generation
- **Cross-platform**: Works with Python, R, QGIS, and other tools

### ✅ Quality Assured
- Sourced from reputable public repositories (NASA, USGS, ESA/Copernicus)
- Consistent file naming and organization
- Standard formats and projections
- Proper metadata and provenance

## 🚀 Quick Start

### For Students

1. **Choose your level:**
   - **Beginner**: Start with [Mark Twain Lake](geotifs/mark_twain_lake/) or [Sea to Sky Gondola](geotifs/sea_to_sky_gondola/) (raster basics)
   - **Intermediate**: Try [Lake of the Ozarks](geotifs/lake_of_the_ozarks/) or [Lunenburg](geotifs/lunenburg/) (terrain & multispectral)
   - **Advanced**: Explore [Columbia](geotifs/columbia/), [Utah Lake](geotifs/utah_lake/), or [Albuquerque](geotifs/albuquerque/) (ML, water quality, multi-scale)

2. **Download the data** from the specific dataset directory

3. **Follow the examples** in the dataset's README

### For Instructors

1. **Browse available datasets** by category and difficulty level
2. **Review documentation** to align with course objectives
3. **Use provided examples** as starting points for assignments
4. **Adapt GEE scripts** to create custom datasets for your region

### Prerequisites

```bash
# Install required Python packages
pip install rasterio numpy matplotlib scipy scikit-learn geopandas
```

```python
# Basic usage example
import rasterio
import matplotlib.pyplot as plt

with rasterio.open('path/to/dataset.tif') as src:
    data = src.read(1)  # Read first band
    plt.imshow(data, cmap='viridis')
    plt.colorbar()
    plt.show()
```

## 📚 Documentation Structure

```
educational_datasets/
├── README.md (overview - you are here)
├── geotifs/
│   ├── README.md (GeoTIFF collection overview)
│   ├── albuquerque/
│   │   ├── README.md
│   │   └── *.tif
│   ├── banff_town/
│   │   ├── README.md
│   │   └── *.tif
│   ├── columbia/
│   │   ├── README.md
│   │   └── *.tif
│   ├── lake_louise/
│   │   ├── README.md
│   │   └── *.tif
│   ├── lake_of_the_ozarks/
│   │   ├── README.md
│   │   └── *.tif
│   ├── lunenburg/
│   │   ├── README.md
│   │   └── *.tif
│   ├── mark_twain_lake/
│   │   ├── README.md
│   │   └── *.tif
│   ├── quebec_city/
│   │   ├── README.md
│   │   └── *.tif
│   ├── sea_to_sky_gondola/
│   │   ├── README.md
│   │   └── *.tif
│   └── utah_lake/
│       ├── README.md
│       └── *.tif
└── [future categories]/
```

## 🔬 Data Sources & Credits

All datasets are derived from reputable public sources:

- **Landsat Program**: USGS Landsat 8/9 Level-2, Collection 2, Tier 1
  - [USGS Landsat Missions](https://www.usgs.gov/landsat-missions)
  - DOI: [10.5066/P9OGBGM6](https://doi.org/10.5066/P9OGBGM6)

- **Sentinel-2**: ESA Copernicus Sentinel-2 Level-2A Surface Reflectance
  - [Copernicus Open Access Hub](https://scihub.copernicus.eu/)
  - Available via [Google Earth Engine](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)

- **MODIS**: NASA Terra MODIS Vegetation Indices (MOD13Q1) and Land Surface Temperature (MOD11A2)
  - [NASA LP DAAC](https://lpdaac.usgs.gov/)

- **SRTM**: NASA Shuttle Radar Topography Mission
  - [USGS SRTM Archive](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1)
  - DOI: [10.5069/G9445JDF](https://doi.org/10.5069/G9445JDF)

- **Processing Platform**: [Google Earth Engine](https://earthengine.google.com/)

**All data in this repository is in the public domain** and freely available for educational, research, and commercial use.

## 📖 Citation

If you use these datasets in your research, publications, or educational materials, please cite both this repository and the original data sources.

**This Repository:**
```
Dastour, H. (2026). Educational Datasets Repository. 
University of Missouri. https://github.com/HatefDastour/educational_datasets
```

**Original Data Sources:** See individual dataset READMEs for specific citations.

## 🤝 Contributing

This repository is actively maintained. We welcome:
- **Feedback** on dataset quality and documentation
- **Suggestions** for new datasets or locations
- **Issue reports** for errors or improvements
- **Use cases** and success stories from your classroom

Please open an issue or contact the repository maintainer.

## 📧 Contact & Support

**Maintainer**: Dr. Hatef Dastour  
**Position**: Assistant Teaching Professor of Data Science and Analytics  
**Institution**: University of Missouri  
**Research Focus**: Geospatial AI, Remote Sensing, Environmental Data Science

**For questions:**
- Open an [issue](../../issues) for dataset-specific questions
- Contact through university channels for collaboration inquiries

## 📄 License

This repository structure and documentation are provided under the MIT License. Individual datasets retain their original public domain status from USGS/NASA/ESA sources. See individual dataset directories for specific licensing information.

## 🌟 Acknowledgments

- **USGS** for providing free and open access to Landsat data
- **NASA** for SRTM elevation data and MODIS products
- **ESA / Copernicus** for Sentinel-2 data
- **Google Earth Engine** for cloud-based processing infrastructure
- **University of Missouri** for supporting educational data curation
- **Students and colleagues** who provided feedback on dataset quality and usability

---

**Last Updated**: March 2026  
**Repository Status**: Active Development  
**Total Datasets**: 10 locations with 39 GeoTIFF files
