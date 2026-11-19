# Soccer xG Shot Map

An interactive, single-file expected goals (xG) explorer for soccer. Click or drag anywhere on the pitch, adjust shot conditions, and immediately see the predicted scoring probability from a logistic regression model trained on StatsBomb Open Data — plus a live spatial heatmap and a head-to-head benchmark against StatsBomb's own xG model.

> **Note:** This is a simple vibe-coded app built with **Gemini 4 Neon**.

## Features

- **Interactive SVG Pitch**: Attacking-half view by default with a toggle for the full pitch, shot-to-goalposts cone, and live distance (meters) and goal-angle (degrees) readouts.
- **Shot Condition Controls**: Body part (`Right Foot`, `Left Foot`, `Head`, `Other`), `Free kick`, `Under pressure`, `First-time shot`, and `Defenders in cone` (`0–8`).
- **Live xG Result Card**: 1-decimal percentage + raw `0`–`1` probability, plain-English translation (*"About 1 goal in 5 such shots"*), a **"what changed"** delta line, and inline reliability/free-kick wall notes.
- **Spatial xG Heatmap**: Toggleable `1.8 m` (`2 yd`) grid across the attacking half that updates dynamically with shot conditions.
- **Real Shots Layer**: Toggleable overlay of 300 sampled non-penalty shots from `data/xg_shots.csv` (goals in green, misses in rose) with hover readouts comparing Our xG vs StatsBomb xG.
- **Model Comparison Tab**: Out-of-sample metrics table (`ROC AUC`, `Log Loss`, `Brier Score`, `Correlation with StatsBomb`) and an interactive SVG calibration chart with the perfect-calibration diagonal.
- **Built-in Golden Self-Test**: Automatically verifies all 8 reference test cases from `data/tests.json` within `±0.001` on every page load.
- **Zero Dependencies**: Everything (HTML, CSS, vanilla JS, SVG graphics, model coefficients, and comparison data) lives in a single standalone `index.html` file that works directly over `file://`.

## How to Run

No build step, server, or package install required:

1. Clone or download this repository.
2. Double-click `index.html` to open it in any modern browser.

## Model & Data

- **Dataset**: 16,164 non-penalty shots (1,484 goals) from [StatsBomb Open Data](https://github.com/statsbomb/open-data) across major international tournaments and one Premier League season.
- **Model**: Logistic regression fitted by the author on 8 features (`log(distance)`, goal angle, `head`, `foot_other`, `free_kick`, `under_pressure`, `first_time`, `defenders_in_cone`).
- **Units**: Displayed distances and coordinates are shown in meters (`1 yd = 0.9144 m`), while the model computes features internally in StatsBomb's native `120 × 80` yard coordinate system.

### Benchmark Summary (5 Match-Level 75/25 Splits)

| Metric | Our Model | StatsBomb xG |
| :--- | :---: | :---: |
| ROC AUC | `0.7582` | `0.7916` |
| Log Loss | `0.2710` | `0.2547` |
| Brier Score | `0.0761` | `0.0711` |
| Corr. with StatsBomb | `0.8145` | `1.0000` |

## Known Limitations

- Trained on 16,164 shots from major tournaments and one Premier League season, so it reflects those matches only.
- Does not include goalkeeper position, shot speed, or assist type (StatsBomb's own xG uses richer context and scores higher: AUC ~`0.79` vs `0.76`).
- Free kicks and shots within about `2 yards` (`1.8 m`) of goal are the least reliable estimates.
- Defender count in the shooting cone is entered manually here, whereas in the training data it comes from StatsBomb freeze frames.

## Project Files

- `index.html` — Complete standalone web application.
- `data/xg_model.json` — Fitted logistic regression coefficients and intercept.
- `data/comparison.json` — Evaluation metrics and 8-bin calibration data.
- `data/tests.json` — 8 golden test cases for model verification.
- `data/xg_shots.csv` — 16,164 non-penalty shots dataset.

## Credits & Data Attribution

- **Data provided by StatsBomb** ([StatsBomb Open Data](https://github.com/statsbomb/open-data)).
- **Model**: Logistic regression fitted by the author.

### StatsBomb Open Data Terms

Per the StatsBomb Open Data repository terms and Public Data User Agreement (`LICENSE.pdf` in `github.com/statsbomb/open-data`):

> *"If you publish, share or distribute any research, analysis or insights based on this data, please state the data source as StatsBomb and use our logo, available in our Media Pack."*

- The data is provided for non-commercial research, educational, and public analysis use only; all intellectual property rights remain with StatsBomb.
- If you publish or share screenshots/demos of this project publicly, include the official StatsBomb logo from their Media Pack alongside the attribution above, and avoid redistributing the raw bulk dataset (`data/xg_shots.csv`) directly.
