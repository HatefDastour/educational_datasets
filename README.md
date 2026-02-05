# Educational Datasets Repository

This repository serves as a centralized collection of curated datasets for educational purposes in data science, geospatial analysis, and machine learning applications. The datasets are organized by type and include comprehensive documentation to facilitate learning and research.

## Repository Structure

Currently, this repository contains the following dataset categories:

### GeoTIFFs

Geospatial raster datasets in GeoTIFF format for remote sensing and spatial analysis applications.

- **[Lake of the Ozarks](geotifs/lake_of_the_ozarks/)**: SRTM elevation data and Landsat 9 multispectral imagery for a study area in Missouri
  - SRTM 30m Digital Elevation Model
  - Landsat 9 RGB Composite (2023 growing season, scaled)
  - Landsat 9 Multispectral Composite (2023 growing season, unscaled, 7 bands)

- **[Columbia, MO](geotifs/columbia/)**: Landsat 9 urban and mixed-use area dataset covering the University of Missouri campus and surrounding region
  - Landsat 9 Multispectral Composite (Summer 2025, unscaled, 7 bands)
  - Ideal for urban analysis, vegetation monitoring, and land cover classification

## Educational Use

These datasets are designed to support:
- Hands-on learning in remote sensing and GIS courses
- Machine learning and deep learning model development
- Data science project demonstrations
- Research methodology training
- Reproducible analysis examples
- Urban and environmental analysis
- Spectral index calculation and interpretation
- Land cover classification exercises

## Data Sources

Datasets are sourced from reputable public repositories including:
- NASA/USGS Earth observation missions
- Google Earth Engine Data Catalog
- Other open geospatial data platforms

Each dataset directory contains detailed documentation about data provenance, processing steps, and licensing information.

## Getting Started

Navigate to the specific dataset directory to access:
- Data files in standard formats
- Detailed README with metadata and specifications
- Google Earth Engine scripts for data reproduction
- Python usage examples and tutorials
- Educational exercise suggestions

## Dataset Comparison

| Location | Area Type | Temporal Coverage | Bands | Key Features |
|----------|-----------|-------------------|-------|-------------|
| Lake of the Ozarks | Rural/Natural | April-Oct 2023 | 1, 3, or 7 | Terrain, water bodies, forests |
| Columbia, MO | Urban/Mixed | June-Aug 2025 | 7 | Campus, urban areas, agriculture |

## Contributing

This repository is maintained for educational purposes. For questions, suggestions, or issues, please open an issue or contact the repository maintainer.

## Future Additions

Planned dataset categories include:
- Additional Missouri locations (Kansas City, St. Louis)
- Multi-temporal datasets for change detection
- Time series data
- Point cloud datasets
- Vector geospatial data
- Labeled training datasets for supervised learning
- Climate and environmental monitoring data

## License

Individual datasets may have different licenses based on their original sources. Please refer to the README in each dataset directory for specific licensing information and usage restrictions. All Landsat data is in the public domain.

## About

This repository is maintained by Dr. Hatef Dastour, Assistant Teaching Professor of Data Science and Analytics at the University of Missouri, for educational and research purposes.
