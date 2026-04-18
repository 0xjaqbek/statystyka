# Design: Layout Redesign & Missing Elements

**Date:** 2026-04-18
**Project:** Statystyka PANS — index.html
**Scope:** Add missing guides.md elements; remove all toggle/click interactions

---

## Problem

The current `index.html` is missing three categories of required content from `guides.md`:
1. Single-variable statistical parameters (mean, median, quartiles, variance, std dev, IQR, skewness)
2. Regression model display (equation, β coefficients, R², significance)
3. Time series analysis section (multi-year data, trend model, average rate of change)

Additionally, the disease section requires clicking a toggle to switch between hypertension and diabetes — all data must be visible simultaneously.

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

The dataset is extended from 2 time points (2000, 2020) to 5 time points: **2000, 2005, 2010, 2015, 2020** for both regions (Mazowieckie, Lubelskie) and both diseases (nadciśnienie, cukrzyca). Values are simulated to approximate real BDL trends (BDL API does not expose voivodeship-level chronic disease incidence data; the professor's own notes acknowledge BDL limitations for this data).

PKB per capita is similarly extended to 5 points.

Total dataset: 5 years × 2 regions × 3 variables (PKB, nadciśnienie, cukrzyca) = 30 data points.

---

## Page Structure

### Header / Nav
- Existing nav bar retained
- Intro paragraph shortened to 2 sentences
- Small sticky jump-nav (top-right, `position: fixed`) with anchor links to `#s1`, `#s2`, `#s3` for navigation without revealing hidden content

### Section 1 — Analiza jednej zmiennej (nadciśnienie)
**Variable:** nadciśnienie (hypertension), both regions, all 5 years → 10 observations

**Left column — Stats panel:**
- Średnia (mean)
- Mediana (median)
- Q1, Q3 (quartiles)
- Wariancja (variance)
- Odchylenie standardowe (std dev)
- Odchylenie ćwiartkowe / IQR (Q3 − Q1)
- Współczynnik skośności (skewness — Pearson's moment coefficient)

All computed in JS from the 10-value array at runtime.

**Right column — Charts (stacked):**
- Grouped bar chart: all 10 observations (x-axis: year+region label, y-axis: cases per 10k) — serves as histogram for discrete observations
- Box plot visualization: rendered via Chart.js using min, Q1, median, Q3, max of the 10-value array, displayed as a floating bar chart approximation (Chart.js has no native box plot; use `bar` type with computed segments)

---

### Section 2 — Analiza dwóch zmiennych
**Variables:** nadciśnienie and cukrzyca vs PKB per capita

**Row 1 — Two side-by-side line charts (no toggle):**
- Left: Nadciśnienie — Mazowieckie vs Lubelskie, 2000–2020
- Right: Cukrzyca — Mazowieckie vs Lubelskie, 2000–2020

**Row 2 — Scatter plot (korelogram):**
- X-axis: PKB per capita; Y-axis: disease rate per 10k
- Two datasets: nadciśnienie (red/coral) and cukrzyca (blue), 10 points each
- Covers "wykres (korelogram)" requirement

**Row 3 — Regression summary card:**
- Model displayed: `Ŷ = β₀ + β₁ · PKB`
- Computed β₀ and β₁ via JS ordinary least squares (OLS) on 10 data points
- R² displayed
- Significance note: qualitative interpretation of R² (e.g. "model wyjaśnia XX% zmienności") — formal p-value testing is done in Gretl, not JS; note this explicitly in the card
- Covers: model regresji liniowej, interpretacja parametrów, R², ocena istotności

---

### Section 3 — Analiza szeregu czasowego
**Variable:** nadciśnienie, Mazowieckie only (single series, 5 time points)

**Left column — Stats panel:**
- Średnie tempo zmian: geometric mean of chain relatives `(V_t / V_{t-1})` over 4 periods
- Trend model: `Ŷ = a + b·t` (t = 1…5), computed via JS OLS on time index
- R² for trend model
- Significance note: qualitative R² interpretation; formal testing in Gretl

**Right column — Chart:**
- Line chart with two series: actual values (solid) and fitted trend line (dashed)
- X-axis: year labels; Y-axis: cases per 10k

**Below both columns:**
- Expanded raw data table (5 years × 2 regions × all variables, sortable by year)
- "Kopiuj do schowka" button retained
- Data sources card (moved from old position)

---

### Footer
Unchanged.

---

## Statistical Formulas (JS implementation)

| Statistic | Formula |
|---|---|
| Mean | `Σx / n` |
| Median | sort, middle value (average of two middle if even n) |
| Q1, Q3 | lower/upper half medians |
| IQR | Q3 − Q1 |
| Variance | `Σ(x − mean)² / n` (population) |
| Std dev | `√variance` |
| Skewness | `(mean − median) / std_dev` × 3 (Pearson's 2nd) |
| OLS β₁ | `Σ(x−x̄)(y−ȳ) / Σ(x−x̄)²` |
| OLS β₀ | `ȳ − β₁·x̄` |
| R² | `1 − SS_res / SS_tot` |
| Avg rate of change | `(V_n / V_1)^(1/(n-1))` |

---

## What is NOT changing
- Color palette (indigo/emerald/sky)
- Nav bar and footer
- Font Awesome icons
- Copy-to-clipboard functionality
- Data sources section content (only repositioned)
- Overall card/shadow styling

---

## Out of scope
- Analiza przeżycia (survival analysis) — guides.md marks this as "tylko wykład" (lecture only)
- Wnioskowanie statystyczne (inferential statistics) — not in the project rubric
- Real BDL data fetch — API does not expose required indicators
