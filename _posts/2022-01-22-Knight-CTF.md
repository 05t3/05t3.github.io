---

title: "KnightCTF-2022"
date: 2022-01-22 01:09:33 +0300
author: oste
image: /assets/img/Posts/knight_ctf_logo.png
categories: [CTF-TIME]
tags: [wireshark, osint, rev, steg]
---

Hey there. I'm glad you are here. Here is a writeup of the challenges i managed to solve in the first **Knight CTF** - an online jeopardy style Capture the Flag (CTF) competition hosted by the Knight Squad community from Bangladesh.

| CTF            | KnightCTF                                            |
| -------------- | ---------------------------------------------------- |
| CTF Start Time | 20 January 2022 09:00PM GMT +6                       |
| CTF End Time   | 21 January 2022 11:59PM GMT +6                       |
| CTF Time Event | [link](https://ctftime.org/event/1545/)              |
| Team           | Fr334aks-Mini                                        |
| Players        | 05t3, Winter, ★sW33t_1mPur1t13s, rs_on3, n3rd, Parsz |

We managed pos 120💪🏼out of 752 teams with 2750 points.

![image](https://user-images.githubusercontent.com/58165365/150654367-bbd59f1b-0188-4cc5-a2af-1ad6d5e53677.png)

Enough with the small talk...Let geet started.

---

# Networking

## How's the Shark?

Find the flag from the following. [Download Link](https://kctf2022.nstechvalley.com/knight-ctf-2022-challenges/Network/How's%20the%20Shark/data.pcapng)

Whenever i get a pcap file for analysis, the first thing i do is analyze the protocal hierarchy statistics to see what protocols have been used frequently. In this case, HTTP protocol is a good place to start since traffic is mostly unencrypted. We see that we have some `png` files that we'd be interested in analyzing.

![image](https://user-images.githubusercontent.com/58165365/150390712-c3ab95cb-0111-47a7-9c1b-952c5adf89d6.png)

Navigating to `File> Export objects> HTTP`, we can then proceed to export all the objects for further analysis.

> Please note that in some cases, lets say you are analyzing Malicious traffic and find an executable file eg, `fsdfd.exe` , be careful not to execute it on your host OS...Might be good to study the executable in a sandbox environment.

![image](https://user-images.githubusercontent.com/58165365/150390612-31754a5b-233f-4f1d-a257-3aac02b001f0.png)

Out of the 18 exported images, we can easily get out first flag. This was easy, right?

![image](https://user-images.githubusercontent.com/58165365/150390229-4b26b973-ff8c-4fce-a6c1-07b5eb6bc11c.png)

FLAG: `KCTF{A_ShARk_iN_tHe_WirE}`

## Find the Flag

_Find the flag from the following file._

Ok, so this challenge was also relatively easy to solve, I used the `strings` command to print out any NUL-terminated sequence of atlest 20 characters. (By default, its 4 characters).

```bash
➜  file file
file: data
➜  wc file
 10184  18859 758654 file
➜  strings -n20 file

//redacted
```

From the output, we get a hint of flag.txt and some base64 string.

![image](https://user-images.githubusercontent.com/58165365/150652543-a92a245a-9c8f-44e2-af9b-2544d7156837.png)

Well, you can decode the string via the cli or an online converter like [Cyberchef](https://gchq.github.io/CyberChef/) as shown below.

```bash
➜  echo S0NURntGVFBfUDRDSzNUX0M0cFR1cjNfVXNJbmdfV2lyZVNINFJLfQo= | base64 -d
KCTF{FTP_P4CK3T_C4pTur3_UsIng_WireSH4RK}
```

or

![image](https://user-images.githubusercontent.com/58165365/150391423-3be6f692-56a5-4ce1-a520-299d24d6996e.png)

FLAG: `KCTF{FTP_P4CK3T_C4pTur3_UsIng_WireSH4RK}`

## Compromised CTF Platform

_I created a CTF platform of my own & hosted on a server. It seems like someone got access to my site. I have captured the traffic. Help me find out who he is._

> N.B: I am a n00b developer.

_What is the username & password that the attacker got._

Awesome, so like in the previous pcap file, we start by analyzing the Protocal Hierarchy Statistics.

![image](https://user-images.githubusercontent.com/58165365/150653735-f1736b7e-638b-40ef-8338-0b15bf530599.png)

`MySQL's` malformed packets sure sounds interesting...Could this be an indicator of sql injection attempts? Lets start by filtering out MySQL packets

![image](https://user-images.githubusercontent.com/58165365/150645654-07900589-f3ed-45c3-ae21-dceaac7a1430.png)

Looking disecting one packet, we find an error message that `192.168.1.8` is not allowed to connect to the DB server. At this point, i kinda figured out that this was probably the attacker and `192.168.1.4` was the server. Just to be sure , we can look at the pcap's endpoints (`Statistics > Endpoints`)

![image](https://user-images.githubusercontent.com/58165365/150646024-0fbb56ea-8385-4f6b-b321-37f799e56d3f.png)

So yeah, we have a huge number of packets originating from this two Ip's. Cool...lets proceed and look at the export objects under `File> Export objects> HTTP`

![image](https://user-images.githubusercontent.com/58165365/150648327-730c6481-b47c-4af2-b949-0f91c9a2749c.png)

After sometime of inpecting the objects, the login pages seem interesting.
The first four `login.php` packets had the following username's and passwords, meaning that he tried to login using default credentials.

```
admin_admin123
test_test
admin_password
admin_admin
```

Looking at the traffic, we see a POST request with the username and password. The hacker is then redirected to http://192.168.1.4/hackerz_arena/includes/dashboard.php after a successful login. Seems he got lucky on packet `666` by using `demo_demo` as the username and password.

![image](https://user-images.githubusercontent.com/58165365/150648251-8e1fdcde-5adc-421d-adfc-7253aeb6521c.png)

FLAG: `KCTF{demo_demo}`

## Hashed Password

_What is tareq's password hash?_

> Use Compromised CTF Platform's Challenge file to analyze.

While still at it, we see a fairly large number of packets where the attacker tried some sql injection payloads. Seems like tareq's CTF platform was vulnerable to sql injection🤷🏼‍♂️🤷🏼‍♂️? Lets see if this actually worked.So what i did was to export all the objects, including images to see if we can find anything useful.

![image](https://user-images.githubusercontent.com/58165365/150648720-2ecb9a60-b3d7-4056-b882-3524347cac5e.png)

Opening the files with a text editor and inspecting the code will give you preety much of the next series of questions flags. In this case, we can see the attacker managed to dump hashes for two users, DEMO & TAREQ.

![image](https://user-images.githubusercontent.com/58165365/150652584-eeb11ff5-b1b8-4e45-a875-eb7b415e320d.png)

FLAG: `KCTF{TAREQ : $2Y$10$XVKEZO/NKM4KE073CPTEG.VKFTHMH1CCDPRDD5JWYWKFEZ6GZKZN}`

## Attacker

_What is the attacker name?_

> Use Compromised CTF Platform's Challenge file to analyze.

![image](https://user-images.githubusercontent.com/58165365/150480547-4f35fdeb-1ebf-48c7-904a-04f706825bd9.png)

FLAG: `KCTF{MOSH}`

## PHP Version

_What version of php the server is using?_

> Use Compromised CTF Platform's Challenge file to analyze.

From `about`, the server spit the version of webserver and php being used

![image](https://user-images.githubusercontent.com/58165365/150480947-869c43b5-ce74-4ed9-97f7-a7afc0cbb6ff.png)

FLAG: `KCTF{PHP/7.4.27}`

## Vuln Columns

_How many columns were vulnerable?_

> Use Compromised CTF Platform's Challenge file to analyze.

Well, i'm no web expert but i kept seeing the value 4 in most of the payloads he tried, so i kinda guessed on this one but hey, were chasing flags, right? 😅😅

![image](https://user-images.githubusercontent.com/58165365/150649787-7fcea993-c917-4db8-9c02-1f49ce81d354.png)

FLAG: `KCTF{4}`

## Database Flag

_What is the retrived flag from database?_

> Use Compromised CTF Platform's Challenge file to analyze.

From analysis, the attacker used the following payload to dump the base64 encoded flag from `vulnerable`

`users.php?id=-1' Union Select 1,2,group_concat(flag),4,5,6,7,8 from vulnerable-- `

![image](https://user-images.githubusercontent.com/58165365/150481813-44d99567-2d54-4534-aa5b-99c41ab2a594.png)

Using [Cyberchef](https://gchq.github.io/CyberChef/), we can decode the string and get the flag as shown below.

![image](https://user-images.githubusercontent.com/58165365/150652624-03230a44-7bd0-4b02-872e-ed772f55b34b.png)

FLAG: `KCTF{SqL_1Nj3C7i0n}`

## KCTF

_It's all about kctf._

> Use Compromised CTF Platform's Challenge file to analyze.

Looking at the exported images, we get the final flag.

![image](https://user-images.githubusercontent.com/58165365/150396670-49ad518c-487f-43fb-a4fa-652e8cefda99.png)

FLAG: `KCTF{Ev3rY_UsEr_1nPuT_SH0uLD_B3_S4niT1z3D}`

# Forensics

## The Lost Flag

_We recovered a image file from an incident. There might be something interesting in the file. Give it a try._

Downloaded the file and tried to look at its properties.

```bash
➜  wget https://kctf2022.nstechvalley.com/knight-ctf-2022-challenges/Digital%20Forensics/Lost%20Flag/Lost%20Flag%20.png
--2022-01-20 15:23:23--  https://kctf2022.nstechvalley.com/knight-ctf-2022-challenges/Digital%20Forensics/Lost%20Flag/Lost%20Flag%20.png
Resolving kctf2022.nstechvalley.com (kctf2022.nstechvalley.com)... 192.99.167.83
Connecting to kctf2022.nstechvalley.com (kctf2022.nstechvalley.com)|192.99.167.83|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 65755 (64K) [image/png]
Saving to: ‘Lost Flag .png’

Lost Flag .png                                      100%[================================================================================================================>]  64.21K   130KB/s    in 0.5s

2022-01-20 15:23:25 (130 KB/s) - ‘Lost Flag .png’ saved [65755/65755]

➜  file Lost\ Flag\ .png
Lost Flag .png: PNG image data, 1200 x 600, 8-bit/color RGBA, non-interlaced
```

I tried various steg tools to see if there might have been some hidden file innit but i did not succed. Using a tool called `stegoveristas`, i tried to transform the image provided.

`stegoveritas -imageTransform -extractLSB Lost\ Flag\ .png`

- `imageTransform` - Perform various image transformations on the input image and save them to the output directory
- `extractLSB` - Extract a specific LSB RGB from the image. Use with -red, -green, -blue, and -alpha

After running the command above, we should an output folder with several transformations of the image.

![image](https://user-images.githubusercontent.com/58165365/150417436-0024e13a-dc9c-47f7-a721-1a4dd3bd1121.png)

If you start looking at them one by one, you get the hidden flag.

![image](https://user-images.githubusercontent.com/58165365/150417625-ec411772-f579-4d4d-bc39-df35e8370a31.png)

FLAG: `KCTF{Y0U_F0uNd_M3}`

## Compromised FTP

_We detected some malicious activity on our FTP server. Someone has performed bruteforce attack to gain access to our FTP server. Find out the Compromised FTP account username & the attacker IP from the following._

This challenge was preety easy. You are provided with a long ftp log file and expected to find out the compromised FTP account username...If try `cating` the contents of the file, you'll get a bunch of :

`FAIL LOGIN: Client "::ffff:192.168.1.7"`

What this challenge was testing is you are capable of filtering out only important information from a large data set. In my case, i used the grep command to filter out lines with `"OK LOGIN"` status.

```bash
➜  wget https://kctf2022.nstechvalley.com/knight-ctf-2022-challenges/Digital%20Forensics/Compromised%20FTP/ftp.log
--2022-01-20 15:36:38--  https://kctf2022.nstechvalley.com/knight-ctf-2022-challenges/Digital%20Forensics/Compromised%20FTP/ftp.log
Resolving kctf2022.nstechvalley.com (kctf2022.nstechvalley.com)... 192.99.167.83
Connecting to kctf2022.nstechvalley.com (kctf2022.nstechvalley.com)|192.99.167.83|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 480478 (469K) [text/x-log]
Saving to: ‘ftp.log’

ftp.log                                             100%[================================================================================================================>] 469.22K   385KB/s    in 1.2s

2022-01-20 15:36:41 (385 KB/s) - ‘ftp.log’ saved [480478/480478]
➜  wc ftp.log
  5880  67614 480478 ftp.log
➜  grep "OK LOGIN" ftp.log
Mon Jan  3 15:24:13 2022 [pid 5399] [ftpuser] OK LOGIN: Client "::ffff:192.168.1.7"
```

FLAG: `KCTF{ftpuser_192.168.1.7}`

# OSINT

## Explosion In Front Of Bank Of Spain

_One of my friend sent me the picture and told me that, there was an explotion in front of the Bank of Spain by some robbers a few days ago. After hearing that, I googled about incident. But I discovered that, The picture he gave is not the picture of Bank Of Spain. So, now I want to know the exact location of the picture so that I can know about the incident of that explotion. Can you please help me to find that place? Please send me the coordinates of that location if you can figure it out._

Honestly, when i saw this heading , it gave me some **Money Heist** vibe. Well, lets actually download the file and have a look at it.

![image](https://user-images.githubusercontent.com/58165365/150650470-249ff6a2-9591-4ad2-90fa-712eb534d52d.png)

Doing a quick search on [google](https://www.google.com/search?q=explosion+scene+in+money+heist+outside+the+bank+of+spain&client=firefox-b-e&sxsrf=AOaemvJrieHYVUKYw5WiORoWIZbmNaOmnQ:1642874703343&source=lnms&tbm=isch&sa=X&ved=2ahUKEwjJi-mY-cX1AhUD14UKHasoDwQQ_AUoAXoECAEQAw#imgrc=HRpHbOkKMbyDwM&imgdii=VAC3uynJsKd71M), shows the exact same photo

![image](https://user-images.githubusercontent.com/58165365/150652643-c904aeac-e3a8-4b16-a346-0c63f2a3f48b.png)

Well, reading the [article](https://news.otakukart.com/490403/la-casa-de-papel-season-4-release-date-spoilers-streaming-details-and-trailer/), it doesn't give much but spoilers. So we move on. Doing a quick google search on where exactly they shot the whole scene of the Royal Mint, we get a [blog](https://www.klook.com/en-PH/blog/money-heist-film-locations/) revealing the location as `Ministerio de Fomento (Ministry of Public Works and Transport) as the Bank of Spain`

![image](https://user-images.githubusercontent.com/58165365/150650739-fba8fae0-600f-43d0-8050-e92062259a24.png)

![image](https://user-images.githubusercontent.com/58165365/150652658-1419e7ca-d70b-45c2-8ee2-d52ba43faa06.png)

We now have some solid stuff, with this in mind, we can head over to google maps and locate where the ministry is located.

[Google Map Location](https://www.google.com/maps/place/Ministerio+de+Transportes,+Movilidad+y+Agenda+Urbana/@40.4442164,-3.695797,17z/data=!3m1!4b1!4m12!1m6!3m5!1s0xd4228faa4f3ceff:0x45144593d64bd75f!2sMinisterio+de+Transportes,+Movilidad+y+Agenda+Urbana!8m2!3d40.4442164!4d-3.6936083!3m4!1s0xd4228faa4f3ceff:0x45144593d64bd75f!8m2!3d40.4442164!4d-3.6936083)

![image](https://user-images.githubusercontent.com/58165365/150653876-af22d2e1-1575-4198-9388-2f2d599cac13.png)

FLAG: `KCTF{40.4442164,-3.695797}`

# Steg

## Bangladesh

_My friend John was interested to know my country . He told me that to give him some images and articles about my county . I gave him some images and articles. In one image I provided some hidden data but he can not find hidden data . I told him Always remember 3 number sum equal to a game-changer. but he can not find hidden data . For that reason, I gave him that game-changer key._

I used a tool called stegseek to try bruteforce the passphrase required to extract the contents of the image. `cating` the contents of the hidden file reveals the flag.

```bash
➜  file Bangladesh.jpg
Bangladesh.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 1280x652, components 3
➜  stegseek Bangladesh.jpg /usr/share/wordlists/rockyou.txt -xf output.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "2262")
[i] Original filename: "not_real".
[i] Extracting to "output.txt".
➜  steg cat output.txt
KCTF{Do_We_Remember_Cicada_3301}#
```

FLAG: `KCTF{Do_We_Remember_Cicada_3301}`

<!-- # Misc

## Look Closely

![image](https://user-images.githubusercontent.com/58165365/150550037-1cbdfdda-a049-40d9-9b4f-2a28b6891dae.png)




## The Hungry Dragon

Last night, the Knight Squad members were having relax and enjoying doughnuts and sweets together on the roof of their castle. Suddenly, a hungry dragon attacked on them and ate some of their food. The Knights were angry and then they all attacked on the dragon and managed to capture it. And now they are handing over the dragon to you. Can you figure out how many doughnuts and sweets were eaten up by the dragon? -->

# Reverse Engineering

## Baby Shark

_During my holiday in Bahamas, I met a baby shark. The shark wanted to sing me something but couldn't. Can you sing that for me?_

I'm gonna be honest with you...I have always thought Rev category was so difficult to solve but hey, i gave it a shot and got lucky with this one. I simply did a quick google search on how to reverse engineer a `.jar` file and i found a tool called [JD_GUI](https://java-decompiler.github.io/) that could do the trick.

So, did a quick installation and launched it. Looking at the different classes, `Flag.class` sounded promising but contained a fake flag.

![image](https://user-images.githubusercontent.com/58165365/150553705-73c45da5-6dee-4ee3-9bd2-653d8b6f8e52.png)

Drilling down on constants, i found two classes whereby the strings class had a String called `String _0xflag` containing a base64 string. (*I'm not even sure i used the correct programmatic expression explaining this but.....you feel me, right?*😂😂)

![image](https://user-images.githubusercontent.com/58165365/150553584-c5283d43-9f11-44ce-80ec-1dd8a6d8be77.png)

You can proceed to decode this with Cyberchef or the cli and get the flag

![image](https://user-images.githubusercontent.com/58165365/150553834-27eae156-0a55-4d27-a28b-d3775a9cf4e9.png)

FLAG: `KCTF{7H15_W@5_345Y_R16H7?}`

---

That brings me to the end of this writeup and thanks for reading through.😃 I'd be interested to see how you solved the same challenges. Feel free to reach out to me on Twitter [**@oste_ke**](https://twitter.com/oste_ke)
