---
layout: post
title: Monitoring Windows NPS logs with Splunk
date: 2019-12-18 21:51
author: fraserkr160118
comments: true
categories: [nps, radius, Splunk]
---
<!-- wp:paragraph -->
<p>In our environment we use Windows Network Policy Server (NPS) for our Radius solution. When we were having some issues with authentication to certain switches I realised that the logs were not being monitored by Splunk as I previously thought. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>After some digging, I found NPS can write logs to two places: </p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>SQL Server database</li><li>Flat text files located at C:\System32\LogFiles</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>This will be dealing with the second option, the flat text files.<br>So we need Splunk to do a few things:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Read the text files and index it</li><li>Parse the data into useful information</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>The overview of tasks we need to do to achieve this is firstly to create a new deployment application for our forwarders, then modify the inputs.conf to monitor the log files and finally modify props.conf to parse the data in a useful way. The index named "radius" must also be present on the search head.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Ingesting the data</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>First thing to do is to get Splunk to ingest the data, I will document exactly how I did this and it may be different from the way you choose to do it but hopefully it will be helpful.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>What you want to do is to create a deployment application which can be pushed to our forwarders. If your environment doesn't use a deployment server then this app will need to be created and copied to each universal forwarder.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Firstly create a new application:</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>On your deployment server navigate to <em>$SPLUNK_HOME$/opt/splunk/etc/deployment-apps/</em><br><br>Within this folder create a new folder (the name of which will be the name of your deployment application) and ensure that the splunk user has permissions over the file.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>su splunk # switch user to splunk user
mkdir nps_monitor # create folder 
cd nps_monitor # move to new folder
mkdir local # create local folder
cd local # move to local folder
touch app.conf inputs.conf props.conf # create the files required</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>This app should then be pushed to your NPS servers. Make sure that the splunk user has permission over these files as I have been bitten a few times by this after creating deployment apps with the root user.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>inputs.conf</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The following configuration should go in your inputs.conf folder.<br>The monitor stanza is assuming NPS logs are being logged to the default location and are following the default naming convention.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>&#091;monitor://C:\Windows\System32\LogFiles\IN*.log] #monitors the specified location for any .log files beginning with IN
sourcetype = ias #sets sourcetype to ias
index = radius #sets index to radius
disabled = 0</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3>props.conf</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The props.conf file is where the data shall be extracted and parsed for splunk to create fields etc.<br>I cannot take credit for this, I found it on this answers.splunk.com <a href="https://answers.splunk.com/answers/600737/how-to-parse-radius-log-files-into-splunk-what-the.html">post</a>.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>
&#091;ias] #specify which sourcetype to modify
 SHOULD_LINEMERGE = false
 KV_MODE = NONE
 INDEXED_EXTRACTIONS = CSV # The type of file that Splunk software should expect for a given sourcetype, and the extraction and/or parsing method that should be used on the file.
 # This setting tells Splunk to specify the header field names directly
 FIELD_NAMES = ComputerName,ServiceName,Record_Date,Record_Time,Packet_Type,User_Name,Fully_Qualified_Distinguished_Name,
Called_Station_ID,Calling_Station_ID,Callback_Number,Framed_IP_Address,NAS_Identifier,NAS_IP_Address,NAS_Port,Client_Vendor,Client_IP_Address,
Client_Friendly_Name,Event_Timestamp,Port_Limit,NAS_Port_Type,Connect_Info,Framed_Protocol,Service_Type,Authentication_Type,Policy_Name,Reason_Code,
Class,Session_Timeout,Idle_Timeout,Termination_Action,EAP_Friendly_Name,Acct_Status_Type,Acct_Delay_Time,Acct_Input_Octets,Acct_Output_Octets,Acct_Session_Id,
Acct_Authentic,Acct_Session_Time,Acct_Input_Packets,Acct_Output_Packets,Acct_Terminate_Cause,Acct_Multi_Ssn_ID,Acct_Link_Count,Acct_Interim_Interval,
Tunnel_Type,Tunnel_Medium_Type,Tunnel_Client_Endpt,Tunnel_Server_Endpt,Acct_Tunnel_Conn,Tunnel_Pvt_Group_ID,Tunnel_Assignment_ID,Tunnel_Preference,
MS_Acct_Auth_Type,MS_Acct_EAP_Type,MS_RAS_Version,MS_RAS_Vendor,MS_CHAP_Error,MS_CHAP_Domain,MS_MPPE_Encryption_Types,MS_MPPE_Encryption_Policy,
Proxy_Policy_Name,Provider_Type,Provider_Name,Remote_Server_Address,MS_RAS_Client_Name,MS_RAS_Client_Version
 TIME_FORMAT = %m/%d/%Y%n%H:%M:%S
 MAX_TIMESTAMP_LOOKAHEAD = 20
 TIMESTAMP_FIELDS = Record_Date,Record_Time
 DATETIME_CONFIG =
 NO_BINARY_CHECK = true
 disabled = false
 pulldown_type = true</code></pre>
<!-- /wp:code -->

<!-- wp:heading -->
<h2>The end result</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>If all is configured correctly you should be able to see data in the "radius" index and if done correctly you go from events like this:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":112,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://fraserclark926577729.files.wordpress.com/2019/12/image.png?w=613" alt="" class="wp-image-112" /></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>To events like below:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":114,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://fraserclark926577729.files.wordpress.com/2019/12/image-1.png?w=784" alt="" class="wp-image-114" /></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Hope this helps someone!</p>
<!-- /wp:paragraph -->
