---
layout: post
title:  "How to statically label network interfaces on Ubuntu 14.04"
date:   2015-01-18 08:58:01
categories: How to
---

## The Problem

Many of my projects involve custom-built systems with over 48 FastEthernet interfaces spread 
out over 12 quad FastEthernet NICs.  I found that Ubuntu would sometimes generate strange 
labels/names for some of these interfaces (e.g. rename01, rename02...).  Also, even without 
the strange generated names, it would be dificult for me to figure out exactly which
physical interface I was dealing with.

## The Solution

In order to have strick control over interface naming, and to know exactly which
interface I was dealing with at all-times, I needed to statically label all interfaces.

There are many ways to go about accomplishing this task which can be broken down into 2
categories.  You can do it manually, or you can use a script to automate the process.

At first I did things manually as described below in "Solution 01", but then I found myself
constantly going through this everytime I modified the host (e.g. moved a NIC, or added some
new PCI card to the system).  So I decided to figure out a way to automate the process and 
came up with "Solution 02".  There are many ways to automate the process, just keep in mind 
that I am using a host with Ubuntu 14.04 LTS as the operating system.

### Solution 1: Manually label the network interfaces

Statically name all NICs in the /etc/udev/rules.d/70-persistent-net.rules file based 
on the BUS IDs obtained in the following steps.  This name will be applied at boot and will overide 
the dynamic names that would otherwise be applied by the OS.  This also allows for referencing 
the proper NIC in future configurations and troubleshooting.  The file should be made to 
contain one entry for each NIC that you want to statically name. 

For the static name to take effect, the dynamic system generated name must be removed.
New NICs added after the fact will cause dynamic entries to be added by the OS for the new NICs.

Step 1: Enumerate USB devices to find USB NICs  

	lsusb 

Output:

	Bus 002 Device 003: ID 0b95:7720 ASIX Electronics Corp. AX88772 

Step 2: Enumerate onboard PCI based NICs

	lspci | grep Ethernet

Output:

	00:19.0 Ethernet controller: Intel Corporation 82579V Gigabit Network Connection (rev 04) 
	02:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection 
	03:04.0 Ethernet controller: Qualcomm Atheros AR2413/AR2414 Wireless Network Adapter [AR5005G(S) 802.11bg] (rev 01) 
	04:00.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 
	04:01.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 
	04:02.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 
	04:03.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 
	05:00.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 
	05:01.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 
	05:02.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 
	05:03.1 Ethernet controller: Oracle/SUN Happy Meal 10/100 Ethernet [hme] (rev 01) 

Step 3: Verify system awareness of NICs

	find /sys/devices -name net

Output:

	/sys/devices/pci0000:00/0000:00:19.0/net 
	/sys/devices/pci0000:00/0000:00:1c.2/0000:02:00.0/net 
	/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1:1.0/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:02.0/0000:04:00.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:02.0/0000:04:01.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:02.0/0000:04:02.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:02.0/0000:04:03.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:03.0/0000:05:00.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:03.0/0000:05:01.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:03.0/0000:05:02.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:03.0/0000:05:03.1/net 
	/sys/devices/pci0000:00/0000:00:1e.0/0000:03:04.0/net

Step 4: Statically label each NIC based on BUS ID

	sudo cat <<EOF > /etc/udev/rules.d/70-persistent-net.rules 

	KERNELS=="0000:03:04.0", SUBSYSTEM=="net", NAME="wlan0" 
	KERNELS=="2-1.1:1.0", SUBSYSTEM=="net", NAME="eth0" 
	KERNELS=="0000:02:00.0", SUBSYSTEM=="net", NAME="eth1" 
	KERNELS=="0000:00:19.0", SUBSYSTEM=="net", NAME="eth2" 
	KERNELS=="0000:04:00.1", SUBSYSTEM=="net", NAME="port-1" 
	KERNELS=="0000:04:01.1", SUBSYSTEM=="net", NAME="port-2" 
	KERNELS=="0000:04:02.1", SUBSYSTEM=="net", NAME="port-3" 
	KERNELS=="0000:04:03.1", SUBSYSTEM=="net", NAME="port-4" 
	KERNELS=="0000:05:00.1", SUBSYSTEM=="net", NAME="port-5" 
	KERNELS=="0000:05:01.1", SUBSYSTEM=="net", NAME="port-6" 
	KERNELS=="0000:05:02.1", SUBSYSTEM=="net", NAME="port-7" 
	KERNELS=="0000:05:03.1", SUBSYSTEM=="net", NAME="port-8" 
	EOF

At this point, reboot the host so the interfaces take their new names/labels!

Step 5: Setup persistent NIC configurations

You will need to modify the /etc/network/interfaces file with your desired
NIC configurations.  This will ensure the configurations are persistent.

	cat <<EOF > /etc/network/interfaces 

	auto lo 
	iface lo inet loopback 

	# WIRELESS LAN NETWORK / ISP LINK 
	auto wlan0 
	iface wlan0 inet dhcp 
	wpa-ssid <SSID>
	wpa-psk <PASS PHRASE> 

	# WIRED LAN NETWORK 
	auto eth0 
	iface eth0 inet static 
	address 192.168.206.1 
	network 192.168.206.0 
	netmask 255.255.255.0 
	broadcast 192.168.206.255 
	#gateway 192.168.206.254 
	#dns-nameservers 192.168.206.254 

	auto eth1 
	iface eth1 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto eth2 
	iface eth2 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-1 
	iface port-1 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-2 
	iface port-2 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-3 
	iface port-3 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-4 
	iface port-4 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-5 
	iface port-5 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-6 
	iface port-6 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-7 
	iface port-7 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 

	auto port-8 
	iface port-8 inet manual 
	up ifconfig $IFACE 0.0.0.0 up 
	up ip link set $IFACE promisc on 
	EOF

Step 6: Reboot the system 

After the system reboots, do _ifconfig_ and you will find all the NICs labeled as specified 
and in an UP state.

### Solution 2: Use a script to label the network interfaces

As you well know there are countless ways in linux to automate a process with scripts, the scripts
used here are based on my own trial and error.  Using scripting makes things go 100 times faster and
scripts are portable to other hosts; however, the labelling is automated so it may not be as precise
as you would like.  If you require extreme precision, use solution 1 and do it manually.

Step 1: Use AWK to generate the /etc/udev/rules.d/70-persistent-net.rules file

As noted above in solution 1, the /etc/udev/rules.d/70-persistent-net.rules file is used by the OS to 
dynamically label all NICs.  The script below uses AWK to read the BUS IDs from the 
_find /sys/devices/pci* -name net_ output into variable (i).  AWK next prints a formatted line for each 
BUS ID and names the interface using NR-1 as the numerical counter.  Lastly the output is directed to 
a new /etc/udev/rules.d/70-persistent-net.rules file that overwrites the previous file of the same name.
Note that I only perform a _find_ on PCI devices, so USB NICs will not be statically labeled.

	find /sys/devices/pci* -name net | \
	awk -F/ '{i=NF-1; print "KERNELS==\""$i"\", SUBSYSTEM==\"net\", NAME=\"eth"NR-1"\"";}' \
	> /etc/udev/rules.d/70-persistent-net.rules

Reboot the system for the new names to be applied!

Step 2: Use AWK and printf to generate the /etc/network/interfaces file

Network interface configurations in the /etc/network/interfaces file are persistent across reboots.

	for i in $(ifconfig -a | grep eth | awk '{ print $1 }'); do \
	printf "\n# AUTOMATED CONFIGURATION \nauto $i \niface $i inet manual \n \
	\t up ifconfig \$IFACE 0.0.0.0 up \
	\n\t up ip link set \$IFACE promisc on \n" >> /etc/network/interfaces;done 
	
Step 3: Reboot the system 

After the system reboots, do _ifconfig_ and you will find all the NICs labeled as specified 
and in an UP state.

All the code shown above can be combined and placed in a shell script for repeated use!
