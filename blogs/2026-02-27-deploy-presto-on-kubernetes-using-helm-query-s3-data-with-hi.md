---
title: "Deploy Presto on Kubernetes using Helm: Query S3 Data with Hive Metastore"
url: "https://prestodb.io/blog/2026/02/27/deploy-presto-on-kubernetes-using-helm-query-s3-data-with-hive-metastore/"
date: "Fri, 27 Feb 2026 13:57:52 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<p>Deploying Presto on <strong>Kubernetes</strong> transforms this powerful engine into a cloud-native, resilient service that automatically handles failures, scales seamlessly, and optimizes resource utilization. When combined with&nbsp;<strong>Helm charts</strong>, the deployment becomes&nbsp;standardized, version-controlled, and easily reproducible across environments.</p>



<p>This comprehensive guide&nbsp;will walk you through deploying a production-capable baseline Presto cluster on Kubernetes using the official Presto Helm charts, covering everything from basic setup to advanced concepts like&nbsp;high availability, autoscaling, and monitoring integration.</p>



<h2 class="wp-block-heading"><strong>Prerequisites: Setting Up Your Environment</strong></h2>



<p>Before diving into deployment, ensure your environment meets these requirements:</p>



<h3 class="wp-block-heading">Required Software&nbsp;Versions</h3>



<ul class="wp-block-list">
<li><strong>Kubernetes Cluster</strong>: Version 1.30+ (EKS, GKE, AKS, local machine or on-premise)</li>



<li><strong>Helm 3</strong>: Latest stable version (3.14+ recommended)</li>



<li><strong>kubectl</strong>: Configured to communicate with target cluster</li>



<li><strong>Container Runtime</strong>: Docker Desktop / OrbStack or any container management tool</li>
</ul>



<h3 class="wp-block-heading">Infrastructure&nbsp;Requirements</h3>



<ul class="wp-block-list">
<li><strong>Minimum Cluster Resources</strong>: 4 CPU cores, 16GB RAM for basic cluster</li>



<li><strong>Production Cluster</strong>: 8+ CPU cores, 32GB+ RAM for workloads</li>



<li><strong>Network</strong>:&nbsp;Pod-to-Pod communication enabled</li>



<li><strong>Storage</strong>: Persistent storage available (optional&nbsp;but recommended)</li>
</ul>



<h3 class="wp-block-heading">Knowledge Prerequisites</h3>



<ul class="wp-block-list">
<li>Basic understanding of Kubernetes concepts (Pods, Services, ConfigMaps, Secrets)</li>



<li>Familiarity with Presto architecture (Coordinator, Worker, Catalogs)</li>



<li>Experience with YAML configuration&nbsp;and Helm basics</li>
</ul>



<h2 class="wp-block-heading"><strong>Presto Kubernetes Architecture: Components and Deployment Modes</strong></h2>



<p>Before deployment, let&#8217;s understand the key components:</p>



<h3 class="wp-block-heading">Core Architecture&nbsp;Components</h3>



<ul class="wp-block-list">
<li><strong>Coordinator Pod</strong>: The brain that parses SQL statements,&nbsp;plans queries, and manages worker nodes</li>



<li><strong>Worker Pods</strong>: The compute engines that execute tasks and process&nbsp;data</li>



<li><strong>Discovery Service</strong>: Headless service enabling worker-to-coordinator communication</li>



<li><strong>ConfigMaps</strong>: Store configuration files&nbsp;(config.properties, jvm.config, log.properties)</li>



<li><strong>Catalog ConfigMaps</strong>: Define data source connectors (Hive, MySQL, PostgreSQL,&nbsp;etc.)</li>



<li><strong>Secrets</strong>: Securely store credentials and sensitive configuration</li>



<li><strong>Services</strong>:&nbsp;Expose Presto endpoints internally and externally</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2739" height="575" src="https://prestodb.io/wp-content/uploads/Presto-Kubernetes-Architecture-1-1024x575.png" style="width: 866px; height: auto;" width="1024" /></figure>



<h3 class="wp-block-heading">Deployment Modes</h3>



<p>The Presto Helm chart supports three deployment&nbsp;modes:</p>



<ul class="wp-block-list">
<li><strong>Single Mode</strong>: One pod acting as both coordinator and worker (ideal&nbsp;for development)</li>



<li><strong>Cluster Mode</strong>: Separate coordinator and worker pods (standard production setup)</li>



<li><strong>HA Cluster Mode</strong>: Multiple active coordinator pods managed by a shared resource manager (high availability)</li>
</ul>



<h2 class="wp-block-heading"><strong>Step-by-Step Deployment Guide</strong></h2>



<h3 class="wp-block-heading">Step 1: Setting up the Presto Helm Chart on Kubernetes (Local)</h3>



<p>Create an organized workspace for your Presto deployments:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">mkdir -p ~/presto-k8s
cd ~/presto-k8s</code></pre></div>



<p>Create a dedicated&nbsp;namespace for Presto to isolate resources:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">#Verify Installation first
kubectl version --client
helm version

# Create a dedicated namespace
kubectl create namespace sql-query-engine

# Verify namespace
kubectl get namespaces</code></pre></div>



<p>Add the official <a href="https://github.com/prestodb/presto-helm-charts">Presto Helm repository</a>:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class=""># Add Presto Helm repository
helm repo add presto https://prestodb.github.io/presto-helm-charts

# Update repository to fetch latest charts
helm repo update

# Verify repository
helm repo list

#Pull the chart to inspect the default configurations
helm pull presto/presto --untar
tree presto</code></pre></div>



<p>Understanding the Helm Chart Structure:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto-helm-charts/
├── charts/
│   └── presto/
│       ├── Chart.yaml              # Chart metadata
│       ├── values.yaml             # Default configuration
│       ├── README.md               # Chart documentation
│       ├── .helmignore             # Files to ignore
│       └── templates/              # Kubernetes manifests
│           ├── configmap-catalog.yaml
│           ├── configmap-coordinator.yaml
│           ├── configmap-resource-manager.yaml
│           ├── configmap-worker.yaml
│           ├── deployment-coordinator.yaml
│           ├── deployment-resource-manager.yaml
│           ├── deployment-worker.yaml
│           ├── ingress.yaml
│           ├── NOTES.txt
│           ├── service-discovery.yaml
│           ├── service.yaml
│           └── serviceaccount.yaml
├── DEVELOPMENT.md
├── README.md
└── LICENSE</code></pre></div>



<p>Create a&nbsp;<code class="">my-presto-config.yaml</code>&nbsp;file in the root directory with minimal custom configurations for quick deployment:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class=""># my-presto-config.yaml

mode: cluster

coordinator:
  replicas: 1
  resources:
    requests:
      memory: &quot;2Gi&quot;
      cpu: &quot;1&quot;
    limits:
      memory: &quot;4Gi&quot;
      cpu: &quot;2&quot;
  jvm: |-
    -server
    -Xmx2G
    -XX:+ExitOnOutOfMemoryError
    -Djdk.attach.allowAttachSelf=true
    --add-opens=java.base/java.lang=ALL-UNNAMED
    --add-opens=java.base/java.io=ALL-UNNAMED
    --add-opens=java.base/java.util=ALL-UNNAMED
    --add-opens=java.base/java.net=ALL-UNNAMED

worker:
  replicas: 2
  resources:
    requests:
      memory: &quot;2Gi&quot;
      cpu: &quot;1&quot;
    limits:
      memory: &quot;4Gi&quot;
      cpu: &quot;2&quot;
  jvm: |-
    -server
    -Xmx2G
    -XX:+ExitOnOutOfMemoryError
    -Djdk.attach.allowAttachSelf=true
    --add-opens=java.base/java.lang=ALL-UNNAMED
    --add-opens=java.base/java.io=ALL-UNNAMED
    --add-opens=java.base/java.util=ALL-UNNAMED
    --add-opens=java.base/java.net=ALL-UNNAMED

# To connect data sources
catalog:
  tpch: |-
    connector.name=tpch
  hive: |-
    connector.name=hive-hadoop2
    hive.metastore.uri=thrift://hive-metastore:9083
    hive.s3.aws-access-key=&lt;YOUR_ACCESS_KEY&gt;
    hive.s3.aws-secret-key=&lt;YOUR_SECRET_KEY&gt;
    hive.s3.endpoint=https://&lt;YOUR_END_POINT&gt;
    hive.s3.path-style-access=true
    hive.s3.ssl.enabled=true</code></pre></div>



<p>Now, let&#8217;s deploy Presto to local machine with custom configuration. It will take about 2-3 minutes for the images to pull and the containers to start.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">helm install my-presto presto/presto -f my-presto-config.yaml --namespace sql-query-engine</code></pre></div>



<p>To apply any future configuration changes, you simply run the&nbsp;<strong>Helm upgrade</strong>&nbsp;command:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">helm upgrade my-presto presto/presto -f my-presto-config.yaml -n sql-query-engine</code></pre></div>



<p>Check the status of the pods.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">kubectl get pods --namespace sql-query-engine</code></pre></div>



<p>Expected Output:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; READY &nbsp; STATUS&nbsp; &nbsp; RESTARTS &nbsp; AGE

my-presto-coordinator-886bc4b5f-wsj6t &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 59m

my-presto-worker-7b5698569b-bg664 &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 59m

my-presto-worker-7b5698569b-wdfcn &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 59m</code></pre></div>



<p>The above status confirms that Presto coordinator and workers are running in the background and presto server has started successfully.</p>



<p>Since the cluster is running inside Kubernetes (an isolated network), you need to create a tunnel to access the Web UI from our browser.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">kubectl port-forward svc/my-presto 8080:8080 -n sql-query-engine</code></pre></div>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>Leave this terminal window running. Do not close it</p>
</blockquote>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2732" height="489" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260216170811-1024x489.png" style="width: 861px; height: auto;" width="1024" /></figure>



<p>Run the below command to execute queries using <strong>presto-cli</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">kubectl exec -it my-presto-coordinator-67565444dd-rvrt7 --namespace sql-query-engine -- presto-cli</code></pre></div>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2733" height="438" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260216173229-1024x438.png" width="1024" /></figure>



<h3 class="wp-block-heading">Step 2 : Scaling Your Deployment (Adding More Workers)</h3>



<p>To add more workers (scale out), edit <code class="">replicas</code> in <strong>my-presto-config.yaml</strong> file and change it to any number under <code class="">worker</code> section</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">worker:
  replicas: 5  # &lt;-- Update this
  # ... rest of config stays the same</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">#Apply the Update

helm upgrade my-presto presto/presto -f my-presto-config.yaml -n sql-query-engine

#Check the pod status

kubectl get pods -n sql-query-engine

NAME &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY    STATUS&nbsp;  RESTARTS &nbsp;  AGE

my-presto-coordinator-67565444dd-rvrt7 &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 91m

my-presto-worker-56f9fd8b84-4rm6v&nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 103s

my-presto-worker-56f9fd8b84-7l2lr&nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 91m

my-presto-worker-56f9fd8b84-89zd7&nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 91m

my-presto-worker-56f9fd8b84-8srz2&nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 103s

my-presto-worker-56f9fd8b84-ts4x5&nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 103s</code></pre></div>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2734" height="482" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260216175807-1024x482.png" style="width: 862px; height: auto;" width="1024" /></figure>



<h3 class="wp-block-heading">Step 3 : Configuring the Apache Hive Metastore for S3</h3>



<p>You can use any modern cloud storage provider (like AWS S3, Wasabi, or MinIO). This layer serves as the massive, scalable foundation where your actual raw data files (CSV, JSON, Parquet) physically reside.</p>



<p>Create an S3 bucket in your preferred region, and set up a dedicated directory inside it to serve as the storage location for your data files.</p>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2735" height="392" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260224165121-1024x392.png" style="width: 868px; height: auto;" width="1024" /></figure>



<p>Set up&nbsp;<code class="">hive-metastore.yaml</code> and initialize a PostgreSQL database to serve as the centralized metadata catalog for the Presto cluster.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class=""># 1. POSTGRES DEPLOYMENT (Holds the Data)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hive-metastore-db
  namespace: sql-query-engine
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hive-metastore-db
  template:
    metadata:
      labels:
        app: hive-metastore-db
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: metastore
        - name: POSTGRES_USER
          value: hive
        - name: POSTGRES_PASSWORD
          value: hivepassword
        ports:
        - containerPort: 5432
---
# 2. POSTGRES SERVICE
apiVersion: v1
kind: Service
metadata:
  name: hive-metastore-db
  namespace: sql-query-engine
spec:
  ports:
  - port: 5432
  selector:
    app: hive-metastore-db
---
# 3. HIVE CONFIGURATION (With S3 Keys)
apiVersion: v1
kind: ConfigMap
metadata:
  name: hive-config
  namespace: sql-query-engine
data:
  hive-site.xml: |
    &lt;configuration&gt;
      &lt;property&gt;
        &lt;name&gt;javax.jdo.option.ConnectionURL&lt;/name&gt;
        &lt;value&gt;jdbc:postgresql://hive-metastore-db:5432/metastore&lt;/value&gt;
      &lt;/property&gt;
      &lt;property&gt;
        &lt;name&gt;javax.jdo.option.ConnectionUserName&lt;/name&gt;
        &lt;value&gt;hive&lt;/value&gt;
      &lt;/property&gt;
      &lt;property&gt;
        &lt;name&gt;javax.jdo.option.ConnectionPassword&lt;/name&gt;
        &lt;value&gt;hivepassword&lt;/value&gt;
      &lt;/property&gt;
      &lt;property&gt;
        &lt;name&gt;fs.s3a.access.key&lt;/name&gt;
        &lt;value&gt;YOUR_ACCESS_KEY&lt;/value&gt;
      &lt;/property&gt;
      &lt;property&gt;
        &lt;name&gt;fs.s3a.secret.key&lt;/name&gt;
        &lt;value&gt;YOUR_SECRET_KEY&lt;/value&gt;
      &lt;/property&gt;
      &lt;property&gt;
        &lt;name&gt;fs.s3a.endpoint&lt;/name&gt;
        &lt;value&gt;YOUR_ENDPOINT&lt;/value&gt;
      &lt;/property&gt;
      &lt;property&gt;
        &lt;name&gt;fs.s3a.path.style.access&lt;/name&gt;
        &lt;value&gt;true&lt;/value&gt;
      &lt;/property&gt;
    &lt;/configuration&gt;
---
# 4. HIVE METASTORE DEPLOYMENT
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hive-metastore
  namespace: sql-query-engine
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hive-metastore
  template:
    metadata:
      labels:
        app: hive-metastore
    spec:
      # Shared Volume for Driver Jar
      volumes:
      - name: hive-config
        configMap:
          name: hive-config
      - name: lib-share
        emptyDir: {}

      initContainers:
      # A. Download Postgres Driver + AWS/S3 JARs
      - name: download-driver
        image: python:3.9-slim
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
        command: &#091;&quot;python3&quot;, &quot;-c&quot;]
        args:
          - |
            import urllib.request
            files = &#091;
              (&#039;https://jdbc.postgresql.org/download/postgresql-42.2.18.jar&#039;, &#039;/lib-share/postgresql-42.2.18.jar&#039;),
              (&#039;https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.3.4/hadoop-aws-3.3.4.jar&#039;, &#039;/lib-share/hadoop-aws-3.3.4.jar&#039;),
              (&#039;https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.12.367/aws-java-sdk-bundle-1.12.367.jar&#039;, &#039;/lib-share/aws-java-sdk-bundle-1.12.367.jar&#039;),
            ]
            for url, dest in files:
              print(f&#039;Downloading {url}...&#039;)
              urllib.request.urlretrieve(url, dest)
              print(&#039;Done.&#039;)
        volumeMounts:
        - name: lib-share
          mountPath: /lib-share

      # B. Initialize Schema (Run as ROOT to allow copying to /opt/hive/lib)
      - name: init-schema
        image: apache/hive:4.0.0
        securityContext:
          runAsUser: 0
        command: &#091;&quot;sh&quot;, &quot;-c&quot;]
        args:
          - &quot;cp /lib-share/*.jar /opt/hive/lib/ &amp;&amp; (/opt/hive/bin/schematool -dbType postgres -info || /opt/hive/bin/schematool -dbType postgres -initSchema)&quot;
        volumeMounts:
        - name: hive-config
          mountPath: /opt/hive/conf/hive-site.xml
          subPath: hive-site.xml
        - name: lib-share
          mountPath: /lib-share

      containers:
      # C. Run Metastore Service (Run as ROOT to allow copying to /opt/hive/lib)
      - name: metastore
        image: apache/hive:4.0.0
        securityContext:
          runAsUser: 0
        command: &#091;&quot;sh&quot;, &quot;-c&quot;]
        args:
          - &quot;cp /lib-share/*.jar /opt/hive/lib/ &amp;&amp; /opt/hive/bin/hive --service metastore&quot;
        ports:
        - containerPort: 9083
        volumeMounts:
        - name: hive-config
          mountPath: /opt/hive/conf/hive-site.xml
          subPath: hive-site.xml
        - name: lib-share
          mountPath: /lib-share
---
# 5. HIVE METASTORE SERVICE
apiVersion: v1
kind: Service
metadata:
  name: hive-metastore
  namespace: sql-query-engine
spec:
  ports:
  - port: 9083
    targetPort: 9083
  selector:
    app: hive-metastore</code></pre></div>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>In production, always use Kubernetes Secrets or cloud-native identity mechanisms (IRSA) to store the keys.</p>
</blockquote>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class=""># To apply above configuration, run the below command:

kubectl apply -f hive-metastore.yaml

# Check status of the pods

kubectl get pods -n sql-query-engine</code></pre></div>



<p>Expected Output:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">NAME &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; READY &nbsp; STATUS&nbsp; &nbsp; RESTARTS &nbsp; AGE

hive-metastore-5b964db94-dg2wm &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 102m

hive-metastore-db-658f9d4546-89n8l &nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 151m

my-presto-coordinator-68b85f9566-rt5jw &nbsp;   1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 138m

my-presto-worker-66bb49fb6b-bs6f7&nbsp; &nbsp; &nbsp; &nbsp;   1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 138m

my-presto-worker-66bb49fb6b-gvtwd&nbsp; &nbsp; &nbsp; &nbsp;   1/1 &nbsp; &nbsp; Running &nbsp; 0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 138m</code></pre></div>



<p>Let&#8217;s confirm that <strong>hive</strong> registered as a catalog with Presto.</p>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2736" height="554" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260224193219-1024x554.png" style="width: 861px; height: auto;" width="1024" /></figure>



<p>Let&#8217;s create the schema, if it doesn&#8217;t already exist.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">CREATE SCHEMA IF NOT EXISTS hive.default;</code></pre></div>



<p>It&#8217;s time to connect Presto to actual S3 data, We will use&nbsp;<strong>External Tables</strong>. When we define an External Table, we are simply telling the Hive Metastore to draw a <strong>map</strong> or an <strong>index card</strong>.</p>



<p>Run the following SQL command through Presto-UI or CLI:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">CREATE TABLE IF NOT EXISTS hive.default.subscriptions (
  user_id VARCHAR, 
  phone_number VARCHAR, 
  subscription_type VARCHAR,
  region VARCHAR, 
  subscription_date VARCHAR, 
  source VARCHAR
) WITH (
  format = &#039;CSV&#039;,
  external_location = &#039;s3a://presto-cluster/my_data/&#039;
);</code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-5888539 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-5888539-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-snuoiod"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-snuoiod" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-i2tu1nq"><p class="stk-block-text__text">This deployment uses an ephemeral, stateless PostgreSQL database. If the database pod restarts, you will simply need to re-run your&nbsp;<code class="">CREATE TABLE</code>&nbsp;scripts to map your S3 data back into Presto. For permanent metadata retention, attach a <strong>PersistentVolumeClaim (PVC)</strong> to the Postgres container.</p></div>
</div></div></blockquote>



<h3 class="wp-block-heading">Step 4: Querying S3 Data</h3>



<p>Run the command to query the data:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">SELECT subscription_type, COUNT(*) as total_users
FROM hive.default.subscriptions
GROUP BY subscription_type
ORDER BY total_users DESC limit 100</code></pre></div>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2737" height="403" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260225212501-1024x403.png" style="width: 860px; height: auto;" width="1024" /></figure>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>Presto executed the query successfully and delivered the aggregated result.</p>
</blockquote>



<h2 class="wp-block-heading"><strong>Troubleshooting Common Issues</strong></h2>



<h4 class="wp-block-heading">Memory Thrashing (OOMKilled), CrashLoopBackOff</h4>



<ul class="wp-block-list">
<li><strong>Reason</strong>: Presto Coordinator is requesting more RAM than your Kubernetes node has available, or the Java Heap size (<code class="">-Xmx</code>) is set incorrectly.</li>



<li><strong>Fix</strong>: Check your&nbsp;<code class="">resources.limits.memory</code>. If you set it to&nbsp;<code class="">2Gi</code>, you cannot set your Java Heap (<code class="">-Xmx</code>) to&nbsp;<code class="">2G</code>. The JVM needs overhead room.</li>
</ul>



<h4 class="wp-block-heading">Error:&nbsp;Connector hive not found</h4>



<ul class="wp-block-list">
<li><strong>Reason:</strong>&nbsp;This issue almost always stems from mixing up&nbsp;<strong>PrestoDB</strong>&nbsp;configuration syntax with&nbsp;<strong>Trino</strong>.</li>



<li><strong>Fix</strong>: Change the connector name to exactly:&nbsp;<code class="">connector.name=hive-hadoop2</code></li>
</ul>



<h2 class="wp-block-heading"><strong>Summary</strong></h2>



<p>By deploying&nbsp;<strong>Presto on Kubernetes</strong>&nbsp;alongside the&nbsp;<strong>Apache Hive Metastore</strong>, you can seamlessly query terabytes of external data stored securely in S3-compatible storage. By embracing a stateless metadata catalog mapped to external tables, you now have a highly resilient, cost-effective, and containerized analytics engine ready for production.</p>



<p>Refer to Presto Documentation on <a href="https://github.com/prestodb/presto-helm-charts/"><strong>Presto Helm Deployment</strong></a> and <a href="https://prestodb.io/docs/current/connector/hive.html"><strong>Hive Connector</strong></a> for more information.</p>



<h2 class="wp-block-heading">Follow Us </h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/02/27/deploy-presto-on-kubernetes-using-helm-query-s3-data-with-hive-metastore/">Deploy Presto on Kubernetes using Helm: Query S3 Data with Hive Metastore</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
