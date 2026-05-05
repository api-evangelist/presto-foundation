---
title: "TPC-H vs TPC-DS : Benchmarking Modern Distributed SQL Engines like Presto"
url: "https://prestodb.io/blog/2026/01/30/tpc-h-vs-tpc-ds-benchmarking-modern-distributed-sql-engines-presto/"
date: "Fri, 30 Jan 2026 23:02:19 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<p>In the world of big data, <strong>performance is the ultimate currency.</strong> But when you are processing petabytes of data across a distributed cluster, <strong>speed</strong> isn&#8217;t just about a stopwatch, it&#8217;s a high-stakes engineering challenge.</p>



<p>Whether you are evaluating <strong>Presto, Spark or any other engine</strong>, you need an objective yardstick. Performance in a distributed SQL engine is a multi-dimensional function of:</p>



<ul class="wp-block-list">
<li><strong>I/O Throughput:</strong> How fast can data be pulled from S3 or HDFS?</li>



<li><strong>CPU Efficiency:</strong> How effectively does the engine handle serialization and compression?</li>



<li><strong>Network Latency:</strong> How much shuffling occurs between nodes?</li>



<li><strong>Optimizer Intelligence:</strong> Can the engine rewrite a query to avoid unnecessary work?</li>
</ul>



<p>To cut through the marketing noise, the industry relies on two gold standards: <strong>TPC-H</strong> and <strong>TPC-DS</strong>.</p>



<h2 class="wp-block-heading"><strong>Why Analytical Benchmarks Exist?</strong></h2>



<p>Before standardization, database benchmarking was dominated by <strong>vanity metrics.</strong> Vendors often cherry-picked simplistic queries to claim high performance while masking architectural weaknesses. Without a common framework for data complexity, consistency, or hardware specs, these claims lacked engineering context, making objective, apples-to-apples comparisons impossible for data architects.</p>



<h2 class="wp-block-heading"><strong>Workload Classification: OLTP vs. OLAP</strong></h2>



<p>To understand Presto’s architecture, we must distinguish between the two primary data access patterns.</p>



<ul class="wp-block-list">
<li><strong>OLTP (Online Transaction Processing):</strong> Characterized by high concurrency, point lookups (Index Scans), and small write transactions (ACID compliance). <em>Example: Postgres serving an API.</em></li>



<li><strong>OLAP (Online Analytical Processing):</strong> Characterized by scanning massive datasets, columnar reads, and complex aggregations. <em>Example: Presto generating a quarterly report.</em></li>
</ul>



<p>Presto is an analytical SQL engine optimized for <strong>OLAP-style workloads</strong> such as large scans, joins, and aggregations, rather than transactional OLTP operations.</p>



<h2 class="wp-block-heading"><strong>Why Query Engines Are Benchmarked?</strong></h2>



<p>Unlike a transactional database, an analytical engine is benchmarked on&nbsp;intelligence. How well can it scan billions of rows? How efficiently can it join 10 tables? Benchmarks like TPC-H and TPC-DS are designed specifically to stress-test these analytical capabilities.</p>



<h2 class="wp-block-heading"><strong>The TPC: Establishing the Industry&#8217;s Standard Ruler</strong></h2>



<p>Founded in 1988, the <strong>Transaction Processing Performance Council (TPC)</strong> is the non-profit authority that moved the industry away from marketing claims toward verified engineering. By defining rigorous, open-source standards, the TPC provided the data ecosystem with its first <strong>&#8220;Standard Ruler&#8221;</strong>, enabling architects to make data-driven decisions based on transparent, audited performance data rather than vendor speculation.</p>



<figure class="wp-block-image size-full is-resized"><img alt="" class="wp-image-2718" height="640" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260130235147.png" style="width: 725px; height: auto;" width="640" /></figure>



<p>Benchmarks solve this by replacing ad-hoc testing with a scientifically reproducible process:</p>



<ol class="wp-block-list">
<li><strong>Fixed Schemas:</strong> Everyone uses the exact same table structures.</li>



<li><strong>Deterministic Data:</strong> Standardization of data generation ensures the <strong>1TB</strong> dataset is statistically identical for everyone.</li>



<li><strong>Auditable Queries:</strong> Queries are fixed at the SQL level, while execution strategies may vary within the benchmark’s compliance rules.</li>
</ol>



<h2 class="wp-block-heading"><strong>TPC-H Explained</strong></h2>



<p>TPC-H is a standardized decision-support benchmark for analytical SQL engines. It provides a <strong>fixed data schema</strong> and <strong>22 complex queries</strong> designed to measure how efficiently a system handles large-scale data processing under real-world business constraints.</p>



<h3 class="wp-block-heading">How it Works?</h3>



<p>Without a benchmark, fast is a subjective term. TPC-H levels the playing field by standardizing:</p>



<ul class="wp-block-list">
<li><strong>The Data Model:</strong> A normalized schema reflecting a wholesale supplier.</li>



<li><strong>Data Generation:</strong> Using <code class="">dbgen</code> to ensure data distribution is identical across all tested systems.</li>



<li><strong>Query Set:</strong> 22 hand-crafted SQL queries ranging from simple filters to complex multi-way joins.</li>



<li><strong>Metrics:</strong> Standardized reporting for Power (how fast) and Throughput (how many).</li>
</ul>



<h3 class="wp-block-heading">The Business Scenario: Wholesale Supplier Operations</h3>



<p>TPC-H models a <strong>wholesale supplier managing orders and parts</strong>. The fictional business scenario includes:</p>



<ul class="wp-block-list">
<li>Suppliers providing parts to customers</li>



<li>Customers placing orders for parts</li>



<li>Orders consisting of line items (individual part quantities)</li>



<li>Parts sourced from specific suppliers at negotiated prices</li>
</ul>



<p>This scenario reflects traditional supply chain and order management systems, providing a realistic context for analytical queries about sales, inventory, supplier performance, and customer behavior.</p>



<h3 class="wp-block-heading">TPC-H Schema Design: Normalized Relational Model</h3>



<p>TPC-H uses a highly normalized 3NF relational schema, this model tests an engine&#8217;s ability to handle <strong>heavy joins</strong> and <strong>complex relationships</strong>.</p>



<figure class="wp-block-table"><table class="has-fixed-layout"><thead><tr><th><strong>Table</strong></th><th><strong>Scale Factor (SF1000)</strong></th><th><strong>Role</strong></th></tr></thead><tbody><tr><td><code class="">lineitem</code></td><td>~6 Billion Rows</td><td>The &#8220;fact&#8221; table; contains every item in every order.</td></tr><tr><td><code class="">orders</code></td><td>~1.5 Billion Rows</td><td>Order headers, dates, and priorities.</td></tr><tr><td><code class="">partsupp</code></td><td>~800 Million Rows</td><td>Relationship between parts and their suppliers.</td></tr><tr><td><code class="">part / customer</code></td><td>~200M / ~150M</td><td>Dimensional data for filtering by segment or type.</td></tr><tr><td><code class="">supplier</code></td><td>~10 Million Rows</td><td>Supplier details and account balances.</td></tr><tr><td><code class="">nation / region</code></td><td>25 / 5 Rows</td><td>Static lookup tables for geographic analysis.</td></tr></tbody></table></figure>



<p></p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p><strong>Scale Factor (SF):</strong> <strong>SF1 = 1GB</strong> of raw data. This helps in visualizing the scale. Most modern engines (Presto, Spark, etc) benchmark at <strong>SF100 (100 GB)</strong>, <strong>SF1000 (1TB)</strong>, <strong>SF10000 (10TB)</strong>, or higher.</p>
</blockquote>



<h3 class="wp-block-heading">What&nbsp;the 22 queries&nbsp;test?</h3>



<p>They’re designed to force different skills in a query engine:</p>



<ul class="wp-block-list">
<li><strong>Big scans + filters + group by</strong> (can you stream and aggregate efficiently?)</li>



<li><strong>Multi-way joins</strong> (can you reorder joins and choose good join algorithms?)</li>



<li><strong>Skew and selectivity</strong> (do you handle <strong>rare filters</strong> vs <strong>common filters</strong>?)</li>



<li><strong>Sorting / top-N</strong> (do you spill? do you use partial top-N?)</li>



<li><strong>Subqueries / correlated logic</strong> (optimizer maturity)</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2719" height="728" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260128180402-1024x728.png" style="width: 727px; height: auto;" width="1024" /></figure>



<h2 class="wp-block-heading"><strong>TPC-DS Explained</strong></h2>



<p><strong>TPC-DS (Transaction Processing Performance Council – Decision Support)</strong> is the industry standard benchmark for measuring the performance of decision support systems (like SQL engines, Data Warehouses, and Big Data platforms). It is intentionally designed to introduce complexity, skew, and optimizer stress conditions commonly seen in production analytics. It forces the engine to solve messy, real-world problems.</p>



<h3 class="wp-block-heading">How it Works?</h3>



<p>TPC-DS tests the sophistication of the query engine. It simulates a decision support system by standardizing:</p>



<ul class="wp-block-list">
<li><strong>The Data Model:</strong> A multi-dimensional <strong>Snowflake Schema</strong> representing an omnichannel retailer (Store, Web, and Catalog sales).</li>



<li><strong>Data Generation:</strong> Using <code class="">dsdgen</code> to produce datasets with significant <strong>Data Skew</strong> and non-uniform distributions, creating <strong>hot spots</strong> that punish naive execution plans.</li>



<li><strong>Query Set:</strong> 99 complex SQL queries (with variations totaling ~100+) using advanced SQL features like GROUPING SETS , ROLLUP , INTERSECT, and extensive Window Functions.</li>



<li><strong>Metrics:</strong> A rigorous score combining Query Response Time (Power), Multi-user Performance (Throughput), and Data Maintenance time.</li>
</ul>



<h3 class="wp-block-heading">The Business Scenario: Global Retail Enterprise</h3>



<p>TPC-DS models&nbsp;a&nbsp;<strong>large retail store chain</strong>&nbsp;with multiple sales&nbsp;channels and&nbsp;complex business operations. This&nbsp;scenario was&nbsp;chosen because&nbsp;it represents the&nbsp;complexity&nbsp;of modern retail analytics and decision support systems. The scenario includes:</p>



<ul class="wp-block-list">
<li>Multiple sales channels (store, catalog, web)</li>



<li>Product catalog with hierarchical categorization</li>



<li>Customer demographics and purchasing behavior</li>



<li>Promotions and marketing campaigns</li>



<li>Inventory management across warehouses</li>



<li>Returns and refunds</li>
</ul>



<h3 class="wp-block-heading">TPC-DS Schema Design: Snowflake&nbsp;Schema (or Constellation Schema)</h3>



<p>TPC-DS uses&nbsp;a&nbsp;<strong>snowflake schema</strong>&nbsp;with&nbsp;24 tables. This is significantly&nbsp;more&nbsp;complex than TPC-H&#8217;s highly normalized 3NF schema.</p>



<p><strong>Fact Tables (7)</strong>:</p>



<figure class="wp-block-table"><table class="has-fixed-layout"><thead><tr><th><strong>Category</strong></th><th><strong>Table Name</strong></th><th><strong>Business Context &amp; Primary Data</strong></th></tr></thead><tbody><tr><td><strong>Store</strong></td><td><code class="">store_sales</code></td><td>Physical store transactions; measures quantity, price, and net profit.</td></tr><tr><td></td><td><code class="">store_returns</code></td><td>In-store product returns; tracks refund amounts and processing fees.</td></tr><tr><td><strong>Web</strong></td><td><code class="">web_sales</code></td><td>Online orders; measures shipping costs, sales price, and order numbers.</td></tr><tr><td></td><td><code class="">web_returns</code></td><td>Web-originated returns; tracks lost revenue and account credits.</td></tr><tr><td><strong>Catalog</strong></td><td><code class="">catalog_sales</code></td><td>Phone/mail orders; measures wholesale cost, markups, and coupons.</td></tr><tr><td></td><td><code class="">catalog_returns</code></td><td>Catalog-based returns; tracks return taxes and refunded amounts.</td></tr><tr><td><strong>Inventory</strong></td><td><code class="">inventory</code></td><td>Stock snapshots; measures quantity on hand per item and warehouse.</td></tr></tbody></table></figure>



<p></p>



<p><strong>Dimension Tables (17)</strong>:</p>



<figure class="wp-block-table"><table class="has-fixed-layout"><thead><tr><th><strong>Category</strong></th><th><strong>Table Name</strong></th><th><strong>Business Context &amp; Primary Data</strong></th></tr></thead><tbody><tr><td><strong>Customer</strong></td><td><code class="">customer</code></td><td>The master identity record containing unique buyer IDs and basic info.</td></tr><tr><td></td><td><code class="">customer_address</code></td><td>Geographical location data used for shipping, tax, and regional reporting.</td></tr><tr><td></td><td><code class="">customer_demographics</code></td><td>Profile attributes like gender, marital status, and education level.</td></tr><tr><td></td><td><code class="">household_demographics</code></td><td>Domestic context including number of dependents and vehicle ownership.</td></tr><tr><td></td><td><code class="">income_band</code></td><td>Specific annual income ranges to categorize purchasing power.</td></tr><tr><td><strong>Product</strong></td><td><code class="">item</code></td><td>The product master tracking descriptions, brands, sizes, and prices.</td></tr><tr><td></td><td><code class="">item_category</code></td><td>Logical grouping data for product hierarchies and sub-categories.</td></tr><tr><td></td><td><code class="">promotion</code></td><td>Marketing event details, discount types, and campaign durations.</td></tr><tr><td><strong>Logistics</strong></td><td><code class="">warehouse</code></td><td>Storage facility metadata, including location and square footage.</td></tr><tr><td></td><td><code class="">ship_mode</code></td><td>Shipping logistics methods like Air, Express, or Ground.</td></tr><tr><td><strong>Time</strong></td><td><code class="">date_dim</code></td><td>Rich calendar data mapping dates to holidays, seasons, and fiscal years.</td></tr><tr><td></td><td><code class="">time_dim</code></td><td>Clock-level breakdown of hours, minutes, and AM/PM indicators.</td></tr><tr><td><strong>Channels</strong></td><td><code class="">store</code></td><td>Physical store location details, manager names, and operating hours.</td></tr><tr><td></td><td><code class="">call_center</code></td><td>Attributes of phone/catalog sales centers and their tax jurisdictions.</td></tr><tr><td></td><td><code class="">web_site</code></td><td>Global configuration and metadata for the online sales platform.</td></tr><tr><td></td><td><code class="">web_page</code></td><td>Characteristics of specific URLs where web sales are triggered.</td></tr><tr><td><strong>Feedback</strong></td><td><code class="">reason</code></td><td>Standardized descriptive codes explaining the &#8220;Why&#8221; behind returns.</td></tr></tbody></table></figure>



<p></p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>This schema complexity is what makes TPC-DS significantly harder for optimizers than TPC-H</p>
</blockquote>



<h3 class="wp-block-heading">What&nbsp;the 99 queries&nbsp;test?</h3>



<p>The&nbsp;99 TPC-DS&nbsp;queries are organized into categories&nbsp;that test&nbsp;different&nbsp;aspects of database performance&nbsp;and SQL&nbsp;capabilities.</p>



<ol class="wp-block-list" start="1">
<li>Iterative Reporting (The Daily Dashboard)</li>
</ol>



<ul class="wp-block-list">
<li><strong>What it is:</strong> Queries that compute periodic reports (e.g., Weekly Sales by State).</li>



<li><strong>What it tests:</strong> Ability to scan large amounts of data quickly and perform standard aggregations (SUM, AVG, GROUP BY).</li>
</ul>



<ol class="wp-block-list" start="2">
<li>Data Mining (The Deep Dive)</li>
</ol>



<ul class="wp-block-list">
<li><strong>What it is:</strong> Queries that look for hidden patterns or trends (e.g., Find items that are bought together).</li>



<li><strong>What it tests:</strong> Complex statistical functions, self-joins (joining a table to itself), and multiple sub-queries.</li>
</ul>



<ol class="wp-block-list" start="3">
<li>Ad-Hoc Analytics (The Random Question)</li>
</ol>



<ul class="wp-block-list">
<li><strong>What it is:</strong> Unpredictable questions to answer immediate business needs.</li>



<li><strong>What it tests:</strong> The <strong>Cost-Based Optimizer (CBO)</strong>. Since these queries are not pre-tuned, the engine must guess the best way to execute them on the fly.</li>
</ul>



<ol class="wp-block-list" start="4">
<li>Technical Complexity: Beyond the business content, the SQL logic itself is designed to break weak engines using:</li>
</ol>



<ul class="wp-block-list">
<li><strong>Common Table Expressions (CTEs):</strong> Using  <code class="">WITH</code> clauses to define temporary tables.</li>



<li><strong>Window Functions:</strong> Advanced math like <code class="">RANK()</code>, <code class="">LEAD()</code>, <code class="">LAG()</code>, and <code class="">PARTITION BY</code></li>



<li><strong>Set Operations:</strong> Using <code class="">UNION</code>, <code class="">INTERSECT</code>, and <code class="">EXCEPT</code>.</li>



<li><strong>Large Joins:</strong> Joining 10+ tables in a single query.</li>
</ul>



<p><strong>Query Complexity Levels:</strong></p>



<ul class="wp-block-list">
<li><strong>Simple</strong> (Queries 1-30): Basic aggregations, 1-3 table joins</li>



<li><strong>Medium</strong> (Queries 31-60): 4-7 table joins, subqueries, window functions</li>



<li><strong>Complex</strong> (Queries 61-99): 8+ table joins, nested subqueries, advanced analytics</li>
</ul>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>Presto has built-in <code class="">tpch</code> and <code class="">tpcds</code> connector that generates data on the fly. To enable the connector, just add a catalog <code class="">tpch.properties</code> and <code class="">tpcds.properties</code> respectively. Unlike other engines where you must generate files (CSV/Parquet) and load them into a database. Presto’s <code class="">tpch</code>, <code class="">tpcds</code> connectors are <strong>zero-storage</strong> and can be used for testing and education purposes only.</p>
</blockquote>



<p></p>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2720" height="742" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260130232201-1024x742.png" style="width: 729px; height: auto;" width="1024" /></figure>



<h2 class="wp-block-heading"><strong>Data Skew in TPC-DS : The Critical Differentiator</strong></h2>



<p>A defining characteristic of&nbsp;<strong>TPC-DS</strong>, distinguishing it from its predecessor&nbsp;<strong>TPC-H</strong>, is the presence of significant&nbsp;<strong>Data Skew</strong>.</p>



<ul class="wp-block-list">
<li><strong>TPC-H (Uniform Distribution):</strong> The TPC-H dataset generation follows a uniform probability distribution. Every partition key possesses roughly equal cardinality. While valid for baseline throughput testing, this vastly simplifies the resource management challenge by creating an artificially balanced cluster load.</li>



<li><strong>TPC-DS (Real-World Skew):</strong> TPC-DS models real-world retail phenomena using non-uniform, domain-specific statistical distributions that create realistic data skew. Certain items, dates, or customer IDs appear with orders of magnitude higher frequency than others, mimicking best-sellers or seasonal variations.</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2721" height="876" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260131015623-1024x876.png" style="width: 728px; height: auto;" width="1024" /></figure>



<h2 class="wp-block-heading"><strong>Breaking the Java Barrier: Why Presto C++ is the Future of TPC-H and TPC-DS Benchmarking</strong></h2>



<p>When analyzing <strong>TPC-H</strong> and <strong>TPC-DS</strong> benchmark results, it often hit a performance ceiling that isn&#8217;t caused by poor SQL, it’s caused by the <strong>Java Virtual Machine (JVM)</strong>. To reach the next frontier of data lakehouse performance, the Presto community has turned to <strong>Presto C++ (powered by Velox)</strong>.</p>



<p>If you are looking to scale your infrastructure while slashing cloud costs, here is why a native C++ worker is the single biggest architectural upgrade for your Presto cluster.</p>



<ul class="wp-block-list">
<li>1. Drastic Reduction in CPU Overhead</li>



<li>2. Relieving Memory Pressure with Fine-Grained Control</li>



<li>3. Eliminating the Hidden Tax of Garbage Collection (GC)</li>
</ul>



<p><a href="https://prestodb.io/docs/current/presto-cpp.html"><strong>Learn</strong></a> more about the Presto C++ native execution engine.</p>



<h2 class="wp-block-heading"><strong>Comparison at a Glance</strong></h2>



<figure class="wp-block-table"><table class="has-fixed-layout"><thead><tr><th>Feature</th><th>TPC-H</th><th>TPC-DS</th></tr></thead><tbody><tr><td><strong>Model</strong></td><td>Supply Chain (Parts &amp; Orders)</td><td>Retail (Stores, Web, Catalog)</td></tr><tr><td><strong>Schema</strong></td><td>3NF Normalized (8 Tables)</td><td>Constellation / Snowflake (24+ Tables)</td></tr><tr><td><strong>Query Count</strong></td><td>22</td><td>99</td></tr><tr><td><strong>Primary Test</strong></td><td>Join Efficiency &amp; Raw Scan Speed</td><td>Optimizer Intelligence &amp; Skew Handling</td></tr><tr><td><strong>Use Case</strong></td><td>System validation, regression testing.</td><td>Production readiness, optimizer tuning.</td></tr></tbody></table></figure>



<p></p>



<p>Refer to Presto Documentation on <strong><a href="https://prestodb.io/docs/current/connector/tpch.html">TPC-H</a> </strong>and <a href="https://prestodb.io/docs/current/connector/tpcds.html#"><strong>TPC-DS</strong></a> for more information.</p>



<h2 class="wp-block-heading">Follow Us </h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/01/30/tpc-h-vs-tpc-ds-benchmarking-modern-distributed-sql-engines-presto/">TPC-H vs TPC-DS : Benchmarking Modern Distributed SQL Engines like Presto</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
