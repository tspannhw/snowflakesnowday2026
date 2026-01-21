─ Cortex Code v0.26.120+161743.a758c3df ──────────────────────

     ░░░░░░
    ░ ░░░░░░░░                            ░░░░░░
   ░░░░░░░░░░░░░                         ░░░░░░░░░
   *        *                   *      ░░░░░░░
       █▀█▀█             *                    *
      ███████                               *
     ███   ███        *                                 *
───── ███████ ────────────────────────────────────────────────

Agent Connection:  snowhouse
SQL Connection:    snowprod
Warehouse:         INGEST
Model:             auto

Working Directory:
  /Users/tspann/Downloads/code/nifi-2.7.2

> provide a chart showing the largest 10 queries today 

❄ I'll query Snowflake for today's largest queries and create a chart. Let me first get the data.

✓  SNOWFLAKE_SQL_EXECUTE  (10 rows)
  SQL Result (10 rows × 8 cols) [ctrl+t to navigate]
  ┌──────┬────────────────────────────────┬────────────────────────────────┬───────────┐
  │    # │ QUERY_ID                       │ QUERY_TEXT                     │ USER_NAME │
  ├──────┼────────────────────────────────┼────────────────────────────────┼───────────┤
  │    1 │ 01c1e2f9-0210-ceb3-000a-7fe... │ SELECT AI_PARSE_DOCUMENT( T... │ TSPANN    │
  │    2 │ 01c1e2b6-0210-dd71-000a-7fe... │ SELECT AI_PARSE_DOCUMENT(TO... │ TSPANN    │
  │    3 │ 01c1e030-0210-d7ff-000a-7fe... │                                │ SYSTEM    │
  └──────┴────────────────────────────────┴────────────────────────────────┴───────────┘
  Rows 1-3/10 | Cols 1-3/8

❄ Now let me create a chart visualization for this data.

✓  BASH  cat << 'EOF' | python3
         import matplotlib.pyplot as plt
         import numpy as np

         # Data from the query
         ...
  ├─ import matplotlib.pyplot as plt
  ├─ import numpy as np
  └─ ... (39 more lines)

✓  READ  /tmp/top_queries_chart.png

❄ Here's the chart showing the top 10 largest queries today by execution time.

  Key findings:

    * AI_PARSE_DOCUMENT operations dominate, taking ~127 seconds each (PDF parsing)
    * System tasks (background compute) took 43-50 seconds
    * GRANTS_TO_ROLES count query scanned ~12GB and took 43 seconds
    * Cortex Search Index Build and Cortex Analyst Optimization took ~37 seconds each
    * Most queries were run by user TSPANN on the INGEST warehouse

  The chart is saved at /tmp/top_queries_chart.png.

> Can you minimize unneeded Systems tasks and disable all cortex searches 

❄ I'll help you disable Cortex Search services and look into the system tasks. Let me first identify what Cortex 
  Search services exist in your account.

✓  SNOWFLAKE_SQL_EXECUTE  (5 rows)
  SQL Result (5 rows × 25 cols) [ctrl+t to navigate]
  ┌─────┬────────────────────────────────┬──────────────────────────┬────────────────────────────────┐
  │   # │ created_on                     │ name                     │ id                             │
  ├─────┼────────────────────────────────┼──────────────────────────┼────────────────────────────────┤
  │   1 │ "2025-11-12 12:45:55.314 -0... │ TASK_DAILY_METRICS       │ 01c0595d-1ed9-ffd3-0000-000... │
  │   2 │ "2025-10-27 12:27:41.958 -0... │ TASK_AGENT_ANALYZE       │ 01bfff0f-7bab-56e6-0000-000... │
  │   3 │ "2025-10-27 12:27:42.248 -0... │ TASK_AGENT_DAILY_SUMMARY │ 01bfff0f-dca5-4876-0000-000... │
  └─────┴────────────────────────────────┴──────────────────────────┴────────────────────────────────┘
  Rows 1-3/5 | Cols 1-3/25

✓  SNOWFLAKE_SQL_EXECUTE  (13 rows)
  SQL Result (13 rows × 19 cols) [ctrl+t to navigate]
  ┌──────┬────────────────────────────────┬────────────────────────────────┬────────────────────────┐
  │    # │ created_on                     │ name                           │ database_name          │
  ├──────┼────────────────────────────────┼────────────────────────────────┼────────────────────────┤
  │    1 │ "2025-10-02 07:22:44.988 -0... │ SUPPORT_TICKETS                │ DASH_MCP_DB            │
  │    2 │ "2025-05-19 14:02:31.341 -0... │ AIRQUALITY_SRVC                │ DEMO                   │
  │    3 │ "2025-10-10 11:35:39.886 -0... │ MTASEARCH                      │ DEMO                   │
  └──────┴────────────────────────────────┴────────────────────────────────┴────────────────────────┘
  Rows 1-3/13 | Cols 1-3/19

❄ I found 13 Cortex Search services in your account. Most are already suspended, but a few are still active. Let 
  me suspend all of them that are currently active.

✓  SNOWFLAKE_SQL_EXECUTE  (1 rows)
  SQL Result (1 rows × 1 cols)
  ┌─────┬────────────────────────────────┐
  │   # │ status                         │
  ├─────┼────────────────────────────────┤
  │   1 │ Cortex search service AIRQU... │
  └─────┴────────────────────────────────┘

×  SNOWFLAKE_SQL_EXECUTE  ALTER CORTEX SEARCH SERVICE SNOWFLAKEDOCUMENTATION.SHARED...
  └─SQL access control error: Insufficient privileges to operate on cortex_search_service 'CKE_SNOWFLAKE_DOCS_SE…

✓  SNOWFLAKE_SQL_EXECUTE  (1 rows)
  SQL Result (1 rows × 1 cols)
  ┌─────┬────────────────────────────────┐
  │   # │ status                         │
  ├─────┼────────────────────────────────┤
  │   1 │ Cortex search service MTASE... │
  └─────┴────────────────────────────────┘

❄ Two of the three active services have been suspended. The CKE_SNOWFLAKE_DOCS_SERVICE requires additional 
  privileges. Let me provide a summary:

  Summary

  Cortex Search Services - Disabled

  ┌────────────────────────────┬───────────────────────────────┬──────────────────────────────────┐
  │ Service                    │ Database.Schema               │ Status                           │
  ├────────────────────────────┼───────────────────────────────┼──────────────────────────────────┤
  │ AIRQUALITY_SRVC            │ DEMO.DEMO                     │ Suspended                        │
  ├────────────────────────────┼───────────────────────────────┼──────────────────────────────────┤
  │ MTASEARCH                  │ DEMO.DEMO                     │ Suspended                        │
  ├────────────────────────────┼───────────────────────────────┼──────────────────────────────────┤
  │ CKE_SNOWFLAKE_DOCS_SERVICE │ SNOWFLAKEDOCUMENTATION.SHARED │ Failed - insufficient privileges │
  └────────────────────────────┴───────────────────────────────┴──────────────────────────────────┘

  Already Suspended (11 services)

    * SUPPORT_TICKETS (DASH_MCP_DB.DATA)
    * MTASEARCH2 (DEMO.DEMO)
    * TRAFFICIMAGESEARCH (DEMO.DEMO)
    * TRANSCOMS (DEMO.DEMO)
    * CORTEX_ANALYST_TRANSCOM_DESCRIPTION* (DEMO.DEMO)
    * CORTEX_ANALYST_TRANSCOM_TITLE* (DEMO.DEMO)
    * GHOST_EVIDENCE_SEARCH (GHOST_DETECTION.APP)
    * GHOST_SIGHTINGS_SEARCH (GHOST_DETECTION.APP)
    * SALES_CONVERSATION_SEARCH (SALES_INTELLIGENCE.DATA)
    * FESTIVAL_CONTRACT_SEARCH (SNOWFLAKE_INTELLIGENCE.CONFIG)

  Tasks

  All 5 tasks in your account are already suspended - no action needed.

  System Tasks

  The "System Task" queries you saw (running on COMPUTE_SERVICE_WH_USER_TASKS_POOL_XSMALL_0) are internal 
  Snowflake background operations for features like Cortex Search index building. By suspending the Cortex Search
   services above, those system tasks should stop.

  To suspend CKE_SNOWFLAKE_DOCS_SERVICE, you'll need ACCOUNTADMIN or the owner role of that service.
