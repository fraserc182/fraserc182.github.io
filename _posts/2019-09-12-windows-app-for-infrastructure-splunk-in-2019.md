---
layout: post
title: Windows App for Infrastructure & Splunk in 2019
date: 2019-09-12 20:43
author: fraserkr160118
comments: true
categories: [Active Directory, Logging, Logging and monitoring, Monitoring, Splunk, Splunk Cloud]
---
<!-- wp:paragraph -->
<p>When configuring the Windows App for Infrastructure I ran into a lot of problems which caused me a whole lot of grief and a lot of wasted time. The problems being that documentation is confusing, contradictory and often out of date. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So there is some context, I will explain our infrastructure.<br>We have a managed instance of Splunk Cloud and an on premise heavy forwarder configured as a deployment server.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>What I was trying to achieve</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>I was already getting Windows security event logs from our domain controllers and some of our Windows servers, however I wanted to get active directory information &amp; statistics as well. This is where I ran into a whole load of issues and conflicting information.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>A caveat</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p> The one caveat I ran into is because of our usage of Splunk Cloud.<br>The Windows App for Infrastructure uses some of the searches from the Splunk Add-on for Active Directory, primarily the ldapsearch command. <br><br>So for those searches to work you must configure the  Splunk Add-on for Active Directory, however this cannot be done on Splunk cloud (unless you want to open port 389/636 directly to splunk cloud). So that means some of the dashboards will not work.  <br></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>To help myself and anyone else in the future I am going to document how I got this working in the end.<br>However, this guide assumes the following:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>You have understanding of how to get data to Splunk</li><li>Understanding of the file structure of Splunk</li><li> General splunk jargon </li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>The basic steps are as follows:</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li>Install Splunk Add-On for Windows  where required (basically everywhere).</li><li>Configure inputs.conf</li><li>Enjoy your data</li></ol>
<!-- /wp:list -->

<!-- wp:heading -->
<h2>Install Splunk Add-On for Windows</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Follow Splunk's official documentation and install the splunk add-on for windows in the correct areas:  <a href="https://docs.splunk.com/Documentation/WindowsAddOn/6.0.0/User/Install">https://docs.splunk.com/Documentation/WindowsAddOn/6.0.0/User/Install</a> <br>For my installation I installed it in the following places:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Search head &amp; indexer</li><li>Heavy Forwarder/Deployment server</li><li>Universal Forwarders</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>To deploy to the universal forwarders I created a new server class on my deployment server for domain controllers and put the app in the server class.<br>However, if your splunk deployment does not include a deployment server then just install the add-on manually to the domain controllers.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Configure inputs.conf</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>After the add-on has been installed where required, you now need to configure the inputs.conf file of the add-on. This should be on the servers/DC's you have the add-on installed to.<br><br>If doing this with a deployment server, edit the inputs.conf file under: %SPLUNK_HOME%\etc\deployment-apps\Splunk_TA_windows\local\inputs.conf<br><br>If you do not want active directory inputs then simply remove everything below "Active Directory Inputs".<br>Below is what my inputs.conf file looks like:</p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>[WinEventLog://Security]<br> index = wineventlog<br> disabled = 0<br> start_from = -7d<br> renderXml=false<br><br> #Active Directory Inputs<br><br> [WinEventLog://DFS Replication]<br> index = msad<br> disabled = 0<br> renderXml=false<br><br> [WinEventLog://File Replication Service]<br> index = msad<br> disabled = 0<br> renderXml=false<br><br> [WinEventLog://Key Management Service]<br> index = msad<br> disabled = 0<br> renderXml=false<br><br> [admon://default]<br> monitorSubtree = 1<br> baseline = 1<br> index = msad<br> disabled = 0<br> renderXml=false<br><br> [monitor://$WINDIR\debug\netlogon.log]<br> index = msad<br> disabled = 0<br> renderXml=false<br><br> [script://.\bin\runpowershell.cmd nt6-repl-stat.ps1]<br> source=Powershell<br> sourcetype=MSAD:NT6:Replication<br> interval=3600<br> index = msad<br> disabled = 0<br> renderXml=false<br><br> [script://.\bin\runpowershell.cmd nt6-health.ps1]<br> source=Powershell<br> sourcetype=MSAD:NT6:Health<br> interval=3600<br> index = msad<br> disabled = 0<br> renderXml=false<br><br> [script://.\bin\runpowershell.cmd nt6-siteinfo.ps1]<br> source=Powershell<br> sourcetype=MSAD:NT6:SiteInfo<br> interval=3600<br> index = msad<br> disabled = 0<br> renderXml=false</p></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>I highly recommend keeping the <em>renderXml=false </em>stanza set to false. Have a look at <a href="https://old.reddit.com/r/Splunk/comments/co11z4/windows_security_logs_ingested_as_xml_missing_data/">this</a> reddit thread for more info on why not to ingest XML Windows event logs.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->
