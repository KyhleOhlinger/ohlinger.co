---
layout: walkthrough
title: HackTheBox - Delivery
date: 2021-05-23 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/delivery.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Delivery --`10.10.10.222`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/Overview.png"  style="width: 35%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Delivery$ nmap -sC -sV 10.10.10.222
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-15 10:09 EDT
Nmap scan report for 10.10.10.222
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.33 seconds
```
</details>

From the output shown above, we can see that the machine is a Linux machine and I added `delivery.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I started by navigating to the web application:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img1.png"  style="width: 65%" />
</p>

I clicked around the application, and the "Contact Us" page provided some additional information as well as a link to two new pages:
* MatterMost - delivery.htb:8065
* HelpDesk - helpdesk.delivery.htb

Clicking on the `MatterMost` link redirected me to port 8065: <http://delivery.htb:8065/>:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img2.png"  style="width: 65%" />
</p>

> Mattermost is an “open source, self-hosted Slack-alternative”, which basically means that it’s a real-time messaging solution for teams and enterprises that you host yourself. It’s written in the Go programming language.

Some Google searching lead me to the following blog post on MatterMost: <https://haxx.ml/post/142844845111/hacking-mattermost-from-unauthenticated-to-system>. The blog post covers areas once a user has already registered for the application. I attempted to register, however I was unable to since it sent an email waiting to confirm receipt of the registration. 

### Abusing HelpDesk Logic Flaw

Since I had no way of retrieving the email, I added `helpdesk.delivery.htb` to the `/etc/hosts` file and navigated to the HelpDesk application which I identified earlier:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img3.png"  style="width: 65%" />
</p>

I attempted to register an account within the support center, however, as with the MatterMost application, it asked me to confirm my account. At this point I needed to figure out a way in which I could register an account. The Support Center had two additional options: `Open a New Ticket` and `Check Ticket Status`. I attempted to open a new ticket as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img4.png"  style="width: 65%" />
</p>

The only `Help Topic` option that was available to me was `Contact Us`, so I used that as the default option. After submitting the ticket, I was provided with an email address for the organization and a Ticket ID:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img5.png"  style="width: 65%" />
</p>

In order to abuse this, I created an account on MatterMost using the email address which was provisioned by HelpDesk:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img6.png"  style="width: 65%" />
</p>

After submitting the request, I went back to the service desk and viewed the ticket thread based on the email address and ticket reference:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img7.png"  style="width: 65%" />
</p>

### Password Disclosure in MatterMost

As shown above, registration was successful and I had an email verification link. I pasted the link into the search bar and was able to log in to the web application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img8.png"  style="width: 65%" />
</p>

Now that I had access to the application, I started off by browsing the web application and came across the following public thread:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img9.png"  style="width: 65%" />
</p>

I recalled from the Support Center web page that there was an Agent Sign-In link: <http://helpdesk.delivery.htb/scp/login.php>. I went to the link and entered the new credentials which successfully signed me in to the application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img10.png"  style="width: 65%" />
</p>

The web application didn't provide me with any new information, so I decided to try and access the host directly with SSH which worked:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img11.png"  style="width: 80%" />
</p>

## Privilege Escalation using Maildeliverer

Success! I now had access to the host. I started off by doing some basic enumeration with linpeas and attempting a `sudo -l`. However, the user was unable to run sudo on the host.

<details>

```bat
maildeliverer@Delivery:/var/www/osticket/upload/include$ echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
Linux Delivery 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64 GNU/Linux
 
hostname:
Delivery
 
id
uid=1000(maildeliverer) gid=1000(maildeliverer) groups=1000(maildeliverer)
 
ifconfig:
ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.222  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:849d  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:849d  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:84:9d  txqueuelen 1000  (Ethernet)
        RX packets 4643  bytes 851585 (831.6 KiB)
        RX errors 0  dropped 264  overruns 0  frame 0
        TX packets 6386  bytes 2237643 (2.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 5587  bytes 979514 (956.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5587  bytes 979514 (956.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
maildeliverer
```

</details>

After enumerating the host, I found the MatterMost directory which was in `/opt`. I looked in the config file and found some credentials:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img12.png"  style="width: 80%" />
</p>

Using those credentials, I was able to log in to the MySQL instance on the host. There were a large number of tables within the database, but I started off with the `Users` table. I began with a `Select * From Users` statement and then refined it to the following:

```sql
MariaDB [mattermost]> select Username, Password from Users;
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img13.png"  style="width: 80%" />
</p>

### Cracking Passwords using Hashcat Rulesets

Since the table contained a hash for `root`, I sent it through JohnTheRipper with no luck. If you recall from the MatterMost web application:

> Also please create a program to help us stop re-using the same  passwords everywhere. Especially those that are a variant of  “PleaseSubscribe!”
> PleaseSubscribe! may not be in RockYou but if any hacker manages to  get our hashes, they can use hashcat rules to easily crack all variations of common words or phrases.

To make use of HashCat, I needed to know the mode. I knew that it was a bcrypt hash (mode 3200) based on the first few characters, but to ensure that I was correct, I ran it through HashID:

```bash
vagrant@ko:~/Desktop/HackTheBox/Delivery$ hashid hashes.txt 
--File 'hashes.txt'--
Analyzing '$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.s1STWb4.4ScG.anuu7v0EFJwgjjO'
[+] Blowfish(OpenBSD) 
[+] Woltlab Burning Board 4.x 
[+] bcrypt 
```

The next step in the process was creating my own wordlist and ruleset. For the wordlist I simply created a file called wordlist which contained `PleaseSubscribe!`. For the ruleset, I made use of the following Github project: <https://github.com/praetorian-inc/Hob0Rules.git>. The final command with hashcat was as follows:

```bat
hashcat -a 0 -m 3200 hashes.txt wordlist.txt -r Hob0Rules/hob064.rule -o cracked.txt
```

The `hob064.rule` ruleset did not work, so I needed to change it to the `d3adhob0.rule` which eventually cracked and identified the following password:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img14.png"  style="width: 80%" />
</p>

Since it was for the `root` user, I attempted to elevate my privileges using the `su` command and successfully authenticated as `root`:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Delivery/img15.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.