---
title: BSides Nairobi 2023 Cyber Challenge Walkthrough
description: BSides Nairobi 2023 Cyber Challenge Walkthrough
author: oste
date: 2023-11-05 01:09:33 +0300
categories: [CTF-TIME]
tags: [crossword, jigsaw, puzzle, malware, AgentTesla, eml, phishing, AnyRun]
image: /assets/img/Posts/BSIDES2023-1.png
#   path: assets/img/Posts/forest.png
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---

Hey there and welcome back to another blog post. I will be discussing my challenges for the `BSides Nairobi 2023 Cyber Challenge` that took place on the 04/11/2023 at [USIU](https://www.usiu.ac.ke/). The challenge was tackled by 45 registerd teams.

After 8hrs of competing, [p3rf3ctr00t](https://ctftime.org/team/193649/), led by [_k4p3re_](https://twitter.com/_k4p3r3) emerged 🥇 place. 

![image](https://user-images.githubusercontent.com/58165365/280506659-67706b5d-3e73-4712-8db1-287554516e8b.png)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">well well well, here we are again, champions for the recently concluded <a href="https://twitter.com/BSidesNairobi?ref_src=twsrc%5Etfw">@BSidesNairobi</a> in conjunction with <a href="https://twitter.com/CTF_Room?ref_src=twsrc%5Etfw">@CTF_Room</a> CTF.<a href="https://twitter.com/fr334aks?ref_src=twsrc%5Etfw">@fr334aks</a> in your face 😂 <a href="https://t.co/9eahGqPlG1">pic.twitter.com/9eahGqPlG1</a></p>&mdash; p3rf3ctr00t (@p3rf3ctr00t) <a href="https://twitter.com/p3rf3ctr00t/status/1720832721809731885?ref_src=twsrc%5Etfw">November 4, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The audacity to rub it on @fr334aks face

![gif](https://media.giphy.com/media/wID3zXURLH1jrjCcZy/giphy.gif)

This year, the event was special as the first team bagged home a trophy

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Lol. Watch the new shiny trophy effect. <a href="https://twitter.com/Amarjit_Labu?ref_src=twsrc%5Etfw">@Amarjit_Labu</a> <a href="https://twitter.com/JonesBaraza?ref_src=twsrc%5Etfw">@JonesBaraza</a> were mesmerized. The effects tell it all 🤣 <a href="https://twitter.com/hashtag/BsidesNairobi?src=hash&amp;ref_src=twsrc%5Etfw">#BsidesNairobi</a> <a href="https://twitter.com/hashtag/BsidesNairobi2023?src=hash&amp;ref_src=twsrc%5Etfw">#BsidesNairobi2023</a> <a href="https://t.co/ZHEr1UogbF">pic.twitter.com/ZHEr1UogbF</a></p>&mdash; #BSidesNairobi 2023 (@BSidesNairobi) <a href="https://twitter.com/BSidesNairobi/status/1720795590446006654?ref_src=twsrc%5Etfw">November 4, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



# Misc

## CroffFit

### Crossword

This challenge brings together an all-rounded quiz that encompasses questions on Kenyan current affairs, global news, and mastery of both Windows and Linux. Can you solve this crossword and prove your expertise?

**Instructions.**
- This is an online based activity and individual activity is not tracked.
- Sharing solutions with other teams is PROHIBITED.
- Access the challenge [here](https://bit.ly/40lQew2)
- Once your team completes the challenge,  come get a physical - printed flag

![image](https://user-images.githubusercontent.com/58165365/280506840-16e5d54d-cf99-423d-ba6a-b5e6ceb8e00c.png)

**Solution**

![image](https://user-images.githubusercontent.com/58165365/280506929-e0568381-2faf-4a3e-8972-75ed0229c601.png)



## Jigsaw

A scrambled image awaits your expertise. Piece it together to reveal a message or picture. But there's a twist! This isn't just a virtual game. Once you believe the image is whole and the message clear, the team captain must approach the challenge master (05t3) to claim a physical flag. But be wary: precision matters, and only a perfectly assembled image will lead to victory.

**Instructions.**
- This is an online based activity and individual activity is not tracked.
- Sharing solutions with other teams is PROHIBITED.
- Access the challenge [here](https://puzzel.org/jigsaw/play?p=-NiFruyx4iRKc6qaVPP7)
- Once your team completes the challenge,  come get a physical - printed flag

![image](https://user-images.githubusercontent.com/58165365/280507276-298a15d0-de1f-4550-8aa3-43e8b7cf7b87.png)

**Solution**

![image](https://user-images.githubusercontent.com/58165365/280507316-442c71b9-f8b3-48a5-b893-9d260ac2dfb1.png)

# Forensics
## Mystique

I recently encountered a potentially malicious email and thought it would be interesting to turn it into a CTF challenge. The email has an attachment that seems suspicious. Can you help analyze and decipher its purpose?


**Guidelines:**
- The provided files are genuine malware samples, so treat them with caution.
- Always examine them in a sandboxed environment to ensure safety.
- For your convenience, the zip files containing these samples are password-protected. Use the password "**infected**" to access them.
- If needed, feel free to use online resources for assistance. One recommended online sandbox tool is [AnyRun](https://app.any.run/).

### Module: Email Forensics
#### Questions

Start by analyzing this email.

- Who is the sender of this email? (25pts)

`BSidesNBI{2export@ekofood.com.tr}`

- What is the JARM fingerprint of the Originating IP rDNS address? (25pts)

`BSidesNBI{15d3fd16d29d29d00042d43d00000071784fa9f8305ba9220d0a7894b6ff2c}`

- What is the md5sum of the email attachment? (25pts)

`BSidesNBI{2dae57b509d72eb69166e9d48995e530}`

- Without opening the attachment on your host machine, use an online sandbox like AnyRun to observe what happens when the document is opened. From your analysis, what CVE is associate with the attachment? (25pts)

`BSidesNBI{CVE-2017-11882}`

- What malware family is likely associated with the attachment? (25pts)

`BSidesNBI{agenttesla}`

- Take a look at the malware configuration. What is the c2 Domain address? (25pts)

`BSidesNBI{cp5ua.hyperhost.ua}`

- In your opinion, what protocol do you suspect could have been leveraged on for potential exfiltration? (25pts)

`BSidesNBI{smtp}`

- What was the username and password used in the protocol mentioned above? (25pts)

`BSidesNBI{arinzelog@saonline.xyz_7213575aceACE@#$}`

----

#### **Solution**

Let me walk you through my thought process of how i analyzed the email upon receiving it. First, I was at work and didnt have my malware environment available at my disposal to disect the mail. So i downloaded the `.eml` file and used my favourite online sandbox for some quick analysis.([PhishTool](https://app.phishtool.com/)).

Upon loading the file, it rendered as so:

![image](https://user-images.githubusercontent.com/58165365/280507601-9ee88b92-248a-4837-8cb2-1f816135c860.png)

Straight up, I know this was a phishing email.
- I dont own any products that i'm selling 😂
- The attached list was a `.doc` (Items list.doc) - potentially containing malware 🚩
- Inconsistent display-name. The 'From' email address local-part `2export` is inconsistent with the display-name `Fatih ALTINDAŞ` provided in the email.
- I looked up the display-name online (`Fatih ALTINDAŞ`) and got a hit on [LinkedIn](https://www.linkedin.com/in/fatih-alt%C4%B1nda%C5%9F-4b87104b/).

![image](https://user-images.githubusercontent.com/58165365/280508284-55461410-a286-470b-b1ac-442e2741a2b0.png)

From his profile, there are no records of him working at `megaendustri`. He's also from Turkey. Impersonation? 🤔

- Doing a quick WHOIS lookup, I learnt that the domain could be legit as it's creation date is kinda old (1999). The company also seems to be in Turkey

![image](https://user-images.githubusercontent.com/58165365/280508423-9ea214ff-89bf-4096-9258-b75539759bc7.png)

- Confirming if the domain is malicious on [VT](https://www.virustotal.com/gui/url/d3f288c18224ecf1d598eef889b96aaaa10891229972a596552d9870ee0eb730) , I didn't get any hits.

![image](https://user-images.githubusercontent.com/58165365/280508682-0e817b3d-2e4c-42b2-8a79-2c91363bfd05.png)

With that in mind, I proceeded to inspect the document attached. On [VT](https://www.virustotal.com/gui/file/56d16f65b67c4b1ff6e09e36489d507838b92e3ecd8aab44ccbb00e280f933b0) , it had a reputation score of 35/60.

![image](https://user-images.githubusercontent.com/58165365/280508782-4be04a66-2265-4fac-9bcd-e200664c7004.png)

![image](https://user-images.githubusercontent.com/58165365/280508925-0387cf44-668b-40d4-9fa7-925b6ce51427.png)

Red 🚩🚩🚩. 

Lets look at the Originating IP rDNS.

![image](https://user-images.githubusercontent.com/58165365/280511751-d0cb7a75-ce46-4161-ba58-c1fa060e5600.png)

> _When you hear the term "Originating IP rDNS," it's referring to the domain name that is associated with the originating IP address of a network activity, based on a reverse DNS lookup._

Looking at it's WHOIS Records, we realise its origin is still in Turkey.

![image](https://user-images.githubusercontent.com/58165365/280511794-af53c3a9-912f-4ba9-bbc6-b4bbd72d4ff3.png)

Doing a quick lookup on VT i realised the IP is also linked to other uploaded `.eml` & `.msg` files.🚩🚩🚩

![image](https://user-images.githubusercontent.com/58165365/280511924-58db278c-6163-4da8-b485-68cf2001a7d7.png)

It's JARM Fingerprint is `15d3fd16d29d29d00042d43d00000071784fa9f8305ba9220d0a7894b6ff2c`

_~ Source_

- [Virus Total](https://www.virustotal.com/gui/ip-address/46.20.147.225/details)
- [Censys](https://search.censys.io/hosts/46.20.147.225)


I was abit curious what would happen when the document is opened. As i waited to head home to conduct an in depth analysis on my lab, i resolved to [AnyRun](https://app.any.run/), an online malware sandbox environment.

I uploaded the original `.eml` file and manually interacted with the document. A plaback can be found [here](https://app.any.run/tasks/b1aa2ce3-bdd0-4327-8511-119c794b359d/)

Lets start with the process graph.

![image](https://user-images.githubusercontent.com/58165365/280509222-9c12f8e0-e65d-44f0-8b0b-ef0380ec8544.png)


The task involved the execution of several processes, starting with Microsoft Outlook and Microsoft Word. These processes were launched with specific command lines and had parent-child relationships. After the execution of these processes, another executable file named "`arinze963004.exe`" was run twice. In addition to the process tree, there were modifications made to various files and registry keys.

The most interesting event in this task is the execution of the "`arinze963004.exe`" file. This file was run twice, with the second instance being a child process of the first instance. This behavior could indicate the presence of malware, as the file is located in the user's AppData\Roaming directory, which is a common location for malicious files. Another interesting point is the modification of registry keys related to Microsoft Office and Internet settings. These modifications could be an attempt to persist the malware or change the behavior of the system.

Lets take a look at the DNS requests

![image](https://user-images.githubusercontent.com/58165365/280509795-28115cfc-7550-46ca-9585-d505962fb849.png)

Taking a look at the suricata  rules fired, we see:

![image](https://user-images.githubusercontent.com/58165365/280509872-af66f18b-2593-4bd6-a5b5-494922582b71.png)

One notable rule is `Successful Credential Theft Detected` where an attempt to exfiltrate data via SMTP was made. The process image involved is "`C:\Users\admin\AppData\Roaming\arinze963004.exe`". This event is triggered by a malicious program.
            
Lets disect it. 

![image](https://user-images.githubusercontent.com/58165365/280510373-f0aa8c8c-8f97-48ae-8ffd-eae9f3884111.png)

The process observed is a file named "arinze963004.exe" running in the user's AppData\Roaming folder. It appears to be both the parent and child process. The process is associated with the `AgentTesla` malware, as indicated by the presence of the "`AGENTTESLA`" tag and the detection of `AgentTesla` using YARA rules.

Legitimate programs may read the computer name, environment values, and machine GUID from the registry for various purposes such as system configuration or user identification. They may also access Microsoft Outlook profiles for legitimate email operations. Additionally, legitimate programs may connect to SMTP ports for sending emails.

Malicious programs, like `AgentTesla`, can abuse these actions for malicious purposes. For example, they may steal personal data by accessing and exfiltrating files or by stealing credentials from web browsers. They may also read browser cookies to gather sensitive information. Additionally, they may read the settings of system certificates to bypass security measures or perform man-in-the-middle attacks. The presence of `AgentTesla` in this analysis suggests that the observed process is likely malicious.



Dissecting the process ,we get the malware configuration:

```json
{
  "Protocol": "smtp",
  "Host": "cp5ua.hyperhost.ua",
  "Port": "587",
  "Username": "arinzelog@saonline.xyz",
  "Password": "  7213575aceACE@#$   "
}
```

> _ A malware configuration refers to the set of instructions or parameters that dictate how the malware operates. These configurations can determine various aspects of the malware's behavior, including its communication methods, target information, exfiltration pathways, and more. The configuration is often stored in a structured format, such as JSON, XML, or a custom data structure, depending on the malware's design_

I also observed that the document could be linked to `CVE-2017-11882`

![image](https://user-images.githubusercontent.com/58165365/280510588-7d51797f-1188-4584-a49a-7b36eaa46038.png)

I was abit curious if there are other public submissions of the same exploit in the wild - linked to AgentTesla and indeed its still being actively exploited.

![image](https://user-images.githubusercontent.com/58165365/280510678-79f3c99e-d659-40a9-9726-e490d506af8f.png)

I tracked the CVE on [MALWARE bazaar](https://bazaar.abuse.ch/browse/tag/CVE-2017-11882/) and found recent submissions.

![image](https://user-images.githubusercontent.com/58165365/280510907-a0d47148-94c5-4223-8a5f-97713ed1930c.png)

In summary:

This is a phishing email  spreading the AgentTesla malware. This malware primarily spreads through phishing emails. It has been observed in spear phishing campaigns against multiple different industries, including energy, logistics, finance, and government. it was first seen in the wild around  1 January, 2014 and likely originating from Turkey. This confirms our earlier findings.

You can read more about it or track it [here](https://any.run/malware-trends/agenttesla)

### Module: Network Analysis

**Linking the analysis above with the network traffic around that period, can identify what really transpired?**

File can be downloaded here

- How long did it take to capture these packets? (25pts)

`BSidesNBI{00:01:36}`

- What is the victim's IP address? (25pts)

`BSidesNBI{192.168.100.56}`

- An executable was downloaded from a remote server. What is it's md5sum?	(25pts)

`BSidesNBI{d85ad0ba989beb96da04aae8d44add7f}`

- Can you provide the full URI used to download the executable. (25pts)	

`BSidesNBI{http://zang1.almashreaq.top/_errorpages/arinzezx.exe}`

- A DNS query was made to the domain above, can you identify the IP4 addresses included in the response? (25pts)

`BSidesNBI{104.21.70.74_172.67.221.26}`

-----

#### **Solution**

To answer the first question, simply load the pcap on wireshark and click on the `Statistics` tab and select `Capture File Properties` or `ctrl+alt+shift+c`

![image](https://user-images.githubusercontent.com/58165365/280516010-64acba4f-ee25-472e-89f5-ee534d2b916f.png)

![image](https://user-images.githubusercontent.com/58165365/280515844-9c635dd0-1c40-4d65-abb9-477fd0c5d8a8.png)

Moving on, we can also investigate the IPv4 addresses captured in this traffic. This way, we might identify the victim address. Simply head to `Statistic > Endpoints` and click on the IPv4 tab.

![image](https://user-images.githubusercontent.com/58165365/280516098-783e5e6c-1987-4e28-835f-b7b6337b873d.png)

![image](https://user-images.githubusercontent.com/58165365/280516228-3a2b4de3-2c48-44c2-a00f-a603e3c2797f.png)

Here we see most packets between `192.168.100.56` (A private IP) and `104.21.70.74` (Public IP). Going a step further, we can also get a glimpse of the protocols captured in this traffic. To do so, click on the `Statistics > Protocol Hierarchy` 

![image](https://user-images.githubusercontent.com/58165365/280516836-cb44efad-9217-4084-bf87-2e6a7eee53ca.png)

HTTP ...

![gif](https://media.giphy.com/media/KIxBn0rcMbhII/giphy.gif)

Filtering for http traffic, we see the local ip identified earlier downloading an executable from the remote server. (hxxp:[//]zang1.almashreaq.top/_errorpages/arinzezx.exe)

![image](https://user-images.githubusercontent.com/58165365/280516673-8863ea15-c0b1-424d-b28f-63bc45f39165.png)

We can export the executable for further analysis by simply: 

![image](https://user-images.githubusercontent.com/58165365/280517363-20cec1f8-34d0-431e-a2b2-494a9d39de81.png)

![image](https://user-images.githubusercontent.com/58165365/280517263-ea16268e-c734-4c44-ba36-79d7752cfd51.png)

This is the same executable we saw in our analysis ealier.

![image](https://user-images.githubusercontent.com/58165365/280517522-a37b155c-0572-45c8-a52f-802befa4237a.png)

Inspecting dns queries to the said malicious domain, we see it resolving to two addresses.

![image](https://user-images.githubusercontent.com/58165365/280517913-49ba57d9-6a52-4463-819f-f936a41d3727.png)

We can a step further and inspect the `smtp` exflitration attempt we found earlier. Network traffic consistent with SMTP communication was observed, aligning with the host cp5ua.hyperhost.ua on port 587. Given the context of the discovered malware configuration, it is highly probable that this SMTP communication represents data exfiltration from the compromised system.

![image](https://user-images.githubusercontent.com/58165365/280518897-e497bc0b-76cf-43b8-b703-38a9f7111001.png)

![image](https://user-images.githubusercontent.com/58165365/280518831-199584f4-4ac9-4df7-b428-85ef234fd1bd.png)

After the `220 TLS go ahead` message, the subsequent lines (with seemingly random characters and periods) represent the beginning of the TLS handshake. This is a binary process where the client and server exchange information to securely encrypt their communication. The text you're seeing is a mix of ASCII representation and raw bytes of this binary data.

Within that garbled text, you can find some readable information about a certificate, such as mentions of "Greater Manchester", "Salford", "Sectigo Limited", and "`hyperhost.ua`". This suggests that the certificate was issued by "Sectigo Limited" for "`hyperhost.ua`", and the organization is likely based in Salford, Greater Manchester.


Generally, this was a very easy and fun challenge. I hope you got to learn a thing or two.

![gif](https://media.giphy.com/media/wjBMdDqMCyxJoCJ2yZ/giphy.gif)

 Watch out for PART 2 of this blog post where i'll be diging into the intricacies of iOS Forensics Challenge.