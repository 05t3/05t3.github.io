---

title: "Easy peasy"
date: 2021-12-1 01:09:33 +0300
author: oste
image: /assets/img/Posts/easypeasy.png
categories: [Tryhackme, Easy]
tags: [cyberchef, gobuster, john, linepeas, pspy64, roth13]
---

# Prerequisite

Practice using tools such as Nmap and GoBuster to locate a hidden directory to get initial access to a vulnerable machine. Then escalate your privileges through a vulnerable cronjob.

# Room Link

[Easy Peasy](https://tryhackme.com/room/easypeasyctf)

# Task 1 Enumeration through Nmap

After deploying the machine, we first start by performing an nmap scan to determine what ports are open and what services are running behind each port.

```bash
‌┌──(root💀kali)-[/home/kali]
└─# nmap -sC -sV -p- -T4 10.10.214.87
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-23 14:07 EDT
Nmap scan report for 10.10.214.87
Host is up (0.17s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: nginx/1.16.1
|_http-title: Welcome to nginx!
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: Apache/2.4.43 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 759.80 seconds
```

From the output above, we can see that we have three ports open, one running `ssh` and the other two are running web-servers.

Another notable thing from the output is that we have _robots.txt_ files on both web servers with one disallowed entries. Interesting! we'll have a look at that later. If you wanna learn more on robots.txt file, you can check out **[@CMNatic's](https://twitter.com/CMNatic)** room on [Google Dorking- Task 4](https://tryhackme.com/room/googledorking)

## How many ports are open?

**`3`**

## What is the version of nginx?

**`1.16.1`**

## What is running on the highest port?

**`Apache`**

---

# Task 2 Compromising the machine

## Using GoBuster, find flag 1.

```bash
┌──(root💀kali)-[/home/kali/Desktop/easyp]
└─# gobuster dir -u http://10.10.214.87 -w /usr/share/wordlists/dirb/big.txt -x html,php,txt,db -t 50                                                                                                                            148 ⨯ 3 ⚙
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.214.87
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,txt,db,html
[+] Timeout:                 10s
===============================================================
2021/09/23 16:30:49 Starting gobuster in directory enumeration mode
===============================================================
/hidden               (Status: 301) [Size: 169] [--> http://10.10.214.87/hidden/]
/index.html           (Status: 200) [Size: 612]
/robots.txt           (Status: 200) [Size: 43]
/robots.txt           (Status: 200) [Size: 43]

===============================================================
2021/09/23 16:36:51 Finished
===============================================================
```

Lets have a look at the pages & directory found

![easy](/assets/img/Posts/EasyPeasy/Untitled.png)

index.html

![easy](/assets/img/Posts/EasyPeasy/Untitled%201.png)

robots.txt

![easy](/assets/img/Posts/EasyPeasy/Untitled%202.png)

Looking at the `/hidden` directory, i only found a static image, nothing much really. Downloaded the image to check whether it had some files embedded but i figured it was a dead-end 🥲

So i decided to enumerate the directory instead to see whether there could be more sub-directories.

```bash
┌──(root💀kali)-[/home/kali/Desktop/easyp]
└─# gobuster dir -u http://10.10.214.87/hidden -w /usr/share/wordlists/dirb/big.txt -x html,php,txt,db -t 50                                                                                       148 ⨯ 1 ⚙
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.214.87/hidden
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,php,txt,db
[+] Timeout:                 10s
===============================================================
2021/09/23 16:19:30 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 390]
/whatever             (Status: 301) [Size: 169] [--> http://10.10.214.87/hidden/whatever/]

===============================================================
2021/09/23 16:26:22 Finished
===============================================================
```

Awesome, so now we have a new directory called `/whatever`. Lets check it out.

![easy](/assets/img/Posts/EasyPeasy/Untitled%203.png)

Another static image? 🤔 Looking at the page source, i found what looks like a base64 string

![easy](/assets/img/Posts/EasyPeasy/Untitled%204.png)

Lets try decode that on our terminal

![easy](/assets/img/Posts/EasyPeasy/Untitled%205.png)

Nice...We just found our first flag. Alternatively, you can use [**CyberChef**](https://gchq.github.io/CyberChef/) to decode the string as follows:

![easy](/assets/img/Posts/EasyPeasy/Untitled%206.png)

`flag{f1rs7_fl4g}`

---

## Further enumerate the machine, what is flag 2?

We also found _Apache_ running on port `65524` , we can go ahead and enumerate the direcories as follows:

```bash
┌──(root💀kali)-[/home/kali/Desktop/easyp]
└─# gobuster dir -u http://10.10.214.87:65524 -w /usr/share/wordlists/dirb/big.txt -x html,php,txt,db -t 50                                                                                                                            3 ⚙
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.214.87:65524
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,txt,db,html
[+] Timeout:                 10s
===============================================================
2021/09/23 16:38:00 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 280]
/.htpasswd.html       (Status: 403) [Size: 280]
/.htaccess.db         (Status: 403) [Size: 280]
/.htpasswd.php        (Status: 403) [Size: 280]
/.htpasswd.txt        (Status: 403) [Size: 280]
/.htaccess.html       (Status: 403) [Size: 280]
/.htpasswd.db         (Status: 403) [Size: 280]
/.htaccess.php        (Status: 403) [Size: 280]
/.htpasswd            (Status: 403) [Size: 280]
/.htaccess.txt        (Status: 403) [Size: 280]
/index.html           (Status: 200) [Size: 10818]
/robots.txt           (Status: 200) [Size: 153]
/robots.txt           (Status: 200) [Size: 153]
/server-status        (Status: 403) [Size: 280]

===============================================================
2021/09/23 16:43:59 Finished
===============================================================
```

Nothing much really, but first lets have a look at the _robots.txt_ file.

![easy](/assets/img/Posts/EasyPeasy/Untitled%207.png)

Looking at the User-Agent , we can see what looks like an MD5 hash. I spent quite sometime playing around with the user-agent on burp, only to realize later i just had to decode it 🤦🏾‍♂️.

Using this site, i managed to get the second flag as shown below.

[MD5Hashing](https://md5hashing.net/)

However, it took quite sometime to decode 😫and i had to use to use the search tab to try find if the hash was found in the database.

![easy](/assets/img/Posts/EasyPeasy/Untitled%208.png)

Oooh, looks like we got a hit. 😀 Lets have a look at it.

![easy](/assets/img/Posts/EasyPeasy/Untitled%209.png)

We now have the second flag 🚩

`flag{1m_s3c0nd_fl4g}`

## Crack the hash with easypeasy.txt, What is the flag 3?

This was very easy. We found `/index.html` and upon visiting it, we're welcomed with the default Apache page. Well, at first i just ignored it but taking a closer look, we can easily see the flag.

> _I did not quite understand the question asking us to crack the hash with easypeasy.txt file that we downloaded. I'd be interested to know how you guys solved that._

![easy](/assets/img/Posts/EasyPeasy/Untitled%2010.png)

![easy](/assets/img/Posts/EasyPeasy/Untitled%2011.png)

`flag{9fdafbd64c47471a8f54cd3fc64cd312}`

## What is the hidden directory?

Since i got a hit on the third flag, i decided to inspect the page source.

> Some of the keywords i like looking for in a web CTF once i view the open source include: `/img` , `==` for base64 strings, `hidden` , `theme`, `version` if i wanted to check what version of a CMS is running or plugins

In this scenario, i found something interesting while using the `hidden` keyword

![easy](/assets/img/Posts/EasyPeasy/Untitled%2012.png)

We can see on line 194 that the string has been encoded with _ba...._ At least i have an idea it could be any of the base x notations

![easy](/assets/img/Posts/EasyPeasy/Untitled%2013.png)

After trying the above notations, i found out that it was actually base62 encoded. We then get our hidden directory

![easy](/assets/img/Posts/EasyPeasy/Untitled%2014.png)

`/n0th1ng3ls3m4tt3r`

## Using the wordlist that provided to you in this task crack the hash what is the password?

Visiting the hidden directory, gives us a matrix kinda vibe. So i opened the developers tools (**ctrl+shift+i**) and begun inspecting the code. We find an image called _binarycodepixabay.jpg_ embedded. This image could be of interest to perform some stenography.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2015.png)

We also find a string which is hardly noticeable.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2016.png)

> Here's a nifty little trick i learnt from one of my friend @dwambia write-up sometime back. Whenever you get a page with a static image, hold `**ctrl+a`\*\* to check whether there is hidden text or a string hidden. 😉

For example in this case:

![easy](/assets/img/Posts/EasyPeasy/Untitled%2017.png)

Inorder to determine what kind of hash it was, i used **hashid** tool on my kali

![easy](/assets/img/Posts/EasyPeasy/Untitled%2018.png)

We can see we have several suggestions. I saved the hash to a file and used **john** to crack the hash.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2019.png)

We now have the password required for this question. The hash type used was _gost._

Alternatively, we can try crack the hash using the site we used earlier. Again, this takes a really long time to crack 😫 🤦🏾‍♂️

![easy](/assets/img/Posts/EasyPeasy/Untitled%2020.png)

Since we were successful the first time searching for the md5 hash, we can try searching this one.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2021.png)

We actually got some results on the same. Lets check it out

![easy](/assets/img/Posts/EasyPeasy/Untitled%2022.png)

> \*_\*\*I'm not sure if that's cheating, but as long as we have the flag_ 💁🏾‍♂️

`mypasswordforthatjob`

## What is the password to login to the machine via SSH?

Ok, now we a password that seems to give a hint, what other job though? After sometime of thinking, i remembered i was to get the embedded image and perform some steg on it. Lets go ahead and try that.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2023.png)

Nice...So after downloading the image and using the password we cracked, i extracted a secret text file which had the ssh username to use and password which looks like binary its encoded.

Using Cyberchef, we can convert the binary back to its raw format.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2024.png)

`iconvertedmypasswordtobinary`

## What is the user flag?

We can now ssh into the machine since we have both the username and password and grab the user flag in the home directory.

> ⚠️Dont forget ssh is running on port `6498`

![easy](/assets/img/Posts/EasyPeasy/Untitled%2025.png)

Looking at the user flag, we get a not that it has been **rotated.** Right away, i figured this could be ROT13 cipher

> ROT13 is a simple letter substitution cipher that replaces a letter with the 13th letter after it in the alphabet.
> _~**Source** - [Wikipedia](https://en.wikipedia.org/wiki/ROT13)_

We can easily decode the flag on our terminal as follows:

```bash
boring@kral4-PC:~$ cat user.txt | tr '[a-zA-Z]' '[n-za-m][N-ZA-M]'
Ffre Qynt [hg Tg Drrzf Hebat Wvxr Tg`f Cbgngrq Ze Dbzrguvat
flag{n0wits33msn0rm4l}
boring@kral4-PC:~$
```

or you can also use:

[ROT Cipher - Rotation - Online Rot Decoder, Solver, Translator](https://www.dcode.fr/rot-cipher)

![easy](/assets/img/Posts/EasyPeasy/Untitled%2026.png)

`flag{n0wits33msn0rm4l}`

## What is the root flag?

After performing some basic recon on the machine, i got nothing of interest. So i always like to upload [**linepeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) & [**pspy64**](https://github.com/DominicBreuker/pspy) on the target machine to search for possible paths to escalate privileges and monitor processes running respectively.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2027.png)

Once uploaded, make sure you make them executable

> `chmod +x linepeas.sh chmod +x pspy64`

Running the linepeas script first, we see a process running as root

![easy](/assets/img/Posts/EasyPeasy/Untitled%2028.png)

Lets now use pspy to monitor how long the script takes to execute

```bash
boring@kral4-PC:~$ ./pspy64
pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855

     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░
                   ░           ░ ░
                               ░ ░

Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scannning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2021/09/23 15:23:56 CMD: UID=0    PID=1      | /sbin/init splash
2021/09/23 15:24:01 CMD: UID=0    PID=2424   | sudo bash .mysecretcronjob.sh
2021/09/23 **15:24:01** CMD: UID=0    PID=2423   | /bin/sh -c    cd /var/www/ && sudo bash .mysecretcronjob.sh
2021/09/23 15:24:01 CMD: UID=0    PID=2422   | /usr/sbin/CRON -f
2021/09/23 15:24:01 CMD: UID=0    PID=2425   | bash .mysecretcronjob.sh
2021/09/23 15:25:01 CMD: UID=0    PID=2428   | sudo bash .mysecretcronjob.sh
2021/09/23 **15:25:01** CMD: UID=0    PID=2427   | /bin/sh -c    cd /var/www/ && sudo bash .mysecretcronjob.sh
2021/09/23 15:25:01 CMD: UID=0    PID=2426   | /usr/sbin/CRON -f
```

We can see that the script runs every one minute. Looking at the contents of the script, we see that we have write permission to it. We can add a reverse shell to the script and setup a netcat listener that will get a shell after the script executes.

You can use this site by [\*\*@0dayCTF](https://twitter.com/0dayCTF)\*\* to generate your reverse shell. There tons of them

[Online - Reverse Shell Generator](https://www.revshells.com/)

You only need to add your `IP` address and a `Port` of your liking and select what type of reverse shell you want.

![easy](/assets/img/Posts/EasyPeasy/Untitled%2029.png)

![easy](/assets/img/Posts/EasyPeasy/Untitled%2030.png)

`flag{63a9f0ea7bb98050796b649e85481845}`

# Lessons and tips:

1. I learnt how to decode various kinds of strings and hashes.
2. Never under estimate the power of recon. Dig deeper. TRY HARDER.
3. For this room, they recommended one to use [**gobuster**](https://github.com/OJ/gobuster) to enumerate directories. However, feel free to use other tools for practice.

[ffuf](https://github.com/ffuf/ffuf)

[wfuzz](https://github.com/xmendez/wfuzz)

[dirsearch](https://github.com/maurosoria/dirsearch)

[dirb](https://github.com/v0re/dirb)

[feroxbuster](https://github.com/epi052/feroxbuster)

Thanks for reading my write-up. Again this was my first write-up. i would really appreciate it if i got your feedback on the same, like was it informative, too long, my grammar and use of terminologies was wrong?

Feel free to reach out to me on Twitter [**@oste_ke**](https://twitter.com/oste_ke)
