# Project brief: Soccer xG shot map

## Goal
A single-page website where the user clicks a spot on a soccer pitch, sets a few shot conditions, and instantly sees the expected goals (xG) from our own logistic regression model. A second panel compares our model with StatsBomb's xG.

## Hard constraints
- One standalone file: `index.html` (HTML, CSS and vanilla JavaScript). No build step, no framework, no server.
- It must work when opened by double-click (file://). Browsers block `fetch` of local JSON there, so embed the model coefficients, the comparison data and the heatmap parameters directly inside `index.html`.
- No external dependencies except, optionally, Chart.js from a CDN. Plain SVG charts are preferred.
- Footer must say: "Data: StatsBomb Open Data (github.com/statsbomb/open-data). Model: logistic regression fitted by the author."
- Works on a laptop screen and does not break on a phone width.

## Files in `data/`
- `xg_model.json`: final model. `features` (ordered), `coef` (same order), `intercept`, `n_shots`.
- `comparison.json`: metrics for our model vs StatsBomb (mean over 5 match-level splits) and calibration bins (observed goal rate vs predicted, 8 bins each).
- `tests.json`: 8 golden test cases with exact expected distance, angle and xG. The JavaScript must reproduce them within 0.001.
- `xg_shots.csv`: 16,164 non-penalty shots (columns: x, y, body_part, shot_type, outcome, sb_xg, under_pressure, first_time, defenders_in_cone and others). Use it only if you want to draw real shots on the pitch. It is not needed to compute xG.

## Coordinates (StatsBomb)
- Pitch is 120 x 80. The attacking goal is at x = 120. Goal posts are at y = 36 and y = 44 (goal mouth 8 wide). The center of the goal is (120, 40).
- Draw only the attacking half (x from 60 to 120) by default, with a toggle for the full pitch.

## Model (must match exactly)
For a shot at (x, y):
- dx = 120 - x, dy = |y - 40|
- distance = sqrt(dx^2 + dy^2)
- logd = ln(max(distance, 1))
- angle = |atan2(44 - y, dx) - atan2(36 - y, dx)|   (radians, the angle the goal mouth subtends)
- head = 1 if body part is Head else 0
- foot_other = 1 if body part is Other else 0 (knee, chest and similar)
- fk = 1 if the shot is a free kick else 0
- under_pressure = 0 or 1, first_time = 0 or 1
- cone = number of defenders inside the triangle shooter, left post, right post (user sets 0 to 8 with a stepper)
- z = intercept + sum(coef_i * feature_i), in the order given by `features`
- xG = 1 / (1 + exp(-z))
Penalties are not modeled. Right foot and left foot are the same category.

## UI requirements
1. Pitch (SVG): click or drag to place the shot. Show the shot-to-goal triangle (cone) and a live label with distance (yards) and angle (degrees).
2. Controls: body part (Right foot, Left foot, Head, Other), free kick toggle, under pressure toggle, first-time shot toggle, defenders in cone (0 to 8).
3. Result card: xG as a percentage with a one-line plain-English reading ("about 1 goal in 8 such shots").
4. Heatmap toggle: color every point of the half pitch by xG for the current settings (use a grid of about 2 yards).
5. "What changed" line: shows how xG moved versus the previous click.
6. Comparison tab: table of AUC, log loss, Brier and correlation with StatsBomb for both models, and a calibration chart (predicted vs observed, both models, with the diagonal).
7. About tab: how the model works in 5 plain bullets, data source, and limitations (below).

## Known limitations (show them on the page)
- Trained on 16,164 shots from major tournaments and one Premier League season, so it reflects those matches only.
- No goalkeeper position, no shot speed, no assist type. StatsBomb's own xG uses more context and scores better (AUC about 0.79 vs 0.76).
- Free kicks, and shots from within about 2 yards, are the least reliable estimates.
- Defender count is entered by hand here, while in the data it comes from the freeze frame.

## Acceptance checklist
- [ ] All 8 cases in `tests.json` match within 0.001 (add a visible self-test, for example a small "tests passed" badge or a console message).
- [ ] Moving the shot closer to the goal centre raises xG. A header gives a lower xG than a foot shot from the same spot. More defenders in the cone lowers xG.
- [ ] Heatmap is highest near the goal centre and fades toward the corners and sidelines.
- [ ] Works from file:// with no console errors.
- [ ] Comparison numbers on the page equal `comparison.json`.
- [ ] Footer credit present.

## Prompts to give the agent, in order
Paste one at a time. Review the result, run it, then continue.

### Prompt 1 (use /plan first)
Read BRIEF.md and the files in data/. Plan, then build index.html with the pitch (SVG, attacking half), click to place a shot, the controls, and the xG calculation from the embedded coefficients. Add the self-test against tests.json. Do not build the heatmap or the comparison tab yet. Show me the plan before you code.

### Prompt 2
Add the heatmap toggle and the "what changed" line. Check that xG rises toward the goal centre and fades toward the corners. Take a screenshot of the heatmap for a header and for a free kick.

### Prompt 3
Add the Comparison tab using comparison.json: the metrics table and an SVG calibration chart for both models with the diagonal. Embed the data in the file. Make sure the numbers match the JSON exactly.

### Prompt 4
Add the About tab with the 5 plain-English bullets, the data source, and the limitations from BRIEF.md. Add the footer credit. Polish the layout for a laptop and a phone width. Then run through the acceptance checklist and report each item as pass or fail.

### Prompt 5 (optional)
Add a "real shots" layer: a toggle that draws sample shots from xg_shots.csv (embed about 300 random shots) with goals in one color and misses in another, and show our xG and StatsBomb xG on hover.

## Process log (for the write-up)
Keep a short log while you work. For each step note: the prompt, what the agent produced (a screenshot helps), what you had to fix, and what you learned. Antigravity's plan and walkthrough artifacts are good evidence to save.
