# Chicago Crime Forecasting & Tail-Risk Analysis

A grid-based, time-aware forecasting model for daily crime counts across
Chicago, paired with a statistical method for flagging locations where the
model's errors show heavy-tailed ("tail risk") behavior — i.e. places the
model is systematically surprised by extreme spikes.

## What this project does

1. **Loads and cleans** the Chicago crime dataset, filtered to 2020 onward.
2. **Builds a spatial grid** (~500m cells) and aggregates incidents into a
   complete *date x grid* time series, so that zero-crime days/locations are
   explicitly represented.
3. **Engineers features**: 1-day lag, 7-day rolling average, calendar
   features (month, day of week), and a US holiday flag.
4. **Trains an XGBoost regressor** on a chronological (time-aware) train/validation
   split to forecast daily crime counts per grid cell.
5. **Analyzes residuals per grid cell** using kurtosis to classify each cell
   as High Tail Risk or Low Risk — surfacing locations with unpredictable,
   spiky crime activity rather than just high average crime.
6. **Visualizes** overall trends, seasonality, top predicted-crime hotspots,
   and the risk classification.

## Project structure

```
chicago-crime-forecasting/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
└── notebooks/
    └── crime_forecasting_analysis.ipynb
```

## Getting started

### 1. Clone and install dependencies

```bash
git clone <this-repo-url>
cd chicago-crime-forecasting
python -m venv .venv && source .venv/bin/activate   # optional but recommended
pip install -r requirements.txt
```

### 2. Get the data

The raw dataset is too large to check into the repo, so it isn't included.
You have two options, both configured in the notebook's data-loading cell:

- **Download automatically** (default): the notebook uses `gdown` to pull a
  pre-exported CSV from Google Drive. This is convenient but depends on that
  file remaining shared/available — treat it as a demo convenience, not a
  stable data pipeline.
- **Use your own copy**: download the source data directly from the
  [Chicago Data Portal – Crimes dataset](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2),
  save it as `data/crime_data.csv`, and set `DOWNLOAD_DATA = False` in the
  notebook.

### 3. Run the notebook

```bash
jupyter notebook notebooks/crime_forecasting_analysis.ipynb
```

Run all cells top to bottom. Expect the full data load + feature engineering
step to take a while on the full dataset — filtering to a shorter date range
or a smaller geographic bounding box will speed up iteration.

## Method notes

- **Grid resolution** is ~0.005 degrees (~500m) per cell, configurable via
  `GRID_RESOLUTION`.
- **Train/validation split** is chronological, not random: the most recent
  10% of dates are held out, which better reflects real forecasting
  conditions and avoids leaking future information into training.
- **Tail-risk threshold**: grid cells with residual (excess) kurtosis above
  `KURTOSIS_THRESHOLD = 2.5` are flagged as High Tail Risk. This threshold is
  a starting point, not a statistically derived cutoff — adjust it for your
  use case.

## Known limitations / future work

- **Weather features are placeholders.** `Avg_Temp` and `Precip_mm` are
  referenced in the feature list but no weather data source is wired up yet,
  so they're currently filled with `0` and contribute no signal. Integrating
  a real weather dataset (e.g. NOAA) keyed on date is a natural next step.
- **Fixed grid resolution.** A uniform grid is simple but not
  crime-density-aware; a quadtree or H3-based grid could balance cell sizes
  better.
- **Single global model.** One XGBoost model is fit across all grid cells;
  per-region or hierarchical models could improve accuracy in sparse areas.
- **`shap` is imported but not yet used** for feature-importance /
  explainability analysis — it's included for aspiring extensions to the
  notebook (e.g. explaining *why* a given grid is forecast to be high-risk).

## License

This project is provided under the [MIT License](LICENSE). The Chicago
crime data itself is subject to the City of Chicago's own data terms — see
the [Chicago Data Portal](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2)
for details.
