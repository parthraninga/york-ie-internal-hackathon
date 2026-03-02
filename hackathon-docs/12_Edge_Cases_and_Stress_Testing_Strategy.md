# 12 – Edge Cases & Stress Testing Strategy

## Purpose of This Document

This document defines:

- Edge cases the system must handle
- Failure scenarios
- Stress testing approach
- Scaling simulation strategy
- Pre-demo validation checklist

This ensures production-grade stability.

---

# 1. Edge Case Categories

We divide edge cases into:

1. Query-level edge cases
2. Data-level edge cases
3. Performance-level edge cases
4. Update-level edge cases

---

# 2. Query-Level Edge Cases

---

## 2.1 Misspelled Company Name

Example:
"Infosis reveneu"

Expected Behavior:
- Still match "Infosys"
- Fuzzy score compensates

Test:
Run trigram similarity-based query.

---

## 2.2 Numeric-Only Query

Example:
"40000 revenue"

Expected Behavior:
- Identify metric_value condition
- Return relevant metrics

Must not fail due to missing text match.

---

## 2.3 Very Long Query

Example:
"Which company had highest operating margin in Q2 FY25 and strong AI investments?"

Expected Behavior:
- Filter structured part
- Rank relevant sections
- Not crash
- Not sequential scan

---

## 2.4 No Match Query

Example:
"Quantum profit unicorn banana"

Expected Behavior:
- Return empty result
- Not throw error
- Not full table scan

---

## 2.5 Mixed Case & Special Characters

Example:
"infosys REVENUE!!!"

Expected Behavior:
- Normalize query
- Still return correct result

Handled by:
normalize_query() function + unaccent.

---

# 3. Data-Level Edge Cases

---

## 3.1 Missing Metric Value

If metric_value is NULL:
- Must not break ranking
- Use COALESCE where needed

---

## 3.2 Multiple Same Metrics Per Period

If multiple rows exist:
- Ensure composite index filtering
- Ensure proper ordering

---

## 3.3 Very Large Content Section

If section content > 10,000 characters:
- search_vector still generated
- GIN index still used
- Ranking stable

---

# 4. Performance-Level Edge Cases

---

## 4.1 Dataset Grows to 5,000+ Records

Test Plan:

1. Insert 5,000+ dummy sections.
2. Insert 5,000+ metrics.
3. Run hybrid query.
4. Verify execution time.

Goal:
Remain <150ms.

---

## 4.2 High Similarity Load

If many rows have similar text:

Ensure:
- Pre-filtering reduces candidate rows.
- similarity() applied only after index match.

---

## 4.3 Numeric Heavy Filtering

Example:
"Revenue > 0"

Should:
Use B-tree index.
Not sequential scan entire table.

---

# 5. Update-Level Edge Cases

---

## 5.1 Update Section Content

Test:

UPDATE sections
SET content = 'New AI strategy'
WHERE id = X;

Expected:
search_vector auto-updates.
Next search reflects change.

---

## 5.2 Delete Document

DELETE FROM documents WHERE id = X;

Expected:
All sections deleted via CASCADE.
All metrics deleted via CASCADE.
No orphan records.

---

# 6. Stress Testing Plan

Before demo:

1. Generate dummy dataset:
   - 5,000+ sections
   - 5,000+ metrics

2. Benchmark:
   EXPLAIN ANALYZE SELECT ...

3. Confirm:
   - Execution Time <150ms
   - No Seq Scan
   - Bitmap Index Scan present

---

# 7. Monitoring During Demo

If execution time is borderline:

- Reduce fuzzy weight.
- Ensure WHERE clause filters.
- Confirm LIMIT is applied.
- Check index existence.

---

# 8. Safety Checklist Before Final Demo

✓ 5,000+ test rows inserted
✓ Fuzzy search works
✓ Numeric filter works
✓ Section weighting works
✓ Recency boost works
✓ No sequential scan
✓ <150ms execution
✓ Updates reflect automatically

---

# 9. Final Strategic Insight

Most teams will not test edge cases.

We will.

This demonstrates:
- Engineering maturity
- Production thinking
- Database expertise

---

This concludes system design documentation.

Next step:

Generate MASTER_CURSOR_IMPLEMENTATION_PROMPT.md

This will instruct Cursor to read all files and generate full SQL implementation.