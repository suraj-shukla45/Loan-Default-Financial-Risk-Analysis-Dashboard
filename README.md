# Loan Default & Financial Risk Analysis Dashboard (2013–2018)

### Dashboard Link : https://suraj-shukla45.github.io/Loan-Default-Financial-Risk-Analysis-Dashboard/

## Problem Statement

Financial institutions need to understand which applicant characteristics actually predict default risk, not just how loan amounts are distributed across the portfolio. This dashboard analyzes a 6-year loan portfolio (2013–2018) across employment type, credit category, age, education, marital status, and mortgage status, so patterns like which factors genuinely drive default — and which ones don't — can be identified easily.

Using the dashboard's 3 pages, loan and default behavior can be viewed from many angles: by purpose, by employment type, by age group, by credit category, by education, and year-over-year, which helps separate the one meaningful risk signal from variables that carry no real signal.

## Tools Used

- Power BI Desktop
- Power BI Service
- DAX
- Power Query
- Microsoft SQL Server
- On-premises Data Gateway (Standard Mode)

## Report Structure

The report has **3 pages**.

| Page | Content |
|---|---|
| Page 1 | Loan Default & Overview — Loan Amount by Purpose, Average Income by Employment Type, Default Rate by Employment Type, Average Loan Amount by Age Group, Default Rate by Year |
| Page 2 | Applicant Demographics & Financial Profile — Median Loan Amount by Credit Category, Loan Amount by Age Group & Marital Status (donut), Total Loan by Credit Score Bins (Adults), Total Loan by Mortgage Status (Middle Age Adults), Loans by Education Type |
| Page 3 | Financial Risk Metrics — YOY Default Loan Change, YOY Loan Amount Change, YTD Loan Amount by Credit Score Bins & Marital Status (Sankey), Loan Amount by Income Bracket → Employment Type (decomposition tree) |

## Steps followed

- Step 1 : Install and configure the **Standard Mode Gateway**, required so Power BI Service can reach an on-premises SQL Server.
- Step 2 : Install **Microsoft SQL Server** locally as the source database.
- Step 3 : Import the raw dataset into SQL Server.
- Step 4 : Create a **Dataflow in Power BI Service**, pointing at the SQL Server tables, as the reusable centralized data layer.
- Step 5 : Connect **Power BI Desktop to the Dataflow** (not directly to SQL) to build the report.
- Step 6 : Document column definitions and dataset description before modeling.
- Step 7 : Open Power Query Editor and check column distribution, column quality and column profile (based on entire dataset) to find errors and empty values.
- Step 8 : Clean the data and correct data types where required.
- Step 9 : Create calculated columns for **Age Groups**, **Credit Score Bins** and **Income Bracket** to bucket continuous fields into categories.
- Step 10 : Build DAX measures for loan amount, average/median calculations, default rate, and YOY/YTD time-intelligence metrics, organized into 3 measure tables:
  - Measure Table 1 (Core Portfolio Metrics): Average Income by Employment type, Default Rate by Employment type, Default Rate by Year, Loan Amount by Purpose
  - Measure Table 2 (Demographic & Segment Metrics): Loans by Education type, Total Loan (Credit Bins), Total Loan (Middle Age Adults)
  - Measure Table 3 (Time Intelligence & Risk Trend): YOY Default Loans change, YOY Loan Amount Change, YTD Loan Amount
- Step 11 : Add a **Decomposition Tree** on Page 3 so the Income Bracket → Employment Type breakdown branches dynamically based on which field the user expands next.
- Step 12 : Validate each measure against manual/expected values.
- Step 13 : Design the report across 3 pages with consistent theme, formatting and cross-filtering.
- Step 14 : Publish the report to Power BI Service with **scheduled refresh** on the dataflow and **incremental refresh** configured.

<!-- DAX measures used: -->
<!-- Default Rate by Employment type = DIVIDE(CALCULATE(COUNTROWS(...), ALLEXCEPT(...)), CALCULATE(COUNTROWS(...), ALL(...))) -->
<!-- Loan Amount by Purpose = SUMX(FILTER(Loan_default, NOT(ISBLANK(LoanAmount))), LoanAmount) -->

# Snapshot of Dashboard

## Page 1
<img width="1512" height="841" alt="Image" src="https://github.com/user-attachments/assets/27267d5a-e713-48b2-9c48-d90c20fac1e6" />

## Page 2
<img width="1507" height="826" alt="Image" src="https://github.com/user-attachments/assets/0dcff296-7958-4132-a191-75a316a99937" />

## Page 3
<img width="1497" height="831" alt="Image" src="https://github.com/user-attachments/assets/aeb12e06-3ade-471a-b9f8-5e8dd429931e" />

# Insights

## Page 1 – Loan Default & Overview

### Chart 1 : Loan Amount by Purpose

| Purpose | Loan Amount |
|---|---:|
| Home | 6,545M |
| Business | 6,522M |
| Education | 6,511M |
| Auto | 6,501M |
| Other | 6,498M |

        thus, all purposes stay within about 0.7% of each other, so loan purpose is not a meaningful differentiator of loan amount.

### Chart 2 : Average Income by Employment Type

| Employment Type | Average Income |
|---|---:|
| Full-time | 82,890 |
| Self-employed | 82,447 |
| Part-time | 82,389 |
| Unemployed | 82,272 |

        thus, average income is nearly identical across employment types, so income alone does not separate these groups.

### Chart 3 : Default Rate by Employment Type

| Employment Type | Default Rate |
|---|---:|
| Unemployed | 3.4% |
| Part-time | 3.0% |
| Self-employed | 2.9% |
| Full-time | 2.4% |

        thus, Unemployed applicants default at a rate ~42% higher (relative) than Full-time applicants — this is the only variable in the dataset with a meaningfully large spread, and the one worth acting on for underwriting/pricing decisions.

### Chart 4 : Average Loan Amount by Age Group

| Age Group | Average Loan Amount |
|---|---:|
| Adults | 127,901.01 |
| Middle Age Adults | 127,459.65 |
| Senior Citizens | 127,355.19 |
| Teen | 126,673.94 |

        thus, average loan amount is flat across age groups (well under 1% spread), so age group is not a driver of loan size.

### Chart 5 : Default Rate by Year

| Year | Default Rate |
|---|---:|
| 2013 | 11.62% |
| 2014 | 11.50% |
| 2015 | 11.70% |
| 2016 | 11.75% |
| 2017 | 11.50% |
| 2018 | 11.60% |

        thus, the year-wise default rate stays in a narrow 11.5%–11.75% band across 2013–2018, with no meaningful trend or crisis year.

## Page 2 – Applicant Demographics & Financial Profile

### Chart 1 : Median Loan Amount by Credit Category

| Credit Category | Median Loan Amount |
|---|---:|
| Low | 128,397 |
| Medium | 127,765 |
| Very Low | 127,515 |
| High | 127,149 |

        thus, median loan amount is counter-intuitively flat across credit categories — worth flagging rather than over-interpreting as a risk signal.

### Chart 2 : Average Loan Amount by Age Group & Marital Status

A donut chart compares Adults, Middle Age Adults, Senior Citizens and Teens against Single, Married and Divorced status.

        thus, this view is useful for cross-segment exploration but does not surface a standout combination.

### Chart 3 : Total Loan by Credit Score Bins (Adults)

| Credit Score Bin | Total Loan |
|---|---:|
| Medium | 4.6bn |
| High | 4.5bn |
| Very Low | 2.3bn |
| Low | 1.1bn |

        thus, Medium and High credit segments carry the bulk of loan volume among adults — useful for portfolio concentration, not for risk.

### Chart 4 : Total Loan by Mortgage Status (Middle Age Adults)

| Has Mortgage | Total Loan |
|---|---:|
| False | 3.1bn |
| True | 3.1bn |

        thus, mortgage status makes almost no difference to total loan amount for this age group.

### Chart 5 : Loans by Education Type

| Education | Average Loan Amount |
|---|---:|
| Bachelor's | 64,365 |
| High School | 63,903 |
| Master's | 63,541 |
| PhD | 63,537 |

        thus, education level shows only a small spread (~1.3%) — not a strong differentiator of loan amount.

## Page 3 – Financial Risk Metrics

### Chart 1 : YOY Default Loans Change by Year

| Year | YOY Default Loan Change |
|---|---:|
| 2013 | 0.0 |
| 2014 | -2.2 |
| 2015 | 2.7 |
| 2016 | 0.8 |
| 2017 | -2.8 |
| 2018 | 1.9 |

        thus, the largest positive change is in 2015 (+2.7) and the largest negative change is in 2017 (-2.8), with no sustained upward or downward trend.

### Chart 2 : YOY Loan Amount Change by Year

| Year | YOY Loan Amount Change |
|---|---:|
| 2013 | 0.0 |
| 2014 | -1.8 |
| 2015 | 1.3 |
| 2016 | 0.0 |
| 2017 | -1.1 |
| 2018 | 1.7 |

        thus, loan amount also oscillates year to year with no compounding growth or decline.

### Chart 3 : YTD Loan Amount by Credit Score Bins & Marital Status

A Sankey-style flow moves Medium → High → Very Low → Low credit bins, split by Divorced, Married and Single.

        thus, this view is best used for segment-level drill-down rather than a single headline number.

### Chart 4 : Loan Amount by Income Bracket → Employment Type

A decomposition tree breaks down Sum of Loan Amount by Income Bracket (High/Medium/Low) then Employment Type (Full-time/Part-time/Self-employed).

        thus, Full-time consistently carries the largest loan-amount share within every income bracket.

## Interactivity

- The decomposition tree on Page 3 lets the user expand Income Bracket → Employment Type dynamically to explore any branch.
- Cross-filtering is enabled across visuals on each page.

All values will change if different slicers/filters are applied.

---

## Limitations & What I'd Do Differently

- The dataset shows very tight, narrow-range values across most dimensions, which limits how far the analysis can go beyond the employment-type finding.
- I would add a correlation or feature-importance check before building visuals, to confirm which variables are worth dashboarding.
- I would add a confidence/sample-size note per segment, since some age-group and marital-status combinations likely have small sample sizes.
- Next iteration: add a cohort view (loan vintage vs default rate over time) rather than just aggregate year-over-year default rate.

## Skills Demonstrated

Power BI · DAX · Power Query · SQL Server · Dataflows · Data Cleaning · Data Transformation · Data Visualization · Financial Risk Analysis · Exploratory Data Analysis · Insight Prioritization
