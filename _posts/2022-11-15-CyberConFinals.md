---

title: "R2D2 Backdoor - Memory Forensics"
date: 2022-11-15 01:09:33 +0300
author: oste
image: /assets/img/Posts/cyberconfinals.png
categories: [CTF-TIME]
tags: [volatility, r2d2 backdoor]
---

Hey all and welcome once more to my blog. This past weekend i had the privilege to create a forensics challenge for the CyberCon Finals CTF which was held at USIU University. Top 4 Proffesional teams and 10 University teams battled out different challenges ranging from: Web, Forensics, Pwn, Reverse Engineering. Anyway, i didn't have enough time to properly setup a challenge from scratch, so i resolved to create some questions on a memory sample containing some nasty backdoor i was analyzing sometime back. I was happy to see folks solve most of the challenges and almost comming to the same conclusion i had before. With that said, i will be sharing my thought process on how the challenges were meant to be solved.

---

So, what is a memory dump? Simply put, a memory dump (also known as a core dump or system dump) is a snapshot capture of computer memory data from a specific instant. A memory dump can contain valuable forensics data about the state of the system before an incident such as a crash or security compromise.

> _Memory forensics can provide unique insights into runtime system activity, including open network connections and recently executed commands or processes. In many cases, critical data pertaining to attacks or threats will exist solely in system memory – examples include network connections, account credentials, chat messages, encryption keys, running processes, injected code fragments, and internet history which is non-cacheable. Any program – malicious or otherwise – must be loaded in memory in order to execute, making memory forensics critical for identifying otherwise obfuscated attacks._ ~Source: [DigitalGuardian](https://digitalguardian.com/blog/what-are-memory-forensics-definition-memory-forensics)

First, you unzip the provided memory sample. Password: _infected_

To answer the first question, you were only required to get the md5sum of the `.vmem` file. This was as easy as:

```bash
➜  md5sum malware.vmem
cb47af710c7ab59e43d1396aa7ce950c  malware.vmem
```

Moving on, we can use a tool like [volatility](https://github.com/volatilityfoundation/volatility) to analyze the memory dump.

Other tools you can explore on memory forensics include:

- [Rekall Framework](https://www.rekall-forensic.com/)
- [Redline](https://fireeye.market/apps/211364)

First, you need to find more information about the dump you are analying such as the image profile to use. To do so, you can use the `imageinfo` plugin or `kdbgscan`.

```bash
➜  python2 vol.py -f malware.vmem imageinfo
Volatility Foundation Volatility Framework 2.6.1

INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (/home/kali/Desktop/CyberCON/malware.vmem)
                      PAE type : PAE
                           DTB : 0x319000L
                          KDBG : 0x80544ce0L
          Number of Processors : 1
     Image Type (Service Pack) : 2
                KPCR for CPU 0 : 0xffdff000L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2011-10-10 17:06:54 UTC+0000
     Image local date and time : 2011-10-10 13:06:54 -0400
```

With that determined, you can begin investigating the processes running on the target system by using the `pslist` plugin.

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 pslist
Volatility Foundation Volatility Framework 2.6.1
Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit
---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0x819cc830 System                    4      0     55      162 ------      0
0x81945020 smss.exe                536      4      3       21 ------      0 2011-10-10 17:03:56 UTC+0000
0x816c6020 csrss.exe               608    536     11      355      0      0 2011-10-10 17:03:58 UTC+0000
0x813a9020 winlogon.exe            632    536     24      533      0      0 2011-10-10 17:03:58 UTC+0000
0x816da020 services.exe            676    632     16      261      0      0 2011-10-10 17:03:58 UTC+0000
0x813c4020 lsass.exe               688    632     23      336      0      0 2011-10-10 17:03:58 UTC+0000
0x81772ca8 vmacthlp.exe            832    676      1       24      0      0 2011-10-10 17:03:59 UTC+0000
0x8167e9d0 svchost.exe             848    676     20      194      0      0 2011-10-10 17:03:59 UTC+0000
0x817757f0 svchost.exe             916    676      9      217      0      0 2011-10-10 17:03:59 UTC+0000
0x816c6da0 svchost.exe             964    676     63     1058      0      0 2011-10-10 17:03:59 UTC+0000
0x815daca8 svchost.exe            1020    676      5       58      0      0 2011-10-10 17:03:59 UTC+0000
0x813aeda0 svchost.exe            1148    676     12      187      0      0 2011-10-10 17:04:00 UTC+0000
0x817937e0 spoolsv.exe            1260    676     13      140      0      0 2011-10-10 17:04:00 UTC+0000
0x81754990 VMwareService.e        1444    676      3      145      0      0 2011-10-10 17:04:00 UTC+0000
0x8136c5a0 alg.exe                1616    676      7       99      0      0 2011-10-10 17:04:01 UTC+0000
0x815c4da0 wscntfy.exe            1920    964      1       27      0      0 2011-10-10 17:04:39 UTC+0000
0x813bcda0 explorer.exe           1956   1884     18      322      0      0 2011-10-10 17:04:39 UTC+0000
0x816d63d0 VMwareTray.exe          184   1956      1       28      0      0 2011-10-10 17:04:41 UTC+0000
0x8180b478 VMwareUser.exe          192   1956      6       83      0      0 2011-10-10 17:04:41 UTC+0000
0x818233c8 reader_sl.exe           228   1956      2       26      0      0 2011-10-10 17:04:41 UTC+0000
0x815e7be0 wuauclt.exe             400    964      8      173      0      0 2011-10-10 17:04:46 UTC+0000
0x817a34b0 cmd.exe                 544   1956      1       30      0      0 2011-10-10 17:06:42 UTC+0000
```

No process looks suspicious at this point. We can take this a notch higher and use the `pstree` plugin to print process list as a tree. This would make you understand which process triggered other processes.

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 pstree
Volatility Foundation Volatility Framework 2.6.1
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0x819cc830:System                                      4      0     55    162 1970-01-01 00:00:00 UTC+0000
. 0x81945020:smss.exe                                 536      4      3     21 2011-10-10 17:03:56 UTC+0000
.. 0x816c6020:csrss.exe                               608    536     11    355 2011-10-10 17:03:58 UTC+0000
.. 0x813a9020:winlogon.exe                            632    536     24    533 2011-10-10 17:03:58 UTC+0000
... 0x816da020:services.exe                           676    632     16    261 2011-10-10 17:03:58 UTC+0000
.... 0x817757f0:svchost.exe                           916    676      9    217 2011-10-10 17:03:59 UTC+0000
.... 0x81772ca8:vmacthlp.exe                          832    676      1     24 2011-10-10 17:03:59 UTC+0000
.... 0x816c6da0:svchost.exe                           964    676     63   1058 2011-10-10 17:03:59 UTC+0000
..... 0x815c4da0:wscntfy.exe                         1920    964      1     27 2011-10-10 17:04:39 UTC+0000
..... 0x815e7be0:wuauclt.exe                          400    964      8    173 2011-10-10 17:04:46 UTC+0000
.... 0x8167e9d0:svchost.exe                           848    676     20    194 2011-10-10 17:03:59 UTC+0000
.... 0x81754990:VMwareService.e                      1444    676      3    145 2011-10-10 17:04:00 UTC+0000
.... 0x8136c5a0:alg.exe                              1616    676      7     99 2011-10-10 17:04:01 UTC+0000
.... 0x813aeda0:svchost.exe                          1148    676     12    187 2011-10-10 17:04:00 UTC+0000
.... 0x817937e0:spoolsv.exe                          1260    676     13    140 2011-10-10 17:04:00 UTC+0000
.... 0x815daca8:svchost.exe                          1020    676      5     58 2011-10-10 17:03:59 UTC+0000
... 0x813c4020:lsass.exe                              688    632     23    336 2011-10-10 17:03:58 UTC+0000
 0x813bcda0:explorer.exe                             1956   1884     18    322 2011-10-10 17:04:39 UTC+0000
. 0x8180b478:VMwareUser.exe                           192   1956      6     83 2011-10-10 17:04:41 UTC+0000
. 0x817a34b0:cmd.exe                                  544   1956      1     30 2011-10-10 17:06:42 UTC+0000
. 0x816d63d0:VMwareTray.exe                           184   1956      1     28 2011-10-10 17:04:41 UTC+0000
. 0x818233c8:reader_sl.exe                            228   1956      2     26 2011-10-10 17:04:41 UTC+0000
```

At this point we see explorer.exe (PID: 1956) spawning `VMwareUser.exe` , `cmd.exe` ( _Suspicious_) , `VMwareTray.exe` & `reader_sl.exe` ( _Suspicious_). My thinking, The user probably opened a malicious pdf file that spawned cmd and finally Adobe reader to view it. We can use the `cmdscan` plugin to extract command history as shown:

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 cmdscan
Volatility Foundation Volatility Framework 2.6.1
**************************************************
CommandProcess: csrss.exe Pid: 608
CommandHistory: 0x11132d8 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 2 LastAdded: 1 LastDisplayed: 1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x4c4
Cmd #0 @ 0x4e1eb8: sc query malwar
Cmd #1 @ 0x11135e8: sc query malware
```

We see `sc query malware` command was executed.

> _The `sc` command is used to configure, query, stop, start, delete, and add system services on the Windows command line._

In our case, the command seems to be querying a service called `malware`. The first command entry looks like a typo though.

Similar to cmdscan the `consoles` plugin finds commands that attackers typed into cmd.exe or executed via backdoors. The major advantage to this plugin is it not only prints the commands attackers typed, but it collects the entire screen buffer (input and output). For instance, lets see what output was generated when `sc query malware` command was executed.

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 consoles
Volatility Foundation Volatility Framework 2.6.1
**************************************************
ConsoleProcess: csrss.exe Pid: 608
Console: 0x4e2370 CommandHistorySize: 50
HistoryBufferCount: 2 HistoryBufferMax: 4
OriginalTitle: %SystemRoot%\system32\cmd.exe
Title: C:\WINDOWS\system32\cmd.exe
AttachedProcess: cmd.exe Pid: 544 Handle: 0x4c4
----
CommandHistory: 0x1113498 Application: sc.exe Flags:
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x0
----
CommandHistory: 0x11132d8 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 2 LastAdded: 1 LastDisplayed: 1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x4c4
Cmd #0 at 0x4e1eb8: sc query malwar
Cmd #1 at 0x11135e8: sc query malware
----
Screen 0x4e2a70 X:80 Y:300
Dump:
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\Documents and Settings\Administrator>sc query malwar
[SC] EnumQueryServicesStatus:OpenService FAILED 1060:

The specified service does not exist as an installed service.


C:\Documents and Settings\Administrator>sc query malware

SERVICE_NAME: malware
        TYPE               : 1  KERNEL_DRIVER
        STATE              : 4  RUNNING
                                (STOPPABLE,NOT_PAUSABLE,IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

C:\Documents and Settings\Administrator>
```

More information about the service can be queried using the `svcscan` plugin. This plugin gives more detail to the running processes in the event that the analyst requires additional details such as the display name, binary path, or service type.

![image](https://user-images.githubusercontent.com/58165365/201180589-aab791b1-0387-4b12-b4e2-b6f58c67496d.png)

We see that the service is running. Next thing we can try establish if there were network connections. Using `connscan` plugin, we see

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 connscan
Volatility Foundation Volatility Framework 2.6.1
Offset(P)  Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ---
0x01a25a50 0.0.0.0:1026              172.16.98.1:6666          1956
```

We can detect listening sockets for any protocol (TCP, UDP, RAW, etc), using the `sockets` plugin.

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 sockets
Volatility Foundation Volatility Framework 2.6.1
Offset(V)       PID   Port  Proto Protocol        Address         Create Time
---------- -------- ------ ------ --------------- --------------- -----------
0x8177e3c0     1956   1026      6 TCP             0.0.0.0         2011-10-10 17:04:39 UTC+0000
0x81596a78      688    500     17 UDP             0.0.0.0         2011-10-10 17:04:00 UTC+0000
0x8166a008      964   1029     17 UDP             127.0.0.1       2011-10-10 17:04:42 UTC+0000
0x818ddc08        4    445      6 TCP             0.0.0.0         2011-10-10 17:03:55 UTC+0000
0x818328d8      916    135      6 TCP             0.0.0.0         2011-10-10 17:03:59 UTC+0000
0x81687e98     1616   1025      6 TCP             127.0.0.1       2011-10-10 17:04:01 UTC+0000
0x817517e8      964    123     17 UDP             127.0.0.1       2011-10-10 17:04:00 UTC+0000
0x81753b20      688      0    255 Reserved        0.0.0.0         2011-10-10 17:04:00 UTC+0000
0x8174fe98     1148   1900     17 UDP             127.0.0.1       2011-10-10 17:04:41 UTC+0000
0x81753008      688   4500     17 UDP             0.0.0.0         2011-10-10 17:04:00 UTC+0000
0x816118d8        4    445     17 UDP             0.0.0.0         2011-10-10 17:03:55 UTC+0000
```

Moving on, there was a question asking about the computer name and OS version. This information can be found using the `envars` plugin. Typically this will show the number of CPUs installed and the hardware architecture (though the kdbgscan output is a much more reliable source), the process's current directory, temporary directory, session name, computer name, user name, and various other interesting artifacts.

![image](https://user-images.githubusercontent.com/58165365/201176232-3a6a0f7a-dcae-43ab-847c-a4cfab4834a4.png)

But what really triggered PID 1956? My hypothesis was that the user opened suspicious PDF documents that led to cmd & Adobe reader spawning. We also saw a network connection linked to this process. Using the `iehistory` plugin we can recover fragments of IE history index.dat cache files. It can find basic accessed links (via FTP or HTTP), redirected links (--REDR), and deleted entries (--LEAK). It applies to any process which loads and uses the `wininet.dll` library, not just Internet Explorer. Typically that includes Windows Explorer and even malware samples. For more info about the same, you can read this [blog post](https://volatility-labs.blogspot.com/2012/09/howto-scan-for-internet-cachehistory.html)

![image](https://user-images.githubusercontent.com/58165365/201176806-c1823268-2d18-4b1c-9771-99b426841f26.png)

From the above, we see two suspicious pdf documents linked to the same process: `listener.pdf` & `Exploit.pdf`

Lets scan for malicious drivers loaded, and moreso any that contains the `malware` string or name.

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 driverscan | grep "malware"
Volatility Foundation Volatility Framework 2.6.1
Offset(P)          #Ptr #Hnd Start                            Size Service Key          Name         Driver Name
------------------ ---- ---- ------------------ ------------------ -------------------- ------------ -----------
0x0000000001a498b8        3        0 0xf9eb4000     0x1500 malware              malware      \Driver\malware
```

To extract a kernel driver to a file, use the `moddump` pluggin. Supply the output directory with -D or --dump-dir=DIR.

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 moddump -b 0xf9eb4000 --dump-dir=$PWD
Volatility Foundation Volatility Framework 2.6.1
Module Base Module Name          Result
----------- -------------------- ------
0x0f9eb4000 winsys32.sys         OK: driver.f9eb4000.sys
```

We can get its md5sum and check if its flagged by VT

```bash
➜  md5sum driver.f9eb4000.sys
63270d2354695cabde0baeb0aed60e2a  driver.f9eb4000.sys
```

From the [VT - driver.f9eb4000.sys Analysis](https://www.virustotal.com/gui/file/e0ffcd04beca62d18ef84250c7e39d5e54ed7e0151ab67a819abab92284a17ff), we find 53/71 vendors flagged this file as malicious. We also learn that it linked with R2D2 backdoor.

![image](https://user-images.githubusercontent.com/58165365/201525395-f37c1b31-5a1e-4b97-97dc-6ae6ecaf15da.png)

Running strings on the malicious driver, we find another interesting dll referenced. Doing a quick google search on the same, we still get hits on R2D2 backdoor.

![image](https://user-images.githubusercontent.com/58165365/201756656-4e1936e1-22c9-484f-8682-f23893f0a2a9.png)

Lets find out if the dll is used by `explorer.exe`. To display a process's loaded DLLs, use the `dlllist` plugin with the `-p` or `--pid` filter as shown below.

![image](https://user-images.githubusercontent.com/58165365/201187838-00279ddf-7ba0-46d2-af28-eef7516cd579.png)

From the output shown above, we see `mfc42ul.dll` has a wierd base address compared to the rest (0x10000000). What we can do is extract the DLL from the process's memory space and dump it to disk for analysis as shown below:

```bash
➜  python2 vol.py -f malware.vmem --profile=WinXPSP2x86 moddump -b 0x10000000 --dump-dir=$PWD
Volatility Foundation Volatility Framework 2.6.1
Module Base Module Name          Result
----------- -------------------- ------
0x010000000 UNKNOWN              OK: driver.10000000.sys
➜  md5sum driver.10000000.sys
4986f678160ccd516f2bd589719ea0c4  driver.10000000.sys
```

Doing a quick check on [VirusTotal](https://www.virustotal.com/gui/file/2b7bbea0b5f8f82c0597e1e710afa2b9f5a0acaf6f46a8fcc62ab103ecb6a319), we see 46/68 flagged this file as malicious.

![image](https://user-images.githubusercontent.com/58165365/201760672-31fefdc6-d807-4be0-be4f-df12792a5678.png)

I did some research on this backdoor and came across this report by the [Chaos Computer Club (CCC)](https://www.ccc.de/en/updates/2011/staatstrojaner), a long-established German hacker group that claimed that the German government may have unleashed a backdoor Trojan Horse program, allowing them to spy on its people. While running strings on the suspicious dll (mfc42ul.dll), i noticed some wierd string, `C3PO-r2d2-POE`. Starwars characters? I don't know.

![image](https://user-images.githubusercontent.com/58165365/201888100-13cb0c93-269c-4d9e-8bf5-1582f7ffda99.png)

From CCC's report, i learnt that the backdoor was dubbed "R2D2" due to the "C3PO-r2d2-POE" string located inside the Trojan. CCC claims that it allows the monitoring Internet activity, including the recording of MSN Messenger conversations, Skype calls and Yahoo messenger chat.

![image](https://user-images.githubusercontent.com/58165365/201194394-e8dc3d4d-53bf-4429-b6fd-3faa6ccbbac9.png)

That wraps up my analysis of the R2D2 backdoor, atleast for the context of the CTF. Upon completion of my analysis, I stumbled upon several blog posts documenting analysis of the same sample. Please check them out!

- [Invoke-IR - R2D2 Memory Sample Analysis ](https://www.invoke-ir.com/2013/08/r2d2-memory-sample-analysis.html)
- [ka1d0 - Analyzing Memory Dumps — R2D2 Malware](https://nikhilh20.medium.com/analyzing-memory-dumps-r2d2-malware-7e44d7320fae)
- [Skills that matter - Hunting R2D2 Malware](https://digitalitskills.com/volatility-forensic-analysis-r2d2-malware/)
- [evild3ad - Federal Trojan aka R2D2](https://evild3ad.com/1136/volatility-memory-forensics-federal-trojan-aka-r2d2/)

If anyone has additional details that i might have missed, or has any feedback to improve my methodology it would be greatly appreciated.

---

# Questions

1. What is the md5sum of the memory sample (50pts)

`ccke{cb47af710c7ab59e43d1396aa7ce950c}`

2. What is the computer name of the machine captured (50pts)

`ccke{GENERALLEE}`

3. What is the name of the OS from which the memory sample was captured from? (50pts)

`ccke{Windows_NT}`

4. What process triggered a series of suspicious processes? (50pts)

`ccke{1956}`

5. What Remote Address and Port was the victim machine communicating with? (50pts)

`ccke{172.16.98.1:6666}`

6. There are two suspicious PDFs that were likely opened. Can you find their names? (50pts)

`ccke{listener.pdf,Exploit.pdf}`

7. What is the md5sum of the malicious dll? (300pts)

`ccke{4986f678160ccd516f2bd589719ea0c4}`

8. What is the md5sum of the malicious driver? (300pts)

`ccke{63270d2354695cabde0baeb0aed60e2a}`

9. What is the name of this backdoor? (100pts)

`ccke{R2D2}` or `ccke{r2d2}`
