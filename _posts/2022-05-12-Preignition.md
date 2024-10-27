---

title: "Preigition"
date: 2022-05-12 01:09:33 +0300
author: oste
image: /assets/img/Posts/preignition.png
categories: [HTB, Starting Point, Tier 0]
tags: [windows, smb, smbclient]
---

#### What is considered to be one of the most essential skills to possess as a Penetration Tester?

`dir busting`

#### What switch do we use for nmap's scan to specify that we want to perform version detection

`-sV`

#### What service type is identified as running on port 80/tcp in our nmap scan?

```bash
➜  nmap -sC -sV -p- -T4 10.129.110.232
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-22 15:15 EST
Nmap scan report for 10.129.110.232 (10.129.110.232)
Host is up (0.24s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.14.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 679.19 seconds
```

`http`

#### What service name and version of service is running on port 80/tcp in our nmap scan?

`nginx 1.14.2`

#### What is a popular directory busting tool we can use to explore hidden web directories and resources?

`gobuster`

#### What switch do we use to specify to gobuster we want to perform dir busting specifically?

`dir`

#### What page is found during our dir busting activities?

```bash
┌──(root💀kali)-[/home/kali]
└─# gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.129.110.232 -t 50
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.110.232
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/12/22 15:17:58 Starting gobuster in directory enumeration mode
===============================================================
/admin.php            (Status: 200) [Size: 999]

===============================================================
2021/12/22 15:18:22 Finished
===============================================================
```

`admin.php`

#### What is the status code reported by gobuster upon finding a successful page?

`200`

#### Submit root flag

![image](https://user-images.githubusercontent.com/58165365/147151294-a0e40974-6a0d-4cbc-bf5b-7c1b2faafee8.png)

You can login using default credentials of `admin:admin`

![image](https://user-images.githubusercontent.com/58165365/147151542-e1c9b22b-34e6-4448-bab3-43801e06a24d.png)

![image](https://user-images.githubusercontent.com/58165365/147151597-57e4b0d2-fd92-46de-a220-76e1e45b5cb7.png)
