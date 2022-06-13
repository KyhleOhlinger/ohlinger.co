---
layout: post
title: Setting up a Basic AD Lab Environment
date: 2020-06-24 12:00:00 +0200
img: active-directory.png # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical]
---
 
I wasn't going to be making a post on this topic, but since a few of my posts refer back to having a domain set up, I thought it would be useful to explain the process of setting up your own local testing environment in the event that you find yourself wanting to play around with Active Directory (AD). I've come across a lot of examples of how to set up AD environments using PowerShell one-liners, various vagrant files, and any other automation tactics that you can think of, but one thing that seems to have fallen away has been people setting up their AD lab environments manually. 

While the other methods are really fantastic if you want a fully fledged lab that you can start exploiting almost instantly, you miss out on the learning opportunities that come with setting up your own environment from scratch. The main idea of this post is to (at a high-level) demystify a bit of the set up of AD and let you build your own lab. Once you have the lab set up, you can expand and attempt to intentionally misconfigure your environment to be vulnerable to issues that you want to learn about and then test out the tools for both finding and exploiting each misconfiguration. 

**Prerequisites:**
In order to begin creating your own local AD lab environment, you will need the following:

* Windows Server 2016/2019 VM
* Windows 10 VM
* Kali VM or related security distro e.g. ParrotOS (if you want to abuse any misconfigurations)

Evaluation Versions of Windows can be found at:
* [Windows 10](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise)
* [Windows Server](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019)

(These work for 180 days, unless you delete the service that reboots the box after that period, then they work like normal ISOs that just prompt you to activate).  Non-evaluation versions of consumer (Pro and Home) Windows 10 can be found [here](https://www.microsoft.com/engb/software-download/windows10) as long as you are coming from a non-windows user agent. 

The setup is divided into the following parts:
1. [Creating a Domain Controller](#setting-up-active-directory)
2. [Networking](#networking)
3. [AD Lab Basics](#ad-lab-basics)
4. [Linking a Workstation to your Domain](#linking-a-workstation-to-the-domain)

## Setting up Active Directory

This section will provide the basic steps required to set up -- and administer -- a small domain. These sections only include the basic aspects as a point of reference, the [Useful Resources](#useful-resources) section below contains links to resources which can be used when attempting to set up your own basic domain.

### Initial Setup

* Install Windows Server 2016 and install Guest Additions (Personal preference).
* Rename the host to something which you are able to identify on the network.

Using the Windows Server VM that we installed, we are now ready to create out first Domain Controller (DC). In order to create a DC, we need to do the following:

1. Open the Start Menu on the new Windows Server instance, go to PowerShell (as Administrator) and type ServerManager.exe and press enter.
2. Within the Server Manager, click on “Add Roles and Features”.
3. The manager should open the “Add Roles and Features” wizard. Click on next to proceed.
4. Select “Active Directory Domain Services” and click “Add Features”.

Once the role is installed, click the “Warning sign” in Server Manager and select “Promote this Server to Domain Controller”.
1. You will first need to create an Active Directory Forest, so select “New Forest” and enter the domain name and recovery (DSRM) password.
2. An error will pop up because the DNS infrastructure does not exist yet.
3. Click through until arriving at the installation page, this page will then install AD and DNS.

Once the system reboots, log in to the system as before. You are still administrator, only now you are a Domain Administrator within your lab environment. There are no local accounts on the server as the new DC is the domain. You can create additional DCs for replication if you want to, however for this post we are going to focus on a single DC and a single Workstation setup.

## Networking
Within the Server Manager on the newly created DC:
1. Right click on your server -- “Add Roles and Features”.
2. Click next until you reach “Server Roles”.
3. Within Server Roles &rarr; Select DHCP Server and complete setup.
4. Click on Notifications and “Complete DHCP Configuration”.

Next, we need to configure DHCP. In Server Manager click on the Tools menu in the upper right corner and select DHCP:
1. Expand your domain on the left-hand side, right click IPv4 and select Add New Scope.
2. Click Next through the Wizard and when prompted, name your DHCP scope whatever you want to.
3. When prompted for the scope, create a range of 50 to 100 IPs within your network and set your subnet mask appropriately (Unless you want more hosts in the environment).
4. Keep clicking next in the wizard until you are asked if you would like set additional options, select yes and click next.

If you are using VirtualBox, you can go to Network &rarr; Right Click on your host adapter, e.g. vboxnet0 &rarr; edit, and the IP Address should be there. Within your DC's IPv4 Settings, set a Static IP Address based on your Host-Only Network Settings within your virtual environment:
* IP Address should use the same 3 initial octets, you can set the final one to anything you like.
* Set the Subnet Mask to 255.255.255.0
* Set the Default Gateway to the initial 3 octets followed by 1 (e.g. 192.168.56.1)
* Set a static DNS Server to: 127.0.0.1 (localhost)

## AD Lab Basics
Now that we have the basic setup for a domain completed, we need to actually be able to do something with it. This includes, creating Users, Computers, Group Policy Objects (GPOs), Access Control Lists (ACLs), and more. GPOs and ACLs are out-of-scope for this blog post, however the basics on how to add them to your environment are included in the subsections below.

### Creating Users
We need to create users within our Domain, we’ll be using “Active Directory Administrative Center” to do this.
* We’ll be creating Organization Units or “OUs”, to being -- create a Users and Computers OU.
* At this point you can create as many Users as you want to and add them to the new OU.
    * I would recommend creating at least one normal user and one administrative user.
* Create a new windows 10 VM and then join it to the domain (AD) as described in [Linking a Workstation to your Domain](#linking-a-workstation-to-the-domain) below.
* Once your VM has been connected to the domain, the new PC will show up in the "Active Directory Administrative Center".
    * Right click on the PC under the Computers OU and move it to the OU that you created above.

### Group Policy
Group Policy is one of the main reasons for AD’s success since it allows you to granularly configure settings throughout a domain. In order to create a new Computer GPO, you can do the following:
1. Open “Group Policy Management” &rarr; Open the domains tabs until reaching Computers &rarr; Right click and add “create GPO...”
2. Right click on the new GPO and edit it as per your specific requirements. 

Computers will check for updates approximately every 45 minutes and you can speed this up by running `gpupdate /force` on the computer that you want to update. **Note:** In order to allow a specific user to modify the GPO, you would need to add the permission within the “Delegation” tab.

### Access Control Lists
Access Control Lists (ACLs) enable administrators to set permissions over AD objects. It allows you to granularly configure User, Computer, and Group access permissions throughout a domain.
* Open “Active Directory Users and Computers” &rarr; Open the View tab &rarr; Enable “Advanced Features”
* This will allow you to access the “Security” tab when viewing and editing an AD object's ACLs.

## Linking a Workstation to the Domain
The final portion of creating a domain environment is to allow someone to access it. In order to do this, you need to create a workstation and link it to the new domain. From there, you can use the User accounts that you created before in order to access the workstation once it has been linked to the domain. In order to link a workstation, you need to:
1. Open Control Panel &rarr; System and Security &rarr; System
2. Click on “Change settings”
3. Change the computer name &rarr; Member of: Domain &rarr; FQDN
4. Set the IPv4 DNS Server to the DCs IP address within your new workstation

In order to ensure that your PC is linked to the Domain and that you can communicate with the DC, open command prompt and ping your DC's IP address. If you are able to ping the DC then the networking portion has been successful. 

The approach described above is a high-level overview of setting up a basic lab environment. If this is your first time setting up an AD lab, the links below might be useful if you do get stuck during the installation process.

## Useful Resources

* <https://blogs.technet.microsoft.com/canitpro/2017/02/22/step-by-step-setting-up-active-directory-in-windows-server-2016/>
* <https://www.c2.lol/setting-up-an-active-directory-lab/part-1>
* <http://thehackerplaybook.com/Windows_Domain.htm>
* <https://1337red.wordpress.com/building-and-attacking-an-active-directory-lab-with-powershell/>
