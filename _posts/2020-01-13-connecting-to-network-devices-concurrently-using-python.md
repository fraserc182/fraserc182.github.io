---
layout: post
title: Connecting to network devices concurrently using Python
date: 2020-01-13 15:32
author: Fraser Clark
comments: true
tags: [python, network automation]
---
<!-- wp:paragraph -->
<p>There was one issue with my previous </a>about using python to find interfaces on a certain vlan and this was the length of time it took to actually complete the script.<br>From start to finish the script took 3 minutes and 20 seconds, which is still significantly better than doing it manually but it can be improved.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The reason it took so long was that it was connecting to each device in the list one at a time, doing the tasks and then moving on to the next device.<br>I found a blog post by Kirk Byers (creator of netmiko) which pointed towards his <a href="https://github.com/ktbyers/netmiko/tree/develop/examples/use_cases/case16_concurrency">GitHub </a>page and this had examples of using processes to connect to devices concurrently.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The code is below and it has been modified significantly to what is in Kirk's GitHub page as I used that as a jumping off point.<br>This script now takes about 6 seconds to run through the same amount of devices.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>#!/usr/bin/env python
"""
Use processes and Netmiko to connect to each of the devices. Execute
'show version' on each device. Use a queue to pass the output back to the parent process.
Record the amount of time required to do this.
"""
from multiprocessing import Process, Queue

from datetime import datetime
from netmiko import ConnectHandler
from ntc_templates.parse import parse_output
from termcolor import colored, cprint
import sys
import time


## Defines a menu so the user can choose which devices to work on

def ciscoMenu():
    print("Please choose which device group to work on: ")
    print("")
    print ("1. Edge Switches")
    print ("2. Core Switches")
    print ("3. Internet Switches")
    print ("4. Out of Band Switches")
    print ("5. Server Switches")
    print ("6. Storage Switches")
    print ("7. Lab Switches")
    print ("8. Exit Program")

## This while loop displays the menu and when the user chooses an option
##  it will import the required section of the my_devices.py file

while True:
    ciscoMenu()
    menuchoice = input("Enter your choice &#091;1-8]: ")
    if menuchoice == '1':
        from my_devices import edge_switches as devices
        break
    elif menuchoice == '2':
        from my_devices import core_switches as devices
        break
    elif menuchoice == '3':
        from my_devices import internet_switches as devices
        break
    elif menuchoice == '4':
        from my_devices import oob_switches as devices   
        break
    elif menuchoice == '5':
        from my_devices import server_switches as devices
        break
    elif menuchoice == '6':
        from my_devices import storage_switches as devices
        break
    elif menuchoice == '7':
        from my_devices import lab_switches as devices
        break
    elif menuchoice == '8':
        sys.exit("Goodbye!")
    else:
        input("\nInvalid option. Press Enter to try again..")


def show_cisco_commands(a_device, output_q):
    """
    Use Netmiko to execute commands. Use a queue to pass the data back to
    the main process.
    """
    output_dict = {}
    remote_conn = ConnectHandler(**a_device)
    hostname = remote_conn.base_prompt
    sh_commands = remote_conn.send_command("show int status")
    sh_commands_parsed = parse_output(platform="cisco_ios", command="show int status", data=sh_commands)
    cprint(("=" * 80) + "\n", 'red', attrs=&#091;'bold'])
    cprint((hostname), 'green', attrs=&#091;'bold'])
    print("")
    for i in sh_commands_parsed:
        if i&#091;'vlan'] == '12':
            intf = (i&#091;'port'])
            vlans = (i&#091;'vlan'])
            status = (i&#091;'status'])
            print('Found port '+intf, 'on VLAN ' +vlans, "\n" 'Current Port status is: ' +status, 
            "\n" 'Port will be shutdown')
            print("")

    print("")
    cprint(("=" * 80) + "\n", 'red', attrs=&#091;'bold'])
    output_q.put(output_dict)


def main():
    """
    Use processes and Netmiko to connect to each of the devices. Execute
    'show version' on each device. Use a queue to pass the output back to the parent process.
    Record the amount of time required to do this.
    """
    start_time = datetime.now()
    output_q = Queue(maxsize=35)

    procs = &#091;]
    for a_device in devices:
        my_proc = Process(target=show_cisco_commands, args=(a_device, output_q))
        my_proc.start()
        procs.append(my_proc)

    # Make sure all processes have finished
    for a_proc in procs:
        a_proc.join()


    print("\nElapsed time: " + str(datetime.now() - start_time))


if __name__ == "__main__":
    main()
</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The first thing to know is that I have a file called my_devices.py which acts as an inventory file. It's basic structure is as below and it can be easily expanded for whatever devices you have.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>from getpass import getpass

ssh_pass = getpass("Enter ssh password: ")


## lab switches

lab_sw = {
    "device_type": "cisco_ios",
    "host": "test_lab_switch.blag",
    "username": "admin",
    "password": ssh_pass,
}



lab_switches = &#091;
    lab_sw,
]</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>I am also using cprint to make terminal output look nice, but this can be ignored if you do not want to use cprint.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>On my github page you can find a barebones script which can be modified easily.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The parts that need to be explained in a bit more detail is the show_ciscon_commands function.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>def show_cisco_commands(a_device, output_q):
    """
    Use Netmiko to execute commands. Use a queue to pass the data back to
    the main process.
    """
    output_dict = {}
    remote_conn = ConnectHandler(**a_device)
    hostname = remote_conn.base_prompt
    sh_commands = remote_conn.send_command("show int status")
    sh_commands_parsed = parse_output(platform="cisco_ios", command="show int status", data=sh_commands)
    cprint(("=" * 80) + "\n", 'red', attrs=&#091;'bold'])
    cprint((hostname), 'green', attrs=&#091;'bold'])
    print("")
    for i in sh_commands_parsed:
        if i&#091;'vlan'] == '12':
            intf = (i&#091;'port'])
            vlans = (i&#091;'vlan'])
            status = (i&#091;'status'])
            print('Found port '+intf, 'on VLAN ' +vlans, "\n" 'Current Port status is: ' +status, 
            "\n" 'Port will be shutdown')
            print("")

    print("")
    cprint(("=" * 80) + "\n", 'red', attrs=&#091;'bold'])
    output_q.put(output_dict)
</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>What this does is connects to the device, sends the <em>show int status</em> command  then this output is parsed using ntc templates. ntc templates parses commands into a nice dictionary which can be worked with a lot easier than the normal output.<br>The next part is the for loop, which loops through the <em>sh_commands_parsed </em>output and looks for the values you have specified.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Again there is room for improvement in the script as there always is, but I have been improving all my attempts over the last few months so feeling hopeful about the next project!</p>
<!-- /wp:paragraph -->
