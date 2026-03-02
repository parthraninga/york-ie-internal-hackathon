# 01 – Problem Statement & Strategic Context

## Hackathon Theme
Postgres-Native Document Intelligence Engine

---

# 1. Executive Overview

We are required to build a **fully PostgreSQL-native document intelligence engine**.

We are given 4 financial PDF documents containing:

- Structured financial tables
- Semi-structured highlights
- Long unstructured executive commentary
- Numeric performance data
- Company comparison material

The system must:

1. Parse documents
2. Store raw and processed data in PostgreSQL
3. Create index-backed retrieval structures
4. Compute ranking entirely in SQL
5. Return ordered results in under 150ms
6. Handle fuzzy spelling errors
7. Support real-time update reflection (no manual reindex)

---

# 2. What This Hackathon Is Actually Testing

This is NOT a PDF parsing contest.

This is testing:

- Deep PostgreSQL knowledge
- Index architecture design
- Ranking algorithm design in SQL
- Performance engineering
- Query planner awareness
- Extension usage
- Schema modeling maturity
- Real-time indexing strategy

If indexing strategy is unclear in 30 minutes → team is behind.

---

# 3. Hard Constraints (Non-Negotiable)

### Intelligence must live inside Postgres

Forbidden:
- ElasticSearch
- Redis
- External vector databases
- Application-side ranking
- In-memory filtering

Allowed:
- PostgreSQL extensions
- SQL functions
- Triggers
- Generated columns
- JSONB
- GIN / GiST indexes

---

# 4. Mandatory Requirements

- Ranking must be computed in SQL
- Must scale to 5,000+ records
- Query latency under 150ms
- Minimum 3 Postgres extensions
- At least 3 handwritten SQL functions
- Schema must be raw SQL (no ORM DDL)
- Must tolerate spelling mistakes
- Search results must auto-update when documents change

---

# 5. Evaluation Format

Each team gets 5 minutes.

Must demonstrate:

1. Schema DDL
2. Enabled extensions:
   SELECT extname FROM pg_extension;
3. Live search query
4. EXPLAIN ANALYZE
5. Explanation of ranking logic
6. Explanation of indexing strategy

---

# 6. Strategic Interpretation

We are not building:
- A simple full-text search
- A keyword search engine

We are building:
A multi-layer SQL-native intelligence engine.

This means:
- Hybrid ranking
- Numeric-aware scoring
- Section-aware boosting
- Real-time index refresh
- Performance-optimized query plans

---

# 7. Success Criteria

We win if:

- Queries execute in <150ms
- EXPLAIN shows GIN/Bitmap Index Scan
- Ranking logic is explainable
- Schema is clean and normalized
- Fuzzy queries work
- Numeric filtering works
- Updates reflect automatically

---

# 8. Our Positioning Strategy

We will differentiate by:

- Hybrid ranking model (ts_rank + trigram + numeric boost)
- Section-weight scoring
- Structured + unstructured unified search
- Clean schema separation
- No sequential scans
- Generated tsvector columns

---

This document defines the strategic direction.

Next file will break down document structure deeply and extract system-level implications.