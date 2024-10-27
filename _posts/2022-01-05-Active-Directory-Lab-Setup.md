---

title: "Active Directory Lab Setup - (Mayor's Movement, Pivoting & Persistence Course Walkthrough)"
date: 2022-01-05 01:09:33 +0300
author: oste
image: /assets/img/Posts/AD1.png
categories: [Active Directory, MPP Course]
tags: [windows, active directory, Virtual Box]
---

---

# Creating network

While in Virtual Box, hold `Ctrl+G` to launch Preference. Alternatively, you can click the `File` tab and select Preference.

![image](https://user-images.githubusercontent.com/58165365/148050845-d60cb798-e5f5-48ac-81f1-04192487a0cc.png)

Then click the `Network` tab and click the green icon with the `+` sign. You need to create three networks namely: external,internal & secure. In my case, here are the CIDR assigned to each. Also ensure that after creating each network, you enable it by clicking the checkbox.

| Network Name | CIDR            |
| ------------ | --------------- |
| External     | 192.168.20.0/24 |
| Internal     | 192.168.30.0/24 |
| Secure       | 192.168.40.0/24 |

![image](https://user-images.githubusercontent.com/58165365/148051055-af294a2b-c19d-435e-bf19-27288240c3ef.png)

# Creating Virtual Machines

This segment is for people who might be new to creating Windows Virtual Machines. If you already know how to do this, you can skip to

Click on `New` icon

![image](https://user-images.githubusercontent.com/58165365/148179921-8c5b08ad-fdaa-46b8-bdbc-317051a88add.png)

Name your Vitual Machine and select the type of Operating system and version of OS you wish to setup.

![image](https://user-images.githubusercontent.com/58165365/148048997-2cf3f587-b6c1-4996-b06d-3ec26d173cfa.png)

Allocate the RAM which will be used by the VM. (_2GB is good for quick windows installation if you have minimal resources on your system_)

![image](https://user-images.githubusercontent.com/58165365/148049037-8f9673bf-3c90-4e11-8ddb-233f3cd386d9.png)

Choose `Create a virtual hard disk`

![image](https://user-images.githubusercontent.com/58165365/148049075-1283bf7b-883f-4b64-bb15-1297965cff1c.png)

Select a virtual disk image

![image](https://user-images.githubusercontent.com/58165365/148049108-4c0230c8-4f18-4e6e-b8f0-6543a2f511e8.png)

Select a Dynamically allocated hard disk file

![image](https://user-images.githubusercontent.com/58165365/148049166-299d7191-59ea-47e2-925c-746d5469e86b.png)

In this last step, you need to allocate the size of the hard disk the VM wil use. In this case, i'll assign 50Gb to each machine.

![image](https://user-images.githubusercontent.com/58165365/148049211-f8b3207c-e595-4945-aa0a-1b5a329c5ea7.png)

Do the same for the second Workstation and the Windows Server. I find it a good practice to group my VM's based on the lab exercise at hand. In this case, i've named my group `Pivoting Lab`. You can do so by right-clicking on a machine and click `Group`.

![image](https://user-images.githubusercontent.com/58165365/148049893-9df0da28-6686-4296-b46a-c1f4f3c90134.png)

# Os Installation

Boot the Windows Server and accept the EULA

![image](https://user-images.githubusercontent.com/58165365/148054538-765bb18a-4277-4199-9743-09f13e51dbcc.png)

Choose "Custom: Install Windows only (advanced)

![image](https://user-images.githubusercontent.com/58165365/148054690-3dc10f33-3b65-4566-86a0-8b330ec831ba.png)

Choose the drive and hit _Next_

![image](https://user-images.githubusercontent.com/58165365/148054778-d6278f2f-09bb-4916-a55f-a4e06ab20e1d.png)

After this step, the installation process begins. Once complete, you will be prompted to input the Administrators password. In this case and for lab purpose, we are gonna setup a "really secure passoword" of `Password123!` and hit next.

![image](https://user-images.githubusercontent.com/58165365/148060277-c559c6eb-aeb1-4d2b-b4e1-b080b5687a72.png)

Once you login, you will be welcomed by the Server Manager Dashboard.

![image](https://user-images.githubusercontent.com/58165365/148185241-791c27b9-9b10-4454-a982-a2f47b32b6e3.png)

Upto this point, i think we are good to go. Lets proceed to Setup our Windows 10 Workstations.

Choose your region.

![image](https://user-images.githubusercontent.com/58165365/148071261-fbb8d428-4898-478b-a567-53ba54c83d29.png)

Select the keyboard layout that suits you.

![image](https://user-images.githubusercontent.com/58165365/148071349-fc7c38ae-b04f-45ee-9aa5-237c49a80b7c.png)

For this lab setup, we'll go with _Set up for personal use_

![image](https://user-images.githubusercontent.com/58165365/148072978-8c2394e6-c2b9-4c6d-bccf-3f7a218db9b2.png)

You will be promted to add your account. In the bottom left corner, choose an offline account.

![image](https://user-images.githubusercontent.com/58165365/148073162-2924e29d-a97d-46e7-be58-41cd569b4d1e.png)

For now, we'll go with limited experience

![image](https://user-images.githubusercontent.com/58165365/148073238-c7cbfd68-c3a5-42f9-bbe5-836a10e54a31.png)

Input a username for this PC

![image](https://user-images.githubusercontent.com/58165365/148073489-b857d179-4af9-450a-9613-e2ce09b131e0.png)

Enter a password.

![image](https://user-images.githubusercontent.com/58165365/148073940-27919e5e-86e5-41fe-b667-4a3fbcd4a039.png)

Creds:

| Workstation | Username   | Password     |
| ----------- | ---------- | ------------ |
| 1           | s.chisholm | FallOutBoy1! |
| 2           | m.seitz    | Phi11i35@44  |

The next step, you need to choose three security questions, so go ahead and do that.

![image](https://user-images.githubusercontent.com/58165365/148074408-4f67c512-f546-4694-8dbc-95faf03b5d7a.png)

For privacy settings, we can go ahead and disable all as we don't really require them.

![image](https://user-images.githubusercontent.com/58165365/148074533-b54e1a96-0b88-4cc0-bc93-b3f24393115d.png)

And with that done...both workstations should take a moment to load and start.

We also need to ensure that both workstations and the server have the same time settings. To do so, we can head over to `Settings> Time & Language` and ensure that you change the `Time zone` to match your current region.

![image](https://user-images.githubusercontent.com/58165365/148100372-1fb87fd3-9b39-461a-b847-d70a10da80eb.png)

# Active Directory generation scripts

Before we can proceed setting up our DC, we need to create a snapshot of all VM's. I find this a good practice incase the script doesn't work as expected, or if it breaks the system, we can easily revert back to the initial state.

To create a snapshot, click the `Machine` tab and select Take Snapshot.

![image](https://user-images.githubusercontent.com/58165365/148185878-ecff6c05-fee4-4d6f-8499-5c0b2b86b338.png)

You will then be prompted to enter the snapshot name and a description which is optional.

![image](https://user-images.githubusercontent.com/58165365/148186192-2723f1dc-61f8-48f7-9eee-f39bed94011b.png)

Once done, click ok and the snapshot will be taken in a few minutes.

![image](https://user-images.githubusercontent.com/58165365/148186295-ffcea020-62fa-4cdb-833b-3774229ac361.png)

For the purpose of this course, rather than manually setting up the Domain Controller, we can use a set of Powershell scripts by [TheMayor/Joe Helle](https://twitter.com/joehelle) which will create a vulnerable environment for us to pentest and learn a few concepts which i will be covering in te next series of blogs. You can download the scripts from this github repo: [dievus/ADGenerator](https://github.com/dievus/ADGenerator) and follow along.

The first script we need to execute is `Invoke-ForestDeploy.ps1`

> This will install the Windows Active Directory Domain Services toolset and generate the actual domain. Follow the instructions on screen, making note of the domain name used as this will be needed later. The scripts are hardcoded for `mayorsec.local` , and any deviation from that domain name will likely break the `ADGenerator.ps1` functionality.

```powershell
PS C:\Users\Administrator\Desktop\ADGenerator-main> dir


    Directory: C:\Users\Administrator\Desktop\ADGenerator-main


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         1/5/2022  12:17 AM                images
-a----         1/5/2022  12:17 AM          18405 ADGenerator.ps1
-a----         1/5/2022  12:17 AM           2034 coursewordlist
-a----         1/5/2022  12:17 AM           1912 Invoke-ForestDeploy.ps1
-a----         1/5/2022  12:17 AM            951 nameGen.ps1
-a----         1/5/2022  12:17 AM           1902 README.md


PS C:\Users\Administrator\Desktop\ADGenerator-main> Set-ExecutionPolicy Unrestricted

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): Y
PS C:\Users\Administrator\Desktop\ADGenerator-main> . .\Invoke-ForestDeploy.ps1

Security warning
Run only scripts that you trust. While scripts from the internet can be useful, this script can potentially harm your computer. If you trust this script, use the Unblock-File cmdlet to allow the script to run without this warning
message. Do you want to run C:\Users\Administrator\Desktop\ADGenerator-main\Invoke-ForestDeploy.ps1?
[D] Do not run  [R] Run once  [S] Suspend  [?] Help (default is "D"): R
PS C:\Users\Administrator\Desktop\ADGenerator-main> Invoke-ForestDeploy -DomainName mayorsec.local

             ______                     __        ____             __
            / ____/___  ________  _____/ /_      / __ \___  ____  / /___  __  __
           / /_  / __ \/ ___/ _ \/ ___/ __/_____/ / / / _ \/ __ \/ / __ \/ / / /
          / __/ / /_/ / /  /  __(__  ) /_/_____/ /_/ /  __/ /_/ / / /_/ / /_/ /
         /_/    \____/_/   \___/____/\__/     /_____/\___/ .___/_/\____/\__, /
                                                        /_/            /____/
        Domain Deployment Script by TheMayor

        [*] Installing Windows AD Domain Services Toolset. [*]

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Active Directory Domain Services, Group P...


Toolset installed.


        [*] Generating the domain. Make note of the domain name for the ADGenerator Script to be ran after the controller is built. [*]
SafeModeAdministratorPassword: ************
Confirm SafeModeAdministratorPassword: ************

The target server will be configured as a domain controller and restarted when this operation is complete.
Do you want to continue with this operation?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): Y
WARNING: Windows Server 2019 domain controllers have a default for the security setting named "Allow cryptography algorithms compatible with Windows NT 4.0" that prevents weaker cryptography algorithms when establishing security channel
 sessions.

For more information about this setting, see Knowledge Base article 942564 (http://go.microsoft.com/fwlink/?LinkId=104751).

WARNING: This computer has at least one physical network adapter that does not have static IP address(es) assigned to its IP Properties. If both IPv4 and IPv6 are enabled for a network adapter, both IPv4 and IPv6 static IP addresses
should be assigned to both IPv4 and IPv6 Properties of the physical network adapter. Such static IP address(es) assignment should be done to all the physical network adapters for reliable Domain Name System (DNS) operation.

WARNING: A delegation for this DNS server cannot be created because the authoritative parent zone cannot be found or it does not run Windows DNS server. If you are integrating with an existing DNS infrastructure, you should manually
create a delegation to this DNS server in the parent zone to ensure reliable name resolution from outside the domain "mayorsec.local". Otherwise, no action is required.

WARNING: Windows Server 2019 domain controllers have a default for the security setting named "Allow cryptography algorithms compatible with Windows NT 4.0" that prevents weaker cryptography algorithms when establishing security channel
 sessions.

For more information about this setting, see Knowledge Base article 942564 (http://go.microsoft.com/fwlink/?LinkId=104751).

WARNING: This computer has at least one physical network adapter that does not have static IP address(es) assigned to its IP Properties. If both IPv4 and IPv6 are enabled for a network adapter, both IPv4 and IPv6 static IP addresses
should be assigned to both IPv4 and IPv6 Properties of the physical network adapter. Such static IP address(es) assignment should be done to all the physical network adapters for reliable Domain Name System (DNS) operation.

WARNING: A delegation for this DNS server cannot be created because the authoritative parent zone cannot be found or it does not run Windows DNS server. If you are integrating with an existing DNS infrastructure, you should manually
create a delegation to this DNS server in the parent zone to ensure reliable name resolution from outside the domain "mayorsec.local". Otherwise, no action is required.


Message        : Operation completed successfully
Context        : DCPromo.General.3
RebootRequired : False
Status         : Success


Restart the controller if not instructed.

```

Don't mind the warning messages. Once the Server restarts, you will notice that we are already in a domain called `mayorsec`

![image](https://user-images.githubusercontent.com/58165365/148214166-06590ba7-997b-4658-b893-65e7fec31d9d.png)

The next step is running the ADGenerator Script. Like we did in the previous step, you need to set the execution policy to unrestricted and invoke the ADGenerator script and run it with DomainName set to `mayorsec.local`.

But before executing the script, i browsed it to see what it does. Here are some of the stuff it does needed for the actual course.

- Group generation - Senior Management, IT Admins, Engineering, Sales
- Domain Information
- Renaming the domain controller to DC01
- Creates a new share called `Shared`
- Creates and adds users to groups
- Allow WinRM TCP 5985 To Domain Joined Systems
- Configuring GPO policies to enable PowerShell remoting on hosts.
- Creating ACL misconfigurations, Kerberoastable service, Administrative privilege delegation and modifying ASREP settings

Lets get into it

```powershell
PS C:\Users\Administrator\Desktop\ADGenerator-main> dir


    Directory: C:\Users\Administrator\Desktop\ADGenerator-main


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         1/5/2022  12:17 AM                images
-a----         1/5/2022  12:17 AM          18405 ADGenerator.ps1
-a----         1/5/2022  12:17 AM           2034 coursewordlist
-a----         1/5/2022  12:17 AM           1912 Invoke-ForestDeploy.ps1
-a----         1/5/2022  12:17 AM            951 nameGen.ps1
-a----         1/5/2022  12:17 AM           1902 README.md


PS C:\Users\Administrator\Desktop\ADGenerator-main> Set-ExecutionPolicy Unrestricted

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): Y
PS C:\Users\Administrator\Desktop\ADGenerator-main> . .\ADGenerator.ps1

Security warning
Run only scripts that you trust. While scripts from the internet can be useful, this script can potentially harm your computer. If you trust this script, use the Unblock-File cmdlet to allow the script to run without this warning
message. Do you want to run C:\Users\Administrator\Desktop\ADGenerator-main\ADGenerator.ps1?
[D] Do not run  [R] Run once  [S] Suspend  [?] Help (default is "D"): R
PS C:\Users\Administrator\Desktop\ADGenerator-main> Invoke-ADGenerator -DomainName mayorsec.local

            ___    ____     ______                           __
           /   |  / __ \   / ____/__  ____  ___  _________ _/ /_____  _____
          / /| | / / / /  / / __/ _ \/ __ \/ _ \/ ___/ __ `/ __/ __ \/ ___/
         / ___ |/ /_/ /  / /_/ /  __/ / / /  __/ /  / /_/ / /_/ /_/ / /
        /_/  |_/_____/   \____/\___/_/ /_/\___/_/   \__,_/\__/\____/_/
        Vulnerable Active Directory Domain Generator by The Mayor

        [*] Promoting Administrator to appropriate Domain Administrative roles required for the course. [*]
        [+] Promoting Administrator to Enterprise Administrator.
User Administrator is already a member of group Enterprise Admins.

More help is available by typing NET HELPMSG 3754.

        [+] Promoting Administrator to Domain Administrator.
User Administrator is already a member of group Domain Admins.

More help is available by typing NET HELPMSG 3754.

        [+] Promoting Administrator to Group Policy Creator Owners.
User Administrator is already a member of group Group Policy Creator Owners.

More help is available by typing NET HELPMSG 3754.

        [+] Promoting Administrator to Local Administrator (error output may occur - this is expected).
System error 1378 has occurred.

The specified account name is already a member of the group.

        [*] Administrative privilege delegation completed. [*]
        [*] Renaming the domain controller to DC01 [*]
WARNING: The changes will take effect after you restart the computer WIN-SOKFK09IVBB.


    Directory: C:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         1/5/2022   4:05 AM                Shared

AvailabilityType      : NonClustered
CachingMode           : Manual
CATimeout             : 0
ConcurrentUserLimit   : 0
ContinuouslyAvailable : False
CurrentUsers          : 0
Description           :
EncryptData           : False
FolderEnumerationMode : Unrestricted
IdentityRemoting      : False
Infrastructure        : False
LeasingMode           : Full
Name                  : Shared
Path                  : C:\Shared
Scoped                : False
ScopeName             : *
SecurityDescriptor    : O:SYG:SYD:(A;;0x1200a9;;;BU)
ShadowCopy            : False
ShareState            : Online
ShareType             : FileSystemDirectory
SmbInstance           : Default
Special               : False
Temporary             : False
Volume                : \\?\Volume{8bba7c49-0000-0000-0000-602200000000}\
PSComputerName        :
PresetPathAcl         : System.Security.AccessControl.DirectorySecurity

        [*] Domain controller renamed. [*]
        [*] Creating Domain Groups [*]
                [+] Adding Senior Management to mayorsec.local
        [+] Adding IT Admins to mayorsec.local
        [+] Adding Engineering to mayorsec.local
        [+] Adding Sales to mayorsec.local
        [*] Generating Organizational Units for the mayorsec.local. [*]
        [+] Organizational Units added.
        [*] Group creation completed. [*]
        [*] Creating Domain Users [*]
        [+] a.adams added
        [+] Adding a.adams to Senior Management Group
        [+] Adding a.adams to Domain Administrators Group
        [+] j.taylor added
        [+] Adding j.taylor to IT Admins Group
        [+] j.anthony added
        [+] Adding j.anthony to Engineering Group
        [+] t.carter added
        [+] Adding t.carter to Engineering Group
        [+] m.phillips added
        [+] Adding m.phillips to Engineering Group
        [+] r.smith added
        [+] Adding r.smith to Engineering Group
        [+] s.chisholm added
        [+] Adding s.chisholm to Sales
        [+] m.seitz added
        [+] Adding m.seitz to Engineering Group
        [+] a.tarolli added
        [+] Adding a.tarolli to Sales
        [+] z.dickens added
        [+] Adding z.dickens to Sales
        [*] User creation completed [*]
        [*] Modifying pre-authentication privileges [*]
        [+] ASREP privileges granted to a.tarolli
        [*] ASREP settings update completed. [*]
        [*] Adding Kerberoastable service account to domain [*]
The command completed successfully.

Checking domain DC=mayorsec,DC=local

Registering ServicePrincipalNames for CN=mssql_svc,CN=Users,DC=mayorsec,DC=local
        DC01/mssql_svc.
Updated object
        [+] mssql_svc service account added
        [*] Kerberoastable service creation completed. [*]
        [*] Granting IT Admins GenericAll rights on Domain Admins. [*]
        [+] IT Admins group granted GenericAll permissions for the Domain Admins group.
        [*] Adding misconfigured ACL rule for the Engineering group. [*]
        [+] Whoops! GenericAll rights granted to Engineering.
        [*] Adding misconfigured ACL rule for Margaret Seitz. [*]
        [+] Whoops! GenericAll rights granted to m.seitz.
        [*] Adding misconfigured ACL rule for the Sales group. [*]
        [+] Whoops! GenericAll rights granted to Sales.
        [*] ACL misconfigurations completed. [*]
        [*] Configuring some GPO policies required for the domain. [*]

DisplayName   : WinRM Firewall TCP 5985
GpoId         : fac02e0a-d68e-4d10-979b-16a5f9db31ef
Enabled       : True
Enforced      : False
Order         : 2
Target        : DC=mayorsec,DC=local
GpoDomainName : mayorsec.local


Caption                 :
Description             :
ElementName             : Allow WinRM TCP 5985 To Domain Joined Systems
InstanceID              : {1fe11ce5-117b-48a1-bf7b-116c874d20e9}
CommonName              :
PolicyKeywords          :
Enabled                 : True
PolicyDecisionStrategy  : 2
PolicyRoles             :
ConditionListType       : 3
CreationClassName       : MSFT|FW|FirewallRule|{1fe11ce5-117b-48a1-bf7b-116c874d20e9}
ExecutionStrategy       : 2
Mandatory               :
PolicyRuleName          :
Priority                :
RuleUsage               :
SequencedActions        : 3
SystemCreationClassName :
SystemName              :
Action                  : Allow
Direction               : Inbound
DisplayGroup            :
DisplayName             : Allow WinRM TCP 5985 To Domain Joined Systems
EdgeTraversalPolicy     : Block
EnforcementStatus       : NotApplicable
LocalOnlyMapping        : False
LooseSourceMapping      : False
Owner                   :
Platforms               : {}
PolicyStoreSource       :
PolicyStoreSourceType   : GroupPolicy
PrimaryStatus           : OK
Profiles                : 0
RuleGroup               :
Status                  : The rule was parsed successfully from the store. (65536)
StatusCode              : 65536
PSComputerName          :
Name                    : {1fe11ce5-117b-48a1-bf7b-116c874d20e9}
ID                      : {1fe11ce5-117b-48a1-bf7b-116c874d20e9}
Group                   :
Profile                 : Any
Platform                : {}
LSM                     : False

        [+] A GPO for PowerShell Remoting was created for authenticated users on the domain.
        [*] GPO configurations completed. [*]
        [*] Configuring GPO policies to enable PowerShell remoting on hosts. [*]

DisplayName   : Enable PSRemoting Desktops
GpoId         : 6bc94864-cd95-44ea-9459-e050214175ad
Enabled       : True
Enforced      : False
Order         : 3
Target        : DC=mayorsec,DC=local
GpoDomainName : mayorsec.local


Id               : 6bc94864-cd95-44ea-9459-e050214175ad
DisplayName      : Enable PSRemoting Desktops
Path             : cn={6BC94864-CD95-44EA-9459-E050214175AD},cn=policies,cn=system,DC=mayorsec,DC=local
Owner            : mayorsec\Domain Admins
DomainName       : mayorsec.local
CreationTime     : 1/5/2022 4:06:43 AM
ModificationTime : 1/5/2022 4:06:44 AM
User             : Microsoft.GroupPolicy.UserConfiguration
Computer         : Microsoft.GroupPolicy.ComputerConfiguration
GpoStatus        : AllSettingsEnabled
WmiFilter        :
Description      :


Id               : 6bc94864-cd95-44ea-9459-e050214175ad
DisplayName      : Enable PSRemoting Desktops
Path             : cn={6BC94864-CD95-44EA-9459-E050214175AD},cn=policies,cn=system,DC=mayorsec,DC=local
Owner            : mayorsec\Domain Admins
DomainName       : mayorsec.local
CreationTime     : 1/5/2022 4:06:43 AM
ModificationTime : 1/5/2022 4:06:44 AM
User             : Microsoft.GroupPolicy.UserConfiguration
Computer         : Microsoft.GroupPolicy.ComputerConfiguration
GpoStatus        : AllSettingsEnabled
WmiFilter        :
Description      :


Id               : 6bc94864-cd95-44ea-9459-e050214175ad
DisplayName      : Enable PSRemoting Desktops
Path             : cn={6BC94864-CD95-44EA-9459-E050214175AD},cn=policies,cn=system,DC=mayorsec,DC=local
Owner            : mayorsec\Domain Admins
DomainName       : mayorsec.local
CreationTime     : 1/5/2022 4:06:43 AM
ModificationTime : 1/5/2022 4:06:44 AM
User             : Microsoft.GroupPolicy.UserConfiguration
Computer         : Microsoft.GroupPolicy.ComputerConfiguration
GpoStatus        : AllSettingsEnabled
WmiFilter        :
Description      :

        [+] Registry setting for Powershell Remoting OK!

Id               : 6bc94864-cd95-44ea-9459-e050214175ad
DisplayName      : Enable PSRemoting Desktops
Path             : cn={6BC94864-CD95-44EA-9459-E050214175AD},cn=policies,cn=system,DC=mayorsec,DC=local
Owner            : mayorsec\Domain Admins
DomainName       : mayorsec.local
CreationTime     : 1/5/2022 4:06:43 AM
ModificationTime : 1/5/2022 4:06:44 AM
User             : Microsoft.GroupPolicy.UserConfiguration
Computer         : Microsoft.GroupPolicy.ComputerConfiguration
GpoStatus        : AllSettingsEnabled
WmiFilter        :
Description      :


Id               : 6bc94864-cd95-44ea-9459-e050214175ad
DisplayName      : Enable PSRemoting Desktops
Path             : cn={6BC94864-CD95-44EA-9459-E050214175AD},cn=policies,cn=system,DC=mayorsec,DC=local
Owner            : mayorsec\Domain Admins
DomainName       : mayorsec.local
CreationTime     : 1/5/2022 4:06:43 AM
ModificationTime : 1/5/2022 4:06:46 AM
User             : Microsoft.GroupPolicy.UserConfiguration
Computer         : Microsoft.GroupPolicy.ComputerConfiguration
GpoStatus        : AllSettingsEnabled
WmiFilter        :
Description      :

        [+] Service setting for Powershell Remoting OK!
        [*] Domain-wide PowerShell Remoting GPO configuration completed. [*]
        [*] Some changes require a restart to take effect. Restarting your domain controller in 30 seconds. [*]
```

![image](https://user-images.githubusercontent.com/58165365/148123745-7a8f3fae-cf53-44b5-9403-3af589ff9da3.png)

# Joining Domains

Starting with the DC, since its on the `Secure Network`, we can give it a static ip of `192.168.40.50`. To do this, we need to go to `Control Panel > Network and Internet > Network and Sharing Center`

Head over to `Setting > Accounts > Access work or school` and click on connect

![image](https://user-images.githubusercontent.com/58165365/148374492-9b503c7c-d6ff-44a2-9ab0-b412e7d7d5cf.png)

In our scenario, we want to join a local Active Directory Domain

![image](https://user-images.githubusercontent.com/58165365/148375003-ddd78211-e4ae-42b7-a1dd-462eb016c76d.png)

Add the domain to join (mayorsec.local)

![image](https://user-images.githubusercontent.com/58165365/148375182-e505a4f9-aa93-4f7f-9153-99e251b86ef8.png)

You will be promted to input a user and their password.

![image](https://user-images.githubusercontent.com/58165365/148381911-c6d487c8-8240-4043-bdbb-a258ed6899e1.png)

Make sure you choose Administrator

![image](https://user-images.githubusercontent.com/58165365/148382225-b87070d2-94a5-4592-8d97-db3e4756cfc7.png)

Once the machine reboots, you'll notice we are now part of the domain.

![image](https://user-images.githubusercontent.com/58165365/148383110-72a34419-4cef-4f60-aaf0-8c1bbf753f9f.png)

Repeat this procedure with Workstation2.

If we now go back to our server , launch server manager and on the top right corner, click Tools and select `Active Directory Users and Computers`

![image](https://user-images.githubusercontent.com/58165365/148431487-109a5d94-bfb0-4de8-ac9f-9746ef12b3c4.png)

If we click on the Computers tab, we should be able to find both machines listed.

![image](https://user-images.githubusercontent.com/58165365/148431143-78bf8e86-442e-4838-87c1-0b01e84c380e.png)

This names make it hard to identify a specific computer. We can go ahead an use a script called nameGen to automatically rename the PC's. This has to be executed on both machines though. Here is how you would go about it.

![image](https://user-images.githubusercontent.com/58165365/148440380-f00bef6e-3b49-4970-8290-a538fa2ab502.png)

We find 3 folders. We are particularly interested in `Shared`

![image](https://user-images.githubusercontent.com/58165365/148432723-ac9099fb-12a9-442b-b813-0114b9decf45.png)

If we navigate further, we can get the script and drag it to our desktop

![image](https://user-images.githubusercontent.com/58165365/148433670-7eb080f8-64dd-496b-b687-4ed5a3c9b339.png)

```powershell
PS C:\Users\s.chisholm.mayorsec\Desktop> dir


    Directory: C:\Users\s.chisholm.mayorsec\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         9/23/2021   1:06 PM            951 nameGen.ps1


PS C:\Users\s.chisholm.mayorsec\Desktop> Set-ExecutionPolicy Unrestricted

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): Y
PS C:\Users\s.chisholm.mayorsec\Desktop> . .\nameGen.ps1
PS C:\Users\s.chisholm.mayorsec\Desktop> executeScript -ComputerName WORKSTATION-02
WARNING: The changes will take effect after you restart the computer DESKTOP-C4EGH78.


    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          1/6/2022  10:55 AM                Shared
AvailabilityType      : NonClustered
CachingMode           : Manual
CATimeout             : 0
ConcurrentUserLimit   : 0
ContinuouslyAvailable : False
CurrentUsers          : 0
Description           :
EncryptData           : False
FolderEnumerationMode : Unrestricted
IdentityRemoting      : False
Infrastructure        : False
LeasingMode           : Full
Name                  : Shared
Path                  : C:\Shared
Scoped                : False
ScopeName             : *
SecurityDescriptor    : O:SYG:SYD:(A;;FA;;;BU)
ShadowCopy            : False
ShareState            : Online
ShareType             : FileSystemDirectory
SmbInstance           : Default
Special               : False
Temporary             : False
Volume                : \\?\Volume{dc62328a-0000-0000-0000-300300000000}\
PSComputerName        :
PresetPathAcl         : System.Security.AccessControl.DirectorySecurity
```

![image](https://user-images.githubusercontent.com/58165365/148446404-14be4b45-dcce-4e3e-be3c-2fe9b6f6b336.png)

# Resources

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) / [Vmware](https://www.vmware.com/products/workstation-player.html)
- VirtualBox Guest Additions
- [Windows Server 2019](https://www.microsoft.com/en-US/evalcenter/evaluate-windows-server-2019?filetype=ISO)
- [Windows 10](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise)
- [dievus/ADGenerator](https://github.com/dievus/ADGenerator)
- [Movement, Pivoting, and Persistence for Pentesters and Ethical Hackers Course](https://www.udemy.com/course/movement-pivoting-and-persistence/?referralCode=99A09396FE1258FC3A2A) by - [Joe Helle](https://twitter.com/joehelle)
