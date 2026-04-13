# 🌍 GeoMeld Dataset Card


GeoMeld is a large-scale multimodal remote sensing dataset designed for **semantically grounded foundation modeling**. It contains approximately **2.5 million spatially aligned samples** spanning heterogeneous sensing modalities and spatial resolutions, paired with **semantically grounded captions** generated through an agentic pipeline.



## 🎯 Motivation

Existing remote sensing datasets typically emphasize either:
- multimodal fusion without language grounding, or  
- language supervision derived from limited modalities (e.g., optical imagery).

GeoMeld addresses this gap by combining:
- **spatially aligned multimodal data**,  
- **cross-resolution signals**, and  
- **semantically grounded captions derived from measurable geospatial attributes**.

This design supports scalable learning of representations that integrate both physical and semantic structure.


## 🧩 Dataset Composition

Each GeoMeld sample consists of a spatially aligned multimodal tuple of different resolution: 

### 🔹 High-resolution (~1m) 
-  NAIP imagery

### 🔹 Medium-resolution (10m, standardized grid)
- **Sentinel-2 (S2)**: multi-spectral optical imagery (12 bands)  
- **Sentinel-1 (S1)**: SAR backscatter (VV, VH, HH, HV)  
- **ASTER-DEM**: elevation  
- **Canopy height**  
- **Land-cover products**:
  - Dynamic World  
  - ESA WorldCover  

### 🔹 Additional components
- **Geographic metadata** (location, region descriptors)  
- **Semantically grounded caption**

All 10m modalities are aligned to a **128 × 128 grid**, while high-resolution imagery provides fine-grained spatial context.


## 🌐 Geographic Sourcing

GeoMeld samples are constructed using three complementary sources:

- **MMEarth-based sampling:** ensures biome-balanced global coverage  
- **SkyScript-based sampling:** introduces geographically distributed anchors with high-resolution context  
- **Custom sampling strategy:** expands coverage in underrepresented regions for improved diversity  

Only geographic coordinates are used from these sources. All multimodal data is independently retrieved under a unified alignment protocol. Users should refer to the original datasets for their respective licenses.


## ⏳ Temporal Coverage

GeoMeld captures diverse environmental conditions across time.
- Coverage spans **2017–2022**
- Seasonal variability is introduced through temporal sampling strategies
- Temporal alignment ensures consistency across modalities

## 🤖 Caption Generation

GeoMeld captions are generated using an ** agentic pipeline**:

1. **Orchestrator** — extracts structured signals (land-cover, metadata, geographic context)  
2. **Captioner** — generates candidate descriptions using a vision–language model  
3. **Evaluator** — ranks candidates via vision–text alignment  
4. **Verification** — enforces consistency with geospatial signals  

### 🔍 Cross-modal grounding signals
- **OpenStreetMap (OSM) tags** (multi-scale geographic context)  
- **Water analysis** (Dynamic World + JRC surface water consensus)  
- **Terrain analysis** (geomorphon-based classification from DEM)  

This pipeline ensures captions are **semantically meaningful and physically grounded**.


## 🌍  Dataset Structure

GeoMeld is organized as `.tar` shards, each containing `.h5` files. Each `.h5` file corresponds to a spatially aligned multi-modal sample with associated metadata.

---

### 🛰️ Modalities

| Key | NAIP Subset (`_n.tar`) | Non-NAIP Subset (`geomeld_*.tar`) | dtype | Bands |
|---|---|---|---|---|
| `naip` | `(3, 1280, 1280)` | — | `uint16` | Red, Green, Blue (1m GSD) |
| `sentinel2` | `(9, 128, 128)` | `(12, 128, 128)` | `float32` | Non-NAIP: B1–B12; NAIP: B1–B12 except B2–B4 |
| `sentinel1` | `(8, 128, 128)` | `(8, 128, 128)` | `float32` | VV_asc, VH_asc, HH_asc, HV_asc, VV_desc, VH_desc, HH_desc, HV_desc |
| `aster` | `(2, 128, 128)` | `(2, 128, 128)` | `float32` | elevation, slope |
| `canopy_height` | `(2, 128, 128)` | `(2, 128, 128)` | `float32` | canopy height, standard deviation |

---

### 🗺️ Labels and Metadata

| Key | Shape | dtype | Description |
|---|---|---|---|
| `esa_worldcover` | `(1, 128, 128)` | `uint8` | ESA WorldCover land-cover labels |
| `dynamic_world` | `(1, 128, 128)` | `uint8` | Dynamic World land-cover labels |
| `metadata` | JSON | — | geographic and contextual attributes (includes `file_type_naip`) |

---

### 📄  Metadata Fields

Each sample includes a JSON-encoded `metadata` containing geographic and contextual attributes for each tile. The field file_type_naip is assigned the value false for all samples within this subset.

```json
{
  "tile_id": 1232154454,
  "lat": 71.5545,
  "long": 71.0397,
  "acquisition_date": "2020-09-24",
  "terrain_class": "Flat",
  "file_type_naip": true,
  "osm_tags": {
    "building": "yes",
    "highway": "residential"
  },
  "water_analysis": {
    "detected": true,
    "percentage": 4.98
  }
}


