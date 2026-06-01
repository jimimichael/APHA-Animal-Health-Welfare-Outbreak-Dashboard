# APHA Outbreak Dashboard - Data Model Documentation

## Star Schema Architecture

The Power BI data model implements a **complete star schema** with one fact table and three dimension tables for optimized query performance and flexible analytics.

```
                    ┌──────────────────────────────────────────────────────┐
                    │  ANIMAL_HEALTH_OUTBREAKS (FACT TABLE)                │
                    │  ─────────────────────────────────────────────────   │
                    │  10,000 rows | 36-month time series (May 2023-Jun26) │
                    ├──────────────────────────────────────────────────────┤
                    │ KEY COLUMNS:                                         │
                    │  • outbreak_id (PK)                    [Text]        │
                    │  • outbreak_date (FK)                  [Date]        │
                    │  • region_key (FK)                     [Text]        │
                    │  • farm_id (FK)                        [Text]        │
                    │                                                      │
                    │ DIMENSION COLUMNS:                                  │
                    │  • region, disease, animal_type        [Text]       │
                    │  • lab_result, severity                [Text]       │
                    │                                                      │
                    │ MEASURE COLUMNS:                                    │
                    │  • animals_affected (1-1,000)          [Whole #]    │
                    │  • response_days (1-30)                [Whole #]    │
                    │                                                      │
                    │ CALCULATED COLUMNS:                                 │
                    │  ├─ Animal Category (Livestock/Poultry/Wildlife)   │
                    │  ├─ Severity Group (High/Critical vs Low/Medium)   │
                    │  ├─ is_high_severity (Boolean)                     │
                    │  └─ Is Exception (Boolean) [High/Critical + >14d]  │
                    └──────────────────────────────────────────────────────┘
                              ↙                    ↓                    ↘
            ┌──────────────────────┐  ┌──────────────────────┐  ┌────────────────────┐
            │   Date (DIMENSION)   │  │ Region (DIMENSION)   │  │ Farm Ref (DIMENSION)│
            ├──────────────────────┤  ├──────────────────────┤  ├────────────────────┤
            │ Date (PK)     [Date] │  │ Region Key (PK) [Txt]│  │ farm_id (PK)  [Txt]│
            │                      │  │ Region Name     [Txt]│  │ region        [Txt]│
            │ [Standard date       │  │ APHA Operational     │  │ farm_type     [Txt]│
            │  attributes from     │  │  Area           [Txt]│  │ size          [Txt]│
            │  Power BI Date       │  │ Priority Region      │  │                    │
            │  Table function]    │  │  Flag        [Boolean]│  │ [200 farm records] │
            │                      │  │ Regional Target Days │  │                    │
            │ Enables:             │  │  (SLA)       [Number]│  │ Relationships:     │
            │ • Time hierarchy     │  │                      │  │ • Supports farm    │
            │ • Monthly trends     │  │ [7 APHA regions]     │  │   capacity analysis│
            │ • Seasonal analysis  │  │                      │  │ • Links regional   │
            │                      │  │ Enables:             │  │   outbreaks to     │
            │                      │  │ • Regional ranking   │  │   farm attributes  │
            │                      │  │ • Target comparison  │  │                    │
            │                      │  │ • Hotspot flagging   │  │                    │
            └──────────────────────┘  └──────────────────────┘  └────────────────────┘
```

---

## Table Specifications

### Fact Table: `animal_health_outbreaks`

**Row Count**: 10,000 outbreak records  
**Grain**: One row per outbreak event  
**Cardinality**: 10,000 unique outbreak IDs  
**Time Coverage**: May 2023 - June 2026 (36 months)

#### Primary Key
- `outbreak_id`: Surrogate key format `APH-2023-XXXX`

#### Foreign Keys (Dimensions)
| FK Column | References | Type | Purpose |
|:---|:---|:---|:---|
| `outbreak_date` | Date.Date | Date | Links to Date dimension for temporal analysis |
| `region_key` | Region.Region Key | Text | Links to Region dimension for geographic analysis |
| `farm_id` | Farm_Reference.farm_id | Text | Links to Farm dimension for location details |

#### Dimension Attributes
| Column | Type | Values | Usage |
|:---|:---|:---|:---|
| `region` | Text | 7 APHA regions | Slicer, visual grouping |
| `disease` | Text | 9 notifiable diseases | Filter, analysis |
| `animal_type` | Text | 21 species | Detail, drill-down |
| `severity` | Text | Low, Medium, High, Critical | Filtering, exception detection |
| `lab_result` | Text | Confirmed, Suspected, Negative, Pending | Data quality assessment |

#### Fact Metrics
| Column | Type | Range | Aggregation | Usage |
|:---|:---|:---|:---|:---|
| `animals_affected` | Whole Number | 1-1,000 | SUM | KPI: Total Animals Affected (5M+) |
| `response_days` | Whole Number | 1-30 | AVG | KPI: Avg Response Days (16 days) |

#### Calculated Columns

**1. Animal Category** [Text]
```
Logic: Categorizes animal types into epidemiological groups
Values: "Livestock", "Poultry", "Wildlife"
Used In: Disease By Animal Type stacked bar chart
Mapping:
  - Livestock: Cattle, Sheep, Goats, Pigs, Horses, Alpacas, Llamas, Deer
  - Poultry: Poultry, Gamebirds
  - Wildlife: Wild birds, Bats, Badgers, Foxes, Dogs, Cats, Ferrets, Primates, Rabbits, Marine mammals, Amphibians, Wild rodents
```

**2. Severity Group** [Text]
```
Logic: Aggregates severity into priority buckets
Formula: IF([severity] IN {"High","Critical"}, "High/Critical Sever", "Low/Medium Seve")
Values: "High/Critical Sever", "Low/Medium Seve"
Used In: High Severity Proportion donut chart
Note: Truncated text in display (Power BI limitation)
```

**3. is_high_severity** [Boolean]
```
Logic: Boolean flag for high-priority outbreaks
Formula: [severity] IN {"High","Critical"}
Values: TRUE / FALSE
Used In: Exception calculations, filtering
```

**4. Is Exception** [Text]
```
Logic: Flags outbreaks exceeding operational SLA (High/Critical severity + response > 14 days)
Formula: IF(AND([is_high_severity]=TRUE(), [response_days]>14), "Yes", "No")
Values: "Yes", "No"
Used In: Exception Table on Operational Response page (filters to "Yes" only)
Critical for: Resource allocation, prioritization
```

#### Measures (DAX)

**Total Outbreak**
```dax
Total Outbreak = COUNTROWS('animal_health_outbreaks')
Returns: Total number of outbreak records = 10,000
Used In: KPI cards, regional ranking, proportion calculations
```

**Total Animals Affected**
```dax
Total Animals Affected = SUM('animal_health_outbreaks'[animals_affected])
Returns: Sum of all animals impacted across all outbreaks ≈ 5,000,000+
Used In: KPI card for impact assessment
```

**Avg Response Days**
```dax
Avg Response Days = AVERAGE('animal_health_outbreaks'[response_days])
Returns: Mean days to response across all outbreaks = 16-22 days
Used In: KPI card, gauge chart, performance tracking
Comparison: Against Regional Target Days (14-day SLA)
```

**Confirmed Outbreak Rate**
```dax
Confirmed Outbreak Rate = 
DIVIDE(
  CALCULATE([Total Outbreak], 'animal_health_outbreaks'[lab_result]="Confirmed"),
  [Total Outbreak]
) * 100
Returns: Percentage of lab-confirmed outbreaks = ~60%
Used In: KPI card for data quality assessment
```

**High Severity Count**
```dax
High Severity Count = 
CALCULATE([Total Outbreak], 'animal_health_outbreaks'[severity] IN {"High","Critical"})
Returns: Count of High or Critical severity outbreaks ≈ 3,000
Used In: Severity proportion calculations
```

**Regional Rank**
```dax
Regional Rank = 
RANKX(
  ALL('Region'[Region Name]),
  [Total Outbreak],
  ,
  DESC,
  Dense
)
Returns: 1-7 ranking of regions by outbreak count
Used In: Regional Rank column in Regional Ranking table
Used For: Hotspot identification
```

**Exception Count**
```dax
Exception Count = 
CALCULATE([Total Outbreak], 'animal_health_outbreaks'[Is Exception]="Yes")
Returns: Count of high-priority, high-impact outbreaks
Used In: Exception Table filtering
Used For: Operational prioritization
```

---

### Dimension Table 1: `Date`

**Row Count**: ~1,100 rows (3-year span)  
**Primary Key**: `Date` (Text/Date format)  
**Grain**: One row per calendar day  

#### Standard Date Attributes (Power BI Date Table)
- Year
- Quarter (Q1-Q4)
- Month (January-December)
- Week
- Day of Week (Monday-Sunday)
- Day of Month

**Usage**:
- Time-series filtering on dashboard slicers
- Monthly aggregation for Outbreak Trend chart
- Seasonal pattern analysis
- Date range filtering (01/01/2023 - 31/12/2026)

---

### Dimension Table 2: `Region`

**Row Count**: 7 records (one per APHA operational area)  
**Primary Key**: `Region Key` (Text)  
**Grain**: One row per APHA region  

#### Columns

| Column | Type | Example Values | Purpose |
|:---|:---|:---|:---|
| `Region Key` | Text | "NE", "NW", etc. | Surrogate key for FK relationship |
| `Region Name` | Text | "North East (Newcastle)" | Display name for slicers/visuals |
| `APHA Operational Area` | Text | "Newcastle", "Preston" | Official APHA mapping |
| `Priority Region Flag` | Boolean | TRUE for East Midlands | Flags hotspot regions for focus |
| `Regional Target Days` | Number | 14 | SLA threshold per region (days) |

#### Regions Included

| Rank | Region Name | APHA Area | Priority | Target Days | Outbreaks |
|:---|:---|:---|:---|:---|:---|
| 1 | East Midlands (Crewe) | Crewe | TRUE | 14 | 1,489 |
| 2 | South East (Weybridge/Winchester) | Weybridge/Winchester | FALSE | 14 | 1,482 |
| 3 | Wales (Cardiff/Carmarthen) | Cardiff/Carmarthen | FALSE | 14 | 1,436 |
| 4 | Yorkshire & Humber (York) | York | FALSE | 14 | 1,410 |
| 5 | South West (Bristol/Exeter) | Bristol/Exeter | FALSE | 14 | 1,404 |
| 6 | North West (Preston/Carlisle) | Preston/Carlisle | FALSE | 14 | 1,402 |
| 7 | North East (Newcastle) | Newcastle | FALSE | 14 | 1,377 |

**Usage**:
- Regional Rank table visual
- Regional filtering on dashboard
- Gauge chart for regional target comparison
- Regional Rank calculation

---

### Dimension Table 3: `farm_reference`

**Row Count**: 200 records  
**Primary Key**: `farm_id` (Text)  
**Grain**: One row per farm  

#### Columns

| Column | Type | Domain | Example |
|:---|:---|:---|:---|
| `farm_id` | Text | FARM-0001 to FARM-0200 | FARM-0047 |
| `region` | Text | 7 APHA regions | "East Midlands (Crewe)" |
| `farm_type` | Text | Commercial, Smallholding, Organic, Research | Commercial |
| `size` | Text | Small (<50), Medium (50-200), Large (200+) | Large (200+) |

#### Distribution

**By Farm Type**:
- Commercial: ~70 farms (large-scale operations)
- Smallholding: ~50 farms (small independent operations)
- Organic: ~40 farms (certified organic)
- Research: ~40 farms (university/research facilities)

**By Size**:
- Small (<50 animals): ~70 farms
- Medium (50-200): ~65 farms
- Large (200+): ~65 farms

**By Region**: Evenly distributed across 7 regions (~29 farms per region)

**Usage**:
- Dimension for farm-level analysis
- Capacity planning visuals (farms per region)
- Links outbreak events to farm locations
- Supports drill-down from region → farm

---

## Relationships & Cardinality

| Relationship | Source | Target | Cardinality | Active | Purpose |
|:---|:---|:---|:---|:---|:---|
| Fact → Date | animal_health_outbreaks.outbreak_date | Date.Date | Many-to-One (N:1) | YES | Temporal analysis |
| Fact → Region | animal_health_outbreaks.region_key | Region.Region Key | Many-to-One (N:1) | YES | Geographic filtering |
| Fact → Farm | animal_health_outbreaks.farm_id | farm_reference.farm_id | Many-to-One (N:1) | NO* | Farm capacity analysis |

*Farm relationship is inactive to avoid ambiguity with region; can be activated for specific analyses.

---

## Data Quality & Validation

### Assertion Checks (Python Generator)
1. Dates within 36-month window
2. No duplicate outbreak_ids
3. Severity values from allowed set
4. Lab results from allowed set
5. Animal types valid for each disease
6. Animals affected > 0

### Expected Data Characteristics
- **Severity Distribution**: Low 40%, Medium 30%, High 20%, Critical 10%
- **Lab Result Distribution**: Confirmed 60%, Suspected 20%, Negative 10%, Pending 10%
- **Animals Affected**: Uniform random 1-1,000 (realistic farm scale)
- **Response Days**: Uniform random 1-30 (realistic operational window)
- **High Severity Exceptions**: ~2,958 outbreaks (29.58% with High/Critical severity)

---

## Usage Patterns

### KPI Dashboard (Executive Overview)
```
Total Outbreak ← COUNT of all outbreak_id
Total Animals Affected ← SUM(animals_affected)
Avg Response Days ← AVERAGE(response_days)
Confirmed Outbreak Rate ← CONFIRMED / TOTAL * 100
```

### Regional Analysis (Executive Overview)
```
Regional Rank Table ← [Total Outbreak] ranked by region
→ Identifies East Midlands as #1 hotspot
→ Used for resource allocation
```

### Disease Analysis
```
Disease By Animal Type ← Stacked bar by [Animal Category]
High Severity Proportion ← Donut by [Severity Group]
Top 5 Diseases ← Top 5 by [Total Outbreak]
Lab Result Breakdown ← Distribution of [lab_result]
```

### Operational Response
```
Avg Response Days Gauge ← [Avg Response Days] vs 14-day target
Exception Table ← Filtered to [Is Exception] = "Yes"
Monthly Response Trend ← [Avg Response Days] by Month
```

---

## Performance Optimization

1. **Fact Table**: Indexed on outbreak_id, region_key, outbreak_date
2. **Dimension Tables**: Small (7-200 rows), no indexing needed
3. **Calculated Columns**: Pre-computed at refresh to avoid runtime overhead
4. **Relationships**: Cardinality set to N:1 for proper filter propagation

---

## Extension Points

### Future Enhancements
- [ ] Add Cost Impact column to fact table (animals affected × severity weight × disease cost)
- [ ] Add time-to-result KPI (lab confirmation lag)
- [ ] Add cumulative burden metric (rolling 90-day outbreak count)
- [ ] Add forecast measure using DAX time intelligence
- [ ] Implement regional capacity utilization ratio (outbreaks / farms)

### Additional Dimensions
- [ ] Disease Dimension (disease metadata, typical mortality rate, etc.)
- [ ] Lab Dimension (lab codes, turnaround time standards)
- [ ] Operator Dimension (APHA region codes, contact info)

---

## Metadata Summary

| Attribute | Value |
|:---|:---|
| Model Name | APHA Outbreak Dashboard |
| Last Refresh | [Dynamic - on-demand] |
| Total Tables | 4 (1 Fact + 3 Dimensions) |
| Total Columns | 40+ |
| Total Rows | ~11,200 |
| Primary Fact Table Rows | 10,000 |
| Time Period Covered | 36 months (May 2023 - June 2026) |
| Relationships | 3 active |
| Calculated Columns | 4 |
| DAX Measures | 7 |
| Dashboard Pages | 3 |

