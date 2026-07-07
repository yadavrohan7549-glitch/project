# AML Transaction Monitoring System

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-Dashboard-FF4B4B?logo=streamlit&logoColor=white)
![SQLite](https://img.shields.io/badge/Database-SQLite-003B57?logo=sqlite&logoColor=white)
![scikit-learn](https://img.shields.io/badge/ML-Isolation%20Forest-F7931E?logo=scikitlearn&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

A Python-based transaction monitoring platform that replicates how financial institutions detect suspicious activity, score customer risk, generate investigation cases, and report on operational metrics — built end-to-end on synthetic data with a rule engine, an unsupervised anomaly detection model, and an interactive Streamlit dashboard.

> This project uses only synthetically generated data for educational and portfolio purposes. No real customer, transaction, or financial institution data is included anywhere in this repository.

---

## Live Demo

Dashboard:  [aml-transaction-monitoring-system.streamlit.app](https://aml-transaction-monitoring-system.streamlit.app)
Repository: [github.com/yadavrohan7549-glitch/aml-transaction-monitoring-system](https://github.com/yadavrohan7549-glitch/aml-transaction-monitoring-system)

---

## Table of Contents

- [Overview](#overview)
- [Motivation](#motivation)
- [Features](#features)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Folder Structure](#folder-structure)
- [Technical Design Decisions](#technical-design-decisions)
- [Risk Scoring Model](#risk-scoring-model)
- [Rule Engine](#rule-engine)
- [Machine Learning](#machine-learning)
- [Database Schema](#database-schema)
- [Reports](#reports)
- [Dashboard](#dashboard)
- [Screenshots](#screenshots)
- [Installation](#installation)
- [Project Statistics](#project-statistics)
- [Key Learning Outcomes](#key-learning-outcomes)
- [Limitations](#limitations)
- [Future Improvements](#future-improvements)
- [License](#license)
- [Author](#author)

---

## Overview

This project simulates a bank's internal transaction monitoring stack from data intake through to case reporting. It generates a synthetic dataset of customers, merchants, and transactions, runs that data through a fifteen-rule AML detection engine and an Isolation Forest anomaly model, combines both signals into a transparent weighted risk score, converts alerts into investigation cases, and exposes the whole thing through a Streamlit dashboard with search, filtering, and Excel reporting.

The scope was chosen deliberately to mirror the actual workflow an AML analyst works inside — not to build a general-purpose fraud platform. Everything in this repository maps to a specific compliance function: alert generation, case triage, risk banding, and report production.

No real customer, transaction, or institutional data is used anywhere in this project.

---

## Motivation

Transaction monitoring exists because financial institutions are legally required to detect and report activity connected to money laundering, terrorist financing, and other financial crime. In practice, that requirement gets implemented as a pipeline: transactions flow in, rules and models flag the ones that look wrong, analysts investigate the flags, and a subset of those investigations get escalated into a Suspicious Activity Report.

Most public AML portfolio projects stop at "detect fraud in a CSV." That doesn't reflect how the job actually works. An analyst's day involves risk bands, case queues, alert prioritization, and knowing why a customer was flagged well enough to defend that decision to a reviewer. This project was built to demonstrate that full workflow — not just a detection model — because that's what the role actually requires.

---

## Features

- Synthetic generation of 500 customers, 50 merchants, and 10,000+ transactions, including deliberately seeded suspicious patterns (structuring bursts, rapid movement, velocity spikes) so the rule engine has real positives to catch
- Data cleaning and validation with a full audit trail
- Fifteen-rule AML detection engine covering structuring, velocity, sanctions, PEP exposure, and more
- Isolation Forest anomaly detection layered on top of the rule engine, not in place of it
- Transparent, weighted customer risk scoring (Low / Medium / High / Critical)
- Automatic case generation that deduplicates per-customer alerts into a single investigation case
- SQLite persistence across eight tables
- Seven auto-generated Excel reports, including a SAR-candidate report
- Ten static charts plus a fully interactive Plotly-driven dashboard
- Six-tab Streamlit dashboard: Executive Summary, Alerts, Risk & Customers, Cases, Charts, Search
- Customer and case search with transaction-history drill-down
- Full audit logging to both a text log and a database table
- Unit tests covering rule logic, risk banding, and data cleaning

---

## Technology Stack

| Category | Technology |
|---|---|
| Language | Python 3 |
| Data processing | Pandas, NumPy |
| Machine learning | Scikit-learn (Isolation Forest) |
| Database | SQLite |
| Dashboard | Streamlit, Plotly |
| Static charts | Matplotlib |
| Reporting | OpenPyXL (Excel) |
| Data generation | Custom synthetic generator |
| Testing | Pytest |
| Logging | Python `logging` module + SQLite audit table |

---

## Architecture

The pipeline runs as a single linear flow, orchestrated by `main.py`:

```
Synthetic Data Generation
        │
        ▼
   Data Cleaning
        │
        ▼
ML Anomaly Detection (Isolation Forest)
        │
        ▼
   Rule Engine (15 rules)
        │
        ▼
 Weighted Risk Scoring
        │
        ▼
  Case Generation
        │
        ▼
 SQLite Persistence
        │
        ▼
Reports + Charts
        │
        ▼
Streamlit Dashboard (reads from SQLite)
```

**Synthetic data generation** produces customers, merchants, and transactions with realistic distributions — income follows a log-normal curve, country selection is weighted toward higher-risk jurisdictions for a minority of accounts, and a subset of customers get explicitly seeded with structuring, rapid-movement, or high-velocity behavior so the downstream rules have genuine signal to detect, not just noise.

**Data cleaning** deduplicates records, validates types and date formats, drops invalid amounts, and normalizes every transaction into a USD-equivalent field so cross-currency rules compare on a common basis. Every action taken is logged with a row count.

**ML anomaly detection** runs before the rule engine so its output — an anomaly flag per transaction — can feed into both the rule layer and the risk scoring layer as one more signal.

**The rule engine** evaluates fifteen independent, unit-testable detection rules against the cleaned transaction set and produces one alert row per rule trigger, each carrying a reason string, a risk score, and a priority.

**Risk scoring** aggregates alerts, ML flags, and static customer attributes (PEP status, sanctions status) into a single weighted score per customer, which is then banded into Low, Medium, High, or Critical.

**Case generation** groups alerts by customer — not by transaction — because that mirrors how an analyst actually works a queue: one customer can trip several rules, and that becomes one case, not five.

**Everything is persisted to SQLite**, then used to generate seven Excel reports and ten charts, and finally served through the Streamlit dashboard, which reads directly from the database rather than holding its own copy of the pipeline's output.

---

## Folder Structure

```
project/
├── data/
│   ├── raw/            Generated CSVs: customers, transactions, merchants
│   ├── processed/       Reserved for cleaned intermediate exports
│   └── generated/       audit_log.txt and other run artifacts
├── database/
│   └── aml_system.db    SQLite database — customers, transactions, merchants,
│                         alerts, cases, risk_scores, countries, audit_logs
├── models/
│   └── isolation_forest.joblib   Trained anomaly detection model
├── reports/              Seven auto-generated Excel reports
├── charts/                Ten auto-generated PNG charts
├── dashboard/
│   └── app.py             Streamlit dashboard, six tabs
├── src/
│   ├── config/
│   │   └── settings.py    Every threshold, weight, and path in one place
│   ├── data_generator.py  Synthetic data generation
│   ├── data_cleaner.py    Validation and cleaning
│   ├── rule_engine.py     Fifteen AML detection rules
│   ├── risk_scoring.py    Weighted risk model
│   ├── anomaly_detection.py  Isolation Forest
│   ├── case_manager.py    Alert-to-case conversion
│   ├── database.py        SQLite persistence layer
│   ├── report_generator.py   Excel report generation
│   ├── chart_generator.py    Static chart generation
│   └── logger.py           Audit logging
├── tests/
│   └── test_rules.py      Unit tests
├── main.py                 Pipeline entry point
├── requirements.txt
└── README.md
```

---

## Technical Design Decisions

**Why SQLite.** This is a single-user, file-based portfolio project with a dataset in the tens of thousands of rows, not millions. SQLite requires no server process, ships with Python, and keeps the whole system runnable with a single `python main.py` — a Postgres or MySQL dependency would add operational overhead without adding capability at this scale.

**Why Streamlit.** The dashboard needed to be interactive — filterable alert tables, drill-down customer search, live charts — without building a separate frontend and API layer. Streamlit reads directly from the same SQLite file the pipeline writes to, which keeps the presentation layer thin and avoids duplicating business logic between a backend and a UI.

**Why Isolation Forest.** Isolation Forest is unsupervised, which matters here because there are no labeled "confirmed suspicious" transactions to train against — that data doesn't exist in a synthetic environment or, realistically, in most banks' historical records at the volume needed for supervised learning. It's also computationally cheap at this scale and produces an anomaly score that's straightforward to explain to a non-technical reviewer: how isolated is this transaction from typical behavior, given a small set of behavioral features.

**Why weighted risk scoring instead of a single model score.** A black-box score is hard to defend in a compliance context. Every point in this model traces back to a named, documented factor — PEP status, a sanctions match, a structuring pattern — so an analyst (or a model validation reviewer) can look at any customer's score and see exactly why it is what it is. That transparency is a design requirement in regulated environments, not a nice-to-have.

**Why synthetic data.** Real transaction data is not something that can legally or practically be used in a public repository. The generator instead constructs data with realistic statistical shape — log-normal income distribution, weighted country risk, seeded suspicious patterns — so the rule engine and scoring model have genuine signal to work against, while nothing in the dataset corresponds to any real person or institution.

---

## Risk Scoring Model

Each customer's risk score is built from a fixed set of weighted factors. No factor is hidden — every point on a customer's score can be traced to a specific reason.

| Factor | Points | Rationale |
|---|---|---|
| Sanctions match | 40 | Direct regulatory and legal exposure |
| PEP status | 20 | Enhanced due diligence requirement |
| High-risk country activity | 15 | Jurisdictional risk exposure |
| Structuring pattern detected | 15 | Deliberate reporting-threshold avoidance |
| ML anomaly flag | 15 | Statistical outlier not captured by a named rule |
| High transaction velocity | 10 | Behavior sharply above the customer's own baseline |
| Large cash activity | 10 | Cash deposits at or above the CTR-equivalent threshold |
| Dormant account reactivation | 10 | Inactive account resuming activity without explanation |
| Two or more prior alerts | 10 | Repeat flags compound risk |

Scores are summed and banded as follows:

| Band | Score Range |
|---|---|
| Low | 0–19 |
| Medium | 20–44 |
| High | 45–69 |
| Critical | 70+ |

The banding thresholds and every weight above live in `src/config/settings.py`, structured as a parameter register rather than hardcoded constants scattered through the codebase — the same pattern a real risk model's documentation would follow for audit purposes.

---

## Rule Engine

Fifteen rules run against the cleaned transaction set. Each is implemented as an independent, testable function.

| Rule | Detects |
|---|---|
| Large Cash Transaction | Cash deposits at or above the reporting threshold |
| Round Amount Detection | Transactions in suspiciously clean, round figures |
| Just Below Reporting Threshold | Amounts sitting at 90–99% of the threshold |
| High Risk Country Transfer | Transactions routed through elevated-risk jurisdictions |
| Structuring (Smurfing) | Three or more sub-threshold deposits from one customer within 24 hours |
| Rapid Movement of Funds | Three or more outgoing transfers within 60 minutes |
| Velocity Detection | Daily transaction count spiking against the customer's own baseline |
| Dormant Account Reactivation | A previously inactive account resuming activity |
| Sudden Increase in Transaction Volume | Monthly volume five times above prior average |
| PEP Monitoring | Activity from a Politically Exposed Person |
| Sanction Screening | A match against the internal sanctions flag |
| Unusual Merchant Activity | High-value transactions with high-risk merchant categories |
| Multiple Accounts to Same Beneficiary | Several distinct senders paying one beneficiary in a short window |
| Multiple Countries in Short Time | Transactions spanning three or more countries within 24 hours |
| Account Takeover Indicator | Same customer, same day, multiple devices and IP addresses |

Every alert carries a rule name, a risk score, a plain-text reason describing exactly why it fired, and a priority level. Thresholds — the cash amount, the structuring window, the velocity multiplier, and so on — are configuration values, not magic numbers buried in rule logic.

---

## Machine Learning

The anomaly detection layer uses an **Isolation Forest**, trained on a small set of behavioral features per transaction: the USD-normalized amount, hour of day, day of week, a high-risk-country flag, a cash-payment flag, and the customer's own historical transaction count and average amount.

Isolation Forest works by randomly partitioning the data and measuring how quickly each point gets isolated from the rest — transactions that separate out in very few splits are the ones that look statistically unusual given the feature set. It doesn't need labeled examples of "suspicious" transactions to work, which matters because no such labels exist here or, in most cases, at scale in a real institution's historical data.

The model's anomaly flag is not used to override or suppress rule-based alerts. It's added as one more factor in the weighted risk score, on the same footing as any named rule. That design choice avoids a common failure mode where an ML layer silently filters out true positives a rule already caught correctly.

---

## Database Schema

SQLite database (`database/aml_system.db`) with eight tables:

| Table | Contents |
|---|---|
| `customers` | Identity, KYC status, PEP/sanctions flags, risk level, account metadata |
| `transactions` | Full transaction detail including USD-normalized amount and anomaly flag |
| `merchants` | Merchant category and high-risk classification |
| `alerts` | One row per rule trigger, with reason and priority |
| `cases` | One row per customer with an active investigation, aggregating alerts |
| `risk_scores` | Current weighted score, band, and contributing factors per customer |
| `countries` | Country-to-risk-category mapping |
| `audit_logs` | Timestamped record of every pipeline stage and dashboard access |

---

## Reports

Seven Excel reports are generated automatically on every pipeline run, written to `reports/`:

- **AML Investigation Report** — full alert and case detail
- **Customer Risk Report** — every customer with score, band, and contributing factors
- **Daily Alert Report** — alert counts by rule, by day
- **Monthly Summary Report** — alert volume and flagged value by month
- **High Risk Customer List** — customers in the High or Critical band
- **Suspicious Activity Report** — case candidates scoring at or above the SAR threshold, merged with customer context
- **Executive Dashboard Report** — top-line KPIs in a single summary sheet

All reports are formatted with styled headers and auto-sized columns rather than raw `DataFrame.to_excel()` output.

---

## Dashboard

`streamlit run dashboard/app.py` launches a six-tab interface, reading live from SQLite:

- **Executive Summary** — headline KPIs, risk distribution, daily transaction volume, and a country-level transaction heatmap
- **Alerts** — the full alert table, filterable by rule and priority
- **Risk & Customers** — top risky customers by score, top risky merchants by alert volume
- **Cases** — the investigation queue, filterable by status
- **Charts** — interactive Plotly versions of the monthly volume, customer segmentation, payment type mix, and ML anomaly scatter plot
- **Search** — customer and case lookup, with transaction-history drill-down for a single matched customer

---

## Screenshots

*(Add screenshots here after deployment)*

- Executive Summary dashboard
- Alerts tab with filters applied
- Case investigation queue
- Interactive charts view
- Customer search and risk drill-down

---

## Installation

```bash
git clone https://github.com/yadavrohan7549-glitch/aml-transaction-monitoring-system.git
cd aml-transaction-monitoring-system

python -m venv venv
source venv/bin/activate      # venv\Scripts\activate on Windows

pip install -r requirements.txt

python main.py                # runs the full pipeline
streamlit run dashboard/app.py   # launches the dashboard
```

Re-running `main.py` regenerates a fresh dataset and rebuilds every table, report, and chart from scratch — the generation seed is fixed, so results are reproducible run to run.

---

## Project Statistics

| Metric | Value |
|---|---|
| Customers generated | 500 |
| Merchants generated | 50 |
| Transactions generated | 10,000+ |
| AML rules implemented | 15 |
| Excel reports generated | 7 |
| Charts generated | 10 |
| Dashboard tabs | 6 |
| Database tables | 8 |

Exact alert, case, and risk-band counts vary slightly between runs due to the randomized (but seeded) generation process — see `data/generated/audit_log.txt` after running the pipeline for the figures from your own run.

---

## Key Learning Outcomes

Through building this project I gained hands-on experience with:

- AML transaction monitoring workflows, from raw transaction intake to case escalation
- Rule-based detection logic, including threshold tuning and time-window pattern matching
- Weighted customer risk scoring methodologies and risk banding
- Investigation case management and alert-to-case workflow design
- SQLite database design and schema normalization for compliance data
- Unsupervised machine learning (Isolation Forest) for anomaly detection
- Interactive dashboard development with Streamlit and Plotly
- Data visualization and automated Excel report generation
- Git and GitHub version control for a full project lifecycle
- Deploying analytical applications with Streamlit Community Cloud

---

## Limitations

- Built entirely on synthetic data — no real customer, transaction, or institutional data is used or referenced anywhere in this project
- An educational and portfolio project, not a production compliance system
- No real banking, payments, or core-system integration
- No true false-positive rate is available, since that requires labeled analyst dispositions that don't exist for synthetic data
- No authentication or access control — not designed for multi-user or production deployment as-is

---

## Future Improvements

- **Real-time streaming** — replace batch-mode CSV generation with a Kafka-backed transaction stream for near-real-time alerting
- **REST API layer** — expose the rule engine and risk scoring as callable endpoints, decoupling detection logic from the dashboard
- **PostgreSQL migration** — move off SQLite for concurrent multi-user access and larger data volumes
- **Docker and Kubernetes** — containerize the pipeline and dashboard for reproducible deployment and horizontal scaling
- **Cloud deployment** — host the full stack on a managed cloud platform rather than a local or single-instance environment
- **Role-based authentication** — separate analyst, reviewer, and administrator access levels
- **Formal SAR generation workflow** — structured draft creation with analyst notes and edit history, distinct from the current SAR-candidate report
- **Case notes and audit trail** — persistent, timestamped analyst commentary attached to each case
- **Live sanctions and PEP API integration** — replace the static internal flags with real screening providers
- **Graph analytics** — model customers, merchants, and beneficiaries as a network to surface fund-flow patterns a row-level rule engine can't see
- **Entity resolution and link analysis** — connect accounts sharing devices, IPs, or beneficiaries across the customer base
- **Neo4j and graph ML** — move network detection from ad hoc SQL joins to a proper graph database for mule network and fraud ring detection

---

## License

MIT License

---

## Author

**Name:** Rohan Yadav
**LinkedIn:** https://www.linkedin.com/in/rohan1245
**Email:** yadavrohan7549@gmail.com
**GitHub:** [github.com/yadavrohan7549-glitch](https://github.com/yadavrohan7549-glitch)
