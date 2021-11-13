---
layout: post
title: Configuring a GNS3 Lab for network automation
date: 2019-11-11 07:41
author: fraserkr160118
comments: true
categories: [ansible, GNS3, Lab, Network Automation]
---
<!-- wp:paragraph -->
<p>Having a lab environment when learning network automation is essential, personally I have used GNS3. <br>I found it challenging to set up in the first place as it was my first real dive into GNS3 and I spent a lot of time trying different things and googling the myriad of issues I seemed to find.<br>So I'm documenting what I did here, mainly for myself but maybe for others who are setting up the same thing.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Overview</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>There are a few things you'll need in order to get this working which are listed below.<br>To download the VMWare product you will need a VMWare account which can be created here:  <a href="https://my.vmware.com/">https://my.vmware.com/</a> </p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>VMWare Player (I am on version 15)<ul><li>I tried to get this working with virtualbox as that is what I am used to but it was far too difficult in the end. Almost all of the guides out there will tell you to download VMWare Player</li></ul></li><li>GNS3 VM - installed on VMWare Player<ul><li>Now this isn't a VM running an OS with GNS3 installed, this is the GNS3 VM. This is a server that your GNS3 installation will connect to and spawn resources in</li></ul></li><li>Linux VM - installed on VMWare Player<ul><li>I am using Ubuntu but any linux OS is fine</li><li>This is where I run Ansible/scripts from</li></ul></li><li>GNS3 installed on host machine</li><li> VMWare VIXRequired to get past a few error messages that you may encounter </li><li>VMWare Workstation Pro<ul><li>Obviously this is a paid service but it can be used on a trial license for 30 days and that is long enough to give you what you need. The reason for this is that only VMWare Workstation Pro will give you access to the Virtual Network Editor.</li><li>Ensure you create a custom network with whatever subnet you like using the Virtual Network Editor and make sure the Type is set to NAT to give internet connectivity, mine looks like this (VMnet8):</li></ul></li></ul>
<!-- /wp:list -->

<!-- wp:image {"id":82,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://fraserclark926577729.files.wordpress.com/2019/11/image-1.png?w=599" alt="" class="wp-image-82" /><figcaption>Virtual Network Editor</figcaption></figure>
<!-- /wp:image -->

<!-- wp:heading -->
<h2>Virtual Machines</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>First thing to do is to sort out the virtual machines. So ensure you have VMWare Player, VMWare VIX and optionally VMWare Workstation Pro all installed.<br>Install the GNS3 VM with all the default options, there are lots of guides out there how to install GNS3 VM if required. <br>After installing, you should have a screen similar to the below.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":77,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://fraserclark926577729.files.wordpress.com/2019/11/image.png?w=1000" alt="" class="wp-image-77" /><figcaption>GNS3 VM Information</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>If "KVM support available: False", you first need to ensure that virtualisation is enabled at the BIOS and then also enable it in VMWare settings. Virtual Machine Settings &gt; Processors &gt; Virtualize Intel VT - x/EPT or AMD - V/RVI<br>Ensure that the VM has two NICs, one attached to host only network and one attached to the custom network that was created earlier.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>After the GNS3 VM and GNS3 have been installed next install the linux VM. Again I used Ubuntu but you can use any linux distro you are comfortable with.<br>The network setup I went with is to give the VM two NICs, have the first connected to Host Only network and the second NIC connected to your custom network.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>GNS3 Installation</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p> After the GNS3 VM is configured the next thing to ensure is your GNS3 installation is using the GNS3 VM as it's server.<br>Install GNS3 on your host machine and when you run it for the first time it should automatically detect the GNS3 VM running and use this as it's server. If this is not the case then you can follow these steps to ensure it can connect:  <a href="https://docs.gns3.com/1hEoK0rmtdBRnMaUaVoMHUbYwtDAltXYShiMJUp1GMxk/index.html#h.1ykeiohp7oul">https://docs.gns3.com/1hEoK0rmtdBRnMaUaVoMHUbYwtDAltXYShiMJUp1GMxk/index.html#h.1ykeiohp7oul</a><br><br>Each time you go to launch GNS3, make sure the GNS3VM is running first.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Connectivity</h2>
<!-- /wp:heading -->

<!-- wp:image {"id":85,"width":366,"height":362,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large is-resized"><img src="https://fraserclark926577729.files.wordpress.com/2019/11/image-2.png?w=304" alt="" class="wp-image-85" width="366" height="362" /><figcaption>Cloud connected to Router1</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Getting connectivity between the VM's was the hardest part for me, I just couldn't seem to get it to work and the way I now have it configured may not be the best way but it works. I think I'll come back eventually to simplify it.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So to get connectivity between GNS3 machines &amp; your Linux VM, first of all make sure your Linux VM has a network connection on the custom NAT network that was configured earlier. For example, mine is on the 192.168.19.0/24 subnet.<br>Now in GNS3, create a "Cloud" object and double click on it to edit it.<br>You should see network interfaces in the box, eth0 &amp; eth1. If you don't you simply need to add it in, so select the missing interface and click "Add". If it does not show up in the drop-down box then click "Show special interfaces" at the bottom left.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":86,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://fraserclark926577729.files.wordpress.com/2019/11/image-3.png?w=664" alt="" class="wp-image-86" /><figcaption>Cloud configuration screen</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p><br>Each network interface in the cloud object lines up with the network interfaces assigned to the GNS3 VM, so the goal here is to have a router/device connected to the interface that is on the custom network and this device will act as your gateway. <br>In the GNS3 VM the second NIC is connected to the custom network, then this is the eth1 interface in the cloud object. It is like the below table:<br>NIC1 GNS3 VM &gt;&gt;&gt; eth0 cloud object<br>NIC2 GNS3 VM &gt;&gt;&gt; eth1 cloud object</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>When the cloud object is created and the final step is to give your device connected to the cloud object an address on the custom network subnet. When this is done, you should be able to ping between your linux VM and devices in GNS3.</p>
<!-- /wp:paragraph -->
