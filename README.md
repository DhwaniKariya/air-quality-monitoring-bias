# Whose Air Is Being Watched?

**An interactive data story on income, race, and America's air-quality monitoring gap — and how that gap biases the AI models trained on it.**

🔗 **Live page:** _add your GitHub Pages URL here once enabled, e.g. `https://dhwanikariya.github.io/air-quality-monitoring-bias/`_

![status](https://img.shields.io/badge/status-live-2a78d6) ![type](https://img.shields.io/badge/type-data%20story-1baf7a) ![stack](https://img.shields.io/badge/stack-vanilla%20HTML%2FCSS%2FJS-4a3aa7)

## What this is

The EPA's ground-based air-quality monitoring network is the source of truth every
downstream air-quality model — EPA's own tools, academic exposure models, health-risk
websites — relies on to estimate pollution levels everywhere else. That network isn't
spread evenly.

This project merges four public datasets — EPA's monitor registry, EPA's 2024 Annual AQI
file, and two 2023 U.S. Census products (SAIPE income/poverty and county population by
race) — across all 3,143 U.S. counties to answer three questions:

1. **Who gets left off the map?** Two-thirds of U.S. counties have no active monitor, and
   coverage tracks income and poverty almost linearly.
2. **Does race matter once you control for population?** Raw county counts say no — but
   weighting by *who actually lives there* flips the story: low-diversity counties are
   small and rural, so 62.6% of residents in the least-diverse quartile have no monitor
   nearby, versus 10.2% in the most-diverse quartile.
3. **Does this sampling bias actually break a model?** A Random Forest trained only on
   low-poverty counties and tested on held-out high-poverty counties confirms it does —
   and two standard mitigation techniques (reweighting, oversampling) only partly fix it.

The page walks through all of it: the baseline gap, the income story, a toggle between
"by county" and "by resident" views of the racial pattern, a logistic-regression
confound-control chart, and the bias-mitigation experiment with before/after results.

## Origin

This is Part 2 of a Continuous Assessment for *Emerging Artificial Intelligence
Technologies & Sustainability* (H9ETS) at the National College of Ireland — rebuilt here
as a standalone, interactive version of the written analysis. All figures on the page are
taken directly from that report; see [`REPORT.md`](./REPORT.md) for the full write-up.
The original submitted PDF is not included in this repo, since it also contains a signed
cover sheet, student ID, and academic-integrity certificate.

## Tech

Plain HTML/CSS/JS. No build step, no dependencies, no CDN calls — every chart is
hand-rolled inline SVG so the page works offline and loads instantly on GitHub Pages.
Supports light/dark mode (auto-detected + manual toggle) and includes an accessible data
table of every figure used.

## Run locally

```bash
# from this folder
python -m http.server 8000
# then open http://localhost:8000
```

Or just open `index.html` directly in a browser.

## Deploy to GitHub Pages

1. Push this folder to a repo (see below).
2. In the repo settings → **Pages**, set source to the `main` branch, root folder.
3. Your page goes live at `https://<your-username>.github.io/<repo-name>/`.

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

Code in this repository (`index.html` and any scripts) is released under the MIT License
— see [`LICENSE`](./LICENSE). The written analysis and figures are original academic
coursework by the author.

---

Built by [Dhwani Sanjay Kariya](https://github.com/DhwaniKariya) — MSc AI for Business,
National College of Ireland.
