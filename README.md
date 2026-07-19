# Lyme Disease Risk Pattern Analysis

## Research Question

Does official Lyme disease case data reliably reflect actual disease risk or does it under-represent cases in ways that vary systematically by geography, reporting methodology, and rurality?

This project combines CDC tick surveillance data with CDC Lyme disease case data, U.S. Census population estimates, and NCHS urban-rural classifications to test two specific, measurable hypotheses about how well official surveillance data reflects true disease risk.

---

## Data Sources

| Dataset | Source | Description |
|---|---|---|
| Lyme Disease Case Data | [CDC Lyme Disease Data and Surveillance](https://www.cdc.gov/lyme/data-research/facts-stats/) | County-level reported case counts, 2001-2023 |
| Ixodes Tick Surveillance | [CDC Tick Surveillance Data Sets](https://www.cdc.gov/ticks/data-research/facts-stats/tick-surveillance-data-sets.html) | County-level establishment status for *Ixodes scapularis* and *Ixodes pacificus*, the two tick species responsible for the vast majority of U.S. Lyme disease cases |
| Population Estimates | [U.S. Census Bureau](https://www.census.gov/data/tables/time-series/demo/popest/2020s-counties-total.html) | County-level population estimates, 2020-2025 |
| Urban-Rural Classification | [NCHS Urban-Rural Classification Scheme](https://www.cdc.gov/nchs/data_access/urban_rural.htm) | 6-level county classification, from large central metro to noncore (rural) |

---

## Methodology

### Database Design
A relational schema was built in Oracle SQL, linking five tables through a standardized 5-digit FIPS county code:
- **COUNTIES** — county and state names (anchor table)
- **CASE_DATA** — annual case counts, 2001-2023, plus state incidence classification
- **TICK_STATUS** — tick establishment status by species
- **POPULATION** — population estimates, 2020-2023
- **URBAN_RURAL** — urban-rural classification code

### Data Cleaning
Each source required specific cleaning before it could be joined:
- Character encoding fixes (Windows-1252) for special characters in state and municipality names
- Construction of standardized FIPS codes from separate state/county code columns
- Removal of state-level summary rows before matching at the county level
- Manual correction of one FIPS code (Shannon County, SD → Oglala Lakota County, FIPS 46113 → 46102), reflecting the county's 2015 renaming
- Exclusion of Connecticut from the tick-status join: Connecticut's tick surveillance data is organized by 9 "planning regions," while its case data uses 8 traditional counties

Match rates achieved: 3,110 of 3,153 counties matched between case and tick data (98.6%); 3,143 of 3,144 matched for population and urban-rural data (99.97%).

### Analysis Approach
Case rates were calculated per 100,000 residents (rather than raw counts) to allow fair comparison across counties of different population sizes. Tick surveillance data reflects cumulative, current status rather than a single collection year; the four most recent case-data years (2020-2023) were used as the closest reasonable comparison window.

---

## Finding 1: The 2022 Case Definition Change Coincides With a Reporting Spike

In 2022, CDC-designated high-incidence states adopted a stricter, laboratory-confirmation-only case definition, while low-incidence states continued using a broader definition that also allows clinical diagnosis without lab confirmation. This policy change and its stated effect on reported case counts are documented directly in CDC surveillance literature.

This analysis independently measured its effect using the county-level dataset built for this project. Comparing average case counts in counties with established tick populations:

| State Group | Avg. Cases, 2021 | Avg. Cases, 2022 | Change |
|---|---|---|---|
| High-Incidence States | 40.6 | 116.3 | +186% |
| Low-Incidence States | 3.8 | 2.7 | -29% |

![Lyme Disease Case Trends](lyme_trend_chart.png)

The sharp increase is concentrated almost entirely in high-incidence states which is exactly the group whose case definition changed while low-incidence states, whose methodology was unchanged, show no comparable increase. This pattern, measured directly from the data, is consistent with the spike reflecting a change in *measurement methodology* rather than a near-tripling of actual disease transmission in a single year. This analysis cannot fully rule out other contributing factors, but the precise alignment of timing (2022) and geography (high-incidence states only) with a documented, dated policy change makes measurement artifact the most consistent explanation.

---

## Finding 2: Rural Counties Show a Measured Pattern of Zero-Case Reporting Despite Confirmed Tick Presence

Among counties with **confirmed, established** tick populations, a notable share report **zero** Lyme disease cases across all four analysis years (2020-2023) despite documented vector presence. This analysis measured how this pattern varies by county rurality:

| Urban-Rural Classification | % of Established Counties with Zero Cases |
|---|---|
| 1 — Large Central Metro | 2.6% |
| 2 — Large Fringe Metro | 11.9% |
| 3 — Medium Metro | 17.2% |
| 4 — Small Metro | 16.3% |
| 5 — Micropolitan | 20.1% |
| 6 — Noncore (Rural) | 33.4% |

Rural (Noncore) counties are roughly **13 times more likely** to show zero reported cases despite confirmed tick establishment than large central metro counties.

Separately, low-incidence states overall show a substantially higher zero-case rate (34.1%) among established-tick counties than high-incidence states (0.6%).

**What this analysis can and cannot conclude:** this project measured a clear, statistically consistent association between rurality, low-incidence classification, and zero-case reporting in counties with confirmed tick presence. It did not measure *why* this association exists. Plausible explanations include reduced healthcare access, lower clinical suspicion of Lyme disease in non-endemic-adjacent areas, or other unmeasured factors but no healthcare access, provider density, or diagnostic behavior data was collected or tested in this analysis. These explanations are offered as hypotheses for future testing, not as measured findings.

---

## Limitations

- **Tick data is not year-specific.** Tick establishment status reflects cumulative surveillance current as of 2025 (83% based on historic/undated sources, the remainder dated 2004-2025) rather than a single collection year, and is compared here against 2020-2023 case data as the closest reasonable proxy.
- **Case data ends in 2023**, two years before the tick data's reference point. Tick establishment is a slow-changing biological reality, so this gap is not expected to materially affect the comparison, but is noted for transparency.
- **Case data reflects patient residence, not exposure location** (per CDC's own documentation), meaning travel-related cases may be attributed to counties with low actual tick exposure risk.
- **Tick surveillance is a passive system**, dependent on who submits samples for testing. "No records" status does not confirm tick absence — it may reflect under-sampling rather than true absence.
- **Connecticut is excluded** from the tick-status comparison due to incompatible geographic boundaries between data sources.
- **This analysis is correlational, not causal, and no explanatory mechanism was directly measured.** The rural and low-incidence zero-case patterns are robust, measured associations; healthcare access and clinical awareness are plausible but untested explanations for those associations.

---

## Future Work

- Directly test explanatory factors for the rural/low-incidence reporting gap, using measures such as physicians per capita, hospital density, or insurance coverage by county, rather than relying on rurality as an unexplained association
- Incorporate case data through 2024-2025 as it becomes available, to close the gap with the tick data's reference period
- Extend the pathogen-presence tick dataset (in addition to establishment status) to examine whether confirmed pathogen detection shows a similar rural reporting gap

---

## Repository Contents

- `lyme_disease.py` — Python cleaning pipeline for all four raw data sources
- `lyme_disease_analysis.sql` — Full database schema design and analysis queries
- `lyme_trend_chart.png` — Case trend visualization (2001-2023)
