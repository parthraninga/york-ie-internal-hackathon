# 06 – PostgreSQL Extensions & Indexing Strategy

## Purpose of This Document

This document defines:

- Required PostgreSQL extensions
- Why each extension is necessary
- Exact indexing strategy
- How we achieve <150ms latency
- How we avoid sequential scans

This is the performance backbone of the system.

---

# 1. Required PostgreSQL Extensions

Minimum requirement: 3 extensions.

We will use:

1. pg_trgm
2. unaccent
3. btree_gin

Optional (for testing/performance insight):
4. pg_stat_statements

---

# 2. Extension 1 – pg_trgm

Purpose:
Provides trigram-based similarity matching.

Used for:
- Fuzzy spelling tolerance
- similarity() scoring
- % operator matching

Example:
"Infosis" → matches "Infosys"

Why needed:
Full-text search does NOT handle misspellings.
Trigram similarity does.

Impact:
Enables fuzzy ranking component.

---

# 3. Extension 2 – unaccent

Purpose:
Removes accents and normalizes text.

Example:
"Mahíndra" → "Mahindra"

Why needed:
Improves search normalization.
Prevents mismatch due to accent differences.

Used in:
normalize_query() SQL function.

---

# 4. Extension 3 – btree_gin

Purpose:
Allows B-tree operators inside GIN indexes.

Why useful:
Enables hybrid index support.
Improves multi-column index flexibility.

Supports:
Combined structured + text filtering.

---

# 5. Indexing Strategy Overview

We will create indexes at 3 levels:

1. Full-text search index
2. Fuzzy matching index
3. Numeric and filtering indexes

---

# 6. Full-Text Index

Table: sections

Column: search_vector

Index Type:
GIN

Purpose:
Accelerates full-text search.

Expected Plan:
Bitmap Index Scan

Why:
ts_rank requires GIN-backed tsvector.

---

# 7. Trigram Index

Table: sections

Column: content

Index Type:
GIN with gin_trgm_ops

Purpose:
Accelerates similarity() and % operator.

Used for:
Fuzzy matching.

Expected Plan:
Bitmap Index Scan

---

# 8. Numeric Index

Table: financial_metrics

Column: metric_value

Index Type:
B-tree

Purpose:
Accelerates numeric range queries.

Example:
metric_value > 40000

Expected Plan:
Index Range Scan

---

# 9. Composite Filtering Index

Table: financial_metrics

Columns:
(company_name, metric_name, period_label)

Index Type:
B-tree composite

Purpose:
Fast filtering before ranking.

Example:
WHERE company_name = 'Infosys'
AND metric_name = 'Revenue'

This ensures:
Minimal rows before scoring.

---

# 10. Company Filter Index

Table: documents

Column:
company_name

Index Type:
B-tree

Purpose:
Fast pre-filter by company.

---

# 11. Performance Strategy

To guarantee <150ms:

1. Filter early using WHERE
2. Use indexed columns in WHERE
3. Avoid applying functions on indexed columns
4. Apply LIMIT early
5. Avoid sequential scans

EXPLAIN ANALYZE must show:

- Bitmap Index Scan
- Index Cond
- No Seq Scan

---

# 12. Avoiding Sequential Scans

We avoid:

- LIKE '%term%'
- Lower(content) in WHERE
- Non-indexed computed filtering

Instead:

- Use search_vector @@ query
- Use content % query (trigram operator)
- Use numeric index filtering

---

# 13. Real-Time Update Guarantee

Because:

search_vector is GENERATED STORED

When content changes:
- search_vector updates automatically
- GIN index updates automatically

No manual REINDEX required.

Constraint satisfied.

---

# 14. Scalability Assurance

Even with 5,000+ documents:

- GIN handles text search efficiently
- Numeric B-tree handles range queries
- Composite indexes reduce row scanning

Architecture is scalable.

---

# 15. Why This Strategy Is Stronger Than Others

Most teams:
- Only create one GIN index
- Only use ts_rank
- Ignore numeric indexing
- Ignore composite filtering

Our approach:
Multi-layer indexing.

Structured + Unstructured unified.

Enterprise-grade database engineering.

---

Next document defines the hybrid ranking model in detail.