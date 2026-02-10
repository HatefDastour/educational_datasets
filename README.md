# Educational Datasets Repository

> A curated collection of high-quality datasets for teaching and learning data science, geospatial analysis, machine learning, and environmental science.

This repository serves as a comprehensive resource for educators, students, and researchers seeking well-documented, ready-to-use datasets for educational purposes. All datasets include detailed documentation, processing scripts, and practical code examples.

## 📊 Available Dataset Categories

### [GeoTIFFs](geotifs/) - Geospatial Raster Data
Remote sensing and terrain datasets in GeoTIFF format, including:
- **Landsat 9 multispectral imagery** (optical and thermal bands)
- **SRTM Digital Elevation Models** (terrain data)
- **Derived products** (NDVI, water indices, composites)

**Current Locations:**
- **Missouri**: Columbia (urban), Lake of the Ozarks (rural), Mark Twain Lake (reservoir)
- **Utah**: Utah Lake (water quality, thermal analysis)

[**→ Browse GeoTIFF Datasets**](geotifs/)

### 🔜 Coming Soon
Additional dataset categories planned:
- Time series data
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
- **Real-world applications**: Urban analysis, water quality, terrain modeling
- **Reproducible workflows**: Complete scripts for data generation
- **Cross-platform**: Works with Python, R, QGIS, and other tools

### ✅ Quality Assured
- Sourced from reputable public repositories (NASA, USGS)
- Consistent file naming and organization
- Standard formats and projections
- Proper metadata and provenance

## 🚀 Quick Start

### For Students

1. **Choose your level:**
   - **Beginner**: Start with [Mark Twain Lake](geotifs/mark_twain_lake/) (raster basics)
   - **Intermediate**: Try [Lake of the Ozarks](geotifs/lake_of_the_ozarks/) (terrain & multispectral)
   - **Advanced**: Explore [Columbia](geotifs/columbia/) or [Utah Lake](geotifs/utah_lake/) (ML & water quality)

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
│   ├── columbia/
│   │   ├── README.md (dataset documentation)
│   │   └── *.tif (data files)
│   ├── lake_of_the_ozarks/
│   ├── mark_twain_lake/
│   └── utah_lake/
└── [future categories]/
```

## 🔬 Data Sources & Credits

All datasets are derived from reputable public sources:

- **Landsat Program**: USGS Landsat 9 Level-2, Collection 2, Tier 1
  - [USGS Landsat Missions](https://www.usgs.gov/landsat-missions)
  - DOI: [10.5066/P9OGBGM6](https://doi.org/10.5066/P9OGBGM6)

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

This repository structure and documentation are provided under the MIT License. Individual datasets retain their original public domain status from USGS/NASA sources. See individual dataset directories for specific licensing information.

## 🌟 Acknowledgments

- **USGS** for providing free and open access to Landsat data
- **NASA** for SRTM elevation data
- **Google Earth Engine** for cloud-based processing infrastructure
- **University of Missouri** for supporting educational data curation
- **Students and colleagues** who provided feedback on dataset quality and usability

---

**Last Updated**: February 2026  
**Repository Status**: Active Development  
**Total Datasets**: 4 locations with 9 GeoTIFF files
