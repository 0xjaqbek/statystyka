# Layout Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Completely rewrite `index.html` into three analysis sections (single-variable, two-variable, time series) matching guides.md requirements, using real WHO GHO + World Bank data for Luxembourg vs Bulgaria, with all content visible on scroll — no toggles.

**Architecture:** Single HTML file with embedded CSS and JS. All statistical computations (mean, median, quartiles, variance, std dev, IQR, skewness, OLS, R², geometric mean) run as pure JS on page load. Data hardcoded from verified WHO GHO and World Bank API responses. Chart.js for all charts; box plot approximated via floating horizontal bar chart.

**Tech Stack:** HTML, Tailwind CSS (CDN), Chart.js (CDN), Font Awesome (CDN), vanilla JavaScript.

---

### Task 1: Replace data store and add statistical utility functions

**Files:**
- Modify: `index.html` — replace entire `<script>` block contents

- [ ] **Step 1: Replace the entire contents of the `<script>` tag** (keep the `<script>` and `</script>` tags, replace everything between them) with:

```javascript
// --- DATA STORE (real data: WHO GHO + World Bank) ---
const db = {
    years: [2000, 2005, 2010, 2015, 2019],
    countries: ['Luksemburg (Bogate)', 'Bułgaria (Biedne)'],
    gdp: {
        luxembourg: [48660, 80988, 110886, 105462, 112697],
        bulgaria:   [1621,  3900,  6854,   7269,   10354]
    },
    hypertension: {
        luxembourg: [40.3, 39.5, 37.3, 33.6, 30.5],
        bulgaria:   [44.8, 44.8, 45.2, 45.2, 45.2]
    },
    diabetes: {
        luxembourg: [5.10, 5.36, 5.55, 5.61, 5.73],
        bulgaria:   [5.92, 6.52, 7.42, 8.55, 9.69]
    }
};

// --- STATISTICAL UTILITIES ---
const stat = {
    mean(arr) {
        return arr.reduce((a, b) => a + b, 0) / arr.length;
    },
    sorted(arr) {
        return [...arr].sort((a, b) => a - b);
    },
    median(arr) {
        const s = this.sorted(arr);
        const mid = Math.floor(s.length / 2);
        return s.length % 2 === 0 ? (s[mid - 1] + s[mid]) / 2 : s[mid];
    },
    quartile(arr, q) {
        const s = this.sorted(arr);
        const pos = (s.length - 1) * q;
        const base = Math.floor(pos);
        const rest = pos - base;
        return s[base + 1] !== undefined
            ? s[base] + rest * (s[base + 1] - s[base])
            : s[base];
    },
    variance(arr) {
        const m = this.mean(arr);
        return arr.reduce((sum, x) => sum + (x - m) ** 2, 0) / arr.length;
    },
    stdDev(arr) {
        return Math.sqrt(this.variance(arr));
    },
    skewness(arr) {
        const sd = this.stdDev(arr);
        return sd === 0 ? 0 : 3 * (this.mean(arr) - this.median(arr)) / sd;
    },
    ols(xArr, yArr) {
        const xMean = this.mean(xArr);
        const yMean = this.mean(yArr);
        const num = xArr.reduce((sum, xi, i) => sum + (xi - xMean) * (yArr[i] - yMean), 0);
        const den = xArr.reduce((sum, xi) => sum + (xi - xMean) ** 2, 0);
        const beta1 = den === 0 ? 0 : num / den;
        const beta0 = yMean - beta1 * xMean;
        return { beta0, beta1 };
    },
    rSquared(xArr, yArr) {
        const { beta0, beta1 } = this.ols(xArr, yArr);
        const yMean = this.mean(yArr);
        const ssTot = yArr.reduce((sum, yi) => sum + (yi - yMean) ** 2, 0);
        const ssRes = xArr.reduce((sum, xi, i) => {
            return sum + (yArr[i] - (beta0 + beta1 * xi)) ** 2;
        }, 0);
        return ssTot === 0 ? 1 : 1 - ssRes / ssTot;
    },
    geoMeanRate(arr) {
        return Math.pow(arr[arr.length - 1] / arr[0], 1 / (arr.length - 1));
    }
};

Chart.defaults.font.family = "'Inter', sans-serif";
Chart.defaults.color = '#4b5563';

document.addEventListener('DOMContentLoaded', () => {
    initSection1();
    initSection2();
    initSection3();
    populateTable();
});

function initSection1() {
    const values = [...db.hypertension.luxembourg, ...db.hypertension.bulgaria];
    const q1 = stat.quartile(values, 0.25);
    const q3 = stat.quartile(values, 0.75);
    const s = {
        mean:     stat.mean(values),
        median:   stat.median(values),
        q1,
        q3,
        variance: stat.variance(values),
        stdDev:   stat.stdDev(values),
        iqr:      q3 - q1,
        skewness: stat.skewness(values),
        min:      Math.min(...values),
        max:      Math.max(...values)
    };

    const tbody = document.querySelector('#s1-stats-table tbody');
    [
        ['Średnia (mean)',            s.mean.toFixed(2) + '%'],
        ['Mediana',                   s.median.toFixed(2) + '%'],
        ['Q1 (dolny kwartyl)',         s.q1.toFixed(2) + '%'],
        ['Q3 (górny kwartyl)',         s.q3.toFixed(2) + '%'],
        ['Wariancja',                 s.variance.toFixed(3)],
        ['Odchylenie standardowe',    s.stdDev.toFixed(3)],
        ['Odchylenie ćwiartkowe IQR', s.iqr.toFixed(2)],
        ['Współczynnik skośności',    s.skewness.toFixed(4)],
    ].forEach(([label, value]) => {
        const tr = document.createElement('tr');
        tr.className = 'border-b border-gray-100';
        tr.innerHTML = `<td class="py-2 pr-4 text-gray-600">${label}</td><td class="py-2 font-semibold text-gray-900 text-right">${value}</td>`;
        tbody.appendChild(tr);
    });

    const labels = db.years.map(y => `LUX ${y}`).concat(db.years.map(y => `BGR ${y}`));
    new Chart(document.getElementById('s1BarChart'), {
        type: 'bar',
        data: {
            labels,
            datasets: [{
                label: 'Nadciśnienie (%)',
                data: values,
                backgroundColor: labels.map((_, i) => i < 5 ? 'rgba(79,70,229,0.75)' : 'rgba(16,185,129,0.75)'),
                borderWidth: 0
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
                y: { beginAtZero: false, grid: { color: '#e5e7eb' }, title: { display: true, text: 'Prevalencja (%)' } },
                x: { ticks: { font: { size: 9 } } }
            }
        }
    });

    new Chart(document.getElementById('s1BoxChart'), {
        type: 'bar',
        data: {
            labels: ['Nadciśnienie (%)'],
            datasets: [
                { label: `Min (${s.min})`, data: [[s.min, s.q1]], backgroundColor: 'rgba(148,163,184,0.3)', borderColor: '#64748b', borderWidth: 1, borderSkipped: false },
                { label: `Q1–Med (${s.q1.toFixed(1)}–${s.median.toFixed(1)})`, data: [[s.q1, s.median]], backgroundColor: 'rgba(14,165,233,0.5)', borderColor: '#0ea5e9', borderWidth: 1, borderSkipped: false },
                { label: `Med–Q3 (${s.median.toFixed(1)}–${s.q3.toFixed(1)})`, data: [[s.median, s.q3]], backgroundColor: 'rgba(14,165,233,0.85)', borderColor: '#0ea5e9', borderWidth: 1, borderSkipped: false },
                { label: `Max (${s.max})`, data: [[s.q3, s.max]], backgroundColor: 'rgba(148,163,184,0.3)', borderColor: '#64748b', borderWidth: 1, borderSkipped: false }
            ]
        },
        options: {
            indexAxis: 'y',
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
                legend: { display: true, position: 'bottom', labels: { font: { size: 10 } } },
                tooltip: { callbacks: { label: ctx => `${ctx.dataset.label}: ${ctx.raw[0]}%–${ctx.raw[1]}%` } }
            },
            scales: {
                x: { grid: { color: '#e5e7eb' }, title: { display: true, text: 'Prevalencja (%)' } },
                y: { display: false }
            }
        }
    });
}

function initSection2() {
    const yearLabels = db.years.map(String);

    function makeLineChart(canvasId, dataset, yLabel) {
        new Chart(document.getElementById(canvasId), {
            type: 'line',
            data: {
                labels: yearLabels,
                datasets: [
                    { label: 'Luksemburg', data: dataset.luxembourg, borderColor: 'rgba(79,70,229,1)', backgroundColor: 'rgba(79,70,229,0.1)', fill: true, tension: 0.3, pointRadius: 4 },
                    { label: 'Bułgaria',   data: dataset.bulgaria,   borderColor: 'rgba(16,185,129,1)', backgroundColor: 'rgba(16,185,129,0.1)', fill: true, tension: 0.3, pointRadius: 4 }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { position: 'bottom' } },
                scales: { y: { beginAtZero: false, grid: { color: '#e5e7eb' }, title: { display: true, text: yLabel } } }
            }
        });
    }

    makeLineChart('s2HypChart', db.hypertension, 'Prevalencja (%)');
    makeLineChart('s2DiaChart', db.diabetes, 'Prevalencja (%)');

    const allGdp = [...db.gdp.luxembourg, ...db.gdp.bulgaria];
    const allHyp = [...db.hypertension.luxembourg, ...db.hypertension.bulgaria];
    const allDia = [...db.diabetes.luxembourg, ...db.diabetes.bulgaria];

    new Chart(document.getElementById('s2ScatterChart'), {
        type: 'scatter',
        data: {
            datasets: [
                { label: 'Nadciśnienie', data: allGdp.map((x, i) => ({ x, y: allHyp[i] })), backgroundColor: 'rgba(239,68,68,0.7)', pointRadius: 6, pointHoverRadius: 8 },
                { label: 'Cukrzyca',    data: allGdp.map((x, i) => ({ x, y: allDia[i] })), backgroundColor: 'rgba(14,165,233,0.7)', pointRadius: 6, pointHoverRadius: 8 }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { position: 'bottom' } },
            scales: {
                x: { title: { display: true, text: 'PKB per capita (USD)' }, grid: { color: '#e5e7eb' } },
                y: { title: { display: true, text: 'Prevalencja (%)' }, grid: { color: '#e5e7eb' } }
            }
        }
    });

    const hypReg = stat.ols(allGdp, allHyp);
    const hypR2  = stat.rSquared(allGdp, allHyp);
    const diaReg = stat.ols(allGdp, allDia);
    const diaR2  = stat.rSquared(allGdp, allDia);

    document.getElementById('s2-regression-content').innerHTML = `
        <div class="bg-white rounded p-3 border border-blue-100">
            <p class="font-semibold text-gray-800 mb-1">Nadciśnienie</p>
            <p class="font-mono text-sm">Ŷ = ${hypReg.beta0.toFixed(2)} + (${hypReg.beta1.toFixed(6)}) · PKB</p>
            <p class="mt-1 text-sm">R² = <strong>${hypR2.toFixed(4)}</strong> — model wyjaśnia ${(hypR2*100).toFixed(1)}% zmienności</p>
            <p class="text-xs text-gray-500 mt-1">β₀ = ${hypReg.beta0.toFixed(4)}, β₁ = ${hypReg.beta1.toFixed(6)}</p>
        </div>
        <div class="bg-white rounded p-3 border border-blue-100">
            <p class="font-semibold text-gray-800 mb-1">Cukrzyca</p>
            <p class="font-mono text-sm">Ŷ = ${diaReg.beta0.toFixed(2)} + ${diaReg.beta1.toFixed(6)} · PKB</p>
            <p class="mt-1 text-sm">R² = <strong>${diaR2.toFixed(4)}</strong> — model wyjaśnia ${(diaR2*100).toFixed(1)}% zmienności</p>
            <p class="text-xs text-gray-500 mt-1">β₀ = ${diaReg.beta0.toFixed(4)}, β₁ = ${diaReg.beta1.toFixed(6)}</p>
        </div>
    `;
}

function initSection3() {
    const series = db.hypertension.luxembourg;
    const timeIndex = [1, 2, 3, 4, 5];
    const yearLabels = db.years.map(String);
    const { beta0: a, beta1: b } = stat.ols(timeIndex, series);
    const fitted = timeIndex.map(t => a + b * t);
    const r2 = stat.rSquared(timeIndex, series);
    const rate = stat.geoMeanRate(series);

    document.getElementById('s3-stats').innerHTML = `
        <div class="bg-gray-50 p-3 rounded border border-gray-200">
            <p class="text-xs font-semibold text-gray-500 uppercase mb-1">Model trendu liniowego</p>
            <p class="font-mono text-base text-gray-900">Ŷ = ${a.toFixed(2)} + (${b.toFixed(2)}) · t</p>
            <p class="text-xs text-gray-500 mt-1">t = 1 (rok 2000) … t = 5 (rok 2019)</p>
        </div>
        <div class="bg-gray-50 p-3 rounded border border-gray-200">
            <p class="text-xs font-semibold text-gray-500 uppercase mb-1">Współczynnik determinacji</p>
            <p class="font-semibold text-gray-900">R² = ${r2.toFixed(4)}</p>
            <p class="text-xs text-gray-500">Model wyjaśnia ${(r2*100).toFixed(1)}% zmienności szeregu</p>
        </div>
        <div class="bg-gray-50 p-3 rounded border border-gray-200">
            <p class="text-xs font-semibold text-gray-500 uppercase mb-1">Średnie tempo zmian (co 5 lat)</p>
            <p class="font-semibold text-gray-900">${(rate*100).toFixed(2)}%</p>
            <p class="text-xs text-gray-500">Średni spadek o ${((1-rate)*100).toFixed(2)}% co 5 lat</p>
        </div>
        <div class="bg-gray-50 p-3 rounded border border-gray-200">
            <p class="text-xs font-semibold text-gray-500 uppercase mb-1">Interpretacja parametrów</p>
            <p class="text-gray-700 text-sm">β₁ = ${b.toFixed(2)}: prevalencja zmienia się o ${b.toFixed(2)} p.p. na każde 5 lat</p>
            <p class="text-xs text-gray-500 mt-1 italic">Ocena istotności (p-value) — w programie Gretl</p>
        </div>
    `;

    new Chart(document.getElementById('s3Chart'), {
        type: 'line',
        data: {
            labels: yearLabels,
            datasets: [
                { label: 'Wartości rzeczywiste', data: series, borderColor: 'rgba(79,70,229,1)', backgroundColor: 'rgba(79,70,229,0.1)', fill: true, tension: 0.3, pointRadius: 5 },
                { label: `Trend: Ŷ = ${a.toFixed(1)} + (${b.toFixed(1)})·t`, data: fitted, borderColor: 'rgba(239,68,68,0.85)', borderDash: [6,4], borderWidth: 2, backgroundColor: 'transparent', fill: false, pointRadius: 0, tension: 0 }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { position: 'bottom' }, title: { display: true, text: 'Nadciśnienie — Luksemburg (2000–2019)' } },
            scales: { y: { beginAtZero: false, grid: { color: '#e5e7eb' }, title: { display: true, text: 'Prevalencja (%)' } } }
        }
    });
}

function populateTable() {
    const tbody = document.querySelector('#dataTable tbody');
    db.years.forEach((year, i) => {
        [
            { key: 'luxembourg', label: 'Luksemburg' },
            { key: 'bulgaria',   label: 'Bułgaria' }
        ].forEach(({ key, label }) => {
            const tr = document.createElement('tr');
            tr.innerHTML = `
                <td class="whitespace-nowrap py-4 pl-4 pr-3 text-sm font-medium text-gray-900">${year}</td>
                <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">${label}</td>
                <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500 text-right">${db.gdp[key][i].toLocaleString()}</td>
                <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500 text-right">${db.hypertension[key][i].toFixed(1)}</td>
                <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500 text-right">${db.diabetes[key][i].toFixed(2)}</td>
            `;
            tbody.appendChild(tr);
        });
    });
}

function copyTable() {
    const table = document.getElementById('dataTable');
    let range, selection;
    if (document.createRange && window.getSelection) {
        range = document.createRange();
        selection = window.getSelection();
        selection.removeAllRanges();
        try { range.selectNodeContents(table); selection.addRange(range); }
        catch (e) { range.selectNode(table); selection.addRange(range); }
        document.execCommand('copy');
    }
    selection.removeAllRanges();
    const feedback = document.getElementById('copy-feedback');
    feedback.classList.remove('hidden');
    setTimeout(() => feedback.classList.add('hidden'), 3000);
}
```

- [ ] **Step 2: Verify in browser console**

Open `index.html`. Open DevTools → Console. Run:
```javascript
stat.mean(db.hypertension.luxembourg)   // expected: ~36.24
stat.geoMeanRate(db.hypertension.luxembourg)  // expected: ~0.9638 (declining)
stat.ols([1,2,3,4,5], db.hypertension.luxembourg)  // expected: { beta0: ~43.1, beta1: ~-2.47 }
```
Page should show no errors (sections will be empty until HTML is added).

- [ ] **Step 3: Commit**
```bash
git add index.html
git commit -m "feat: replace data store with real WHO/World Bank data, add statistical utilities"
```

---

### Task 2: Replace HTML structure with 3-section layout

**Files:**
- Modify: `index.html` — replace `<style>` block, `<nav>` intro stays, replace entire `<main>` block

- [ ] **Step 1: Replace the `<style>` block** (between `<style>` and `</style>`) with:

```css
.chart-container {
    position: relative;
    width: 100%;
    max-width: 800px;
    margin-left: auto;
    margin-right: auto;
    height: 50vh;
    max-height: 400px;
    min-height: 200px;
}
body { background-color: #f3f4f6; color: #1f2937; }
.card { background-color: white; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -1px rgba(0,0,0,0.06); padding: 1.5rem; margin-bottom: 2rem; }
```

- [ ] **Step 2: Add sticky jump-nav directly before `<main>`**

Insert this HTML between the closing `</nav>` of the top navigation bar and the opening `<main>` tag:

```html
<!-- Sticky jump nav -->
<nav class="fixed top-20 right-4 z-40 flex-col space-y-1 hidden sm:flex">
    <a href="#s1" class="text-xs bg-white shadow border border-gray-200 rounded px-3 py-1.5 text-sky-700 hover:bg-sky-50 font-medium">§1 Jedna zmienna</a>
    <a href="#s2" class="text-xs bg-white shadow border border-gray-200 rounded px-3 py-1.5 text-sky-700 hover:bg-sky-50 font-medium">§2 Dwie zmienne</a>
    <a href="#s3" class="text-xs bg-white shadow border border-gray-200 rounded px-3 py-1.5 text-sky-700 hover:bg-sky-50 font-medium">§3 Szereg czasowy</a>
</nav>
```

- [ ] **Step 3: Replace the entire `<main>` block** with:

```html
<main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">

    <!-- Intro -->
    <div class="text-center mb-10">
        <h1 class="text-3xl font-extrabold text-gray-900 sm:text-4xl">
            Wpływ Statusu Ekonomicznego na Choroby Cywilizacyjne w UE
        </h1>
        <p class="mt-3 max-w-2xl mx-auto text-xl text-gray-500 sm:mt-4">
            Porównanie Luksemburga (najbogatszy kraj UE) i Bułgarii (najbiedniejszy kraj UE) pod kątem prevalencji nadciśnienia i cukrzycy w latach 2000–2019.
        </p>
        <p class="mt-2 text-sm text-gray-400">Źródła: WHO Global Health Observatory &amp; Bank Światowy</p>
    </div>

    <!-- SECTION 1 -->
    <section id="s1" class="card mb-10">
        <h2 class="text-2xl font-bold text-gray-800 mb-1 border-b pb-2">
            <span class="text-sky-600 mr-2">§1</span>Analiza jednej zmiennej
        </h2>
        <p class="text-sm text-gray-500 mb-6">Zmienna: Nadciśnienie tętnicze — prevalencja (%) | Luksemburg + Bułgaria, 2000–2019 (n=10)</p>
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
            <div>
                <h3 class="font-semibold text-gray-700 mb-3">Parametry opisowe</h3>
                <table class="w-full text-sm border-collapse" id="s1-stats-table"><tbody></tbody></table>
            </div>
            <div class="space-y-6">
                <div>
                    <p class="text-xs font-semibold text-gray-500 uppercase mb-1">Histogram (rozkład wartości)</p>
                    <div class="chart-container" style="height:200px;max-height:200px;">
                        <canvas id="s1BarChart"></canvas>
                    </div>
                </div>
                <div>
                    <p class="text-xs font-semibold text-gray-500 uppercase mb-1">Wykres pudełkowy</p>
                    <div class="chart-container" style="height:130px;max-height:130px;">
                        <canvas id="s1BoxChart"></canvas>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <!-- SECTION 2 -->
    <section id="s2" class="card mb-10">
        <h2 class="text-2xl font-bold text-gray-800 mb-1 border-b pb-2">
            <span class="text-sky-600 mr-2">§2</span>Analiza dwóch zmiennych
        </h2>
        <p class="text-sm text-gray-500 mb-6">Zależność między PKB per capita (USD) a prevalencją chorób</p>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-8">
            <div>
                <p class="text-sm font-semibold text-gray-700 mb-2 text-center">Nadciśnienie tętnicze (%)</p>
                <div class="chart-container" style="height:250px;max-height:250px;">
                    <canvas id="s2HypChart"></canvas>
                </div>
            </div>
            <div>
                <p class="text-sm font-semibold text-gray-700 mb-2 text-center">Cukrzyca (%)</p>
                <div class="chart-container" style="height:250px;max-height:250px;">
                    <canvas id="s2DiaChart"></canvas>
                </div>
            </div>
        </div>

        <div class="mb-8">
            <h3 class="font-semibold text-gray-700 mb-2">Korelogram: PKB per capita vs prevalencja chorób</h3>
            <div class="chart-container" style="height:300px;max-height:300px;">
                <canvas id="s2ScatterChart"></canvas>
            </div>
        </div>

        <div class="bg-blue-50 border border-blue-100 rounded-lg p-5">
            <h3 class="font-bold text-blue-800 mb-3">Model regresji liniowej: Ŷ = β₀ + β₁ · PKB</h3>
            <div id="s2-regression-content" class="grid grid-cols-1 sm:grid-cols-2 gap-4 text-sm"></div>
            <p class="text-xs text-blue-600 mt-3 italic">* Ocena istotności parametrów (p-value) wykonywana w programie Gretl na pełnym zbiorze danych.</p>
        </div>
    </section>

    <!-- SECTION 3 -->
    <section id="s3" class="card mb-10">
        <h2 class="text-2xl font-bold text-gray-800 mb-1 border-b pb-2">
            <span class="text-sky-600 mr-2">§3</span>Analiza szeregu czasowego
        </h2>
        <p class="text-sm text-gray-500 mb-6">Zmienna: Nadciśnienie tętnicze — Luksemburg, 2000–2019</p>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
            <div>
                <h3 class="font-semibold text-gray-700 mb-3">Parametry dynamiki</h3>
                <div id="s3-stats" class="space-y-3 text-sm"></div>
            </div>
            <div>
                <div class="chart-container" style="height:280px;max-height:280px;">
                    <canvas id="s3Chart"></canvas>
                </div>
            </div>
        </div>

        <h3 class="font-semibold text-gray-700 mb-2 mt-4">Dane surowe (do Excela/Gretl)</h3>
        <div class="flex justify-end mb-2">
            <button onclick="copyTable()" class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-slate-800 hover:bg-slate-900">
                <i class="fa-solid fa-copy mr-2"></i> Kopiuj do schowka
            </button>
        </div>
        <div class="overflow-x-auto mb-4">
            <table id="dataTable" class="min-w-full divide-y divide-gray-300 bg-white rounded-lg overflow-hidden shadow-sm">
                <thead class="bg-gray-100">
                    <tr>
                        <th class="py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900">Rok</th>
                        <th class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Kraj</th>
                        <th class="px-3 py-3.5 text-right text-sm font-semibold text-gray-900">PKB per capita (USD)</th>
                        <th class="px-3 py-3.5 text-right text-sm font-semibold text-gray-900">Nadciśnienie (%)</th>
                        <th class="px-3 py-3.5 text-right text-sm font-semibold text-gray-900">Cukrzyca (%)</th>
                    </tr>
                </thead>
                <tbody class="divide-y divide-gray-200"></tbody>
            </table>
        </div>
        <p id="copy-feedback" class="text-green-600 text-sm mt-2 hidden font-medium text-right">Skopiowano pomyślnie!</p>

        <div class="bg-sky-50 border border-sky-100 rounded-lg p-4 mt-6">
            <h3 class="font-bold text-gray-800 mb-3"><i class="fa-solid fa-link text-sky-600 mr-2"></i>Źródła Danych</h3>
            <ul class="space-y-2 text-sm text-gray-700">
                <li class="flex items-start">
                    <i class="fa-solid fa-heart-pulse text-gray-400 mt-1 mr-3"></i>
                    <div>
                        <strong>WHO Global Health Observatory — Nadciśnienie:</strong><br>
                        Wskaźnik: NCD_HYP_PREVALENCE_A (age-standardized, adults 30–79, both sexes)<br>
                        <a href="https://www.who.int/data/gho/data/indicators/indicator-details/GHO/prevalence-of-hypertension-among-adults-aged-30-79-years" target="_blank" rel="noopener noreferrer" class="text-sky-600 hover:text-sky-800 underline">who.int/data/gho</a>
                    </div>
                </li>
                <li class="flex items-start">
                    <i class="fa-solid fa-syringe text-gray-400 mt-1 mr-3"></i>
                    <div>
                        <strong>WHO Global Health Observatory — Cukrzyca:</strong><br>
                        Wskaźnik: NCD_DIABETES_PREVALENCE_AGESTD (age-standardized, both sexes)<br>
                        <a href="https://www.who.int/data/gho/data/indicators/indicator-details/GHO/prevalence-of-diabetes-age-standardized" target="_blank" rel="noopener noreferrer" class="text-sky-600 hover:text-sky-800 underline">who.int/data/gho</a>
                    </div>
                </li>
                <li class="flex items-start">
                    <i class="fa-solid fa-coins text-gray-400 mt-1 mr-3"></i>
                    <div>
                        <strong>Bank Światowy — PKB per capita:</strong><br>
                        Wskaźnik: NY.GDP.PCAP.CD (current USD)<br>
                        <a href="https://data.worldbank.org/indicator/NY.GDP.PCAP.CD?locations=LU-BG" target="_blank" rel="noopener noreferrer" class="text-sky-600 hover:text-sky-800 underline">data.worldbank.org</a>
                    </div>
                </li>
            </ul>
        </div>
    </section>

</main>
```

- [ ] **Step 4: Verify in browser**

Open `index.html`. You should see:
- 3 numbered sections (§1, §2, §3) with correct headings
- §1 stats table filled, two charts rendered (bar + box plot)
- §2 two side-by-side line charts + scatter plot + regression card with computed values
- §3 stats panel + trend chart + data table with 10 rows + data sources
- Sticky nav in top-right with 3 jump links
- No toggle buttons anywhere
- No console errors

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "feat: replace HTML with 3-section layout for Luxembourg vs Bulgaria analysis"
```

---

### Task 3: Update page title and nav bar text

**Files:**
- Modify: `index.html` — `<title>` tag and nav bar `<span>` text

- [ ] **Step 1: Update the `<title>` tag**

Find:
```html
<title>Analiza: Choroby Cywilizacyjne a Status Ekonomiczny</title>
```
Replace with:
```html
<title>Analiza: Choroby Cywilizacyjne w UE — Luksemburg vs Bułgaria</title>
```

- [ ] **Step 2: Update the footer copyright line**

Find:
```html
&copy; 2026 Praca przygotowana przez: Jakub Skwierawski, Łukasz Reszczyński, Krystian Turzyński, Przemysław Choma, Błażej Choma.
```
This line is correct — leave it unchanged.

- [ ] **Step 3: Verify and commit**

Open `index.html`. Check browser tab shows new title.
```bash
git add index.html
git commit -m "feat: update title to reflect Luxembourg vs Bulgaria comparison"
```
