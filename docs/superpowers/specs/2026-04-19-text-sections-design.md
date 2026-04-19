# Design: Intro & Conclusions Text Sections

**Date:** 2026-04-19
**Project:** Statystyka PANS — index.html
**Scope:** Add static intro block before §1 and conclusions block after §3

---

## Placement

- **Intro block:** between the title/subtitle `<div>` and the `<section id="s1">` tag
- **Conclusions block:** after the closing `</section>` of `#s3`, before `</main>`

---

## Intro Block

**Structure:** two paragraphs + hypothesis callout box. Uses existing `.card` class.

**Paragraph 1 — Research context:**
Akademickie uzasadnienie wyboru krajów (Luksemburg jako kraj o najwyższym PKB per capita w UE, Bułgaria jako kraj o najniższym PKB per capita w UE według danych Banku Światowego za rok 2019). Cel: zbadanie związku między poziomem zamożności a prevalencją chorób cywilizacyjnych w skali europejskiej.

**Paragraph 2 — Variables and methodology:**
Zmienna niezależna: PKB per capita w USD (Bank Światowy, wskaźnik NY.GDP.PCAP.CD). Zmienne zależne: prevalencja nadciśnienia tętniczego (WHO GHO, NCD_HYP_PREVALENCE_A, standaryzowana wiekowo, dorośli 30–79 lat) oraz cukrzycy (WHO GHO, NCD_DIABETES_PREVALENCE_AGESTD, standaryzowana wiekowo). Analiza obejmuje lata 2000–2019 (5 punktów pomiarowych co 5 lat, n=10 obserwacji na zmienną).

**Hypothesis callout box** (blue, `bg-blue-50 border-blue-100`):
> „Wyższy poziom zamożności (PKB per capita) jest ujemnie skorelowany z prevalencją nadciśnienia tętniczego i cukrzycy wśród dorosłych — kraje bogatsze wykazują niższe wskaźniki obu chorób cywilizacyjnych."

---

## Conclusions Block

**Structure:** two paragraphs + three stat cards. Uses existing `.card` class.

**Paragraph 1 — Two-variable analysis:**
Analiza regresji liniowej (OLS) wykazała istotny ujemny związek między PKB per capita a prevalencją obu chorób. Dla nadciśnienia tętniczego współczynnik determinacji R²=0.89 wskazuje, że PKB wyjaśnia ok. 89% zmienności wskaźnika chorobowości. Dla cukrzycy R²=0.76. Współczynnik kierunkowy β₁ dla nadciśnienia jest ujemny — wzrost PKB per capita o 1 USD wiąże się ze spadkiem prevalencji. Wyniki potwierdzają hipotezę o ujemnej korelacji bogactwa z poziomem obu chorób cywilizacyjnych.

**Paragraph 2 — Time series:**
Analiza szeregu czasowego prevalencji nadciśnienia w Luksemburgu (2000–2019) wykazała wyraźny trend malejący: liniowy model trendu Ŷ = 43.89 + (−2.55)·t osiągnął R²=0.95, a średnie tempo zmian wyniosło ok. −6.7% co 5 lat. Świadczy to o skuteczności systemu ochrony zdrowia w kraju wysokorozwiniętym. W przeciwieństwie do tego, Bułgaria odnotowała dynamiczny wzrost prevalencji cukrzycy (z 5.92% w 2000 r. do 9.69% w 2019 r.), co sugeruje, że niższy poziom PKB koreluje z ograniczonym dostępem do profilaktyki i leczenia chorób metabolicznych.

**Three stat cards** (side by side, `bg-gray-50 border border-gray-200 rounded p-3`):
- Card 1: `Nadciśnienie R² = 0.89` — "PKB wyjaśnia 89% zmienności prevalencji"
- Card 2: `Cukrzyca R² = 0.76` — "PKB wyjaśnia 76% zmienności prevalencji"
- Card 3: `Trend Luksemburg −6.7% / 5 lat` — "Średnie tempo spadku prevalencji nadciśnienia"

---

## Styling

- Both blocks use existing `.card` class
- Hypothesis box: `bg-blue-50 border border-blue-100 rounded-lg p-4`
- Stat cards: `bg-gray-50 border border-gray-200 rounded-lg p-4 text-center`
- Stat value: `text-2xl font-bold text-sky-700`
- Language: Polish throughout
- No new CSS classes needed
