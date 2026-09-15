# MOBILITY_LOGISTIC

## OMIND Mobility Intelligence

An end-to-end Data & AI decision system for mobility and logistics operations.

The system is designed to answer a practical operational question:

> **Given the current route, traffic, weather, vehicle, driver, cargo, and time context, what is likely to happen next, why, how reliable is the prediction, and what should the operation do?**

This repository is intentionally built as an engineering system rather than a dashboard, notebook, or isolated ML model.

---

## 1. Core Decision Loop

```text
Observe → Validate → Align → Context → Assess State → Check Feasibility
      → Predict → Explain → Recommend → Act → Measure Reality
      → Reconcile → Diagnose → Improve
```

A prediction is not treated as reality, and an incorrect prediction is not automatically treated as a model failure.

---

## 2. System Scope

### External data

Potential sources include weather, traffic/mobility, public transport, route/road-network, incident, road-work, and closure datasets.

### Operational data

The domain model represents trips, routes, drivers, vehicles, maintenance events, cargo, predictions, and observed outcomes.

When real company data is unavailable, operational data is synthetic and explicitly labelled as such. The project never presents synthetic data as real company data.

---

## 3. Master Architecture

```text
External / Internal Sources
          ↓
     Ingestion Layer
          ↓
      Raw Data Layer
          ↓
 Validation & Data Quality
          ↓
     Core Data Model
          ↓
 Time / Spatial Alignment
          ↓
 Analytics & Feature Engineering
          ↓
 Current State Assessment
          ↓
 Prediction & Decision Layer
          ↓
     Serving Layer
          ↓
    Actual Outcome
          ↓
 Prediction Reconciliation
          ↓
 Error / Root-Cause Diagnosis
          ↓
   Feedback & Improvement
```

Canonical system architecture: [`docs/architecture.md`](docs/architecture.md)

---

## 4. Technology Decision — SQLite First

The development environment may restrict installation of database servers. Therefore Phase 1 uses **SQLite** as the local relational database.

```text
Application
    ↓
Repository / Data Access Layer
    ↓
SQLite
```

Business logic must not depend directly on SQLite-specific implementation details.

Intended migration path:

```text
SQLite → PostgreSQL
```

This should be an infrastructure migration rather than a domain rewrite.

### Initial stack

- **Python** — application and data engineering
- **FastAPI** — service/API boundary
- **SQLite** — local zero-server relational database
- **SQLAlchemy** — database abstraction and ORM
- **Pydantic** — data contracts and validation
- **Pandas / Polars** — analytical processing where justified
- **pytest** — automated testing

Docker, CI/CD, PostgreSQL, and advanced ML are deferred until they provide measurable value.

---

## 5. Core Domain Entities

| Entity | Responsibility |
|---|---|
| `trip` | Central transportation event |
| `trip_conditions` | Time-indexed conditions associated with a trip |
| `driver_profile` | Driver attributes and historical evidence |
| `vehicle_profile` | Vehicle characteristics |
| `vehicle_maintenance` | Maintenance, repair, and failure events |
| `route` | Route identity and spatial context |
| `trip_prediction` | Immutable prediction records |
| `trip_outcome` | Observed post-trip reality |
| `prediction_error` | Prediction-vs-reality reconciliation and diagnosis |

Raw source records remain separate from the trusted core model.

---

## 6. Architectural Principles

### Raw data is preserved

Source observations should be retained as received whenever practical. Downstream interpretation must not silently rewrite source truth.

### Current state is different from history

Current observations answer **what is happening now**. Historical context answers **what happened under similar conditions**. They are joined explicitly for decision-making.

### Time alignment is semantic

Variables require different temporal treatment. Precipitation may require interval aggregation, while temperature may use a nearest observation or interpolation.

### Spatial alignment is explicit

An observation is useful only when its location can be meaningfully associated with the trip route or route segment.

### No unexplained magic scores

The system prefers evidence, distributions, sample sizes, and explicit calculations over opaque values such as `Driver Score = 83`.

### Feasibility before optimization

A recommendation must first be physically and operationally executable. Vehicle failure, for example, requires assessment of safety, severity, repair time, resources, alternatives, and future conditions before optimization.

### Prediction and reality are separate

```text
Prediction → Actual Outcome → Reconciliation → Diagnosis
```

Historical predictions remain immutable evidence of what the system believed at the time.

---

## 7. Repository Structure

The structure is derived from system responsibilities, not from arbitrary file categories.

```text
MOBILITY_LOGISTIC/
│
├── app/
│   ├── api/              # HTTP/API boundary
│   ├── core/             # configuration and shared contracts
│   ├── db/               # SQLAlchemy, sessions, models, repositories
│   ├── ingestion/        # source clients and ingestion workflows
│   ├── validation/       # schema and data-quality validation
│   ├── alignment/        # temporal and spatial alignment
│   ├── analytics/        # historical analysis and feature engineering
│   ├── decision/         # state, feasibility, prediction, explanation, recommendation
│   └── services/         # cross-module application orchestration
│
├── data/
│   ├── raw/              # source payloads / snapshots
│   ├── processed/        # validated/transformed datasets
│   └── synthetic/        # explicitly synthetic operational datasets
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
├── migrations/           # introduced when schema migration workflow is needed
├── .github/workflows/    # CI introduced after the executable foundation exists
│
├── pyproject.toml
├── .env.example
├── Dockerfile            # deferred until containerization is useful
└── README.md
```

Empty folders are not created merely for appearance. Each directory is introduced when its responsibility and first real artifact are defined.

---

## 8. Development Order

```text
1. Freeze data-source contracts
2. Freeze domain model
3. Design SQLite schema through SQLAlchemy
4. Create project/package structure
5. Build configuration layer
6. Build one ingestion client end-to-end
7. Preserve raw data
8. Validate data
9. Load trusted core entities
10. Implement time/spatial alignment
11. Build historical context retrieval
12. Implement current-state assessment
13. Implement feasibility checks
14. Build a transparent baseline predictor
15. Store predictions
16. Store actual outcomes
17. Reconcile prediction vs reality
18. Diagnose errors
19. Expose stable FastAPI endpoints
20. Add tests and observability
21. Migrate to PostgreSQL when justified
22. Introduce advanced ML / optimization only when baseline evidence supports it
```

---

## 9. Phase 1 Definition of Done

The first meaningful milestone is a working vertical slice:

```text
Real External Data
      ↓
Ingestion
      ↓
Validation
      ↓
SQLite Storage
      ↓
Context Construction
      ↓
Baseline Prediction / Decision
      ↓
API Output
      ↓
Stored Prediction
      ↓
Observed Outcome
      ↓
Reconciliation
```

A small complete slice is preferred over many unfinished modules.

---

## 10. Project Status

**Current status: Architecture → Data Model → Implementation Foundation**

- [x] Problem definition
- [x] System concept
- [x] Master architecture
- [x] Decision loop
- [x] Core entity list
- [x] SQLite decision for Phase 1
- [ ] Final domain/database schema
- [ ] Data-source contracts
- [ ] Project package structure
- [ ] First ingestion pipeline
- [ ] Validation layer
- [ ] Core storage
- [ ] Alignment engine
- [ ] Baseline predictor
- [ ] Reconciliation loop
- [ ] FastAPI
- [ ] Tests
- [ ] Production deployment

---

## 11. Engineering Rule

> **Do not code a component until its responsibility, inputs, outputs, dependencies, failure modes, and tests are understood.**

Architecture first. Evidence second. Implementation third.
