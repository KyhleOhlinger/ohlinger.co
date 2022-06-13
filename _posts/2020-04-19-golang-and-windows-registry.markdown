---
title: Issues with Golang and Windows Registry
author: kyhle
date: 2020-04-19 12:00:00 +0200
categories: [InfoSec, Technical]
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
image:
  path: /assets/img/golang.jpg
  width: 800
  height: 500

---



I started out with the idea of creating Golang payloads which would be able to bypass Antivirus (AV) software when doing security assessments. However, many AV solutions are becoming increasingly sophisticated and, while it is by no means impossible to bypass them, I decided to try and find a solution which would be guaranteed to work every time (Under the right conditions of course). After looking in to a few different AV solutions, I realised that the ones I looked at all had an exclusions list, which is where this blog post comes in. The idea was to create a non-malicious executable which would write an entry to Windows Defender's exclusion path which could then be used to ensure that malicious payloads do not get flagged by Windows Defender. 

In order to do this, I first had a look at the registry and saw that the path `“HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths”` contained information which I thought would be interesting. The Windows Registry Editor (Regedit) path and the directory information is shown below:

<p class="imgMiddle">
<img src="/assets/img/Golang/img1.png" style="width:80%" />
</p>

In order to test this theory (adding an exclusion to registry), I used the following PowerShell command to create a new exclusion for my "malicious" executable -- test.exe:

```powershell
powershell -inputformat none -outputformat none -NonInteractive -Command Add-MpPreference -ExclusionPath "C:\Windows\Temp\test.exe"
```

As expected, it worked and the path was added to the registry. Now why didn't I just stop here if the solution worked? Well, PowerShell is a solution, a lot of organisations are either disabling or flagging PowerShell usage, and in order to stay relatively silent in a network, new solutions need to be explored. Since I was looking into Golang at the time -- I decided to try and implement this myself. After a bunch of Googling and looking into the Golang Registry documentation, I found some code which would allow me to at least read from the registry. I decided to try this out, with the goal of potentially accessing some basic information. The code is shown below:

```go
winInfo, err := registry.OpenKey(registry.LOCAL_MACHINE, `SOFTWARE\Microsoft\Windows NT\CurrentVersion`, registry.QUERY_VALUE)
check(err)
defer winInfo.Close()

CurrentVersion, _, err := winInfo.GetStringValue("CurrentVersion")
check(err)

fmt.Printf("Value: " + CurrentVersion +"\n")
```

This returned the expected information and I thought that I was going to be able to fly through this little experiment. I updated the code and tried to write to the registry, but it kept failing and I had no idea why. In order to debug this, I used the following code to read the backup location from the Windows Defender registry key:

```go
regInfo, err := registry.OpenKey(registry.LOCAL_MACHINE, `SOFTWARE\Microsoft\Windows Defender`, registry.QUERY_VALUE)
check(err)
defer regInfo.Close()

BackVersion, _, err := regInfo.GetStringValue("BackupLocation")
check(err)
fmt.Printf("Value: " + BackVersion)
```

## The Problem

The strange thing was, it returned the following error: `The system cannot find the file specified`. I was not expecting this, and I really didn't understand why it couldn't find the information even though I could clearly see it in Regedit. I went over the above code multiple times and even created a Stack Overflow [post](https://stackoverflow.com/questions/58819887/creating-registry-key-with-golang) to figure out if anyone has had the same issues. At this point I was stumped, I had no idea what was wrong and even the almighty Google did not have any tangible information for me to refer back to. 

## The Rabbit Hole

I didn't feel like just waiting around, so I decided to take a deep dive into the problem. I looked into the XML Definitions that the `Add-MPPreference` PowerShell module uses: `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\ConfigDefender`, but that didn't help since it only contained the definitions and no actual calls. Trying to figure out what the `Add-MPPReference` function called turned out to be a whole new kettle of fish, I started out by reading the following [Sysmon blogpost](https://www.blackhillsinfosec.com/getting-started-with-sysmon/) and though that this would solve all my problems. I followed the guide, started Event Viewer, and navigated to Applications and Service logs &rarr; Microsoft &rarr; Windows &rarr; Sysmon. I decided to see what it logged when I ran my code implementation, the result is shown below:

<p class="imgMiddle">
<img src="/assets/img/Golang/img4.png" style="width:80%" />
</p>

This was the first time that I used Sysmon, and needless to say, I had no idea how Sysmon actually worked. I was just poking around and trying to see if anything stuck out. I also found the log for the `Add-MPPReference` call shown below, unfortunately, it did not show registry keys:

<p class="imgMiddle">
<img src="/assets/img/Golang/img5.png" style="width:70%" />
</p>

Looking at the output, I wondered if I could have done a lookup on the `ProcessGuid` or `ParentProcessGuid` in order to find the registry key associated with the call. However, since I was already on my way down one rabbit hole and didn't feel like learning exactly how Sysmon worked, I decided to use Procmon -- also from the Sysinternals suite -- to get some more detailed information about the services and processes being called from the PowerShell module. I found the call and that specific process didn't contain any really useful information. After searching through the processes, I found that it made use of WMI calls under the hood:

<p class="imgMiddle">
<img src="/assets/img/Golang/img7.png" style="width:450px"/>
</p>

In my opinion, the following calls seemed to be relevant:
* `C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.1910.4-0\MpOAV.dll`
* `wmiprvse.exe (HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths)`

Great, this was something new to go on and I tried to figure out how it was calling the functions, which lead me to the following [documentation](https://docs.microsoft.com/en-gb/previous-versions/windows/desktop/defender/msft-mppreference). It had a lot of useful information, but nothing about the registry keys that it was calling, even the following call to the `WMIObject` did not return the output that I was looking for:

```powershell
Get-WmiObject -namespace "root\Microsoft\Windows\Defender" -List
```

I decided to keep looking in Procmon and eventually found that the WMI call actually did make use of the `“HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths”` registry key that I was initially looking into. At this point, I was no further than I was when I started and once again I was stumped. I decided to take a break and think about possible reasons that I couldn't read this information. The next day I decided to view the associated Access Control Lists (ACLs) to see if the user context was relevant when reading the registry key.
 
 <p class="imgMiddle">
<img src="/assets/img/Golang/img9.png"  style="width: 450px" />
</p>

As shown above, the *Everyone* Group had `Read` access to the Parent object as well as the Subkeys, and Administrators had the same permissions. This meant that for all intents and purposes, any user should have been able to read this information. I then tried using `REG QUERY` to see if that could read from the registry:

```bat
REG QUERY "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender" /v BackupLocation

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender
    BackupLocation    REG_SZ    C:\ProgramData\Microsoft\Windows Defender\platform\4.18.1909.6-0
```

Turns out that it could. So once again I went back to my code and decided to try and see if there was another way of reading registry keys and subkeys. I found the following [post](https://stackoverflow.com/questions/18425465/enumerating-registry-values-in-go-golang) and implemented it. The output is provided below:


**Windows NT version:**
```bat
C:\Users\Kyhle\Desktop>EnumKeys.exe
map[string]string{"BaseBuildRevisionNumber":"239", "BuildBranch":"19h1_release_svc_prod1", "BuildGUID":"ffffffff-ffff-ffff-ffff-ffffffffffff", "BuildLab":"18362.19h1_release_svc_prod1.190628-1641", "BuildLabEx":"18362.239.x86fre.19h1_release_svc_prod1.190628-1641", "CompositionEditionID":"Enterprise", "CurrentBuild":"18362", "CurrentBuildNumber":"18362", "CurrentMajorVersionNumber":"10", "CurrentMinorVersionNumber":"0", "CurrentType":"Multiprocessor Free", "CurrentVersion":"6.3", "EditionID":"Enterprise", "EditionSubManufacturer":"", "EditionSubVersion":"", "EditionSubstring":"", "InstallDate":"0", "InstallationType":"Client", "ProductName":"Windows 10 Enterprise", "RegisteredOrganization":"", "RegisteredOwner":"Kyhle", "ReleaseId":"1903", "SoftwareType":"System", "SystemRoot":"C:\\WINDOWS", "UBR":"418"}
```

**Windows Defender:**
```bat
C:\Users\Kyhle\Desktop>EnumKeys.exe
map[string]string{}
```

As shown above, even though it could read from the Windows NT registry, it could still not read from the Windows Defender registry. This was both annoying and a relief, because at least I knew that it wasn't only my implementation of Golang which could not return the required information. From here, I decided to look into 3rd party software, the thought being that it's possible that only signed Microsoft executables would be able to access the Registry. I decided to try [Registry Viewer](https://accessdata.com/product-download/registry-viewer-1-8-0-5), but it turned out that even that was able to read the registry keys. 

## The end
Finally, I decided to try and implement the exact same calls using an alternative programming language, in this case python. 

```python
import errno, os, winreg

RawKey = winreg.OpenKey(winreg.HKEY_LOCAL_MACHINE, r"SOFTWARE\Microsoft\Windows NT\CurrentVersion",0, winreg.KEY_READ)
print(winreg.QueryValueEx(RawKey,"CurrentVersion"))

RawKey = winreg.OpenKey(winreg.HKEY_LOCAL_MACHINE, r"SOFTWARE\Microsoft\Windows Defender",0, winreg.KEY_READ)
print(winreg.QueryValueEx(RawKey,"BackupLocation"))
```

After implementing the code shown above, Python was able to read the registry key which meant that either my implementation of Golang's registry was incorrect or Golang's implementation had a bug. I reached out to the Golang team with the issues I was having:

* [Github](https://github.com/golang/go/issues/35730)
* [Google Groups](https://groups.google.com/forum/#!topic/golang-nuts/V-0GT02SMIs)
* [Stack Overflow](https://stackoverflow.com/questions/58819887/creating-registry-key-with-golang/58827659?noredirect=1#comment103961786_58827659)

Unfortunately, they were not able to replicate the problems that I was facing using my exact implementation. I used the following version: `go version go1.13.4 linux/amd64` and environment: `Compiled to Windows Exe: env GOOS=windows GOARCH=386`.

After all of this, I still learnt quite a lot more about Golang, Windows Registry, and Procmon than I would have if everything worked correctly from the start. You always need to try and look at the silver lining and even though I wasn't able to get exactly what I wanted out of the experience, I still managed to figure a lot of things out along the way. 