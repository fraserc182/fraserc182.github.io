---
layout: post
title: Using input from csv to run commands against network devices
date: 2021-02-21 15:53
author: Fraser Clark
comments: true
tags: [python, network automation]
---
<!-- wp:paragraph -->
<p>This is a script I use in a variety of ways. It's good to have it available when you need to run commands against a lot of devices.<br>It's a simple concept, write what you want in the csv file and then the script loops through it executing those commands on the switches concurrently.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The csv file looks like below:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
{% highlight %}
<pre class="wp-block-code"><code>hostname,interface,vlan,description
sw1,Gi1/0/1,45,**Office**
sw1,Gi1/0/2,43,**EDI-IPTV012**
sw3,Gi1/0/10,45,**Dev Team**
sw6,Gi1/0/17,45,**Dev Team**
sw7,Gi1/0/18,45,**Dev Team**
sw12,Gi1/0/22,45,**Dev Team**
sw12,Gi1/0/23,45,**Dev Team**
sw12,Gi1/0/24,45,**Dev Team**</code></pre>
{% endhighlight %}
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The python script as is as follows.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
{% highlight python %}
<pre class="wp-block-code"><code>#!/usr/bin/env python

'''
Takes input from the ports.csv file and will update the ports based on what is in the csv
'''

from multiprocessing import Process, Queue
from datetime import datetime
from netmiko import ConnectHandler
from ntc_templates.parse import parse_output
import csv
from getpass import getpass

ssh_pass = getpass("Enter ssh password: ")


def show_cisco_commands(a_device, output_q, intf, vlan, description):
    """
    Use Netmiko to execute commands. Use a queue to pass the data back to
    the main process.
    """
    config_commands = (&#091; 'interface '+ intf, 
                    'switchport access vlan '+ vlan, 
                    'description '+ description ])
 
    output_dict = {}
    
    remote_conn = ConnectHandler(**a_device)
    print(remote_conn.base_prompt)
    print(remote_conn.send_config_set(config_commands))
    output_q.put(output_dict)



def main():
    start_time = datetime.now()
    output_q = Queue(maxsize=20)

    procs = &#091;]

    with open('ports.csv', mode='r') as csv_file:
        csv_reader = csv.DictReader(csv_file)
        line_count = 0
        for row in csv_reader:
            if line_count == 0:
                ", ".join(row)
                line_count += 1
            line_count += 1
            '''
            In this section you can specify any different values from the csv file
            you may want to add in. Perhaps one for enabled/disabled.
            '''
            hostname = row&#091;"hostname"]
            port = row&#091;"interface"]
            vlan = row&#091;"vlan"]
            desc = row&#091;"description"]
            cisco = {
                'device_type': 'cisco_ios',
                'host': hostname,
                'username': 'USERNAME',
                'password': ssh_pass,
                }
            my_proc = Process(target=show_cisco_commands, args=(cisco, output_q, port, vlan, desc))
            my_proc.start()
            procs.append(my_proc)

    # Make sure all processes have finished
    for a_proc in procs:
        a_proc.join()


    print("\nElapsed time: " + str(datetime.now() - start_time))


if __name__ == "__main__":
    main()</code></pre>
<!-- /wp:code -->
{% endhighlight %}

<!-- wp:paragraph -->
<p>Running it is fairly straightforward and just needs credentials for the devices you are connecting to.<br>It will output some pretty ugly lines saying what is being connected and what commands are being run. This can be tidied up by modifying these lines:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
{% highlight python %}
<pre class="wp-block-code"><code>    print(remote_conn.base_prompt)
    print(remote_conn.send_config_set(config_commands))</code></pre>
<!-- /wp:code -->
{% endhighlight %}
<!-- wp:paragraph -->
<p>Anyway, that's all there is to it. This can be adapted into a multitude of uses.</p>
<!-- /wp:paragraph -->
