# Snowflake Cortex Agents + MCP: Logistics Operations Intelligence

**Built by:** Sonali Dash | MIS Graduate Student, Texas A&M University (Mays Business School) | Source: https://github.com/sfc-gh-dbansal/mcp-hol/tree/main

---

## What This Project Does

A logistics operations manager wants to ask one question:

> *"Which carrier has the worst on-time delivery rate, and why?"*

This project builds the full data-to-AI pipeline in Snowflake to answer that -- structured metrics via **Cortex Analyst**, root-cause narratives via **Cortex Search**, and a **Snowflake-managed MCP server** that lets external AI clients (Claude, ChatGPT) query governed Snowflake data with zero custom infrastructure.

---

## Architecture

```
                        ┌──────────────────────┐
                        │   AI Client (Claude)  │
                        │   "Why is DHL late?"  │
                        └──────────┬───────────┘
                                   │
                          MCP Protocol (OAuth 2.0)
                                   │
                        ┌──────────▼───────────┐
                        │  Snowflake-Managed    │
                        │  MCP Server           │
                        │  (LOGISTICS_MCP)      │
                        └──────────┬───────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                  │
        ┌────────▼──────┐  ┌──────▼───────┐  ┌──────▼───────┐
        │ Cortex Analyst │  │ Cortex Search│  │ SQL Executor │
        │ (Text-to-SQL)  │  │ (Hybrid      │  │ (Runs the    │
        │                │  │  Retrieval)  │  │  generated   │
        │ "DHL: 46.7%   │  │              │  │  SQL)        │
        │  on-time"     │  │ "Mechanical  │  │              │
        │               │  │  failures,   │  │              │
        │               │  │  congestion" │  │              │
        └───────┬───────┘  └──────┬───────┘  └──────┬───────┘
                │                 │                  │
        ┌───────▼─────────────────▼──────────────────▼───────┐
        │              Snowflake Data Layer                   │
        │                                                     │
        │  SHIPMENTS (60 rows)     DELIVERY_INCIDENTS (30)    │
        │  Structured metrics      Free-text incident reports │
        │  4 carriers, Q1-Q2 2026  Root-cause narratives      │
        └─────────────────────────────────────────────────────┘
```

---

## What MCP Replaced

```
  BEFORE MCP                              AFTER MCP
  ──────────                              ─────────

  AI Client                               AI Client
      │                                       │
      ▼                                       ▼
  ┌──────────────┐                     ┌──────────────┐
  │ Custom API   │  ← You build,      │ MCP Server   │  ← Snowflake
  │ Server       │    deploy,         │ (managed)    │    manages it
  │ (Flask/Fast  │    monitor,        └──────────────┘
  │  API)        │    scale this
  │              │
  │ - Auth logic │
  │ - SQL exec   │
  │ - Search API │
  │ - Error      │
  │   handling   │
  │ - Token      │
  │   refresh    │
  └──────────────┘
      │
      ▼
  Snowflake                               Snowflake
```

**MCP eliminates the middleware entirely.** One Snowflake object, one standard protocol, governed by RBAC.

---

## Key Results

| Metric | Value |
|---|---|
| DHL on-time rate | 46.7% (worst of 4 carriers) |
| FedEx on-time rate | 86.7% (best) |
| Root causes surfaced | Mechanical failures, facility congestion, driver errors |
| Customer churn risk | 3 enterprise customers documenting DHL for carrier review |

---

## Snowflake Objects Built

| Layer | Object | Purpose |
|---|---|---|
| Data | `SHIPMENTS`, `DELIVERY_INCIDENTS` | Gold-layer structured + unstructured tables |
| Analyst | `LOGISTICS_ANALYTICS_SV` | Semantic view for text-to-SQL (dimensions, facts, metrics) |
| Search | `INCIDENT_SEARCH_SVC` | Hybrid retrieval over 30 incident report narratives |
| Agent | `LOGISTICS_AGENT` | In-database orchestration via `DATA_AGENT_RUN` |
| MCP | `LOGISTICS_MCP` | Exposes Analyst + Search + SQL execution as MCP tools |
| Security | `LOGISTICS_MCP_OAUTH` | OAuth 2.0 integration for external AI clients |

---

## Skills Demonstrated

- Snowflake Cortex AI services (Analyst, Search, Agents)
- Semantic view design (dimensions, facts, metrics, synonyms)
- MCP server architecture and OAuth security configuration
- Data modeling (medallion architecture: bronze/silver/gold)
- Snowflake RBAC and governance for AI workloads
- Git version control for Snowflake artifacts

---

## Repo Structure

```
├── notebooks/
│   ├── 01_Environment_Setup.ipynb       # Role, database, warehouse, data loading
│   ├── 02_Cortex_Analyst.ipynb          # Semantic view + structured queries
│   ├── 03_Cortex_Search.ipynb           # Hybrid search over incident reports
│   └── 04_Cortex_Agent_and_MCP.ipynb    # Agent + MCP server + OAuth + client
├── assets/
│   └── 00_logistics_hol_bootstrap.sql   # Setup script
└── README.md
```

---

## How to Run

1. Create a Snowflake trial account (Enterprise edition)
2. Run notebooks 01-04 sequentially in Snowsight (Projects → Notebooks)
3. Connect Claude or ChatGPT as an MCP client using the OAuth credentials from Notebook 04
4. Ask: *"Which carrier has the worst on-time delivery rate, and why?"*

---

## Tools Used

Snowflake (Cortex AI, Semantic Views, MCP Server) · GitHub Codespaces · Claude AI (MCP Client) · Git
