# 08 – Real-Time Update & Data Consistency Strategy

## Purpose of This Document

This document defines:

- How search results reflect updates immediately
- How we avoid manual re-indexing
- How data consistency is maintained
- Why our approach satisfies hackathon constraints

This is a critical evaluation requirement.

---

# 1. Hackathon Constraint

"When a document is updated, search results must reflect changes automatically. Manual re-index commands are not allowed."

This means:

- No manual REFRESH MATERIALIZED VIEW
- No REINDEX command
- No scheduled re-index job
- No application-layer rebuild

The database must update search structures automatically.

---

# 2. Core Strategy

We will use:

GENERATED STORED tsvector columns.

Example concept:

search_vector TSVECTOR
GENERATED ALWAYS AS (
    to_tsvector('english', content)
) STORED

This means:

- When content changes
- PostgreSQL automatically recomputes search_vector
- GIN index automatically updates
- No manual action required

This satisfies constraint fully.

---

# 3. Why Generated Column Instead of Trigger?

Alternative approach:
Use BEFORE UPDATE trigger to recompute tsvector.

Problems with triggers:
- More complex
- Slight overhead
- Harder to explain in 5-minute demo
- Can be error-prone

Generated column is:
- Cleaner
- Declarative
- Easier to explain
- Automatically maintained

Judges prefer simplicity.

---

# 4. What Happens When Content Updates?

Scenario:

UPDATE sections
SET content = 'Updated AI investment strategy'
WHERE id = 10;

Immediately:

- search_vector auto-updates
- GIN index reflects change
- Next query returns updated ranking

No REINDEX needed.

---

# 5. What Happens When Financial Metric Updates?

Scenario:

UPDATE financial_metrics
SET metric_value = 45000
WHERE metric_name = 'Revenue'
AND company_name = 'Infosys';

Immediately:

- B-tree index updated
- Numeric filtering reflects new value
- Ranking recalculates in next SELECT

Again, no manual refresh required.

---

# 6. Handling Insertions

When new document inserted:

1. Insert into documents
2. Insert into sections
3. Insert into financial_metrics

search_vector auto-computed
Indexes auto-maintained

System instantly searchable.

---

# 7. Data Consistency Guarantees

Because of:

- Foreign keys with ON DELETE CASCADE
- Generated columns
- Indexed numeric fields

We ensure:

- No orphaned sections
- No orphaned metrics
- Clean deletion
- Clean updates

---

# 8. Why This Is Stronger Than Materialized View Approach

If we used materialized view:

- Would require manual refresh
- Violates hackathon constraint
- Adds complexity

Our design:
Direct indexing.
Always current.
No refresh needed.

---

# 9. Scalability Implication

Even with 5,000+ records:

- Updates affect only modified rows
- GIN updates are incremental
- No full-table reprocessing

Efficient and scalable.

---

# 10. Demo Explanation Strategy

During demo:

Explain clearly:

"We use generated tsvector columns. When content updates, the search vector is recalculated automatically, and GIN index updates instantly. Therefore, we require no manual reindexing."

Short.
Confident.
Technical.

---

# 11. Summary

We have satisfied:

- Real-time update requirement
- No manual re-index requirement
- Clean automatic consistency
- Scalable indexing approach

Next document will define performance optimization and EXPLAIN strategy.