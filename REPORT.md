# Data Analysis and Bias Mitigation in AI Models

*Written report behind the [interactive data story](./index.html). This is the analysis
text only — the assignment cover sheet, student ID, and signature pages from the original
submission are not included.*

Part 2 focuses on learning if some communities' air is being "watched" for pollution less
than other communities and if that gap is correlated with the wealth or poverty of the
communities, as well as their racial diversity. Public government data on income, race,
population and location of air-quality monitors were merged together because, combined,
they enable this difference to be measured with real values rather than guessing — and
because who is exposed to clean-air data directly affects public health and fairness,
which is what the Sustainable Development Goals care about.

## 1. Data Collection and Exploration

Four public datasets were merged at the US-county level (n = 3,143 counties):

- **EPA AQS Monitors file** (`aqs_monitors.csv`) — all air-quality monitoring sites ever
  registered in the US; filtered to only show monitors that have sampled on or after
  2023-01-01, for currently active monitoring.
- **EPA AirData Annual AQI by County (2024)** — Median AQI and related summary statistics
  are only available for those counties that have adequate active monitoring.
- **US Census Bureau, SAIPE (2023)** — county-level median household income and poverty
  rate.
- **US Census Bureau Population Estimates (2023, county/age/sex/race)** — total population
  and racial/ethnic composition (% non-Hispanic White vs. % minority, % Black, % Hispanic).

The four sources were joined together based on the county FIPS code (or county + state
name, for the AQI file, since it does not publish the FIPS code). All 3,143 counties in
the United States are included in the merged dataset for income, poverty, population and
monitor presence.

The initial analysis of the combined data shows that as of 2023–2024, **1,049 of 3,143
counties (33.4%)** had an active EPA air-quality monitor. Two-thirds of the country has no
ground-truth air-quality reading, which means there is no way of measuring air quality if
a model or a map of US air quality is generated for these areas.

The next step of the analysis shows that the poorer a county is, the less likely it is to
have a sensor — the richest counties have sensors almost twice as often as the poorest.
Monitoring coverage drops from **41.7%** of counties in the lowest poverty quartile to
**21.5%** in the highest poverty quartile.

On the minority-population basis, counties with more minority residents *appear* to have
slightly more sensors, not fewer — raw coverage rises from **19.8%** in the least-diverse
quartile to **38.3%** in the most-diverse quartile. This turns out to be misleading once
actual people are counted instead of counties (see Section 2ii).

## 2. Bias Identification

This dataset contained two types of bias, and disentangling them was not simply a matter
of comparing the data by quartiles.

### i) Economic bias

Sensor coverage increases gradually as income increases, ranging from **15.1%** in the
lowest income quartile to **54.5%** in the highest quartile. A logistic regression was fit
to test whether this was just a population-size effect (larger, denser counties are more
likely to have regulatory infrastructure of all kinds, including monitors), using minority
%, poverty %, median income and log(population) as predictors.

Population size is, unsurprisingly, the single strongest predictor (odds ratio ≈ 2.64 per
tripling of population — counties with about triple the population have about triple the
odds of having a monitor). However, even once population size is controlled for, poverty
rate has an independent **negative** association with the likelihood of monitoring (odds
ratio ≈ 0.985 per percentage point of poverty), and median income has an independent
**positive** association (odds ratio ≈ 1.0074 per $1,000 of income — a roughly three-fold
difference in odds across the observed income range). Monitoring is not only going where
the people go, it is going where the money goes, even among counties of comparable size.

### ii) Racial bias (present, but only manifested at the population level)

The raw quartile comparison indicated that higher-minority counties are, if anything,
better monitored — but this is confounded by urbanicity: higher-minority counties in this
dataset are larger, more urban counties, and population size is the dominant factor in
monitor placement. The independent effect of minority percentage, once population size is
accounted for in the regression, is almost negligible (odds ratio ≈ 1.0015).

The picture is far more significant, and more disturbing, from a **population-weighted**
perspective. When people are counted instead of counties: low-diversity counties tend to
be smaller and more rural, and **62.6%** of the people living in the least-diverse quartile
of counties have no active air-quality monitor nearby, compared to only **10.2%** in the
most-diverse quartile. This is a true and serious monitoring gap — not the
urban-minority-neighbourhood gap that most environmental-justice monitoring literature
tends to focus on (which is typically analyzed at a finer geographic level than the
county).

### iii) Downstream impact on AI models

Air-quality AI systems used by the EPA, academic scientists, and health-risk websites rely
on exactly this type of monitored-county information to estimate air quality at other,
non-monitored locations. The relationships such a model learns are likely to differ in the
poorer, less populous counties it is then asked to predict for — a textbook example of
covariate shift due to non-random sample selection. Section 3 tests this directly.

## 3. Bias Mitigation

Knowing a bias exists doesn't establish that it matters, so an experiment was run to prove
the point. A Random Forest model was trained on **484 below-median-poverty counties** to
predict a county's Median AQI from population, race, poverty rate and income, then tested
on the poorer counties it had never been exposed to — mirroring the exact real-world gap
uncovered above (963 counties total had a valid Median AQI value and formed the modelling
subset; the remaining 479 above-median-poverty counties were held out entirely as the test
set).

**Mitigation 1 — Reweighting (preprocessing).** The low-poverty training counties are
weighted according to how similar they are to the missing high-poverty group, as
determined by a propensity model; counties most similar to the missing group get the
greatest weight. This is needed because a model trained on unweighted data implicitly
assumes all counties are equally important, baking the original sampling error into its
predictions.

**Mitigation 2 — Sampling (preprocessing).** The low-poverty counties closest to the
high-poverty boundary are identified and bootstrap-sampled (duplicated) into the training
set via `sklearn.utils.resample`. Reweighting can only even out the influence of examples
that already exist; oversampling instead creates additional, more concrete training
examples in the region where the model is weakest.

**Results.** The baseline model — trained with no mitigation — had a mean absolute error
of **7.73 AQI points** on the held-out poorer counties. Both mitigation strategies reduced
this to **7.57 AQI points** (≈2% improvement), and both nudged the training sample's mean
poverty rate closer to the true population value it should represent: from a raw training
mean of **9.5%** poverty toward the true population mean of **13.0%** (10.4% after
reweighting, 10.5% after oversampling).

Neither fix closed the gap entirely, for a simple reason: reweighting and oversampling can
only rearrange data that is already available — they can't manufacture a reading that was
never taken. **The durable solution is to add monitors where they're needed, not to
retroactively patch for the ones that are missing.**

## 4. Formulation of the Problem

Air pollution is a major environmental health threat, and accurate monitoring is essential
for protecting public health, enabling regulatory action, and supporting fair urban
planning. This directly contributes to **SDG 11** (Sustainable Cities and Communities) and
**SDG 13** (Climate Action) — and, given the increasing use of air-quality data in
AI-based exposure and health-risk models, to **SDG 3** (Good Health and Well-Being) and
**SDG 10** (Reduced Inequalities). But the US EPA's Air Quality System (AQS) is the sole
data source for almost all downstream air-quality models (interpolation, exposure
prediction, health-risk scoring), and all of them rest on a network of physical ground
monitors that is not uniformly distributed across the nation.

**Problem statement.** The distribution of monitors in the ground-based AQS is not random,
and this non-random placement results in a sample-selection bias. AI models developed with
AQS data are trained on the kind of air-quality data collected in areas with monitors, then
asked to generalize to areas without them. This project examines the racial, income and
poverty-related bias present in the US air-quality monitoring network, the extent to which
that bias continues to manifest in a downstream predictive AI model, and the potential for
standard bias-mitigation techniques to diminish the harm.

**Objectives achieved:**

1. Measured income, poverty rate and racial composition against monitoring coverage across
   counties of every size.
2. Statistically isolated the causes of the coverage gap while controlling for population
   size.
3. Demonstrated, with a concrete predictive model, the real-world impact of this sampling
   bias on performance for underrepresented counties.
4. Implemented and compared two bias-mitigation techniques.

Closing this monitoring and modelling gap is a prerequisite for any AI system that claims
to support equitable, SDG-aligned decisions for communities that are not well represented.

## Data sources

- EPA Air Quality System (AQS) Monitors
- EPA AirData Annual AQI by County (2024)
- U.S. Census Bureau, Small Area Income and Poverty Estimates (SAIPE, 2023)
- U.S. Census Bureau, Population Estimates (2023)

---

Written by Dhwani Sanjay Kariya as coursework for *Emerging Artificial Intelligence
Technologies & Sustainability* (H9ETS), National College of Ireland. Figures referenced
above (Figures 1–8 in the original submission) are reproduced as live, interactive charts
on the [project page](./index.html) rather than as static images here.
