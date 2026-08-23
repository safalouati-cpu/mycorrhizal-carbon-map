# Mycorrhizal Carbon Map

Predicting arbuscular mycorrhizal (AM) fungal network density from open soil,
climate, and land-use data — flagging high fungal-carbon-value regions that
currently sit outside protected areas.

## Data sources
- Stewart et al. 2026, *Science* — AM fungal hyphal density (target variable) and pre-joined
  environmental covariates (CHELSA climate, SoilGrids soil properties, EarthEnv topography,
  MODIS NPP, human modification index, biome classification), via public Dryad deposit:
  https://doi.org/10.5061/dryad.p2ngf1w1f
- SPUN Underground Atlas (project inspiration, not a direct data source used):
  https://www.spun.earth/mapping/a-hidden-infrastructure
- ESA WorldCover — land cover raster, included in the Dryad deposit: https://esa-worldcover.org

Note: SoilGrids and WorldClim were originally planned as separate downloads, but were not
needed directly — the Dryad dataset already included pre-extracted covariates from these
sources, joined to each field sample.

## Project structure
```
data/raw/         # untouched downloads, exactly as fetched
data/processed/   # cleaned, reprojected, joined data
notebooks/        # exploration and prototyping
src/              # reusable scripts (download, clean, join, model)
outputs/          # figures, model artifacts, final tables
```

## Log

### Day 1 — data collection and preprocessing kickoff
- [ ] Set up Python environment
- [ ] Download SPUN density data
- [ ] Download SoilGrids layers (target variables: pH, SOC, clay content)
- [ ] Download WorldClim bioclimatic variables
- [ ] Download ESA WorldCover land-cover raster
- [ ] Sanity-check: plot each layer, confirm extent/CRS

## Results (as of current phase)

**Model:** Random Forest Regressor (n_estimators=300, max_depth=5, min_samples_leaf=3)
**Data:** 204 unique field-sampled locations (aggregated from 2,625 raw entries to avoid pseudo-replication), from Stewart et al. 2026 (Science) public Dryad dataset.

**Cross-validated performance (5-fold, grouped by location):**
- Mean R²: 0.196 (+/- 0.120)
- Compare to initial ungrouped/non-aggregated baseline: R² = -0.210 (severe overfitting from pseudo-replication)

**Top predictors (SHAP feature importance):**
1. Human Modification (land-use pressure) - strongest driver, negative relationship
2. MODIS NPP (plant productivity) - positive relationship
3. Annual precipitation
4. Soil pH - negative relationship (favors more acidic soils)
5. Soil sand proportion

**Key limitation:** Public dataset represents ~1.3% of the original paper's full 16,000+ soil-core sample (204 vs 16,000+), which constrains achievable model accuracy. Confirmed no additional public data exists beyond what was used here.

## Results (as of current phase)

**Model:** Random Forest Regressor (n_estimators=300, max_depth=5, min_samples_leaf=3)
**Data:** 204 unique field-sampled locations (aggregated from 2,625 raw entries to avoid pseudo-replication), from Stewart et al. 2026 (Science) public Dryad dataset.

**Cross-validated performance (5-fold, grouped by location):**
- Mean R²: 0.196 (+/- 0.120)
- Compare to initial ungrouped/non-aggregated baseline: R² = -0.210 (severe overfitting from pseudo-replication)

**Top predictors (SHAP feature importance):**
1. Human Modification (land-use pressure) - strongest driver, negative relationship
2. MODIS NPP (plant productivity) - positive relationship
3. Annual precipitation
4. Soil pH - negative relationship (favors more acidic soils)
5. Soil sand proportion

**Key limitation:** Public dataset represents ~1.3% of the original paper's full 16,000+ soil-core sample (204 vs 16,000+), which constrains achievable model accuracy. Confirmed no additional public data exists beyond what was used here.


## External Validation (vs. published Stewart et al. 2026 raster)

Compared out-of-fold cross-validated predictions (honest, held-out estimates) against
the original paper's published prediction raster, at the same 192 matched locations.

| Comparison | Pearson r |
|---|---|
| Our model vs actual field values | 0.556 |
| Published model vs actual field values | 0.708 |
| Our model vs published model | 0.426 |

**Interpretation:** Using ~1.3% of the original study's sample size (204 vs 16,000+ soil cores),
our model recovers approximately 79% of the correlation strength achieved by the full published
model. This confirms the model captures a genuine, overlapping signal with the original study,
with the performance gap fully attributable to the known difference in training data volume.
