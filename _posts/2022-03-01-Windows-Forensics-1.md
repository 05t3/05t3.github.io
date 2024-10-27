---

title: "Windows-Forensics-1"
date: 2022-02-27 01:09:33 +0300
author: oste
image: /assets/img/Posts/windowsforensics1.png
categories: [Tryhackme, Informational]
tags: [RID, Registry Explorer, NTUSER.DAT, Registry Hive, windows, Forensics]
---

| Room         | [Windows Forensics 1](https://tryhackme.com/room/windowsforensics1)                                    |
| ------------ | ------------------------------------------------------------------------------------------------------ |
| Room Creator | [umairalizafar](https://tryhackme.com/p/umairalizafar), [tryhackme](https://tryhackme.com/p/tryhackme) |

# Scenario:

> One of the Desktops in the research lab at Organization X is suspected to have been accessed by someone unauthorized. Although they generally have only one user account per Desktop, there were multiple user accounts observed on this system. It is also suspected that the system was connected to some network drive, and a USB device was connected to the system. The triage data from the system was collected and placed on the attached VM. Can you help Organization X with finding answers to the below questions?

**Note:** _When loading registry hives in RegistryExplorer, it will caution you that the hives are dirty. This is nothing to be afraid of. You just need to point RegistryExplorer to the .LOG1 and .LOG2 files with the same filename as the registry hive. It will automatically integrate the transaction logs and create a 'clean' hive. Once we tell RegistryExplorer where to save the clean hive, we can use that for our analysis and we won't need to load the dirty hives anymore. RegistryExplorer will guide you through this process._

# Questions

### How many user created accounts are present on the system?

In a windows system, if you want to get user account information, login information, and group information, you can analyse the `SAM` hive using a tool like Registry explorer. In this case, you can navigate to following location `SAM\Domains\Account\Users` as shown in the screenshot attached. If we look at the RID(Relative identifiers) of the users present we can affirm that there are 3 created accounts.

![image](https://user-images.githubusercontent.com/58165365/155883803-ded8d015-b2e0-4a2d-bd5b-fe1ae0009980.png)

> Relative identifier (RID) is a variable length number that is assigned to objects at creation and becomes part of the object's Security Identifier (SID) that uniquely identifies an account or group within a domain. Any group or user that is not created by default will have a Relative ID of `1000` or greater.

`3`

### What is the username of the account that has never been logged in?

Looking at the 5th column, we can see that user `thm-user2` has never logged in.

![image](https://user-images.githubusercontent.com/58165365/155883989-bee91119-ee89-498d-ad9d-f20afe8b2cae.png)

`thm-user2`

### What's the password hint for the user THM-4n6?

11th column easily gives us the user's password hint

![image](https://user-images.githubusercontent.com/58165365/155883872-4c77d9ca-2827-47af-a2f3-8a17d9e28690.png)

`count`

### When was the file 'Changelog.txt' accessed?

Windows maintains a list of recently opened files for each user. This information is stored in the `NTUSER hive` and can be found on the following location: `C\Users\<user>`.

In this case, we are provided with some triage data located in `C:\Users\THM-4n6\Desktop\triage`. Since we saw `THM-4n6` is an active user, and belongs to the Administators group, we can get their `NTUSER.DAT` located in `C:\Users\THM-4n6\Desktop\triage\C\Users\THM-4n6` and load it to RegistryExplorer.

The information we are interested in can be found in `SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` as shown in the screenshot attached.

![image](https://user-images.githubusercontent.com/58165365/155885782-a3d3a21b-1430-4370-9a70-772b9ece6b2f.png)

`2021-11-24 18:18:48`

### What is the complete path from where the python 3.8.2 installer was run?

Windows keeps track of applications launched by the user using Windows Explorer for statistical purposes in the User Assist registry keys. These keys contain information about the programs launched, the time of their launch, and the number of times they were executed.However, programs that were run using the command line can't be found in the User Assist keys. The User Assist key is present in the NTUSER hive, mapped to each user's GUID. We can find it at the following location: `SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count`

![image](https://user-images.githubusercontent.com/58165365/155886498-28362e83-7395-4290-ae8e-199f41a48bfc.png)

> _GUIDs are used to identify user accounts, documents, software, hardware, software interfaces, sessions, database keys and other items._

We can then see the complete path in which the python installer was run is `Z:\setups\python-3.8.2.exe`

### When was the USB device with the friendly name 'USB' last connected?

To score this question, you first need to identify connected drives on the system. The device name of the connected drive can be found at the following location:

`SOFTWARE\Microsoft\Windows Portable Devices\Devices`

![image](https://user-images.githubusercontent.com/58165365/155884269-c75a2368-9423-486f-961e-7a5b784fd83d.png)

In this case, we see the USB device with the friendly name 'USB'. Take note of its GUID.

In order to get more information about the USB such as vendor id, product id, version of the USB device & time the devices were plugged into the system, we can get the information on `SYSTEM\CurrentControlSet\Enum\USBSTOR`. Comparing the GUID we saw earlier with the Disk ID, we can acertain that `USB` is the one in the first row alongside the last connected timestamp.

![image](https://user-images.githubusercontent.com/58165365/155884462-977006e0-56f2-469a-b626-782ecea9658a.png)

`2021-11-24 18:40:06`

Thats brings me to the end of the walkthrough, if you need a cheetsheet on Windows-Registry forensics, i'll leave a cheat sheet compiled by the THM crew.

![image](https://user-images.githubusercontent.com/58165365/157232143-3c8785ec-164b-4843-bde8-9d9a22350159.png)

# Resources

- [Relative identifier](https://en.wikipedia.org/wiki/Relative_identifier)
- [Security Identifier](https://en.wikipedia.org/wiki/Security_Identifier)
- [SANS - Eric Zimmerman's tools Cheat Sheet](https://www.sans.org/posters/eric-zimmerman-tools-cheat-sheet/)
- [SANS - Windows Forensic Analysis](https://www.sans.org/posters/windows-forensic-analysis/)
