# Mycorrhizal Carbon Map

Predicting arbuscular mycorrhizal (AM) fungal network density from open soil,
climate, and land-use data — flagging high fungal-carbon-value regions that
currently sit outside protected areas.

## Data sources
- SPUN — AM fungal network density (target variable): https://www.spun.earth/mapping/a-hidden-infrastructure
- SoilGrids — soil properties (pH, carbon, texture): https://soilgrids.org
- WorldClim — bioclimatic variables: https://www.worldclim.org
- ESA WorldCover — land-use/land-cover classes: https://esa-worldcover.org

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
