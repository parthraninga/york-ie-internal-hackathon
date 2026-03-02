# 11 – Demo Strategy & Judge Q&A Preparation

## Purpose of This Document

This document prepares us for:

- The 5-minute live evaluation
- Schema explanation
- Index explanation
- Ranking explanation
- Handling judge questions confidently

In hackathons, presentation clarity wins.

---

# 1. 5-Minute Demo Structure (Strict Order)

You must control the flow.

Follow this exact structure:

---

## Step 1 – Show Extensions (30 seconds)

Run:

SELECT extname FROM pg_extension;

Explain:

"We use pg_trgm for fuzzy search, unaccent for normalization, and btree_gin for hybrid indexing support."

Keep it short and confident.

---

## Step 2 – Show Schema (60 seconds)

Briefly show:

- documents table
- sections table
- financial_metrics table

Explain:

"We separate structured and unstructured data to enable numeric filtering and hybrid ranking."

Important:
Do NOT read all columns.
Explain design reasoning.

---

## Step 3 – Show Indexes (45 seconds)

Show:

- GIN index on search_vector
- GIN trigram index
- B-tree numeric index
- Composite filtering index

Explain:

"All ranking is index-backed. We avoid sequential scans."

---

## Step 4 – Run Normal Query (45 seconds)

Example:

Search:
"Infosys revenue Q2"

Explain result.

Highlight:
- Correct company
- Correct metric
- Proper ranking

---

## Step 5 – Run Fuzzy Query (45 seconds)

Example:

"Infosis reveneu"

Show:

Correct result still ranks first.

Explain:

"Trigram similarity handles spelling mistakes."

Judges love this demo.

---

## Step 6 – Run Numeric Query (45 seconds)

Example:

"Revenue above 40000"

Show:

Correct filtering.

Explain:

"Numeric filtering is index-backed using B-tree."

---

## Step 7 – Show EXPLAIN ANALYZE (60 seconds)

Run:

EXPLAIN ANALYZE SELECT ...

Highlight:

- Bitmap Index Scan
- No Seq Scan
- Execution Time <150ms

Say:

"We filter early, then rank only candidate rows."

Confident. Calm.

---

# 2. Questions Judges May Ask

Prepare answers in advance.

---

## Q1: Why not use only ts_rank?

Answer:

"ts_rank does not handle numeric constraints or fuzzy spelling. Our hybrid model combines structured numeric intelligence with trigram similarity for robustness."

---

## Q2: How do you guarantee real-time updates?

Answer:

"We use GENERATED STORED tsvector columns. When content updates, search_vector auto-updates and GIN index reflects immediately. No manual reindex required."

---

## Q3: How do you scale to 5,000+ records?

Answer:

"All filtering is index-backed. We use composite B-tree for structured metrics and GIN for full-text search. EXPLAIN shows no sequential scans."

---

## Q4: What happens if dataset grows to 100,000 rows?

Answer:

"GIN indexes scale logarithmically. We also filter before ranking, so scoring cost remains bounded."

---

## Q5: Why separate sections from documents?

Answer:

"Section separation enables section-weight ranking and avoids scoring entire documents unnecessarily."

---

# 3. What Impresses Judges Most

1. Clean schema separation.
2. Hybrid ranking explanation.
3. Fuzzy query working live.
4. Numeric filtering working live.
5. EXPLAIN without sequential scan.
6. Confident explanation of index usage.

---

# 4. What To Avoid During Demo

- Reading code line-by-line.
- Over-explaining ranking math.
- Apologizing.
- Running slow queries.
- Showing errors.

Stay controlled and precise.

---

# 5. Final Demo Checklist

Before presenting, confirm:

✓ Extensions enabled
✓ Tables created
✓ Indexes created
✓ Sample data loaded
✓ Fuzzy search works
✓ Numeric filter works
✓ EXPLAIN shows index usage
✓ Query under 150ms

---

Next document will define:

- Edge cases
- Stress testing plan
- Data scaling simulation