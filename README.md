# SAP Business Data Cloud — Sales Revenue Analytics Platform

[![CI](https://github.com/OWNER/sap-bdc-capstone/actions/workflows/ci.yml/badge.svg)](https://github.com/OWNER/sap-bdc-capstone/actions/workflows/ci.yml)
[![Python 3.11+](https://img.shields.io/badge/python-3.11%2B-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Code style: ruff](https://img.shields.io/badge/lint-ruff-261230.svg)](https://github.com/astral-sh/ruff)

An end-to-end **Sales & Revenue Analytics Platform** modelled on **SAP Business Data Cloud (BDC)**.
It demonstrates the complete Order-to-Cash (O2C) analytics scenario across the BDC
**Acquire → Prepare → Consume** layers.

> The data sources are simulated S/4HANA SD-module tables so the pipeline runs fully offline and
> reproducibly. In production, the extract layer is replaced by a Datasphere JDBC / OData connector.

## Features

- **Acquire** — Simulated extraction from SAP S/4HANA SD module (VBAK, VBAP, KNA1, MARA).
- **Prepare** — A pandas ETL pipeline that cleanses, deduplicates, derives date dimensions, and
  enforces O2C business rules into a Datasphere-style star schema (`FACT_SALES`, `DIM_CUSTOMER`,
  `DIM_PRODUCT`).
- **KPI layer** — Six ANSI SQL analytical views (monthly trend, regional share, top products /
  customers, QoQ growth via `LAG`, fulfillment) compatible with the SAP Datasphere dialect.
- **Consume** — A self-contained SAC-style HTML dashboard (Chart.js) with KPI cards and live filters.
- **Tested & reproducible** — Seeded RNG for deterministic output and a pytest suite covering
  transforms, KPIs, and data-quality rules.

## Tech Stack

| Layer | Technology |
|---|---|
| Data Platform | SAP Business Data Cloud (BDC) |
| Data Fabric | SAP Datasphere |
| ETL Pipeline | Python 3.11+ · pandas |
| SQL Models | ANSI SQL (Datasphere-compatible) |
| Dashboard | HTML5 · Chart.js 4 |
| Testing | pytest · pytest-cov |
| Tooling | ruff (lint + format) |

## Project Structure

```text
sap-bdc-capstone/
├── config/
│   └── bdc_config.json          # BDC space / connection configuration
├── dashboard/
│   └── index.html               # SAC-style analytics dashboard
├── docs/
│   └── PROJECT_DOCUMENTATION.md  # Architecture & design write-up
├── etl/
│   ├── extract.py               # S/4HANA data extraction (Acquire)
│   ├── transform.py             # Cleansing & enrichment (Prepare)
│   ├── load.py                  # Datasphere load layer (Prepare)
│   ├── pipeline.py              # Orchestrator — run this
│   └── data/
│       ├── raw/                 # Source data (generated, git-ignored)
│       └── processed/           # Pipeline output + KPIs (git-ignored)
├── models/sql/
│   ├── fact_sales.sql           # Fact table view
│   ├── dim_tables.sql           # Dimension views
│   └── kpi_revenue_analysis.sql # 6 KPI analytical views
├── tests/
│   └── test_transform.py        # Unit tests
├── pyproject.toml               # Build metadata + tool config
├── requirements.txt             # Pinned runtime/dev dependencies
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.11 or newer

### Installation

```bash
python -m venv .venv
# Linux/macOS:
source .venv/bin/activate
# Windows (PowerShell):
.venv\Scripts\Activate.ps1

pip install -r requirements.txt
```

### Run the pipeline

```bash
cd etl
python pipeline.py
```

This regenerates the raw data, runs Extract → Transform → Load, and writes the processed
star-schema tables plus `kpis.json` to `etl/data/processed/`.

### View the dashboard

Open `dashboard/index.html` in any modern browser.

## Testing

```bash
pytest                 # run the suite
pytest --cov=etl       # with coverage
```

## Architecture

```text
ACQUIRE          →    PREPARE                →    CONSUME
extract.py            transform.py / load.py      dashboard/index.html
S/4HANA SD tables     Star-schema modeling        KPI cards + charts
                      6 SQL KPI views
```

### Order-to-Cash (O2C) scenario

1. Sales Orders (VBAK / VBAP) → BDC ingestion layer.
2. Customer Master (KNA1) + Product Master (MARA) → dimension tables.
3. Datasphere → star-schema modeling + SQL analytical views.
4. SAC-style dashboard → KPI visualization.

See [`docs/PROJECT_DOCUMENTATION.md`](docs/PROJECT_DOCUMENTATION.md) for the full design write-up.

## Contributing

Contributions are welcome. Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull
request.

## License

Distributed under the MIT License. See [`LICENSE`](LICENSE) for details.
