# Project Structure

## Principle

The repository is organized by **responsibility**. A folder exists because a part of the system owns a specific responsibility.

We do not create a large tree first and then invent work for it.

## Target Structure

```text
MOBILITY_LOGISTIC/
│
├── app/
│   ├── api/
│   ├── core/
│   ├── db/
│   ├── ingestion/
│   ├── validation/
│   ├── alignment/
│   ├── analytics/
│   ├── decision/
│   └── services/
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── synthetic/
│
├── docs/
│   ├── architecture.md
│   ├── project-structure.md
│   ├── data-model.md
│   └── decisions.md
│
├── scripts/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── data_quality/
│
├── migrations/
└── .github/workflows/
```

## Responsibilities

| Directory | Responsibility | First expected artifacts |
|---|---|---|
| `app/api` | HTTP interface | routers, request/response schemas |
| `app/core` | cross-cutting application concerns | settings, shared contracts, constants |
| `app/db` | persistence boundary | engine, session, ORM models, repositories |
| `app/ingestion` | obtaining source data | source clients, parsers, ingestion jobs |
| `app/validation` | determining whether data is trustworthy | validators, quality rules |
| `app/alignment` | time and spatial matching | alignment functions/services |
| `app/analytics` | history and feature generation | historical queries, features, distributions |
| `app/decision` | operational reasoning | state, feasibility, prediction, recommendation |
| `app/services` | orchestration | application use-cases combining modules |
| `data/raw` | source snapshots | raw payloads or local raw extracts |
| `data/processed` | validated/transformed datasets | normalized exports/intermediate data |
| `data/synthetic` | synthetic operational data | generators and generated fixtures |
| `tests/unit` | isolated behavior tests | module-level tests |
| `tests/integration` | component interaction tests | DB/API/pipeline tests |
| `tests/data_quality` | data contract and quality tests | validation fixtures |
| `scripts` | repeatable local utilities | setup/seed/import scripts |

## Dependency Direction

The intended dependency flow is:

```text
api
 ↓
services
 ↓
decision / analytics / alignment / validation / ingestion
 ↓
db
```

The database layer must not call the API layer.

Business and decision logic must not directly open SQLite connections. Persistence goes through the database/repository boundary.

## What We Build First

We will not implement every directory immediately.

### Step 1

```text
app/core
app/db
app/validation
```

Purpose: establish configuration, persistence, and data contracts.

### Step 2

```text
app/ingestion
```

Purpose: build one complete real-data ingestion path.

### Step 3

```text
app/alignment
app/analytics
```

Purpose: construct context from the ingested data.

### Step 4

```text
app/decision
```

Purpose: current state → feasibility → baseline prediction → recommendation.

### Step 5

```text
app/services
app/api
```

Purpose: expose the working system through stable application interfaces.

## SQLite Rule

The local database file is an environment artifact, not source code.

```text
data/mobility.db
```

must be ignored by Git.

The schema/model definitions belong in `app/db`, while the actual database file stays local.

## Architectural Test

For every new Python file, answer:

```text
What responsibility does this file own?
What does it receive?
What does it return?
What may it depend on?
What must it never depend on?
How will it be tested?
```

If these answers are unclear, the file should not be created yet.
