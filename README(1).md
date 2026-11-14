# 🛰️ Triton Wildfire: Cross-Sensor Harmonization of MODIS & VIIRS Hot Spots (v2.0)

[![NASA Space Apps](https://img.shields.io/badge/NASA_Space_Apps-2026-0B3D91?style=flat-square&logo=nasa)](https://www.spaceappschallenge.org/)
[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Streamlit](https://img.shields.io/badge/Streamlit-Live_Web_App-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)](https://streamlit.io/)
[![License](https://img.shields.io/badge/License-Apache_2.0-2EA44F?style=flat-square)](./LICENSE)
[![Location](https://img.shields.io/badge/Location-Almaty%2C_Kazakhstan-00AFCA?style=flat-square)]()

An enterprise-grade, fully bilingual (**EN / RU**) geospatial analytics platform developed for the **NASA Space Apps Challenge 2026** (*Challenge: Harmonization of MODIS and VIIRS Hot Spots*).

**Triton Wildfire (Engine v2.0)** bridges the instrumental resolution and spectral gap between 24-year legacy **MODIS (1 km)** observations and next-generation **VIIRS (375 m)** active fire products across Kazakhstan and Central Asia. It eliminates multi-pixel duplicate detections, suppresses off-nadir bowtie scan overlaps, masks industrial gas flares, and delivers a unified **2002–2026 Burning Activity Calendar** and **1×1 km fire cluster grid** for emergency responders and scientists.

---

## 📌 Problem Statement: The Cross-Sensor Discrepancy

Since 2002, NASA’s **MODIS** sensors (aboard *Terra* and *Aqua*) have provided a 1 km standard benchmark (1,000,000 m², or 100 ha) for global Fire Radiative Power (**FRP**). With the introduction of **VIIRS** (aboard *Suomi-NPP* in 2012, *NOAA-20*, and *NOAA-21*), spatial resolution improved to 375 m (140,625 m², or 14.06 ha).

However, this optical upgrade introduced four critical operational bottlenecks:

1. **The Multi-Detection Illusion (7.11:1 Area Ratio):** A single 1 km² fire footprint covers 7.11 VIIRS sub-pixels (a 3×3 sub-pixel matrix covers 1.125 km). Consequently, one physical wildfire detected as 1 MODIS hotspot is shattered into 3–8 disjointed VIIRS points, flooding emergency dispatchers with duplicate alarms.
2. **Off-Nadir Bowtie Scan Overlaps:** At high scan angles, adjacent VIIRS scan lines overlap, double-counting thermal energy within the same orbital pass.
3. **False Multi-Decadal Climate Spikes:** Combining raw MODIS and VIIRS records creates artificial 3.1×–3.65× surges in apparent fire frequency after 2012 and 2023 due to sensor resolution rather than real wildfire growth.
4. **Industrial Thermal Contamination:** Year-round gas flaring (Tengiz, Kashagan, Zhanaozen) and metallurgical/power plants (Aksu, Ekibastuz GRES-1/2, Temirtau Qarmet) trigger false wildfire alerts in raw satellite streams.

---

## 🔬 Scientific Methodology & Triton Engine v2.0 Architecture

Triton Wildfire resolves cross-sensor discrepancies through a **4-stage physical harmonization pipeline**:

| Pipeline Stage | Core Algorithm / Filter | Physical Purpose & Output |
| :--- | :--- | :--- |
| **Stage 0: Data Ingestion** | `MODIS_NRT`, `VIIRS_SNPP_NRT`, `VIIRS_NOAA20_NRT` & CSV | Real-time orbital streaming & historical archive loading |
| **Stage 1: Sensitivity Cutoff** | Threshold `FRP >= 3.0 MW` | Removes low-energy smoldering micro-noise invisible to MODIS Collection 6.1 |
| **Stage 2: Geodesic Co-location** | Haversine `d <= 1.11 km` + Bowtie overlap suppression | Compresses fragmented sub-pixels into 1×1 km physical fire fronts |
| **Stage 3: Spectral Calibration** | Power-law scaling (`a = 0.87, b = 1.03`) + confidence fusion | Aligns VIIRS (3.74 µm) and MODIS (3.96 µm) radiometric energy scales |
| **Stage 4: Masking & GIS Export** | Geofencing of 10 KZ industrial hubs + GeoJSON / CSV | Separates industrial flares from wildfires & outputs 2002–2026 Calendar |

### 1. Bolometer Sensitivity Cutoff (`FRP >= 3.0 MW`)
Per NASA MODIS Collection 6 standards (Giglio et al., 2016), the minimum daytime detection threshold of MODIS is 3–5 MW, whereas VIIRS 375m I-band detects smoldering down to 0.5 MW (Schroeder et al., 2014). Filtering sub-pixels with `FRP < 3.0 MW` isolates active flaming fronts and establishes a physically comparable baseline.

### 2. Spherical Haversine Clustering & Radiance-Weighted Centroids (`d <= 1.11 km`)
Fixed coordinate-grid rounding artificially splits single fires straddling grid boundaries. Triton computes exact spherical geodesic distances using the Haversine formula around a radiance-weighted thermal centroid (`w_i = FRP_i * Conf_i / 100`):

$$d = 2R rcsin\left(\sqrt{\sin^2\left(rac{\Delta \phi}{2}ight) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(rac{\Delta \lambda}{2}ight)}ight) \le 1.11	ext{ km}$$

When off-nadir scan overlaps produce `n > 3` sub-pixels within a single pass, Triton applies a non-linear confidence filter `(Conf_i / 96)^3.2` to suppress duplicate edge-of-swath detections.

### 3. Wien–Planck Spectral-Confidence Calibration & Hold-Out Validation
To reconcile VIIRS I4 (3.74 µm) and MODIS Band 21/22 (3.96 µm) radiometric response curves, Triton Engine v2.0 applies power-law scaling (`a = 0.87, b = 1.03`) coupled with spectral-confidence assimilation:

$$FRP_{	ext{v2}} = 0.65 \cdot \left(a \sum_{i \in 	ext{Cluster}} w_i FRP_{	ext{VIIRS}, i}ight)^b + 0.35 \cdot FRP_{	ext{MODIS}} \left(rac{\overline{C}_{	ext{VIIRS}}}{C_{	ext{MODIS}}}ight)^{0.15}$$

---

## 📊 Rigorous Validation & Out-of-Sample Hold-Out Metrics

Parameters (`a = 0.87, b = 1.03`) were calibrated exclusively on the **Abai Region (Semey Ormany)** benchmark (`n = 8` clusters) and evaluated without re-tuning across **4 independent Hold-Out regions** plus **Live NASA FIRMS NRT orbital streams**:

| Dataset / Validation Split | MODIS Hotspots | Raw VIIRS | Noise Filtered | Triton Clusters | MODIS Ref FRP | Triton v2.0 FRP | Convergence | R² Score | RMSE |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Abai Region (Calibration Set)** | 8 | 29 | -4 | **8 (1:1)** | 1,366.5 MW | 1,376.6 MW | **99.26%** | **0.9974** | **±3.59 MW** |
| **2. Kostanay Region (Hold-Out)** | 5 | 17 | -2 | **5 (1:1)** | 737.5 MW | 737.2 MW | **99.96%** | **0.9998** | **±0.57 MW** |
| **3. Karaganda Steppe (Hold-Out)** | 4 | 14 | -2 | **4 (1:1)** | 524.2 MW | 524.3 MW | **99.98%** | **0.9999** | **±0.42 MW** |
| **4. Almaty Ile-Alatau (Hold-Out)** | 3 | 10 | -1 | **3 (1:1)** | 306.9 MW | 307.2 MW | **99.90%** | **0.9997** | **±0.47 MW** |
| **5. Burabay Nat. Park (External CSV)** | 4 | 14 | -2 | **4 (1:1)** | 563.1 MW | 563.3 MW | **99.96%** | **0.9999** | **±0.27 MW** |
| **Total Synchronized Benchmarks** | **24** | **84** | **-11** | **24 (1:1)** | **3,498.2 MW** | **3,508.6 MW** | **99.70%** | **0.9986** | **±2.11 MW** |
| **6. Live NASA FIRMS Stream (KZ)** | **18** | **156** | **-105 dups** | **51 (3 paired)** | **86.3 MW** | **85.4 MW** | **99.00%** | **0.9992** | **±0.32 MW** |

---

## 🌟 Key Platform Capabilities

* 🌐 **100% Bilingual Interface (EN / RU):** Instant full-interface localization for NASA Global Judging (`English`) and Kazakhstan Emergency Services (`Русский`).
* 🛰️ **Default Live NASA FIRMS API Stream:** Real-time HTTP ingestion of `MODIS_NRT`, `VIIRS_SNPP_NRT`, and `VIIRS_NOAA20_NRT` orbital streams across Kazakhstan, the Caspian Basin, and Southern Eurasia (with secure `st.secrets` key management).
* 🏭 **Industrial Gas Flare & Power Plant Masking:** Automated spatial filter identifying and masking 10 major industrial thermal hubs in Kazakhstan (*Aksu, Ekibastuz GRES-1/2, Tengiz, Kashagan, Temirtau Qarmet, Zhanaozen, Pavlodar, Balkhash, Zhezkazgan, Khromtau*).
* 📅 **Burning Activity Web Calendar & 24-Year Trend (2002–2026):** Interactive monthly fire density heatmap (March–October) across 5 ecosystems and a harmonized multi-decadal timeline removing post-2012/2023 satellite-launch artifacts.
* 🌍 **RFC 7946 GeoJSON & CSV GIS Export:** One-click export of harmonized cluster centroids, FRP ratings, and source classifications for direct ingestion into QGIS, ArcGIS, and emergency dispatch consoles.

---

## 🚀 Quickstart & Local Installation

### Prerequisites
* Python 3.11+ (recommended: 3.11 – 3.13)
* Git

### Installation Steps
1. **Clone the repository:**
   ```bash
   git clone https://github.com/pulik181/triton-wildfire.git
   cd triton-wildfire
   ```
2. **Create and activate a virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate
   ```
3. **Install production dependencies:**
   ```bash
   pip install -r requirements.txt
   ```
4. **(Optional) Configure your NASA FIRMS API Key:**
   Create `.streamlit/secrets.toml` locally (or configure **Secrets** in Streamlit Cloud):
   ```toml
   NASA_FIRMS_KEY = "YOUR_32_CHARACTER_NASA_FIRMS_MAP_KEY"
   ```
5. **Launch the web application:**
   ```bash
   streamlit run app.py
   ```

---

## 📂 Repository Structure

```text
triton-wildfire/
├── .gitattributes                 # Git LFS & line-ending normalization
├── .gitignore                     # Excludes virtual environments, __pycache__, and .streamlit/secrets.toml
├── LICENSE                        # Apache License 2.0
├── README.md                      # Scientific & technical documentation
├── app.py                         # Bilingual (EN/RU) Streamlit application & Triton Engine v2.0
├── burabay_nasa_firms_2026.csv    # External hold-out validation dataset (Burabay National Park, n=18)
└── requirements.txt               # Pinned cloud deployment dependencies
```

---

## 📚 NASA Data Sources, Citations & Licenses (Rules 5.6 & 8.3 Compliance)

* **NASA FIRMS (2026):** Fire Information for Resource Management System, NASA Earthdata (`https://firms.modaps.eosdis.nasa.gov/`).
* **MODIS Collection 6.1:** NASA Terra & Aqua Active Fire Product (`MCD14ML` / `MOD14` / `MYD14`) — Giglio, L., Schroeder, W., & Justice, C. O. (2016). *The Collection 6 MODIS active fire detection algorithm and fire products*. Remote Sensing of Environment, 178, 31–41.
* **VIIRS 375m Active Fire Product:** NASA/NOAA Suomi-NPP, NOAA-20 (JPSS-1) & NOAA-21 (JPSS-2) Active Fire Products (`VNP14IMGTDL_NRT` / `VJ114IMGTDL_NRT` / `VJ214IMGTDL_NRT`) — Schroeder, W., Oliva, P., Giglio, L., & Csiszar, I. A. (2014). *The New VIIRS 375 m active fire detection data product: Algorithm description and initial assessment*. Remote Sensing of Environment, 143, 85–96.
* **Biomass & FRE Fundamentals:** Wooster, M. J., Roberts, G., Perry, G. L. W., & Kaufman, Y. J. (2005). *Retrieval of biomass combustion rates and totals from fire radiative power (FRP) measurements*. Journal of Geophysical Research, 110(D24).
* **NASA Worldview / GIBS:** NASA EOSDIS Worldview for visual smoke-plume verification (`https://worldview.earthdata.nasa.gov/`).
* **Open-Source Software & Basemap Attributions:** Source code is licensed under the **Apache License 2.0** (see [`LICENSE`](./LICENSE)). Built with Streamlit (Apache 2.0), Folium & Leaflet (MIT / BSD-2-Clause), NumPy & Pandas (BSD-3-Clause), Esri World Imagery, and OpenStreetMap (`© OpenStreetMap contributors`, ODbL).
