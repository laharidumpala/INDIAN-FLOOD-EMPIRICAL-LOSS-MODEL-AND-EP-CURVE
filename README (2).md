# An Empirical Loss Model and Exceedance Probability Curve for Indian Flood, Validated Against Independent National Data

## What this is

A bottom-up empirical loss model for Indian flood, built from historical event loss data (EM-DAT, India, Flood, 2000–2025), producing an Average Annual Loss (AAL) figure and full Exceedance Probability (EP) curves with bootstrapped uncertainty.

This is **not a catastrophe model** — there is no hazard module, no vulnerability function, and no exposure database, because those require licensed vendor data (Verisk, Moody's RMS, and similar) not publicly available. This is the statistical and simulation half of that pipeline: distribution fitting, Monte Carlo simulation, and return-period estimation, built entirely from historical loss records.

**Notebook:** `flood_ep_curve_analysis.ipynb` — fully runnable in Google Colab, upload your own EM-DAT export and run top to bottom.

## Headline results

- **Average Annual Loss (AAL):** ~US\$13.2 billion (aggregate basis)
- **1-in-100 year loss:** ~US\$91.0 billion (90% bootstrap CI: \$44.4B – \$210.7B)
- **1-in-250 year loss:** ~US\$121.5 billion (90% bootstrap CI: \$51.7B – \$443.3B)

The wide confidence intervals at long return periods are an honest, expected consequence of fitting extreme-tail behavior from a ~70-event historical sample — reported explicitly rather than hidden behind a single point estimate.

## Method

**Data:** EM-DAT, filtered to India / Flood / 2000–2025. 1990–1999 excluded due to EM-DAT's own documented pre-2000 reporting-completeness bias. Of 205 total flood records in this window, 70 carry a usable damage estimate; those 70 form the severity-fitting sample.

**Normalization:** EM-DAT's "Adjusted" damage column already corrects for inflation. A second adjustment — exposure growth — is applied on top, using a smoothed compound annual growth rate in nominal GDP (anchored at FY2000 ≈ \$468B and FY2024 = \$3,912.69B, World Bank) as a Pielke-style proxy for how much more built-up exposure India has today than in 2000.

**Frequency:** Tested for overdispersion (variance/mean ratio) before choosing between Poisson and Negative Binomial, rather than assuming Poisson by default.

**Severity:** A blended model — lognormal fit to the bulk of the distribution (chosen over gamma by AIC), and a Generalized Pareto Distribution fit to the tail above the 75th percentile (peaks-over-threshold, standard extreme value theory practice). A threshold stability check was run; it did not show a clean plateau, indicating the tail parameters carry real sensitivity to threshold choice given the sample size — this is reported as a limitation, not smoothed over.

**Simulation:** 10,000-year Monte Carlo, producing both AEP (aggregate annual loss) and OEP (single worst event per year) curves — reporting both, since they answer different questions and conflating them is a common modelling error.

**Uncertainty:** 1,000-resample bootstrap around every return-period estimate, reported as a 90% confidence interval rather than a bare point figure.

## Independent validation

EM-DAT's aggregate 2000–2022 damage total was cross-checked against India's own primary source: the **Central Water Commission's "Report on Flood Damage Statistics (1953–2022)"** (July 2024), which compiles state-confirmed administrative flood damage into a full national annual series.

After converting CWC's nominal Rupee figures to USD and applying US CPI inflation adjustment (matching EM-DAT's own normalization convention) to a common 2000–2022 window:

| | CWC (national administrative) | EM-DAT (international, threshold-filtered) |
|---|---|---|
| Total damage, 2000–2022 | ~\$105.4 billion | ~\$105.4 billion |
| Total deaths, 2000–2022 | 42,133 | 26,343 (damage-bearing events only) |

**Two independently compiled databases — one Indian government administrative, one international humanitarian/academic — converge closely on the same order of magnitude for India's flood damage over 23 years.** This is genuine evidence the model's overall scale is reasonable.

At the same time, **individual years diverge sharply, in both directions** (some years CWC is many times EM-DAT's figure, other years the reverse). This is explained by a structural difference rather than error: EM-DAT logs only events clearing an international severity threshold, while CWC sums *every* state-reported flood loss regardless of scale. The close aggregate match should not be read as proof that either series is accurate event-by-event — with this much offsetting divergence, it is partly a product of errors cancelling out across years. The honest claim is: same order of magnitude in aggregate, sharp divergence at the individual-year level, for an explainable reason.

## Key limitations

- **DesInventar** was investigated as a cross-check source but has no national India coverage (only three individual states are digitized).
- The **India Flood Inventory** (IIT Delhi / HydroSense Lab, peer-reviewed) was investigated but has no structured dollar-damage field and stops at 2023 — it strengthens frequency evidence qualitatively but couldn't extend the severity model.
- The exposure-growth normalization uses a two-point GDP CAGR rather than a full annual GDP series — a standard simplification, chosen to avoid injecting short-term GDP noise into a structural exposure-growth adjustment.
- The GPD tail fit shows real sensitivity to threshold choice (no stability plateau found), directly contributing to the wide confidence intervals at 1-in-100 and 1-in-250.
- The CWC cross-check's USD/INR exchange rates and US CPI figures are standard published annual averages, not independently re-verified against RBI/FBIL or BLS primary tables — the ~\$105B convergence is directionally solid, not precise to the dollar.

## Repository contents

- `flood_ep_curve_analysis.ipynb` — full runnable Colab notebook
- `ep_curve_results.csv` — return-period results with bootstrap confidence intervals
- `normalized_events.csv` — the 70-event normalized severity dataset
- `cwc_emdat_comparison.csv` — the independent validation comparison table
- Three figures: normalization effect, EP curve, threshold stability check

## Sources

EM-DAT, The International Disaster Database (CRED / UCLouvain); Central Water Commission, Government of India, *Report on Flood Damage Statistics (1953–2022)*, July 2024; World Bank national accounts data (GDP); U.S. Bureau of Labor Statistics (CPI-U).
