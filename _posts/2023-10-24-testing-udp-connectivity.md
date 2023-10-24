---
layout: post
title: How to test UDP port connectivity
date: 2023-10-24
author: Fraser Clark
comments: true
tags: [linux, bash]
---

I had to configure a new service which listened on a UDP port and would accept data.
This was setup but my question was how to actually test the connectivity of the port and ensure data was acceepted.

To do this I wrote a python script:

```python
import socket

server_address = ('your_server', UDP PORT)  # Replace 'your_server' with the actual server IP address/hostname
# Replace UDP PORT with the actual UDP port number

client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
message = b"Hello, server!"  # Your message goes here

try:
    client_socket.sendto(message, server_address)
    print("Message .")
except Exception as e:
    print(f"Error sending message: {e}")
finally:
    client_socket.close()

```

Now on the remote machine you can run:

```bash
sudo tcpdump -i <interface> udp port <UDP PORT>
```

Ensure `tcpdump` is installed.
Replace `<interface>` with the actual interface name, e.g. `eth0` and `<UDP PORT>` with the actual UDP port number.
Find the interface with `ifconfig` or similar

Run tcpdump and then run the python script, you should see the message appear in the tcpdump output.