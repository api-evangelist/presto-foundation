---
title: "From Zero to Contributor: A Complete Guide to Contributing to Presto Open Source"
url: "https://prestodb.io/blog/2026/01/09/from-zero-to-contributor-a-complete-guide-to-contributing-to-presto-open-source/"
date: "Fri, 09 Jan 2026 08:43:23 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<p>PrestoDB is a powerful distributed SQL query engine used widely for large-scale data analytics. Contributing to Presto is an excellent way to gain hands-on experience with distributed systems, Java, SQL engines, and large open-source codebases.</p>



<p>This step-by-step tutorial is designed specifically for beginners and first-time contributors who want to build Presto from source, run the Presto server locally, understand the codebase, and successfully submit their first pull request. By the end of this guide, you will have a fully working local Presto setup and a clear understanding of the complete contribution workflow, from Git setup to PR merge.</p>



<h2 class="wp-block-heading"><strong>Prerequisites</strong></h2>



<figure class="wp-block-table"><table class="has-fixed-layout"><thead><tr><th>Requirement</th><th>Details</th></tr></thead><tbody><tr><td><strong>Operating System</strong></td><td>macOS or Linux (Windows via WSL2)</td></tr><tr><td><strong>Java</strong></td><td>Java 17 (64-bit) &#8211; OpenJDK or Oracle JDK</td></tr><tr><td><strong>Python</strong></td><td>Python 2.4+ (Not Required for Development, only for launcher scripts)</td></tr><tr><td><strong>Git</strong></td><td>Latest version</td></tr><tr><td><strong>RAM</strong></td><td>Minimum 8GB (16GB recommended)</td></tr><tr><td><strong>Disk Space</strong></td><td>At least 10GB free</td></tr></tbody></table></figure>



<p></p>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-cf4a19f is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-cf4a19f-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-0riove4"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-0riove4" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-hc9iqrg"><p class="stk-block-text__text">Tip<br /><strong>IntelliJ IDEA</strong>&nbsp;is recommended for Presto development.</p></div>
</div></div></blockquote>



<h3 class="wp-block-heading">Knowledge Prerequisites</h3>



<ul class="wp-block-list">
<li>Java Programming Knowledge.</li>



<li>Familiarity with command-line tools</li>



<li>Git &amp; Version Control Basics.</li>



<li>Maven Build Tool.</li>



<li>SQL &amp; Distributed Systems knowledge (helpful but not required).</li>
</ul>



<h3 class="wp-block-heading">Required Accounts</h3>



<ul class="wp-block-list">
<li>GitHub Account</li>



<li>Slack Account (To Join Presto Slack <a href="https://communityinviter.com/apps/prestodb/prestodb" rel="noreferrer noopener" target="_blank"><code class="">Community</code></a>)</li>
</ul>



<h2 class="wp-block-heading">Configuring Git</h2>



<p>Before contributing to Presto, you must properly configure Git and GitHub to ensure your contributions are correctly attributed to your profile. If you are a macOS user, use the pre-installed <strong>Git</strong> . But, if you are using Linux or Windows, download the latest version of Git from <a href="https://git-scm.com/install/mac" rel="noreferrer noopener" target="_blank"><code class="">here</code></a>.</p>



<p>Open the terminal and run the below command.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git --version</code></pre></div>



<p>Configure Git with your name and email.</p>



<ul class="wp-block-list">
<li>Set your name</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git config --global user.name &quot;Your Name&quot;</code></pre></div>



<ul class="wp-block-list">
<li>Set your email</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git config --global user.email &quot;example@gmail.com&quot;</code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-08d60db is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-08d60db-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-2z8wlqn"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-2z8wlqn" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-8x4qpyd"><p class="stk-block-text__text">Important<br />Your Git commits must use an email address that is verified on your GitHub account. If the email does not match or is not verified, your contributions will not appear in your GitHub contribution graph, even if your PR is merged.</p></div>
</div></div></blockquote>



<h2 class="wp-block-heading">Installing Java Development Kit (JDK)</h2>



<ul class="wp-block-list">
<li>Download and Install Java 17 (either OpenJDK or Oracle JDK).</li>



<li>Download&nbsp;<strong>.dmg</strong>&nbsp;file (if you are on macOS), click <a href="https://www.azul.com/downloads/?version=java-17-lts&amp;os=macos&amp;package=jdk#zulu">here</a> to download.</li>



<li>Set <strong>JAVA_HOME</strong> (add to ~/.zshrc or ~/.bash_profile).</li>



<li>Verify JDK Installation</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">java -version</code></pre></div>



<h2 class="wp-block-heading">Forking and Cloning the Repository</h2>



<h5 class="wp-block-heading">Forking The Repository</h5>



<ul class="wp-block-list">
<li>Navigate to the&nbsp;<a href="https://github.com/prestodb/presto" rel="noreferrer noopener" target="_blank"><code class="">Presto GitHub repository</code></a></li>



<li>Click the&nbsp;<strong>&#8220;Fork&#8221;</strong>&nbsp;button in the top-right corner</li>



<li>Create a copy of the Presto repository under your GitHub account.</li>
</ul>



<h5 class="wp-block-heading">Cloning Your Forked Repository</h5>



<ul class="wp-block-list">
<li>Navigate to the directory where you want to store the project on your local machine.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">cd ~/ Desktop (or your preferred Directory)</code></pre></div>



<ul class="wp-block-list">
<li>Clone the project</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git clone https://github.com/YOUR_USERNAME/presto.git</code></pre></div>



<ul class="wp-block-list">
<li>Navigate to the directory</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">cd ~/ presto</code></pre></div>



<ul class="wp-block-list">
<li>Verify the cloned repository.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">ls -la</code></pre></div>



<h2 class="wp-block-heading">Understanding the Presto Project Structure (Codebase)</h2>



<p>Understand the key modules in the Presto codebase:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto/
├── .github/&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # GitHub workflows and templates
├── docker/ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Docker configurations
├── presto-accumulo/&nbsp; &nbsp; &nbsp; # Accumulo connector
├── presto-analyzer/&nbsp; &nbsp; &nbsp; # Semantic analysis and query validation
├── presto-base-jdbc/ &nbsp; &nbsp; # Base JDBC connector
├── presto-cli/ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Command-line interface
├── presto-client/&nbsp; &nbsp; &nbsp; &nbsp; # Client libraries
├── presto-common/&nbsp; &nbsp; &nbsp; &nbsp; # Common utilities
├── presto-docs/&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Documentation (Sphinx/RST)
├── presto-hive/&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Hive connector
├── presto-main/&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Core engine
├── presto-parser/&nbsp; &nbsp; &nbsp; &nbsp; # SQL parser
├── presto-server/&nbsp; &nbsp; &nbsp; &nbsp; # Server packaging
├── presto-spi/ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Service Provider Interface
├── presto-tests/ &nbsp; &nbsp; &nbsp; &nbsp; # Integration tests
├── pom.xml &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Root Maven configuration
├── CONTRIBUTING.md &nbsp; &nbsp; &nbsp; # Contribution guidelines
└── README.md &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Project overview
</code></pre></div>



<figure class="wp-block-table"><table class="has-fixed-layout"><thead><tr><th>File</th><th>Purpose</th></tr></thead><tbody><tr><td><a href="https://github.com/prestodb/presto/blob/master/README.md" rel="noreferrer noopener" target="_blank"><code class="">README.md</code></a></td><td>Project overview and build instructions</td></tr><tr><td><a href="https://github.com/prestodb/presto/blob/master/CONTRIBUTING.md" rel="noreferrer noopener" target="_blank"><code class="">CONTRIBUTING.md</code></a></td><td>Contribution guidelines</td></tr><tr><td><a href="https://github.com/prestodb/presto/blob/master/ARCHITECTURE.md" rel="noreferrer noopener" target="_blank"><code class="">ARCHITECTURE.md</code></a></td><td>Mission and technical architecture</td></tr><tr><td><a href="https://github.com/prestodb/presto/blob/master/pom.xml" rel="noreferrer noopener" target="_blank"><code class="">pom.xml</code></a></td><td>Maven project configuration</td></tr><tr><td><a href="https://github.com/prestodb/presto/blob/master/CODEOWNERS" rel="noreferrer noopener" target="_blank"><code class="">CODEOWNERS</code></a></td><td>Code ownership and module maintainers</td></tr></tbody></table></figure>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-a86c442 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-a86c442-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-4u98egs"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-4u98egs" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-cfw4ply"><p class="stk-block-text__text">Info<br />Note that each module has its own&nbsp;<code class="">pom.xml</code>&nbsp;and follows Maven’s standard directory structure.</p></div>
</div></div></blockquote>



<h2 class="wp-block-heading">Setting Up IntelliJ IDEA for Presto</h2>



<h5 class="wp-block-heading">1. Configure Maven:</h5>



<ul class="wp-block-list">
<li>Go to IntelliJ IDEA → Settings → Build, Execution, Deployment → Build Tools → Maven → Use Maven wrapper</li>
</ul>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2649" height="516" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260105130259-1024x516.png" width="1024" /></figure>



<ul class="wp-block-list">
<li>Go to Maven → Runner → Enable&nbsp;<strong>Skip Tests</strong>&nbsp;→ Click Apply → OK</li>
</ul>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2650" height="512" src="https://prestodb.io/wp-content/uploads/Screenshot-2026-01-06-at-6.31.36-PM-1024x512.png" width="1024" /></figure>



<h5 class="wp-block-heading">2. Configure JDK:</h5>



<ul class="wp-block-list">
<li>Go to File → Project Structure → Project</li>



<li>Set SDK to Java 17 (or download directly)</li>
</ul>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2651" height="512" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260105130903-1024x512.png" width="1024" /></figure>



<ul class="wp-block-list">
<li>Go to Run → Edit Configurations</li>
</ul>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2652" height="376" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260106183949-1024x376.png" width="1024" /></figure>



<ul class="wp-block-list">
<li>Set&nbsp;<strong>Java 17</strong>&nbsp;as SDK and&nbsp;<strong>presto-main</strong>&nbsp;as class path.</li>



<li>Add below configurations in&nbsp;<strong>VM Options</strong>.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">-ea  
-XX:+UseG1GC  
-XX:G1HeapRegionSize=32M  
-XX:+UseGCOverheadLimit  
-XX:+ExplicitGCInvokesConcurrent  
-Xmx2G  
-Dconfig=/Users/your-complete-path/presto/presto-main/etc/config.properties  ----&gt; Paste presto-main/etc/config.properties path
-Dlog.levels-file=etc/log.properties  
-Djdk.attach.allowAttachSelf=true   
--add-opens=java.base/java.io=ALL-UNNAMED  
--add-opens=java.base/java.lang=ALL-UNNAMED  
--add-opens=java.base/java.lang.ref=ALL-UNNAMED  
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED  
--add-opens=java.base/java.net=ALL-UNNAMED  
--add-opens=java.base/java.nio=ALL-UNNAMED  
--add-opens=java.base/java.security=ALL-UNNAMED  
--add-opens=java.base/javax.security.auth=ALL-UNNAMED  
--add-opens=java.base/javax.security.auth.login=ALL-UNNAMED  
--add-opens=java.base/java.text=ALL-UNNAMED  
--add-opens=java.base/java.util=ALL-UNNAMED  
--add-opens=java.base/java.util.concurrent=ALL-UNNAMED  
--add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED  
--add-opens=java.base/java.util.regex=ALL-UNNAMED  
--add-opens=java.base/jdk.internal.loader=ALL-UNNAMED  
--add-opens=java.base/sun.security.action=ALL-UNNAMED  
--add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED
</code></pre></div>



<ul class="wp-block-list">
<li>Set Main Class to&nbsp;<code class="">com.facebook.presto.server.PrestoServer</code></li>



<li>Set Working Directory&nbsp;<code class="">$MODULE_DIR$</code></li>



<li>Click on Modify options and tick the options, as mentioned in below snapshot.</li>
</ul>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2653" height="508" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260109124506-1024x508.png" width="1024" /></figure>



<ul class="wp-block-list">
<li>Click Apply to save the configuration. IntelliJ is now set up and ready for Presto development.</li>
</ul>



<h2 class="wp-block-heading">Building Presto from Source</h2>



<p>Build Presto from source to verify your environment and test your changes.</p>



<ul class="wp-block-list">
<li>Navigate to the directory where presto is cloned.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">cd ~/Desktop/presto</code></pre></div>



<ul class="wp-block-list">
<li>Run the following command to build Presto for the first time.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">./mvnw clean install -DskipTests</code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-a8db243 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-a8db243-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-48lehvn"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-48lehvn" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-gf3p3vj"><p class="stk-block-text__text">Info<br /><strong>mvnw</strong>: Runs Maven Wrapper<br /><strong>clean</strong>: Removes previous build artifacts<br /><strong>install</strong>: Compiles code, packages it, and installs to local Maven repository<br /><strong>DskipTests</strong>: Skips running tests (faster for initial build)</p></div>
</div></div></blockquote>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-227eca9 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-227eca9-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-065srlk"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-065srlk" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-kjbeqwi"><p class="stk-block-text__text">Important<br />Skipping tests is acceptable&nbsp;<strong>only for initial setup or local exploration</strong>.<br /><strong>All PRs must pass tests</strong>&nbsp;and should be built&nbsp;<strong>without&nbsp;<code class="">-DskipTests</code></strong>&nbsp;before submission.</p></div>
</div></div></blockquote>



<p><strong>Expected Output</strong></p>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2654" height="819" src="https://prestodb.io/wp-content/uploads/Pasted-image-20251226010310-1024x819.png" width="1024" /></figure>



<ul class="wp-block-list">
<li>Verify the setup and run the Presto server directly from IntelliJ.</li>
</ul>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2655" height="645" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260106185654-1024x645.png" width="1024" /></figure>



<ul class="wp-block-list">
<li>Confirm that the Presto server is running on&nbsp;<code class="">localhost:8080</code></li>
</ul>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2656" height="563" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260106185929-1024x563.png" width="1024" /></figure>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-8e43458 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-8e43458-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-epv5q9t"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-epv5q9t" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-vz80u53"><p class="stk-block-text__text">Success<br />Server is running on&nbsp;<strong>localhost:8080</strong></p></div>
</div></div></blockquote>



<h5 class="wp-block-heading">Connect with Presto CLI</h5>



<ul class="wp-block-list">
<li>Open a new terminal, navigate to the&nbsp;<strong>presto-cli</strong>&nbsp;directory, and then move into the&nbsp;<strong>target</strong>&nbsp;directory. Run below command.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">cd presto-cli/target</code></pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">java -jar presto-cli-*-SNAPSHOT-executable.jar \
  --server localhost:8080</code></pre></div>



<ul class="wp-block-list">
<li>Verify that the presto prompt appears.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">presto&gt; show catalogs;</code></pre></div>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2657" height="569" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260106211811-1024x569.png" width="1024" /></figure>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-41f2e62 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-41f2e62-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-hejsnrp"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-hejsnrp" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-zdnmd3s"><p class="stk-block-text__text">Success<br />Connected with Presto CLI</p></div>
</div></div></blockquote>



<h5 class="wp-block-heading">Build Specific Modules</h5>



<ul class="wp-block-list">
<li>To build only a specific module, run the below command:</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">./mvnw clean install -DskipTests -pl presto-cli</code></pre></div>



<h2 class="wp-block-heading">Setting Up Upstream Remote</h2>



<p>Keep your fork synchronized with the main Presto repository.</p>



<ul class="wp-block-list">
<li>Add the upstream remote</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git remote add upstream https://github.com/prestodb/presto.git</code></pre></div>



<ul class="wp-block-list">
<li>Verify your remote</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git remote -v</code></pre></div>



<p><strong>Expected Output</strong></p>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2658" height="289" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260106190033-1024x289.png" width="1024" /></figure>



<ul class="wp-block-list">
<li><strong>origin</strong>: Your fork on GitHub (where you push changes)</li>



<li><strong>upstream</strong>: The main Presto repository (where you pull updates)</li>
</ul>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-bf17873 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-bf17873-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-i0an5q1"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-i0an5q1" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-9hz0rvg"><p class="stk-block-text__text">Note<br />Do not commit changes directly to&nbsp;<code class="">master</code>. Always create a feature branch.</p></div>
</div></div></blockquote>



<h2 class="wp-block-heading">Making Code Contributions</h2>



<h5 class="wp-block-heading">Syncing Your Fork</h5>



<ul class="wp-block-list">
<li>Before creating a new branch, ensure your fork is up to date:</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">#Make sure you&#039;re on master
git checkout master 

#Fetch updates from upstream
git fetch upstream 

#Merge upstream changes into your master
git merge upstream/master 

#Push updates to your fork
git push origin master </code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-b47609e is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-b47609e-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-1sbykk8"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-1sbykk8" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-1wumf82"><p class="stk-block-text__text">Tip<br />Sync your fork regularly to avoid merge conflicts.</p></div>
</div></div></blockquote>



<h5 class="wp-block-heading">Find an Issue to Fix</h5>



<ul class="wp-block-list">
<li>Visit the&nbsp;<a href="https://github.com/prestodb/presto/issues" rel="noreferrer noopener" target="_blank"><code class="">Presto Issues Page</code></a>, or</li>



<li>Look for issues that interest you, or</li>



<li>Find an Issue to fix with labels like&nbsp;<code class="">good first issues</code>,&nbsp;<code class="">documentation</code>, etc</li>
</ul>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-a1d427b is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-a1d427b-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-zjazxtu"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-zjazxtu" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-l9ybyic"><p class="stk-block-text__text">Important<br />Comment on the issue to let others know you&#8217;re working on it.</p></div>
</div></div></blockquote>



<h5 class="wp-block-heading">Creating a Feature Branch</h5>



<ul class="wp-block-list">
<li>Create and switch to a new branch, use descriptive branch names that indicate what you&#8217;re working on:</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git checkout -b feature/meaningful-branch-name</code></pre></div>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-f1b57a1 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-f1b57a1-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-e11zory"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-e11zory" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-ir77hex"><p class="stk-block-text__text"><strong>Example</strong><br />feature/add-trim-function<br />bugfix/fix-null-pointer-in-parser<br />docs/update-contribution-guide<br />refactor/simplify-planner-logic<br />test/add-hive-connector-tests</p></div>
</div></div></blockquote>



<p></p>



<ul class="wp-block-list">
<li>Verify that you are on the new branch.</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git branch</code></pre></div>



<p><strong>Expected Output</strong></p>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-2659" height="298" src="https://prestodb.io/wp-content/uploads/Pasted-image-20260106211440-1024x298.png" width="1024" /></figure>



<ul class="wp-block-list">
<li>Make Your Changes in the Code</li>



<li>Write or Update Tests</li>



<li>Run Tests Locally</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class=""># Run tests for the module you changed
cd presto-main
../mvnw test

# If all pass, run full build
cd ..
./mvnw clean install</code></pre></div>



<ul class="wp-block-list">
<li>Commit your changes</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class=""># Stage your changes
git add .

# Commit with proper format
git commit -m &quot;Your Message&quot;
</code></pre></div>



<ul class="wp-block-list">
<li>Write clear and descriptive commit messages.</li>
</ul>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-36d68e4 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-36d68e4-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-vluoscr"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-vluoscr" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-37lkrkx"><p class="stk-block-text__text">Example<br /><strong>Commit Message Format:</strong><br /><code class="">&lt;type&gt; [(scope)]: &lt;subject&gt;</code><br /><code class="">[optional body]</code><br /><code class="">[optional footer]</code><br /><strong>Types:</strong><br />feat: New feature<br />fix: Bug fix<br />docs: Documentation only<br />refactor: Code refactoring<br />perf: Performance improvement<br />test: Adding or modifying tests<br />build: Build system changes</p></div>
</div></div></blockquote>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-3ecbd58 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-3ecbd58-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-e24wk9v"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-e24wk9v" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-8mi5no6"><p class="stk-block-text__text">Todo<br /><strong>Subject Line Rules</strong>:<br />Start with capital letter<br />Use imperative mood (&#8220;Add feature&#8221; not &#8220;Added feature&#8221;)<br />No period at the end<br />Be concise but descriptive<br />Limit to 50-72 characters when possible</p></div>
</div></div></blockquote>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-b09cbee is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-b09cbee-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-gvjgf5k"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-gvjgf5k" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-9y7eqb1"><p class="stk-block-text__text">Example<br />git commit -m &#8220;feat(function): Add trim function with custom characters&#8221;<br />git commit -m &#8220;fix(parser): Handle null values in WHERE clause&#8221;<br />git commit -m &#8220;docs(connector): Update Hive connector configuration guide&#8221;</p></div>
</div></div></blockquote>



<p></p>



<ul class="wp-block-list">
<li>Push to your Fork</li>
</ul>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-code"><code class="">git push origin feature/meaningful-branch-name</code></pre></div>



<ul class="wp-block-list">
<li>Create a&nbsp;<strong>Pull Request (PR)</strong></li>
</ul>



<ol class="wp-block-list">
<li>Go to your fork on GitHub:&nbsp;<code class="">https://github.com/YOUR_USERNAME/presto</code></li>



<li>Click&nbsp;<strong>&#8220;Compare &amp; pull request&#8221;</strong></li>



<li>Set:&nbsp;<strong>Base repo</strong>:&nbsp;<code class="">prestodb/presto</code>,&nbsp;<strong>Base branch</strong>:&nbsp;<code class="">master</code>,&nbsp;<strong>Head repo</strong>: your fork,&nbsp;<strong>Compare branch</strong>:&nbsp;<code class="">your feature branch</code></li>



<li>Fill out PR Template.</li>
</ol>



<ul class="wp-block-list">
<li>Sign the CLA</li>
</ul>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-773d941 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-773d941-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-vyt79hn"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-vyt79hn" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-9wy6s8i"><p class="stk-block-text__text">Important<br />On your first PR, the&nbsp;<strong>CLA bot</strong>&nbsp;will comment asking you to sign the Contributor License Agreement:<br />Click the link provided by the bot<br />Sign the CLA electronically<br />The bot will update your PR status</p></div>
</div></div></blockquote>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-fb362a8 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-fb362a8-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-lfr3iby"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-lfr3iby" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-qvl8kpo"><p class="stk-block-text__text">Success<br />You have successfully submitted your PR to Presto Open Source Project</p></div>
</div></div></blockquote>



<h2 class="wp-block-heading">After Your PR is Merged</h2>



<p>Congratulations! You are now a contributor to the Presto project.</p>



<h2 class="wp-block-heading">Creating Your Own Issues</h2>



<p>If you find a bug or have an idea for improvement:</p>



<h5 class="wp-block-heading">Search for Existing Issues</h5>



<ul class="wp-block-list">
<li>Make sure it hasn&#8217;t been reported already</li>



<li>Search closed issues too (it might have been fixed)</li>
</ul>



<h5 class="wp-block-heading">Create a New Issue</h5>



<ul class="wp-block-list">
<li>Go to&nbsp;<a href="https://github.com/prestodb/presto/issues/new/choose" rel="noreferrer noopener" target="_blank">github.com/prestodb/presto/issues/new/choose</a></li>



<li>Choose the appropriate template (Bug Report, Feature Request, etc.)</li>



<li>Fill out all sections thoroughly</li>



<li>Add relevant labels</li>



<li>Submit the issue</li>
</ul>



<h5 class="wp-block-heading">Discuss Before Implementing</h5>



<ul class="wp-block-list">
<li>Wait for feedback from maintainers</li>



<li>Get approval before starting work</li>
</ul>



<blockquote class="wp-block-stackable-blockquote stk-block-blockquote stk-block stk-d85fc32 is-style-default"><div class="has-text-align-left stk-block-blockquote__content stk-container stk-d85fc32-container stk-hover-parent"><div class="stk-block-content stk-inner-blocks">
<div class="wp-block-stackable-icon stk-block-icon stk-block stk-dwjkry9"><span class="stk--svg-wrapper"><div class="stk--inner-svg"><svg xmlns="http://www.w3.org/2000/svg"><defs><linearGradient id="linear-gradient-dwjkry9" x1="0" x2="100%" y1="0" y2="0"><stop offset="0%"></stop><stop offset="100%"></stop></linearGradient></defs></svg><svg height="32" viewBox="0 0 50 50" width="32" xmlns="http://www.w3.org/2000/svg"><path d="M19.8 9.3C10.5 11.8 4.6 17 2.1 24.8c2.3-3.6 5.6-5.4 9.9-5.4 3.3 0 6 1.1 8.3 3.3 2.2 2.2 3.4 5 3.4 8.3 0 3.2-1.1 5.8-3.3 8-2.2 2.2-5.1 3.2-8.7 3.2-3.7 0-6.5-1.2-8.6-3.5C1 36.3 0 33.1 0 29 0 18.3 6.5 11.2 19.6 7.9l.2 1.4zm26.4 0C36.9 11.9 31 17 28.5 24.8c2.2-3.6 5.5-5.4 9.8-5.4 3.2 0 6 1.1 8.3 3.2 2.3 2.2 3.4 4.9 3.4 8.3 0 3.1-1.1 5.8-3.3 7.9-2.2 2.2-5.1 3.3-8.6 3.3-3.7 0-6.6-1.1-8.6-3.4-2.1-2.3-3.1-5.5-3.1-9.7 0-10.7 6.6-17.8 19.7-21.1l.1 1.4z"></path></svg></div></span></div>



<div class="wp-block-stackable-text stk-block-text stk-block stk-t7dbzi6"><p class="stk-block-text__text">Important links<br />Presto Website:&nbsp;<a href="https://prestodb.io/" rel="noreferrer noopener" target="_blank"><code class="">https://prestodb.io</code></a><br />GitHub Repository:&nbsp;<a href="https://github.com/prestodb/presto" rel="noreferrer noopener" target="_blank"><code class="">https://github.com/prestodb/presto</code></a><br />Slack:&nbsp;<a href="https://communityinviter.com/apps/prestodb/prestodb" rel="noreferrer noopener" target="_blank"><code class="">https://communityinviter.com/apps/prestodb/prestodb</code></a></p></div>
</div></div></blockquote>



<p></p>



<h2 class="wp-block-heading">Follow Us </h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/01/09/from-zero-to-contributor-a-complete-guide-to-contributing-to-presto-open-source/">From Zero to Contributor: A Complete Guide to Contributing to Presto Open Source</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
