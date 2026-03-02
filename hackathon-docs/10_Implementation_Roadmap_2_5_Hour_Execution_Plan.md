# 10 – Implementation Roadmap (2.5 Hour Execution Plan)

## Purpose of This Document

This document converts:

Architecture → into execution steps.

It defines:

- What to build first
- What to build next
- What NOT to over-engineer
- How to stay ahead during evaluation

Time is limited.
Strategy matters.

---

# Total Duration: 2.5 Hours

Breakdown:

20 minutes → Planning
10 minutes → Clarification window
2 hours → Build
Final 30 minutes → Evaluation

We optimize for this timeline.

---

# Phase 1 – Planning (0–20 Minutes)

Goals:

1. Finalize schema design.
2. Finalize ranking formula.
3. Finalize required extensions.
4. Confirm indexing plan.

Deliverables:

- Raw SQL DDL outline
- List of extensions
- Ranking formula structure

DO NOT start coding blindly.

---

# Phase 2 – Foundation Setup (20–40 Minutes)

Tasks:

1. Enable extensions:
   - pg_trgm
   - unaccent
   - btree_gin

2. Create tables:
   - documents
   - sections
   - financial_metrics

3. Create foreign keys.

4. Add generated search_vector column.

Deliverable:

Database structure complete.

---

# Phase 3 – Indexing Setup (40–60 Minutes)

Tasks:

1. Create GIN index on search_vector.
2. Create GIN trigram index on sections.content.
3. Create B-tree index on financial_metrics.metric_value.
4. Create composite index on (company_name, metric_name, period_label).
5. Create index on documents.company_name.

Deliverable:

All performance structures ready.

---

# Phase 4 – Insert Sample Data (60–75 Minutes)

Tasks:

1. Insert structured financial metrics.
2. Insert sample document sections.
3. Insert at least 100+ rows for testing.
4. Optionally simulate 5,000+ records.

Deliverable:

Searchable dataset ready.

---

# Phase 5 – Implement Search Query (75–105 Minutes)

Tasks:

1. Write normalize_query() SQL function.
2. Write compute_relevance_score() function.
3. Write final search SELECT query.
4. Integrate:
   - ts_rank
   - similarity()
   - numeric bonus
   - section weight
   - recency boost

Deliverable:

Working hybrid search engine.

---

# Phase 6 – Add Numeric Filtering Logic (105–120 Minutes)

Tasks:

1. Parse numeric intent (basic logic).
2. Add metric_value filtering.
3. Boost numeric matches.
4. Test comparison queries.

Deliverable:

Structured + unstructured hybrid working.

---

# Phase 7 – Performance Testing (120–140 Minutes)

Tasks:

1. Run EXPLAIN ANALYZE.
2. Verify:
   - Bitmap Index Scan
   - No Seq Scan
   - <150ms execution time
3. Tune WHERE clauses if needed.

Deliverable:

Performance validated.

---

# Phase 8 – Demo Preparation (Final 30 Minutes)

Prepare explanation script:

1. Explain schema design.
2. Show enabled extensions.
3. Show ranking formula.
4. Run fuzzy query.
5. Run numeric query.
6. Show EXPLAIN ANALYZE.
7. Explain index usage.

Practice concise explanation.

---

# Critical Mistakes To Avoid

- Overcomplicating numeric parsing.
- Scoring entire table without filtering.
- Forgetting LIMIT.
- Using LIKE '%term%'.
- Writing ranking in application code.

Keep everything SQL-native.

---

# Winning Strategy

Most teams:
- Build search.
- Then try to optimize.

We:
- Build optimized structure first.
- Then plug search into it.

Architecture first.
Code second.

---

Next document will define:

- Demo strategy
- Judge Q&A preparation
- What to say in 5 minutes