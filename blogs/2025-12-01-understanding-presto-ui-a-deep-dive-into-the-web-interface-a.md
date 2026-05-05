---
title: "Understanding Presto UI: A Deep Dive into the Web Interface Architecture"
url: "https://prestodb.io/blog/2025/12/01/understanding-presto-ui-a-deep-dive-into-the-web-interface-architecture/"
date: "Mon, 01 Dec 2025 20:15:09 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<p><strong>Presto UI</strong> is a modern, React-based web interface that provides real-time monitoring, query management, and cluster administration capabilities for the Presto distributed SQL query engine. Whether you&#8217;re a database administrator, data engineer, or developer, Presto UI offers intuitive tools to visualize query execution, monitor cluster health, and interact with the Presto coordinator.<br /></p>



<h2 class="wp-block-heading"><strong>Key Benefits of Presto UI:</strong></h2>



<p><img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> <strong>Real-time Query Monitoring</strong> &#8211; Track running, queued, and completed queries<br /><img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> <strong>Cluster Health Dashboard</strong> &#8211; Monitor worker nodes and resource utilization<br /><img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> <strong>Interactive SQL Client</strong> &#8211; Execute queries directly from your browser<br /><img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> <strong>Performance Analytics</strong> &#8211; Analyze query execution plans and bottlenecks<br /><img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> <strong>Resource Management</strong> &#8211; View memory pools and resource group allocations</p>



<h2 class="wp-block-heading"><strong><strong>Architecture Overview</strong>&nbsp;</strong></h2>



<h3 class="wp-block-heading">Technology Stack&nbsp;</h3>



<p>The Presto UI&nbsp;leverages&nbsp;a modern JavaScript ecosystem:</p>



<figure class="wp-block-image size-full is-resized"><img alt="" class="wp-image-2637" height="252" src="https://prestodb.io/wp-content/uploads/image-97.png" style="width: 904px; height: auto;" width="925" /></figure>



<p><strong>Core Dependencies:</strong>&nbsp;</p>



<ul class="wp-block-list">
<li><strong><span class="stk-highlight" style="color: #000000;">React 18.3.1</span></strong>: Modern UI framework with concurrent features&nbsp;</li>



<li><strong>D3.js 7.9.0</strong>: Advanced data visualization for query plans and metrics&nbsp;</li>



<li><strong>dagre-d3-es</strong>: Graph visualization for query execution plans&nbsp;</li>



<li><strong>vis-timeline</strong>: Interactive timeline visualizations&nbsp;</li>



<li><strong><a href="https://github.com/prestodb/presto-js-client">@prestodb/presto-js-client</a></strong>: Official Presto JavaScript client&nbsp;</li>



<li><strong>react-data-table-component</strong>: Efficient data table rendering&nbsp;</li>



<li><strong>Prism.js</strong>: Syntax highlighting for SQL queries&nbsp;</li>
</ul>



<h3 class="wp-block-heading">Presto UI Build Configuration</h3>



<p>The UI uses a sophisticated Webpack configuration (<a href="https://github.com/prestodb/presto/blob/master/presto-ui/src/webpack.config.js" rel="noreferrer noopener" target="_blank"><strong>webpack.config.js</strong></a><strong>)</strong>&nbsp;that supports:&nbsp;</p>



<ul class="wp-block-list">
<li><strong>Multiple Entry Points</strong>: Separate bundles for different pages (index, query, plan, worker, etc.)&nbsp;</li>



<li><strong>Code Splitting</strong>: Automatic chunking for&nbsp;optimal&nbsp;loading performance&nbsp;</li>



<li><strong>Development Server</strong>: Built-in proxy for API calls during development&nbsp;</li>



<li><strong>Production Optimization</strong>: Terser minification and tree-shaking&nbsp;</li>
</ul>



<h2 class="wp-block-heading"><strong>Core Components: Understanding Presto UI Features</strong></h2>



<h3 class="wp-block-heading">1. Cluster Overview Dashboard &#8211; Real-time Monitoring</h3>



<p>The main landing page provides a real-time cluster health dashboard through the&nbsp;<a href="https://github.com/prestodb/presto/blob/master/presto-ui/src/components/ClusterHUD.jsx" rel="noreferrer noopener" target="_blank"><strong>ClusterHUD</strong></a><strong>&nbsp;</strong>component.</p>



<figure class="wp-block-image size-full is-resized"><img alt="" class="wp-image-2633" height="722" src="https://prestodb.io/wp-content/uploads/image-93.png" style="width: 664px; height: auto;" width="752" /></figure>



<p><strong>Key Performance Indicators (KPIs):</strong></p>



<ul class="wp-block-list">
<li><strong>Running Queries</strong>: Current active query count with trend sparkline</li>



<li><strong>Queued Queries</strong>: Queries waiting for execution resources</li>



<li><strong>Blocked Queries</strong>: Queries unable to progress due to resource constraints</li>



<li><strong>Active Workers</strong>: Number of healthy worker nodes in the cluster</li>



<li><strong>Running Drivers</strong>: Total parallel execution units across all queries</li>



<li><strong>Reserved Memory</strong>: Current memory allocation across the cluster</li>



<li><strong>Row Input Rate</strong>: Rows processed per second (moving average)</li>



<li><strong>Byte Input Rate</strong>: Data throughput in bytes per second</li>



<li><strong>Worker Parallelism</strong>: CPU utilization per worker node</li>
</ul>



<p>The&nbsp;component&nbsp;uses&nbsp;<strong>exponentially weighted moving averages</strong>&nbsp;for smooth metric visualization:&nbsp;</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">// Real-time metric calculation from ClusterHUD.jsx
newRowInputRate = addExponentiallyWeightedToHistory(
    rowsInputSinceRefresh / secsSinceRefresh, 
    this.state.rowInputRate
);</code></pre></div>



<p><strong>Refresh Interval</strong>: Updates every 1 second for real-time cluster monitoring.</p>



<h3 class="wp-block-heading">2. Query List Component&nbsp;</h3>



<p>The&nbsp;<a href="https://github.com/prestodb/presto/blob/master/presto-ui/src/components/QueryList.jsx" rel="noreferrer noopener" target="_blank"><strong>QueryList</strong></a>&nbsp;component is your command center for managing and monitoring all Presto queries with powerful filtering and sorting capabilities.</p>



<h5 class="wp-block-heading">How to Monitor Presto Queries:</h5>



<p><strong>Advanced Filtering Options:</strong></p>



<ul class="wp-block-list">
<li><strong>State Filters</strong>: Running, Queued, Finished queries</li>



<li><strong>Error Type Filters</strong>: Internal Error, External Error, Resources Error, User Error</li>



<li><strong>Full-text Search</strong>: Search across query ID, user, source, resource group, and SQL text</li>



<li><strong>Real-time Updates</strong>: Automatic refresh with configurable intervals (1s, 5s, 10s, 30s)</li>
</ul>



<p><strong>Query Sorting Capabilities:</strong></p>



<ul class="wp-block-list">
<li>Creation time (newest/oldest first)</li>



<li>Elapsed time (longest/shortest running)</li>



<li>CPU time (most/least intensive)</li>



<li>Execution time (actual processing time)</li>



<li>Current memory usage</li>



<li>Cumulative user memory</li>
</ul>



<p><strong>Comprehensive Query Information Display:</strong></p>



<p>Each query card in the Presto UI shows:</p>



<ul class="wp-block-list">
<li><strong>Query ID</strong>: Clickable link to detailed query analysis</li>



<li><strong>User &amp; Source</strong>: Who submitted the query and from which application</li>



<li><strong>Resource Group</strong>: Hierarchical resource allocation (clickable breadcrumb navigation)</li>



<li><strong>Driver Statistics</strong>: Completed, running, and queued drivers/splits</li>



<li><strong>Timing Metrics</strong>: Elapsed time, execution time, CPU time</li>



<li><strong>Memory Metrics</strong>: Current reservation, peak usage, cumulative memory</li>



<li><strong>Progress Bar</strong>: Visual state indicator with color-coded status</li>



<li><strong>SQL Preview</strong>: Formatted query text with intelligent truncation</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2635" height="338" src="https://prestodb.io/wp-content/uploads/image-95-1024x338.png" style="width: 997px; height: auto;" width="1024" /></figure>



<p><strong>Pro Tip</strong>: Use the search bar to quickly find queries by user, source application, or SQL keywords for faster troubleshooting.<br /></p>



<h3 class="wp-block-heading">3. Query Detail View</h3>



<p>The&nbsp;<a href="https://github.com/prestodb/presto/blob/master/presto-ui/src/components/QueryDetail.jsx" rel="noreferrer noopener" target="_blank"><strong>QueryDetail</strong></a>&nbsp;component provides comprehensive query analysis for performance optimization and debugging.</p>



<h5 class="wp-block-heading">Understanding Query Execution in Presto UI:</h5>



<p><strong>Session Information:</strong></p>



<ul class="wp-block-list">
<li>User identity and principal</li>



<li>Source application details</li>



<li>Catalog and schema context</li>



<li>Client IP address and connection tags</li>



<li>Session properties and configurations</li>



<li>Resource estimates and allocations</li>
</ul>



<p><strong>Execution Timeline Analysis:</strong></p>



<ul class="wp-block-list">
<li><strong>Submission Time</strong>: When the query was received</li>



<li><strong>Completion Time</strong>: When execution finished</li>



<li><strong>Prerequisites Wait Time</strong>: Time waiting for dependencies</li>



<li><strong>Queued Time</strong>: Time spent in the queue</li>



<li><strong>Planning Time</strong>: Query optimization duration</li>



<li><strong>Execution Time</strong>: Actual processing time</li>



<li><strong>Coordinator</strong>: Which node coordinated the query</li>
</ul>



<p><strong>Resource Utilization Metrics:</strong></p>



<ul class="wp-block-list">
<li><strong>CPU Time</strong>: Total CPU seconds consumed</li>



<li><strong>Scheduled Time</strong>: Time tasks were scheduled</li>



<li><strong>Input Rows/Data</strong>: Amount of data processed</li>



<li><strong>Output Rows/Data</strong>: Results produced</li>



<li><strong>Shuffled Data</strong>: Data transferred between stages</li>



<li><strong>Memory Metrics</strong>: Peak, cumulative, and current usage</li>



<li><strong>Spilled Data</strong>: Data written to disk (performance indicator)</li>
</ul>



<p><strong>Real-time Performance Sparklines:</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">// Live metrics updated every second
- Parallelism: CPU time rate showing query concurrency
- Scheduled Time/s: Task scheduling efficiency
- Input Rows/s: Data ingestion rate
- Input Bytes/s: Network throughput
- Memory Utilization: Current memory consumption</code></pre></div>



<p><strong>Stage-by-Stage Breakdown:</strong><br /></p>



<p>The Presto UI visualizes query execution stages with:</p>



<ul class="wp-block-list">
<li><strong>Stage Statistics</strong>: Time, memory, and task counts per stage</li>



<li><strong>Task Distribution</strong>: Histogram showing task balance across workers</li>



<li><strong>Skew Analysis</strong>: Scheduled time and CPU time distribution</li>



<li><strong>Expandable Task Lists</strong>: Individual task details with filtering</li>



<li><strong>Auto-Refresh Toggle</strong>: Freeze stage view for detailed analysis</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2642" height="425" src="https://prestodb.io/wp-content/uploads/image-102-1024x425.png" style="width: 1087px; height: auto;" width="1024" /></figure>



<h3 class="wp-block-heading">4. Presto SQL Client &#8211; Interactive Query Interface</h3>



<p>The&nbsp;<a href="https://github.com/prestodb/presto/blob/master/presto-ui/src/components/SQLClient.tsx" rel="noreferrer noopener" target="_blank"><strong>SQLClient</strong></a>&nbsp;component&nbsp;provides an interactive SQL query interface directly in the browser.&nbsp;</p>



<h4 class="wp-block-heading">How to Use the Presto SQL Client:</h4>



<p><strong>Query Execution Features:</strong></p>



<ul class="wp-block-list">
<li><strong>Syntax-Highlighted Editor</strong>: PrismJS-powered SQL editor with auto-formatting</li>



<li><strong>Catalog Selection</strong>: Choose your data source from available catalogs</li>



<li><strong>Schema Selection</strong>: Navigate database schemas easily</li>



<li><strong>Session Properties</strong>: Configure query behavior and optimizations</li>



<li><strong>Real-time Execution</strong>: Submit queries and see results instantly</li>
</ul>



<p><strong>Results Display:</strong></p>



<ul class="wp-block-list">
<li><strong>Tabular Presentation</strong>: Clean, sortable data tables</li>



<li><strong>Column Sorting</strong>: Click headers to sort results</li>



<li><strong>Export Options</strong>: Download results in various formats</li>



<li><strong>Error Messages</strong>: Clear, actionable error information</li>



<li><strong>Query Statistics</strong>: Execution time and row counts</li>
</ul>



<p><strong>Security Considerations:</strong></p>



<p>The SQL client includes a prominent security warning:</p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>&#8220;SQL client directly accesses the coordinator APIs and submits SQL queries. Users who can access the Web UI can use this client to query, update, and even delete data in the catalogs.&#8221;</p>
</blockquote>



<figure class="wp-block-image size-full is-resized"><img alt="" class="wp-image-2640" height="440" src="https://prestodb.io/wp-content/uploads/image-100.png" style="width: 667px; height: auto;" width="725" /></figure>



<h3 class="wp-block-heading">5. Worker Status View &#8211; Node-Level Monitoring</h3>



<p>The&nbsp;<a href="https://github.com/prestodb/presto/blob/master/presto-ui/src/components/WorkerStatus.jsx" rel="noreferrer noopener" target="_blank"><strong>WorkerStatus</strong></a>&nbsp;component provides detailed monitoring of individual Presto worker nodes.</p>



<h4 class="wp-block-heading">Worker Node Metrics:</h4>



<p><strong>System Overview:</strong></p>



<ul class="wp-block-list">
<li><strong>Node ID</strong>: Unique worker identifier</li>



<li><strong>Heap Memory</strong>: Total JVM heap capacity</li>



<li><strong>Processor Count</strong>: Available CPU cores</li>



<li><strong>Uptime</strong>: Time since worker started</li>



<li><strong>External Address</strong>: Public-facing network address</li>



<li><strong>Internal Address</strong>: Cluster-internal network address</li>
</ul>



<p><strong>Resource Utilization Monitoring:</strong></p>



<ul class="wp-block-list">
<li><strong>Process CPU Utilization</strong>: CPU usage by Presto process (sparkline)</li>



<li><strong>System CPU Utilization</strong>: Overall system CPU usage (sparkline)</li>



<li><strong>Heap Utilization</strong>: JVM heap memory percentage (sparkline)</li>



<li><strong>Non-Heap Memory</strong>: Off-heap memory usage (sparkline)</li>
</ul>



<p><strong>Memory Pool Management:</strong></p>



<p>Presto UI visualizes two critical memory pools:</p>



<ol class="wp-block-list">
<li><strong>General Pool</strong>: Memory for query execution
<ul class="wp-block-list">
<li>Total capacity</li>



<li>Reserved (non-revocable) memory</li>



<li>Revocable memory</li>



<li>Free memory</li>



<li>Per-query breakdown</li>
</ul>
</li>



<li><strong>Reserved Pool</strong>: Memory for system operations
<ul class="wp-block-list">
<li>Same metrics as General Pool</li>



<li>System-level reservations</li>



<li>Query-specific allocations</li>
</ul>
</li>
</ol>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2641" height="517" src="https://prestodb.io/wp-content/uploads/image-101-1024x517.png" style="width: 862px; height: auto;" width="1024" /></figure>



<h3 class="wp-block-heading">6. Resource Group View&nbsp;</h3>



<p>The resource group&nbsp;component&nbsp;provides hierarchical visualization of query resource allocation and management.&nbsp;</p>



<h3 class="wp-block-heading">7. Query Plan Visualization</h3>



<p>The UI includes sophisticated query plan visualization using <strong>D3.js</strong> and <strong>dagre-d3</strong>.</p>



<p><strong>Features:</strong>&nbsp;</p>



<ul class="wp-block-list">
<li>Interactive DAG (Directed Acyclic Graph) representation&nbsp;</li>



<li>Stage dependencies and data flow&nbsp;</li>



<li>Operator-level details&nbsp;</li>



<li>Zoom and pan capabilities&nbsp;</li>



<li>Export to various formats</li>
</ul>



<h2 class="wp-block-heading"><strong>Data Flow Architecture: How Presto UI Communicates<br /></strong></h2>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2643" height="573" src="https://prestodb.io/wp-content/uploads/image-103-1024x573.png" style="width: 924px; height: auto;" width="1024" /></figure>



<h4 class="wp-block-heading">Presto REST API Endpoints</h4>



<p>The UI communicates with these key coordinator endpoints:</p>



<p><strong>Cluster Monitoring:</strong></p>



<ul class="wp-block-list">
<li><code class="">/v1/cluster</code>&nbsp;&#8211; Cluster-wide statistics and health metrics</li>



<li><code class="">/v1/worker/{nodeId}/status</code>&nbsp;&#8211; Individual worker node status</li>
</ul>



<p><strong>Query Management:</strong></p>



<ul class="wp-block-list">
<li><code class="">/v1/queryState</code>&nbsp;&#8211; List of all queries with basic information</li>



<li><code class="">/v1/query/{queryId}</code>&nbsp;&#8211; Detailed query information including stages</li>



<li><code class="">/v1/taskInfo/{taskId}</code>&nbsp;&#8211; Individual task execution details</li>
</ul>



<p><strong>Resource Management:</strong></p>



<p><code class="">/v1/resourceGroupState</code>&nbsp;&#8211; Resource group configurations and usage<br /></p>



<h2 class="wp-block-heading"><strong>Performance Optimizations: How Presto UI Stays Fast</strong></h2>



<h3 class="wp-block-heading">1. Lazy Loading&nbsp;</h3>



<p>The UI uses a custom lazy loading mechanism (<a href="https://github.com/prestodb/presto/blob/master/presto-ui/src/lazy.tsx" rel="noreferrer noopener" target="_blank"><strong>lazy.jsx</strong></a>) for code splitting:&nbsp;</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">// Components load on-demand
const ClusterHUD = lazy(&#039;ClusterHUD&#039;);
const QueryList = lazy(&#039;QueryList&#039;);
const WorkerStatus = lazy(&#039;WorkerStatus&#039;);</code></pre></div>



<h3 class="wp-block-heading">2. Efficient Re-rendering Strategy</h3>



<p>Components implement intelligent update strategies:</p>



<ul class="wp-block-list">
<li><strong>Throttled Sparkline Rendering</strong>: 1-second minimum interval prevents excessive redraws</li>



<li><strong>Incremental Query List Updates</strong>: Only changed queries re-render</li>



<li><strong>Conditional Stage Refresh</strong>: Auto-refresh toggle for detailed analysis</li>



<li><strong>Memoized Calculations</strong>: Expensive computations cached</li>
</ul>



<h3 class="wp-block-heading">3. Data Aggregation Techniques</h3>



<p>Moving averages and exponentially weighted calculations:</p>



<ul class="wp-block-list">
<li>Reduce data noise in visualizations</li>



<li>Smooth out temporary spikes</li>



<li>Provide accurate trend analysis</li>



<li>Minimize memory footprint</li>
</ul>



<h2 class="wp-block-heading"><strong>Deployment Architecture: How to Deploy Presto UI</strong></h2>



<figure class="wp-block-image size-full"><img alt="" class="wp-image-2644" height="502" src="https://prestodb.io/wp-content/uploads/image-104.png" width="773" /></figure>



<h4 class="wp-block-heading"><strong><br /></strong>Build Process Steps:</h4>



<ol class="wp-block-list">
<li><strong>Maven Trigger</strong>:&nbsp;<code class="">frontend-maven-plugin</code>&nbsp;initiates build</li>



<li><strong>Node.js Installation</strong>: v22.15.1 installed automatically</li>



<li><strong>Yarn Installation</strong>: v1.22.22 for dependency management</li>



<li><strong>Webpack Bundling</strong>: Optimized production bundles created</li>



<li><strong>Asset Copying</strong>: Static files moved to&nbsp;<code class="">target/webapp</code></li>



<li><strong>Server Integration</strong>: Presto server serves UI at root path</li>
</ol>



<h2 class="wp-block-heading"><strong>Ready to Contribute? Start Your Presto UI Journey Today!</strong></h2>



<h3 class="wp-block-heading">Find Your First Frontend Issue</h3>



<p>The Presto community welcomes frontend developers of all skill levels! We&#8217;ve made it easy to find issues that match your expertise and interests.</p>



<h3 class="wp-block-heading">Browse WebUI Issues on <a href="https://github.com/prestodb/presto">GitHub</a></h3>



<p>All frontend-related issues are tagged with the <code class="">webui</code> label for easy discovery:</p>



<p><strong><img alt="🔗" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f517.png" style="height: 1em;" /> <a href="https://github.com/prestodb/presto/issues?q=is%3Aissue%20state%3Aopen%20label%3Awebui">View All WebUI Issues on GitHub</a></strong></p>



<p class="has-large-font-size">Follow Presto at&nbsp;<a href="https://www.linkedin.com/company/presto-foundation/" rel="noreferrer noopener" target="_blank">Linkedin</a>,&nbsp;<a href="https://www.youtube.com/@PrestoFoundation" rel="noreferrer noopener" target="_blank">Youtube</a>,&nbsp;and Join&nbsp;<a href="https://communityinviter.com/apps/prestodb/prestodb" rel="noreferrer noopener" target="_blank">Slack</a>&nbsp;channel to interact with the community.</p>
<p>The post <a href="https://prestodb.io/blog/2025/12/01/understanding-presto-ui-a-deep-dive-into-the-web-interface-architecture/">Understanding Presto UI: A Deep Dive into the Web Interface Architecture </a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
