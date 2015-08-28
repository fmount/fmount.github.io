---
layout: post
title: "Qemu Multiple Networks using TAP networking Interfaces"
description: ""
category: 
tags: [virtualization, networking]
comments: true
---
First of all we have to configure the host, because the guest needs to share the network
configuration using the virtualization method proposed by the TAP mechanism.
Suppose we want two network interface: the first is a NAT "hand made" and the second is a private
vlan between two or more VMs.
So, we have to work in the host adding two bridges interfaces and binding them to the relative TAP
virtual devices:


	brctl addbr br0
	ip tuntap add tap0 mode tap
	ifconfig tap0 promisc
	brctl addif br0 tap0
	dnsmasq --interface=br0 --bind-interfaces --dhcp-range=172.20.0.2,172.20.255.254

This is the first lan obtained by a bridging operation; if you want to share the internet access from
the host, you only have to add some iptables rules to enable traffic on the external network (and
allow the incoming one which is dropped by default).
In this way, you can obtain a NAT mode over a _hostonly_ configuration.

Specifically, we should perform these commands:


	echo 1 > /proc/sys/net/ipv4/ip_forward
	iptables -t nat -A POSTROUTING -o eth0 -J MASQUERADE
	iptables -I FORWARD 1 -i tap0 -J ACCEPT
	iptables -I FORWARD 1 -o tap0 -m state RELATED, ESTABLISHED -J ACCEPT

After that, in order to create a second vlan we just follow the above described approach and perform
the following commands:


	brctl addbr br1
	ip tuntap add tap1 mode tap
	ifconfig tap1 promisc
	brctl addif br1 tap1

Unlike before, an important advice is to set up the network parameters manually: in this way we can
control and better manage the private VLANs between VMs:


	ifconfig br1 up
	ifconfig br1 192.168.56.1 netmask 255.255.255.0

Finally, after this configuration phase, we can run the VM using the network devices configured before:


	qemu-system-x86_64 --enable-kvm -net nic -net bridge,br=br0 -net nic, vlan=1 -net bridge,
	vlan=1,br=br1 -hda <img\_path> -m 2048 -cpu host

Using this method, we can create all the network configurations needed to the components of the architecture, adding and removing devices according to your needs.
