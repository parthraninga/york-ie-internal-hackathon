# 02 – Document Intelligence Breakdown

## Purpose of This Document

This file converts the provided financial PDFs into a database intelligence model.

We are not treating them as files.
We are treating them as structured data sources.

---

# 1. Types of Documents Provided

We have 4 financial documents from different companies.

They contain:

- Financial tables
- Quarterly metrics
- CEO/CFO commentary
- Business highlights
- Client wins
- Awards & recognitions
- Balance sheets
- Segment breakdowns
- Guidance statements

---

# 2. Data Classification

We classify all content into 3 major categories:

---

## A. Structured Data

Highly organized, numeric, tabular data.

Examples:
- Revenue
- EBIT
- Profit After Tax
- EPS
- Margin %
- Segment Revenue
- Assets
- Liabilities
- Headcount
- Cash Flow
- Dividend

Characteristics:
- Numeric
- Period-based (Quarter, Year)
- Company-specific
- Comparable across companies

Database Implication:
→ Must be stored in a normalized structured table.
→ Must support numeric filtering and comparisons.
→ Must support range queries.

---

## B. Semi-Structured Data

Formatted text sections with meaningful grouping.

Examples:
- Financial Highlights bullet lists
- Segment Information
- Key Deal Wins
- Business Highlights
- Awards & Recognitions
- Guidance

Characteristics:
- Text-heavy
- Categorized by section
- Contains numeric references inside text

Database Implication:
→ Must be split into logical sections.
→ Must store section name.
→ Must assign weight to each section.
→ Must support text + numeric hybrid search.

---

## C. Unstructured Data

Narrative commentary and long text paragraphs.

Examples:
- CEO statements
- CFO commentary
- Testimonials
- Market outlook
- Risk statements
- Safe Harbor disclaimers

Characteristics:
- Long-form text
- Contextual meaning
- High semantic value

Database Implication:
→ Must support full-text search.
→ Must support fuzzy spelling.
→ Must support phrase ranking.
→ Should have lower weight than structured highlights.

---

# 3. Key Search Use Cases We Must Support

---

## Case 1 – Direct Metric Query

"Infosys revenue Q2"

System must:
- Identify company
- Identify period
- Retrieve revenue metric
- Rank correct result first

Requires:
- Structured metric storage
- Period filtering
- Text ranking

---

## Case 2 – Fuzzy Query

"Infosis reveneu growht"

System must:
- Tolerate spelling mistakes
- Still match "Infosys revenue growth"

Requires:
- pg_trgm extension
- similarity scoring
- Combined ranking logic

---

## Case 3 – Numeric Filtering

"Revenue above 40000 crore"

System must:
- Parse numeric intent
- Compare metric_value > 40000
- Return filtered result

Requires:
- Numeric index
- Range query support
- SQL-based ranking bonus

---

## Case 4 – Comparison Query

"Which company has higher operating margin?"

System must:
- Retrieve operating margin for all companies
- Compare values
- Return sorted result

Requires:
- Structured metrics table
- Aggregation
- SQL ORDER BY

---

## Case 5 – Section-Specific Query

"AI investments in Tech Mahindra"

System must:
- Identify section likely containing AI references
- Boost Business Highlights or Commentary sections

Requires:
- Section-aware weighting

---

# 4. Intelligence Requirements Derived From Documents

Based on analysis, system must:

- Separate raw document from sections
- Separate structured metrics from narrative
- Support hybrid scoring
- Store metadata (company, year, quarter)
- Support multi-company queries
- Support multi-period queries
- Support large dataset scaling

---

# 5. Data Modeling Implications

We must design:

1. documents table
2. sections table
3. financial_metrics table
4. search vector column
5. ranking functions

We cannot rely only on tsvector.
We must combine:

- Full-text search
- Trigram similarity
- Numeric filters
- Section weights
- Recency boost

---

# 6. Strategic Insight

Most teams will:
- Dump raw text into one table
- Add tsvector
- Use ts_rank
- Call it done

That approach will fail numeric queries and comparisons.

Our approach:
Structured + Semi-Structured + Unstructured unified under hybrid ranking model.

---

# 7. Summary

The PDFs are not documents.
They are structured intelligence containers.

Our database must reflect that structure.

Next file will define exact system functional and non-functional requirements.