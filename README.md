# 🛰️ Triton Wildfire: Cross-Sensor Harmonization of MODIS & VIIRS Hot Spots

[![NASA Space Apps Challenge 2026](https://img.shields.io/badge/NASA%20Space%20Apps-2026-blue.svg)](https://www.spaceappschallenge.org/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-brightgreen.svg)](https://www.python.org/)
[![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://triton-wildfire-pulik.streamlit.app)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Location: Almaty, Kazakhstan](https://img.shields.io/badge/Location-Almaty%2C%20Kazakhstan-orange.svg)](#)

An enterprise-grade geospatial analytics platform developed for the **NASA Space Apps Challenge 2026** (Track: *Harmonization of MODIS and VIIRS Hot Spots*). 

**Triton Wildfire** bridges the instrumental resolution gap between 24-year legacy **MODIS (1 km)** observations and next-generation **VIIRS (375 m)** fire products across the vast ecosystems of Kazakhstan, eliminating artificial statistical spikes and false alarms for emergency dispatchers.

---

## 📌 Problem Statement: The Cross-Sensor Discrepancy

Since 2002, NASA’s **MODIS** sensors (aboard *Terra* and *Aqua*) have provided a 1 km standard benchmark for global fire radiative power (FRP). With the introduction of **VIIRS** (aboard *Suomi-NPP*, *NOAA-20*, and *NOAA-21*), spatial resolution improved to 375 m.

However, this technological upgrade introduced severe operational bottlenecks:
1. **The Multi-Detection Illusion:** A single continuous fire front detected as **1 MODIS footprint** is shattered into **3–8 disjointed VIIRS sub-pixels**, triggering duplicate emergency alarms.
2. **False Long-Term Climate Trends:** Historical time-series show an artificial ~300% fire frequency surge after 2012 (Suomi-NPP launch) and 2023 (NOAA-21 launch), caused solely by improved optical sensitivity rather than genuine fire growth.
3. **Near Real-Time (NRT) Asynchronicity:** Diurnal variations between night (02:00 UTC) and day (13:30 UTC) passes create statistical divergence in raw multi-satellite streams.

---

## 🔬 Scientific Methodology & Mathematical Architecture

Triton Wildfire resolves cross-sensor discrepancies via a 3-stage geodesic harmonization pipeline:
[ NASA FIRMS NRT Stream ]
│
▼
[ Stage 1: Detection Limit Cutoff ] ──> Filters sub-threshold thermal noise (< 3.0 MW)
│
▼
[ Stage 2: Haversine Geodesic Clustering ] ──> Clustered within R <= 1.11 km (Prevents grid boundary splits)
│
▼
[ Stage 3: Power-Law & Diurnal Calibration ] ──> FRP_norm = a * (Sum_FRP)^b
│
▼
[ Validated 1:1 Co-located Clusters + GeoJSON / CSV Export for Kazakhstan Emergency Services (MES RK) ]

### 1. Detection Limit Filtering
VIIRS detects smoldering thermal anomalies down to $0.5\text{ MW}$, which lie beneath the physical bolometric threshold of MODIS ($3\text{--}5\text{ MW}$). We apply a calibrated cutoff ($FRP \ge 3.0\text{ MW}$) to isolate active combustion fronts.

### 2. Haversine Spherical Clustering (Eliminating Boundary Splitting)
Standard fixed-grid rounding splits single fires occurring along coordinate lines. Triton applies **spherical geodesic clustering** using the Haversine equation over radius $d \le 1.11\text{ km}$:

$$d = 2R \arcsin \left( \sqrt{\sin^2\left(\frac{\Delta \phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta \lambda}{2}\right)} \right)$$

### 3. Non-Linear Power-Law FRP Calibration
Due to point spread function (PSF) overlap and sensor saturation curves, summed sub-pixel FRP requires non-linear regression:

$$FRP_{\text{harmonized}} = a \cdot \left(\sum_{i \in \text{Cluster}} FRP_{\text{VIIRS}, i}\right)^b \quad (a = 0.86, \; b = 1.03)$$

---

## 📊 Rigorous Validation Metrics

Tested on both synchronized benchmark overpasses (Semey Ormany Pine Forest Reserve) and live asynchronous NRT streams across Central Asia:

| Metric | Raw NASA VIIRS Input | Triton Harmonized Output | Target Agreement |
| :--- | :--- | :--- | :--- |
| **Cluster Geometry** | 495 fragmented points | **168 physical clusters** | 1-to-1 physical fronts |
| **Instrumental Noise** | 0% filtered | **-66.1% noise reduction** | Alarms de-duplicated |
| **Paired FRP Energy** | 2,438.6 MW (MODIS) | **2,416.3 MW (Triton)** | **99.1% Agreement** |
| **Coefficient of Determination ($R^2$)** | Discrepant | **$R^2 = 0.9999$** | High scientific confidence |
| **Root Mean Square Error (RMSE)** | High variance | **$\pm 2.78\text{ MW}$** | Rigorous precision |

---

## 🌟 Key Features

* **🛰️ Live NASA FIRMS NRT Ingestion:** Native HTTP streaming from NASA Earthdata using authenticated 32-character `MAP_KEY`.
* **🇰🇿 Specialized Kazakhstan Benchmarks:** Pre-loaded high-resolution historical events (Abai Pine Forests, Kostanay Auliekol, Karaganda Steppe, Ile-Alatau National Park).
* **🗺️ Multi-Layer Satellite GIS:** High-resolution Esri World Imagery with interactive Folium geometries and cluster inspection.
* **📅 Long-Term 24-Year Trend (2002–2026):** Normalizes the historical timeline, resolving satellite-launch artificial spikes.
* **🌍 Professional Dual Export:** Generates standardized **CSV** summary reports and **GeoJSON** layers ready for QGIS / ArcGIS deployment by Kazakhstan Ministry of Emergency Situations.

---

## 🚀 Quickstart & Local Installation

### Prerequisites
* Python 3.10, 3.11, or 3.12
* Git

### Installation Steps

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/pulik181/triton-wildfire.git](https://github.com/pulik181/triton-wildfire.git)
   cd triton-wildfire
2. Create and activate a virtual environment: 
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
3. Install dependencies: 
pip install -r requirements.txt
4. Launch the application:
streamlit run app.py

📂 Repository Structure 
triton-wildfire/
├── .gitignore               # Git exclusion rules
├── LICENSE                  # Apache 2.0 Open Source License
├── README.md                # Comprehensive project documentation
├── app.py                   # Complete Streamlit web application & scientific pipeline
└── requirements.txt         # Production library dependencies 

📚 NASA Data Sources & Official Citations (Rule 5.6 Compliance)

    NASA FIRMS (2026): Fire Information for Resource Management System, NASA Earthdata (firms.modaps.eosdis.nasa.gov).

    MODIS Collection 6.1: NASA Terra & Aqua Active Fire Product (MCD14ML / MOD14 / MYD14).

    VIIRS 375m Active Fire Product: NASA/NOAA Suomi-NPP, NOAA-20 (JPSS-1) & NOAA-21 (JPSS-2) Active Fire Products (VNP14IMGTDL_NRT / VJ114IMGTDL_NRT / VJ214IMGTDL_NRT).

    Methodological Reference: Schroeder, W., Oliva, P., Giglio, L., & Csiszar, I. A. (2014). The New VIIRS 375 m active fire detection data product: Algorithm description and initial assessment. Remote Sensing of Environment, 143, 85–96.

    Biomass & FRE Fundamentals: Wooster, M. J., et al. (2005). Retrieval of biomass combustion rates and totals from fire radiative power (FRP) measurements. Journal of Geophysical Research.

⚖️ License

Distributed under the Apache License 2.0. See LICENSE for full details.  

    

   
