---
layout: post
title: Using Python to find interfaces on a certain VLAN
date: 2019-12-18 22:57
author: Fraser Clark
comments: true
tags: [python, network automation]
---
<!-- wp:paragraph -->
<p>This topic was the first thing I ever wanted to get out of automation and is what kick-started my learning.<br>And again, this is all available on my github.<br>I had a simple use case which went like so:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Iterate through a list of switches</li><li>Run the <em>show interface status</em> command</li><li>Check if a certain vlan existed in that output</li><li>Print it if that vlan was found</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>Even though it seemed somewhat simple, I struggled with it and only now have managed to get this sorted. And in the end it was very simple!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The way I initially managed to do it was using <a href="https://github.com/networktocode/pyntc">pynetc</a> and <a href="https://github.com/networktocode/ntc-templates">ntc-templates</a> to parse the output and make it easier to work with.<br>However after doing some testing I found using pynetc was not the fastest way to do this and instead switched to using netmiko.<br>For comparison, the run times for the below code were:<br> pynetc - 17 seconds<br>netmiko - 9 seconds<br>So when running over 40+ devices, this makes a big difference.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
{% highlight python %}
<pre class="wp-block-code"><code>
## import required modules
from netmiko import ConnectHandler
from ntc_templates.parse import parse_output
from getpass import getpass

## read the hosts text file
switches = open("./hosts_files/edge_switches.txt").read().splitlines()

## set user and ask for ssh password
ssh_user = "USERNAME"
ssh_pass = getpass()

## first loops through switches and runs show int status and gathers facts
for x in switches:
    switch = ConnectHandler(device_type='cisco_ios', host=(x), username=(ssh_user), password=(ssh_pass))
    show_int = switch.send_command("show int status")
    ## parses output so it can be worked with easier
    show_int_parsed = parse_output(platform="cisco_ios", command="show int status", data=show_int)
    ## pull hostname from running config
    sw_hostname = switch.send_command("show run | in hostname")
    sw_hostname = sw_hostname.split()
    hostname = sw_hostname&#091;1]
    print(hostname)
    print("")
    
    ## loops through parsed interface output and checks for interfaces on vlan 12
    for i in show_int_parsed:
        if i&#091;'vlan'] == '12':
            intf = (i&#091;'port'])
            vlans = (i&#091;'vlan'])
            status = (i&#091;'status'])
            print('Found port '+intf, 'on VLAN ' +vlans, '&amp; Port is: ' +status)</code></pre>
<!-- /wp:code -->
{% endhighlight %}
<!-- wp:paragraph -->
<p>I am used to working with hosts files in Ansible, so to keep things similar I created a hosts.txt filevand bring this into the script as a list. <br>This list is then iterated through. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I think there will be a better way to do this, but this works for me and it gives me the output I was looking for!</p>
<!-- /wp:paragraph -->
