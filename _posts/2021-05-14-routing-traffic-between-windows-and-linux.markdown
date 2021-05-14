---
layout: post
title: Routing Traffic Between Windows and Linux
date: 2021-05-14 12:00:00 +0200
img: routing.jpg # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical]
---

I recently had a situation where I needed to exploit a Windows service but I wanted to catch the reverse shell on my Kali machine. Obviously proxychains or the like could help, but I was already connecting to the network through a Virtual Private Network (VPN). There are likely several posts describing situations similar to this one, and I am not going to do a deep dive into the semantics. Instead, I am going to provide the commands that I used below! 

### Breakdown of Commands
IP Forwarding describes sending a network package from one network interface to another on the same device. In order to achieve this, the commands listed below are divided between Kali and Windows. The steps included within the Kali setup are:

* Enable IP Forwarding
* Create IP Table Rules
* Nat firewall rules

The steps included within the Windows setup are:
* Create a route within the routing table

#### Enable Forwarding on Kali Machine:
"IP forwarding" is a synonym for "routing." It is called "kernel IP forwarding" because it is a feature of the Linux kernel. Further information is available [here](https://linuxhint.com/enable_ip_forwarding_ipv4_debian_linux/#:~:text=On%20a%20Linux%20system%20the,in%20need%20of%20that%2C%20usually).

```bash
# Enable IP Forwarding (Assuming hosts are bridged)
echo 1 > /proc/sysy/net/ipv4/ip_forward
```

#### Create IP Table Rules on Kali Machine:
IPtables is used to set up, maintain, and inspect the tables of IP packet filter rules in the Linux kernel. More information is available [here](Targets
https://linux.die.net/man/8/iptables). In addition, I have broken down the commands below:

* **FORWARD** -- All packets neither destined for nor originating from the host computer, but passing through (routed by) the host computer.
* **ESTABLISHED** -- meaning that the packet is associated with a connection which has seen packets in both directions.
* **RELATED** -- meaning that the packet is starting a new connection, but is associated with an existing connection, such as an FTP data transfer, or an ICMP error.
* **ACCEPT** -- means to let the packet through.

```bash
# Create IP Tables Rule for Forwarding
iptables -A FORWARD -i tun0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i eth0 -o tun0 -j ACCEPT
```

#### Create a Route on Windows Machine:
A routing table dictates where all packets go when they leave a system, whether that system is a physical router or a PC. More information is available [here](https://www.howtogeek.com/howto/windows/adding-a-tcpip-route-to-the-windows-routing-table/).

```bash
# Add Route
route add <tun0 range> mask <netmask ip> <ip of Linux host>
e.g. route add 10.10.10.0 mask 255.255.255.0 192.168.1.131

# Confirm that it's been added
route print
```

#### Masquerade Traffic on Kali Machine:
* **POSTROUTING** -- for altering packets as they are about to go out.
* **MASQUERADE** -- masquerading is equivalent to specifying a mapping to the IP address of the interface the packet is going out.
    * This target is only valid in the NAT (Network Address Translation) table, in the POSTROUTING chain.

```bash
# Add NAT
iptables -t nat -A POSTROUTING -s <eth0 Range> -o tun0 -j MASQUERADE
e.g. iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o tun0 -j MASQUERADE
```

#### Confirm Connection on Windows Machine:
Ping uses the ICMP protocol's mandatory ECHO_REQUEST datagram to elicit an ICMP ECHO_RESPONSE from a host or gateway. More information is available [here](https://linux.die.net/man/8/ping).

```bash
# Ping a host on the VPN network to confirm
ping 10.10.10.219
```

#### Capture Traffic on Kali Machine:
Tcpdump prints out a description of the contents of packets on a network interface that match the Boolean expression. More information is available [here](https://www.tcpdump.org/manpages/tcpdump.1.html).
    
```bash
# Check for connection via Ping
## -n excludes DNS records
sudo tcpdump -i tun0 icmp -n
```

If everything has been routed correctly, the ping from the Windows host should look like it's coming from your Kali machine. Using this method, you will be able to exploit the service using your Windows host and catch any potential shells on your Kali machine using a single VPN connection.

### Bonus Logging
The following iptables command can be used to log all connections to your machine, e.g. whenever a machine connects back to you. This iptable rule can be very useful when debugging, e.g. verifying the port that the external machine is using to connect to you. It can also be used as a trigger if you are ever detected on an engagement or someone is attacking your box.

```bash
# Add Rule
iptables -A INPUT -p tcp -m state --state NEW -j LOG --prefix "IPTables New-Connection: " -i tun0

# View log information
sudo grep iptables /var/log/messages
```

Hopefully this helps someone and if you would like a more in-depth explanation, please feel free to reach out!