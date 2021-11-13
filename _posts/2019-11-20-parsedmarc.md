---
layout: post
title: Parsedmarc
date: 2019-11-20 23:42
author: fraserkr160118
comments: true
categories: [archive]
---
<!-- wp:paragraph -->
<p>Parsedmarc is an open source linux tool that can read DMARC reports, parse the results into an RFC compliant format and then output it as JSON.<br>It is an excellent tool and also integrates with Splunk very nicely which is why I thought to write about it here.<br>The parsedmarc documentation can be found here:  <a href="https://domainaware.github.io/parsedmarc/">https://domainaware.github.io/parsedmarc/</a> <br>The documentation is fairly robust but I did have some troubles initially when setting it up.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Requirements</h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>Linux machine<ul><li>You can even run this on the Windows Sub-system for linux on Windows 10 if you wish</li></ul></li><li>Splunk<ul><li>Doesn't matter if it is enterprise or cloud and can also be a free license (&lt;500MB a day)</li></ul></li><li>A DMARC record on your domain<ul><li>With an rua &amp; ruf section which send to a mailbox you can access/control</li></ul></li></ul>
<!-- /wp:list -->

<!-- wp:heading -->
<h2>Installation</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Installing parsedmarc is fairly simple but one thing to bare in mind is that it only works with python3.<br>The installation instructions should be followed from the parsedmarc documentation at the top of the page</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Configuration</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Once installed, the parsedmarc.ini file must be configured to be able to run. This file can be stored anywhere on the machine but the default location is the /etc/ folder. Below is a sanitised version of my parsedmarc.ini file.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>&#091;general]
save_aggregate = True
save_forensic = True
nameservers = # your ntp servers
 
&#091;imap]
host = # your mail server
user = # your user, that has rights to the inbox where your reports are being sent
password = Enter_Password
skip_certificate_verification = True # This is required as parsedmarc will not run otherwise.
reports_folder = INBOX
 
&#091;splunk_hec] # Splunk settings, no need to edit these
url = https://http-inputs-INSTANCE.com:443/services/collector
token = # token
index = email</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>A few notes about the above after some trial and error I have went through.</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>nameservers<ul><li>The parsedmarc documentation strongly recommends you to use the default cloudflare resolvers. However if you are in an enterprise environment with DNS resolvers, then you will definitely want to use those instead.</li></ul></li><li>skip_certificate_validation<ul><li>I eventually had to use this setting or else I would get intermittent errors about certificates</li></ul></li><li>splunk section<ul><li>This only applies if you have a splunk instance you are sending the output to but it is fairly self explanatory.</li><li>You simply need to create an HTTP Event Collector on your instance, configure the settings like the above and you should be good to go</li></ul></li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>After installing then configuring you are ready to test.<br>To run parsedmarc, you just run the parsedmarc and supply the location to the .ini file.<br>I like to run the file with --debug when running it manually as it shows you progress and can potentially show you problematic DMARC reports it is hanging on.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>parsedmarc -c /etc/parsedmarc.ini --debug</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>When you have verified it runs correctly, you can either set it to run as a service or you can create a cronjb to run the command every couple of hours.<br>Currently I simply run it as a cronjob as it is nice and easy to setup then troubleshoot if necessary.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Conclusion</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Parsedmarc is an excellent tool that not many free tools can provide and the logging capability through Kibana or Splunk makes it even better.<br>I highly recommend giving it a shot and if you run into issues I have found the creator to be very responsive and helpful on the github repo.</p>
<!-- /wp:paragraph -->
