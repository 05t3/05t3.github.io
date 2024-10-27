---

title: "Browser & Blockchain Forensics"
date: 2022-10-07 01:09:33 +0300
author: oste
image: /assets/img/Posts/cybercon1.png
categories: [CTF-TIME]
tags: [Browser, Blockchain]
---

| CTF      | CyberCon Preliminaries 2022                   |
| -------- | --------------------------------------------- |
| Date     | 24th-25th September, 2022                     |
| Platform | [CyberSpace CTFRoom](https://www.ctfroom.com) |

# Browser & Blockchain Forensics

_~Author: **OSTE**_

> _In this investigation, you will be analyzing data retrieved from web browsers to determine how a malware infection began.
> An employee has visited a malicious site and downloaded a malicious program to their system, which began compromising system security by disabling security features. This ations were captured by the SOC team. They We are unsure of the variant of malware that has hit the organization, but it spread to 4 other systems before we were able to contain it. We were able to acquire key browser files via BHC, we need your skills to analyze them and discover the malicious site that hosted the malware so we can take defensive measures and block it. We can also work with our threat intelligence team to inform other organizations about the malicious domain and IOCs you'll discover. We have constructed a report, and need you to collect evidence to be able to answer the questions presented._

# Challenge Creation.

I spin my windows forensics/test lab and browsed around to gather enough data for analysis later. I also intentionally visited a malicious live url to download a piece of malware for analysis later.

I then used a tool by Foxton Forensics called [Browser History Capturer](https://www.foxtonforensics.com/browser-history-capturer/) to easily capture web browser history from a Windows computer. The tool can be run from a USB dongle or via a Remote Desktop connection to capture history from Chrome, Edge, Firefox and Internet Explorer web browsers as shown in the screenshot below:

![image](https://user-images.githubusercontent.com/58165365/190234628-137b192c-b43a-4096-916f-228edc097ed7.png)

For analysis, i will be foussing on another awesome tool by Foxton, [Browser History Examiner](https://www.foxtonforensics.com/browser-history-examiner/), to analyse and report internet history from the main desktop web browsers. It can assist in various digital investigations such as civil & criminal digital forensics cases, security incidents, human resources investigations and general employee activity reporting.

However, i should mention that this is a paid product but they allow you to download a free trial with no time limits, it includes all features but shows 25 records per data type.😫

![image](https://user-images.githubusercontent.com/58165365/190233541-dc463867-e591-4ae2-846b-cbf716630cfb.png)

All you need to do is load the history (artifacts we gathered using BHC as shown:

![image](https://user-images.githubusercontent.com/58165365/190233021-4a65dcf6-0b4c-4992-b823-3d0e5372e782.png)

One its done processing , data will be sorted/categorized as shown in the highlighted pane on the left.

On the right, we have an advanced filtering option where you can find relevant data faster using a variety of filters such as keywords and date/time range.

At the bottom, we have the Website Activity Timeline which identifies peaks in internet activity using the interactive timeline.

![image](https://user-images.githubusercontent.com/58165365/190233889-4fb62c36-4630-4bec-919a-7ddf83c929ad.png)

Another great tool Foxton offers is [Browser History Viewer](https://www.foxtonforensics.com/browser-history-viewer/), a forensic software tool for extracting and viewing internet history from the main desktop web browsers. Its more or less similar to BHE but with less funtionalities but display all records per data type as oppossed to BHE which displays 25 records

Load the captured artifacts as shown below:

![image](https://user-images.githubusercontent.com/58165365/190234379-06e3a3e4-42d6-4e50-a5b0-2e759e0d1f58.png)

![image](https://user-images.githubusercontent.com/58165365/190233245-a0173728-5ad2-44ae-84f9-6de4177bde93.png)

One the history is loaded, you'll get the following view.

![image](https://user-images.githubusercontent.com/58165365/190232813-1e6c2a69-8cac-4702-ac94-2353f70992e6.png)

The first active tab on the top left is the `Web History` and displays the URLs & Dates Visited, number of visits alongside the Web Browser used and the relevant profiles found. For this analysis, there are about 202 records found.

Moving on, we have the second tab, **Cached Images** which displays cache images , url loation they were fetched from and dates alongside the Web Browser(Profile) info.

![image](https://user-images.githubusercontent.com/58165365/190232682-34aa9e03-faa6-4118-8db1-e496c52f5fbf.png)

# Challenge Questions

### 1. Users in the organization are encouraged not to save passwords on their browsers. However the user has a common email address and password used on several platforms. With that in mind, the user created a pastebin and locked the paste. Can you retrieve the contents of the paste?

Applying `pastebin` as our filter, we get a link to the paste created, `https://pastebin.com/3i48shgE` . However, its password protected.

![image](https://user-images.githubusercontent.com/58165365/194722582-949ba138-0207-4656-9186-6e285f065af5.png)

![image](https://user-images.githubusercontent.com/58165365/194722796-0e92bedd-b019-4c48-a77a-b58872565511.png)

Inspecting the form history data we get the password used : `jstar1337`

![image](https://user-images.githubusercontent.com/58165365/194722710-4eca85e7-19cf-4ba5-ae6a-d49a448345d8.png)

Visiting the link we found earlier and try use the password retrieved, we get the flag:

![image](https://user-images.githubusercontent.com/58165365/194722796-0e92bedd-b019-4c48-a77a-b58872565511.png)

![image](https://user-images.githubusercontent.com/58165365/194722833-def070c7-fe61-42bd-ad60-e6132d8f1991.png)

Flag `ccke{11221e90758526cf7249771fee28ab54}`

### 2. The company has a policy against employees visiting social media on corporate devices. What sites has the employee visited, listed alphabetically?

---

A couple of friends DM'd me in regard to this question to enquire if some platforms like `exploit.in` , `hackforums.net` , `4chan`, `evilzone.org` are social media sites. Unfortunately, this are forumns 😢. Social media sites present on the artifacts were:

![image](https://user-images.githubusercontent.com/58165365/194538884-8e40ed60-e60a-4d99-a73a-da15f25e8c5d.png)

![image](https://user-images.githubusercontent.com/58165365/194538985-40da44bc-2ffb-4018-967d-38d644b16771.png)

`ccke{reddit, twitter}`

### 3. Between 09/14/2022 12:59:23 - 09/14/2022 13:04:15 , the user visited a malicious site and downloaded the executable attached. Provide the full URL visited by the user and determine the common name of the malware (Google is your friend).

---

Looking at the downloaded artifacts, with the given time range, we get a hit to `http://185.215.113.66/tpeinf.exe`. ( _first part of the flag_)

![image](https://user-images.githubusercontent.com/58165365/190241890-6a378edb-7b2a-4c91-9ac5-3b93c7abad2d.png)

Since you are provided for with the suspicious executable, we can confirm if the file is indeed an executable by running `file` command.

We can also confirm the HEX Magic Bytes by using the `xxd` command as shown below. We can also get the hash values of the executable so that we can use to find relevant info from public threat feeds and databases.

![image](https://user-images.githubusercontent.com/58165365/190278384-ecefaa05-a928-4149-9a1b-67a5b1c44430.png)

The next series of steps you'd have taken was perform some Crowdsourced intelligence. Starting with [VirusTotal](https://www.virustotal.com/gui/file/22f524abc98f958705febd3761bedc85ec1ae859316a653b67c0c01327533092/detection)
we get hits to a malware called `Phorpiex`.

![image](https://user-images.githubusercontent.com/58165365/190242207-bf669525-0e79-45dc-b96b-6a9d5c779b46.png)

Other sources:

- [MalShare](https://malshare.com/sample.php?action=detail&hash=ed2d7b25bb360cccb4f0f6a4f8732d7a)

![image](https://user-images.githubusercontent.com/58165365/190242776-e7f5af61-b158-4a44-9c6b-bc1b58e41b98.png)

- [MalwareBazaar Database](https://bazaar.abuse.ch/sample/22f524abc98f958705febd3761bedc85ec1ae859316a653b67c0c01327533092/)

![image](https://user-images.githubusercontent.com/58165365/190243325-e3ef2364-1c98-4651-9ff4-4b9d2358ce27.png)

- [JoeSandbox](https://www.joesandbox.com/analysis/684211/0/html)

![image](https://user-images.githubusercontent.com/58165365/190245140-f67c0986-fe33-4cc2-b42b-2ed5c01f6088.png)

- [URLhaus Database](https://urlhaus.abuse.ch/browse.php?search=22f524abc98f958705febd3761bedc85ec1ae859316a653b67c0c01327533092)

![image](https://user-images.githubusercontent.com/58165365/190246427-e1af68f8-9fcc-49e2-a240-540b4b80b1fb.png)

- [AnyRun Analysis Report](https://any.run/report/5fd1c0ca483fac14d2bc830430ac8ed5f2676a68656301dfefc8514057d01907/05f8d850-41c8-4392-879c-04d0b78aa0cf)

> _Phorpiex is a worm which spreads via removable drives and network drives. Some Phorpiex variants will also download additional malware such as cryptominer and execute them. Its an old threat known for its sextortion spam campaigns, crypto-jacking, cryptocurrency clipping and ransomware spread_

If you wanna read more about the same, i'd suggest you read the following:

- Microsoft 365 Defender Threat Intelligence Team. (2021, May 21). Phorpiex morphs: How a longstanding botnet persists and thrives in the current threat environment. [Microsoft Security Blog](https://www.microsoft.com/security/blog/2021/05/20/phorpiex-morphs-how-a-longstanding-botnet-persists-and-thrives-in-the-current-threat-environment/)
- Phorpiex botnet is back with a new Twizt: Hijacking hundreds of crypto transactions. (2021, December 27). [Check Point Research](https://research.checkpoint.com/2021/phorpiex-botnet-is-back-with-a-new-twizt-hijacking-hundreds-of-crypto-transactions/)
- Phorpiex botnet returns with new tricks making it harder to disrupt. (2021, December 16). [BleepingComputer](https://www.bleepingcomputer.com/news/security/phorpiex-botnet-returns-with-new-tricks-making-it-harder-to-disrupt/)

ANSWER: `ccke{http://185.215.113.66/tpeinf.exe,Phorpiex}`

### 4. By performing some static analysis on the binary provided, there are several crypto wallets hard coded in it. With some OSINT , we suspect one of the Ethereum Wallets might be linked to some Nigerian Scammer. Can you find his twitter handle and the the transaction hash for the last transation he conduted on (Aug 27, 2022 7:14 PM UTC)

---

Running strings on the malicious binary, we find a ton on crypto wallets.

![image](https://user-images.githubusercontent.com/58165365/190261185-8e2723a3-f11f-4c16-a06f-a4b7e0253033.png)

However, Doing some OSINT/Blockchain forensics (i dunno) we come across a twitter post , where a user [@Wale57195907](https://twitter.com/Wale57195907) has made a comment in some ETH related tweet.

`0xb899fC445a1b61Cdd62266795193203aa72351fE`

![image](https://user-images.githubusercontent.com/58165365/190261809-4c9ed245-5363-4f63-836c-747e43eddc37.png)

![image](https://user-images.githubusercontent.com/58165365/190262107-77f6cb19-4182-46f0-8eae-7735fd8b2e82.png)

With that in mind, we can use a platform like [Blockchair](https://blockchair.com/ethereum/address/0xb899fc445a1b61cdd62266795193203aa72351fe)
to look into basically any kind of crypto addresses.

![image](https://user-images.githubusercontent.com/58165365/190265015-e1e4fc87-295c-4ec8-ad1b-771e1669a92e.png)

Looking at the transaction conducted on the specified date, (Aug 27, 2022 7:14 PM UTC), we get the transaction hash `0x9c3e5e624bb8d086826083cee567bdc08be71898e66546f1fc7d6e85e268c584`

> _BTW [Blockchair](https://blockchair.com/) is the first blockchain explorer which incorporates a multitude of different blockchains into one search engine. Super awesome stuff!_

We can also use [Etherscan's](https://etherscan.io/address/0xb899fC445a1b61Cdd62266795193203aa72351fE) site to get the same info.

![image](https://user-images.githubusercontent.com/58165365/190285277-44ba57bd-5998-4bc9-8b0f-4552ff667799.png)

Flag `ccke{@Wale57195907,0x9c3e5e624bb8d086826083cee567bdc08be71898e66546f1fc7d6e85e268c584}`

### 5. What was the C2 Server's callback URL?

Scrolling further on the strings, we get some potential C2 callback URL's

![image](https://user-images.githubusercontent.com/58165365/190262923-05abc66d-3f8b-46d5-b51d-91931d6ddf01.png)

Flag: `ccke{http://185.215.113.66/twizt/}`

Easy Peasy, huh 😅
