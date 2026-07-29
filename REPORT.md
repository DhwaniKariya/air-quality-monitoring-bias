# Data Analysis and Bias Mitigation in AI Models

*This is the written report behind the [interactive data story](./index.html).*

Part 2 is about figuring out whether some communities' air is being watched for pollution
less than others, and whether that gap lines up with how wealthy or poor a community is,
or how racially diverse it is. I merged public government data on income, race,
population, and the location of air-quality monitors, mainly because putting them
together let me actually measure the difference instead of guessing at it, and because
who gets clean air data directly affects public health and fairness, which is what the
Sustainable Development Goals are about.

## 1. Data Collection and Exploration

I merged four public datasets at the US county level (n = 3,143 counties):

- **EPA AQS Monitors file** (`aqs_monitors.csv`), which lists every air-quality monitoring
  site ever registered in the US. I filtered it down to monitors that had sampled on or
  after 2023-01-01, so it reflects currently active monitoring.
- **EPA AirData Annual AQI by County (2024)**. Median AQI and the related summary stats
  are only available for counties that have adequate active monitoring.
- **US Census Bureau, SAIPE (2023)**, for county-level median household income and
  poverty rate.
- **US Census Bureau Population Estimates (2023, county/age/sex/race)**, for total
  population and racial and ethnic composition (percent non-Hispanic White versus percent
  minority, percent Black, percent Hispanic).

I joined the four sources on the county FIPS code, or on county plus state name for the
AQI file, since that one doesn't publish FIPS codes. All 3,143 US counties ended up in the
merged dataset for income, poverty, population, and monitor presence.

The first thing that jumped out: as of 2023 to 2024, only **1,049 of 3,143 counties
(33.4%)** had an active EPA air-quality monitor. Two thirds of the country has no
ground-truth air-quality reading at all, which means there's no real way to know the air
quality there if a model or a map claims to describe it.

Next, I looked at poverty. The poorer a county is, the less likely it is to have a sensor.
Coverage drops from **41.7%** of counties in the lowest poverty quartile down to **21.5%**
in the highest poverty quartile, roughly a two times gap between the richest and poorest
counties.

On the racial composition side, the raw numbers actually look backwards at first: coverage
rises from **19.8%** in the least diverse quartile up to **38.3%** in the most diverse
quartile, meaning more diverse counties appear better monitored. That turned out to be
misleading once I weighted by population instead of just counting counties (more on that
below).

## 2. Bias Identification

There were two different kinds of bias hiding in this dataset, and untangling them took
more than just comparing quartiles.

### i) Economic bias

Sensor coverage climbs steadily with income, from **15.1%** in the lowest income quartile
up to **54.5%** in the highest. The obvious question is whether this is just a population
size effect, since bigger and denser counties tend to have more regulatory infrastructure
of every kind, monitors included. I fit a logistic regression using minority percent,
poverty percent, median income, and log(population) as predictors to check.

Population size turned out to be the strongest predictor by far, which isn't surprising
(odds ratio of about 2.64 per tripling of population, so a county with roughly three times
the population has roughly three times the odds of having a monitor). But even after
controlling for population size, poverty rate still has an independent negative effect on
the odds of getting a monitor (odds ratio around 0.985 per percentage point of poverty),
and median income still has an independent positive effect (odds ratio around 1.0074 per
$1,000 of income, which works out to roughly a threefold difference in odds across the
income range in this data). So monitoring isn't just going where the people are, it's also
going where the money is, even among similarly sized counties.

### ii) Racial bias, but only visible once you weight by population

The raw quartile comparison made it look like higher-minority counties were actually
better monitored, but that's confounded by urbanicity: the higher-minority counties in
this dataset also tend to be larger and more urban, and population size is the single
biggest factor in where monitors get placed. Once I control for population size in the
regression, the independent effect of minority percentage is almost nothing (odds ratio
around 1.0015).

The picture changes a lot once you weight by population instead of by county. Low-diversity
counties tend to be smaller and more rural, so a small number of monitored counties can
hide a huge number of unmonitored people. **62.6%** of people living in the least diverse
quartile of counties have no active monitor nearby, compared to only **10.2%** in the most
diverse quartile. That's a real and pretty serious monitoring gap, and it's not the kind of
urban-minority-neighbourhood gap that most environmental justice research usually focuses
on, which is typically looked at with much finer geographic resolution than a whole county.

### iii) What this means downstream, for AI models

Air-quality AI systems, whether that's EPA's own tools, academic models, or health-risk
websites, rely on exactly this kind of monitored-county data to estimate air quality
somewhere else, at a county with no monitor. Whatever relationship a model learns from
monitored counties is probably not the same relationship that holds in the poorer, less
populous counties it then has to predict for. That's covariate shift caused by a sample
that isn't random. I tested this directly in the next section.

## 3. Bias Mitigation

Knowing there's a bias in the data doesn't tell you whether it actually matters, so I ran
an experiment to check. I trained a Random Forest on **484 below-median-poverty counties**
to predict a county's Median AQI from population, race, poverty rate, and income, then
tested it on the poorer counties it had never seen during training. That mirrors, on a
small scale, exactly what a national air-quality model does every day (963 counties in
total had a valid Median AQI value and made up the modelling subset, with the remaining
479 above-median-poverty counties held out entirely as the test set).

**Mitigation 1, reweighting.** I reweighted the low-poverty training counties based on how
similar they are to the missing high-poverty group, using a propensity model, so counties
most like the missing group get the most weight. This matters because a model trained on
unweighted data implicitly treats every county as equally important, which just bakes the
original sampling error straight into its predictions.

**Mitigation 2, oversampling.** I found the low-poverty counties closest to the
high-poverty boundary and bootstrap-sampled them into the training set using
`sklearn.utils.resample`. The idea is that reweighting can only redistribute the influence
of examples that already exist, while oversampling actually adds more concrete training
examples in the region where the model is weakest.

**What happened.** The baseline model, no mitigation at all, had a mean absolute error of
**7.73 AQI points** on the held-out poorer counties. Both mitigations brought that down to
**7.57 AQI points**, about a 2% improvement, and both nudged the training sample's average
poverty rate closer to the true population value: the raw training sample averaged **9.5%**
poverty, versus a true population mean of **13.0%**, and that moved to 10.4% after
reweighting and 10.5% after oversampling.

Neither fix actually closed the gap, and I think the reason is fairly simple. Reweighting
and oversampling can only work with data that already exists, they can't invent a reading
that was never taken in the first place. If I had to draw one conclusion from this whole
section, it's that the real fix is putting more monitors where they're needed, not finding
cleverer ways to patch around the ones that are missing.

## 4. Formulation of the Problem

Air pollution is a serious environmental health risk, and accurate monitoring matters for
public health protection, regulatory decisions, and fair urban planning. That connects
directly to **SDG 11** (Sustainable Cities and Communities) and **SDG 13** (Climate
Action), and given how much air-quality data now feeds into AI-based exposure and
health-risk models, it also touches **SDG 3** (Good Health and Well-Being) and **SDG 10**
(Reduced Inequalities). The problem is that the US EPA's Air Quality System (AQS) is
basically the only data source for almost every downstream air-quality model out there,
whether that's interpolation, exposure prediction, or health-risk scoring, and all of it
rests on a network of physical ground monitors that isn't spread evenly across the
country.

**Problem statement.** Monitor placement in the ground-based AQS isn't random, and that
non-random placement creates a sample-selection bias. AI models trained on AQS data learn
from the kind of air-quality readings collected in monitored areas, and then get asked to
generalize to areas with no monitors at all. This project looks at the racial, income, and
poverty-related bias present in the US air-quality monitoring network, how much that bias
actually shows up in a downstream predictive model, and whether standard bias mitigation
techniques can meaningfully reduce the harm.

**What I set out to do, and did:**

1. Measured income, poverty rate, and racial composition against monitoring coverage
   across counties of every size.
2. Worked out statistically what's actually driving the coverage gap, after controlling
   for population size.
3. Showed with an actual predictive model what this sampling bias does to performance for
   underrepresented counties.
4. Implemented and compared two bias mitigation techniques.

I think closing this monitoring and modelling gap is a basic requirement for any AI system
that wants to claim it supports fair, SDG-aligned decisions for communities that aren't
well represented in the data to begin with.

## Data sources

- EPA Air Quality System (AQS) Monitors
- EPA AirData Annual AQI by County (2024)
- U.S. Census Bureau, Small Area Income and Poverty Estimates (SAIPE, 2023)
- U.S. Census Bureau, Population Estimates (2023)

---

Written by Dhwani Sanjay Kariya as coursework for *Emerging Artificial Intelligence
Technologies & Sustainability* (H9ETS), National College of Ireland. The original figures
(Figures 1 to 8 in the submitted report) are reproduced here as live, interactive charts on
the [project page](./index.html) instead of as static images.
