# 🇮🇳 India Climate Digital Twin

A dark-themed, interactive dashboard for the XGBoost climate forecasting models — live India map, 7/14‑day forecasts, disaster-risk alerts, natural‑language queries, and a hidden model‑accuracy panel.

**This is a pure static site — 3 files, zero build step, zero dependencies to install.** That's deliberate: it's the most reliable way to avoid the deployment errors you've been hitting.

## Files

| File | What it is |
|---|---|
| `index.html` | Page structure + all CSS |
| `app.js` | All interactivity (map clicks, query parsing, charts) |
| `data.js` | Your forecast data, baked in as a plain JS object (`const DATA = {...}`) |

No npm, no React build, no backend, no API keys. It's just HTML/CSS/JS that any browser can run directly.

## How the data got in here

- **`State Weather Forecasts`** + **`National Risk Forecasts`** sheets from your `India_Climate_Digital_Twin_Forecasts.xlsx` → the 14‑day per‑state forecast and national hazard trend.
- **`india_all_states_climate_10year_master.xlsx`** (your 10‑year, 102k‑row historical file) → used to:
  1. Compute a **real, honest model‑accuracy table** (chronological 85/15 train/test split, same methodology as your notebook — R², MAE, RMSE per target). These are genuinely computed, not made up.
  2. Build a **day‑of‑year climatology** (±3‑day window average, same trick your notebook's `forecast_7_day` engine uses) so the site can answer questions about *any* date, even ones far outside the 14‑day forecast window — e.g. "temperature in Andhra Pradesh on 2026‑12‑3" falls back to this seasonal climatology automatically and says so.
- State‑level flood/drought/heatwave/cyclone **severity multipliers** (e.g. Odisha/Andhra/Tamil Nadu weighted up for cyclone, Rajasthan/Gujarat weighted up for drought) are applied on top of the national hazard forecast, since your model outputs hazard probability nationally rather than per‑state. This is a transparent, documented approximation — swap in real per‑state hazard model output later and it'll slot in the same place (`DATA.states[name].risk`).

If you retrain your models and export a new `India_Climate_Digital_Twin_Forecasts.xlsx`, just re-run the same extraction approach to regenerate `data.js` — the page itself needs no changes as long as the shape of `DATA` stays the same (see the `DATA` shape notes at the top of `app.js`).

## Run it locally right now

You don't need anything installed except a browser. Two options:

**Option A — just open it**
Double-click `index.html`. (Some browsers restrict local `fetch`/script loading from `file://` — if the map doesn't load, use Option B.)

**Option B — tiny local server (recommended)**
```bash
cd india-climate-digital-twin
python3 -m http.server 8000
```
Then open `http://localhost:8000` in your browser.

## Deploy to GitHub Pages (no build step)

1. Create a new repo on GitHub, e.g. `india-climate-digital-twin`.
2. Put `index.html`, `app.js`, and `data.js` in the **root** of the repo (not in a subfolder, unless you adjust the `<script src>` paths).
3. Push:
   ```bash
   git init
   git add index.html app.js data.js
   git commit -m "India Climate Digital Twin"
   git branch -M main
   git remote add origin https://github.com/<your-username>/india-climate-digital-twin.git
   git push -u origin main
   ```
4. On GitHub: **Settings → Pages → Source → Deploy from a branch → `main` / `(root)` → Save**.
5. Your site goes live at `https://<your-username>.github.io/india-climate-digital-twin/` within a minute or two.

No build pipeline, no `node_modules`, no environment variables, no API keys — so there's nothing to misconfigure. If the page is blank on Pages, it's almost always one of: wrong branch selected, files not in repo root, or a typo in a filename's case (GitHub Pages is case-sensitive even if your OS isn't).

## What's interactive

- **Tap any state node on the map** → every panel (temperature, humidity, rainfall, wind, 7‑day strip, disaster alerts, 14‑day trend chart) updates to that state.
- **Ask box** understands things like:
  - "What's the weather in Kerala?"
  - "What are the chances of rainfall in Tamil Nadu?"
  - "What will be the temperature of Andhra Pradesh on 2026-12-3?" (handles the typo "Andra" too — there's light fuzzy matching)
  - "Cyclone risk in Odisha?"
  - "How accurate is this model?" → opens the hidden accuracy panel
- **Model accuracy panel** is collapsed by default (📊 button, bottom-right) and only opens on click or when asked — exactly as you asked for "hidden, shown only when asked."
- **National watch ticker** at the top auto-surfaces the highest-risk hazard/state combinations across all 28 states, cycling every few seconds.

## Honest limitations (worth knowing before you present this)

- The forecast workbook you gave me covers **2026‑01‑01 to 2026‑01‑14** — so "today/tomorrow" in the cards reflect that window, not today's real-world date. Anything outside that 14‑day window uses the climatology fallback described above, and the UI always tells you which source it used.
- Per‑state disaster‑risk numbers are the national hazard forecast scaled by a state exposure multiplier (documented above), not a true per‑state hazard model — your notebook only produces national-level hazard probabilities. This is flagged in the README and is easy to replace once/if you train per-state hazard models.
- The India outline on the map is a simplified decorative silhouette (not an official administrative boundary file) — state *positions* are accurate (plotted from your forecast file's lat/lon), but coastlines are stylized for the dark "radar" look.
