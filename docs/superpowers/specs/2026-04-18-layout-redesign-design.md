# Design: Layout Redesign & Missing Elements

**Date:** 2026-04-18
**Project:** Statystyka PANS — index.html
**Scope:** Add missing guides.md elements; remove all toggle/click interactions; switch from Polish regional data (BDL) to real EU country data (WHO GHO + World Bank)

---

## Problem

The current `index.html` is missing three categories of required content from `guides.md`:
1. Single-variable statistical parameters (mean, median, quartiles, variance, std dev, IQR, skewness)
2. Regression model display (equation, β coefficients, R², significance)
3. Time series analysis section (multi-year data, trend model, average rate of change)

Additionally, the disease section requires clicking a toggle to switch between hypertension and diabetes — all data must be visible simultaneously. The original BDL data (2 Polish voivodeships, 2 simulated time points) is replaced with real WHO GHO + World Bank data for Luxembourg vs Bulgaria (5 real time points).

---

## Solution: Option B — Restructure around guides.md sections

Reorganize `index.html` into three clearly numbered analysis sections that map 1:1 to the professor's rubric. All content is visible on scroll — no toggles, no tabs, no clicks required.

---

## Architecture

Single HTML file. No new files. Existing dependencies retained:
- Tailwind CSS (CDN)
- Chart.js (CDN)
- Font Awesome (CDN)

All statistical computations (mean, median, quartiles, variance, std dev, IQR, skewness, least-squares regression, geometric mean) are implemented in JavaScript and run on page load.

---

## Data

**Comparison groups:** Luxembourg (LU — richest EU member) vs Bulgaria (BG — poorest EU member)

**Years:** 2000, 2005, 2010, 2015, 2019 (5 points, every 5 years)

**Hypertension prevalence % — WHO GHO (NCD_HYP_PREVALENCE_A, age-standardized, both sexes, adults 30–79):**
| Year | Luxembourg | Bulgaria |
|------|-----------|---------|
| 2000 | 40.3 | 44.8 |
| 2005 | 39.5 | 44.8 |
| 2010 | 37.3 | 45.2 |
| 2015 | 33.6 | 45.2 |
| 2019 | 30.5 | 45.2 |

**Diabetes prevalence % — WHO GHO (NCD_DIABETES_PREVALENCE_AGESTD, age-standardized, both sexes):**
| Year | Luxembourg | Bulgaria |
|------|-----------|---------|
| 2000 | 5.10 | 5.92 |
| 2005 | 5.36 | 6.52 |
| 2010 | 5.55 | 7.42 |
| 2015 | 5.61 | 8.55 |
| 2019 | 5.73 | 9.69 |

**GDP per capita (current USD) — World Bank (NY.GDP.PCAP.CD):**
| Year | Luxembourg | Bulgaria |
|------|-----------|---------|
| 2000 | 48660 | 1621 |
| 2005 | 80988 | 3900 |
| 2010 | 110886 | 6854 |
| 2015 | 105462 | 7269 |
| 2019 | 112697 | 10354 |

**Narrative:** The richer country (Luxembourg) shows declining hypertension and stable low diabetes. The poorer country (Bulgaria) shows stable high hypertension and rapidly rising diabetes. Both indicators correlate negatively with GDP — richer = healthier outcomes.

Total dataset: 5 years × 2 countries × 3 variables = 30 real data points.

---

## Page Structure

### Header / Nav
- Existing nav bar retained
- Title updated to reflect Luxembourg vs Bulgaria comparison
- Intro paragraph shortened to 2 sentences
- Small sticky jump-nav (top-right, `position: fixed`) with anchor links to `#s1`, `#s2`, `#s3`

### Section 1 — Analiza jednej zmiennej (nadciśnienie)
**Variable:** hypertension prevalence (%), both countries, all 5 years → 10 observations

**Left column — Stats panel:**
- Średnia (mean)
- Mediana (median)
- Q1, Q3 (quartiles)
- Wariancja (variance)
- Odchylenie standardowe (std dev)
- Odchylenie ćwiartkowe / IQR (Q3 − Q1)
- Współczynnik skośności (skewness — Pearson's 2nd moment coefficient)

All computed in JS from the 10-value array at runtime.

**Right column — Charts (stacked):**
- Grouped bar chart: all 10 observations (x-axis: year+country label, y-axis: prevalence %) — serves as histogram for discrete observations
- Box plot visualization: floating bar chart approximation using min, Q1, median, Q3, max (Chart.js has no native box plot; implemented via horizontal `bar` type with 4 stacked floating segments)

---

### Section 2 — Analiza dwóch zmiennych
**Variables:** hypertension and diabetes prevalence vs GDP per capita

**Row 1 — Two side-by-side line charts (no toggle):**
- Left: Hypertension — Luxembourg vs Bulgaria, 2000–2019
- Right: Diabetes — Luxembourg vs Bulgaria, 2000–2019

**Row 2 — Scatter plot (korelogram):**
- X-axis: GDP per capita (USD); Y-axis: prevalence (%)
- Two datasets: hypertension (red/coral) and diabetes (blue), 10 points each
- Covers "wykres (korelogram)" requirement

**Row 3 — Regression summary card:**
- Model displayed: `Ŷ = β₀ + β₁ · GDP`
- Computed β₀ and β₁ via JS OLS on 10 data points (both countries, all years)
- R² displayed with qualitative interpretation ("model wyjaśnia XX% zmienności")
- Note that formal p-value testing is done in Gretl
- Covers: model regresji liniowej, interpretacja parametrów, R², ocena istotności

---

### Section 3 — Analiza szeregu czasowego
**Variable:** hypertension prevalence, Luxembourg only (single series, 5 time points — strong declining trend)

**Left column — Stats panel:**
- Średnie tempo zmian: geometric mean of chain relatives over 4 periods
- Trend model: `Ŷ = a + b·t` (t = 1…5), computed via JS OLS on time index
- R² for trend model with qualitative interpretation
- Interpretation of β₁ (change per 5-year period)
- Note: formal significance testing in Gretl

**Right column — Chart:**
- Line chart with two series: actual values (solid indigo) and fitted trend line (dashed red)
- X-axis: year labels; Y-axis: prevalence (%)

**Below both columns:**
- Raw data table (5 years × 2 countries × all 3 variables)
- "Kopiuj do schowka" button
- Data sources card with WHO GHO and World Bank links

---

### Footer
Unchanged.

---

## Statistical Formulas (JS implementation)

| Statistic | Formula |
|---|---|
| Mean | `Σx / n` |
| Median | sort, middle value (average of two middle if even n) |
| Q1, Q3 | linear interpolation on sorted array |
| IQR | Q3 − Q1 |
| Variance | `Σ(x − mean)² / n` (population) |
| Std dev | `√variance` |
| Skewness | `3 × (mean − median) / std_dev` (Pearson's 2nd) |
| OLS β₁ | `Σ(x−x̄)(y−ȳ) / Σ(x−x̄)²` |
| OLS β₀ | `ȳ − β₁·x̄` |
| R² | `1 − SS_res / SS_tot` |
| Avg rate of change | `(V_n / V_1)^(1/(n-1))` |

---

## What is NOT changing
- Color palette (indigo for Luxembourg, emerald for Bulgaria, sky accents)
- Nav bar and footer
- Font Awesome icons
- Copy-to-clipboard functionality
- Overall card/shadow styling

---

## Out of scope
- Analiza przeżycia (survival analysis) — guides.md marks this as "tylko wykład" (lecture only)
- Wnioskowanie statystyczne (inferential statistics) — not in the project rubric
- Fetching data live from WHO/World Bank APIs at runtime — data is hardcoded from verified API responses
