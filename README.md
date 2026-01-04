# Ticketmaster Discovery API — Analytics Engineering Project

## Project Overview

This project is an end-to-end analytics engineering use case built on top of the **Ticketmaster Discovery API**.  
The goal is to design a reliable data pipeline and analytical model that enables **Account Managers** to monitor and prioritize venues based on their upcoming activity over the next 30 days.

The project covers the full data lifecycle:
- Data extraction via API (Python)
- Loading into a cloud data warehouse (BigQuery)
- Transformation and modeling using dbt
- Data consumption through a Metabase dashboard

---

## Architecture Overview

![Ticketmaster Data Pipeline Architecture](docs/architecture.png)

## Data Source — Ticketmaster Discovery API

The data is extracted from the Ticketmaster Discovery API, which allows access to:
- Events
- Venues
- Attractions
- Classifications (segments, genres, subgenres)

### API Constraints

The Discovery API comes with several limitations:
- Limited access to ticket pricing data (incomplete or missing for many events)
- Rate limits and pagination constraints
- A practical limit on the number of events that can be collected efficiently

To work within these constraints, the scope of the project was intentionally limited to:
- Events occurring in the next 30 days
- Geographic focus on the United States and Canada

This ensured data freshness while keeping the pipeline realistic and production-oriented.

---

## Data Pipeline Architecture

### 1. Extraction (Python)
- Data is extracted from the Ticketmaster Discovery API using Python
- Separate endpoints are queried for:
  - Events
  - Venues
  - Attractions
  - Classifications
- The extraction logic handles pagination and API limits
- Raw data is stored without transformations to preserve source fidelity

### 2. Loading (BigQuery)
- Raw datasets are loaded into BigQuery
- Each API entity is stored in its own raw table:
  - `raw_events`
  - `raw_venues`
  - `raw_attractions`
  - `raw_classifications`

BigQuery is used as the central analytical warehouse.

---

## Transformations & Data Modeling (dbt)

Transformations are implemented using dbt, following analytics engineering best practices.

### Staging Layer
The first layer consists of **staging models**:
- `stg_events`
- `stg_venues`
- `stg_attractions`
- `stg_classifications`

Key principles:
- Light transformations only (renaming, type casting)
- One-to-one mapping with raw sources
- No business logic at this stage

### Mart Layer
On top of staging models, a **star-schema–oriented mart layer** is built.

#### Fact Table
- **`fct_events`**
  - One row per event
  - Event-level grain
  - Includes:
    - Venue reference
    - Attraction reference
    - Classification attributes
    - Derived temporal features (hour, day of week, sorted weekday)

Although this is not a transactional fact table with monetary measures, it is modeled as a fact table because it represents **business activity occurrences** (events) and supports aggregation and analysis.

#### Dimension Tables
- **`dim_venues`**
- **`dim_attractions`**
- **`dim_classifications`**

These dimensions enrich the event data and allow slicing by:
- Venue location
- Attraction popularity
- Segment / genre

Special care was taken to:
- Avoid fan-out joins
- Control grain consistency
- Ensure event-level uniqueness despite classification complexity

---

## Dashboard & Business Use Case (Metabase)

### Target Users
**Account Managers at Ticketmaster**

### Business Objective
Enable Account Managers to:
- Monitor venue activity over the next 30 days
- Identify high-activity venues
- Prioritize efforts on accounts with the highest operational impact

### Dashboard Focus
- Number of upcoming events
- Daily distribution of events
- Events by hour and by day of week
- Ranking of venues by activity volume

Due to API limitations, ticket price data could not be reliably used, which prevents revenue-based prioritization.  
As a result, **event volume** is used as the main proxy to assess account importance.

---

## Key Learnings & Trade-offs
- API limitations strongly influence data modeling decisions
- Not all “fact tables” need to be transactional to be analytically valuable
- Classification data from Ticketmaster requires careful handling to avoid row duplication
- Designing with a clear business user in mind (Account Managers) helps guide both modeling and dashboard structure

---

## Tech Stack
- **Python** — API extraction
- **BigQuery** — Data warehouse
- **dbt** — Transformations and modeling
- **Metabase** — Data visualization

---

## Key Insights

- **Event activity is highly concentrated during evenings and weekends:**
  - Around 65% of events take place between 7pm and 9pm
  - Nearly 50% of events occur on Fridays and Saturdays

- **Event volume is largely driven by three main segments:**
  - Arts & Theatre
  - Music
  - Sports

- As expected, event activity is concentrated in the largest US and Canadian states

### Additional Observations
- The venues with the highest number of upcoming events include several tourist and experiential activities, highlighting that Ticketmaster manages ticketing beyond traditional entertainment use cases
