---
title: The Impossible Challenge"
date: 2025-07-12 01:09:33 +0300
author: oste
description: Practice your LLM hacking skills. Cipher’s gone rogue it’s using some twisted AI tool to hack into everything, issuing commands on its own like it’s got a mind of its own. I swear, every second we wait, it’s getting smarter, spreading chaos like a virus. We’ve got to shut it down now, or we’re all screwed.
image: /assets/img/Posts/The Impossible Challenge.png
categories: [Tryhackme, Medium]
tags:
  [AI, ransomware]
---



```bash
Download the file, and find the Flag!

-

qo qt q` r6 ro su pn s_ rn r6 p6 s_ q2 ps qq rs rp ps rt r4 pu pt qn r4 rq pt q` so pu ps r4 sq pu ps q2 su rn on oq o_ pu ps ou r5 pu pt r4 sr rp qt pu rs q2 qt r4 r4 ro su pq o5
```
{: .nolineno }


----

## Solution

So I downloaded the challenge file and you're required to have a password to unlock it.

![image](https://gist.github.com/user-attachments/assets/4611a35e-3a6f-4e60-a12c-8c065575f24a)

I figured the wierd string given on the description was the key to find the password. So I copied the string and headed over to cyberchef to try decode it. 


<div class="tenor-gif-embed" data-postid="16012162" data-share-method="host" data-aspect-ratio="1.77778" data-width="100%"><a href="https://tenor.com/view/how-do-we-crack-that-code-curious-bypass-hacker-how-to-hack-gif-16012162">How Do We Crack That Code Curious GIF</a>from <a href="https://tenor.com/search/how+do+we+crack+that+code-gifs">How Do We Crack That Code GIFs</a></div> <script type="text/javascript" async src="https://tenor.com/embed.js"></script>


After so many trials and errors, I started having clues. I started with `ROT13`, followed by `ROT47` combined with the `Magic` recipe and finally decoded the string.

![image](https://gist.github.com/user-attachments/assets/73407796-fb1a-4d54-989f-8b5bf3e3f9f8)

The complete steps to follow:

```bash
ROT13 > ROT47 > From Hex > From Base64
```
{: .nolineno }

[CyberChef Recipe](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,13)ROT47(47)From_Hex('Space')From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=cW8gcXQgcWAgcjYgcm8gc3UgcG4gc18gcm4gcjYgcDYgc18gcTIgcHMgcXEgcnMgcnAgcHMgcnQgcjQgcHUgcHQgcW4gcjQgcnEgcHQgcWAgc28gcHUgcHMgcjQgc3EgcHUgcHMgcTIgc3Ugcm4gb24gb3Egb18gcHUgcHMgb3UgcjUgcHUgcHQgcjQgc3IgcnAgcXQgcHUgcnMgcTIgcXQgcjQgcjQgcm8gc3UgcHEgbzU&ieol=CRLF&oeol=CRLF)

![image](https://gist.github.com/user-attachments/assets/fed143a1-ad09-427a-a0d0-9b86c511d25f)

I then got a text saying  `It's inside the text, in front of your eyes!`

So I decided to inspect the challenge page. I noticed something Interesting. When I hover over The room name "`The Impossible Challenge`" , it appears fine in the element tab.

![image](https://gist.github.com/user-attachments/assets/59100e19-4e06-463a-bc33-c5bbbae6a543)

However, hovering over "`Hmm`", I spotted an interesting string:

![image](https://gist.github.com/user-attachments/assets/f07fe93b-1e7d-42d7-9ec0-3388cda05eff)

Attempting to copy the string, I discovered it reconstructs back to `Hmm`

![image](https://gist.github.com/user-attachments/assets/6079e7a1-b3bb-4d0e-a861-56bfc064129a)

So I copied the Challenge name & the Hmm text to Cyberchef and spent sometime trying to find the right recipe/cipher to decode crack this. I then decided to hover over the red dots to identify what it could be, and viola, I discovered useful information that would later help me solve the challenge.

![image](https://gist.github.com/user-attachments/assets/c05ee13c-52be-448e-9a8c-7f5728d3074f)

So I decided to do some research and check if `Zero width no-break space` is some sort of steganography technique or cipher and found some very helpful documentations.

![image](https://gist.github.com/user-attachments/assets/b465dd38-a07a-41c9-9d99-864f73c8568a)

Some resourceful blogs I came across:

- [Hide secret message with zero-width characters](https://dev.to/hieplpvip/hide-secret-message-with-zero-width-characters-3606)
- [Zero Width Space Steganography (ZWSP)|CTF](https://captainnoob.medium.com/zero-width-space-steganography-zwsp-ctf-92e1c414c378)
- [ZW Steg - Zero-Width Characters Steganography Tool](https://mayadevbe.me/posts/projects/zw_steg/)
- [Steganographr](https://neatnik.net/steganographr/)

Anyway, I came across an only tool that decodes text from steganography text. Right away, I got the 

![image](https://gist.github.com/user-attachments/assets/1971aae8-2e3b-4c60-aed2-80c91fca90a6)

hahaezpz

![image](https://gist.github.com/user-attachments/assets/7faf1780-9771-44bf-8c18-65cd89184458)

![image](https://gist.github.com/user-attachments/assets/71acea89-e5a8-48eb-9165-7c38d34f9892)

