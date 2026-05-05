---
title: "Iceberg Branches and Tags with Presto"
url: "https://prestodb.io/blog/2026/03/25/iceberg-branches-and-tags-with-presto/"
date: "Wed, 25 Mar 2026 16:22:59 +0000"
author: "PrestoDB"
feed_url: "https://prestodb.io/feed/"
---
<p>Modern data lakehouses increasingly require versioned data access, auditability, and safe experimentation without affecting production systems. Apache Iceberg allows you to maintain multiple concurrent timelines of a table through&nbsp;<strong>Branches</strong>&nbsp;and capture static historical points using&nbsp;<strong>Tags</strong>. This mechanism is heavily inspired by Git but operates on underlying table snapshots.</p>



<figure class="wp-block-image size-large is-resized"><img alt="" class="wp-image-2743" height="538" src="https://prestodb.io/wp-content/uploads/image-115-1024x538.png" style="width: 666px; height: auto;" width="1024" /></figure>



<p id="c9b4">In this blog, we are going to see the support provided by PrestoDB for <strong>Apache Iceberg</strong> branches and tags, which includes:</p>



<ul class="wp-block-list">
<li>Creating and dropping branches and tags</li>



<li>Querying branches and tags</li>



<li>Mutations on branches</li>



<li>Running workloads without impacting the production data</li>



<li>Support for Java workers and Prestissimo (Presto C++)</li>
</ul>



<p id="25ab">All the mentioned Iceberg branch and tag functionalities are available in&nbsp;<strong>PrestoDB version ≥ 0.297</strong></p>



<h2 class="wp-block-heading" id="13fd"><strong>What are Iceberg Branches and Tags?</strong></h2>



<p id="1ad9"><strong>Branch:&nbsp;</strong>A branch is a&nbsp;<strong><em>mutable reference</em></strong>&nbsp;to a snapshot. Write operations can be performed on it independently.</p>



<p id="4753"><strong>Use cases:</strong></p>



<ul class="wp-block-list">
<li>Data validation pipelines</li>



<li>Audit workflows</li>



<li>Experimentation</li>



<li>CI/CD for data</li>
</ul>



<p id="d7bf"><strong>Tag:</strong>&nbsp;A tag is an immutable reference to a snapshot.</p>



<p><strong>Use case:&nbsp;</strong>Compliance snapshots</p>



<h2 class="wp-block-heading"><strong>Creating Iceberg Branches Or Tags in PrestoDB</strong></h2>



<p>PrestoDB provides SQL syntax to create tags and branches directly.</p>



<h3 class="wp-block-heading">Create a Branch Or a Tag</h3>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; ALTER TABLE iceberg.default.mytable CREATE TAG &#039;audit-tag&#039;;</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; ALTER TABLE iceberg.default.mytable CREATE BRANCH &#039;audit-branch&#039;;</code></pre></div>



<h3 class="wp-block-heading" id="fe62">Create a Branch Or a Tag for a Specific Snapshot</h3>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">presto&gt; ALTER TABLE iceberg.default.mytable<br />CREATE TAG 'audit-tag'<br />FOR SYSTEM_VERSION AS OF 3;</pre>



<pre class="wp-block-code"><code class="">presto&gt; ALTER TABLE iceberg.default.mytable
CREATE BRANCH &#039;audit-branch&#039;
FOR SYSTEM_VERSION AS OF 3;</code></pre></div>



<h3 class="wp-block-heading">Create a Branch Or a Tag Using Timestamp</h3>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; ALTER TABLE iceberg.default.mytable
CREATE TAG &#039;audit-tag&#039;
FOR SYSTEM_TIME AS OF TIMESTAMP
&#039;2024-03-02 13:29:46.822 America/Los_Angeles&#039;;</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; ALTER TABLE iceberg.default.mytable
CREATE BRANCH &#039;audit-branch&#039;
FOR SYSTEM_TIME AS OF TIMESTAMP
&#039;2024-03-02 13:29:46.822 America/Los_Angeles&#039;;</code></pre></div>



<h3 class="wp-block-heading">Create a Branch Or a Tag with a Retention Policy</h3>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; ALTER TABLE iceberg.default.mytable
CREATE TAG &#039;audit-tag&#039;
FOR SYSTEM_VERSION AS OF 3
RETAIN 7 DAYS;</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; ALTER TABLE iceberg.default.mytable
CREATE BRANCH &#039;audit-branch&#039;
FOR SYSTEM_VERSION AS OF 3
RETAIN 7 DAYS;</code></pre></div>



<p><em>Retention allows automatic cleanup of old metadata references.</em></p>



<h2 class="wp-block-heading"><strong>Querying Iceberg Branches and Tags in PrestoDB</strong></h2>



<h3 class="wp-block-heading">Querying a Branch</h3>



<p>PrestoDB supports&nbsp;<strong>two syntaxes</strong>&nbsp;for querying branches.</p>



<ul class="wp-block-list">
<li>Querying the branch using&nbsp;<strong>FOR SYSTEM_VERSION AS OF&nbsp;</strong>syntax:</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; SELECT * FROM table_name FOR SYSTEM_VERSION AS OF &#039;branch_name&#039;;</code></pre></div>



<ul class="wp-block-list">
<li>Querying the branch using&nbsp;<strong>dot notation</strong>&nbsp;syntax:</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; SELECT * FROM &quot;table_name.branch_branchName&quot;;</code></pre></div>



<p><em>Here,</em><code class=""> branch_</code><em> is the keyword required along with the branchName we are trying to query</em></p>



<h2 class="wp-block-heading"><strong>Querying a Tag</strong></h2>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; SELECT * FROM table_name FOR SYSTEM_VERSION AS OF &#039;tag_name&#039;;</code></pre></div>



<h2 class="wp-block-heading" id="68e8"><strong>Dropping Branches and Tags</strong></h2>



<p id="8a6e">Dropping references is straightforward. Dropping a branch or tag&nbsp;<strong>does not delete underlying data</strong>. It simply removes the reference to the snapshot.</p>



<pre class="wp-block-preformatted">presto&gt; ALTER TABLE users DROP BRANCH 'branch1';</pre>



<pre class="wp-block-preformatted">presto&gt; ALTER TABLE users DROP TAG 'tag1';</pre>



<h2 class="wp-block-heading" id="adac"><strong>Mutating Iceberg Branches from PrestoDB</strong></h2>



<p id="3eb0">One of the most powerful capabilities is&nbsp;<strong>isolated mutations on a branch</strong>. All the operations below modify&nbsp;<strong>only the branch snapshot lineage</strong>, leaving the&nbsp;<strong>main branch unaffected</strong>.</p>



<p id="c4f5">This enables powerful workflows such as:</p>



<ul class="wp-block-list">
<li>Staging transformations</li>



<li>Data quality validation</li>



<li>Safe backfills</li>



<li>Pipeline experimentation</li>
</ul>



<p id="dba7"><strong>Example:</strong></p>



<pre class="wp-block-preformatted">presto&gt; ALTER TABLE orders CREATE BRANCH 'audit_branch';</pre>



<p id="953f"><em>All mutations can now be directed to that branch.</em></p>



<h3 class="wp-block-heading" id="dfc5">Insert into a Branch</h3>



<pre class="wp-block-preformatted">presto&gt; INSERT INTO "orders.branch_audit_branch"<br />VALUES (1, 'Product A', 100.00);</pre>



<h3 class="wp-block-heading" id="8586">Update Data in a Branch</h3>



<pre class="wp-block-preformatted">presto&gt; UPDATE "orders.branch_audit_branch" SET price = 120.00 WHERE id = 1;</pre>



<h3 class="wp-block-heading" id="8381">Delete Rows from a Branch</h3>



<pre class="wp-block-preformatted">presto&gt; DELETE FROM "orders.branch_audit_branch" WHERE id = 2;</pre>



<h3 class="wp-block-heading" id="c102">Merge into a Branch</h3>



<pre class="wp-block-preformatted">presto&gt; MERGE INTO "orders.branch_audit_branch" t<br />USING source_table s<br />ON t.id = s.id<br />WHEN MATCHED THEN UPDATE SET price = s.price<br />WHEN NOT MATCHED THEN INSERT (id, product, price)<br />VALUES (s.id, s.product, s.price);</pre>



<h3 class="wp-block-heading" id="c957">Truncate a Branch</h3>



<pre class="wp-block-preformatted">presto&gt; TRUNCATE TABLE "orders.branch_audit_branch";</pre>



<h2 class="wp-block-heading" id="34d4"><strong>Support in Prestissimo (Presto C++)</strong></h2>



<p id="0c13">If you are running PrestoDB with&nbsp;<strong>Prestissimo (C++ workers)</strong>, most of the above branch and tag functionality works seamlessly. In Prestissimo clusters,&nbsp;<strong>branch mutations are currently limited to</strong>:<code class="">INSERT TRUNCATE</code></p>



<p id="1e57"><strong><em>Support for additional mutation operations may be added in future releases.</em></strong></p>



<h2 class="wp-block-heading"><strong>Final Thoughts</strong></h2>



<p>With the help of iceberg branches and tags, version control can be implemented in data lakes using&nbsp;<strong>Git version control systems</strong>. PrestoDB allows access to these using simple SQL constructs. With the help of such features, PrestoDB allows&nbsp;<strong>safe experimentation, auditing, and reproducibility</strong>&nbsp;in modern data systems.</p>



<p>If you’re building lakehouse architectures with Presto and Iceberg,&nbsp;<strong>branches and tags should become a fundamental part of your data workflow design.</strong></p>



<h2 class="wp-block-heading">Follow Us </h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/03/25/iceberg-branches-and-tags-with-presto/">Iceberg Branches and Tags with Presto</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
