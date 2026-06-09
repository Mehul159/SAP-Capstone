# Sales Revenue Analytics Platform — Design Documentation

**Platform:** SAP Business Data Cloud (BDC)
**Scenario:** Order-to-Cash (O2C) revenue analytics

---

## 1. Problem Statement

Enterprises running SAP S/4HANA generate large volumes of sales data across the SD module
(VBAK, VBAP, KNA1, MARA). Without a governed analytics layer, management cannot access real-time
revenue insights. This project addresses:

- **No unified revenue view** — sales data sits in siloed SD tables with no cross-functional KPI
  layer across regions, products, and customers.
- **Unscalable manual reporting** — spreadsheet-based processes cannot handle thousands of monthly
  transactions and offer no drill-down capability.
- **No governed data model** — no star schema compatible with SAP Analytics Cloud (SAC) for live
  dashboard consumption.
- **Revenue leakage** — cancelled-order revenue, fulfillment degradation, and DSO inflation go
  undetected without automated monitoring.

## 2. Objectives

| # | Objective | Expected Outcome |
|---|---|---|
| 1 | Design BDC data ingestion from SAP S/4HANA | Python ETL pipeline — VBAK/VBAP/KNA1/MARA extraction |
| 2 | Build a Datasphere star schema | `FACT_SALES` + `DIM_CUSTOMER` + `DIM_PRODUCT` with surrogate keys |
| 3 | Develop SQL analytical views | 6 Datasphere KPI views with window functions (`LAG`, `PARTITION BY`) |
| 4 | Create a SAC-style dashboard | HTML5 dashboard with live filters, KPI cards, and charts |
| 5 | Identify KPIs and business benefits | Measurable revenue, fulfillment, and efficiency improvements |
| 6 | Analyse risks and propose mitigation | Risk register mapped to SAP BDC business rules and controls |

## 3. Reference Scenario — ABC Electronics Pvt Ltd

A mid-sized B2B/B2C electronics company. Products span laptops, smartphones, tablets, networking
devices, and accessories for retail and wholesale buyers.

| Element | Value |
|---|---|
| Company Code | ABC1 — legal entity |
| Sales Organisation | SO01 |
| Distribution Channels | DI (direct retail) / WH (wholesale B2B) |
| Division | EL — Electronics |
| Plant / Storage | PL01 / SL01 |

## 4. Order-to-Cash Process

The O2C cycle spans eight sequentially integrated steps across Sales, Logistics, and Finance. Data
created in each step is inherited by the next — eliminating redundant entry and ensuring a single
source of truth.

| Step | Activity | T-Code | Dept | Key Output |
|---|---|---|---|---|
| 1 | Sales Order | VA01 | Sales | Standard Order (OR); pricing auto-determined |
| 2 | Credit Check | Auto | Sales / FI | Credit limit validated; order blocked if exceeded |
| 3 | ATP Check | VA01 | Sales / MM | Stock confirmed; delivery date proposed |
| 4 | Delivery Doc | VL01N | Logistics | Outbound Delivery (LF); picking triggered |
| 5 | Pick & Pack | LT0A | Warehouse | Transfer Order; goods staged for dispatch |
| 6 | Post Goods Issue | VL02N | Logistics / MM | Stock reduced; COGS posted to FI (mvt 601) |
| 7 | Invoice | VF01 | Finance | Customer Invoice (F2); A/R debited, Revenue credited |
| 8 | Payment Clearing | F-28 | Finance | Open item cleared; customer credit updated |

## 5. BDC Three-Layer Architecture

The platform implements SAP BDC's **Acquire → Prepare → Consume** architecture. Every component
maps to a real BDC layer and is production-deployable.

| BDC Layer | File / Module | Description |
|---|---|---|
| Acquire | `etl/extract.py` | Reads simulated S/4HANA tables |
| Prepare | `etl/transform.py` | Cleanses, deduplicates, derives date dims, applies O2C rules |
| Prepare | `etl/load.py` | Writes BDC-ready tables and KPI output |
| Prepare | `models/sql/` | 6 Datasphere SQL views: monthly, regional, QoQ, top products/customers, fulfillment |
| Consume | `dashboard/index.html` | SAC-style dashboard — KPI cards + charts + live filters |

## 6. Datasphere Star Schema

The analytical model follows a star schema — one central fact table joined to dimension tables via
surrogate keys — compatible with SAP Analytics Cloud Live Data Connection.

| Table | Type | Key Columns |
|---|---|---|
| `FACT_SALES` | Fact | `fact_key` · `order_id` · `customer_id` · `product_id` · `revenue` · `quantity` · `status` · `region` |
| `DIM_CUSTOMER` | Dimension | `dim_customer_key` · `customer_id` · `name` · `region` · `segment` · `credit_limit` |
| `DIM_PRODUCT` | Dimension | `dim_product_key` · `product_id` · `name` · `category` · `base_price` |

## 7. KPI Analytics

The pipeline computes a full set of KPIs (also expressed as Datasphere SQL views):

- Total revenue, total orders, average order value
- Order fulfillment rate
- Revenue by region, category, month, and quarter
- Quarter-over-quarter growth (window function `LAG`)
- Top customers and top products by revenue

Revenue figures are reproducible because the synthetic source data is generated with a fixed RNG
seed (`seed=42`).

## 8. SAP Module Integration — SD, MM & FI

Data flows automatically between SAP modules at every O2C stage — eliminating manual re-entry and
ensuring a single source of truth.

| Integration Point | Modules | Trigger | Outcome |
|---|---|---|---|
| Availability Check | SD ↔ MM | VA01 save | Delivery date confirmed; ATP stock reserved |
| Post Goods Issue | SD ↔ MM ↔ FI | VL02N PGI | Stock reduced; COGS posted to FI (mvt 601) |
| Revenue Recognition | SD ↔ FI | VF01 Invoice | A/R debited; Revenue credited in G/L |
| Credit Management | SD ↔ FI | Order creation | Credit limit checked; auto-block if exceeded |
| Tax Determination | SD ↔ FI | VA01 pricing | GST/IGST computed; posted to tax GL accounts |
| Payment Clearing | FI ↔ SD | F-28 payment | Open invoice cleared; customer credit updated |

## 9. Business Benefits & KPI Impact

A structured SAP BDC-driven O2C process delivers measurable improvements across operational,
financial, and customer-facing KPIs.

| KPI | Baseline | Post-BDC | Driving Feature |
|---|---|---|---|
| Order Processing Time | 4–6 hours | 30–45 min | Automated pricing, partner determination (VA01) |
| Invoice Accuracy Rate | ~85% | 99%+ | Condition technique, pricing copy control |
| Days Sales Outstanding | 45–55 days | 25–30 days | Automated dunning (F150), real-time A/R |
| Delivery On-Time Rate | 72% | 92%+ | ATP check, route determination, WM integration |
| Stock Discrepancy | ~8% | < 1% | PGI auto-posting; real-time inventory (mvt 601) |
| Month-End Close | 5–7 days | 1–2 days | Auto FI postings; integrated revenue accounts |

## 10. Risk Register

| Risk | Impact | Mitigation | SAP / BDC Control |
|---|---|---|---|
| Incorrect customer master | Wrong pricing / delivery | Mandatory field validation at XD01 | Customer master required fields |
| Stock-out (peak demand) | Delivery delays / lost revenue | Safety stock + reorder points in MM | MRP / ATP in material master |
| Pricing / invoice disputes | DSO spike; payment delays | Pricing simulation review in VA01 | Condition audit via VK13 |
| Credit limit breach | Bad-debt exposure | Tiered credit with auto order block | FD32 / Credit management (FSCM) |
| Pipeline data quality | Incorrect KPI output | Unit tests; null/dedup validation | `transform.py` validation layer |

## 11. Key Design Decisions

- **BDC-native architecture** — strictly follows Acquire → Prepare → Consume; every component maps
  to a real SAP BDC layer.
- **Production-grade SQL views** — all analytical views use ANSI window functions (`LAG` for QoQ
  growth, `SUM OVER PARTITION` for revenue share).
- **O2C business-rule enforcement** — cancelled orders yield zero revenue at the transform layer,
  mirroring SAP FI billing logic.
- **Reproducible & auditable** — seeded RNG ensures identical output every run; unit tests validate
  transforms, KPIs, and data-quality rules.
- **Full data lineage** — every record carries a `load_timestamp` and surrogate key for end-to-end
  traceability from source to dashboard.

## 12. Future Improvements

- **Live Datasphere connection** — replace CSV loads with REST API writes using Datasphere Data
  Flows.
- **SAC story with forecasting** — publish views as SAC Live Data Connections with ML-driven
  revenue forecasting.
- **Delta loading** — implement change-data-capture via `AEDAT` (S/4HANA last-changed date).
- **SAP Fiori self-service** — deploy "My Sales Orders" and "Track Delivery" Fiori apps.
- **P2P extension** — integrate the MM module (EKKO/EKPO) for a full Procure-to-Pay vs O2C
  comparison.
