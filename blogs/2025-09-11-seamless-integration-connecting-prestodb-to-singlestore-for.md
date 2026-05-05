---
title: "Seamless Integration: Connecting PrestoDB to SingleStore for High-Performance Analytics"
url: "https://prestodb.io/blog/2025/09/11/seamless-integration-connecting-prestodb-to-singlestore-for-high-performance-analytics/"
date: "Thu, 11 Sep 2025 10:48:26 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<p>In today&#8217;s data-driven landscape, organization&#8217;s are constantly seeking ways to analyze massive datasets quickly and efficiently. <strong>PrestoDB, a powerful open-source SQL query engine</strong>, and <strong>SingleStore, a distributed SQL database</strong>, are two technologies that, when combined, offer unparalleled capabilities for high-performance data querying and distributed analytics. This guide provides a hands-on, step-by-step tutorial on how to connect PrestoDB to a SingleStore database, enabling you to leverage the strengths of both platforms.</p>



<p>By integrating Presto to query SingleStore, you gain <strong>Presto&#8217;s federated query capabilities</strong> alongside <strong>SingleStore&#8217;s remarkable speed and efficiency</strong>. This setup is a game-changer for ad-hoc analysis and real-time reporting.</p>



<h2 class="wp-block-heading"><strong>Prerequisites <img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /></strong></h2>



<p>Before we dive into the connection setup, ensure you have the following ready:</p>



<ul class="wp-block-list">
<li><strong>PrestoDB Installed:</strong> You should have a running instance of PrestoDB. If you need a quick guide on how to install Presto server, check <a href="https://prestodb.io/blog/2025/07/15/setting-up-presto-a-step-by-step-installation-guide-to-run-sql-queries/"><strong>this</strong></a> out.</li>
</ul>



<ul class="wp-block-list">
<li><strong>Access to a SingleStore Instance:</strong> For this tutorial, we will be using the <strong>Standard Self-Managed (Free) Edition</strong> of SingleStore.</li>
</ul>



<h2 class="wp-block-heading"><strong>Step-by-Step Connection Setup</strong> <img alt="⚙" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2699.png" style="height: 1em;" /></h2>



<p>Let&#8217;s walk through the process of connecting PrestoDB to your SingleStore instance.</p>



<p><strong>1. Create Your SingleStore Workspace and Obtain Credentials.</strong></p>



<p>After creating your SingleStore account, follow these steps to set up your workspace and retrieve necessary connection details:</p>



<ul class="wp-block-list">
<li>Click on <strong>Ingest ➜ Pipelines</strong> to create a workspace. This will allow you to view your workspace details.</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2523" height="529" src="https://prestodb.io/wp-content/uploads/image-81-1024x529.png" style="width: 943px; height: auto;" width="1024" /></figure>



<ul class="wp-block-list">
<li>Next, click on <strong>Connect ➜ SQL IDE</strong>. Here, you will find the credentials required to connect PrestoDB to SingleStore.</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2524" height="512" src="https://prestodb.io/wp-content/uploads/image-82-1024x512.png" style="width: 937px; height: auto;" width="1024" /></figure>



<p><strong>Important:</strong> You will need to <strong>reset the password</strong>, as it is visible only once. Copy and save these credentials, including the <strong>host, port, username, password, and database name</strong>, for future use.</p>



<p><strong>2. Configure the Presto Catalog for SingleStore</strong></p>



<p>The core of connecting Presto to any data source lies in configuring a <strong>catalog</strong>. A Presto catalog informs the query engine which data source to connect to.</p>



<ul class="wp-block-list">
<li>Navigate to your Presto installation directory. Inside, you&#8217;ll find the <code class="">etc/catalog</code> folder</li>



<li>Within the <code class="">catalog</code> folder, create a new file named <code class="">singlestore.properties</code>. This file will contain the configuration for our SingleStore connection, leveraging the SingleStore connector available in Presto.</li>
</ul>



<p><strong>3. Define Connection Properties</strong></p>



<ul class="wp-block-list">
<li>Open the <code class="">singlestore.properties</code> file and add the following configuration:</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">connector.name=singlestore
connection-url=jdbc:singlestore://svc-3482219c-a389-4079-b18b-d50662524e8a-shared-dml.aws-virginia-6.svc.singlestore.com:3333/client_info?useSSL=true&amp;allowPublicKeyRetrieval=true&amp;trustServerCertificate=true
connection-user=speed-d2ce5
connection-password=&lt;Your_Password&gt;</code></pre></div>



<p><em>Key Properties Explained:</em></p>



<p>a. <strong>connector.name</strong>: This property must be set to<strong> singlestore</strong>. It instructs Presto to use its built-in SingleStore connector.</p>



<p>b. <strong>connection-url</strong>: This is the <strong>JDBC URL </strong>for your SingleStore cluster.</p>



<p>    ◦ Replace<strong> &lt;host> </strong>and<strong> &lt;port></strong> with your specific SingleStore aggregator host and port.</p>



<p>    ◦ The parameters <code class="">client_info?useSSL=true&amp;allowPublicKeyRetrieval=true&amp;trustServerCertificate=true</code> are crucial:</p>



<p><strong>Note:</strong> <strong>client_info</strong> represents the database name you created in SingleStore. For Free Tier accounts, providing a default database name is mandatory. <strong>useSSL=true, allowPublicKeyRetrieval=true, and trustServerCertificate=true</strong> are mandatory parameters to <strong>enable SSL and establish a secure connection</strong> with SingleStore.</p>



<p>c. <strong>connection-user</strong>: Your SingleStore username.</p>



<p>d. <strong>connection-password</strong>: Your SingleStore password. Remember to replace <strong>&lt;Your_SingleStore_Password></strong> with the password you saved earlier.</p>



<p><strong>4. Upload Sample Data (Optional but Recommended)</strong></p>



<p>To test your connection effectively, it&#8217;s helpful to have some data in SingleStore.</p>



<ul class="wp-block-list">
<li>You can manually upload data in SingleStore using the <strong>Load Data</strong> option. For example, a <code class="">customers.csv</code> file was used in the source material.</li>
</ul>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2525" height="518" src="https://prestodb.io/wp-content/uploads/image-83-1024x518.png" style="width: 936px; height: auto;" width="1024" /></figure>



<p><strong>5. Restart Presto Server and Verify Connection</strong></p>



<ul class="wp-block-list">
<li>After saving the <code class="">singlestore.properties</code> file, <strong>restart your Presto server</strong>.</li>
</ul>



<ul class="wp-block-list">
<li>Now, let&#8217;s verify that the <code class="">singlestore</code> catalog is recognized by Presto by executing the following query:</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">Show Catalogs;</code></pre></div>



<ul class="wp-block-list">
<li>You should see <code class="">singlestore</code> in the output, confirming that your catalog has been successfully recognized. If you can then see the list of your SingleStore databases, <strong>congratulations! You are now connected and ready to run queries directly through PrestoDB</strong>.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">Show schemas from singlestore;</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">select * from singlestore.client_info.customers order by customer_id limit 100;</code></pre></div>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2527" height="722" src="https://prestodb.io/wp-content/uploads/image-85-1024x722.png" style="width: 935px; height: auto;" width="1024" /></figure>



<h2 class="wp-block-heading"><strong>Troubleshooting Tips <img alt="🛠" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f6e0.png" style="height: 1em;" /></strong></h2>



<p>Encountering issues? Here are some common problems and their solutions:</p>



<ul class="wp-block-list">
<li><strong>Invalid Credentials:</strong> Double-check that your <code class="">connection-user</code> and <code class="">connection-password</code> in the <code class="">singlestore.properties</code> file are correct and case-sensitive.</li>
</ul>



<ul class="wp-block-list">
<li><strong>No SSL Detected:</strong> SingleStore requires SSL for connection to Presto. Ensure all required SSL parameters (<code class="">useSSL</code>, <code class="">allowPublicKeyRetrieval</code>, <code class="">trustServerCertificate</code>) are included in your <code class="">connection-url</code>. For production environments, it&#8217;s recommended to download the CA bundle and save it locally.</li>
</ul>



<ul class="wp-block-list">
<li><strong>Free-Tier Limitations:</strong> If you&#8217;re using the SingleStore Free Tier, remember that you&#8217;re typically limited to one database and a maximum of 10 tables. It&#8217;s also mandatory to pass a default database name in the <code class="">connection-url</code>.</li>
</ul>



<p></p>



<p class="has-large-font-size">Follow Presto at <a href="https://www.linkedin.com/company/presto-foundation/" rel="noreferrer noopener" target="_blank">Linkedin</a>, <a href="https://www.youtube.com/@PrestoFoundation" rel="noreferrer noopener" target="_blank">Youtube</a>, and Join <a href="https://communityinviter.com/apps/prestodb/prestodb" rel="noreferrer noopener" target="_blank">Slack</a> channel to interact with the community.</p>
<p>The post <a href="https://prestodb.io/blog/2025/09/11/seamless-integration-connecting-prestodb-to-singlestore-for-high-performance-analytics/">Seamless Integration: Connecting PrestoDB to SingleStore for High-Performance Analytics</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
