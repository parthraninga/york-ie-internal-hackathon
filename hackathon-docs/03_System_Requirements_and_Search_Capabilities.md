# 03 – System Requirements & Search Capabilities

## Purpose of This Document

This document defines:

- What the system must support
- What queries must work
- What performance targets must be met
- What architectural guarantees must be provided

This is the functional contract of our engine.

---

# 1. Functional Requirements

---

## 1.1 Company-Specific Queries

The system must support:

- "Infosys revenue Q2"
- "Tech Mahindra EBIT margin"
- "TCS dividend declared"

Requirements:
- Company recognition
- Metric recognition
- Period recognition
- Ranked result ordering

---

## 1.2 Numeric Filtering Queries

Examples:

- "Revenue above 40000"
- "Operating margin greater than 20%"
- "EPS below 15"

System must:
- Extract numeric intent
- Perform SQL numeric comparison
- Use indexed metric_value column
- Boost matching results in ranking

This cannot be done using text-only search.

---

## 1.3 Fuzzy Spelling Tolerance

Examples:

- "Infosis reveneu"
- "Tek Mahindr EBIT"
- "operating margn"

System must:
- Detect similarity using trigram
- Combine similarity score with ts_rank
- Still rank correct company at top

This requires:
- pg_trgm extension
- similarity() scoring
- Hybrid ranking formula

---

## 1.4 Section-Aware Queries

Examples:

- "AI investments"
- "deal wins"
- "CEO commentary"
- "guidance for FY26"

System must:
- Detect relevant section
- Apply section weight multiplier
- Rank Business Highlights higher than disclaimers

Requires:
- section_name column
- section_weight column
- Weighted ranking logic

---

## 1.5 Comparison Queries

Examples:

- "Which company has highest revenue?"
- "Compare operating margin across companies"

System must:
- Aggregate financial_metrics
- Order results by metric_value
- Support GROUP BY and ORDER BY

Requires:
- Clean structured metrics table
- Numeric indexing

---

## 1.6 Real-Time Update Handling

If:
- A section is updated
- A metric is modified

Then:
- Search results must reflect immediately
- No manual REINDEX
- No refresh command

Requires:
- Generated tsvector OR trigger-based refresh
- Proper index architecture

---

# 2. Non-Functional Requirements

---

## 2.1 Performance Target

Query latency < 150ms

Even with:
- 5,000+ records
- Fuzzy matching
- Hybrid scoring

Requires:
- GIN index
- Composite indexes
- Pre-filtering
- No sequential scan

---

## 2.2 Scalability Target

System must handle:
- 5,000+ documents
- Thousands of sections
- Thousands of metrics

Design must:
- Avoid full table scans
- Avoid heavy runtime computation
- Use indexed filtering

---

## 2.3 SQL-Only Ranking

All ranking must occur inside SQL.

Prohibited:
- Sorting in application code
- Post-query scoring
- External ranking service

Allowed:
- SQL functions
- ts_rank
- similarity
- Custom scoring logic

---

## 2.4 Clean Schema Design

Schema must:
- Be normalized
- Be written in raw SQL
- Avoid ORM-generated DDL
- Separate structured and unstructured data

---

# 3. Derived Technical Capabilities

To satisfy requirements, system must implement:

1. Full-text search (tsvector)
2. Fuzzy matching (pg_trgm)
3. Numeric filtering (indexed numeric columns)
4. Section-based ranking
5. Recency-based ranking
6. Hybrid scoring model

---

# 4. Core Search Engine Formula

The search engine must combine:

- Text match score
- Fuzzy similarity score
- Numeric match bonus
- Section weight multiplier
- Recency bonus

Final score must be computed in SQL.

---

# 5. Design Philosophy

We are not building:
Keyword search.

We are building:
SQL-native hybrid relevance engine.

The difference:
- Numeric intelligence included
- Section semantics included
- Weighted ranking included
- Performance-optimized

---

# 6. Acceptance Criteria

System is considered complete if:

- Fuzzy queries work
- Numeric filters work
- Ranking is explainable
- EXPLAIN ANALYZE shows index usage
- Query latency <150ms
- Updates reflect automatically
- Schema is clean and structured

---

Next document will define full architecture and system layering.