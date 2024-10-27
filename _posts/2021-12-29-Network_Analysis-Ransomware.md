---

title: "Network Analysis – Ransomware"
date: 2021-12-29 01:09:33 +0300
author: oste
image: /assets/img/Posts/networkanalysisransomware.png
categories: [BTLO, Security Operations]
tags: [dridex, wireshark, malware, ransomware]
---

---

#### What is the operating system of the host from which the network traffic was captured? (Look at Capture File Properties, copy the details exactly) (3 points)

To score this, you need to go the menu bar and select statistics and choose `Capture File Properties`. From here, you can easily get the OS information as shown in the screenshots below

![image](https://user-images.githubusercontent.com/58165365/147353034-e3124b4f-65ae-4028-800f-721dd4ad78a5.png)

![image](https://user-images.githubusercontent.com/58165365/147352112-6e1f6425-85de-491b-8ba6-6ecb0cedea97.png)

`32-bit Windows 7 Service Pack 1, build 7601`

---

#### What is the full URL from which the ransomware executable was downloaded? (3 points)

Headed over to `File > Export objects > HTTP objects` and you'll find one packet with an executable file called safecrypt. Manually exploring the packet, you'll get the full URL in the GET request. Alternatively, you can choose to follow the http stream and get the answer.

![image](https://user-images.githubusercontent.com/58165365/147260450-b994fc90-a188-4f5a-8207-8e726f6fb17f.png)

![image](https://user-images.githubusercontent.com/58165365/147260332-700e614e-6d03-498e-9edf-5e2deaaf32ca.png)

`http://10.0.2.15:8000/safecrypt.exe`

---

#### Name the ransomware executable file? (2 points)

`safecrypt.exe`

---

#### What is the MD5 hash of the ransomware? (2 points)

On the linux terminal, you can use a tool called `md5sum` to get the hash. If you are solving this on windows, you can try tools like [hashtab](https://download.cnet.com/HashTab/3000-2094_4-84837.html), [hashtool](https://www.binaryfortress.com/HashTools/) among others. But inorder to get the hash, you need to export the executable we saw from the previous screenshot.

```bash
➜  md5sum safecrypt.exe
4a1d88603b1007825a9c6b36d1e5de44  safecrypt.exe
➜
```

`4a1d88603b1007825a9c6b36d1e5de44`

---

#### What is the name of the ransomware? (2 points)

In order to get the name, we can lookup if this hash has shown up in malware databses. Using Virustotal for example, we can search the hash of the binary or manually upload it. In this, we find various security vendors recognizing it as TeslaCrypt.

![image](https://user-images.githubusercontent.com/58165365/147265176-86570598-a701-4693-b81c-f2b94290d7e8.png)

`TeslaCrypt`

---

#### What is the encryption algorithm used by the ransomware, according to the ransom note? (2 points)

![image](https://user-images.githubusercontent.com/58165365/147261135-0b76fd00-2269-41ea-ab16-f0c6b9273720.png)

`RSA-4096`

---

#### What is the domain beginning with ‘d’ that is related to ransomware traffic? (3 points)

I filtered dns traffic then manually inspected the DNS query's

![image](https://user-images.githubusercontent.com/58165365/147262778-335e532a-3bee-4060-97c6-4ce7805f3703.png)

`dunyamuzelerimuzesi.com`

---

#### Decrypt the Tender document and submit the flag (3 points)

I did some digging and found a command line tool that can decrypt files encrypted by the ransomware. You can download it at [Mcafee](https://www.mcafee.com/enterprise/en-in/downloads/free-tools/tesladecrypt.html). Instructions on how to use the tool can be found [here](https://www.mcafee.com/enterprise/en-in/downloads/free-tools/how-to-use-tesladecrypt.html). Decrypting was relatively easy and opening the document, we get the flag.

```
$ ./tesladecrypt.exe -h
usage: tesladecrypt.exe [-h] [--version] [-l] [-r] [-d] target_directory

positional arguments:
  target_directory  Directory to search for encrypted teslacrypt files

optional arguments:
  -h, --help        show this help message and exit
  --version         Get version information
  -l, --list        List all encrypted TeslaCrypt files
  -r, --recursive   Process files in sub-directories
  -d, --del         Delete encrypted files after decryption
```

```
$ ./tesladecrypt.exe -d E:\
>
Decrypting [ Tender.pdf.micro ] - OK and DELETED Encrypted File
```

![image](https://user-images.githubusercontent.com/58165365/147355321-54522f2a-5d34-4c51-adee-d67cebbd325c.png)

`BTLO-T3nd3r-Fl@g`
