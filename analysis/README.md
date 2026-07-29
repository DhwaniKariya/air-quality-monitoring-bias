# Analysis code

This is the actual Python behind every number on the [project page](../index.html) and in
[`ANALYSIS.md`](../ANALYSIS.md).

- **`analysis.py`**: the full pipeline, downloads the four raw government datasets,
  merges them into one county-level table, runs the logistic regression and the
  Random Forest bias experiment, and saves every chart and number it produces.
- **`analysis.ipynb`**: the same code as a notebook, with the actual output cells (the
  print statements and inline charts) from the run that produced the numbers on the page.
- **`data/county_master.csv`**: the merged, cleaned county-level dataset (income,
  poverty, race, population, monitor counts). This is the one processed file I'm
  including directly, since it's small and saves you from re-downloading everything.
- **`figs/`**: the matplotlib charts the script generates, plus `results.json` and
  `summary_stats.csv` with the exact numbers used throughout the write-up.

## Running it yourself

```bash
pip install -r requirements.txt
python analysis.py
```

The raw government source files (SAIPE income/poverty, Census population, EPA monitors,
EPA AQI) aren't checked into this repo, since together they're close to 80 MB and are all
public downloads anyway. The script fetches them itself on first run and caches them in
`data/`, so you don't need to do anything by hand.

## What it actually does, briefly

1. Downloads and merges SAIPE income/poverty, Census population/race, and EPA AQS monitor
   data at the county level (FIPS code).
2. Separately joins in EPA's Median AQI by county (matched on state and cleaned county
   name, since that file doesn't use FIPS codes).
3. Computes monitor coverage by poverty, income, and racial composition quartile, both by
   county count and weighted by population.
4. Fits a logistic regression to check whether poverty and race still predict monitor
   presence once you control for population size.
5. Trains a Random Forest on low-poverty counties only, tests it on high-poverty counties,
   and compares that baseline against two bias-mitigation techniques (inverse-propensity
   reweighting and boundary oversampling).
