# APHA Animal Health & Welfare Outbreak Dashboard

> **An end-to-end operational reporting system that monitors animal disease outbreaks across UK regions, tracks response performance against SLA targets, and identifies high-priority cases for field intervention — built to reflect APHA's published surveillance and reporting requirements.**

[![Power BI](https://img.shields.io/badge/Power_BI-Dashboard-yellow)](https://powerbi.microsoft.com)
[![Python](https://img.shields.io/badge/Python-3.8+-blue)](https://python.org)
[![Star Schema](https://img.shields.io/badge/Data_Model-Star_Schema-green)]()
[![Licence: MIT](https://img.shields.io/badge/Licence-MIT-lightgrey)](LICENSE)

---

## The Business Problem

Animal disease surveillance organisations must make fast, evidence-based decisions about where to direct limited field resources. Without structured operational intelligence, three problems emerge:

- **Detection lag** — emerging outbreak clusters aren't identified until they become widespread
- **Response accountability gaps** — no visibility into whether cases are being investigated within target timeframes
- **Resource misallocation** — field teams deployed reactively rather than to highest-risk regions

This dashboard addresses all three, providing a centralised view of outbreak patterns, response performance, and exception cases that require urgent attention.

---

## What Decisions Does This System Support?

| Operational Question | Where Answered |
|---|---|
| Which regions have the highest outbreak concentration right now? | Page 1 — Regional Ranking Table |
| Are we meeting the 14-day response target? | Page 3 — Response Gauge + Exception Table |
| Which diseases and animal types are driving caseload? | Page 2 — Disease Analysis |
| Which specific outbreaks need urgent escalation today? | Page 3 — Exception Filter (High/Critical + >14 days) |
| Is the outbreak trend improving or worsening month-on-month? | Page 1 — Monthly Trend Line |
| How reliable is our data? (Confirmed vs Suspected) | Page 2 — Lab Result Breakdown |

---

## Dashboard Preview

| Page | Focus | Key Insight |
|---|---|---|
| **Executive Overview** | Total volume, regional hotspots, trend | East Midlands leads with 1,489 outbreaks |
| **Disease Analysis** | Disease type, animal susceptibility, severity | Scrapie, African Swine Fever, Bovine TB are top 3 |
| **Operational Response** | SLA performance, exception cases | Avg 22 days vs 14-day target — 2,958 exceptions |

[View full dashboard PDF](dashboard/APHA%20Animal%20Health%20%26%20Welfare%20Dashboard.pdf)

**Page 1 — Executive Overview**
![Executive Overview](screenshots/page1_overview.png)

**Page 2 — Disease Analysis**
![Disease Analysis](screenshots/page2_disease.png)

**Page 3 — Operational Response**
![Operational Response](screenshots/page3_response.png)

---

## Architecture

```
Business Requirements (APHA published standards)
              │
              ▼
┌─────────────────────────────┐
│   Data Generation           │  Python (Pandas, NumPy)
│   (Data_Generator.ipynb)    │  10,000 outbreak records
│                             │  Realistic business rules:
│                             │  - Disease-host restrictions
│                             │  - Severity distributions
│                             │  - Regional SLA targets
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│   Data Validation           │  6 assertion checks:
│                             │  - Date range validation
│                             │  - Categorical integrity
│                             │  - Uniqueness (outbreak_id)
│                             │  - Referential integrity
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│   Star Schema Data Model    │  Fact table + 3 dimension tables
│                             │  Optimised for Power BI reporting
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│   Power BI Dashboard        │  3-page stakeholder report
│                             │  DAX measures + calculated columns
│                             │  Exception-based reporting logic
└─────────────────────────────┘
```

---

## Data Model

```
            ┌──────────────┐    ┌──────────────────┐    ┌─────────────────┐
            │  dim_date    │    │  dim_region       │    │  dim_farm_ref   │
            │──────────────│    │──────────────────│    │─────────────────│
            │ Date (PK)    │    │ region_key (PK)   │    │ farm_id (PK)    │
            │ Year         │    │ Region Name       │    │ region          │
            │ Quarter      │    │ APHA Oper. Area   │    │ farm_type       │
            │ Month        │    │ Priority Flag     │    │ size            │
            │ Week         │    │ Target Days (SLA) │    │                 │
            └──────┬───────┘    └────────┬──────────┘    └────────┬────────┘
                   │                     │                         │
                   └──────────────┬──────┘─────────────────────────┘
                                  │
                    ┌─────────────▼──────────────────┐
                    │  fact_animal_health_outbreaks   │
                    │────────────────────────────────│
                    │ outbreak_id (PK)                │
                    │ outbreak_date (FK → dim_date)   │
                    │ region_key (FK → dim_region)    │
                    │ farm_id (FK → dim_farm_ref)     │
                    │ disease, animal_type, severity  │
                    │ lab_result                      │
                    │ animals_affected (1–1,000)      │
                    │ response_days (1–30)            │
                    │ [Calculated] Animal Category    │
                    │ [Calculated] Severity Group     │
                    │ [Calculated] Is Exception       │
                    └─────────────────────────────────┘
```

**Grain:** One row per outbreak event | **10,000 records** | **36-month time series**

---

## Key DAX Measures

| Measure | Purpose |
|---|---|
| `Total Outbreak` | COUNTROWS of fact table |
| `Total Animals Affected` | SUM of animals_affected |
| `Avg Response Days` | AVERAGE of response_days vs 14-day target |
| `Confirmed Outbreak Rate` | % confirmed vs total (data quality indicator) |
| `High Severity Count` | CALCULATE filtered to High/Critical severity |
| `Regional Rank` | RANKX across regions by outbreak volume |
| `Exception Count` | High/Critical severity AND response_days > 14 |

---

## Data Generation Details

The Python notebook generates synthetic but realistic outbreak data:

**Disease coverage (9 notifiable diseases):** Bovine TB, Avian Influenza, Foot and Mouth, Bluetongue, African Swine Fever, Rabies, Swine Dysentery, Scrapie, Newcastle Disease

**Disease-host restrictions** aligned to APHA statutory guidelines:
- African Swine Fever → suids only (pigs, wild boar)
- Avian Influenza → poultry, gamebirds, wild birds, carnivores
- Rabies → mammals only

**Severity distribution:** Low 40% / Medium 30% / High 20% / Critical 10%

**Lab results:** Confirmed 60% / Suspected 20% / Negative 10% / Pending 10%

**Validation assertions (6 checks):**
```python
assert report_dates.dt.date.min() >= start_date.date()    # Date range
assert report_dates.dt.date.max() <= end_date.date()
assert df['severity'].isin(severities).all()               # Categorical
assert df['lab_result'].isin(lab_results).all()
assert df['animal_type'].isin(valid_animals).all()
assert len(df['outbreak_id']) == df['outbreak_id'].nunique()  # Uniqueness
```

---

## Key Findings from the Dashboard

| Metric | Value | Operational Implication |
|---|---|---|
| Busiest region | East Midlands — 1,489 outbreaks | Priority for resource allocation |
| Top disease | Scrapie — 1,163 outbreaks | Primary surveillance focus |
| Avg response time | 22 days vs 14-day target | 8-day overrun — capacity issue |
| Confirmed rate | 60% | 40% still pending/suspected |
| High/Critical outbreaks | 29.58% of total | ~3 in 10 are elevated risk |
| Exception backlog | 2,958 outbreaks | Significant SLA breach volume |

---

## How to Run

### Prerequisites
- Python 3.8+ with `pandas`, `numpy`, `python-dateutil`
- Power BI Desktop (free from Microsoft)

### 1. Generate data
```bash
git clone https://github.com/jimimichael/APHA-Animal-Health-Welfare-Outbreak-Dashboard.git
cd APHA-Animal-Health-Welfare-Outbreak-Dashboard
pip install pandas numpy python-dateutil

# Open and run all cells in Jupyter
jupyter notebook scripts/Data_Generator.ipynb
```
Expected output:
```
Validation successful: generated outbreak data is within the expected range and format
Outbreak Records Created: 10000 | Farm Records Created: 200
```

### 2. Open the dashboard
Open `dashboard/APHA Animal Health & Welfare Dashboard.pbix` in Power BI Desktop.
If data sources disconnect: File → Options → Data Source Settings → update CSV paths → Refresh.

---

## Project Structure

```
APHA-Animal-Health-Welfare-Outbreak-Dashboard/
├── dashboard/
│   ├── APHA Animal Health & Welfare Dashboard.pbix   # Power BI file
│   └── APHA Animal Health & Welfare Dashboard.pdf    # Export snapshot
├── screenshots/
│   ├── page1_overview.png
│   ├── page2_disease.png
│   └── page3_response.png
├── data/
│   ├── animal_health_outbreaks.csv    # Fact table (10,000 records)
│   └── farm_reference.csv            # Dimension table (200 records)
├── scripts/
│   └── Data_Generator.ipynb          # Data generation + validation
├── documentation/
│   └── data_model.md                 # Full schema and DAX documentation
└── README.md
```

---

## Relevance to Public Sector Roles

The patterns demonstrated here are directly applicable to:
- **Animal and plant health surveillance** (APHA, Defra)
- **Public health performance monitoring** (UKHSA, NHS)
- **Regulatory compliance tracking** (any SLA-governed response function)
- **Environmental monitoring** (EA, NatureScot)

The combination of data quality validation, dimensional modelling, exception-based reporting, and stakeholder-focused dashboard design reflects standard practice in government analytical and performance roles.

---

## Skills Demonstrated

`Python` `Pandas` `NumPy` `Power BI` `DAX` `Power Query` `Star Schema Design` `Data Validation` `KPI Development` `Exception Reporting` `Dashboard Design` `Public Sector Analytics` `APHA Domain Knowledge`

---

> **Note:** This is a portfolio project using entirely synthetic data generated with realistic parameters derived from APHA's published disease surveillance standards. It is not a real APHA dataset or official reporting tool.

---

*Part of a portfolio demonstrating applied data analytics and decision-support systems. See also: [Sentinel Tax Risk Engine](https://github.com/jimimichael/sentinel-tax-risk-engine) | [UK Financial Complaints Pipeline](https://github.com/jimimichael/uk-financial-complaints-pipeline) | [DE ZoomCamp 2026](https://github.com/jimimichael/DE_ZoomCamp_2026)*
