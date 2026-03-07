# GeoTIFF Datasets Collection

This directory contains curated geospatial raster datasets in GeoTIFF format for educational purposes in remote sensing, geospatial analysis, and machine learning applications.

## Available Datasets

### USA – Missouri

#### [Columbia, MO](columbia/) - Urban and Mixed-Use
- **Location**: Columbia metropolitan area, University of Missouri campus
- **Data**: Landsat 9 Multispectral (7 bands, Summer 2025, unscaled DN values)
- **File**: `landsat9_como_30m_unscaled_all_bands.tif`
- **Best For**: Urban analysis, land cover classification, campus mapping, ML applications

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

### USA – Utah

#### [Utah Lake](utah_lake/) - Large Freshwater Lake
- **Location**: Utah Lake, north-central Utah
- **Data**:
  - Landsat 9 (8 bands: 7 optical + thermal, Summer 2024, scaled)
  - Landsat annual summer composites 2020–2024 (5 files, 8 bands each)
  - Sentinel-2 monthly composites April–October 2024 (7 files, 7 bands each, 10 m)
- **Files**: `utah_lake_l9_summer_2024.tif`, `utah_lake_landsat_summer_YYYY.tif`, `utah_lake_s2_2024_30m_MM.tif`
- **Best For**: Water quality analysis, thermal remote sensing, temporal trend detection

### USA – New Mexico

#### [Albuquerque, NM](albuquerque/) - Arid Urban
- **Location**: Albuquerque, New Mexico – Rio Grande Valley and Sandia Mountains
- **Data**:
  - MODIS NDVI (250 m, July 2025)
  - MODIS Land Surface Temperature day (1 km, July 2025)
  - SRTM DEM (30 m)
- **Files**:
  - `albuquerque_modis_ndvi_july2025.tif`
  - `albuquerque_modis_lst_day_july2025.tif`
  - `albuquerque_srtm_elevation.tif`
- **Best For**: Multi-scale sensor fusion, urban heat island analysis, lapse rate studies

### Canada – Alberta

#### [Banff Town](banff_town/) - Alpine Town, Multi-Sensor
- **Location**: Banff Townsite, Bow Valley, Alberta
- **Data**:
  - Sentinel-2 L2A (7 bands, 10 m, Summer 2025)
  - Landsat 9 (5 bands RGB+NIR+thermal, 30 m, Summer 2025)
  - SRTM DEM (30 m)
- **Files**:
  - `banff_town_s2_summer2025.tif`
  - `banff_town_l9_summer2025.tif`
  - `banff_town_srtm_elevation.tif`
- **Best For**: Multi-sensor analysis, urban-river-mountain interaction, sensor comparison

#### [Lake Louise](lake_louise/) - Glacial Lake
- **Location**: Lake Louise, Banff National Park, Alberta
- **Data**:
  - Landsat 9 RGB+NIR (4 bands, 30 m, Summer 2025)
  - Sentinel-2 RGB+NIR (4 bands, 10 m, Summer 2025)
  - SRTM DEM (30 m)
  - NDVI from Landsat 9 (derived)
  - NDVI from Sentinel-2 (derived)
- **Files**:
  - `lake_louise_l9_rgb_nir_2025.tif`
  - `lake_louise_s2_rgb_nir_2025.tif`
  - `lake_louise_srtm_dem.tif`
  - `lake_louise_ndvi_l9_2025.tif`
  - `lake_louise_ndvi_s2_2025.tif`
- **Best For**: Rasterio fundamentals, reprojection/resampling, multi-sensor comparison

### Canada – Nova Scotia

#### [Lunenburg](lunenburg/) - Coastal Town
- **Location**: Lunenburg, NS – UNESCO World Heritage port town
- **Data**:
  - Sentinel-2 L2A (7 bands: coastal, blue, green, red, NIR, SWIR1, SWIR2; 10 m, Summer 2025)
  - SRTM DEM (30 m)
- **Files**:
  - `lunenburg_s2_summer2025.tif`
  - `lunenburg_srtm_30m.tif`
- **Best For**: Coastal remote sensing, multi-spectral analysis, DEM integration

### Canada – British Columbia

#### [Sea to Sky Gondola](sea_to_sky_gondola/) - Mountain Terrain
- **Location**: Sea to Sky Gondola area near Squamish, BC
- **Data**:
  - Landsat 8 Red band (30 m, Summer 2025)
  - Landsat 8 NIR band (30 m, Summer 2025)
- **Files**:
  - `seatosky_l8_red_summer2025.tif`
  - `seatosky_l8_nir_summer2025.tif`
- **Best For**: Single-band workflows, NDVI calculation from separate files, rasterio basics

### Canada – Quebec

#### [Quebec City](quebec_city/) - Urban Corridor
- **Location**: Quebec City, QC – St. Lawrence River corridor
- **Data**:
  - MODIS NDVI (250 m, growing season 2025 median composite)
- **File**: `quebec_city_modis_ndvi_summer2025.tif`
- **Best For**: Vegetation phenology, regional raster analysis, coarse-resolution remote sensing

---

## Dataset Comparison

| Location | Country | Type | Temporal | Key Datasets | Resolution | Features |
|----------|---------|------|----------|--------------|------------|---------|
| **Columbia** | USA | Urban | Summer 2025 | L9 (7 bands, unscaled) | 30 m | Campus, urban, agriculture |
| **Lake of the Ozarks** | USA | Rural | Growing 2023 | DEM + L9 (3 & 7 bands) | 30 m | Terrain, water, forests |
| **Mark Twain Lake** | USA | Reservoir | July 2025 | DEM + L9 (4 bands) + NDVI | 30 m | Large reservoir, varied terrain |
| **Utah Lake** | USA | Water body | 2020–2024 | L9 (8 bands) + time series + S2 monthly | 10–30 m | Turbid water, thermal, trends |
| **Albuquerque** | USA | Arid urban | July 2025 | MODIS NDVI + LST + SRTM | 30 m–1 km | Multi-scale, heat island |
| **Banff Town** | Canada | Alpine town | Summer 2025 | S2 + L9 + SRTM | 10–30 m | Multi-sensor, river, mountains |
| **Lake Louise** | Canada | Glacial lake | Summer 2025 | L9 + S2 + SRTM + NDVI | 10–30 m | Sensor fusion, glacial terrain |
| **Lunenburg** | Canada | Coastal | Summer 2025 | S2 (7 bands) + SRTM | 10–30 m | Coastal, maritime |
| **Sea to Sky** | Canada | Mountain | Summer 2025 | L8 Red + NIR | 30 m | Minimal dataset, NDVI intro |
| **Quebec City** | Canada | Urban/Rural | Summer 2025 | MODIS NDVI | 250 m | Phenology, coarse-resolution |

---

## Learning Progression

### Beginner Level
**Best starting points:**
- **[Sea to Sky Gondola](sea_to_sky_gondola/)** – Minimal 2-file dataset; just Red and NIR bands for NDVI
- **[Mark Twain Lake](mark_twain_lake/)** – 3 complementary rasters covering multi-band, single-band, and derived data
- **[Quebec City](quebec_city/)** – Single-band MODIS; coarse-resolution, lightweight analysis

Key skills: reading GeoTIFFs with rasterio, single vs. multi-band operations, basic visualisation, simple thresholding.

### Intermediate Level
**Progress to:**
- **[Lake of the Ozarks](lake_of_the_ozarks/)** – Terrain analysis (slope, aspect) + spectral indices
- **[Lunenburg](lunenburg/)** – 7-band Sentinel-2; coastal land–water separation
- **[Lake Louise](lake_louise/)** – Multi-sensor, multi-resolution rasters; reprojection workflows
- **[Albuquerque](albuquerque/)** – Multi-scale sensor fusion (30 m, 250 m, 1 km)

Key skills: multi-band spectral indices, false-colour composites, DEM derivatives, multi-resolution fusion, data type conversion.

### Advanced Level
**Explore:**
- **[Columbia](columbia/)** – 7-band Landsat; unsupervised/supervised ML classification
- **[Banff Town](banff_town/)** – Multi-sensor comparison (S2 10 m vs. L9 30 m + thermal)
- **[Utah Lake](utah_lake/)** – Temporal time series, thermal remote sensing, water quality monitoring

Key skills: machine learning, time series analysis, thermal remote sensing, multi-sensor fusion, change detection.

---

## Technical Specifications

### Sensors and Products

| Sensor | Product | Resolution | Scaling | Datasets |
|--------|---------|------------|---------|---------|
| Landsat 8/9 L2 | Surface Reflectance (optical) | 30 m | DN × 0.0000275 − 0.2 | Columbia, Lake of the Ozarks, Mark Twain Lake, Utah Lake, Banff Town, Lake Louise, Sea to Sky |
| Landsat 9 L2 | Land Surface Temperature (ST_B10) | 30 m | DN × 0.00341802 + 149.0 (K) | Utah Lake, Banff Town |
| Sentinel-2 L2A | Surface Reflectance | 10–20 m | ÷ 10 000 → 0–1 | Utah Lake, Banff Town, Lake Louise, Lunenburg |
| MODIS MOD13Q1 | NDVI (16-day composites) | 250 m | × 0.0001 → 0–1 | Albuquerque, Quebec City |
| MODIS MOD11A2 | LST Day (8-day composites) | 1 km | × 0.02 − 273.15 (°C) | Albuquerque |
| NASA SRTM GL1 | Elevation | 30 m | Metres a.s.l. | Lake of the Ozarks, Mark Twain Lake, Albuquerque, Banff Town, Lake Louise, Lunenburg |

### Coordinate Reference Systems

- **Missouri datasets (Columbia, Lake of the Ozarks, Mark Twain Lake)**: EPSG:32615 (WGS 84 / UTM zone 15N)
- **All other datasets**: EPSG:4326 (WGS 84 Geographic)

### Common Properties
- **Format**: GeoTIFF
- **Data Types**: Float32, Int16, UInt16 (varies by dataset)
- **Processing Platform**: Google Earth Engine
- **Cloud Masking**: QA_PIXEL (Landsat), QA60 (Sentinel-2)

---

## File Naming Convention

```
[sensor/source]_[location]_[resolution]_[type/bands]_[year/month].tif
```

**Examples:**
- `landsat9_como_30m_unscaled_all_bands.tif`
- `srtm_lake_ozarks_30m_composite.tif`
- `mark_twain_ndvi_2025.tif`
- `utah_lake_l9_summer_2024.tif`
- `utah_lake_s2_2024_30m_07.tif`
- `banff_town_s2_summer2025.tif`
- `albuquerque_modis_ndvi_july2025.tif`

---

## Data Sources

All datasets are derived from public domain sources:

- **Landsat 8/9**: USGS Landsat 8/9 Level-2, Collection 2, Tier 1 Surface Reflectance
  - Optical bands scaled: DN × 0.0000275 − 0.2
  - Thermal bands scaled: DN × 0.00341802 + 149.0 (Kelvin)
  - [USGS Landsat Missions](https://www.usgs.gov/landsat-missions)

- **Sentinel-2**: ESA Copernicus Sentinel-2 L2A Surface Reflectance
  - Bands scaled: ÷ 10 000 (reflectance 0–1)
  - [Copernicus Open Access Hub](https://scihub.copernicus.eu/)

- **MODIS**: NASA Terra MODIS (MOD13Q1 NDVI, MOD11A2 LST)
  - NDVI: × 0.0001; LST: × 0.02 − 273.15 °C
  - [NASA LP DAAC](https://lpdaac.usgs.gov/)

- **SRTM**: NASA Shuttle Radar Topography Mission Global 1 arc-second
  - Void-filled elevation data
  - [USGS SRTM](https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-shuttle-radar-topography-mission-srtm-1)

- **Processing**: Google Earth Engine Data Catalog
  - Cloud masking, compositing, and export
  - [Google Earth Engine](https://earthengine.google.com/)

---

## Getting Started

### Prerequisites

```python
# Required Python packages
import rasterio
import numpy as np
import matplotlib.pyplot as plt

# Optional but recommended
from scipy.ndimage import sobel       # Terrain analysis
from sklearn.cluster import KMeans    # Classification
import geopandas as gpd               # Vector operations
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
    data = src.read()   # All bands
    band1 = src.read(1) # First band only
    
    # Visualize
    plt.imshow(band1, cmap='gray')
    plt.colorbar()
    plt.show()
```

---

## Educational Applications

### By Dataset

**Sea to Sky Gondola** – Introductory single-band workflows
- Lesson 1: Reading a single-band GeoTIFF
- Lesson 2: NDVI from two separate band files
- Lesson 3: Stacking bands and writing a multi-band output

**Mark Twain Lake** – Introductory remote sensing courses
- Week 1–2: Raster data fundamentals
- Week 3: Water body detection using NDVI
- Week 4: Basic terrain analysis with DEM

**Lake of the Ozarks** – Intermediate GIS/remote sensing
- Multi-band analysis and spectral indices
- Terrain derivatives (slope, aspect)
- Data integration workflows

**Lake Louise / Banff Town** – Multi-sensor comparison
- Resolution comparison (10 m S2 vs. 30 m L9)
- Reprojection and resampling
- Sensor calibration differences

**Albuquerque / Quebec City** – Coarse-resolution applications
- Multi-scale data fusion
- Vegetation phenology
- Urban heat island analysis

**Columbia** – Advanced topics and capstone projects
- Urban remote sensing
- Machine learning classification
- Feature extraction and mapping

**Utah Lake** – Water resources and environmental courses
- Water quality monitoring
- Thermal remote sensing
- Temporal change detection (2020–2024)

### Cross-Dataset Comparisons

1. **Urban vs. Rural**: Columbia (Missouri) vs. Lake of the Ozarks
2. **Water Bodies**: Mark Twain Lake vs. Utah Lake (clear vs. turbid)
3. **Sensor Resolution**: Lake Louise or Banff Town (S2 10 m vs. L9 30 m)
4. **Coastal vs. Inland**: Lunenburg vs. Columbia
5. **Multi-Scale MODIS**: Albuquerque (250 m NDVI + 1 km LST + 30 m SRTM)
6. **Temporal**: Utah Lake time series 2020–2024

---

## Documentation

Each dataset directory contains:
- **README.md**: Detailed documentation
- **GEE Scripts**: Complete Google Earth Engine code for reproduction
- **Python Examples**: 2–5 working code examples
- **Use Cases**: Specific applications and exercises
- **Metadata**: Full band descriptions and specifications

---

## Citation

When using these datasets, please cite the original sources:

**Landsat 8/9:**
> U.S. Geological Survey, 2023–2025, Landsat 8/9 Level-2, Collection 2, Tier 1: U.S. Geological Survey data release, https://doi.org/10.5066/P9OGBGM6

**Sentinel-2:**
> Copernicus Sentinel-2 L2A Surface Reflectance. ESA / Copernicus Programme. Available via Google Earth Engine (COPERNICUS/S2_SR_HARMONIZED).

**MODIS:**
> Didan, K. (2015). MOD13Q1 MODIS/Terra Vegetation Indices 16-Day L3 Global 250m SIN Grid V006. NASA EOSDIS Land Processes DAAC. https://doi.org/10.5067/MODIS/MOD13Q1.006

**SRTM:**
> NASA Shuttle Radar Topography Mission (SRTM) (2013). Shuttle Radar Topography Mission (SRTM) Global. Distributed by OpenTopography. https://doi.org/10.5069/G9445JDF

---

## License

All datasets are derived from public domain sources and are freely available for educational, research, and commercial use.

## Contributing

For questions, suggestions, or to report issues, please open an issue in the main repository.

## About

These datasets are curated and maintained by Dr. Hatef Dastour, Assistant Teaching Professor of Data Science and Analytics at the University of Missouri, for educational and research purposes in geospatial AI and environmental data science.
