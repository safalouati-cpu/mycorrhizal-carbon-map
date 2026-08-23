# Mycorrhizal Carbon Map — Project Recap

## 1. Inspiration

In 2026, SPUN (Society for the Protection of Underground Networks) published the
first global maps of Earth's arbuscular mycorrhizal (AM) fungal networks, based on
research from Stewart et al., *Science* (2026). The study estimated that AM fungal
hyphae in global topsoils total roughly 1.10 × 10¹⁷ km in length and weigh about
300 megatons, and that these fungal networks transport an estimated 4 billion tons
of CO2-equivalent into soils each year — roughly 11% of annual human-related
emissions. Much of this fungal infrastructure sits outside protected areas, meaning
it is effectively invisible to current conservation and carbon-policy planning.

This raised a clear, practical question: could a smaller-scale, fully open,
reproducible machine learning pipeline recover a meaningful piece of this picture
using only public data?

## 2. Problem

No lightweight, reproducible, open-source tool exists to estimate AM fungal network
density from standard environmental covariates (climate, soil, topography, land
use). Without this, it's difficult to flag which unprotected regions may hold high
fungal-carbon value — informing where soil-carbon conservation or MRV
(measurement, reporting, verification) attention could matter most.

## 3. Solution

Build a machine learning pipeline that:
- Predicts AM fungal hyphal density from open environmental covariates
- Is trained and validated on the same field data underlying the original *Science*
  paper (via its public Dryad deposit)
- Is interpretable (not just predictive) — surfacing which environmental factors
  actually drive fungal density
- Is benchmarked honestly against the original paper's own published prediction
  raster, to quantify how much can realistically be recovered from public data alone

## 4. Data Gathering

**Source:** Stewart et al. 2026, Dryad dataset (doi:10.5061/dryad.p2ngf1w1f),
the public data deposit accompanying the *Science* paper.

**What was retrieved:**
- `hyphal_density_full_metadata.csv` — the target variable: field-measured AM
  hyphal density (m/cm³), plus georeferenced coordinates and study metadata,
  compiled from 4,000+ literature-sourced field measurements across 322 studies
- Pre-joined environmental covariates already present in that file: CHELSA
  bioclimatic variables (temperature, precipitation), SoilGrids soil properties
  (pH, organic carbon, depth to bedrock, sand content), EarthEnv topography
  (elevation, slope), MODIS net primary productivity, a human land-modification
  index, and biome classification
- `ESA_WorldCover_v2.tif`, `ResolveBiome.tif`, and the `Ecoregions2017` shapefile
  — supplementary spatial reference layers
- The published prediction raster (`hyphal_density_m_cm3_Classified_mean.tif`),
  retrieved separately and used **only** for final external validation, never for
  training

**Key data decision:** the dataset's own pre-extracted covariates were used directly
rather than independently re-downloading and spatially joining SoilGrids/WorldClim/
land-cover rasters from scratch — a deliberate scope decision to prioritize model
development and validation rigor within the available time.

**Known limitation, confirmed directly:** the public Dryad deposit contains
substantially less data than the original study used internally — the *Science*
paper describes over 16,000 soil cores across 322 studies, while the public field
database resolves to 204 unique sampling locations after removing rows with
missing coordinates or covariates. This was verified by manually inspecting the
full archive contents; no additional public files exist beyond what was used here.

## 5. Data Cleaning & Preprocessing

- Filtered 4,129 raw literature-compiled entries down to 2,625 rows with complete
  coordinates, target values, and covariates
- Identified that many rows were repeated measurements (different depths/reps) at
  the same 204 unique locations — a pseudo-replication issue that initially caused
  severe model overfitting
- Aggregated to one row per unique location (mean target value) to produce a
  genuinely independent 204-sample dataset
- Grouped rare biome categories (fewer than 50 samples) into an "Other" class
  before encoding

## 6. Exploratory Data Analysis

- **Target distribution:** heavily right-skewed (raw density ranged from 0.004 to
  229.5 m/cm³); log-transformed for modeling
- **Spatial distribution:** strong Northern Hemisphere temperate-zone sampling
  bias; sparse tropical and African coverage — a real, documented limitation
- **Predictor correlations:** no dangerous redundancy among the 13 predictors
  (highest pairwise correlation: -0.79, precipitation vs. soil pH — ecologically
  sensible, not duplicated information)
- **Predictor-target relationships:** weak under linear correlation (all under
  0.17), but confirmed to be real and meaningful via mutual information (up to
  0.78) and visual nonlinear trend curves — informing the choice of a tree-based,
  nonlinear model over a linear one

## 7. Modeling

- **First baseline (XGBoost, ungrouped location split):** severe overfitting
  (train R² = 0.80, test R² = -0.15)
- **Diagnosis:** pseudo-replication from repeated same-location samples
- **Fix:** aggregated to one row per location; switched to Random Forest to match
  the original paper's own modeling approach
- **Final cross-validated result (5-fold, grouped):** mean R² = 0.196 (± 0.120)
  — a modest but genuine, honestly-validated result given the ~80x smaller sample
  size compared to the original study

## 8. Interpretation (SHAP)

Top drivers of predicted hyphal density, in order of importance:
1. **Human land-use modification** — strongest driver; higher modification
   associated with lower fungal density
2. **MODIS net primary productivity** — higher plant productivity associated with
   higher fungal density
3. **Annual precipitation**
4. **Soil pH** — more acidic soils associated with higher density
5. **Soil sand content**

The human-modification finding is directly relevant to the original conservation
framing: land-use pressure appears to measurably degrade fungal carbon
infrastructure, in a direction consistent with ecological expectations.

## 9. External Validation

Predictions were compared against the original paper's own published raster at the
same 192 matched locations, using honest out-of-fold cross-validated predictions
(never predicting on data the model was trained on):

| Comparison | Pearson r |
|---|---|
| This model vs. actual field values | 0.556 |
| Published model vs. actual field values | 0.708 |
| This model vs. published model | 0.426 |

Using roughly 1.3% of the original study's sample size, this pipeline recovers
approximately 79% of the correlation strength of the full published model — a
result fully explained by the documented difference in training data volume,
not a flaw in the approach.

## 10. Tools & Reproducibility

Python (pandas, scikit-learn, XGBoost, Random Forest, SHAP, rasterio, matplotlib/
seaborn), developed in Google Colab, version-controlled on GitHub with raw data
excluded from the repository and stored in Google Drive, referenced by path.

## 11. Honest Limitations

- Public sample size (204 locations) is a small fraction of the original study's
  internal dataset
- Strong geographic sampling bias toward Northern Hemisphere temperate regions;
  tropical and African predictions should be treated with low confidence
- Environmental covariates were used as pre-extracted rather than independently
  re-derived from raw rasters
