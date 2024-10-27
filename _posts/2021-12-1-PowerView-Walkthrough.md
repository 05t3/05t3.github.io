---

title: "PowerView Walkthrough"
date: 2021-12-1 01:09:33 +0300
author: oste
image: /assets/img/Posts/Powerview.png
categories: [CYBERSECURITY]
tags: [active directory, powershell, powerview, windows]
---

# PowerView.ps1 can be found here

PowerView.ps1 can be downloaded [here](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)

## Documentation

[Official Documentation](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)

For more functions, check out:

[darkoperator/Veil-PowerView](https://github.com/darkoperator/Veil-PowerView/tree/master/PowerView)

## Some Awesome Cheat Sheets

[HarmJ0y's](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)

[Hacktricks](https://book.hacktricks.xyz/windows/basic-powershell-for-pentesters/powerview)

HarmJ0y also has a good article on using some of PowerView’s interesting functionality.

[Blog](https://www.harmj0y.net/blog/powershell/veil-powerview-a-usage-guide/)

# 📝Personal Notes 📝

In the demo's below, i have created an AD environment where my theme is inspired by the famous American sitcom "The Office"

![image](https://user-images.githubusercontent.com/58165365/137598951-1f8fda57-4fee-4cd7-ac2e-b0f914620488.png)

I have then downloaded PowerView in one of my Windows 10 PC's for practice Purposes.

First, you need to fireup your command prompt and type:

`powershell.exe -nop -exec bypass`

You then need to import the PowerView module as follows:

`Import-Module [full path to powerview.ps1]`

![image](https://user-images.githubusercontent.com/58165365/137599960-e5b6ed58-c975-4100-a468-7eeedb8fc767.png)

Alternatively, You could run:

`powershell -ep bypass`

Followed by:

`. .\PowerView.ps1`

![image](https://user-images.githubusercontent.com/58165365/137600122-ca1aaafd-2514-4bb8-92d9-a02318d84d8a.png)

From here we can start enumerating😉

We can get information about the domain as follows:

`Get-NetDomain`

![image](https://user-images.githubusercontent.com/58165365/137596016-cfd869d8-f497-4279-b2de-f5668f06e8f8.png)

Getting information about DC's

`Get-NetDomainControllers`

![image](https://user-images.githubusercontent.com/58165365/137596029-618d8e31-44a3-4a23-ad6a-c71c54dc5437.png)

To query domain policies

`Get-DomainPolicy`

![image](https://user-images.githubusercontent.com/58165365/137596070-77089819-331e-43d0-8929-e2fc2a751299.png)

If you wanted to look at a specific policy, you would do so as follows:

`(Get-DomainPolicy)."system access"`

![image](https://user-images.githubusercontent.com/58165365/137596092-e4270cf7-32f3-46ff-abdb-f03b9ce5fad5.png)

_Information such as `MinimumPasswordLength` which shows 7 can enable you to know that we can try spray 7 character passwords_

If we wanted to enumerate Users:

`Get-NetUser`

![image](https://user-images.githubusercontent.com/58165365/137596231-d7590525-1f75-4fe9-87a0-f53109fce009.png)

_The above command spits a ton of information for all users. Assuming you are enumerating an organization with many users, this can generate a whole bunch of information. We can filter all users though by running:_

`Get-NetUser | select cn`

or

`Get-NetUser | select samaccountname`

![image](https://user-images.githubusercontent.com/58165365/137596280-89403360-2dfd-460e-b270-a86a4084b529.png)

If we want to get all the groups a user is a member of

`Get-DomainGroup -MemberIdentity username | select cn`

or

`Get-DomainGroup -MemberIdentity username | select samaccountname`

![image](https://user-images.githubusercontent.com/58165365/137596326-00034250-a188-4e13-b2bf-d4dbebf6e0be.png)

If you wanted to check if the current user context has local administrator access

`Invoke-CheckLocalAdminAccess`

![image](https://user-images.githubusercontent.com/58165365/137597411-254cfde5-bc01-48bd-8221-b28b8431c1ed.png)

If you wanted to check if the current user has administrative access to the local (or a remote) machine

`Test-AdminAccess`

![image](https://user-images.githubusercontent.com/58165365/137597489-c5e4f743-1a62-4599-8858-36bc4ac3ae3b.png)

Lazy admins can leave behind some juicy details on the description. We can pull descriptions only by running:

`Get-NetUser | select description`

![image](https://user-images.githubusercontent.com/58165365/137596361-cba575d9-67c7-485f-97bd-e88f170558e6.png)

If we want to get all the groups a user is a member of

`Get-DomainGroup -MemberIdentity username | select cn`

or

`Get-DomainGroup -MemberIdentity username | select samaccountname`

![image](https://user-images.githubusercontent.com/58165365/137594811-0f4af23d-1b78-4ab0-be3d-7559b23cb191.png)

To get all the effective members of a group:

`Get-DomainGroupMember -Identity "Domain Admins" -Recurse`

![image](https://user-images.githubusercontent.com/58165365/137595025-4e0350a8-3a37-4aae-81fe-ae20c77bb7b2.png)

Get Information about all computers:

`Get-NetComputer`

![image](https://user-images.githubusercontent.com/58165365/137591942-ba681958-ee9e-4396-819c-ac89fa1debe7.png)

Enumerating what operating systems are used on the domain

`Get-NetComputer | select operatingsystem`

![image](https://user-images.githubusercontent.com/58165365/137592238-95c0757d-b7b8-4ff1-904a-1a6fc5897f8c.png)

Enumerating groups

`Get-NetGroup`
_This spits a ton of information_

![image](https://user-images.githubusercontent.com/58165365/137592401-c288b3b4-0617-4a7a-a813-48f76fb42069.png)

We can summarise all the information above by running

`Get-NetGroup | select name`

![image](https://user-images.githubusercontent.com/58165365/137592610-3588571c-5ee7-4f6f-9829-8905872dbc78.png)

To locate shares on hosts in the local domain

`Invoke-ShareFinder`

![image](https://user-images.githubusercontent.com/58165365/137593133-fc06a934-bc9f-4380-8f44-0dfda94db9a0.png)

To pull Group Policy Objects (GPO's)

`Get-NetGPO`
_This spits a ton of information_

![image](https://user-images.githubusercontent.com/58165365/137593882-61326d56-198f-4499-92d9-66f2464d9621.png)

We can narrow it down by selecting specific attributes. For example

`Get-NetGPO | select displayname, whenchanged, whencreated`

![image](https://user-images.githubusercontent.com/58165365/137594535-205fdee2-9085-4f27-a5a7-085b6b056c79.png)

If you have any questions, comments or would like to reach out to me:

[![Twitter URL](https://img.shields.io/twitter/url/https/twitter.com/bukotsunikki.svg?style=social&label=DM%20%40oste_ke)](https://twitter.com/oste_ke)
