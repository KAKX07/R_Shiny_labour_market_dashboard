# European Labour Market Dashboard
### An OECD Data Analysis — Block 7 Assignment

---

#### Use of AI
Claude and Gemini was used throughout this project as a research, thinking and coding assistant. Concretely, AI helped with :

**Code cleaning and debugging** — reviewing and improving the data_prep.R script, fixing encoding issues, correcting filter logic in the Shiny server
<br>
**Technical guidance** — explaining Shiny concepts (reactivity, pickerInput, renderPlot)
<br>
**Code readability** - at the end my script looked like a mess, to make it more readable I used Gemini to reorder my code properly.


All analytical decisions, annotation, data choices and final code were reviewed and validated manually. AI was used as a tool to accelerate the process, not to replace understanding.

## Context

This dashboard was built as part of a data visualization course assignment. The scenario is the following : **you are a data analyst at the OECD**, and the Secretary General has asked your department to prepare a dashboard for a high-level meeting with European ministries on the evolution of the labour market.

The dashboard focuses exclusively on **European OECD member countries** — a deliberate choice driven by data availability and analytical depth. A coherent, deep analysis of Europe is more valuable than a superficial worldwide overview.

---

## Data Sources

All data comes from official OECD sources.

| Dataset | Source | Coverage |
|---|---|---|
| Employment by age group | OECD Labour Force Statistics | Europe, 2000–2024 |
| Unemployment by age group | OECD Labour Force Statistics | Europe, 2000–2024 |
| Average annual wages | OECD Average Annual Wages | Europe, 2000–2024 |
| AI adoption by businesses | OECD ICT Access and Usage | Europe, 2017–2024 |
| Employment by sector | OECD National Accounts (NACE Rev.2) | Europe, 2000–2024 |
| Skills shortage indicators | OECD Skills for Jobs Database | Europe, **2022 edition** |

> **Note on the Skills for Jobs data** : this dataset reflects the 2022 edition — the latest publicly available version at the time of analysis.

---

## Dashboard Structure

The dashboard is organized into **4 analytical pages**, each with its own filters.

###  Page 1 — Overview
A macro-level snapshot of the European labour market. Covers employment levels, unemployment rates, wage evolution and employment structure by age group.

**Key indicators :**
- Total employment and unemployment rate (latest selected year)
- Average annual wage in USD PPP
- Wage index evolution (base 100 = 2010) — allows cross-country comparison by neutralizing absolute wage level differences
- Employment growth year-over-year (age 25–54)
- Employment share by age group (15–24, 25–54, 55–64)

###  Page 2 — AI & Labour
Explores the relationship between AI adoption by businesses and labour market outcomes — wages and employment growth.

**Key indicators :**
- % of businesses using AI by country over time
- AI adoption vs average wage (scatter — one dot per country)
- AI adoption vs employment growth (scatter — one dot per country)

The trend line on the scatter plots shows the direction of the relationship across countries.

###  Page 3 — Employment by Sector
Breaks down employment across 10 mutually exclusive NACE Rev.2 macro-sectors to avoid double counting.

**Key indicators :**
- Employment distribution by sector (latest selected year)
- Self-employment share over time — proxy for labour market flexibility or precarity
- Sector employment evolution over time

###  Page 4 — Skills
Maps skill shortages and surpluses across European countries using the OECD Skills for Jobs composite indicator.

**Reading the indicator :**
- **Positive value** → skill shortage (market lacks workers with this skill)
- **Negative value** → skill surplus (too many workers with this skill)
- **Close to 0** → balanced supply and demand

---

## How to Use the Dashboard

### Filters
Each page has its own independent filters — **country picker** and **year picker** — placed directly on the page. All filters work as Excel-style dropdowns with a search bar and Select All / Deselect All buttons.

Filters on one page do not affect other pages — this is intentional, as each page has a different analytical scope and data coverage.

### Navigation
Use the **left sidebar** to navigate between pages.

### Reading the charts
- **Line charts** — use the country color legend to identify each series
- **Bar charts** — sorted by value for easy ranking
- **Scatter plots** — each dot is a country, hover to identify it ; the dashed line shows the overall trend

---

## Technical Notes

The project is structured in two scripts :

```
data_prep.R   ←  data loading, cleaning, and indicator computation
app.R         ←  Shiny dashboard (UI + server)
```

Running `data_prep.R` generates `data_prepared.RData` which is loaded by `app.R` at startup. This separation keeps the app fast — all heavy data processing happens once, not on every user interaction.

**R packages used :**
- `shiny` + `shinydashboard` — dashboard framework
- `tidyverse` + `ggplot2` — data manipulation and visualization
- `shinyWidgets` — Excel-style picker inputs

---


