---
title: "GladiaTOR by Spiro CTF Walkthrough"
date: 2025-01-25 01:09:33 +0300
author: [oste]
description: This blog features writeups for the Hacked & Recovered challenges from the GladiaTOR by Spiro CTF.
image: /assets/img/Posts/Gladiators by Spiro.png
categories: [CTF-TIME]
tags:
  [Malware, forensics,recycle bin, ransomware, Procmon, sandbox, pml, Sysinternals, rhaegal, drogon, dispci.exe, schtasks, infpub.dat, rundll32.exe, Bad Rabbit, Petya, Mitre, S0606, FTK Imager, MAC, RBCmd, Rifiuti2]
---


I had the opportunity to create forensic challenges for the GladiaTOR by Spiro CTF, hosted by CTFRoom on January 24th, 2025. The two challenges focused on recycle bin forensics and analyzing malicious process activity. In this blog, I’ll walk you through the solutions and hope you find it insightful!

## Hacked

**Challenge Description**

> An employee at an electric vehicle company fell victim to a suspicious email containing an attachment. Upon opening it, the attachment triggered a compromise across the network. A sample of the malicious file was analyzed in a sandbox environment, and you now have access to a log file capturing Registry activity, File system changes, Network traffic, and Process & Thread behavior. Can you uncover the details of this incident?


### Question 1 

Can you investigate the suspicious process, its PID and time of execution? 


This challenge was inspired by a recent ransomware investigation lab I did on Cyberdefenders. I figured instead of providing players with the actual binary (which one might infect themselves), why not nuke it in a controlled sandbox and collect its execution activity. I also wanted players to have a feel of basic malware analysis using tools like Procmon. 

That being said, you are given a `.pml` file, which is essentially Procmon's output format. You are expected to use the tool, which is part of the Sysinternals toolkits. You can grab a binary [from Microsofts Page.](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)

Upon execution, it will start capturing Realtime activity on your system. However, in this case, you want to load the provided dump. Simply:

![image](https://gist.github.com/user-attachments/assets/31c7895e-874e-4fcf-9428-7e7dc9e5f017)

At first glance, it might look complicated but an easier way to navigate or understand the Process activity, you can use the process tree functionality for a better view.

![image](https://gist.github.com/user-attachments/assets/299374f3-6ac5-4e89-83a2-1ed943a24d03)

Scrolling to the bottom, you will note a wierd process named `Urgent Contract Action.pdf.exe` with a PID of `7852`  

`.pdf.exe?` 🤔 We can also see some `cmd.exe` processes executing suspicious commands.

![image](https://gist.github.com/user-attachments/assets/57101264-01ac-4259-9c27-ee5ced485735)

Having this in mind, we can filter out events for this process and its children by right clicking on it and selecting that option.

![image](https://gist.github.com/user-attachments/assets/d29cdb6a-bf49-4647-8486-af7e57395561)

Take note when the process was first started: `1/24/2025 4:48:46 AM`. Looking through other events that might have been executed around the same time or after, I also noted . So we can also right click on it and add the process and its children to filter.


![image](https://gist.github.com/user-attachments/assets/ce647139-9fcf-4313-a50f-6c71b400301b)

Going back to the main page, we now have refined events that we can query easily.

![image](https://gist.github.com/user-attachments/assets/6cd7ac94-073a-4151-a9cd-30315a87a039)

Flag format: *processName_processId_startTime*

Flag: `Urgent Contract Action.pdf.exe_7852_01/24/2025 04:48:46 AM`

-----

### Question 2 & 3

- A scheduled task was deleted and then re-created shortly. Can you identify the task name?
- Another scheduled task was created. can you identify the taskname & when was it meant to run? 

Back on the process tree summary, we saw 3 unique scheduled tasks.

![image](https://gist.github.com/user-attachments/assets/d6a7dda8-2ffa-456b-a87e-ffa3e495877f)

- The first command attempts to delete a pre-existing scheduled task named `rhaegal`.
- The second creates a new scheduled task named `rhaegal` to run a program (`dispci.exe`) with elevated privileges whenever the system starts
- The last command creates a one-time task named **`drogon`** that will force the system to restart immediately at **05:06:00**.

```powershell
schtasks  /Delete /F /TN rhaegal

schtasks  /Create /RU SYSTEM /SC ONSTART /TN rhaegal /TR "C:\Windows\system32\cmd.exe /C Start \"\" \"C:\Windows\dispci.exe\" -id 3287817739 && exit"

schtasks  /Create /SC once /TN drogon /RU SYSTEM /TR "C:\Windows\system32\shutdown.exe /r /t 0 /f" /ST 05:06:00
```
{: .nolineno }

Flag 2: `rhaegal`

Flag format: *taskName_hh:mm:ss*

Flag 3: `drogon_05:06:00`

-----

### Question 4. 

- The suspicious process drops a file upon execution. Provide the full command of how this file is executed

While still observing the process tree, we can see that `rundll32.exe` - a  windows utility used to execute functions exported by a DLL, is likely executing a specific function (`#1`) from a suspicious file (`infpub.dat`), which may contain malicious code.

![image](https://gist.github.com/user-attachments/assets/2433613a-e399-45ae-a0d5-c9d0068ee0f5)


```powershell
C:\Windows\system32\rundll32.exe C:\Windows\infpub.dat,#1 15
```
{: .nolineno }

----

### Question 5. 

-  Using crowd sourced intelligence and based on what you have so far, what is the MITRE Software ID for associated with this malware

From the information gathered so far, a quick online search may reveal malware variants associated with these findings. Here, we learn that this activity could potentially indicate the presence of `Bad Rabbit` ransomware, a strain that emerged in 2017 and is believed to be a variant of Petya. Additional details can be found on [Malpedia](https://malpedia.caad.fkie.fraunhofer.de/details/win.eternal_petya) .

![image](https://gist.github.com/user-attachments/assets/6f1e73d8-4898-426b-91f3-1f01fea04c44)

If you search for its Mitre Software ID, you get [S0606](https://attack.mitre.org/software/S0606/)

![image](https://gist.github.com/user-attachments/assets/e5381bb7-f48a-41fc-b7bb-d33216022ac6)

Reading through some of the techniques used, we can spot 3 that we identified.

![image](https://gist.github.com/user-attachments/assets/02ab9356-637a-4352-8387-6bdfa90dfa57)

I would encourage you to identify other techniques used from the process dump file shared.

-----


## Recovered

**Challenge Description**

> Management suspects that the employee may have deleted critical files and requires your assistance to identify and recover them. You’ve been provided with a disk image of the employee’s PC to analyze. Can you uncover and identify a file that was likely recovered?

Flag format: *$Ixxxxxxxx_xxxxxxxxxxxxx.xxx* where DeletedFileName_ActualFileName

Flag: `$ITJ0J0O.pdf_conf_classified_navydoc.pdf`

-----

This forensic challenge was designed to introduce players to the field of recycle bin forensics. Initially planned as a two-part series, only the first part was released due to time constraints, with hopes that the second part will be included in future CTFs for participants to enjoy. For this challenge, you are provided with a disk image for analysis, which can be loaded using tools like FTK Imager as demonstrated below:

Simply, Click on the first icon to select the data source and choose a logical file:

![image](https://gist.github.com/user-attachments/assets/42c63bfa-55bf-4132-b1a4-66e0a9bac125)

Choose image location.

![image](https://gist.github.com/user-attachments/assets/d2915ce3-b800-4112-b44c-a38c58d43bff)

Once imported, you get:

![image](https://gist.github.com/user-attachments/assets/33643938-282b-46d9-9586-1b98fea0edc8)

Particularly we're interested in the recycle bin directory . Right click on it and select `Export Files` option. Choose path to save the artifacts and click Ok.

![image](https://gist.github.com/user-attachments/assets/ff641c70-9b6f-4725-8099-5496aad9c93d)

----

Windows Recycle Bin is a location on a Windows computer that temporarily stores deleted files.

The Recycle Bin is stored at `C:\$Recycle.Bin\`on Windows Vista/7/8/10 systems, and `C:\RECYCLER` on Windows XP systems.

When files are deleted on Windows Vista/7/8/10, through the means of clicking on a file and pressing the Delete key or right-clicking and selecting Delete, the file's information gets separated into **two** separate files inside the Recycle Bin directory.

Windows Vista/7/8/10 implements separate directories inside the `C:\$Recycle.Bin` directory for each user on the system, notated by that user's Security Identifier (**SID**). These directories will be hidden by default.

For example, there may be two users on the system, so each will have their own Recycle Bin directory inside `C:\$Recycle.Bin,` which can be demonstrated by running "`dir /ah`" in Command Prompt to list all the hidden folders inside the `C:\$Recycle.Bin` directory.

![image](https://gist.github.com/user-attachments/assets/f6691464-082e-4441-bd30-fa67a305dcaf)

Once a file is deleted, it is moved to the respective recycle bin folder for that user. Additionally, the file is split into two files: an `$I` file and a `$R` file, each with its own purpose. `$I` or `$R` is then followed by 6 arbitrary alphanumeric characters and the file extension of the original file.

- The `$R` Raw data file is nearly identical to the original file, the only difference being the file name. This can be proved by comparing hashes of the original file to the associated `$R` file once it is deleted.
- The `$I` file only contains the **metadata** information of the original file, which changes as the file is deleted.

Timestamps are an important part of the analysis of `$R` and `$I` files because the `MAC` (Modified, Accessed, Created) timestamps of each file provide additional information about the original file

| Timestamp | $R File                | $I File          |
| --------- | ---------------------- | ---------------- |
| Created   | Original Creation time | Time of deletion |
| Modified  | Original Modified time | Time of deletion |
| Accessed  | Original Accessed time | Time of deletion |


By issuing the `dir` command inside a user's directory , the individual `$R` and `$I` files are visible:

*Where:*

- Yellow represents the `$I` files
- Green represents the `$R` files

![image](https://gist.github.com/user-attachments/assets/73da544b-92a9-4f1f-aef7-d0901ae1c936)

When investigating Recycle Bin artifacts, there are a variety of tools that can assist in making investigation easier and more efficient. Tools such as **RBCmd** and **Rifiuti** can be used to optimize this process.

**RBCmd** can be used to output a file's size, name, path, and deletion time (in UTC) through the command line.

- [**Rifiuti2**](https://abelcheung.github.io/rifiuti2/)
- [Eric Zimmerman's **RBCmd** tool](https://ericzimmerman.github.io/#!index.md)

**RBCmd** Help option:

![image](https://gist.github.com/user-attachments/assets/22a3fe2a-2ad7-4969-b1e1-3b325e68ded2)

**RBCmd**  Usage:

```powershell
PS C:\Users\analyst\Downloads\Exported Artifacts> ..\RBCmd\RBCmd.exe -d '.\$Recycle.Bin'
RBCmd version 1.6.1.0

Author: Eric Zimmerman (saericzimmerman@gmail.com)
https://github.com/EricZimmerman/RBCmd

Command line: -d .\$Recycle.Bin
Warning: Administrator privileges not found!


Looking for files in .\$Recycle.Bin

Found 15 files. Processing...

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$I1LGQAB.mp3

Version: 2 (Windows 10/11)
File size: 114 (114B)
File name: C:\Users\user1\Music\TheBeatles.mp3
Deleted on: 2021-06-12 06:39:43


Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$I33DHXT

Version: 2 (Windows 10/11)
File size: 190 (190B)
File name: C:\Users\user1\Pictures\Saved Pictures
Deleted on: 2021-06-12 06:34:12

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$I4474FX.exe

Version: 2 (Windows 10/11)
File size: 27,278,840 (26MB)
File name: C:\Users\user1\Downloads\nmap-7.91-setup.exe
Deleted on: 2021-06-11 18:35:29

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$I5MGV12.txt

Version: 2 (Windows 10/11)
File size: 114 (114B)
File name: C:\Users\user1\Music\TheBeatles.txt
Deleted on: 2021-06-12 06:39:27

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$I79CLUM.jpg

Version: 2 (Windows 10/11)
File size: 76,957 (75.2KB)
File name: C:\Users\user1\Pictures\frog.jpg
Deleted on: 2021-06-12 05:43:51

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$I97P89I.PNG

Version: 2 (Windows 10/11)
Deleted on: 2021-06-12 06:33:42

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$IC4H4NQ.html

Version: 2 (Windows 10/11)
File size: 50,464 (49.3KB)
File name: C:\Users\user1\Documents\FBI Records_ classified - Part 3 of 16.html

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$ID1B9B9.jpg

Version: 2 (Windows 10/11)
File size: 360,324 (351.9KB)
File name: C:\Users\user1\Pictures\Cat-Wearing-COVID-19-Mask.jpg
Deleted on: 2021-06-12 05:43:43

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$IFDZGFC.exe

Version: 2 (Windows 10/11)
File size: 61,382,664 (58.5MB)
File name: C:\Users\user1\Downloads\Wireshark-win64-3.4.6.exe
Deleted on: 2021-06-12 05:57:57

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$IKNLU1G

Version: 2 (Windows 10/11)
File size: 2,104,381 (2MB)
File name: C:\Users\user1\Pictures\images
Deleted on: 2021-06-12 06:37:52

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$IOPYMBO.txt

Version: 2 (Windows 10/11)
File size: 33 (33B)
File name: C:\Users\user1\Documents\Confidential1.txt
Deleted on: 2021-06-12 06:32:56

Source file: .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$ITJ0J0O.pdf

Version: 2 (Windows 10/11)
File size: 598,903 (584.9KB)
File name: C:\Users\user1\Documents\conf_classified_navydoc.pdf
Deleted on: 2021-06-12 06:33:28


Processed 12 out of 15 files in 0.2597 seconds


Failed files
  .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$I30
  .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$R5PGQR1\$I30
  .\$Recycle.Bin\S-1-5-21-3001983725-1252495303-1083844272-1001\$RKNLU1G\$I30
```
{: .nolineno }

The Recycle Bin acts as a repository for files that were at some point deleted. After a file lands in the Recycle Bin, it can either stay there until Windows removes it automatically, the user permanently deletes it, or the user recovers it. However, once a file is removed through one of these methods, there may be no apparent trace of the file from the Recycle Bin artifacts.

> **When a file is Recovered, Windows will reference the `$I` file to retrieve its original name and location, while it will conceptually move the `$R` file to the location identified with the name specified by the `$I` file, keeping the `$R` file's timestamps.** 
{: .prompt-tip }

However, the `$I` file that was generated does not get immediately moved or deleted from its place in the Recycle Bin.

**How can investigators use this information to locate the file that was recovered?**

Remember that the `$I` file stores most of the metadata about the file that was once deleted, so it should still include the same information even though the `$R` file was moved. **Most likely, when the file is Recovered, it will return to the location specified in the `$I `file.** Eg, in our example earlier, we had `$ITJ0J00.pdf` that wasn't highlighted. *Note that there is no` $R` file for it.*

![image](https://gist.github.com/user-attachments/assets/14d72d48-fab2-42dd-bd5b-81cd6bfa5202)

But Why? 

This one was recovered because only its `$I` file was present in the Recycle Bin.

![image](https://gist.github.com/user-attachments/assets/580332c7-ef89-486b-8937-7d9167033298)

![image](https://gist.github.com/user-attachments/assets/91a6afdd-d077-4624-861f-bb06020582d0)

Hence: Flag: `$ITJ0J0O.pdf_conf_classified_navydoc.pdf`
### Bonus

Files that are Permanently Deleted are not as easy to recover without a forensics tool's assistance. Using **FTK Imag**er, the `$Recycle.Bin` can be reviewed to see the artifacts that were left over from permanently deleting a file. In  FTK Imager we can see  the `$R` and `$I` files for the deleted TXT file, notated with a red "X".


![image](https://gist.github.com/user-attachments/assets/60da50ef-ed4f-4667-b67b-8a2971657f79)

We also have this two directories:

![image](https://gist.github.com/user-attachments/assets/59815019-c69f-406e-87c6-97d3d5a6ae33)

What is interesting about the contents of: `$RKNLU1G` & `$R5PGQR1` directories?

- `$RKNLU1G` is an entry for a directory that was deleted. However, the files inside the deleted folder do not follow the same `$R`... and $I... file format; they are saved with the original file names and information.

	![image](https://gist.github.com/user-attachments/assets/588fe5f0-9a54-4629-9801-75f366266a2c)

- `R5PGQR1` is another deleted folder, but it appears as being permanently deleted (deleted from the Recycle Bin) because it has an X over its icon, and all its contents are also deleted with an X and their original names.

	![image](https://gist.github.com/user-attachments/assets/9298e7bf-617e-4c38-9315-a51082e10596)