# Whose Air Is Being Watched?

**An interactive data story about income, race, and the gaps in America's air-quality monitoring network, and what those gaps do to the AI models trained on that data.**

Live page: https://dhwanikariya.github.io/air-quality-monitoring-bias/

![status](https://img.shields.io/badge/status-live-2a78d6) ![type](https://img.shields.io/badge/type-data%20story-1baf7a) ![stack](https://img.shields.io/badge/stack-vanilla%20HTML%2FCSS%2FJS-4a3aa7)

## What this is

The EPA's ground based air-quality monitors are basically the ground truth that every
downstream air-quality model relies on, whether that's EPA's own tools, an academic
exposure model, or a health-risk website. I wanted to know how evenly that network is
actually spread, so I merged four public datasets (EPA's monitor registry, EPA's 2024
Annual AQI file, and two 2023 Census products for income/poverty and race) across all
3,143 U.S. counties and pulled the numbers apart.

Three things came out of it:

1. **A lot of counties just aren't watched.** Two thirds of U.S. counties have no active
   monitor, and coverage tracks income and poverty almost linearly.
2. **Race matters, but not in the way the raw numbers first suggest.** Counting counties
   makes it look like the most diverse quartile is the best monitored. Counting people
   instead flips that completely, because low-diversity counties tend to be small and
   rural. 62.6% of residents in the least diverse quartile have no monitor nearby, versus
   10.2% in the most diverse quartile.
3. **This bias actually breaks a model, not just in theory.** I trained a Random Forest
   only on low-poverty counties and tested it on high-poverty counties it had never seen.
   It did worse there, and two standard mitigation techniques (reweighting and
   oversampling) only closed part of the gap.

The page walks through all of this: the baseline gap, the income story, a toggle between
"by county" and "by resident" views of the racial pattern, a logistic regression that
controls for population size, and the bias mitigation experiment with before and after
numbers.

## Where this came from

This started as Part 2 of a Continuous Assessment for *Emerging Artificial Intelligence
Technologies & Sustainability* (H9ETS) at the National College of Ireland. I rebuilt it
here as a standalone, interactive version of the written analysis. Every number on the
page comes straight from that report. The full write-up is in
[`REPORT.md`](./REPORT.md), minus the cover sheet, student ID, and signature pages from
the actual submission, since those don't need to be public.

## Tech

Plain HTML, CSS and JS. No build step, no dependencies, nothing loaded from a CDN. Every
chart is inline SVG that I wrote by hand, so the page loads fast and works offline. It
supports light and dark mode (auto detected, plus a manual toggle) and has an accessible
table at the bottom listing every figure used on the page.

## Running it locally

```bash
# from this folder
python -m http.server 8000
# then open http://localhost:8000
```

Or just open `index.html` in a browser, it doesn't need a server.

## Deploying to GitHub Pages

1. Push this folder to a repo.
2. In the repo settings, under Pages, set the source to the `main` branch, root folder.
3. It goes live at `https://<your-username>.github.io/<repo-name>/`.

```bash
git init
git add .
git commit -m "Add air-quality monitoring bias data story"
git branch -M main
git remote add origin https://github.com/DhwaniKariya/air-quality-monitoring-bias.git
git push -u origin main
```

## Data sources

- EPA Air Quality System (AQS) Monitors
- EPA AirData Annual AQI by County (2024)
- U.S. Census Bureau, Small Area Income and Poverty Estimates (SAIPE, 2023)
- U.S. Census Bureau, Population Estimates (2023)

## License

The code in this repo (`index.html` and anything else in here) is under the MIT License,
see [`LICENSE`](./LICENSE). The written analysis and figures are my own coursework.

---

Built by [Dhwani Sanjay Kariya](https://github.com/DhwaniKariya), MSc AI for Business,
National College of Ireland.
