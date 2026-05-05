---
title: "Password Authentication Setup on Local"
url: "https://prestodb.io/blog/2026/04/30/password-authentication-setup-on-local/"
date: "Thu, 30 Apr 2026 16:49:39 +0000"
author: "saurabh"
feed_url: "https://prestodb.io/feed/"
---
<p>Authentication and authorization are two main pillars for data security. While a Presto cluster can be set up to run without authentication for development purposes, production clusters must be secured at all times. Setting up secure clusters comes with its own challenges in terms of the involved setup and configuration changes. In this blog, we try to set up a secure Presto cluster on a local machine using password file based authentication and resolve the errors incrementally that come along the way.</p>



<p>Troubleshooting SSL handshake related errors can be overwhelming: most of the errors are not obvious, and Google searches can frequently give unsatisfactory answers. Before dealing with these errors, it is recommended to have good knowledge of</p>



<ol class="wp-block-list">
<li>What happens in a typical SSL handshake?</li>



<li>How are TLS connections different from mTLS connections?</li>



<li>What is a keystore and a truststore?</li>



<li>What is the role of keystore and truststore in SSL connections?</li>



<li>Basic Java tools like <code class="">keytool</code> for managing keystores, truststores and certificates.</li>
</ol>



<h3 class="wp-block-heading">Expectations</h3>



<ol class="wp-block-list">
<li>Configure a Presto cluster to authenticate users with password file based authentication on a local machine.</li>



<li>Verify that once the Presto cluster is secured, no user is able to connect to the server without authenticating themselves.</li>
</ol>



<h3 class="wp-block-heading">SSL Handshakes</h3>



<p>Let us start with some theory that I feel is important to understand before going to hands on. SSL handshakes are a mechanism by which client and server establish the trust and logistics required to secure their connection over the network. SSL handshake involves many steps, but we go over only the ones relevant for this blog:</p>



<ul class="wp-block-list">
<li>Client provides a list of possible SSL versions and cipher suites to use.</li>



<li>Server agrees on a particular SSL version and cipher suite, responding with its certificate from its keystore.</li>



<li>Client verifies the authenticity of this certificate by checking it against a list of trusted Certificate Authorities (CAs) using its truststore.</li>
</ul>



<h3 class="wp-block-heading">Keystore and Truststore</h3>



<p>Keystore and Truststore are used to manage the keys and certificates required for secure communication.</p>



<ul class="wp-block-list">
<li>The Keystore stores your private keys and the corresponding public certificates. During a SSL handshake, the Presto server uses its keystore to present its identity to the client such as Presto CLI.</li>



<li>A truststore stores only public certificates, typically from trusted Certificate Authorities. Clients such as Presto CLI will make use of its truststore to verify the certificate presented by the Presto coordinator. If the server&#8217;s certificate or the corresponding CA is not in the client&#8217;s truststore, the connection fails with <code class="">SSLHandshakeException</code>.</li>
</ul>



<p>It is important to note that Java provides default truststore (<code class="">cacerts</code>) but there is no default keystore. So during a handshake, the connection can succeed when the server presents a certificate which is issued by well known CAs because the certificates issued by these CAs are already present in the default truststore. When testing, the certificate could be a self-signed one, and a custom truststore will be needed which contains the presented certificate.</p>



<h3 class="wp-block-heading">TLS and mTLS</h3>



<p>mTLS stands for mutual TLS (Transport Layer Security). The key difference between TLS and mTLS is that TLS uses a one way handshake where only the server presents its certificate to the client and the client verifies that certificate using its truststore. However, mTLS adds a mandatory second step where the client also presents its certificate to the server, allowing the server to authenticate the client, ensuring a strict two-way, zero trust security. In a TLS handshake, the client only needs a truststore to verify the server&#8217;s identity, while in a mTLS handshake, both server and client need a keystore and truststore to present the certificate and verify the other&#8217;s identity.</p>



<h4 class="wp-block-heading">Can keystore and truststore be the same?</h4>



<p>Because both a keystore and truststore are needed in an mTLS handshake for both the involved parties, this raises the obvious question &#8211;&nbsp;<strong>Do keystore and truststore need to be different? Can they use the same file?</strong></p>



<p>Technically, a keystore and truststore can use the same physical file as they both use the same underlying formats and management tools like the&nbsp;<code class="">Java Keytool</code>. However, it is recommended to keep them separate due to the following reasons:</p>



<ul class="wp-block-list">
<li><strong>Security Risk</strong>: Truststore often have lower security requirements since they only contain public data. If you combine them, anyone with access to truststore can access your highly sensitive private keys as well.</li>



<li><strong>Maintenance</strong>: Keeping keystore and truststore separate makes it easier to manage the certificates especially when certificates are rotated at regular intervals.</li>



<li><strong>Default Values</strong>: Java provides a default truststore, but there is no default keystore.</li>
</ul>



<h3 class="wp-block-heading">Keytool</h3>



<p>The Keytool command is a key and certificate management utility provided by Java as part of its releases. Please refer to the&nbsp;<a href="https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html">official documentation</a>&nbsp;and&nbsp;<a href="https://www.baeldung.com/keytool-intro">Introduction to Keytool</a>&nbsp;for more information.</p>



<h2 class="wp-block-heading">Set up Presto on a local machine</h2>



<p>Use Intellij IDEA for setting up Presto on local. See&nbsp;<a href="https://github.com/prestodb/presto?tab=readme-ov-file#building-presto">Building Presto</a>&nbsp;for instructions.</p>



<h2 class="wp-block-heading">Set up password file based authentication</h2>



<p>Having almost no idea about how SSL connection works internally, I performed these steps:</p>



<ol class="wp-block-list">
<li>Created a <code class="">password.db</code> file for configuring <code class="">test</code> user using the commands:</li>
</ol>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro Documents % vi password.db 
pratyakshsharma@Pratyakshs-MacBook-Pro Documents % htpasswd -B -C 10 password.db test
New password:
Re-type new password:
Adding password for user test</pre></div>



<p>I was not sure if a keystore path was needed for the Presto coordinator, so I added only these configurations to&nbsp;<code class="">config.properties</code>:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">http-server.https.enabled=true
http-server.https.port=8443
http-server.authentication.type=PASSWORD</pre></div>



<p>I created a new&nbsp;<code class="">password-authenticator.properties</code>&nbsp;file:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">password-authenticator.name=file
file.password-file=/Users/pratyakshsharma/Documents/password.db</pre></div>



<p>When I tried to start the Presto server, this error appeared:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">1) [Guice/ErrorInCustomProvider]: NullPointerException
  while locating HttpServerProvider
  at HttpServerModule.configure(HttpServerModule.java:54)
  while locating HttpServer

Learn more:
  https://github.com/google/guice/wiki/ERROR_IN_CUSTOM_PROVIDER
Caused by: NullPointerException
	at java.base/Objects.requireNonNull(Objects.java:208)
	at java.base/UnixFileSystem.getPath(UnixFileSystem.java:263)
	at java.base/Path.of(Path.java:147)
	at HttpServer.tryLoadPemKeyStore(HttpServer.java:526)
	at HttpServer.&lt;init&gt;(HttpServer.java:238)
	at HttpServerProvider.get(HttpServerProvider.java:145)
	at HttpServerProvider.get(HttpServerProvider.java:43)
	at ProviderInternalFactory.provision(ProviderInternalFactory.java:86)
	at BoundProviderFactory.provision(BoundProviderFactory.java:72)
	at ProviderInternalFactory$1.call(ProviderInternalFactory.java:67)
	at ProvisionListenerStackCallback$Provision.provision(ProvisionListenerStackCallback.java:109)
	at LifeCycleModule.provision(LifeCycleModule.java:53)
	at ProvisionListenerStackCallback$Provision.provision(ProvisionListenerStackCallback.java:117)
	at ProvisionListenerStackCallback.provision(ProvisionListenerStackCallback.java:66)
	at ProviderInternalFactory.circularGet(ProviderInternalFactory.java:62)
	at BoundProviderFactory.get(BoundProviderFactory.java:59)
	at ProviderToInternalFactoryAdapter.get(ProviderToInternalFactoryAdapter.java:40)
	at SingletonScope$1.get(SingletonScope.java:169)
	at InternalFactoryToProviderAdapter.get(InternalFactoryToProviderAdapter.java:45)
	at InternalInjectorCreator.loadEagerSingletons(InternalInjectorCreator.java:213)
	at InternalInjectorCreator.injectDynamically(InternalInjectorCreator.java:186)
	at InternalInjectorCreator.build(InternalInjectorCreator.java:113)
	at Guice.createInjector(Guice.java:87)
	at Bootstrap.initialize(Bootstrap.java:263)
	at PrestoServer.run(PrestoServer.java:169)
	at PrestoServer.main(PrestoServer.java:103)</pre></div>



<p>On checking the line of code which was throwing the&nbsp;<code class="">NullPointerException</code>, I figured out a keystore was needed for the http-server. So I created a keystore on the local system with the following commands:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % keytool -genkeypair -alias presto -keyalg RSA -keystore keystore.jks
Enter keystore password:  
Re-enter new password: 
What is your first and last name?
  [Unknown]:  pratyaksh sharma
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  
What is the name of your State or Province?
  [Unknown]:  
What is the two-letter country code for this unit?
  [Unknown]:  
Is CN=pratyaksh sharma, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Enter key password for &lt;presto&gt;
	(RETURN if same as keystore password):  

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.jks -deststoretype pkcs12".</pre></div>



<p>Then I added two new properties in&nbsp;<code class="">config.properties</code>:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">http-server.https.keystore.path=/Users/pratyakshsharma/Documents/presto-https/keystore.jks
http-server.https.keystore.key=password</pre></div>



<p>Then I tried again to start the Presto server, and the following error was displayed:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">2026-04-14T14:33:18.767+0545	INFO	main	com.facebook.presto.server.security.PasswordAuthenticatorManager	-- Loading password authenticator --
2026-04-14T14:33:18.767+0545	ERROR	main	com.facebook.presto.server.PrestoServer	Password authenticator file is not registered
java.lang.IllegalStateException: Password authenticator file is not registered
	at com.google.common.base.Preconditions.checkState(Preconditions.java:601)
	at com.facebook.presto.server.security.PasswordAuthenticatorManager.loadPasswordAuthenticator(PasswordAuthenticatorManager.java:73)
	at com.facebook.presto.server.PrestoServer.run(PrestoServer.java:209)
	at com.facebook.presto.server.PrestoServer.main(PrestoServer.java:103)</pre></div>



<p>This is because&nbsp;<code class="">PasswordAuthenticatorPlugin</code>&nbsp;is not registered with the server. To fix this, I added this line to&nbsp;<code class="">plugin.bundles</code>&nbsp;in&nbsp;<code class="">config.properties</code>:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">../presto-password-authenticators/pom.xml</pre></div>



<p>When I tried again, the server started successfully.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">2026-04-16T12:14:39.909+0545	INFO	main	com.facebook.presto.server.PrestoServer	======== SERVER STARTED ========</pre></div>



<p>Next I tried to run Presto CLI and the following error appeared:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto % presto-cli/target/presto-cli-0.297-SNAPSHOT-executable.jar --user test --password
Password:
        Exception in thread "main" java.lang.IllegalArgumentException: Authentication using username/password requires HTTPS to be enabled
        at com.google.common.base.Preconditions.checkArgument(Preconditions.java:143)
        at com.facebook.presto.cli.QueryRunner.setupBasicAuth(QueryRunner.java:166)
        at com.facebook.presto.cli.QueryRunner.&lt;init&gt;(QueryRunner.java:95)
        at com.facebook.presto.cli.Console.run(Console.java:143)
        at com.facebook.presto.cli.Presto.main(Presto.java:31)</pre></div>



<p>To enable HTTPS as the error suggested, I had to include the&nbsp;<code class="">--server</code>&nbsp;flag with an HTTPS endpoint because it uses&nbsp;<code class="">http://localhost:8080</code>&nbsp;by default.</p>



<p>I ran the new command &#8211;</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto % presto-cli/target/presto-cli-0.297-SNAPSHOT-executable.jar --user test --server https://localhost:8443 --password
Password: 
presto&gt; </pre></div>



<p>And that was successful!</p>



<p>I was able to access the interactive shell, but when I tried to run a simple query such as&nbsp;<code class="">show catalogs</code>, I hit another error:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">presto&gt; show catalogs;
Error running command: javax.net.ssl.SSLHandshakeException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target</pre></div>



<p>For authentication related errors, you will not see any stacktrace on the Presto console because the query does not reach the Presto coordinator.</p>



<p>On exploring a bit, I figured out that the Common Name that I had used in my certificate in keystore was not correct and it needs to be the unqualified hostname of the Presto coordinator. So I created a fresh keystore as below:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % hostname
Pratyakshs-MacBook-Pro.local</pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % keytool -genkeypair -alias presto -keyalg RSA -keystore keystore1.jks 
Enter keystore password:  
Re-enter new password: 
What is your first and last name?
  [Unknown]:  Pratyakshs-MacBook-Pro.local
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  
What is the name of your State or Province?
  [Unknown]:  
What is the two-letter country code for this unit?
  [Unknown]:  
Is CN=Pratyakshs-MacBook-Pro.local, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Enter key password for &lt;presto&gt;
	(RETURN if same as keystore password):  

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore keystore1.jks -destkeystore keystore1.jks -deststoretype pkcs12".</pre></div>



<p>I updated the value of the property in&nbsp;<code class="">config.properties</code>:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">http-server.https.keystore.path=/Users/pratyakshsharma/Documents/presto-https/keystore1.jks</pre></div>



<p>and restarted the server.</p>



<p>On trying the query on Presto CLI again, the same error appeared as before:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">presto&gt; show catalogs;
Error running command: javax.net.ssl.SSLHandshakeException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target</pre></div>



<p>This error was super confusing and gave no direct idea of what might have gone wrong. At this point I almost gave up, and I created an issue on&nbsp;<a href="https://github.com/prestodb/presto/issues/27485">Presto GitHub</a>.</p>



<p>While waiting for suggestions on the raised ticket, I started reading further about the&nbsp;<a href="https://prestodb.io/docs/current/security/tls.html#java-truststore-file-for-tls">Java Truststore File for TLS</a>&nbsp;in the Presto documentation, and I found out that&nbsp;<em>For the Presto CLI to trust the Presto coordinator, the coordinator’s certificate must be imported to the CLI’s truststore.</em></p>



<p>I followed the documentation in&nbsp;<a href="https://www.ibm.com/docs/en/software-hub/5.1.x?topic=administering-importing-self-signed-certificates-truststore">Importing self-signed certificates from a Presto (Java) server to a Java Truststore</a>&nbsp;to import the server&#8217;s certificate to the CLI&#8217;s truststore.</p>



<p>While the Presto server was running, I ran these commands:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % echo QUIT | openssl s_client -showcerts -connect localhost:8443 | awk '/-----BEGIN CERTIFICATE-----/ {p=1}; p; /-----END CERTIFICATE-----/ {p=0}' &gt; presto.cert
Connecting to ::1
Can't use SSL_get_servername
depth=0 C=Unknown, ST=Unknown, L=Unknown, O=Unknown, OU=Unknown, CN=Pratyakshs-MacBook-Pro.local
verify error:num=18:self-signed certificate
verify return:1
depth=0 C=Unknown, ST=Unknown, L=Unknown, O=Unknown, OU=Unknown, CN=Pratyakshs-MacBook-Pro.local
verify return:1
DONE</pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % keytool -import -alias presto-cert -file ./presto.cert -keystore ./presto-truststore.jks
Enter keystore password:  
Re-enter new password: 
Owner: CN=Pratyakshs-MacBook-Pro.local, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
Issuer: CN=Pratyakshs-MacBook-Pro.local, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
Serial number: cfbc633
Valid from: Wed Apr 01 21:19:34 IST 2026 until: Tue Jun 30 21:19:34 IST 2026
Certificate fingerprints:
	 SHA1: D8:C6:0D:DB:15:FC:EA:0E:C4:03:B3:B7:5F:3A:AB:42:A5:2A:A5:D3
	 SHA256: DA:AD:9F:03:6A:A1:6E:8B:FC:0C:EC:C1:C8:5E:23:07:8D:06:38:D7:48:75:F3:7F:92:D9:86:46:72:33:5D:AA
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

Extensions: 

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 1B 94 02 F7 C4 0B 85 AC   55 DE 9E 5C E2 00 43 0A  ........U..\..C.
0010: 3B ED A6 29                                        ;..)
]
]

Trust this certificate? [no]:  yes
Certificate was added to keystore</pre></div>



<p>Next I started the Presto CLI and included the truststore related flags:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto % presto-cli/target/presto-cli-0.297-SNAPSHOT-executable.jar --server https://localhost:8443 --truststore-path /Users/pratyakshsharma/Documents/presto-https/presto-truststore.jks --truststore-password password --user test --password
Password:
presto&gt;</pre></div>



<p>However, on running&nbsp;<code class="">show catalogs</code>, I hit another error:</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">presto&gt; show catalogs;
Error running command: javax.net.ssl.SSLPeerUnverifiedException: Hostname localhost not verified:
    certificate: sha256/7HpqT7WDjK7UF+pwa/snkEhUEXWp8WmmJwPp7nWnhyc=
    DN: CN=Pratyakshs-MacBook-Pro.local, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
    subjectAltNames: []
presto&gt; 
presto&gt;</pre></div>



<p>This error was more obvious than the previously encountered ones: the certificate included in the truststore had to have&nbsp;<code class="">localhost</code>&nbsp;as the CN.</p>



<p>I repeated the steps already performed earlier, but using&nbsp;<code class="">localhost</code>&nbsp;as the CN this time.</p>



<ol class="wp-block-list">
<li>Generate self-signed certificate for Presto coordinator</li>
</ol>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % keytool -genkeypair -alias presto -keyalg RSA -keystore keystore_localhost.jks
Enter keystore password:  
Re-enter new password: 
What is your first and last name?
  [Unknown]:  localhost
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  
What is the name of your State or Province?
  [Unknown]:  
What is the two-letter country code for this unit?
  [Unknown]:  
Is CN=localhost, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Enter key password for &lt;presto&gt;
	(RETURN if same as keystore password):  

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore keystore_localhost.jks -destkeystore keystore_localhost.jks -deststoretype pkcs12".</pre></div>



<ol class="wp-block-list" start="2">
<li>Update the property in <code class="">config.properties</code>:</li>
</ol>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">http-server.https.keystore.path=/Users/pratyakshsharma/Documents/presto-https/keystore_localhost.jks</pre></div>



<p>and restarted the server.</p>



<ol class="wp-block-list" start="3">
<li>Import the keystore certificate to the CLI&#8217;s truststore using the following commands:</li>
</ol>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % echo QUIT | openssl s_client -showcerts -connect localhost:8443 | awk '/-----BEGIN CERTIFICATE-----/ {p=1}; p; /-----END CERTIFICATE-----/ {p=0}' &gt; presto1.cert
Connecting to ::1
Can't use SSL_get_servername
depth=0 C=Unknown, ST=Unknown, L=Unknown, O=Unknown, OU=Unknown, CN=localhost
verify error:num=18:self-signed certificate
verify return:1
depth=0 C=Unknown, ST=Unknown, L=Unknown, O=Unknown, OU=Unknown, CN=localhost
verify return:1
DONE</pre></div>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto-https % keytool -import -alias presto-cert -file ./presto1.cert -keystore ./presto1-truststore.jks
Enter keystore password:  
Re-enter new password: 
Owner: CN=localhost, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
Issuer: CN=localhost, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
Serial number: 3aff7a41
Valid from: Wed Apr 01 23:37:53 IST 2026 until: Tue Jun 30 23:37:53 IST 2026
Certificate fingerprints:
	 SHA1: 77:C6:D5:EE:49:44:BE:2F:D8:B8:5C:A4:7A:5B:91:54:7A:08:73:97
	 SHA256: 9D:89:93:EB:E9:C7:6E:98:37:27:F0:1B:54:41:38:C1:AB:66:63:59:61:D2:3B:26:E4:49:92:13:75:53:8C:4F
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

Extensions: 

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 93 07 D0 04 B7 02 73 B9   E0 9B F7 8F 06 0A C4 85  ......s.........
0010: A7 43 20 A2                                        .C .
]
]

Trust this certificate? [no]:  yes
Certificate was added to keystore</pre></div>



<ol class="wp-block-list" start="4">
<li>Restart the Presto CLI and run <code class="">show catalogs</code> query, everything worked like a charm:</li>
</ol>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto % presto-cli/target/presto-cli-0.297-SNAPSHOT-executable.jar --server https://localhost:8443 --truststore-path /Users/pratyakshsharma/Documents/presto-https/presto1-truststore.jks --truststore-password password --user test --password
Password: 
presto&gt; show catalogs;
   Catalog   
-------------
 blackhole   
 delta       
 druid       
 example     
 hana        
 hive        
 hudi        
 iceberg     
 jmx         
 localfile   
 memory      
 mysql       
 pinot       
 postgresql  
 prometheus  
 singlestore 
 sqlserver   

Query 20260401_181326_00000_3qt9p, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 0:03, server-side: 0:02] [0 rows, 0B] [0 rows/s, 0B/s]

presto&gt; exit;</pre></div>



<p>To verify that any user cannot access my cluster without authentication, I tried the Presto CLI command without providing&nbsp;<code class="">--user</code>&nbsp;and&nbsp;<code class="">--password</code>&nbsp;flags. As expected, it did not allow me to run any query.</p>



<div class="cbc-code-wrapper"><div class="cbc-code-bar"><button class="cbc-copy-button">Copy</button></div><pre class="wp-block-preformatted">pratyakshsharma@Pratyakshs-MacBook-Pro presto % presto-cli/target/presto-cli-0.297-SNAPSHOT-executable.jar --server https://localhost:8443 --truststore-path /Users/pratyakshsharma/Documents/presto-https/presto1-truststore.jks --truststore-password password                      
presto&gt; show catalogs;
Error running command: Authentication failed: Unauthorized
presto&gt; exit</pre></div>



<p>Upon reading further, I found this super useful link about&nbsp;<a href="https://www.baeldung.com/java-ssl-handshake-failures">SSL Handshake Failures</a>&nbsp;that talks about the different errors which are encountered when setting up SSL connections. I encourage everyone to give it a read.</p>



<h2 class="wp-block-heading">Follow Us </h2>



<ul class="wp-block-social-links is-layout-flex wp-block-social-links-is-layout-flex"><li class="wp-social-link wp-social-link-linkedin  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.linkedin.com/company/presto-foundation/"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M19.7,3H4.3C3.582,3,3,3.582,3,4.3v15.4C3,20.418,3.582,21,4.3,21h15.4c0.718,0,1.3-0.582,1.3-1.3V4.3 C21,3.582,20.418,3,19.7,3z M8.339,18.338H5.667v-8.59h2.672V18.338z M7.004,8.574c-0.857,0-1.549-0.694-1.549-1.548 c0-0.855,0.691-1.548,1.549-1.548c0.854,0,1.547,0.694,1.547,1.548C8.551,7.881,7.858,8.574,7.004,8.574z M18.339,18.338h-2.669 v-4.177c0-0.996-0.017-2.278-1.387-2.278c-1.389,0-1.601,1.086-1.601,2.206v4.249h-2.667v-8.59h2.559v1.174h0.037 c0.356-0.675,1.227-1.387,2.526-1.387c2.703,0,3.203,1.779,3.203,4.092V18.338z"></path></svg><span class="wp-block-social-link-label screen-reader-text">LinkedIn</span></a></li>

<li class="wp-social-link wp-social-link-github  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://github.com/prestodb/presto"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12,2C6.477,2,2,6.477,2,12c0,4.419,2.865,8.166,6.839,9.489c0.5,0.09,0.682-0.218,0.682-0.484 c0-0.236-0.009-0.866-0.014-1.699c-2.782,0.602-3.369-1.34-3.369-1.34c-0.455-1.157-1.11-1.465-1.11-1.465 c-0.909-0.62,0.069-0.608,0.069-0.608c1.004,0.071,1.532,1.03,1.532,1.03c0.891,1.529,2.341,1.089,2.91,0.833 c0.091-0.647,0.349-1.086,0.635-1.337c-2.22-0.251-4.555-1.111-4.555-4.943c0-1.091,0.39-1.984,1.03-2.682 C6.546,8.54,6.202,7.524,6.746,6.148c0,0,0.84-0.269,2.75,1.025C10.295,6.95,11.15,6.84,12,6.836 c0.85,0.004,1.705,0.114,2.504,0.336c1.909-1.294,2.748-1.025,2.748-1.025c0.546,1.376,0.202,2.394,0.1,2.646 c0.64,0.699,1.026,1.591,1.026,2.682c0,3.841-2.337,4.687-4.565,4.935c0.359,0.307,0.679,0.917,0.679,1.852 c0,1.335-0.012,2.415-0.012,2.741c0,0.269,0.18,0.579,0.688,0.481C19.138,20.161,22,16.416,22,12C22,6.477,17.523,2,12,2z"></path></svg><span class="wp-block-social-link-label screen-reader-text">GitHub</span></a></li>

<li class="wp-social-link wp-social-link-youtube  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://www.youtube.com/@PrestoFoundation"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21.8,8.001c0,0-0.195-1.378-0.795-1.985c-0.76-0.797-1.613-0.801-2.004-0.847c-2.799-0.202-6.997-0.202-6.997-0.202 h-0.009c0,0-4.198,0-6.997,0.202C4.608,5.216,3.756,5.22,2.995,6.016C2.395,6.623,2.2,8.001,2.2,8.001S2,9.62,2,11.238v1.517 c0,1.618,0.2,3.237,0.2,3.237s0.195,1.378,0.795,1.985c0.761,0.797,1.76,0.771,2.205,0.855c1.6,0.153,6.8,0.201,6.8,0.201 s4.203-0.006,7.001-0.209c0.391-0.047,1.243-0.051,2.004-0.847c0.6-0.607,0.795-1.985,0.795-1.985s0.2-1.618,0.2-3.237v-1.517 C22,9.62,21.8,8.001,21.8,8.001z M9.935,14.594l-0.001-5.62l5.404,2.82L9.935,14.594z"></path></svg><span class="wp-block-social-link-label screen-reader-text">YouTube</span></a></li>

<li class="wp-social-link wp-social-link-x  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://x.com/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M13.982 10.622 20.54 3h-1.554l-5.693 6.618L8.745 3H3.5l6.876 10.007L3.5 21h1.554l6.012-6.989L15.868 21h5.245l-7.131-10.378Zm-2.128 2.474-.697-.997-5.543-7.93H8l4.474 6.4.697.996 5.815 8.318h-2.387l-4.745-6.787Z"></svg><span class="wp-block-social-link-label screen-reader-text">X</span></a></li>

<li class="wp-social-link wp-social-link-chain  wp-block-social-link"><a class="wp-block-social-link-anchor" href="https://communityinviter.com/apps/prestodb/prestodb"><svg height="24" version="1.1" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M15.6,7.2H14v1.5h1.6c2,0,3.7,1.7,3.7,3.7s-1.7,3.7-3.7,3.7H14v1.5h1.6c2.8,0,5.2-2.3,5.2-5.2,0-2.9-2.3-5.2-5.2-5.2zM4.7,12.4c0-2,1.7-3.7,3.7-3.7H10V7.2H8.4c-2.9,0-5.2,2.3-5.2,5.2,0,2.9,2.3,5.2,5.2,5.2H10v-1.5H8.4c-2,0-3.7-1.7-3.7-3.7zm4.6.9h5.3v-1.5H9.3v1.5z"></path></svg><span class="wp-block-social-link-label screen-reader-text">Link</span></a></li></ul>
<p>The post <a href="https://prestodb.io/blog/2026/04/30/password-authentication-setup-on-local/">Password Authentication Setup on Local</a> appeared first on <a href="https://prestodb.io">PrestoDB</a>.</p>
