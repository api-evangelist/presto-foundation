---
title: "Presto Benchmarking Tutorial – TPC-H & TPC-DS on Iceberg with Google Cloud Storage (GCS)"
url: "https://prestodb.io/blog/2026/05/03/presto-benchmarking-tutorial-tpc-h-tpc-ds-on-iceberg-with-google-cloud-storage-gcs/"
date: "Sun, 03 May 2026 15:40:50 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<h2 class="wp-block-heading">TL;DR: What You Will Build</h2>



<p>In this comprehensive guide, you will deploy a&nbsp;<strong>Presto</strong>&nbsp;benchmarking setup using&nbsp;<strong>Docker Compose</strong>. We will construct a cloud-native&nbsp;<strong>Data Lakehouse</strong>&nbsp;by mapping raw data into&nbsp;<strong>Apache Iceberg</strong>&nbsp;tables stored on&nbsp;<strong>Google Cloud Storage (GCS)</strong>. Finally, we will execute the industry-standard&nbsp;<strong>TPC-H</strong>&nbsp;and&nbsp;<strong>TPC-DS</strong>&nbsp;benchmark suites using&nbsp;<strong>PBench</strong>, and visualize our query latencies in real-time using a persistent&nbsp;<strong>MySQL</strong>&nbsp;and&nbsp;<strong>Grafana</strong>&nbsp;observability stack.</p>



<p>Whether you are stress-testing JVM garbage collection, validating Iceberg partitioning, or evaluating Presto query performance at scale, this tutorial provides the blueprint.</p>



<h3 class="wp-block-heading">Prerequisites</h3>



<figure class="wp-block-table"><table class="has-fixed-layout"><thead><tr><th>Component</th><th class="has-text-align-left">Role</th></tr></thead><tbody><tr><td><strong>Docker &amp; Docker Compose</strong></td><td class="has-text-align-left">The orchestration layer that provides a consistent, isolated environment for all services.</td></tr><tr><td><strong>Presto (Java)</strong></td><td class="has-text-align-left">The distributed SQL query engine. The&nbsp;<strong>Coordinator</strong>&nbsp;handles query planning and metadata retrieval, while the&nbsp;<strong>Workers</strong>&nbsp;perform the actual data processing.</td></tr><tr><td><strong>Hive Metastore (HMS)</strong></td><td class="has-text-align-left">Acts as the <strong>Catalog Service</strong>. It stores the mapping between Presto table names and their physical locations on GCS.</td></tr><tr><td><strong>PostgreSQL</strong></td><td class="has-text-align-left">The persistent storage for the Hive Metastore, ensuring that your table schemas and metadata survive container restarts.</td></tr><tr><td><strong>Google Cloud Storage</strong></td><td class="has-text-align-left">An object storage, where your raw or Iceberg/Parquet data resides.</td></tr><tr><td><strong>PBench</strong></td><td class="has-text-align-left">The benchmark orchestrator. It executes the TPC-H/DS suites and captures the fine-grained execution metrics for each query. Download it from <a href="https://github.com/prestodb/pbench/releases" rel="noreferrer noopener" target="_blank">here.</a></td></tr><tr><td><strong>MySQL 8.0</strong></td><td class="has-text-align-left">A dedicated database used to archive benchmark results, enabling long-term performance trend analysis.</td></tr><tr><td><strong>Grafana</strong></td><td class="has-text-align-left">The visualization layer that transforms raw MySQL metrics into real-time, interactive performance dashboards.</td></tr></tbody></table></figure>



<p></p>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-9bfe9d5 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-9bfe9d5-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-kynu9a4"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-kynu9a4" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-rn98aa3"><p class="stk-block-text__text"><strong>Important</strong><br />GCS Service Account: Download JSON key file (<code class="">gcs-key.json</code>) with&nbsp;<code class="">Storage Admin</code>&nbsp;permissions and save inside root directory.</p></div>
</div></div></blockquote>



<p></p>



<h3 class="wp-block-heading">Infrastructure Deployment</h3>



<h4 class="wp-block-heading">1. Define the Services</h4>



<p>Create a&nbsp;<code class="">docker-compose.yml</code>&nbsp;that maps the Presto cluster to your local environment, ensuring the&nbsp;<strong>MySQL metrics database</strong>&nbsp;is mapped to a non-standard port (e.g.,&nbsp;<strong>3307</strong>) to avoid host conflicts.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">services:

  metastore-db-presto:
    image: postgres:15
    container_name: metastore-db-presto
    hostname: metastore-db-presto
    environment:
      POSTGRES_DB: metastore
      POSTGRES_USER: hive
      POSTGRES_PASSWORD: hive_password
    volumes:
      - metastore-db-data-presto:/var/lib/postgresql/data
    healthcheck:
      test: &#091;&quot;CMD-SHELL&quot;, &quot;pg_isready -U hive -d metastore&quot;]
      interval: 5s
      timeout: 5s
      retries: 10
    networks:
      - presto-network

  hive-metastore-presto:
    image: apache/hive:3.1.3
    container_name: hive-metastore-presto
    hostname: hive-metastore-presto
    platform: linux/amd64
    depends_on:
      metastore-db-presto:
        condition: service_healthy
    environment:
      SERVICE_NAME: metastore
      DB_DRIVER: postgres
      IS_RESUME: &quot;true&quot;

    volumes:
      - ./hive-metastore/hive-site.xml:/opt/hive/conf/hive-site.xml:ro
      - ./hive-metastore/core-site.xml:/opt/hive/conf/core-site.xml:ro
      - ./gcs-key.json:/opt/hive/conf/gcs-key.json:ro
      - ./hive-metastore/postgresql-42.7.2.jar:/opt/hive/lib/postgresql-42.7.2.jar:ro
      - ./hive-metastore/gcs-connector-hadoop3-shaded.jar:/opt/hive/lib/gcs-connector-hadoop3-shaded.jar:ro
    ports:
      - &quot;9083:9083&quot;
    healthcheck:
      test: &#091;&quot;CMD&quot;, &quot;bash&quot;, &quot;-c&quot;, &quot;cat &lt; /dev/null &gt; /dev/tcp/localhost/9083&quot;]
      interval: 10s
      timeout: 10s
      retries: 15
      start_period: 30s
    networks:
      - presto-network

  presto-coordinator:
    image: prestodb/presto:latest
    container_name: presto-coordinator
    hostname: presto-coordinator
    depends_on:
      hive-metastore-presto:
        condition: service_healthy
    volumes:
      - ./coordinator/etc:/opt/presto-server/etc:ro
      - ./gcs-key.json:/etc/presto/gcs-key.json:ro
      - ./hive-metastore/core-site.xml:/etc/presto/core-site.xml:ro
      - ./presto-spill:/tmp/presto-spill
    ports:
      - &quot;8080:8080&quot;
    networks:
      - presto-network

  presto-worker:
    image: prestodb/presto:latest
    container_name: presto-worker
    hostname: presto-worker
    depends_on:
      - presto-coordinator
    environment:
      - GOOGLE_APPLICATION_CREDENTIALS=/etc/presto/gcs-key.json
    volumes:
      - ./worker/etc:/opt/presto-server/etc:ro
      - ./gcs-key.json:/etc/presto/gcs-key.json:ro
      - ./hive-metastore/core-site.xml:/etc/presto/core-site.xml:ro
      - ./presto-spill:/tmp/presto-spill
    networks:
      - presto-network

  mysql:
    image: mysql:8.0
    container_name: mysql
    hostname: mysql
    environment:
      - MYSQL_DATABASE=pbench
      - MYSQL_USER=pbench
      - MYSQL_PASSWORD=pbench_password
      - MYSQL_ROOT_PASSWORD=root_password
    ports:
      - &quot;3307:3306&quot;
    networks:
      - presto-network
    healthcheck:
      test: &#091;&quot;CMD&quot;, &quot;mysqladmin&quot; ,&quot;ping&quot;, &quot;-h&quot;, &quot;localhost&quot;]
      interval: 10s
      timeout: 5s
      retries: 10

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    hostname: grafana
    depends_on:
      mysql:
        condition: service_healthy
    ports:
      - &quot;3000:3000&quot;
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    networks:
      - presto-network

volumes:
  metastore-db-data-presto:
  mysql-data:

networks:
  presto-network:
    driver: bridge</code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-2ac18ff is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-2ac18ff-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-sm6nd16"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-sm6nd16" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-qynno6f"><p class="stk-block-text__text"><strong>Tip</strong><br />If you would like to install Presto manually then follow this <a href="https://prestodb.io/blog/2025/07/15/presto-installation-a-step-by-step-guide-to-run-sql-queries/" rel="noreferrer noopener" target="_blank">guide</a></p></div>
</div></div></blockquote>



<p></p>



<p>To configure coordinator, create <code class="">coordinator</code> folder inside root directory and <code class="">etc</code> folder inside coordinator and add following configurations:</p>



<p><strong>./coordinator/etc/config.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">coordinator=true
node-scheduler.include-coordinator=false
discovery-server.enabled=true
discovery.uri=http://presto-coordinator:8080
http-server.http.port=8080
query.max-memory=6GB
query.max-memory-per-node=5GB
query.max-total-memory-per-node=6GB
experimental.spill-enabled=true
experimental.query-max-spill-per-node=100GB
experimental.spiller-spill-path=/tmp/presto-spill</code></pre></div>



<p><strong>./coordinator/etc/jvm.config</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-server
-Xmx12G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
-XX:ReservedCodeCacheSize=512M
-Djdk.attach.allowAttachSelf=true

# Java 17 module system opens (required for Presto internals)
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.lang.invoke=ALL-UNNAMED
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED
--add-opens=java.base/java.io=ALL-UNNAMED
--add-opens=java.base/java.net=ALL-UNNAMED
--add-opens=java.base/java.nio=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED
--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
--add-opens=java.base/sun.nio.cs=ALL-UNNAMED
--add-opens=java.base/sun.security.action=ALL-UNNAMED
--add-opens=java.base/sun.util.calendar=ALL-UNNAMED
--add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED</code></pre></div>



<p><strong>./coordinator/etc/node.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">node.environment=docker
node.id=coordinator-001
node.data-dir=/var/presto/data
node.location=/docker/coordinator</code></pre></div>



<p>Create <code class="">catalog</code> folder inside <code class="">etc</code> and add following configurations:</p>



<p><strong>./coordinator/etc/catalog/hive.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">connector.name=hive-hadoop2
hive.metastore.uri=thrift://hive-metastore-presto:9083
hive.config.resources=/etc/presto/core-site.xml
hive.gcs.json-key-file-path=/etc/presto/gcs-key.json
hive.allow-drop-table=true
hive.allow-rename-table=true
hive.allow-add-column=true
hive.allow-drop-column=true</code></pre></div>



<p><strong>./coordinator/etc/catalog/iceberg.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">connector.name=iceberg
iceberg.catalog.type=hive
hive.metastore.uri=thrift://hive-metastore-presto:9083
hive.config.resources=/etc/presto/core-site.xml
hive.gcs.json-key-file-path=/etc/presto/gcs-key.json
iceberg.file-format=PARQUET
iceberg.max-partitions-per-writer=5000</code></pre></div>



<p><strong>./coordinator/etc/catalog/tpcds.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">connector.name=tpcds
tpcds.use-varchar-type=true</code></pre></div>



<p><strong>./coordinator/etc/catalog/tpch.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">connector.name=tpch
tpch.use-varchar-type=true</code></pre></div>



<p>To configure workers, create <code class="">worker</code> folder inside root directory and <code class="">etc</code> folder inside worker and add following configurations:</p>



<p><strong>./worker/etc/config.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">coordinator=false
discovery.uri=http://presto-coordinator:8080
http-server.http.port=8080
query.max-memory=6GB
query.max-memory-per-node=5GB
query.max-total-memory-per-node=6GB
experimental.spill-enabled=true
experimental.query-max-spill-per-node=100GB
experimental.spiller-spill-path=/tmp/presto-spill</code></pre></div>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p><code class="">jvm.config</code>&nbsp;<em>(Identical to the Coordinator specifications)</em></p>
</blockquote>



<p><strong>./worker/etc/node.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">node.environment=docker
node.id=worker-001
node.data-dir=/var/presto/data
node.location=/docker/worker</code></pre></div>



<p><strong>./worker/etc/catalog/hive.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">connector.name=hive-hadoop2
hive.metastore.uri=thrift://hive-metastore-presto:9083
hive.config.resources=/etc/presto/core-site.xml
hive.gcs.json-key-file-path=/etc/presto/gcs-key.json</code></pre></div>



<p><strong>./worker/etc/catalog/iceberg.properties</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">connector.name=iceberg
iceberg.catalog.type=hive
hive.metastore.uri=thrift://hive-metastore-presto:9083
hive.config.resources=/etc/presto/core-site.xml
iceberg.file-format=PARQUET
hive.gcs.json-key-file-path=/etc/presto/gcs-key.json</code></pre></div>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>The&nbsp;<code class="">tpch</code>&nbsp;and&nbsp;<code class="">tpcds</code>&nbsp;catalog configurations are identical to those on the Coordinator node.</p>
</blockquote>



<h4 class="wp-block-heading">2. Configure GCS Connectivity</h4>



<p>Presto requires a <code class="">core-site.xml</code> to handle <code class="">gs://</code> URIs. This file must be mounted into the Presto and Hive Metastore containers. Create a <code class="">metastore</code> folder inside root directory to hold <code class="">core-site.xml</code>, <code class="">hive-site.xml</code> and required JARs.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">&lt;!--core-site.xml--&gt;

&lt;configuration&gt;
&lt;!-- Register GCS as a Hadoop filesystem --&gt;
&lt;property&gt;
&lt;name&gt;fs.gs.impl&lt;/name&gt;
&lt;value&gt;com.google.cloud.hadoop.fs.gcs.GoogleHadoopFileSystem&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;fs.AbstractFileSystem.gs.impl&lt;/name&gt;
&lt;value&gt;com.google.cloud.hadoop.fs.gcs.GoogleHadoopFS&lt;/value&gt;
&lt;/property&gt;

&lt;!-- Service Account Authentication --&gt;
&lt;property&gt;
&lt;name&gt;google.cloud.auth.service.account.enable&lt;/name&gt;
&lt;value&gt;true&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;google.cloud.auth.service.account.json.keyfile&lt;/name&gt;
&lt;value&gt;/opt/hive/conf/gcs-key.json&lt;/value&gt;
&lt;/property&gt;
&lt;/configuration&gt;</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">&lt;!--hive-site.xml--&gt;

&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;?xml-stylesheet type=&quot;text/xsl&quot; href=&quot;configuration.xsl&quot;?&gt;
&lt;configuration&gt;
&lt;property&gt;
&lt;name&gt;javax.jdo.option.ConnectionURL&lt;/name&gt;
&lt;value&gt;jdbc:postgresql://metastore-db-presto:5432/metastore&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;javax.jdo.option.ConnectionDriverName&lt;/name&gt;
&lt;value&gt;org.postgresql.Driver&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;javax.jdo.option.ConnectionUserName&lt;/name&gt;
&lt;value&gt;hive&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;javax.jdo.option.ConnectionPassword&lt;/name&gt;
&lt;value&gt;hive_password&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;hive.metastore.uris&lt;/name&gt;
&lt;value&gt;thrift://0.0.0.0:9083&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;hive.metastore.warehouse.dir&lt;/name&gt;
&lt;value&gt;/opt/hive/warehouse&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;datanucleus.autoCreateSchema&lt;/name&gt;
&lt;value&gt;false&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;datanucleus.fixedDatastore&lt;/name&gt;
&lt;value&gt;true&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;fs.gs.impl&lt;/name&gt;
&lt;value&gt;com.google.cloud.hadoop.fs.gcs.GoogleHadoopFileSystem&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;fs.AbstractFileSystem.gs.impl&lt;/name&gt;
&lt;value&gt;com.google.cloud.hadoop.fs.gcs.GoogleHadoopFS&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;fs.gs.project.id&lt;/name&gt;
&lt;value&gt;Your Google Cloud Project ID&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;google.cloud.auth.service.account.enable&lt;/name&gt;
&lt;value&gt;true&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;google.cloud.auth.service.account.json.keyfile&lt;/name&gt;
&lt;value&gt;/opt/hive/conf/gcs-key.json&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
&lt;name&gt;metastore.storage.schema.reader.impl&lt;/name&gt;
&lt;value&gt;org.apache.hadoop.hive.metastore.SerDeStorageSchemaReader&lt;/value&gt;
&lt;/property&gt;
&lt;/configuration&gt;</code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-67c9251 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-67c9251-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-zg6fevo"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-zg6fevo" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-jgwfaxi"><p class="stk-block-text__text"><strong>Info<br /></strong>Hive Metastore image doesn&#8217;t come with the plugins required to talk to <strong>PostgreSQL</strong> or <strong>Google Cloud Storage</strong>. You need 2 JAR files (for PostgreSQL and GCS Connectors), download them from <a href="https://github.com/saurabhmahawar/presto_databenchmarking/tree/main/hive-metastore" rel="noreferrer noopener" target="_blank"><strong>here</strong></a></p></div>
</div></div></blockquote>



<p></p>



<h4 class="wp-block-heading">3. Mapping Schemas &amp; External Tables</h4>



<p>To query raw TPC-H/DS data stored as&nbsp;<code class="">.tbl</code> or <code class="">.dat</code>&nbsp;files, we must first &#8220;bridge&#8221; the physical storage to Presto&#8217;s SQL engine. We do this by creating a&nbsp;<strong>Raw Staging Layer</strong>&nbsp;using the Hive connector. This layer maps the existing files as external tables without moving or duplicating any data. Raw data has been generated through official TPC tools using <code class="">dbgen</code> and <code class="">dsdgen</code>.</p>



<p>We use a dedicated schema with the&nbsp;<code class="">_raw</code>&nbsp;suffix to clearly separate our staging metadata from our optimized production data.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- Create a persistent schema for raw staging metadata
CREATE SCHEMA IF NOT EXISTS hive.tpch_sf10_raw
WITH (location = &#039;gs://data_benchmarking/raw/tpch_sf10_raw/&#039;);</code></pre></div>



<p>We create an&nbsp;<strong>External Table</strong>&nbsp;that points directly to the GCS bucket.</p>



<p><strong>Note</strong>: In this staging layer, we define all columns as&nbsp;<code class="">VARCHAR</code>. We will perform strict type casting later during the migration to Iceberg.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- Create an external table pointing to your GCS location
CREATE TABLE hive.tpch_sf10_raw.nation_raw (
    n_nationkey  VARCHAR,
    n_name       VARCHAR,
    n_regionkey  VARCHAR,
    n_comment    VARCHAR
)
WITH (
    format = &#039;CSV&#039;,
    csv_separator = &#039;|&#039;,
    external_location = &#039;gs://data_benchmarking/raw/tpch_sf10/nation/&#039;
);</code></pre></div>



<h4 class="wp-block-heading">4. The Iceberg Layer</h4>



<p>Once our raw data is queryable, we migrate it into&nbsp;<strong>Apache Iceberg</strong>. This transforms the data into an optimized, columnar Parquet format, adds ACID compliance, and enables Presto’s advanced performance features like hidden partitioning and time-travel.</p>



<p>Unlike the raw layer, this schema uses the&nbsp;<strong>Iceberg connector</strong>&nbsp;and will store its metadata in the Hive Metastore.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- Create the Iceberg schema
CREATE SCHEMA IF NOT EXISTS iceberg.tpch_sf10
WITH (location = &#039;gs://data_benchmarking/iceberg/tpch_sf10/&#039;);</code></pre></div>



<p>Now we define the table with the actual data types (Integers, Decimals, etc.) required for high-speed analytical joins.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- Migrate to Iceberg (with Casting)

CREATE TABLE iceberg.tpch_sf10.nation
WITH (format = &#039;PARQUET&#039;)
AS SELECT 
    CAST(n_nationkey AS BIGINT) AS n_nationkey,
    n_name,
    CAST(n_regionkey AS BIGINT) AS n_regionkey,
    n_comment
FROM hive.tpch_sf10_raw.nation_raw;</code></pre></div>



<p>Scanning flat tables at SF-10, SF-100 or SF-1000 will result in massive GCS costs and slow queries. To prevent this, we use&nbsp;<strong>Iceberg Partitioning</strong> directly in the&nbsp;<code class="">WITH</code>&nbsp;clause using a bucketing strategy.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- Example
CREATE TABLE iceberg.tpch_sf10.lineitem
WITH (
    format = &#039;PARQUET&#039;,
    location = &#039;gs://data_benchmarking/iceberg/tpch_sf10/lineitem/&#039;,
    partitioning = ARRAY&#091;&#039;month(l_shipdate)&#039;]
)
AS
SELECT 
    CAST(l_orderkey AS BIGINT) AS l_orderkey,
    CAST(l_partkey AS BIGINT) AS l_partkey,
    CAST(l_suppkey AS BIGINT) AS l_suppkey,
    CAST(l_linenumber AS INTEGER) AS l_linenumber,
    CAST(l_quantity AS DECIMAL(15,2)) AS l_quantity,
    CAST(l_extendedprice AS DECIMAL(15,2)) AS l_extendedprice,
    CAST(l_discount AS DECIMAL(15,2)) AS l_discount,
    CAST(l_tax AS DECIMAL(15,2)) AS l_tax,
    l_returnflag,
    l_linestatus,
    CAST(l_shipdate AS DATE) AS l_shipdate,
    CAST(l_commitdate AS DATE) AS l_commitdate,
    CAST(l_receiptdate AS DATE) AS l_receiptdate,
    l_shipinstruct,
    l_shipmode,
    l_comment
FROM hive.tpch_sf10_raw.lineitem_raw;</code></pre></div>



<p>The final, most important step for benchmarking is to collect statistics. This allows Presto&#8217;s&nbsp;<strong>Cost-Based Optimizer (CBO)</strong>&nbsp;to choose the most efficient join order.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- Collect row counts and column distributions
ANALYZE iceberg.tpch_sf10.nation;</code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-656ea36 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-656ea36-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-n9yl2qd"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-n9yl2qd" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-b2wxcvf"><p class="stk-block-text__text"><strong>Info</strong><br />Repeat Steps 3 and 4 for all the remaining tables in TPC-H and TPC-DS</p></div>
</div></div></blockquote>



<p></p>



<h4 class="wp-block-heading">5. Automated Benchmarking with PBench</h4>



<p>To get repeatable, professional results, we use&nbsp;<strong><a href="https://github.com/prestodb/pbench/releases" rel="noreferrer noopener" target="_blank">PBench</a></strong>, a high-performance benchmark orchestrator that automates query execution and captures detailed performance metrics. PBench doesn&#8217;t just print results to the console; it persists them in a MySQL database. This allows us to compare &#8220;Run A&#8221; vs &#8220;Run B&#8221; weeks later.</p>



<p>Download the PBench tar file and unpack it into your local project root directory.</p>



<p>Create <strong>mysql.json</strong> file inside pbench directory and paste following config</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">{
  &quot;username&quot;: &quot;pbench&quot;,
  &quot;password&quot;: &quot;pbench_password&quot;,
  &quot;server&quot;: &quot;127.0.0.1:3307&quot;,
  &quot;database&quot;: &quot;pbench&quot;
}</code></pre></div>



<p>Next, we create a JSON manifest that tells PBench which catalog to use, which schema to target, and which SQL files to execute. Notice that we also enable&nbsp;<code class="">save_json</code>&nbsp;and&nbsp;<code class="">save_output</code> for deep debugging.</p>



<p><strong>Example&nbsp;<code class="">tpch_sf10_iceberg.json</code>:</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">{
  &quot;name&quot;: &quot;tpch_sf10_iceberg&quot;,
  &quot;description&quot;: &quot;TPC-H SF-10 Benchmarking on Iceberg&quot;,
  &quot;catalog&quot;: &quot;iceberg&quot;,
  &quot;schema&quot;: &quot;tpch_sf10&quot;,
  &quot;runs&quot;: 1,
  &quot;save_json&quot;: true,
  &quot;save_output&quot;: true,
  &quot;query_files&quot;: &#091;
    &quot;benchmarks/tpch/queries/query_01.sql&quot;,
    &quot;benchmarks/tpch/queries/query_02.sql&quot;
    // ... continues through query_22.sql
  ]
}</code></pre></div>



<p><strong>Example&nbsp;<code class="">tpcds_sf10_iceberg.json</code>:</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">{
  &quot;name&quot;: &quot;tpcds_sf10_iceberg&quot;,
  &quot;description&quot;: &quot;TPC-DS SF-10 Benchmarking on Iceberg&quot;,
  &quot;catalog&quot;: &quot;iceberg&quot;,
  &quot;schema&quot;: &quot;tpcds_sf10&quot;,
  &quot;runs&quot;: 1,
  &quot;save_json&quot;: true,
  &quot;save_output&quot;: true,
  &quot;query_files&quot;: &#091;
    &quot;benchmarks/tpc-ds/queries/query_01.sql&quot;,
    &quot;benchmarks/tpc-ds/queries/query_02.sql&quot;,
	// ... continues through query_99.sql
  ]
}</code></pre></div>



<p>Run the following command to start the suite. PBench will sequentially execute every query, measure the wall time, and stream the results directly to your MySQL container.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">#For TPC-H Benchmarking
./pbench run --mysql mysql.json tpch_sf10_iceberg.json</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">#For TPC-DS Benchmarking
./pbench run --mysql mysql.json tpcds_sf10_iceberg.json</code></pre></div>



<p><code class="">pbench</code> will also save the raw data results and the execution statistics locally into a new folder (e.g.,&nbsp;<code class="">pbench/tpcds_sf10_iceberg_260429-XXXXXX/</code>).</p>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-81422e2 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-81422e2-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-qzh5lwt"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-qzh5lwt" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-9xmeu2a"><p class="stk-block-text__text"><strong>Important<br /></strong>Always discard the results of the first run. The JVM needs time to warm up and JIT-compile the execution paths. Change the JSON to <code class="">&quot;runs&quot;: 3</code> or <code class="">&quot;runs&quot;: 5</code></p></div>
</div></div></blockquote>



<p></p>



<h4 class="wp-block-heading">6. Real-Time Observability with Grafana</h4>



<p>Because&nbsp;<code class="">pbench</code>&nbsp;automatically streams our execution metrics into MySQL, we can use&nbsp;<strong>Grafana</strong> to visualize our cluster&#8217;s performance in real-time. Our Docker Compose stack automatically provisions a Grafana instance alongside our MySQL metrics database.</p>



<ul class="wp-block-list">
<li>Navigate to&nbsp;<code class="">http://localhost:3000</code>&nbsp;(Default login:&nbsp;<code class="">admin</code>&nbsp;/&nbsp;<code class="">admin</code>).</li>



<li>Go to&nbsp;<strong>Connections &gt; Data Sources</strong>&nbsp;and add a new&nbsp;<strong>MySQL</strong>&nbsp;datasource.</li>



<li>Configure the connection using the internal Docker network:
<ul class="wp-block-list">
<li><strong>Host</strong>:&nbsp;<code class="">mysql:3306</code></li>



<li><strong>Database</strong>:&nbsp;<code class="">pbench</code></li>



<li><strong>User</strong>:&nbsp;<code class="">pbench</code></li>



<li><strong>Password</strong>:&nbsp;<code class="">pbench_password</code></li>
</ul>
</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2759" height="532" src="https://prestodb.io/wp-content/uploads/image-116-1024x532.png" style="width: 1009px; height: auto;" width="1024" /></figure>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2760" height="529" src="https://prestodb.io/wp-content/uploads/image-117-1024x529.png" style="width: 1010px; height: auto;" width="1024" /></figure>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-e0d7859 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-e0d7859-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-vjcezik"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-vjcezik" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-1n4fphc"><p class="stk-block-text__text"><strong>Success<br /></strong>Benchmarking stats have been recorded successfully</p></div>
</div></div></blockquote>



<p></p>



<h4 class="wp-block-heading">7. Troubleshooting</h4>



<p>When running complex analytical queries at scale, you are bound to hit the physical limits of your hardware or data anomalies. Here is how you can solve the biggest challenges in this architecture:</p>



<p>TPC-DS Query 72 involves massive multi-way joins that can easily exceed physical RAM.</p>



<ul class="wp-block-list">
<li><strong>The Fix</strong>: We enabled disk spilling. By mounting a&nbsp;<code class="">/tmp/presto-spill</code>&nbsp;volume to our Docker containers and adding&nbsp;<code class="">experimental.spill-enabled=true</code>&nbsp;to our&nbsp;<code class="">config.properties</code>, Presto safely offloads intermediate join data to disk rather than crashing.</li>
</ul>



<p>Legacy TPC-DS data generators often produce sparse data that results in a&nbsp;<code class="">DIVISION_BY_ZERO</code> SQL error on Query 90.</p>



<ul class="wp-block-list">
<li><strong>The Fix</strong>: Instead of failing the benchmark, we wrapped the mathematical operations in the SQL file with a safe division wrapper:&nbsp;<code class="">COALESCE(NULLIF(value, 0), 1)</code>.</li>
</ul>



<h4 class="wp-block-heading">8. Resources</h4>



<ul class="wp-block-list">
<li>GitHub Repo <a href="https://github.com/saurabhmahawar/presto_databenchmarking" id="https://github.com/saurabhmahawar/presto_databenchmarking" type="link"><strong>Link</strong></a></li>



<li>Check out Presto Documentation on <a href="https://prestodb.io/docs/current/connector/iceberg.html" rel="noreferrer noopener" target="_blank"><strong>Iceberg Connector</strong></a> and <a href="https://prestodb.io/docs/current/connector/hive.html" rel="noreferrer noopener" target="_blank"><strong>Hive Connector</strong></a> for more information. Join Presto Slack <a href="https://communityinviter.com/apps/prestodb/prestodb" id="https://communityinviter.com/apps/prestodb/prestodb" type="link"><strong>Community</strong></a></li>
</ul>



<h2 class="wp-block-heading">Follow Us</h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/05/03/presto-benchmarking-tutorial-tpc-h-tpc-ds-on-iceberg-with-google-cloud-storage-gcs/">Presto Benchmarking Tutorial &#8211; TPC-H &amp; TPC-DS on Iceberg with Google Cloud Storage (GCS)</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
