# Cross-Scale Rice Disease Dataset (UAV + Satellite)

A paired, synthetic multispectral feature dataset for **cross-scale plant disease
identification** — combining drone (UAV) and satellite observations of rice
affected by Bacterial Leaf Blight (BLB).

> ⚠️ **This dataset is synthetically generated, not collected from real sensors.**
> Please read [Provenance & Limitations](#provenance--limitations) before use.

---

## Contents

| File | Rows | Cols | Description |
|---|---|---|---|
| `uav_features_1000.csv` | 1,000 | 12 | UAV-only multispectral features |
| `satellite_features_1000.csv` | 1,000 | 13 | Satellite-only multispectral features |
| `fused_features_1000.csv` | 1,000 | 22 | Both sources joined on `sample_id` |

All three files describe the **same 1,000 samples**. Each `sample_id` (1–1000)
refers to one observation unit measured at two spatial scales, so the UAV and
satellite rows for a given ID are genuinely paired — not independent records.

---

## Class labels

Four severity classes, identical across all three files:

| Class | `class_numeric` | Count | Share |
|---|---|---|---|
| `Healthy` | 0 | 250 | 25.0% |
| `Low_severity` | 1 | 210 | 21.0% |
| `High_severity` | 2 | 270 | 27.0% |
| `Others` | 3 | 270 | 27.0% |

`Others` is a heterogeneous catch-all for canopy conditions that are neither
clearly healthy nor clearly BLB-affected (other stresses, mixed pixels,
non-crop cover). It is intentionally the widest-variance class.

---

## Schema

### `uav_features_1000.csv` (12 columns)

| Column | Type | Description |
|---|---|---|
| `sample_id` | int | Unique ID, 1–1000. Join key across all files. |
| `uav_Blue` | float | UAV blue-band reflectance (0–1) |
| `uav_Green` | float | UAV green-band reflectance |
| `uav_Red` | float | UAV red-band reflectance |
| `uav_Rededge` | float | UAV red-edge band reflectance |
| `uav_NIR` | float | UAV near-infrared reflectance |
| `NDVI` | float | (NIR − Red) / (NIR + Red) |
| `NDRE` | float | (NIR − RedEdge) / (NIR + RedEdge) |
| `GNDVI` | float | (NIR − Green) / (NIR + Green) |
| `SAVI` | float | ((NIR − Red) / (NIR + Red + 0.5)) × 1.5 |
| `class` | str | Severity label |
| `class_numeric` | int | Encoded label (0–3) |

Comparable to a 5-band UAV sensor such as MicaSense RedEdge.

### `satellite_features_1000.csv` (13 columns)

| Column | Type | Description |
|---|---|---|
| `sample_id` | int | Unique ID, 1–1000 |
| `sat_Blue` … `sat_NIR` | float | Five bands matching the UAV set |
| `sat_SWIR1` | float | Shortwave infrared 1 — canopy water content |
| `sat_SWIR2` | float | Shortwave infrared 2 |
| `sat_NDVI` | float | Satellite NDVI |
| `sat_NDRE` | float | Satellite NDRE |
| `sat_GNDVI` | float | Satellite GNDVI |
| `class` | str | Severity label |
| `class_numeric` | int | Encoded label (0–3) |

Broadly modelled on Sentinel-2 band availability. The two SWIR bands have no
UAV equivalent — they are the satellite's unique contribution to the fusion
feature space, and are a large part of why fusion can beat UAV alone.

### `fused_features_1000.csv` (22 columns)

Inner join of the two files on `sample_id`. Column order:

```
sample_id
uav_Blue, uav_Green, uav_Red, uav_Rededge, uav_NIR,
uav_NDVI, uav_NDRE, uav_GNDVI, uav_SAVI
sat_Blue, sat_Green, sat_Red, sat_Rededge, sat_NIR, sat_SWIR1, sat_SWIR2,
sat_NDVI, sat_NDRE, sat_GNDVI
class, class_numeric
```

Two notes on how the merge was performed:

1. **UAV indices were renamed** from `NDVI`/`NDRE`/`GNDVI`/`SAVI` to
   `uav_NDVI`/`uav_NDRE`/`uav_GNDVI`/`uav_SAVI`, so they are unambiguous
   alongside their `sat_` counterparts. The standalone UAV file keeps the
   original bare names for backward compatibility.
2. **The duplicated label columns were dropped** from the satellite side. Class
   labels were verified identical across both files for all 1,000 rows
   (100% agreement) before dropping.

Verified: 1:1 join on all 1,000 rows, no duplicate IDs, **zero missing values**
in any of the three files, and all merged values match their source files
exactly.

---

## Descriptive statistics

### UAV

| | Blue | Green | Red | RedEdge | NIR | NDVI | NDRE | GNDVI | SAVI |
|---|---|---|---|---|---|---|---|---|---|
| Mean | 0.064 | 0.120 | 0.122 | 0.232 | 0.368 | 0.489 | 0.223 | 0.496 | 0.367 |
| Std | 0.018 | 0.022 | 0.041 | 0.041 | 0.073 | 0.199 | 0.059 | 0.134 | 0.157 |
| Min | 0.012 | 0.044 | 0.025 | 0.119 | 0.155 | −0.073 | −0.024 | 0.075 | −0.044 |
| Max | 0.120 | 0.182 | 0.228 | 0.343 | 0.536 | 0.896 | 0.409 | 0.827 | 0.680 |

### Satellite

| | Blue | Green | Red | RedEdge | NIR | SWIR1 | SWIR2 | NDVI | NDRE | GNDVI |
|---|---|---|---|---|---|---|---|---|---|---|
| Mean | 0.077 | 0.119 | 0.096 | 0.235 | 0.358 | 0.203 | 0.144 | 0.578 | 0.207 | 0.500 |
| Std | 0.023 | 0.026 | 0.031 | 0.033 | 0.047 | 0.040 | 0.035 | 0.127 | 0.085 | 0.105 |
| Min | 0.008 | 0.038 | 0.006 | 0.124 | 0.202 | 0.089 | 0.033 | 0.157 | −0.095 | 0.143 |
| Max | 0.152 | 0.235 | 0.183 | 0.339 | 0.513 | 0.309 | 0.251 | 0.964 | 0.500 | 0.789 |

### Class separation (mean vegetation index by class)

**UAV** — clear, monotonic severity gradient:

| Class | NDVI | NDRE | GNDVI | SAVI |
|---|---|---|---|---|
| Healthy | 0.692 | 0.238 | 0.622 | 0.531 |
| Low_severity | 0.544 | 0.226 | 0.536 | 0.408 |
| Others | 0.464 | 0.228 | 0.480 | 0.345 |
| High_severity | 0.285 | 0.203 | 0.363 | 0.206 |

**Satellite** — same ordering, but compressed:

| Class | NDVI | NDRE | GNDVI |
|---|---|---|---|
| Healthy | 0.658 | 0.209 | 0.550 |
| Low_severity | 0.598 | 0.210 | 0.519 |
| Others | 0.574 | 0.210 | 0.495 |
| High_severity | 0.493 | 0.199 | 0.446 |

The UAV NDVI spread across classes is ~0.41; the satellite spread is only ~0.17.
This is deliberate — see below.

---

## How the data was generated

Each sample was assigned a latent **health index** in [0, 1] (1 = fully healthy
canopy, 0 = maximum BLB severity), drawn from a class-conditional Beta
distribution:

| Class | Distribution |
|---|---|
| Healthy | Beta(8, 2) |
| Low_severity | Beta(5, 4) |
| High_severity | Beta(2, 6) |
| Others | Beta(2, 2) |

Every band, **for both sensors**, was then generated as a function of that same
latent value — which is what makes the UAV and satellite rows genuinely paired
rather than independently random. Band reflectance was interpolated between
literature-informed "stressed" and "healthy" reference values, plus Gaussian
sensor noise, clipped to [0, 1]. Vegetation indices were then computed *from the
generated bands* using the standard formulas above, so band-index relationships
are internally consistent, as they would be in real data.

Two modelling choices are worth stating explicitly:

1. **Disease stress lowers NIR and red-edge reflectance and raises red
   reflectance** (chlorophyll loss and canopy structural degradation), which
   drives NDVI/NDRE/GNDVI down as severity rises. This follows published
   vegetation-stress spectral behaviour rather than arbitrary class offsets.
2. **Satellite features carry more noise and a compressed dynamic range**
   (compression factor 0.75) relative to UAV. This deliberately models the
   fine-detail-vs-broad-coverage trade-off at the heart of cross-scale fusion:
   coarse-resolution mixed pixels blur true field-level extremes.

Paired UAV/satellite NDVI correlation: **r = 0.56**.

---

## Baseline results

5-fold stratified cross-validation, weighted F1, on `fused_features_1000.csv`.
Reproducible with the snippet in [Quick start](#quick-start).

### Four-class classification

| Model | UAV-only | Satellite-only | Fusion |
|---|---|---|---|
| KNN (k=7) | 0.500 | 0.389 | 0.499 |
| Random Forest | 0.514 | 0.389 | **0.538** |
| SVM-RBF | **0.524** | 0.381 | 0.489 |

### Early detection (Healthy vs Low_severity only)

| Model | UAV-only | Satellite-only | Fusion |
|---|---|---|---|
| KNN (k=7) | 0.737 | 0.634 | **0.741** |
| Random Forest | 0.747 | 0.655 | **0.747** |
| SVM-RBF | **0.747** | 0.640 | 0.688 |

Reading these honestly: satellite-only is consistently the weakest, as designed.
Fusion gives a modest gain **for Random Forest and KNN**, but does *not* beat
UAV-only for SVM-RBF — adding 10 noisier correlated features can hurt a
margin-based model. Absolute scores in the four-class task (~0.5) are moderate
because `Others` overlaps heavily with the severity classes by construction.
These are baselines to improve on, not a demonstration that fusion always wins.

---

## Quick start

```python
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score, StratifiedKFold

df = pd.read_csv("fused_features_1000.csv")

uav_cols = [c for c in df.columns if c.startswith("uav_")]
sat_cols = [c for c in df.columns if c.startswith("sat_")]
y = df["class"]

cv = StratifiedKFold(5, shuffle=True, random_state=42)
rf = RandomForestClassifier(n_estimators=300, random_state=42, n_jobs=-1)

for name, cols in [("UAV-only", uav_cols),
                   ("Satellite-only", sat_cols),
                   ("Fusion", uav_cols + sat_cols)]:
    score = cross_val_score(rf, df[cols], y, cv=cv, scoring="f1_weighted").mean()
    print(f"{name:16s} weighted F1: {score:.3f}")
```

Rebuild the merged file from the two standalone files:

```python
uav = pd.read_csv("uav_features_1000.csv").rename(columns={
    "NDVI": "uav_NDVI", "NDRE": "uav_NDRE",
    "GNDVI": "uav_GNDVI", "SAVI": "uav_SAVI"})
sat = pd.read_csv("satellite_features_1000.csv").drop(columns=["class", "class_numeric"])
fused = uav.merge(sat, on="sample_id", validate="one_to_one")
```

---

## Suggested uses

- Benchmarking UAV-only vs satellite-only vs feature-level fusion
- Early disease detection (binary `Healthy` vs `Low_severity`)
- Feature-importance and dimensionality-reduction studies across sensor scales
- Teaching material for multimodal / multi-sensor fusion
- End-to-end pipeline development before real imagery is available

---

## Provenance & Limitations

**This is synthetic data.** It was not collected from real drones or satellites,
and no real field observations are included.

It was generated after an earlier 100-sample version was found to contain **no
real class-conditional signal** — per-class NDVI means were statistically
indistinguishable and paired UAV/satellite correlation was only r ≈ 0.20,
consistent with independent random draws. This 1,000-sample version was built to
restore realistic, literature-grounded structure.

What that means in practice:

- ✅ **Appropriate for**: pipeline development, method prototyping, hyperparameter
  search, teaching, reproducible demonstrations, code testing.
- ❌ **Not appropriate for**: reporting empirical findings about real rice
  disease, validating a model for field deployment, or any claim about real-world
  accuracy.

If you use this in academic work, state clearly that the data is simulated.
Conclusions drawn here describe the behaviour of the *generator*, not of rice
canopies. Replace these files with features extracted from real imagery before
reporting substantive results — the schema is designed so that swap requires no
downstream code changes.

Other caveats:

- `Others` is deliberately heterogeneous (Beta(2,2)) and overlaps the severity
  classes, capping achievable four-class accuracy.
- Band values are drawn independently given the latent health index, so
  real-world inter-band correlation structure is only partially reproduced.
- No spatial, temporal, phenological, cultivar, or atmospheric variation is
  modelled. Every sample is one point observation with no location or date.
- Class proportions were inherited from the original seed file and are not
  calibrated to real BLB prevalence.

---

## Reproducibility

Generated with a fixed seed (`RANDOM_STATE = 42`), so the files are exactly
reproducible from the generator script.

## Citation

```bibtex
@misc{crossscale_rice_disease_dataset,
  title  = {Cross-Scale Rice Disease Dataset (UAV + Satellite)},
  author = {Obadein, Tofunmi},
  year   = {2026},
  note   = {Synthetic paired UAV--satellite multispectral feature dataset
            for cross-scale plant disease identification},
  howpublished = {\url{https://github.com/<your-username>/<your-repo>}}
}
```

## License

Suggested: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — permissive
and standard for open datasets. Add a `LICENSE` file to your repo to make it
official.
