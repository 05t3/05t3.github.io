---

title: "Metaspike Email Forensics CTF"
date: 2022-02-15 01:09:33 +0300
author: oste
image: /assets/img/Posts/Metaspike.png
categories: [CTF-TIME]
tags: [mbox, thunderbird, RFC, Takeout]
---

# To Read, or Not to Read (100)

> You have exported the contents of a Gmail mailbox via Takeout. Looking at the exported data1, you suspect that one of the top-level messages may have been read by the perpetrator. Enter its Message-ID (i.e., the value of its "Message-ID:" header field as defined in RFC 5322).

> SHA-256: F77C28E0DCC8481279CF99F79DCAD3E5 EAE755884D335BD61B7ED13C34E8FD28

**To Modify**

<!-- WE first begin by checking the integrity of the file downloaded by comparing it to the SHA given.

![image](https://user-images.githubusercontent.com/58165365/153607411-268c8040-ddfb-420b-babd-a0ef370188f5.png)

WE can see that it corresponds.

According to [fileinfo](https://fileinfo.com/extension/mbox):

> _An MBOX file is an email mailbox saved in a mail storage format used for organizing email messages in a single text file. It saves messages in a concatenated format where each message is stored after another, starting with the "From" header. MBOX files were originally used by Unix hosts but are now supported by other email applications, including Apple Mail and Mozilla Thunderbird._

Some of the tools you can use to open `.mbox` files include:

| OS      | Tool                                                               | Free/Paid        |
| ------- | ------------------------------------------------------------------ | ---------------- |
| Windows | [Mozilla Thunderbird](https://www.thunderbird.net/en-US/download/) | Free             |
|         | CoolUtils Total Thunderbird Converter                              | Free Trial       |
|         | Kernel MBOX Viewer                                                 | Free Trial       |
|         | Aryson MBOX to PST Converter                                       | Free Trial       |
|         | Regain MBOX to PST Converter                                       | Free Trial       |
|         | EdbMails                                                           | Free Trial       |
| MAC     | Mozilla Thunderbird                                                | Free             |
|         | Apple Mail                                                         | Included with OS |
|         | Microsoft Outlook 365                                              | Free+            |
| Linux   | Mozilla Thunderbird                                                | Free             |
|         | GNOME Evolution                                                    | Free             |

- Google Takeout (Google Takeaway in some languages, on the site itself called "Download your data") is a project by the Google Data Liberation Front that allows users of Google products, such as YouTube and Gmail, to export their data to a downloadable archive file.

From the instructions given, they have mentioned RFC 5322 (Who even reads that😫). For those not familiar with what RFC's are:

> _RFC (Request for Comments) is a publication in a series, from the principal technical development and standards-setting bodies for the Internet, most prominently the Internet Engineering Task Force._
> ~Source: [Wikipedia](https://en.wikipedia.org/wiki/Request_for_Comments)

Shortly put:

> _RFC documents contain technical specifications and organizational notes for the Internet._
> ~Source: [IETF](https://www.ietf.org/standards/rfcs/)

At first, i downloaded a tool called SysInfo MBOX File viewer to test whether i can access the mails.

![image](https://user-images.githubusercontent.com/58165365/153643529-ebf65db8-d061-4517-b171-ae70d8fd0a22.png)

I did see the mails which are 1812 in total but where do i start? I did not seem to find a way to know if an email was opened or not. So next thing i did was look for a tool that can do that. Thunderbird? Lets check it out.

Parsing through the emails, i narrowed down senders

- Brian Dean
- BYRDIE
- BYRDIE Insider
- Craighill
- Dan Lewis
- Doug at Segment
- Evergreen by The Spruce
- Grovemade
- Health tip of the Day by Verywell
- Investopedia
- Jeff Sauer
- Kylee from Segment
- Madewell
- Madewell insider
- Morning Brew
- NextDraft
- Nicole at Segement
- Overstock | Spring Black Friday
- Overstock.com
- PLAE
- Serious Eats
- Simply Recipes
- Team BigMarker
- Tech Today by Lifewire
- The Balance Today
- The New York Times
- The Spruce Daily
- theSkimm
- Tommy Griffith
- Treehugger
- Troy Hunt
- UPLIFT Desk
- Verywell Health Today
- What I Learned About Today

# Resources

- https://www.forensicfocus.com/articles/email-forensics-investigation-techniques/
  https://www.howtogeek.com/709718/how-to-open-an-mbox-file-in-mozilla-thunderbird/
  https://business.tutsplus.com/articles/did-someone-open-and-read-your-email--cms-29389
  https://support.google.com/accounts/thread/124683174/best-way-to-view-google-takeout-gmail-and-other-products?hl=en
  https://datatracker.ietf.org/doc/html/rfc5322 -->
