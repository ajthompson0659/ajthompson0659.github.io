---
layout: post
title:  "How to create a bare-metal switch (BMS) on Ubuntu 14.04"
date:   2015-01-18 20:47:01
categories: How to
---

A bare-metal switch (BMS) as I refer to it, is a computer platform that is configured to be a switch.  I switch
does not have to have 24 or 48 ports like the OEM switches sold over-the-counter (e.g. Cisco 3560).  A BMS
in this instance can have as few as 2 ports, or as many ports as the platform can support.  The main ingredient
to any BMS is the software.  

I have constructed two BMS that I use in my lab rack.  The hardware for each BMS consists of a [Dell OptiPlex GX270](http://www.dell.com/support/home/us/en/04/product-support/product/optiplex-gx270/manuals) with attached 
[Magma 13 Slot PCI Expansion System](http://www.magma.com/13-slot-pci-expansion) containing 12 x QFE NICs providing 48 total
FastEthernet ports.  This is a bulky setup as the PC and the Magma Chassis take up 8u in my network rack.

I use [Ubuntu 14.04 LTS Server 32-bit](http://releases.ubuntu.com/14.04/) for the OS and [Open vSwitch](http://openvswitch.org/)
for the switching software.

The configuration for the BMS can be done using the following steps.  These steps assume the OS has been properly installed and 
the Magma system with NICs has been properly attached to the host PC.  The following build is used for this post.

* Platform: Dell Opti-plex GX270
* Processor(s): (1)x Pentium 4
* OS: Ubuntu Server 14.04 LTS 32-bit
* RAM: 256MB
* Peripheral: MAGMA 13-Slot PCI Expansion Chassis w/ 12 Sun QFE NICs

### Step 1. Ensure the Magma is off and power on the PC!

If the Magma is on when the PC os turned on, the PC will fail the POST.  This is probably due to all the NICs in the Magma.  A
Dell OptiPlex only has 4 PCI slots on the motherboard, now we are expanding the system by 13 more, I'm certain it was never
intended to function with that may cards.

### Step 2. Prevent the OS from automatically booting at startup

In a nutshell, the Base Address Registration for the NICs can be done by the BIOS on the motherboard, or the OS during bootup.  
If the BIOS does it (i.e. the Magma is on during system boot) the system will fail the POST.  We need the OS to allocate the memory 
addresses for the NICs.  The problem is we need to turn the Magma on after the completion of the POST but before the OS boots.  
To accomplish this, we set the GRUB Timeout to -1 (infinite) so we can manually boot the OS at the GRUB screen after turning on the 
Magma.

Open the _/etc/default/grub_ file and change GRUB_TIMEOUT to -1

	nano /etc/default/grub

Update the system configuration with the GRUB changes

	update-grub

Completely shutdown the computer and then restart it.  After the POST, the GRUB bootloader screen will appear and wait for you to 
select the OS to boot into.  Before selecting the OS, power-on the Magma.  With the Magma powered-on, select the OS to boot.

Ensure you pay close attention to the system messages as the OS boots.  Log into the PC and verify that all the NICs are recognized.

	ifconfig -a

All the ports should be there, if you want to statically name the ports, review my post titled [How to statically label network interfaces on Ubuntu 14.04](how-to-statically-label-nics.html).

### Step 3. Install Open vSwitch

	apt-get -y install openvswitch-common openvswitch-controller openvswitch-dbg \
	openvswitch-ipsec openvswitch-pki openvswitch-switch openvswitch-datapath-source \
	openvswitch-test openvswitch-datapath-dkms

That's it, you should now have a functioning BMS!  I have since upgraded my BMS to using Dell PowerEdge 1850 servers as the bare-metal.
Using the servers does not require that the Magma be turned-off during the POST.  The only draw back of using the servers is that they
are 100 times louder than the PCs.  The steps you need to take to configure a BMS is highly dependent on what you choose for the
bare-metal platform.

Now that you have a working SDN capable BMS, I encourage you to study up on [Open vSwitch](http://openvswitch.org/) and [Floodlight](http://www.projectfloodlight.org/floodlight/).



Below is a picture of my BMS connected to some Cisco switches in my lab rack.

![My Network Lab Rack](/images/labrack-rear-view.jpg)

