# Copilot Instructions for ERM Codebase

## Project Overview

**Extreme Rainfall Monitoring (ERM)** is a Google Earth Engine (GEE) application that detects extreme rainfall and forecasts flood risk. The codebase is a **documentation-first project** with:

- **Primary code**: Google Earth Engine JavaScript (`script/erm.js`) — a 1000+ line interactive GEE app
- **Documentation**: MkDocs-based site served from `/docs` at https://wfpidn.github.io/ERM
- **Deployment**: Automated via GitHub Actions (deploy on `main` push)
- **Data sources**: NASA IMERG (5-day historical rainfall), NOAA GFS (5-day forecast), GHSL population, MODIS cropland

## Architecture & Data Flow

The GEE script processes two parallel data streams:

1. **Near Real-Time (NRT)**: IMERG satellite precipitation → accumulation → extreme rainfall detection
2. **Forecast (FCT)**: GFS model precipitation → same analysis pipeline

**Core processing stages** (`erm.js`):
- **Global setup** (lines 1-200): Map styling, visualization palettes, static datasets (masks, thresholds)
- **Functions** (lines 200-500): Retrieve threshold/slope/intercept images; fetch IMERG/GFS data
- **Analysis** (`getRainfallExtremeImage()`): 
  - Compare rainfall against 4 percentile thresholds (P50, P80, P90, P96) → "impact" classification (0-4)
  - Logistic regression: `prob = 1/(1+e^(-(S*R+I)))` → "likelihood" classification (0-2)
  - Combine impact+likelihood → 15-class matrix → remap to final 10-class flood alert (0-9)
- **Impact Analysis** (lines 700-800): Overlay results with GHSL population & MODIS cropland grids
- **UI** (lines 850-1100): Date slider, day selector, render button; real-time legend updates
- **Export** (lines 1130+): Export all layers to Google Drive

## Critical Patterns & Conventions

### GEE Image Operations
- All datasets use `.float()` for calculations; `.updateMask()` to hide invalid pixels
- Projection consistency: IMERG data reprojected to `IMERGprojection.atScale(11131.949...)`
- Expression syntax for multi-band math: `expression('formula', {variables})` not array notation

### Threshold & Seasonal Data
- 4 thresholds stored as GEE Assets at `users/bennyistanto/datasets/...`:
  - `threshold/idn_cli_[1-5]day_precipthreshold_[q0500-q0960]_...`
  - `slope/idn_cli_day[1-5]_[01_jan-12_dec]_slope_imerg`
  - `intercept/idn_cli_day[1-5]_[01_jan-12_dec]_intercept_imerg`
- Month-based asset selection: convert JS date → month number (1-12) → asset string
- **Do not hardcode asset paths**; use helper functions `getThresholdImage()`, `getSlopeImage()`, `getInterceptImage()`

### Classification Logic
- **4-class impact**: Based on percentile thresholds (P50→1, P80→2, P90→3, P96→4)
- **3-class likelihood**: Logistic regression output binned at 0.6 and 0.8
- **Final flood alert**: 10-class (0-9) from 4×3 matrix; stored in `visFlood` palette

### UI State Management
- Global `uiComponents` object holds all widgets (map, panels, buttons, sliders)
- Event-driven: Date/day changes → `render()` function → async compute
- Loading counter (`loadingCounter`) prevents re-renders during computation
- Population/crop stats computed asynchronously; UI updates when all ready

## Development Workflow

### Building & Deploying Docs
```bash
# Install dependencies
pip install markdown mkdocs-material mkdocs-awesome-pages-plugin pymdown-extensions

# Build locally
mkdocs serve  # Runs on http://127.0.0.1:8000

# Deploy to GitHub Pages (automatic on main push via ci.yml)
mkdocs gh-deploy --force
```

### Testing GEE Code Changes
1. Open GEE Console: https://code.earthengine.google.com
2. Create new script in personal account
3. Copy `script/erm.js` content
4. Modify and test (use `print()` for debug output)
5. Verify layer rendering and export tasks
6. **Key test**: Render with different dates (recent NRT) and forecasts (GFS data from 2015+)

### Common Workflows

**To add a new output layer:**
1. Create new property in `getRainfallExtremeImage()` return object
2. Add `visXxx` palette to global variables
3. Add `mainMap.addLayer()` call in `render()` function
4. Ensure layer respects existing legend structure

**To update thresholds:**
1. Regenerate threshold assets in GEE or ArcGIS Pro
2. Upload to `users/bennyistanto/datasets/raster/extremerainfall/threshold/`
3. **Update asset naming** in `getThresholdImage()` function if pattern changes

**To modify documentation:**
1. Edit `.md` files in `docs/`
2. Update `mkdocs.yml` navigation if adding new sections
3. Commit to `main` → CI auto-deploys

## File Structure Reference

```
docs/
  index.md                 # Home & overview
  data.md                  # Input data sources (IMERG, GFS, GHSL, MODIS)
  methodology/
    eit.md                 # Extreme rainfall threshold methodology
    rof.md                 # Rainfall-triggered flooding
    impact.md              # Population & crop impact analysis
    eaq.md                 # Alert quality evaluation
  how-to/
    imerg.md               # How to acquire IMERG data
    gefs.md                # How to acquire GEFS/GFS data
    erc.md                 # Extreme rainfall calculation process
  implementation/
    gee.md                 # GEE implementation (includes code examples)
    example.md             # Example outputs (Cyclone Seroja, April 2021)
  mkdocs.yml              # Nav structure & theme config

script/
  erm.js                   # Main GEE app (~1100 lines)

.github/workflows/
  ci.yml                   # Auto-deploy docs on main push
```

## External Dependencies & Data Access

- **Google Earth Engine**: Requires GEE account & access to public datasets + custom assets (`users/bennyistanto/*`)
- **NASA IMERG**: Available in GEE catalog as `NASA/GPM_L3/IMERG_V06` (30-min resolution, starts June 2000)
- **NOAA GFS**: Available in GEE catalog as `NOAA/GFS0P25` (forecast cycles, 6-hourly)
- **GHSL & MODIS**: Public GEE datasets
- **Custom Assets**: Hosted in personal GEE account under `users/bennyistanto/`; needed for thresholds/slopes/intercepts

## Key Gotchas & Decisions

1. **Scale consistency**: IMERG operates at ~11km; GHSL at 250m; MODIS at 500m. Always `.reproject()` to consistent scale before overlay
2. **No async/await in GEE JS**: Use `.evaluate()` callbacks for asynchronous computations; prevents UI freezing but requires careful state tracking
3. **5-day windows**: Data aggregation spans 5 days historical (NRT) or 5 days forecast; month-based seasonal parameters must handle month boundaries
4. **Percentile thresholds**: Represent 2yr, 5yr, 10yr, 25yr return periods; reflect local climate variability, not global extremes
5. **Remapping**: Alert classification uses two remap operations (15→10-class, then 15→3-class); verify both outputs for downstream uses

## Useful Resources

- GEE documentation: https://developers.google.com/earth-engine/guides
- ERM live demo: https://bennyistanto.users.earthengine.app/view/wfpid-erm
- Full project docs: https://wfpidn.github.io/ERM
- GitHub repo: https://github.com/wfpidn/ERM
