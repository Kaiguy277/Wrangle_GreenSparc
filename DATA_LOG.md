# Alaska Energy Data — Source Log
**Project:** AKEnergy
**Working directory:** `/home/kai/Documents/AKEnergy/`
**Last updated:** 2026-02-18

---

## Contents
1. [Data Sources](#data-sources)
2. [Files Produced](#files-produced)
3. [Data Quality & Known Issues](#data-quality--known-issues)
4. [Coverage Summary by Topic](#coverage-summary-by-topic)
5. [Cordova-Specific Notes](#cordova-specific-notes)

---

## Data Sources

### Source 1 — Alaska Energy Data Gateway (AEDG)
**Publisher:** Alaska Center for Energy and Power (ACEP), University of Alaska Fairbanks
**URL:** https://akenergygateway.alaska.edu
**Data repository:** https://github.com/acep-aedg/aedg-data-pond
**License:** CC-BY-4.0
**Access method:** Raw CSV download from GitHub (`data/public/` directory)
**Date retrieved:** 2026-02-18

The AEDG aggregates data from multiple state and federal programs into community-level datasets for all 355 tracked Alaska communities. It is funded by the Alaska Energy Authority and the U.S. DOE EPSCoR program.

| File downloaded | AEDG dataset name | Coverage | Rows |
|---|---|---|---|
| `data/public_communities.csv` | Communities | 355 AK communities, static | 355 |
| `data/public_yearly_generation.csv` | Yearly Generation | 2001–2021, by service area & fuel type | 14,080 |
| `data/public_monthly_generation.csv` | Monthly Generation | 2001–2021, by service area & fuel type | 162,975 |
| `data/public_capacity.csv` | Capacity | 2021 only, by community & fuel type | 764 |
| `data/public_fuel_prices.csv` | Fuel Prices | 2005–2025, heating fuel & gasoline | 29,291 |
| `data/public_rates.csv` | Electricity Rates | 2011–2021, by utility | 3,470 |
| `data/public_employment.csv` | Employment | 2001–2016 | 5,250 |
| `data/public_populations_ages_sexes.csv` | Population (ACS) | 2018–2019 ACS estimate period | 710 |
| `data/public_taxes.csv` | Municipal Taxes | 2017–2024 | 1,215 |
| `data/public_transportation.csv` | Transportation Access | Static snapshot | 354 |

**Underlying sources cited by AEDG:**
- Generation & capacity: Alaska Energy Authority (AEA) Power Cost Equalization (PCE) program data
- Fuel prices: Alaska Dept. of Commerce, Community & Economic Development (DCRA); survey vendors (Hovers Mover, Shoreside Petroleum)
- Electricity rates: AEA / utility filings
- Population: U.S. Census Bureau American Community Survey (ACS) 5-year estimates
- Employment: Alaska Dept. of Labor and Workforce Development (DOLWD)
- Community metadata: USGS GNIS, U.S. Census FIPS, ANCSA regional corporation records
- Transportation: Alaska Dept. of Transportation

---

### Source 2 — EIA Form EIA-923 (Electric Power Operations)
**Publisher:** U.S. Energy Information Administration (EIA)
**URL:** https://www.eia.gov/electricity/data/eia923/
**License:** Public domain (U.S. government work)
**Access method:** Annual ZIP archives downloaded directly; extracted XLSX parsed with pandas/openpyxl
**Date retrieved:** 2026-02-18

EIA-923 collects monthly and annual electric power data (generation, fuel consumption, fuel receipts) at the plant and prime-mover level from all certificated U.S. electric utilities. Alaska plants are included. Data is released approximately 2–4 months after the reference period.

| Archive downloaded | Reference year | Release status |
|---|---|---|
| `f923_2019.zip` | 2019 | Final Revision |
| `f923_2020.zip` | 2020 | Final Revision |
| `f923_2021.zip` | 2021 | Final Revision |
| `f923_2022.zip` | 2022 | Final Revision |
| `f923_2023.zip` | 2023 | Final Revision |
| `f923_2024.zip` | 2024 | Final |

Raw Excel files stored in `data/eia923/`. Parsed and combined into:
- `data/eia923_alaska_monthly.csv` — all Alaska plants, monthly net generation by plant/fuel, 2019–2024 (16,132 rows, 149 unique plants)

**Sheet used:** `Page 1 Generation and Fuel Data` (header row 5)
**Key columns extracted:** Plant Id, Plant Name, Plant State, Operator Name, Reported Fuel Type Code, Reported Prime Mover, monthly Netgen columns (Jan–Dec), annual Net Generation (MWh), Total Fuel Consumption Quantity, Physical Unit Label, Elec Fuel Consumption MMBtu, YEAR

**Alaska filter:** `Plant State == 'AK'`

**Fuel code mapping applied:**

| EIA code | Mapped label |
|---|---|
| DFO | Distillate Fuel Oil |
| WAT | Hydro |
| NG | Natural Gas |
| WND | Wind |
| SUB, LIG, WC, BIT | Coal (consolidated) |
| WO | Waste Oil |
| SUN | Solar |
| MWH | Battery Storage |
| LFG | Landfill Gas |
| JF | Jet Fuel |
| RFO | Residual Fuel Oil |
| OBL | Other Biomass |

---

### Source 3 — EIA Open Data API v2
**Publisher:** U.S. Energy Information Administration (EIA)
**Base URL:** https://api.eia.gov/v2/
**Documentation:** https://www.eia.gov/opendata/documentation.php
**License:** Public domain
**Access method:** EIA API v2 — free registration required; API key stored in `.env`
**Date retrieved:** 2026-02-18
**Most recent data available at time of retrieval:** November 2025

Three endpoints queried, all filtered to `location=AK` or `stateid=AK`:

#### 3a. Electric Power Operational Data
**Endpoint:** `electricity/electric-power-operational-data/data/`
**Underlying form:** EIA-923
**Parameters:** `frequency=monthly`, `facets[location][]=AK`, `facets[sectorid][]=99` (All Sectors), `start=2019-01`
**Output file:** `data/eia_api_generation_monthly.csv` (2,691 rows)
**Units:** Returned in thousand MWh → converted to MWh (× 1,000)
**Coverage:** 2019-01 to 2025-11, 33 fuel type codes

#### 3b. Electricity Retail Sales
**Endpoint:** `electricity/retail-sales/data/`
**Underlying forms:** EIA-826, EIA-861, EIA-861M
**Parameters:** `frequency=monthly`, `facets[stateid][]=AK`, `data[]=sales`, `data[]=price`, `data[]=customers`, `start=2019-01`
**Output file:** `data/eia_api_retail_sales.csv` (498 rows)
**Units:**
- `sales`: million kilowatt-hours → converted to MWh (× 1,000)
- `price`: cents per kWh → converted to $/kWh (÷ 100)
**Coverage:** 2019-01 to 2025-11, 6 sectors (all, residential, commercial, industrial, other, transportation)

#### 3c. Operating Generator Capacity (EIA-860)
**Endpoint:** `electricity/operating-generator-capacity/data/`
**Underlying form:** EIA-860, EIA-860M
**Parameters:** `frequency=monthly`, `facets[stateid][]=AK`, `data[]=nameplate-capacity-mw`, `start=2020-01`
**Output file:** `data/eia_api_capacity.csv` (39,155 rows — one row per generator per month)
**Units:** Nameplate capacity in MW
**Coverage:** 2020-01 to 2025-11, generator-level detail (plant name, technology, energy source, status)

---

## Files Produced

### Statewide / All-Alaska
| File | Description | Rows | Sources |
|---|---|---|---|
| `data/public_communities.csv` | 355 AK communities with coordinates, grid, PCE status, region | 355 | AEDG |
| `data/public_*.csv` (×9) | AEDG public datasets as downloaded | varies | AEDG |
| `data/eia923_alaska_monthly.csv` | Plant-level monthly generation, all AK, 2019–2024 | 16,132 | EIA-923 |
| `data/eia_api_generation_monthly.csv` | State-level monthly generation by fuel, 2019–2025-11 | 2,691 | EIA API |
| `data/eia_api_retail_sales.csv` | Monthly retail sales/price/customers by sector, 2019–2025-11 | 498 | EIA API |
| `data/eia_api_capacity.csv` | Generator-level monthly capacity, 2020–2025-11 | 39,155 | EIA API |
| `data/eia923/` | Raw EIA-923 XLSX files, 2019–2024 | — | EIA-923 |

### Cordova-Specific
| File | Description | Rows | Sources |
|---|---|---|---|
| `data/cordova_annual_timeline.csv` | Annual timeline 2002–2024, 19 columns | 23 | AEDG + EIA-923 |
| `data/cordova_monthly_generation.csv` | Monthly generation by plant, 2002–2024 | 276 | AEDG + EIA-923 |

### Notebooks
| File | Description |
|---|---|
| `explore_aedg.ipynb` | Statewide AEDG + EIA-923 + EIA API exploration (11 sections) |
| `cordova_energy.ipynb` | Cordova deep-dive: generation, diesel, fuel prices, rates, capacity |

---

## Data Quality & Known Issues

### Issue 1 — AEDG `service_area_generation_mwh` Double-Counts Shared Grids ⚠️
**Severity:** Critical for statewide aggregation
**Description:** The AEDG yearly and monthly generation datasets assign the full service-area generation total to every community on a shared grid. For example, Anchorage, Beluga, Cooper Landing, Hope, and Moose Pass all report identical Natural Gas generation (937,447 MWh in 2021) because they share the Southcentral grid. Summing across all communities produces ~76M MWh for 2021 — approximately 11× the correct statewide total.
**Correct 2021 total:** ~6.6M MWh (per EIA-923)
**Workaround:** Use EIA-923 or EIA API for statewide totals. AEDG generation is valid for communities on isolated/single-community grids (e.g., Cordova). Use AEDG community metadata (location, PCE eligibility, grid name) as intended.

### Issue 2 — AEDG Generation Data Ends at 2021
**Severity:** Moderate
**Description:** The most recent AEDG generation data is for fiscal year 2021. Capacity data is also 2021 only. Rates end at 2021. Employment ends at 2016.
**Workaround:** EIA-923 (through 2024) and EIA API (through November 2025) used for generation and capacity. AEDG fuel prices (through 2025) remain the best community-level source.

### Issue 3 — EIA-923 Excludes Small Off-Grid Generators
**Severity:** Low for Cordova; moderate for rural PCE communities
**Description:** EIA-923 reporting is mandatory for generators ≥1 MW or plants that sell power to the grid. Very small diesel generators in remote PCE communities (<1 MW) may not be captured. For a community like Cordova with known large plants (Power Creek 6 MW, Orca 7.7 MW, Humpback 1.2 MW), coverage is essentially complete.

### Issue 4 — Fuel Price Units and Source Mixing
**Description:** AEDG fuel prices combine two survey methodologies:
- `price_type = 'regional'`: DCRA statewide/regional price estimates — broader coverage, smoothed
- `price_type = 'survey'`: Point-in-time prices from named local vendors (e.g., Shoreside Petroleum, Hovers Mover) — more accurate for specific community but spottier coverage

In the Cordova annual timeline, the DCRA regional price is used as the primary (`heating_fuel_price_dcra_per_gal`); survey prices are preserved separately (`heating_fuel_price_survey_per_gal`). The DCRA price best represents diesel costs for power generation planning purposes.

### Issue 5 — Cordova 2023 Diesel Anomaly
**Description:** In 2023, Orca consumed 699,300 gallons (vs. ~530,000–586,000 in surrounding years) but generated only 6,290 MWh — yielding ~9.0 kWh/gallon, well below the normal 12–14 kWh/gallon. Simultaneously, Power Creek output was very low (many months near 0 MWh; February only 393 MWh, March only 230 MWh). This pattern is consistent with a significant Power Creek outage forcing Orca to run continuously at part load (diesel generators are less efficient at low/partial load). The cause is not documented in the available data.

### Issue 6 — Population Data is a Single ACS Estimate Period
**Description:** The AEDG populations dataset contains one or two rows per community representing ACS 5-year estimate periods (2018–2019 window), not a time series. Cordova's most recent population estimate is 2,405 (2019–2023 ACS). No annual population series is available in the AEDG data.

### Issue 7 — EIA API Rate Limits
**Description:** The EIA API v2 has rate limits for the public `DEMO_KEY`. The project API key (`J8o6ooGMxozY2UIaXl6vF8iBn7vROadtvvhhvpuk`) allows higher throughput. All API pulls used pagination (`length=5000`, `offset` stepping) to retrieve complete datasets. Data was pulled once and cached as CSVs; subsequent notebook runs read from cache.

---

## Coverage Summary by Topic

| Topic | Best source | Years available | Geographic level |
|---|---|---|---|
| Electricity generation (statewide) | EIA API / EIA-923 | 2019–2025 (monthly) | State / plant |
| Electricity generation (community) | AEDG | 2001–2021 | Community (isolated grids only) |
| Installed capacity | EIA API (EIA-860) | 2020–2025 (monthly) | Plant / generator |
| Retail consumption | EIA API (EIA-861) | 2019–2025 (monthly) | State / sector |
| Electricity rates | AEDG | 2011–2021 | Community / utility |
| Fuel prices (diesel/heating) | AEDG (DCRA + survey) | 2005–2025 | Community |
| Diesel consumption (gallons) | EIA-923 | 2019–2024 | Plant |
| PCE program data | AEA website (PDFs) | Through FY2024 | Community |
| Employment | AEDG (DOLWD) | 2001–2016 | Community |
| Population | AEDG (ACS) | ~2019 estimate | Community |
| Transportation access | AEDG | Static | Community |
| Municipal taxes | AEDG | 2017–2024 | Community |

---

## Cordova-Specific Notes

**FIPS code:** 0217410
**Grid:** Cordova Grid (isolated — not connected to Railbelt or any other grid)
**Utility:** Cordova Electric Cooperative, Inc.
**PCE eligible:** Yes
**AEA plant IDs (AEDG):** 132 (Humpback Creek), 133 (Orca diesel), 134 (Humpback Creek)
**EIA Plant IDs:**

| Plant | EIA Plant ID | Type | Capacity (Nov 2025) |
|---|---|---|---|
| Orca | 789 | Diesel (DFO) | 7.7 MW (3.6 + 2.5 + 1.6 MW operating; 1.1 MW OOS) |
| Power Creek | 7751 | Hydro (WAT) | 6.0 MW (2 × 3.0 MW) |
| Humpback Creek | 7042 | Hydro (WAT) | 1.2 MW (3 units: 0.5 + 0.5 + 0.2 MW) |
| Eyak Service Center BESS | 62714 | Battery (MWH) | 1.0 MW (added ~2019) |

**Generation data bridge:** AEDG (2002–2018) → EIA-923 (2019–2024). The 2019–2021 overlap period shows high agreement between sources (within 1–2%).

**Diesel consumption data availability:** EIA-923 provides actual gallons consumed and MMBtu only from 2019. Pre-2019 diesel consumption can be estimated by dividing diesel generation MWh by a typical efficiency (12–14 kWh/gallon for Orca's generator set), but no metered consumption data exists before 2019 in these datasets.

**PCE fuel price:** The AEDG `service_area_pce_fuel_price` column in the monthly generation table is null for Cordova — likely because Cordova's PCE fuel price is not separately tracked at the community level in this dataset, or because Cordova's large hydro share means diesel volumes are reported differently. The DCRA regional heating fuel price is used as the best available proxy for diesel/fuel oil pricing.

**Rate data gap:** Electricity rates from Cordova Electric Cooperative are available 2011–2021 via AEDG. Post-2021 rates are not in any of the retrieved datasets. Current rates would require direct contact with Cordova Electric Cooperative or an AEA PCE report query.

---

*Log generated 2026-02-18. Data retrieved from public sources; no proprietary data included.*
