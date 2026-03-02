# 05 – Database Schema Detailed Design

## Purpose of This Document

This document defines:

- All core tables
- Columns and data types
- Relationships
- Design reasoning
- Indexing implications

This schema must support:

- Structured search
- Unstructured search
- Hybrid ranking
- Numeric filtering
- Comparison queries
- Real-time updates

---

# 1. Core Design Principles

1. Separate structured and unstructured data.
2. Avoid storing everything in one table.
3. Make numeric queries index-friendly.
4. Make text search GIN-backed.
5. Keep schema normalized and clean.
6. Enable section-level ranking.

---

# 2. Table 1 – documents

Purpose:
Stores one row per document.

Example:
- Infosys Q2 FY25 Report
- Tech Mahindra Q3 FY26 Press Release

---

## Columns

id (BIGSERIAL PRIMARY KEY)

company_name (TEXT NOT NULL)

document_type (TEXT NOT NULL)
Examples:
- "Quarterly Results"
- "Press Release"

quarter (TEXT)
Example:
- "Q1"
- "Q2"
- "Q3"

financial_year (TEXT)
Example:
- "FY25"

raw_text (TEXT NOT NULL)

metadata (JSONB)
Used for flexible document attributes.

created_at (TIMESTAMP DEFAULT NOW())

updated_at (TIMESTAMP DEFAULT NOW())

---

## Why This Table Exists

- Allows company-based filtering
- Allows period-based filtering
- Stores raw document for audit
- Provides foreign key anchor for sections and metrics

---

# 3. Table 2 – sections

Purpose:
Stores logical subdivisions of each document.

Examples:
- "Financial Highlights"
- "CEO Statement"
- "Business Highlights"
- "Awards"
- "Segment Revenue"
- "Balance Sheet"

---

## Columns

id (BIGSERIAL PRIMARY KEY)

document_id (BIGINT REFERENCES documents(id) ON DELETE CASCADE)

section_name (TEXT NOT NULL)

section_weight (NUMERIC DEFAULT 1.0)

content (TEXT NOT NULL)

search_vector (TSVECTOR GENERATED ALWAYS AS
    to_tsvector('english', content)
) STORED

created_at (TIMESTAMP DEFAULT NOW())

---

## Why section_weight?

Some sections are more important.

Example weighting strategy:

- Financial Highlights → 1.5
- Key Metrics → 1.4
- CEO Commentary → 1.2
- Awards → 1.0
- Disclaimer → 0.5

This enables ranking boost logic.

---

## Why Generated search_vector?

- Auto-updates when content changes
- No manual reindex required
- Satisfies real-time update constraint

---

# 4. Table 3 – financial_metrics

Purpose:
Stores structured numeric data.

Each row represents:

One metric
For one company
For one period

---

## Columns

id (BIGSERIAL PRIMARY KEY)

document_id (BIGINT REFERENCES documents(id) ON DELETE CASCADE)

company_name (TEXT NOT NULL)

metric_name (TEXT NOT NULL)
Examples:
- "Revenue"
- "Operating Margin"
- "EBIT"
- "EPS"
- "Net Profit"

metric_value (NUMERIC NOT NULL)

currency (TEXT)
Examples:
- "INR"
- "USD"

period_label (TEXT)
Examples:
- "Q2 FY25"
- "FY26 H1"

created_at (TIMESTAMP DEFAULT NOW())

---

## Why Separate Table?

Because we need:

- Range queries
- Comparison queries
- Aggregation
- Sorting by value
- Numeric filtering

Text-only search cannot handle this correctly.

---

# 5. Optional Table – query_logs (for testing)

Purpose:
Performance benchmarking and testing.

Columns:
- id
- query_text
- execution_time_ms
- created_at

Not mandatory for hackathon but useful for performance tuning.

---

# 6. Relationships

documents (1)
    ↓
sections (many)

documents (1)
    ↓
financial_metrics (many)

This ensures:

- Clean deletion cascade
- Clear data grouping
- Structured retrieval

---

# 7. Schema Strength

This design enables:

- Hybrid ranking
- Numeric comparisons
- Section-aware boosting
- Clean indexing
- Efficient filtering
- Scalable growth

---

# 8. Why This Is Judge-Impressive

Because:

- It is normalized
- It separates concerns
- It supports multiple query types
- It looks like enterprise-grade architecture
- It goes beyond basic full-text search

---

Next document will define:

- Required PostgreSQL extensions
- Index strategy
- Why each index exists
- How to guarantee <150ms performance