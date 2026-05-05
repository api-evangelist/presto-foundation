---
title: "Presto vs Prestissimo – Known differences and workarounds"
url: "https://prestodb.io/blog/2026/01/22/presto-vs-prestissimo-known-differences-and-workarounds/"
date: "Thu, 22 Jan 2026 23:55:02 +0000"
author: "Ali LeClerc"
feed_url: "https://prestodb.io/feed/"
---
<h2 class="wp-block-heading">TL;DR</h2>



<p>This blog outlines the known differences between Presto and Prestissimo where existing Presto queries require adjustment to work in Prestissimo.&nbsp;</p>



<h2 class="wp-block-heading">Details</h2>



<p>Prestissimo is generally available to use and has feature parity (<a href="https://github.com/facebookincubator/velox/issues/16041">except for a few functions</a>) with Presto Java. There are differences in libraries used in both stacks. Also we have ensured that bugs aren&#8217;t ported from the Java stack and have been fixed in the Java stack when applicable. As a result, in rare cases queries that run in Presto Java can fail or provide different results in Prestissimo. These queries should be changed so that they continue to work. Below we publish the known differences between two engines.&nbsp;</p>



<h3 class="wp-block-heading">Array Functions</h3>



<h3 class="wp-block-heading">Array sort with lambda comparator:</h3>



<p><a href="https://velox-lib.io/blog/array-sort/">Array sort with transform lambda works significantly better than comparator lambda. Writing a proper comparator is tricky and error prone.</a></p>



<p>1. For lambda usage, ‘Case’ is not supported. Use ‘If’ Instead. The following is not supported:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">(x, y) -&gt;
CASE
WHEN x.event_time &lt; y.event_time THEN
  -1
WHEN x.event_time &gt; y.event_time THEN
  1
  ELSE 0
END</code></pre></div>



<p>To work with Prestissimo, rewrite using transform lambda. For example: <em>(x) -&gt; x.event_time</em><br />Alternately it can be written with if, like below:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">(x, y) -&gt; IF (x.event_time &lt; y.event_time, -1, IF (x.event_time &gt; y.event_time, 1, 0))</code></pre></div>



<p>When If is used to rewrite, follow these rules for using a lambda in array sort:</p>



<ul class="wp-block-list">
<li>The lambda should use if else (Case is not supported)</li>



<li>The lambda should return 1, 0, -1 (i.e cover all cases)</li>



<li>The lambda should use the same expression when doing the comparison, for example in the above case <em>event_time</em> is used for comparison throughout the lambda. If we rewrote the expression as following, where x and y have different fields, it will fail:<em> (x, y) -&gt; if (x.event_time &lt; y.event_start_time, -1, if (x.event_time &gt; y.event_start_time, 1, 0))</em></li>



<li>Note that any additional nesting apart from the two if’s shown above will fail</li>
</ul>



<p>The best option is to use transform lambda whenever possible.</p>



<p>2. Array_sort can support any transformation lambda that returns a comparable type. For example the lambda below is not supported:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">&quot;array_sort&quot;(&quot;map_values&quot;(m), (a, b) -&gt; (
                CASE WHEN (a&#091;1] &#091;2] &gt; b&#091;1] &#091;2]) THEN 1 
                     WHEN (a&#091;1] &#091;2] &lt; b&#091;1] &#091;2]) THEN -1 
                     WHEN (a&#091;1] &#091;2] = b&#091;1] &#091;2]) THEN 
                          IF((a&#091;3] &gt; b&#091;3]), 1, -1) END)</code></pre></div>



<p>However this can be rewritten to the below format , which is supported:</p>



<p><em>&#8220;array_sort&#8221;(&#8220;map_values&#8221;(m), (a) -&gt; ROW(a[1][2], a[3]))</em></p>



<h3 class="wp-block-heading">JSON Functions</h3>



<ul class="wp-block-list">
<li>json_extract:
<ul class="wp-block-list">
<li><strong>Use of functions in JSON path</strong>
<ul class="wp-block-list">
<li><strong>Issue:</strong> Using functions inside a JSON path is not supported</li>



<li><strong>Workaround:</strong> Rewrite paths to use equivalent and often faster UDFs (User-Defined Functions) outside the JSON path, improving job portability and efficiency.</li>



<li><strong>Example:</strong></li>



<li>Original:</li>
</ul>
</li>
</ul>
</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">CAST(JSON_EXTRACT(config, &#039;$.table_name_to_properties.keys()&#039;
  ) AS ARRAY(ARRAY(VARCHAR)))</code></pre></div>



<p>Rewritten:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">map_keys(JSON_EXTRACT( config, &#039;$.table_name_to_properties&#039;) )</code></pre></div>



<ul class="wp-block-list">
<li>This rewritten approach works in both Presto and Prestissimo. Aggregates might occasionally be necessary. Generally, functions should be extracted from the JSON path for better portability.</li>



<li>Use of expressions in JSON path
<ul class="wp-block-list">
<li><strong>Issue: </strong>Using paths that have filter expressions aren&#8217;t supported&nbsp;</li>



<li><strong>Workaround: </strong>Do the filtering as a part of the SQL expression, query rather than in the JSON path &#8211; This will be more efficient and faster.</li>



<li><strong>Example:</strong>
<ul class="wp-block-list">
<li><strong>Original:</strong></li>
</ul>
</li>
</ul>
</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">JSON_EXTRACT(config, &#039;$.store.book&#091;?(@.price &gt; 10)]&#039;)</code></pre></div>



<ul class="wp-block-list">
<li><strong>Rewritten</strong></li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">filter(
&nbsp;&nbsp;&nbsp;CAST(json_extract(data, &#039;$.store.book&#039;) AS ARRAY&lt;JSON&gt;),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x -&gt; CAST(json_extract_scalar(x.value, &#039;$.price&#039;) AS &nbsp; DOUBLE) &gt; 10)
&nbsp;&nbsp;&nbsp;)</code></pre></div>



<ul class="wp-block-list">
<li><strong>Erroring on Invalid JSON</strong>
<ul class="wp-block-list">
<li><strong>Issue: </strong><a href="https://github.com/prestodb/presto/issues/24090">Presto Java can successfully run json_extract on certain invalid json</a> but Prestissimo will always fail</li>



<li><strong>Workaround: </strong>Extracting data from invalid JSON is indeterminate, and thus relying on that behavior can have unintended consequences. Prestissimo takes the safe approach to always throw on invalid JSON. Wrap your calls in a try to ensure the query succeeds and validate that the results correspond to your expectations.&nbsp;</li>
</ul>
</li>



<li><strong>Canonicalization</strong>
<ul class="wp-block-list">
<li><strong>Issue: </strong><a href="https://github.com/prestodb/presto/issues/24563#issue-2852506643">Presto Java json_extract can return json that is not canonicalized</a></li>



<li><strong>Workaround: </strong>json_extract has been rewritten to always return canonical JSON from json_extract in Prestissimo. This is required for correct and valid results.</li>
</ul>
</li>
</ul>



<h3 class="wp-block-heading">Regex Functions</h3>



<p>Prestissimo uses <a href="https://github.com/google/re2">RE2</a>, a widely adopted modern regular expression parsing library. RE2 provides most of the functionality of PCRE using a C++ interface very close to PCRE&#8217;s, but it guarantees linear <a href="https://swtch.com/~rsc/regexp/regexp3.html#caveats">time execution and a fixed stack footprint</a>. Presto Java uses <a href="https://github.com/jruby/joni">JONI</a> (<a href="https://github.com/kkos/oniguruma">a port of ONI, now archived/deprecated</a>). While both frameworks support almost all regular expression syntaxes, RE2 differs from JONI/PCRE in certain cases. A full list of unsupported regular expressions can be found <a href="https://github.com/google/re2/wiki/syntax">in this wiki</a>. A very interesting read about why RE2 skips certain perl syntax <a href="https://swtch.com/~rsc/regexp/regexp3.html#caveats">is also available here</a>. Fundamentals and detailed design of RE2 can be found in this three part amazing post (<a href="https://swtch.com/~rsc/regexp/regexp1.html">part1</a>, <a href="https://swtch.com/~rsc/regexp/regexp2.html">part2</a> and <a href="https://swtch.com/~rsc/regexp/regexp3.html#caveats">part3</a>).</p>



<p><strong>Regex Compilation Limit in Velox</strong></p>



<p>The number of regular expressions that can be compiled for a query is limited to 250. The hard cap is placed so that each individual query does not compile more than 250 regexes to keep the overall shared cluster environment healthy. If this limit is hit, try to optimize the query to use fewer compiled regular expressions. The reason for this limitation is that Regex compilation is CPU intensive and we have observed several cases where unbounded compilation has caused problems in the cluster historically.<br />NOTE: This number is only for regex’s that are created dynamically , for example:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">code_location_path LIKE &#039;%&#039; || test_name || &#039;%&#039;</code></pre></div>



<p>In this example the regex can change based on the test_name<em> </em>column value and a large enough number of test_name values can quickly exceed the 250 limit.&nbsp;</p>



<p>Instead this can be simply rewritten as:<br /><em>strpos(code_location, test_name) &gt; 0</em></p>



<p>Note: Velox also does <a href="https://velox-lib.io/blog/like/">several optimizations</a> to improve performance by skipping regex completely and using string comparisons when applicable.<br /></p>



<p><strong>Unsupported cases</strong></p>



<ul class="wp-block-list">
<li>RE2 does not support
<ul class="wp-block-list">
<li>before text matching (?=re)</li>



<li>before text not matching (?!re)</li>



<li>after text matching (?&lt;=re)&nbsp;</li>



<li>after text not matching (?&lt;!re)</li>
</ul>
</li>
</ul>



<p>These are supported in JONI</p>



<ul class="wp-block-list">
<li>RE2 wiki has a list of <a href="https://github.com/google/re2/wiki/syntax">unsupported PCRE syntaxes</a>, some of which are supported in JONI today.</li>



<li>Unsupported queries will fail in Prestissimo and must be re-written in a different way. However each rewrite may be different and the solution depends on the use case.</li>
</ul>



<h3 class="wp-block-heading">Aggregate Functions:</h3>



<ul class="wp-block-list">
<li><strong>Reduce Lambda Function:</strong>
<ul class="wp-block-list">
<li>Support is controlled by a session property (native_expression_max_array_size_in_reduce) as it is inefficient to support such cases for arbitrarily large arrays. Currently this is capped at <a href="https://github.com/facebookincubator/velox/blob/5a2aad8f75e5b428cab5099a1ca6e2431adb9338/velox/core/QueryConfig.h#L1041">100K</a>. Queries that fail due to this limit must be revised to meet this limit.</li>
</ul>
</li>
</ul>



<h3 class="wp-block-heading">Window Functions</h3>



<ul class="wp-block-list">
<li><strong>Issue:</strong> Aggregate window functions do not support `IGNORE NULLS`.</li>



<li><strong>Error Message:</strong> `!ignoreNulls Aggregate window functions do not support IGNORE NULLS.`</li>



<li><strong>Workaround:</strong> When encountering this issue, remove the `IGNORE NULLS` clause. This clause is only defined for <a href="https://prestodb.io/docs/current/functions/window.html#value-functions">value functions</a> and does not apply to aggregate window functions; In Presto the results obtained with and without the clause are similar, Prestissimo includes this clause whereas Presto just warns.</li>
</ul>



<h3 class="wp-block-heading">Casting&nbsp;</h3>



<p>&nbsp;&nbsp;<strong>Unicode Numerals</strong></p>



<ul class="wp-block-list">
<li>Casting of unicode strings to digits is not supported
<ul class="wp-block-list">
<li>Example</li>
</ul>
</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">CAST (‘Ⅶ’ as integer)&nbsp; -- Unsupported</code></pre></div>



<h3 class="wp-block-heading">URL Functions</h3>



<p>Presto and Prestissimo implement different URL functions specs which can lead to some URL function mismatches. Prestissimo implements <a href="https://datatracker.ietf.org/doc/html/rfc3986">RFC-3986</a> whereas Presto implements <a href="https://datatracker.ietf.org/doc/html/rfc2396">RFC-2396</a>. This can lead to subtle differences as highlighted in this <a href="https://github.com/facebookincubator/velox/issues/14204">issue</a>.</p>



<h3 class="wp-block-heading">Date Time Functions</h3>



<p>From_unixtime</p>



<ul class="wp-block-list">
<li>Currently the maximum date range supported for from_unixtime is between (<strong>292 Million BCE, 292 Million CE</strong>). The exact values corresponding to this are <strong>[292,275,055-05-16 08:54:06.192 BC, +292,278,994-08-17 00:12:55.807 CE]</strong>, and this corresponds to a unix time between <strong>[-9223372036854775, 9223372036854775].&nbsp; </strong>Presto Java also supports the same range (with JODA library), however it silently truncates (hence queries used to succeed), whereas Prestissimo throws an error if the values exceed this range.&nbsp;</li>
</ul>



<h3 class="wp-block-heading">Geospatial Differences</h3>



<p>There are cosmetic representation changes as well as numerical precision differences. Some of these differences result in different output for spatial predicates such as ST_Intersects. Differences include:</p>



<ol class="wp-block-list">
<li>Equivalent but different representations for geometries. Polygons may have their rings rotated, EMPTY geometries may be of a different type, MULTI-types and GEOMETRYCOLLECTIONs may have their elements in a different order. In general, WKTs/WKBs may be different.</li>



<li>Numerical precision: Differences in numerical techniques may result in different coordinate values, and also different results for predicates (ST_Relates and children, including ST_Contains, ST_Crosses, ST_Disjoint, ST_Equals, ST_Intersects, ST_Overlaps, ST_Relate, ST_Touches, ST_Within).</li>



<li>ST_IsSimple, ST_IsValid, simplify_geometry and geometry_invalid_reason may give different results.<br /></li>
</ol>



<h3 class="wp-block-heading">Time and Time with Time Zone</h3>



<h3 class="wp-block-heading"><strong>IANA Named Timezones Support Removed</strong></h3>



<ul class="wp-block-list">
<li>The support for IANA named time zones (e.g., &#8216;Europe/London&#8217;, &#8216;UTC&#8217;, &#8216;America/New_York&#8217;, &#8216;Asia/Kolkata&#8217;) in TIME and TIME WITH TIME ZONE has been removed to align with the SQL standard. For more details, refer to this<a href="https://github.com/prestodb/presto/issues/25957#issuecomment-3259814139"> Presto issue</a>.</li>



<li>Only fixed-offset time zones (e.g., +02:00) are now supported for these types.</li>



<li><strong>Note:</strong> Named time zones may still work when the Presto Java coordinator handles the query, but this support will be removed in the future. Migrate to fixed-offset time zones as soon as possible.</li>
</ul>



<p><strong>Example of Impacted Queries:</strong></p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- These will fail in Prestissimo (C++ engine), but may still work in legacy Presto:

cast(&#039;14:00:01 UTC&#039; as TIME WITH TIME ZONE) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; -- <img alt="❌" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/274c.png" style="height: 1em;" /> Error
cast(&#039;14:00:01 Europe/Paris&#039; as TIME WITH TIME ZONE) &nbsp; -- <img alt="❌" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/274c.png" style="height: 1em;" /> Error
cast(&#039;14:00:01 America/New_York&#039; as TIME WITH TIME ZONE) -- <img alt="❌" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/274c.png" style="height: 1em;" /> Error
cast(&#039;14:00:01 Asia/Kolkata&#039; as TIME WITH TIME ZONE) &nbsp; -- <img alt="❌" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/274c.png" style="height: 1em;" /> Error

-- These will work (fixed offset):
cast(&#039;14:00:01 +00:00&#039; as TIME WITH TIME ZONE) &nbsp; &nbsp; &nbsp; &nbsp; -- <img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> OK
cast(&#039;14:00:01 +05:30&#039; as TIME WITH TIME ZONE) &nbsp; &nbsp; &nbsp; &nbsp; -- <img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> OK</code></pre></div>



<h3 class="wp-block-heading">Casting from TIMESTAMP to TIME</h3>



<ul class="wp-block-list">
<li>One key difference is when TIME and TIME WITH TIME ZONE are produced when casting from TIMESTAMP (with or without TIME ZONE). (Note that TIMESTAMP behavior in Presto/Prestissimo is unchanged).</li>



<li>Previously, the result of CAST(TIMESTAMP AS TIME/TIME WITH TIME ZONE) would change based on the session property legacy_timestamp (true by default), when applied to the user&#8217;s time zone. In Prestissimo for TIME/TIME WITH TIMEZONE the behavior will be equivalent to the property being false.</li>
</ul>



<p><strong>Example of Impacted Queries:</strong></p>



<h3 class="wp-block-heading">Legacy/Default Presto Behavior</h3>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- Default behavior with legacy_timestamp=true:
-- Session Timezone - America/Los_Angeles

-- DST Active Dates 
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000&#039; as TIME);
-- Returns: 09:15:00.000
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000&#039; as TIME WITH TIME ZONE);
-- Returns: 09:15:00.000 America/Los_Angeles
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000 America/Los_Angeles&#039; as TIME);
-- Returns: 09:15:00.000
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000 America/Los_Angeles&#039; as TIME WITH TIME ZONE);
-- Returns: 09:15:00.000

-- DST Inactive Dates
select cast(TIMESTAMP &#039;2023-12-05 10:15:00.000&#039; as TIME);
-- Returns: 10:15:00.000
select cast(TIMESTAMP &#039;2023-12-05 10:15:00.000&#039; as TIME WITH TIME ZONE);
-- Returns: 10:15:00.000 America/Los_Angeles
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000 America/Los_Angeles&#039; as TIME);
-- Returns: 10:15:00.000
select cast(TIMESTAMP &#039;2023-12-05 10:15:00.000 America/Los_Angeles&#039; as TIME WITH TIME ZONE);
-- 10:15:00.000 America/Los_Angeles</code></pre></div>



<h3 class="wp-block-heading">New Behavior With Prestissimo (Velox)</h3>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-- New Expected behavior similar to what currently exists if legacy_timestamp=false:
-- Session Timezone - America/Los_Angeles


-- DST Active Dates 
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000&#039; as TIME);
-- Returns: 10:15:00.000
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000&#039; as TIME WITH TIME ZONE);
-- Returns: 10:15:00.000 -07:00
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000 America/Los_Angeles&#039; as TIME);
-- Returns: 10:15:00.000
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000 America/Los_Angeles&#039; as TIME WITH TIME ZONE);
-- Returns: 10:15:00.000 -07:00

-- DST Inactive Dates
select cast(TIMESTAMP &#039;2023-12-05 10:15:00.000&#039; as TIME);
-- Returns: 10:15:00.000
select cast(TIMESTAMP &#039;2023-12-05 10:15:00.000&#039; as TIME WITH TIME ZONE);
-- Returns: 10:15:00.000 -08:00
select cast(TIMESTAMP &#039;2023-08-05 10:15:00.000 America/Los_Angeles&#039; as TIME);
-- Returns: 10:15:00.000
select cast(TIMESTAMP &#039;2023-12-05 10:15:00.000 America/Los_Angeles&#039; as TIME WITH TIME ZONE);
-- Returns: 10:15:00.000 -08:00</code></pre></div>



<ul class="wp-block-list">
<li><strong>Note:</strong> TIMESTAMP will continue to support named time zones unlike TIME and TIME WITH TIME ZONE</li>
</ul>



<h2 class="wp-block-heading">DST Implications</h2>



<ul class="wp-block-list">
<li>Since IANA zones are not supported for TIME (still supported for Timestamp!), <strong>Prestissimo does not manage DST transitions</strong>. All time interpretation is strictly in the provided offset, not local civil time.</li>



<li>For example, &#8217;14:00:00 +02:00&#8242; always means 14:00 at a +02:00 fixed offset, regardless of DST changes that might apply under an IANA zone.</li>
</ul>



<h2 class="wp-block-heading">Recommendations</h2>



<ul class="wp-block-list">
<li><strong>Don’t use IANA time zone names for TIME/TIME WITH TIME ZONE:</strong> Use fixed-offset time zones like +02:00 instead of Europe/Paris or similar for TIME/TIME WITH TIME ZONE.</li>



<li><strong>Confirm your Prestissimo usage does not depend on legacy timestamp behavior: </strong>If your workload depends on having legacy TIME behavior (including support of IANA timezones), handle this outside Presto or reach out so that we can discuss alternative solutions.</li>



<li><strong>Test:</strong> Try your most critical workflows with these settings.</li>
</ul>



<h2 class="wp-block-heading">Follow Us </h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/01/22/presto-vs-prestissimo-known-differences-and-workarounds/">Presto vs Prestissimo &#8211; Known differences and workarounds</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
