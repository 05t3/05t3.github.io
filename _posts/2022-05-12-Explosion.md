---

title: "Explosion"
date: 2022-05-12 01:09:33 +0300
author: oste
image: /assets/img/Posts/explosion.png
categories: [HTB, Starting Point, Tier 0]
tags: [windows, xfreerdp, rdp]
---

#### What does the 3-letter acronym RDP stand for?

`Remote Desktop Protocol`

#### What is a 3-letter acronym that refers to interaction with the host through a command line interface?

`CLI`

#### What about graphical user interface interactions?

`GUI`

#### What is the name of an old remote access tool that came without encryption by default?

`Telnet`

#### What is the concept used to verify the identity of the remote host with SSH connections?

`Public-Key cryptography`

#### What is the name of the tool that we can use to initiate a desktop projection to our host using the terminal?

`xfreerdp`

#### What is the name of the service running on port 3389 TCP?

```bash
➜  nmap -sC -sV -p- -Pn -T4 10.129.154.204
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-22 09:09 EST
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
Nmap scan report for 10.129.154.204 (10.129.154.204)
Host is up (0.24s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: EXPLOSION
|   NetBIOS_Domain_Name: EXPLOSION
|   NetBIOS_Computer_Name: EXPLOSION
|   DNS_Domain_Name: Explosion
|   DNS_Computer_Name: Explosion
|   Product_Version: 10.0.17763
|_  System_Time: 2021-12-22T14:26:19+00:00
| ssl-cert: Subject: commonName=Explosion
| Not valid before: 2021-09-20T16:22:34
|_Not valid after:  2022-03-22T16:22:34
|_ssl-date: 2021-12-22T14:26:31+00:00; +3s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-12-22T14:26:21
|_  start_date: N/A
|_clock-skew: mean: 2s, deviation: 0s, median: 2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1048.58 seconds
```

`ms-wbt-server`

#### What is the switch used to specify the target host's IP address when using xfreerdp?

```bash
➜  xfreerdp -h

FreeRDP - A Free Remote Desktop Protocol Implementation
See www.freerdp.com for more information

Usage: xfreerdp [file] [options] [/v:<server>[:port]]

```

`/v:`

#### Submit root flag

Logging in as an Administrator without specifying the password, we can get the flag on the desktop as shown below.

```bash
➜  xfreerdp /v:10.129.154.204 /cert:ignore /u:Administrator
[09:34:40:835] [2284:2285] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[09:34:40:835] [2284:2285] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
[09:34:40:835] [2284:2285] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd
[09:34:40:835] [2284:2285] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprdr
[09:34:40:204] [2284:2285] [INFO][com.freerdp.primitives] - primitives autodetect, using optimized
[09:34:40:231] [2284:2285] [INFO][com.freerdp.core] - freerdp_tcp_is_hostname_resolvable:freerdp_set_last_error_ex resetting error state
[09:34:40:231] [2284:2285] [INFO][com.freerdp.core] - freerdp_tcp_connect:freerdp_set_last_error_ex resetting error state
Password:
[09:34:57:717] [2284:2285] [INFO][com.winpr.sspi.NTLM] - VERSION ={
[09:34:57:717] [2284:2285] [INFO][com.winpr.sspi.NTLM] -        ProductMajorVersion: 6
[09:34:57:717] [2284:2285] [INFO][com.winpr.sspi.NTLM] -        ProductMinorVersion: 1
[09:34:57:717] [2284:2285] [INFO][com.winpr.sspi.NTLM] -        ProductBuild: 7601
[09:34:57:717] [2284:2285] [INFO][com.winpr.sspi.NTLM] -        Reserved: 0x000000
[09:34:57:718] [2284:2285] [INFO][com.winpr.sspi.NTLM] -        NTLMRevisionCurrent: 0x0F
...snip...
```

![image](https://user-images.githubusercontent.com/58165365/147109143-24a61df9-77ca-4def-8f6c-9931fe8d8172.png)

`HTB{951fa96d7830c451b536be5a6be008a0}`
