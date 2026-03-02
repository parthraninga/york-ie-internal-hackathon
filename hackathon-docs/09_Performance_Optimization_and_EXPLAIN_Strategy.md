# 09 – Performance Optimization & EXPLAIN Strategy

## Purpose of This Document

This document defines:

- How we guarantee <150ms query latency
- How we avoid sequential scans
- How to structure WHERE clauses
- How to read and explain EXPLAIN ANALYZE output
- How to handle 5,000+ records efficiently

This document prepares us for live evaluation.

---

# 1. Performance Target

Query latency must be:

< 150ms

Even with:

- Fuzzy matching
- Hybrid ranking
- Numeric filtering
- 5,000+ records

Performance must be consistent.

---

# 2. Golden Rule of Performance

Filter first.
Rank later.

Never compute ranking across entire dataset.

Always narrow down rows using indexed WHERE clauses.

---

# 3. Query Execution Strategy

Correct order:

1. Detect company in query → filter documents first.
2. Detect metric in query → filter financial_metrics first.
3. Apply numeric constraint → filter using B-tree index.
4. Only then compute ranking score.

This ensures minimal rows are scored.

---

# 4. Avoiding Sequential Scans

Sequential scan occurs when:

- No suitable index
- Function applied to indexed column
- Too many rows selected

We must avoid:

WHERE LOWER(content) LIKE '%term%'
WHERE CAST(metric_value AS TEXT) LIKE '%40000%'

These break indexes.

Instead use:

search_vector @@ to_tsquery(...)
content % user_query
metric_value > 40000

---

# 5. Expected EXPLAIN ANALYZE Output

For text search:

Expect:
Bitmap Index Scan on sections_search_vector_idx
Bitmap Heap Scan
Index Cond: (search_vector @@ query)

For trigram:

Expect:
Bitmap Index Scan using sections_content_trgm_idx
Recheck Cond

For numeric:

Expect:
Index Scan using financial_metrics_metric_value_idx

We must NOT see:
Seq Scan on sections
Seq Scan on financial_metrics

If we see Seq Scan → performance problem.

---

# 6. Using LIMIT Effectively

Always apply:

LIMIT 10

Early in query.

This helps PostgreSQL stop scoring unnecessary rows.

---

# 7. Composite Filtering Optimization

If query mentions:

"Infosys revenue"

We should:

WHERE company_name = 'Infosys'
AND metric_name = 'Revenue'

This uses composite B-tree index.

Reduces row count drastically.

---

# 8. Hybrid Search Optimization

Instead of:

Ranking all sections

Do:

WHERE search_vector @@ query
OR content % user_query

This reduces candidate set.

Then compute score only for matched rows.

---

# 9. Cost Control Strategy

If fuzzy matching slows query:

We can:

- Reduce trigram similarity threshold
- Apply fuzzy only when full-text returns low results
- Limit trigram evaluation to pre-filtered rows

Never compute similarity across entire table.

---

# 10. Testing at Scale

Before demo:

Insert 5,000+ dummy records.

Test:

SELECT search query

Measure:

Execution Time in EXPLAIN ANALYZE.

Ensure:

< 150ms.

---

# 11. Reading EXPLAIN ANALYZE

Important fields:

- Execution Time
- Planning Time
- Rows Removed by Filter
- Index Cond
- Bitmap Index Scan

Explain to judges:

"We see Bitmap Index Scan using GIN index, which ensures logarithmic lookup rather than sequential scanning."

Clear explanation = strong impression.

---

# 12. Performance Checklist Before Demo

✓ GIN index exists
✓ Trigram index exists
✓ Numeric B-tree index exists
✓ Composite filtering index exists
✓ No Seq Scan in plan
✓ Execution Time < 150ms
✓ LIMIT applied

---

# 13. Why This Strategy Wins

Most teams:
- Focus on ranking
- Ignore execution plan
- Forget index conditions

We:
- Design query around index usage
- Control row count early
- Guarantee consistent performance

This is database engineering thinking.

---

Next document defines:

- 2.5-hour build execution plan
- Implementation sequencing
- Task breakdown for Cursor/Copilot