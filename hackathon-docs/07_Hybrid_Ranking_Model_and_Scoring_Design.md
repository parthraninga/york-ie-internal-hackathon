# 07 – Hybrid Ranking Model & Scoring Design

## Purpose of This Document

This document defines:

- The ranking formula
- Each scoring component
- How SQL will compute final score
- Why hybrid scoring is superior to basic ts_rank

This is the intelligence engine.

---

# 1. Why Basic Full-Text Ranking Is Not Enough

If we only use:

ts_rank(search_vector, query)

Problems:

- No fuzzy spelling tolerance
- No numeric awareness
- No section importance
- No recency boost
- No comparison intelligence

We need hybrid ranking.

---

# 2. Hybrid Ranking Philosophy

Final Score must combine:

1. Text relevance
2. Fuzzy similarity
3. Numeric match bonus
4. Section importance
5. Recency boost

All computed inside SQL.

---

# 3. Final Ranking Formula

Final Score =

(Text Score * 0.50)
+ (Fuzzy Score * 0.20)
+ (Numeric Bonus * 0.15)
+ (Section Weight Multiplier * 0.10)
+ (Recency Boost * 0.05)

Weights can be tuned.

---

# 4. Component 1 – Text Score

Definition:
Full-text relevance score.

Computed using:
ts_rank(search_vector, query)

Purpose:
Measures keyword match density and importance.

Indexed via:
GIN index on search_vector.

---

# 5. Component 2 – Fuzzy Score

Definition:
Trigram similarity between content and query.

Computed using:
similarity(content, user_query)

Purpose:
Handles misspellings.

Example:
"Infosis" should match "Infosys".

Indexed via:
GIN trigram index.

---

# 6. Component 3 – Numeric Bonus

Definition:
If numeric constraint matches, boost score.

Example:
Query:
"Revenue above 40000"

If metric_value > 40000:
Add numeric bonus.

Why needed:
Text match alone does not ensure numeric correctness.

This ensures:
Correct financial rows rank above narrative sections.

---

# 7. Component 4 – Section Weight Multiplier

Each section has a predefined weight.

Examples:

- Financial Highlights → 1.5
- Key Metrics → 1.4
- CEO Commentary → 1.2
- Awards → 1.0
- Disclaimer → 0.5

We multiply text score by section_weight.

This ensures:
Important sections rank higher.

---

# 8. Component 5 – Recency Boost

More recent documents should rank slightly higher.

Formula example:

1 / (1 + age_in_days / 365)

This gives:
Small advantage to recent results.

Optional but impressive for judges.

---

# 9. Full SQL Ranking Strategy (Conceptual)

Inside SELECT:

Compute:

text_score
fuzzy_score
numeric_bonus
section_weight
recency_factor

Then:

final_score =
(text_score * 0.5)
+ (fuzzy_score * 0.2)
+ (numeric_bonus * 0.15)
+ (section_weight * 0.1)
+ (recency_factor * 0.05)

ORDER BY final_score DESC

LIMIT 10

---

# 10. Filtering Strategy Before Ranking

To improve performance:

1. If company detected → filter first.
2. If metric detected → filter financial_metrics first.
3. If numeric detected → apply numeric filter before scoring.

Never score entire table.

---

# 11. Why This Model Wins

Most teams:
- Use ts_rank only.
- No fuzzy support.
- No numeric awareness.
- No section weighting.

Our model:
Hybrid intelligence.
Multi-factor scoring.
SQL-native.
Performance-aware.

This is enterprise-grade.

---

# 12. Explainability (Important for Judges)

We must be able to explain:

- Why this result ranked higher
- How numeric bonus works
- Why section weight matters
- Why fuzzy score saved misspelled query

Transparency impresses judges.

---

# 13. Tuning Strategy

If performance is slow:

- Reduce fuzzy weight
- Add pre-filtering
- Narrow search space using WHERE
- Tune GIN index parameters

Ranking formula must remain inside SQL.

---

Next document defines:

- Real-time update strategy
- Trigger vs Generated column choice
- How to guarantee no manual reindex