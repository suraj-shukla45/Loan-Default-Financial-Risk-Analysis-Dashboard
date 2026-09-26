#  Loan Default & Financial Risk Analysis Dashboard

A 3-page interactive Power BI dashboard analyzing loan applications, applicant demographics, employment, credit categories, and default behavior — built to surface which factors actually drive default risk in a loan portfolio.
### Dashboard Link : https://suraj-shukla45.github.io/Loan-Default-Financial-Risk-Analysis-Dashboard/
---

## Problem Statement

Financial institutions need to know **which applicant characteristics actually predict default risk**, not just how loan amounts are distributed. This project analyzes a 6-year loan portfolio (2013–2018) across employment type, credit category, age, education, marital status, and mortgage status to separate real risk signals from noise.

---

## Key Insight (the finding that matters)

**Employment type is the strongest predictor of default in this dataset — everything else is noise.**

| Employment Type | Default Rate |
|---|---:|
| Unemployed | 3.4% |
| Part-time | 3.0% |
| Self-employed | 2.9% |
| Full-time | 2.4% |

Unemployed applicants default at a rate **~42% higher (relative)** than full-time applicants. This is the only variable in the dataset with a meaningfully large spread.

**Recommendation:** Underwriting/pricing models should weight employment type explicitly as a risk factor. A flat interest rate or approval threshold across employment types is leaving risk unpriced.

**What did *not* predict risk (and why that matters):**
Age group, education level, credit category, and mortgage status all showed loan amounts and default behavior within a narrow band (roughly 1–2% variation) — statistically close to flat. Two possible explanations, and this is a limitation worth stating rather than hiding:
1. The underwriting process may already be neutral on these factors (a good sign), or
2. These fields carry weak signal in this particular dataset (it shows characteristics of a synthetic/benchmark dataset rather than raw production data).

Either way, the honest conclusion is: **don't build a story around variables that don't move the needle.** The employment-type signal is the one worth acting on.

---

## Report Structure

| Page | Focus |
|---|---|
| Page 1 | Loan Default & Overview — portfolio-level view, employment risk |
| Page 2 | Applicant Demographics & Financial Profile — credit, age, education, marital status |
| Page 3 | Financial Risk Metrics — year-over-year trends, income × employment breakdown |

---

## Page 1 – Loan Default & Overview

- **Loan Amount by Purpose:** Home (6,545M), Business (6,522M), Education (6,511M), Auto (6,501M), Other (6,498M) — purposes are within ~0.7% of each other; purpose is not a differentiator.
- **Default Rate by Employment Type:** the core insight above.
- **Average Loan Amount by Age Group:** Adults (127,901) → Teen (126,674) — flat.
- **Default Rate by Year (2013–2018):** stable in an 11.5%–11.75% band — no meaningful trend, no crisis year.

## Page 2 – Applicant Demographics & Financial Profile

- **Median Loan Amount by Credit Category:** Low (128,397) to High (127,149) — counter-intuitively flat, worth flagging rather than over-interpreting.
- **Loan Amount by Age Group × Marital Status:** donut view for cross-segment exploration.
- **Total Loan by Credit Score Bins (Adults):** Medium (4.6B) and High (4.5B) segments carry the bulk of loan volume — useful for portfolio concentration, not for risk.
- **Loans by Education Type:** Bachelor's (64,365) to PhD (63,537) — flat.

## Page 3 – Financial Risk Metrics

- **YOY Default Loan Change:** oscillates (+2.7 in 2015, −2.8 in 2017) with no sustained trend — consistent with the flat year-wise default rate on Page 1.
- **YOY Loan Amount Change:** similarly oscillating, no compounding growth or decline.
- **YTD Loan Amount by Credit Score Bins × Marital Status:** Sankey-style flow for segment-level drill-down.
- **Loan Amount by Income Bracket → Employment Type:** decomposition tree; Full-time consistently carries the largest share within every income bracket.

---

## Tools & Techniques

- **Power BI Desktop & Service**, **DAX**, **Power Query**
- Data cleaning: type correction, missing-value handling, category binning (age groups, credit score bins, income brackets)

---

## Data Model — Calculated Columns & DAX Measures

### Calculated Columns (base table: `Loan_default`)

Built directly on the fact table to bucket continuous fields into categories used across all three report pages:

| Calculated Column | Purpose |
|---|---|
| `Age Groups` | Buckets raw `Age` into Teen / Adult / Middle Age Adult / Senior Citizen |
| `Credit Score Bins` | Buckets raw `CreditScore` into Low / Medium / High / Very Low |
| `Income Bracket` | Buckets raw `Income` into Low / Medium / High Income |

Base table also carries: `Age`, `CreditScore`, `Income`, `DTIRatio` (aggregatable numeric fields), and categorical fields `Default`, `Education`, `EmploymentType`, `HasCoSigner`, `HasDependents`, `HasMortgage`, `Index`.

**Why columns, not measures:** these three values need to exist row-by-row on every applicant record — they're used as slicers, axis categories, and grouping keys across visuals. A measure only produces a single aggregated number in a visual's context; it can't act as a group-by/filter category the way a column can. Row-level classification always calls for a calculated column, not a measure.

### DAX Measures (organized into 3 measure tables for a clean model)

**Measure Table 1 — Core Portfolio Metrics**
| Measure | DAX Concepts Used |
|---|---|
| `Average Income by Employment type` | `CALCULATE`, `AVERAGE`, `ALLEXCEPT` |
| `Default Rate by Employment type` | `CALCULATE`, `COUNTROWS`, `DIVIDE`, `ALL`, `ALLEXCEPT`, `FILTER` |
| `Default Rate by Year` | `CALCULATE`, `COUNTROWS`, `DIVIDE`, `ALLEXCEPT`, `FILTER` |
| `Loan Amount by Purpose` | `SUMX`, `FILTER`, `NOT`, `ISBLANK` |

**Measure Table 2 — Demographic & Segment Metrics**
| Measure | DAX Concepts Used |
|---|---|
| `Loans by Education type` | `AVERAGE` |
| `Total Loan (Credit Bins)` | `AVERAGEX`, `VALUES` |
| `Total Loan (Middle Age Adults)` | `CALCULATE`, `SUM`, `FILTER` |

**Measure Table 3 — Time Intelligence & Risk Trend**
| Measure | DAX Concepts Used |
|---|---|
| `YOY Default Loans change` | `CALCULATE`, time-based filter over `Year` |
| `YOY Loan Amount Change` | `CALCULATE`, time-based filter over `Year` |
| `YTD Loan Amount` | `CALCULATE`, cumulative filter over `Year` |

**Why `ALLEXCEPT` over `ALL` in the rate/average measures:** `ALLEXCEPT` removes filters from the table except the ones explicitly kept (e.g. `EmploymentType`), so the calculation still respects the category being sliced while ignoring unrelated filters like page-level slicers. Using plain `ALL` would strip every filter, including the one the visual is grouping by — collapsing every category to the same portfolio-wide number instead of a per-category rate.

**Decomposition Tree (`SWITCH`):** used to let the Income Bracket → Employment Type breakdown on Page 3 branch dynamically based on which field the user expands next, rather than hardcoding one fixed hierarchy.

---

## Data Pipeline (Dataflow Architecture)

Instead of loading a flat file directly into Power BI Desktop, this project was built as a proper Dataflow-backed pipeline so the data source is centralized and refreshable independent of the report file:

1. **Standard Mode Gateway** installed and configured — required so Power BI Service can reach an on-premises SQL Server.
2. **Microsoft SQL Server** installed locally as the source database.
3. **Data imported into SQL Server** from the raw dataset.
4. **Dataflow created in Power BI Service**, pointing at the SQL Server tables — this is the reusable, centralized data layer (other reports/datasets could reuse the same dataflow instead of re-importing from SQL each time).
5. **Power BI Desktop connects to the Dataflow** (not directly to SQL) to build the report — decouples report development from the source connection.
6. Column definitions and dataset description documented before modeling.
7. Data types, profiling, and quality checks done in Power Query Editor (nulls, distributions, consistency).
8. Categorical bins (`Age Groups`, `Credit Score Bins`, `Income Bracket`) built as calculated columns.
9. DAX measures built (see below) and validated against manual/expected values for each metric.
10. Report designed across 3 pages with consistent theme and cross-filtering.
11. Published to Power BI Service with **scheduled refresh** on the dataflow and **incremental refresh** configured, so the report updates without a full manual reload each time.

**Why this matters:** a Dataflow + Gateway setup mirrors how BI is actually run in a company — a report built by loading a CSV directly into Desktop doesn't scale or refresh on its own. This structure is closer to a real production setup.

---

## Limitations & What I'd Do Differently

- The dataset shows very tight, narrow-range values across most dimensions, which limits how far the analysis can go beyond the employment-type finding. A dataset with more real-world variance (or actual production data) would let this go further.
- I would add a **correlation or feature-importance check** (even a simple logistic regression) before building visuals, to confirm which variables are actually worth dashboarding — rather than visualizing every available column.
- I would add a **confidence/sample-size note** per segment, since some age-group and marital-status combinations likely have small sample sizes that make their averages less reliable.
- Next iteration: add a **cohort view** (loan vintage vs default rate over time) to check if risk concentration is changing across origination years, not just aggregate year-over-year default rate.

---

## Skills Demonstrated

Power BI · DAX · Power Query · Data Cleaning · Data Transformation · Data Visualization · Financial Risk Analysis · Exploratory Data Analysis · Insight Prioritization

---

## Dashboard Screenshots

*(Add exported PNGs here, e.g. `assets/page1-overview.png`, and replace the lines below)*

![Loan Default & Overview](assets/page1-overview.png)
![Applicant Demographics & Financial Profile](assets/page2-demographics.png)
![Financial Risk Metrics](assets/page3-risk-metrics.png)
