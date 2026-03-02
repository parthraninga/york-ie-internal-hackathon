# MASTER IMPLEMENTATION PROMPT
# Postgres-Native Document Intelligence Engine

You are a senior PostgreSQL systems architect.

You have access to all project documentation files:

01_Problem_Statement_and_Context.md  
02_Document_Intelligence_Breakdown.md  
03_System_Requirements_and_Search_Capabilities.md  
04_System_Architecture_and_Layered_Design.md  
05_Database_Schema_Detailed_Design.md  
06_Postgres_Extensions_and_Indexing_Strategy.md  
07_Hybrid_Ranking_Model_and_Scoring_Design.md  
08_RealTime_Update_and_Data_Consistency_Strategy.md  
09_Performance_Optimization_and_EXPLAIN_Strategy.md  
10_Implementation_Roadmap_2_5_Hour_Execution_Plan.md  
11_Demo_Strategy_and_Judge_QA_Preparation.md  
12_Edge_Cases_and_Stress_Testing_Strategy.md  

Your task is to:

--------------------------------------------------
PHASE 1 – VALIDATE DESIGN
--------------------------------------------------

1. Read all documents carefully.
2. Validate consistency across:
   - Schema
   - Ranking model
   - Indexing strategy
   - Performance strategy
3. Suggest improvements if necessary.

--------------------------------------------------
PHASE 2 – GENERATE FULL PRODUCTION SQL
--------------------------------------------------

Generate a complete SQL script that includes:

1. Extension enablement:
   - pg_trgm
   - unaccent
   - btree_gin

2. Table creation:
   - documents
   - sections
   - financial_metrics

3. Generated tsvector column for sections.

4. All required indexes:
   - GIN index on search_vector
   - GIN trigram index on content
   - B-tree index on metric_value
   - Composite index (company_name, metric_name, period_label)
   - Index on documents.company_name

--------------------------------------------------
PHASE 3 – CREATE REQUIRED SQL FUNCTIONS
--------------------------------------------------

Write at least 3 handwritten SQL functions:

1. normalize_query(user_input TEXT)
   - Apply lower()
   - Apply unaccent()
   - Return cleaned query

2. compute_numeric_bonus(...)
   - Accept metric_value and numeric constraint
   - Return score bonus

3. compute_final_score(...)
   - Combine:
       ts_rank
       similarity
       numeric bonus
       section_weight
       recency boost

All ranking must remain inside SQL.

--------------------------------------------------
PHASE 4 – IMPLEMENT SEARCH QUERY
--------------------------------------------------

Create a production-ready SELECT query that:

- Accepts user query
- Applies full-text search
- Applies trigram similarity
- Applies numeric filtering
- Computes hybrid ranking score
- Orders by final_score DESC
- Applies LIMIT 10
- Avoids sequential scan

--------------------------------------------------
PHASE 5 – PERFORMANCE VALIDATION
--------------------------------------------------

Provide:

1. Example EXPLAIN ANALYZE query.
2. Expected execution plan explanation.
3. Why no sequential scan occurs.
4. How filtering is applied before scoring.

--------------------------------------------------
PHASE 6 – TEST DATA GENERATION
--------------------------------------------------

Generate SQL to:

- Insert sample documents
- Insert sample sections
- Insert sample financial metrics
- Optionally simulate 5,000 rows for stress testing

--------------------------------------------------
CRITICAL CONSTRAINTS
--------------------------------------------------

- Do NOT use ORM.
- Do NOT use external search engines.
- Do NOT perform ranking in application layer.
- Everything must be pure PostgreSQL.
- Ranking must be computed inside SQL.
- Real-time updates must require no manual reindex.

--------------------------------------------------
OUTPUT FORMAT
--------------------------------------------------

Return:

1. Full SQL script (ready to execute).
2. Search query examples.
3. EXPLAIN ANALYZE example.
4. Short explanation summary for demo.

Think like a PostgreSQL performance engineer.
Optimize for clarity, correctness, and performance.