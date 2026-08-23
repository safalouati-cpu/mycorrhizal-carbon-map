# Mycorrhizal Carbon Map

Predicting arbuscular mycorrhizal (AM) fungal network density from open
environmental data, replicating and validating against Stewart et al. 2026
(Science) - inspired by SPUN's Underground Atlas mapping project.

For the full narrative (inspiration, problem, methodology, results, limitations),
see PROJECT_SUMMARY.md

## Data sources

- Stewart et al. 2026, Science - AM fungal hyphal density (target variable) and
  pre-joined environmental covariates (CHELSA climate, SoilGrids soil properties,
  EarthEnv topography, MODIS NPP, human modification index, biome classification),
  via public Dryad deposit: https://doi.org/10.5061/dryad.p2ngf1w1f
- SPUN Underground Atlas (project inspiration): https://www.spun.earth/mapping/a-hidden-infrastructure
- ESA WorldCover - land cover raster, included in the Dryad deposit: https://esa-worldcover.org

## Project structure

data/raw/         # untouched downloads (gitignored, stored in Google Drive)
data/processed/   # cleaned, model-ready dataset
notebooks/        # exploration and modeling notebook
src/              # reusable scripts
outputs/          # trained model, SHAP plots

## Results summary

Model: Random Forest Regressor (n_estimators=300, max_depth=5, min_samples_leaf=3)
Data: 204 unique field-sampled locations (aggregated from 2,625 raw entries to
avoid pseudo-replication)

Cross-validated performance (5-fold, grouped by location):
- Mean R2: 0.196 (+/- 0.120)
- Initial ungrouped baseline (before fixing pseudo-replication): R2 = -0.210

Top predictors (SHAP): human land-use modification (strongest, negative
relationship), plant productivity (MODIS NPP, positive), annual precipitation,
soil pH (negative), soil sand content.

External validation (out-of-fold predictions vs. the original paper's published
prediction raster, 192 matched locations):

| Comparison | Pearson r |
|---|---|
| This model vs actual field values | 0.556 |
| Published model vs actual field values | 0.708 |
| This model vs published model | 0.426 |

Using ~1.3% of the original study's sample size, this model recovers approximately
79% of the correlation strength of the full published model - a gap fully
attributable to the known difference in training data volume.

Key limitation: the public Dryad dataset resolves to 204 unique locations,
versus the 16,000+ soil cores described in the original paper. Confirmed directly
that no additional public data exists beyond what was used here.

See PROJECT_SUMMARY.md for full methodology, EDA findings, and discussion.