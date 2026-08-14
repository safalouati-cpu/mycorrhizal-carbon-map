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
