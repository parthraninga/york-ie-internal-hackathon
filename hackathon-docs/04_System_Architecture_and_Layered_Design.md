# High-Level Architecture

## Philosophy

We are not building search.
We are building a SQL-native intelligence engine.

---

## Core Layers

1. Storage Layer
   - documents table
   - sections table
   - financial_metrics table

2. Index Layer
   - Full-text GIN index
   - Trigram index
   - Composite indexes

3. Ranking Layer
   - ts_rank
   - similarity()
   - numeric boost
   - section weight
   - recency weight

4. Query Execution Layer
   - Single SELECT computing final score
   - ORDER BY score DESC
   - LIMIT applied early