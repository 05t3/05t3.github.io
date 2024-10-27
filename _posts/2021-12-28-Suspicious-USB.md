---

title: "Suspicious USB"
date: 2021-12-28 01:09:33 +0300
author: oste
image: /assets/img/Posts/suspicious-usb.png
categories: [BTLO, Digital Forensics]
tags:
  [
    autorun.inf,
    pdf-parser,
    virustotal,
    magic number,
    file,
    file signature,
    pdfid,
    pdftool,
    peepdf,
  ]
---

Today we will be tackling a task from BTLO called Suspicious USB. At the time of writing, the challenge is retired. This was fun tackling and and researching new stuff that i was well acquinted with.

> **Scenario**

> _One of our clients informed us they recently suffered an employee data breach. As a startup company, they had a constrained budget allocated for security and employee training. I visited them and spoke with the relevant stakeholders. I also collected some suspicious emails and a USB drive an employee found on their premises. While I am analyzing the suspicious emails, can you check the contents on the USB drive?_

---

#### What file is the autorun.inf running? (3 points)

Lets first take a moment to know what autorun.inf is. Shall we? I did some reasearch from various sites like [Wikipedia](https://en.wikipedia.org/wiki/Autorun.inf), [Bleeping Computers](https://www.bleepingcomputer.com/forums/t/592940/should-i-remove-autoruninf-in-my-usb/) to dig some info on the same...Heres what i got.

An autorun.inf file is a text file that can be used by the AutoRun and AutoPlay components of Microsoft Windows operating systems. For the file to be discovered and used by these component, it must be located in the root directory of a volume.

> _Quick History_
> Wayback in Windows 95, the AutoRun component was introduced. AutoRun enabled application CD-ROMs to automatically launch a program which could then guide the user through the installation process. By placing settings in an autorun.inf file, manufacturers could decide what actions were taken when their CD-ROM was inserted. The simplest autorun.inf files have just two settings: one specifying an icon to represent the CD in Windows Explorer (or "My Computer") and one specifying which application to run.if you want to learn more about AutoPlay device types, feel free to read more [here](https://en.wikipedia.org/wiki/AutoPlay#Device_types)

Well, TL;DR?

`autorun.inf` is an ASCII text file located in the root folder of a CD-ROM or other volume device medium.

The structure is that of a classic `Windows.ini` file, containing information and commands as "key=value" pairs, grouped into sections.These keys specify:

- The name and the location of a program to call when the medium is inserted (the "AutoRun task").
- The name of a file that contains an icon that represents the medium in Explorer (instead of the standard drive icon).
- Commands for the menu that appears when the user right-clicks the drive icon.
- The default command that runs when the user double-clicks the drive icon.
- Settings that alter AutoPlay detection routines or search parameters.
- Settings that indicate the presence of drivers.

According to [Bleeping Computer's Forumn](https://www.bleepingcomputer.com/forums/t/592940/should-i-remove-autoruninf-in-my-usb/) , "_Autorun.inf can be exploited to allow a malicious program to run automatically without the user knowing since it is a loading point for legitimate programs. Such an exploit involves malware that modifies/loads an autorun.inf file into the root folder of all drives (internal, external, removable) along with a malicious executable. When removable media is inserted (mounted), autorun looks for autorun.inf and automatically executes the malicious file to run silently on your computer. Since autorun.ini can be a legitimate file which other legitimate programs depend on, the presence of that file may not always be an indication of infection. Usually when it is bad, there will be other signs or symptoms of infection to include other malicious files._"

For this reason, This functionality was removed in Windows 7 and a patch for Windows XP and Vista was released on August 25, 2009 and included in Microsoft Automatic Updates on February 8, 2011.

Ok, i know this is starting to sound so technical and boring...Let's have a look at the structure of a simple `Autorun.inf` file.

```bash
[autorun]
open=setup.exe
icon=setup.exe,0
label=My install CD
```

This simple autorun.inf file specifies setup.exe as the application to run when AutoRun is activated. The first icon stored within setup.exe itself will represent the drive in Explorer.

In this case, when i cat the contents of the autorun.inf provided here is what i got:

```bash
➜  file autorun.inf
autorun.inf: Microsoft Windows Autorun file
➜  file README.pdf
README.pdf: PDF document, version 1.7 (password protected)
➜  cat autorun.inf
[autorun]
open=README.pdf
icon=autorun.ico
```

- `[autorun]`
  The autorun section contains the default AutoRun commands. _An autorun.inf file must contain this section to be valid. Keys allowed are:_

- `open=`[exepath\]exefile [param1 [param2 ...]]
  _It Specifies the path, file name and optional parameters to the application that AutoRun launches when a user inserts a disc in the drive. It is the CreateProcess function that is called by AutoRun. Note that if the application name includes spaces the path should be enclosed in double quote, e.g. open=""spread sheets.exe""_
- `icon=`iconfilename[,index]
  _The name of a file resource containing an icon. This icon replaces the standard drive icon in Windows Explorer. This file must be in the same directory as the file specified by the open key.
  label=text_

More keys can be found [here](https://en.wikipedia.org/wiki/Autorun.inf). From the above, we can deduce that the file being run is README.pdf

`README.pdf`

---

#### Does the pdf file pass virustotal scan? (No malicious results returned) (2 points)

Here is how i went about scoring this correctly. I first took the sha256sum of the `.pdf` file and looked it up on virustotal. [Here](https://www.virustotal.com/gui/file/c868cd6ae39dc3ebbc225c5f8dc86e3b01097aa4b0076eac7960256038e60b43) is what i got.(_You can also upload the pdf file directly_)

```bash
➜  sha256sum README.pdf
c868cd6ae39dc3ebbc225c5f8dc86e3b01097aa4b0076eac7960256038e60b43  README.pdf
```

![image](https://user-images.githubusercontent.com/58165365/147153373-d87badbe-eda5-4c96-8ff0-9184ce9ebaa4.png)

`False`

---

#### Does the file have the correct magic number? (2 points)

[GeeksforGeeks](https://www.geeksforgeeks.org/working-with-magic-numbers-in-linux/) does an awesome job explaining what magic numbers are. I'll try make this simple. Magic numbers are the first few bytes of a file that are unique to a particular file type. This number is in the first 512 bytes of the file. These unique bits are referred to as magic numbers, also sometimes referred to as a **file signature**. These bytes can be used by the system to “_differentiate between and recognize different files_” without a file extension.

Some files, however, do not have magic numbers, such as plain text files, but can be identified by checking the character set (ASCII in the case of text files).
This can be done by using the command:

```bash
➜  file -i README.pdf
README.pdf: application/pdf; charset=binary
```

Magic numbers/File signatures are typically not visible to the user but can be seen by using a hex editor or by using the `xxd` command as demonstrated below. We can view the hex of a file by typing the following command in a Linux terminal

```bash
➜  xxd README.pdf | head
00000000: 2550 4446 2d31 2e37 0d0a 25b5 b5b5 b50d  %PDF-1.7..%.....
00000010: 0a31 2030 206f 626a 0d0a 3c3c 2f54 7970  .1 0 obj..<</Typ
00000020: 652f 4361 7461 6c6f 672f 5061 6765 7320  e/Catalog/Pages
00000030: 3220 3020 522f 4c61 6e67 2865 6e2d 5553  2 0 R/Lang(en-US
00000040: 2920 2f53 7472 7563 7454 7265 6552 6f6f  ) /StructTreeRoo
00000050: 7420 3130 2030 2052 2f4d 6172 6b49 6e66  t 10 0 R/MarkInf
00000060: 6f3c 3c2f 4d61 726b 6564 2074 7275 653e  o<</Marked true>
00000070: 3e2f 4d65 7461 6461 7461 2032 3020 3020  >/Metadata 20 0
00000080: 522f 5669 6577 6572 5072 6566 6572 656e  R/ViewerPreferen
00000090: 6365 7320 3231 2030 2052 3e3e 0d0a 656e  ces 21 0 R>>..en
```

In terminal output above, we see that the first set of bytes of the file are `2550 4446 2d`. If we cross-reference this bytes with this [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures), We can see that indeed this is indeed a `.pdf` file

![image](https://user-images.githubusercontent.com/58165365/147153500-e61c59cd-48a3-40cd-aae1-c6e6ce41993d.png)

We can also use another tools like hexdump to get the magic numbers as shown in the screenshot below

![image](https://user-images.githubusercontent.com/58165365/147153179-60413471-2dc7-4eea-89ce-34ce49edd752.png)

Lets break down the flags used in the command above:

- -C - _canonical hex+ASCII display_
- -n - _interpret only length bytes of input_

`True`

---

#### What OS type can the file exploit? (Linux, MacOS, Windows, etc) (5 points)

Autorun runs on Windows, so this was preety straight forward.

`Windows`

---

#### A Windows executable is mentioned in the pdf file, what is it? (3 points)

There are several tools you can use to walk a malicious pdf. In this walkthrough, i will try go through some of my favourite. Lets start with pdf-parser.

```bash
➜  pdf-parser -a README.pdf
This program has not been tested with this version of Python (3.9.7)
Should you encounter problems, please use Python version 3.9.5
Comment: 8
XREF: 4
Trailer: 4
StartXref: 4
Indirect object: 24
  8: 4, 9, 18, 19, 21, 24, 26, 9
 /Action 2: 27, 28
 /Catalog 2: 1, 1
 /ExtGState 2: 7, 8
 /Filespec 1: 25
 /Font 1: 5
 /FontDescriptor 1: 6
 /Metadata 2: 20, 20
 /ObjStm 1: 17
 /Page 2: 3, 3
 /Pages 1: 2
 /XRef 1: 22
Search keywords:
 /JS 1: 27
 /JavaScript 1: 27
 /AA 1: 3
 /OpenAction 1: 1
 /Launch 1: 28
```

`/Launch 1: 28` seems interesting., so we can run the command above again but this time round with `-o` flag to select and display the specific indirect object.

```bash
➜  pdf-parser README.pdf -o 28
This program has not been tested with this version of Python (3.9.7)
Should you encounter problems, please use Python version 3.9.5
obj 28 0
 Type: /Action
 Referencing:

  <<
    /S /Launch
    /Type /Action
    /Win
      <<
        /F (cmd.exe)
        /D '(c:\\\\windows\\\\system32)'
        /P '(/Q /C %HOMEDRIVE%&cd %HOMEPATH%&(if exist "Desktop\\\\README.pdf" (cd "Desktop"))&(if exist "My Documents\\\\README.pdf" (cd "My Documents"))&(if exist "Documents\\\\README.pdf" (cd "Documents"))&(if exist "Escritorio\\\\README.pdf" (cd "Escritorio"))&(if exist "Mis Documentos\\\\README.pdf" (cd "Mis Documentos"))&(start README.pdf)\n\n\n\n\n\n\n\n\n\nTo view the encrypted content please tick the "Do not show this message again" box and press Open.)'
      >>
  >>
```

You can see we have `/F` , `/D` and `/P`. I might be wrong but here is what i made of this.

- /F is the file
- /D indicates the directory
- /P indicates the path

From this i came to a conclusion that the file is `cmd.exe`. Other tools and scripts you can use to analyse `.pdf's` include:

- [pdfid.py](https://didierstevens.com/files/software/pdfid_v0_2_8.zip)
- [pdftool.py](https://didierstevens.com/files/software/pdftool_V0_0_1.zip)
- [peepdf_0.3](https://eternal-todo.com/tools/peepdf-pdf-analysis-tool)

i.e:

```bash
➜  python2 pdfid.py README.pdf
PDFiD 0.2.8 README.pdf
 PDF Header: %PDF-1.7
 obj                   25
 endobj                25
 stream                 7
 endstream              7
 xref                   4
 trailer                4
 startxref              4
 /Page                  2
 /Encrypt               0
 /ObjStm                1
 /JS                    1
 /JavaScript            1
 /AA                    1
 /OpenAction            1
 /AcroForm              0
 /JBIG2Decode           0
 /RichMedia             0
 /Launch                1
 /EmbeddedFile          0
 /XFA                   0
 /URI                   0
 /Colors > 2^24         0
```

Where:

- `/Page` - _gives an indication of the number of pages in the PDF document._
- `/ObjStm` - _counts the number of object streams. An object stream is a stream object that can contain other objects, and can therefor be used to obfuscate objects (by using different filters)._
- `/JS` & `/JavaScript` - _indicate that the PDF document contains JavaScript._
- `/AA` & `/OpenAction` - _indicate an automatic action to be performed when the page/document is viewed. All malicious PDF documents with JavaScript seen in the wild most likely have an automatic action to launch the JavaScript without user interaction._
- `/Launch` - _counts launch actions._

`cmd.exe`

---

#### How many suspicious /OpenAction elements does the file have? (5 points)

From the previous tool, we can be able to conclude there is one `/OpenAction` element, butlets use a different tool to confirm the same.

```bash
➜  ./peepdf.py README.pdf
Warning: PyV8 is not installed!!
Warning: pylibemu is not installed!!

File: README.pdf
MD5: 140d0bf280fe5ba50aadb146b37d5395
SHA1: 7cd77a35f53e170a26b02d27b48386f83e90501b
Size: 136561 bytes
Version: 1.7
Binary: True
Linearized: False
Encrypted: False
Updates: 3
Objects: 31
Streams: 7
Comments: 0
Errors: 1

Version 0:
        Catalog: 1
        Info: 9
        Objects (22): [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22]
        Compressed objects (7): [10, 11, 12, 13, 14, 15, 16]
        Streams (5): [4, 17, 19, 20, 22]
                Xref streams (1): [22]
                Object streams (1): [17]
                Encoded (4): [4, 17, 19, 22]
        Suspicious elements:
                /Names: [13]


Version 1:
        Catalog: 1
        Info: 9
        Objects (0): []
        Streams (0): []

Version 2:
        Catalog: 1
        Info: 9
        Objects (7): [1, 3, 24, 25, 26, 27, 28]
        Streams (1): [26]
                Encoded (1): [26]
        Suspicious elements:
                /OpenAction: [1]
                /Names: [24, 1]
                /AA: [3]
                /JS: [27]
                /Launch: [28]
                /JavaScript: [27]


Version 3:
        Catalog: 1
        Info: 9
        Objects (2): [9, 20]
        Streams (1): [20]
                Encoded (0): []
```

`1`

Thanks for reading my write-up.😃 I would really appreciate it if i got your feedback on the same, like was it informative, too long, my grammar and use of terminologies was wrong?

Feel free to reach out to me on Twitter [**@oste_ke**](https://twitter.com/oste_ke)
