---
title: "PBench 1.2.1: End-to-End Benchmarking and Performance Testing for Presto"
url: "https://prestodb.io/blog/2026/02/24/pbench-1-2-1-end-to-end-benchmarking-and-performance-testing-for-presto/"
date: "Tue, 24 Feb 2026 18:46:39 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<p>Benchmarking a distributed SQL engine like Presto involves much more than running a few queries and recording wall-clock times. Real-world performance evaluation demands multi-phase test execution, concurrent workloads, production traffic replay, and deep offline analysis. <a href="https://github.com/prestodb/pbench"><strong>PBench</strong></a> is a purpose-built benchmarking toolkit for Presto that handles all of this through a declarative, composable stage system. With the 1.2.1 release, PBench becomes significantly more robust and flexible, adding parallel stream execution, dynamic query generation, richer script integration, and a fully data-driven cluster configuration generator, while continuing to provide a unified interface for standard benchmarks, A/B testing, and production workload analysis. In this post, we&#8217;ll walk through implementing a spec-compliant TPC-DS benchmark with PBench, deep offline analysis with <code class="">pbench loadjson</code>, and how the same building blocks extend to real-world workflows like schema capture, A/B testing, and traffic replay.</p>



<h2 class="wp-block-heading"><strong>What&#8217;s New in PBench 1.2.1</strong></h2>



<p>The key new capabilities that enable the workflows described in this post:</p>



<ul class="wp-block-list">
<li><strong>Parallel stream execution</strong> — the <code class="">stream_count</code> parameter runs N parallel instances of a stage, each with a deterministically derived random seed, mapping directly to the TPC-DS throughput test model</li>



<li><strong>Directory expansion in <code class="">query_files</code></strong> — entries can now point to directories, expanded to contained SQL files at execution time (after <code class="">pre_stage_scripts</code>), enabling dynamic query generation workflows like <code class="">dsqgen</code></li>



<li><strong><code class="">no_random_duplicates</code></strong> — shuffled random execution that cycles through all queries before repeating, useful for throughput tests requiring full query coverage</li>



<li><strong>Shell script environment variables</strong> — <code class="">PBENCH_STAGE_ID</code>, <code class="">PBENCH_OUTPUT_DIR</code>, <code class="">PBENCH_QUERY_FILE</code>, <code class="">PBENCH_QUERY_ID</code>, etc. are injected into all script hooks</li>



<li><strong>Data-driven <a href="https://github.com/prestodb/pbench/wiki/Generating-Benchmark-Configurations"><code class="">genconfig</code></a></strong> — generalized to use generic maps and templates with arithmetic/string functions, so adding new cluster configuration fields requires only JSON and template changes</li>
</ul>



<p>The full release notes are available for&nbsp;<a href="https://github.com/prestodb/pbench/releases/tag/v1.2">1.2</a>&nbsp;and&nbsp;<a href="https://github.com/prestodb/pbench/releases/tag/v1.2.1">1.2.1</a>.</p>



<h2 class="wp-block-heading"><strong>Implementing a Full TPC-DS Benchmark with PBench</strong></h2>



<p>The&nbsp;<a href="https://www.tpc.org/tpcds/">TPC-DS specification</a>&nbsp;defines a multi-phase benchmark: data loading, a sequential power test, a concurrent throughput test, data maintenance operations, and a second throughput test. PBench&#8217;s&nbsp;<a href="https://github.com/prestodb/pbench/wiki/Configuring-PBench">DAG-based stage system</a>&nbsp;maps naturally to this structure. Let&#8217;s walk through how to implement the complete TPC-DS benchmark lifecycle.</p>



<h3 class="wp-block-heading">The Stage DAG</h3>



<p>PBench benchmarks are defined as JSON stage files. Each stage specifies queries to run, session settings, and optionally a&nbsp;<code class="">next</code>&nbsp;field that points to child stages. Child stages execute in parallel after the parent completes, forming a DAG. Settings like&nbsp;<code class="">catalog</code>,&nbsp;<code class="">schema</code>, and&nbsp;<code class="">session_params</code>&nbsp;are inherited by child stages unless overridden.</p>



<p>The overall DAG looks like this:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">data_load (CREATE TABLE, CTAS or INSERT SELECT, ANALYZE)
  └→ power_test (99 queries, sequential)
       └→ throughput_test_1 (N parallel streams)
            └→ data_maintenance (INSERT/DELETE operations)
                 └→ throughput_test_2 (N parallel streams)</pre></div>



<h3 class="wp-block-heading">Phase 1: Data Loading</h3>



<p>In TPC-DS, raw data is pre-generated as CSV flat files by the&nbsp;<code class="">dsdgen</code>&nbsp;tool — this happens outside PBench. The loading phase creates tables in your target format and populates them. A typical approach is to create external tables on the CSV files, then use CREATE TABLE AS SELECT (CTAS) or INSERT SELECT to load the data into Iceberg or Hive Parquet tables with proper type casting, partitioning schemes, and compression:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "id": "data_load",
  "description": "Create tables, load from CSV source, and gather statistics",
  "catalog": "iceberg",
  "schema": "tpcds_sf1000_parquet",
  "query_files": [
    "./ddl/",
    "./data_loading/",
    "./analyze/"
  ],
  "abort_on_error": true,
  "next": ["power_test.json"]
}</pre></div>



<p>Since&nbsp;<code class="">query_files</code>&nbsp;supports directories as of PBench 1.2.1, each directory is expanded to its contained SQL files in sorted order. Here&nbsp;<code class="">ddl/</code>&nbsp;holds the CREATE TABLE statements,&nbsp;<code class="">data_loading/</code>&nbsp;holds the INSERT SELECT or CTAS statements that populate the tables from the CSV-backed source, and&nbsp;<code class="">analyze/</code>&nbsp;holds the ANALYZE statements for gathering column statistics. The directories are processed in the order listed, so the natural separation also gives you the correct execution sequence.</p>



<p>When benchmarking across different table formats (Iceberg vs. Hive), partitioning schemes, or compression methods, the number of DDL scripts and configurations can quickly become unwieldy. PBench includes&nbsp;<a href="https://github.com/prestodb/pbench/wiki/Command-Reference#pbench-genddl"><code class="">genddl</code></a>&nbsp;and&nbsp;<a href="https://github.com/prestodb/pbench/wiki/Generating-Benchmark-Configurations"><code class="">genconfig</code></a>&nbsp;helper commands that generate these scripts and cluster configurations from templates, reducing the chance of human error when managing many variations.<a href="https://github.com/prestodb/prestodb.github.io/blob/bf95ce86d6a2b086e54bdb1cded833c20aa619ef/website/blog/2026-02-24-pbench-1.2.md#phase-1-data-loading"></a></p>



<h3 class="wp-block-heading">Phase 2: Power Test</h3>



<p>The power test runs all 99 TPC-DS queries sequentially on a single stream. There are two approaches depending on your use case.</p>



<p><strong>For development and regression testing</strong>, you can use a fixed set of pre-generated queries with known expected row counts. This is useful during iterative development where you want fast feedback on correctness:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "id": "power_test",
  "description": "TPC-DS Power Test: 99 pre-generated queries with row count validation",
  "query_files": [
    "queries/query_01.sql",
    "queries/query_02.sql",
    "...",
    "queries/query_99.sql"
  ],
  "cold_runs": 1,
  "save_json": true,
  "expected_row_counts": {
    "tpcds_sf1000": [100, 2520, 9, "..."]
  },
  "next": ["throughput_test_1.json"]
}</pre></div>



<p>The&nbsp;<code class="">expected_row_counts</code>&nbsp;field validates that each query returns the correct number of rows for the given scale factor, catching silent correctness regressions. For row-by-row correctness checking, set&nbsp;<code class="">save_output: true</code>&nbsp;to write the full query result to disk — you can then use&nbsp;<code class="">pbench cmp</code>&nbsp;to diff outputs between runs. Setting&nbsp;<code class="">save_json: true</code>&nbsp;captures the full Presto query JSON for each query, which we can later load into a database for deep analysis on the detailed metrics (more on this below).</p>



<p><strong>For a spec-compliant TPC-DS run</strong>, queries are generated on the fly by&nbsp;<code class="">dsqgen</code>&nbsp;with a specific random seed, producing a unique query set each time. We use a pre-stage script to invoke&nbsp;<code class="">dsqgen</code>&nbsp;and point&nbsp;<code class="">query_files</code>&nbsp;at the output directory:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "id": "power_test",
  "description": "TPC-DS Power Test: queries generated by dsqgen",
  "pre_stage_scripts": ["./scripts/generate_power_queries.sh"],
  "query_files": ["./generated_queries/power/"],
  "cold_runs": 1,
  "save_json": true,
  "next": ["throughput_test_1.json"]
}</pre></div>



<p>The&nbsp;<code class="">generate_power_queries.sh</code>&nbsp;script invokes&nbsp;<code class="">dsqgen</code>&nbsp;to produce the 99 queries into the directory that PBench will discover:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">#!/bin/bash
SCALE_FACTOR=1000
SEED=12345
OUTPUT_DIR="./generated_queries/power"

mkdir -p "$OUTPUT_DIR"
dsqgen \
  -DIRECTORY ../query_templates \
  -INPUT ../query_templates/templates.lst \
  -SCALE "$SCALE_FACTOR" \
  -RNGSEED "$SEED" \
  -DIALECT presto \
  -OUTPUT_DIR "$OUTPUT_DIR"</pre></div>



<p>Since&nbsp;<code class="">query_files</code>&nbsp;supports directories as of PBench 1.2.1, the generated SQL files are automatically discovered and executed in sorted order after the pre-stage script completes. In this mode there are no expected row counts — the queries are fresh from the generator and the focus is on performance measurement rather than regression checking.</p>



<h3 class="wp-block-heading">Phase 3: Throughput Test</h3>



<p>The TPC-DS throughput test runs N concurrent query streams, each executing all 99 queries in a different permutation order (defined in the spec&#8217;s Appendix D). There are two ways to model this in PBench.</p>



<p><strong>Option A: Explicit streams via DAG.</strong>&nbsp;Define each stream as a separate stage file with its own query ordering, and fan them out from a parent stage:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "id": "throughput_test_1",
  "description": "TPC-DS Throughput Test: 4 concurrent streams",
  "next": [
    "streams/stream_01.json",
    "streams/stream_02.json",
    "streams/stream_03.json",
    "streams/stream_04.json"
  ],
  "next": ["data_maintenance.json"]
}</pre></div>



<p>Each stream file (e.g.,&nbsp;<a href="https://github.com/prestodb/pbench/blob/main/benchmarks/tpc-ds/streams/stream_01.json"><code class="">stream_01.json</code></a>) lists the 99 queries in the spec-defined order for that stream and sets&nbsp;<code class="">start_on_new_client: true</code>&nbsp;so each stream gets its own Presto session:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "start_on_new_client": true,
  "query_files": [
    "../queries/query_96.sql",
    "../queries/query_07.sql",
    "../queries/query_75.sql",
    "..."
  ]
}</pre></div>



<p>PBench ships with&nbsp;<a href="https://github.com/prestodb/pbench/tree/main/benchmarks/tpc-ds/streams">21 pre-built stream orderings</a>&nbsp;matching the TPC-DS Appendix D specification.</p>



<p><strong>Option B:&nbsp;<code class="">stream_count</code>&nbsp;for randomized throughput.</strong>&nbsp;New in PBench 1.2.1, if you don&#8217;t need the exact spec-defined orderings, you can use&nbsp;<code class="">stream_count</code>&nbsp;to spin up N parallel instances of a single stage:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "id": "throughput_test_1",
  "description": "4 concurrent random streams, no duplicates within each stream",
  "stream_count": 4,
  "random_execution": true,
  "randomly_execute_until": "99",
  "no_random_duplicates": true,
  "query_files": ["queries/query_01.sql", "...", "queries/query_99.sql"],
  "next": ["data_maintenance.json"]
}</pre></div>



<p>Each stream gets a deterministic seed derived from the base seed (<code class="">seed + stream_index * 1000</code>), so the entire run is reproducible from a single&nbsp;<code class="">--seed</code>&nbsp;value. The&nbsp;<code class="">no_random_duplicates</code>&nbsp;flag ensures each stream cycles through all 99 queries before repeating.</p>



<h3 class="wp-block-heading">Phase 4: Data Maintenance</h3>



<p>The TPC-DS spec includes data maintenance operations (INSERTs and DELETEs) between throughput tests. The refresh data is generated by&nbsp;<code class="">dsdgen</code>, so we use a&nbsp;<code class="">pre_stage_scripts</code>&nbsp;hook to generate the refresh flat files and create external staging tables on them, then execute the maintenance SQL:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">#!/bin/bash
# scripts/generate_refresh_data.sh
SCALE_FACTOR=1000
UPDATE_SET=1
REFRESH_DIR="./refresh_data"

mkdir -p "$REFRESH_DIR"
dsdgen -SCALE "$SCALE_FACTOR" -UPDATE "$UPDATE_SET" -DIR "$REFRESH_DIR"</pre></div>



<p>This produces new-row flat files and delete-key files for each affected fact table. The script can also create external staging tables pointing to these files (or that SQL can be part of the queries). The maintenance queries then reference the staging tables:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "id": "data_maintenance",
  "description": "TPC-DS Data Maintenance: generate refresh data, then INSERT/DELETE",
  "pre_stage_scripts": ["./scripts/generate_refresh_data.sh"],
  "queries": [
    "INSERT INTO catalog_sales SELECT * FROM catalog_sales_staging",
    "INSERT INTO catalog_returns SELECT * FROM catalog_returns_staging",
    "DELETE FROM catalog_sales WHERE cs_item_sk || cs_order_number IN (SELECT cs_item_sk || cs_order_number FROM catalog_sales_delete)",
    "DELETE FROM catalog_returns WHERE cr_item_sk || cr_order_number IN (SELECT cr_item_sk || cr_order_number FROM catalog_returns_delete)"
  ],
  "next": ["throughput_test_2.json"]
}</pre></div>



<h3 class="wp-block-heading">Phase 5: Second Throughput Test</h3>



<p>The second throughput test is identical in structure to the first, running after data maintenance to measure performance on the modified dataset. Simply define another throughput stage with the same query set:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "id": "throughput_test_2",
  "description": "Post-maintenance throughput test",
  "stream_count": 4,
  "random_execution": true,
  "randomly_execute_until": "99",
  "no_random_duplicates": true,
  "query_files": ["queries/query_01.sql", "...", "queries/query_99.sql"]
}</pre></div>



<h3 class="wp-block-heading">Running the Full Benchmark</h3>



<p>With the stage DAG defined, the entire multi-phase benchmark is launched with a single command:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench run \
  --server-url http://presto-coordinator:8080 \
  --name "tpcds_sf1000_%t" \
  --output-path ./results \
  --mysql mysql_config.json \  # record per-query metrics for analysis and scoring
  benchmarks/tpc-ds/sf1k.json benchmarks/tpc-ds/tpcds_full.json</pre></div>



<p>The&nbsp;<code class="">sf1k.json</code>&nbsp;file sets the scale-factor-specific schema, and&nbsp;<code class="">tpcds_full.json</code>&nbsp;defines the DAG. PBench merges these, inheriting the schema across all stages. The&nbsp;<code class="">--mysql</code>&nbsp;flag records per-query metrics (duration, row count, success/failure, etc.) into a MySQL database, enabling you to analyze slow queries and compute the TPC-DS performance score from the recorded timings. Results are also written to local CSV files.</p>



<h2 class="wp-block-heading"><strong>Offline Analysis with pbench loadjson</strong></h2>



<p><a href="https://github.com/prestodb/prestodb.github.io/blob/bf95ce86d6a2b086e54bdb1cded833c20aa619ef/website/blog/2026-02-24-pbench-1.2.md#offline-analysis-with-pbench-loadjson"></a>Running benchmarks is only half the story — understanding <em>why</em> performance differs between runs requires deep query-level analysis. This is where <code class="">pbench loadjson</code> and the Presto event listener come in.</p>



<h3 class="wp-block-heading">Capturing Query Details</h3>



<p>When PBench runs with&nbsp;<code class="">save_json: true</code>, it captures the full Presto query JSON (from the&nbsp;<code class="">/v1/query/{id}</code>&nbsp;API) for every query execution. These JSON files contain the complete query plan, operator statistics, stage-level metrics, and timing breakdowns.</p>



<p>Independently, Presto&#8217;s <a href="https://prestodb.io/docs/current/develop/event-listener.html">event listener</a> can be configured to write query completion events to disk as JSON files. This captures <em>all</em> queries on a cluster — not just those from PBench — providing a complete picture of cluster activity during benchmarks.</p>



<h3 class="wp-block-heading">Loading into a Database</h3>



<p><code class="">pbench loadjson</code>&nbsp;processes these JSON files and loads them into MySQL tables for structured analysis:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench loadjson \
  --mysql mysql_config.json \
  --name "tpcds_sf1000_analysis" \
  ./results/tpcds_sf1000/</pre></div>



<p>When&nbsp;<code class="">--mysql</code>&nbsp;is provided to&nbsp;<code class="">pbench run</code>, run metadata (run name, timing, query durations, success/failure) is automatically recorded to the database. The detailed per-query metrics (operator stats, stage stats, query plans) come from a separate path — either via&nbsp;<code class="">save_json: true</code>&nbsp;in PBench or by enabling a Presto&nbsp;<a href="https://prestodb.io/docs/current/develop/event-listener.html">event listener</a>&nbsp;plugin that writes query JSON to disk.&nbsp;<code class="">pbench loadjson</code>&nbsp;then parses these JSON files and loads them into the database — you must provide&nbsp;<code class="">--mysql</code>&nbsp;for it to write to the database.</p>



<p>This populates five tables (DDL is available in&nbsp;<a href="https://github.com/prestodb/pbench/blob/main/stage/event_listener_ddl.sql"><code class="">event_listener_ddl.sql</code></a>):</p>



<ul class="wp-block-list">
<li><strong><code class="">presto_query_creation_info</code></strong> — query text, catalog, schema, session properties, user, and resource group for each query</li>



<li><strong><code class="">presto_query_statistics</code></strong> — top-level execution metrics: wall time, CPU time, peak memory, input/output rows and bytes, queued time, and failure info</li>



<li><strong><code class="">presto_query_stage_stats</code></strong> — per-stage breakdown (each Presto execution stage, not PBench stages): CPU time, I/O, memory, and GC statistics</li>



<li><strong><code class="">presto_query_operator_stats</code></strong> — per-operator metrics: CPU and wall time, memory reservations, input/output rows for each scan, join, aggregation, etc.</li>



<li><strong><code class="">presto_query_plans</code></strong> — query plan in both text and JSON format for plan comparison across runs</li>
</ul>



<p>Together these tables let you drill down from a slow query all the way to the specific operator and stage responsible.<a href="https://github.com/prestodb/prestodb.github.io/blob/bf95ce86d6a2b086e54bdb1cded833c20aa619ef/website/blog/2026-02-24-pbench-1.2.md#loading-into-a-database"></a><a href="https://github.com/prestodb/prestodb.github.io/blob/bf95ce86d6a2b086e54bdb1cded833c20aa619ef/website/blog/2026-02-24-pbench-1.2.md#capturing-query-details"></a></p>



<h3 class="wp-block-heading">Comparative Analysis</h3>



<p>With results from multiple PBench runs loaded into the same database, you can perform comparative analysis using SQL:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">-- Compare query durations between two runs
SELECT
  a.query_file,
  a.duration_ms AS baseline_ms,
  b.duration_ms AS candidate_ms,
  ROUND((b.duration_ms - a.duration_ms) / a.duration_ms * 100, 1) AS pct_change
FROM pbench_queries a
JOIN pbench_queries b
  ON a.query_file = b.query_file
  AND a.sequence_no = b.sequence_no
WHERE a.run_id = (SELECT id FROM pbench_runs WHERE run_name = 'baseline_run')
  AND b.run_id = (SELECT id FROM pbench_runs WHERE run_name = 'candidate_run')
ORDER BY pct_change DESC;</pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">-- Find operators with the highest CPU increase in the candidate build
SELECT
  b.operator_type,
  SUM(b.wall_nanos - a.wall_nanos) / 1e9 AS wall_time_increase_sec,
  SUM(b.input_rows - a.input_rows) AS input_rows_increase
FROM presto_query_operator_stats a
JOIN presto_query_operator_stats b
  ON a.query_id = b.query_id AND a.operator_type = b.operator_type
WHERE a.query_id IN (SELECT query_id FROM pbench_queries WHERE run_id = 1)
  AND b.query_id IN (SELECT query_id FROM pbench_queries WHERE run_id = 2)
GROUP BY b.operator_type
ORDER BY wall_time_increase_sec DESC;</pre></div>



<p>This turns PBench + event listener data into a queryable performance analysis platform. You can identify exactly which operators regressed, which stages are bottlenecked, and how resource consumption changed between Presto versions.</p>



<h2 class="wp-block-heading"><strong>Beyond Standard Benchmarks: Real-World Performance Testing</strong></h2>



<p>Standard benchmarks like TPC-DS are valuable, but production workloads often behave very differently. PBench provides three tools for testing with real-world query patterns.</p>



<h3 class="wp-block-heading">Capturing Production Schemas with pbench save</h3>



<p>Before you can replay production queries against a test cluster, you need to reproduce the schema.&nbsp;<code class="">pbench save</code>&nbsp;exports table metadata, column statistics, and partition information from a live cluster:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench save \
  --server-url http://production-presto:8080 \
  --catalog hive \
  --schema production_db \
  --output-path ./saved_schemas \
  --parallel 8 \
  customers orders transactions</pre></div>



<p>This generates one JSON file per table, named&nbsp;<code class="">{catalog}_{schema}_{table}.json</code>&nbsp;(e.g.,&nbsp;<code class="">hive_production_db_orders.json</code>). Each file captures the full DDL and column-level statistics:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">{
  "name": "orders",
  "catalog": "hive",
  "schema": "production_db",
  "ddl": "CREATE TABLE hive.production_db.orders (\n   \"order_id\" bigint,\n   \"customer_id\" bigint,\n   \"order_date\" date,\n   \"total_amount\" decimal(12,2),\n   \"status\" varchar,\n   \"region\" varchar\n)\nWITH (\n   format = 'PARQUET',\n   partitioned_by = ARRAY['region']\n)",
  "columnStats": [
    {
      "column_name": "order_id",
      "distinct_values_count": 5000000,
      "nulls_fraction": 0,
      "low_value": "1",
      "high_value": "5000000",
      "data_type": "bigint"
    },
    {
      "column_name": "total_amount",
      "distinct_values_count": 48923,
      "nulls_fraction": 0.002,
      "low_value": "0.99",
      "high_value": "9999.99",
      "data_type": "decimal(12,2)"
    },
    {
      "column_name": "region",
      "data_size": 42000000,
      "distinct_values_count": 12,
      "nulls_fraction": 0,
      "data_type": "varchar",
      "extra": "partition key"
    },
    {"row_count": 5000000}
  ],
  "rowCount": 5000000
}</pre></div>



<p>This gives you the complete DDL to recreate the table on a test cluster, along with column statistics (cardinality, null fractions, min/max values, data sizes) that inform the query optimizer. For bulk capture, you can pass a CSV file listing all tables:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench save \
  --server-url http://production-presto:8080 \
  -f tables.csv \
  --output-path ./saved_schemas</pre></div>



<h3 class="wp-block-heading">A/B Testing with pbench forward</h3>



<p><code class="">pbench forward</code>&nbsp;monitors a source Presto cluster and mirrors every incoming query to one or more target clusters in real time. This enables transparent A/B testing between Presto versions or configurations without modifying any client applications:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench forward \
  --server-url http://current-presto:8080 \
  --server-url http://candidate-presto:8080 \
  --poll-interval 5s \
  --exclude "^(EXPLAIN|DESCRIBE|SHOW)" \
  --schema-mapping prod_schema,test_schema \
  --output-path ./forward_results \
  --name "v0.286_vs_v0.287"</pre></div>



<p>This forwards all queries from the current cluster to the candidate, excluding DDL/metadata queries. The&nbsp;<code class="">--schema-mapping</code>&nbsp;flag handles cases where the test cluster uses a different schema name. You can add multiple&nbsp;<code class="">--server-url</code>&nbsp;targets to test more than two configurations simultaneously.</p>



<p><code class="">pbench forward</code>&nbsp;also supports query rewriting via regex patterns, useful when table names or function names differ between versions:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench forward \
  --server-url http://source:8080 \
  --server-url http://target:8080 \
  --replace "old_udf\((.*?)\)" "new_udf(\1)" \
  --replace "legacy_table" "migrated_table"</pre></div>



<h3 class="wp-block-heading">Traffic Replay with pbench replay</h3>



<p>While&nbsp;<code class="">forward</code>&nbsp;mirrors live traffic,&nbsp;<code class="">replay</code>&nbsp;replays recorded traffic from a CSV file. This is useful for reproducible testing — record a production workload once, then replay it against multiple cluster configurations:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench replay \
  --server-url http://test-presto:8080 \
  --parallel 150 \
  --name "peak_hour_replay" \
  --output-path ./replay_results \
  workload_capture.csv</pre></div>



<p>The CSV file contains query metadata captured from production (query text, original timing, catalog, schema, session properties). PBench replays queries with the original inter-query timing to simulate realistic load patterns, and respects the parallelism limit to avoid overwhelming the test cluster.<a href="https://github.com/prestodb/prestodb.github.io/blob/bf95ce86d6a2b086e54bdb1cded833c20aa619ef/website/blog/2026-02-24-pbench-1.2.md#traffic-replay-with-pbench-replay"></a></p>



<h3 class="wp-block-heading">An End-to-End Real-World Testing Workflow</h3>



<p>Here&#8217;s how these tools come together for a complete Presto version upgrade validation:</p>



<p><strong>Step 1: Capture the baseline.</strong>&nbsp;Save production schemas and run standard benchmarks against the current version:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted"># Export schemas
pbench save --server-url http://current:8080 \
  --catalog hive --schema prod -f all_tables.csv \
  --output-path ./schemas

# Run TPC-DS baseline
pbench run --server-url http://current:8080 \
  --name "baseline_tpcds_%t" --mysql mysql.json \
  --output-path ./results \
  benchmarks/tpc-ds/sf1k.json benchmarks/tpc-ds/ds_full.json</pre></div>



<p><strong>Step 2: Live A/B testing.</strong>&nbsp;Forward production traffic to the candidate cluster and collect real-world comparison data:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench forward \
  --server-url http://current:8080 \
  --server-url http://candidate:8080 \
  --exclude "^(EXPLAIN|DESCRIBE|SHOW)" \
  --output-path ./forward_results \
  --name "upgrade_ab_test"</pre></div>



<p><strong>Step 3: Replay and compare.</strong>&nbsp;Take a recorded peak-hour workload and replay it against both clusters:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench replay --server-url http://current:8080 \
  --name "peak_replay_baseline" workload.csv

pbench replay --server-url http://candidate:8080 \
  --name "peak_replay_candidate" workload.csv</pre></div>



<p><strong>Step 4: Run TPC-DS on the candidate.</strong>&nbsp;Run the same standard benchmark on the new version:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench run --server-url http://candidate:8080 \
  --name "candidate_tpcds_%t" --mysql mysql.json \
  --output-path ./results \
  benchmarks/tpc-ds/sf1k.json benchmarks/tpc-ds/ds_full.json</pre></div>



<p><strong>Step 5: Deep analysis.</strong>&nbsp;Load all query JSON files into the database and compare:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted"># Load event listener output from both clusters
pbench loadjson --mysql mysql.json \
  --name "baseline" ./results/baseline_tpcds/

pbench loadjson --mysql mysql.json \
  --name "candidate" ./results/candidate_tpcds/</pre></div>



<p>Now you can query the MySQL database to compare operator-level metrics, identify regressions, and validate that the candidate version meets your performance bar — all without writing a single line of custom tooling.</p>



<p><strong>Step 6: Validate correctness.</strong>&nbsp;Use&nbsp;<code class="">pbench cmp</code>&nbsp;to diff query outputs between the two runs:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pbench cmp \
  ./results/baseline_tpcds/ \
  ./results/candidate_tpcds/ \
  --output-path ./diffs</pre></div>



<p>This generates diffs for any queries that returned different results, catching correctness regressions that performance metrics alone would miss.</p>



<h2 class="wp-block-heading"><strong>Getting Started</strong></h2>



<p>Check out the&nbsp;<a href="https://github.com/prestodb/pbench/releases/tag/v1.2.1">PBench 1.2.1 release</a>&nbsp;for pre-built binaries for macOS and Linux, or build from source:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">git clone https://github.com/prestodb/pbench.git
cd pbench
make install</pre></div>



<p>The&nbsp;<a href="https://github.com/prestodb/pbench/wiki">PBench Wiki</a>&nbsp;has comprehensive documentation on configuring stages, setting up result databases, and writing benchmark suites.</p>



<p>Whether you&#8217;re running standard TPC-DS benchmarks, validating a Presto version upgrade with production traffic, or building a continuous performance regression pipeline, PBench 1.2.1 provides the building blocks to do it declaratively and reproducibly. We welcome contributions — check out the&nbsp;<a href="https://github.com/prestodb/pbench">GitHub repository</a>&nbsp;to get involved.</p>



<h2 class="wp-block-heading"><a href="https://github.com/prestodb/prestodb.github.io/blob/bf95ce86d6a2b086e54bdb1cded833c20aa619ef/website/blog/2026-02-24-pbench-1.2.md#getting-started"></a><strong>Follow Us</strong></h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/02/24/pbench-1-2-1-end-to-end-benchmarking-and-performance-testing-for-presto/">PBench 1.2.1: End-to-End Benchmarking and Performance Testing for Presto</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
