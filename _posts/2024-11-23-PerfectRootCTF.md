---
title: "P3rf3ctr00t CTF Challenge Creation"
date: 2024-11-23 01:09:33 +0300
author: [oste]
description: This blog post explains the thought process in creating the P3rf3ctr00t CTF as well as how different challenges were meant to be solved.
image: /assets/img/Posts/P3rf3ctr00t CTF.png
categories: [CTF-TIME]
tags:
  [dive, docker, Dockerfile, docker layers, corrupt, GIF, File Signatures, hex, Zines, VHDX, FTK Imager, diskmgmt.msc, VHD, zip2john, john, Excel, strings, IRIS-H Digital Forensics, dcode.fr, CyberChef]
---

I had the privilege to create EASY CTF challenges for the [P3rf3ctr00t CTF](https://perfectroot.wiki/)  held on the weekend of 15th - 17th November 2024. This was fun and successful being the first timing hosting a local CTF that attracted players from all over the world. I created 8 easy challenges summarized below:

| Challenge              | Solves/142 |
| ---------------------- | ---------- |
| SheetsNLayers - Flag 1 | 38         |
| See Ya                 | 33         |
| Dive                   | 25         |
| SheetsNLayers - Flag 2 | 21         |
| SheetsNLayers - Flag 3 | 21         |
| SheetsNLayers - Flag 4 | 20         |
| tGIF                   | 16         |
| YAZ (Yet Another Zine) | 5          |


In this Post, I get to share with you my thought process in creating some of the challenges and perhaps how they were meant to be solved. Feel free to leave a comment incase you used a different technique to solve any of the challenges. I'm sure some challs had some un-intended solves 😅


# Misc
## DIVE

### Challenge Overview: 

Docker is really fun. DIVE deeper and you'll probably get the flag 😎

### Challenge Creation:

```dockerfile
# Base Image
FROM alpine:latest

# Install basic tools
RUN apk update && apk add curl vim git
RUN echo "This is just a dummy layer" > /dummy_layer.txt

# Copy the flag file into the container
COPY flag.txt /root/flag.txt

# Final layer to make it less obvious
RUN apk del vim curl git && rm -rf /var/cache/apk/*

# Remove the flag file
RUN rm /root/flag.txt
```
{: .nolineno }

- Base Image: _Alpine Linux (alpine:latest)_
- Flag Placement: _The flag is initially placed in a file named flag.txt located in the `/root` directory. It is then removed in a subsequent layer to simulate the need for forensic analysis._


Challenge Artifact: The Docker image is saved as a tarball named dive.tar.


```bash
# Build image
sudo docker build -t dive .

#Confirm image has been build:
sudo docker images

# Export docker image to a tarball
sudo docker save -o dive.tar dive:latest
```
{: .nolineno }


### Solution

In order to interact with the file: 

Import the docker image by running:

```bash
➜  docker load -i dive.tar
```
{: .nolineno }


Next we can analyze the image using a tool like [Dive](https://github.com/wagoodman/dive)

```bash
➜  sudo dive dive:latest
```
{: .nolineno }


> I'm sure most people went straight into uncompressing the file and retrieving the flag, but perhaps we can try and understand a little bit about docker and how one can analyze/do forensics on an image.
{: .prompt-tip }

But before we continue, lets first talk about docker layers. Docker images are built in layers, with each layer representing a single instruction in the `Dockerfile` (e.g., `RUN`, `COPY`, `ADD`). These layers are:

- **Immutable:** *Once created, a layer cannot be changed. If a `Dockerfile` is modified, only the changed layers and the ones that depend on them are rebuilt.*
- **Shared:** *Layers can be reused across multiple images, reducing storage requirements and speeding up builds.*
- **Union Filesystem:** *Layers are stacked, and the topmost layer represents the current state of the image.*

Feel free to watch the video linked below for more information about docker images/layers.

{% include embed/youtube.html id='CvSFl4jI8Hg' %}


When analyzing Docker layers, especially from a compressed tarball of the image, each layer corresponds to a filesystem snapshot and metadata detailing how it was created. 

Lets see how we can analyze the docker image.

Run [Dive](https://github.com/wagoodman/dive) as follows:

```bash
➜  sudo dive dive:latest
```
{: .nolineno }

You will then get an terminal interactive. In the top most layer, you will get a list of layers and respective commands executed in each layer. To navigate/inspect different layers, use arrow keys

In this case, you will notice a flag is being copied and then removed in the `/root/flag.txt`. 

> Take note of the Digest value (SHA-256 hash) highlighted.
{: .prompt-info }

![image](https://gist.github.com/user-attachments/assets/b32a4183-3f10-4298-859f-5b9efa230a2b)

To further interact with the layer, press `Tab` followed by `Ctrl + U` to confirm the flag was actually copied to `/root/flag.txt` 

![image](https://gist.github.com/user-attachments/assets/a89be22a-6e3c-497a-9aba-647278ed553e)

If you press `Tab` again and navigate to the last layer, you will see the flag has deleted (In red)

![image](https://gist.github.com/user-attachments/assets/7b7a191b-f99d-4ae5-92d3-3ce0534bcb41)

However, `dive` cannot directly extract files from the layers. For manual extraction of files, you need to decompress and inspect the specific layers individually.

Simply decompress as follows:

```bash
➜  sudo tar -xvf dive.tar

blobs/
blobs/sha256/
blobs/sha256/19dab3bdc1fcfe59f45ba4071f6b39bdac309e2a587c118ba4e56f3b1a69e44d
blobs/sha256/26b69c4d39add31e9fcbb2617c849d8b27d0a9aeb871a15b045130c0f7dd573a
blobs/sha256/32595609fc55156dd4899f32e2e028da7bea813bb8b47647960069b4ea3b2ef7
blobs/sha256/3a3688710208498d9f2acfd70943158de61368020992f8f9f240ab34bc5dfdef
blobs/sha256/3d850ad76eff62bd6752b61c8b9013704b466b07f8cb65ccd3b016baf065e2d9
blobs/sha256/63dc39fd7e9b99b37201364b25a176b4c419ad819aa4da6fdfd76bdce3eb386a
blobs/sha256/6c091594b15387b41416296ad385a363c788af1cf0d55d234750bb3848c8e758
blobs/sha256/7186a421d902123801bd7ccb204f9e9631175ccaf11af0de7d01f4e6b5d2ed8e
blobs/sha256/75654b8eeebd3beae97271a102f57cdeb794cc91e442648544963a7e951e9558
blobs/sha256/7cbb6c3a98354fe830deee81003184b1299dd36f428f9ebf67ca786724929962
blobs/sha256/8f2795eff7243c570c41a0286e8ede760453f6624245669cf3f392480a2d23bf
blobs/sha256/e7ee579f9e5e6fc497a781027dfe843141e644be474043ebca05859c271583b2
blobs/sha256/e846a361a9a5ab27aff3e73ed6388d661645d561fd2d9af1f10bd2ae1e2a98a0
blobs/sha256/e9eec7ad63db20407abcb5f63fa90e2ee06d8a94787b001f23614f729ce82957
index.json
manifest.json
oci-layout
repositories
```
{: .nolineno }

If you care to understand each of the extracted files, I'll just do a high level overview of each.

- **repositories** - *Maps repository names to their respective digests or tags.*

![image](https://gist.github.com/user-attachments/assets/f94fc66f-dbdd-4d2c-9c8d-9dffc0b5c1ea)

- **[oci-layout](https://github.com/opencontainers/image-spec)** - *Contains the version of the layout.*

![image](https://gist.github.com/user-attachments/assets/b32909d9-2fba-491d-851b-505c5d73202b)


- [manifest.json](https://docs.docker.com/reference/cli/docker/manifest/) - *lists all the layers and configuration details of the Docker image in the order they are applied. It acts as a roadmap, linking layer blobs and metadata required to reconstruct the image.*

![image](https://gist.github.com/user-attachments/assets/8c288fe5-5361-4c7e-bbc5-46a25653291a)

- **index.json** - *specifies the schema version and the manifest type (e.g., `application/vnd.oci.image.manifest.v1+json`) for the image. It serves as a catalog linking image metadata to its respective layers and configurations for OCI-compliant tools.*

![image](https://gist.github.com/user-attachments/assets/3a538952-d296-4535-9742-24babcf0bc31)


Finally, we have the `blobs/sha256` directory , which contains all the image layers and metadata stored as content-addressable objects, identified by their SHA-256 hashes. Each file in this directory represents either a filesystem layer (added or modified files) or configuration data for the image. These blobs are referenced in `manifest.json` to reconstruct the image's filesystem and settings. 

![image](https://gist.github.com/user-attachments/assets/c1521974-b6b4-479f-8d4c-68e314267349)

It is also worth noting that this layers are usually a `.tar.gz` archives and need to be decompressed further. 

While using Dive, I'd mentioned you need to note down the Digest value (SHA-256 hash) of the layer containing the flag.txt, In this case: `8f2795eff7243c570c41a0286e8ede760453f6624245669cf3f392480a2d23bf`

Extract the specific layer containing the flag, you get:

![image](https://gist.github.com/user-attachments/assets/f700c552-0694-4578-958d-ccb5952f7ff2)


`r00t{41W4Y5_4N41YZ3_D0CK3r_14Y3r5}`

### Complimentary Resources

- https://www.docker.com/resources/container-images-interactive-deep-dive-dockercon-2023/
- https://docs.docker.com/get-started/docker-concepts/building-images/


## tGIF

### Challenge Overview: 

Docker is really fun. DIVE deeper and you'll probably get the flag 😎

### Solution

This was another relatively easy and common challenge that involved fixing a corrupt GIF file (as the name suggests `tGIF`) and resizing its size in order to reveal the flag. Lets start looking into it.

First, the file doesnt have a file extension. Running file on it, you will notice its a bunch of data. Perhaps we can check it as well with a hex editor.

```bash
➜  file tGIF
tGIF: data
➜  xxd tGIF | head
00000000: 4704 4638 3961 9001 c800 f700 0000 0000  G.F89a..........
00000010: 251e 253a 3134 3d43 6246 3e48 4835 324e  %.%:14=CbF>HH52N
00000020: 4d65 4f3a 3456 4f61 5745 465a 3f38 5e46  MeO:4VOaWEFZ?8^F
00000030: 3a5e 515c 5f47 425f 6272 5f65 8660 4c4c  :^Q\_GB_br_e.`LL
00000040: 605b 6264 6671 674d 4568 6773 696d 856a  `[bdfqgMEhgsim.j
00000050: 483b 6a70 7a6b 5951 6c4f 3d6c 504c 6f50  H;jpzkYQlO=lPLoP
00000060: 456f 5150 6f5f 626f 6f85 7052 5671 737a  EoQPo_boo.pRVqsz
00000070: 7277 8173 5a51 7367 7273 7990 766f 787a  rw.sZQsgrsy.voxz
00000080: 5c53 7a60 5f7b 7782 7c7e 907c 8089 7c82  \Sz`_{w.|~.|..|.
00000090: a183 5546 835f 5783 615f 838f ac84 685a  ..UF._W.a_....hZ
```
{: .nolineno }

Here you will notice it is a GIF file. `(G.F89a)`. This looks odd as the oiginal GIF header would look like `GIF89a`. This means that the magic bytes for `I` have been modified.

From xxd's output above, we have: `47 04 46 38 39 61`. If you head over to [Wikipeda -File Signatures](https://en.wikipedia.org/wiki/List_of_file_signatures), we get the GIFs correct magic bytes as `47 49 46 38 39 61` . Notice `49` was swapped with `04`. That's what you need to fix.

![image](https://gist.github.com/user-attachments/assets/8c0c1524-dc46-407f-ac40-257878452497)

Notice `49` was swapped with `04`. That's what you need to fix. Using your favourite hex editor, change that part and save the file as `tGIF.gif ` as shown:

![image](https://gist.github.com/user-attachments/assets/a77c38fc-f20d-4b88-9162-e5af23123e3f)

Opening the file, it renders as follows:

![image](https://gist.github.com/user-attachments/assets/42510885-32df-435b-81c6-55d269abc82f)

This is where the second part of this challenge continues. It involves fixing the GIF size in order to view the specific part containing a flag. Part of the challenge description, there was a second hint mentioning "*dimensions are just off*"

Using exiftool, you can check the `Image Size` property as shown. 

```bash
➜  exiftool tGIF.gif
ExifTool Version Number         : 13.00
File Name                       : tGIF.gif
Directory                       : .
File Size                       : 3.0 MB
File Modification Date/Time     : 2024:11:23 18:23:59+03:00
File Access Date/Time           : 2024:11:23 18:37:05+03:00
File Inode Change Date/Time     : 2024:11:23 18:37:05+03:00
File Permissions                : -rw-------
File Type                       : GIF
File Type Extension             : gif
MIME Type                       : image/gif
GIF Version                     : 89a
Image Width                     : 400
Image Height                    : 200
Has Color Map                   : Yes
Color Resolution Depth          : 8
Bits Per Pixel                  : 8
Background Color                : 0
Animation Iterations            : Infinite
Comment                         : GIF edited with https://ezgif.com/add-text
Frame Count                     : 21
Duration                        : 2.73 s
Image Size                      : 400x200
Megapixels                      : 0.080
```
{: .nolineno }


Here, we see the size as `400x200`. Perhaps not the standard size for a GIF? Doing a quick search on google, you will learn the standards size is `640 width ×480 height`. There are different ways one could go about fixing this:


- [Ezgif image resizer](https://ezgif.com/resize/)

![image](https://gist.github.com/user-attachments/assets/ddbb9bda-1098-43f6-93e6-fbfae44e11f3)

Or

- [RedKetchup GIF Resizer](https://redketchup.io/gif-resizer)

This site auto resizes and renders the complete GIF for you

![image](https://gist.github.com/user-attachments/assets/29e523d1-4887-4418-b101-8e239496bc6e)

Others Include:

- [iloveimg](https://www.iloveimg.com/resize-image/resize-gif)
- [ImageGIFMemesFonts](https://www.gifgit.com/gif/resize)
- [kapwing](https://www.kapwing.com/studio/editor)


![render](/assets/img/Posts/tGIF-incomplete.gif)


`r00t{d38252762d3d4fd229faae637fd13f4e}`



## See Ya

### Challenge Overview: 

You a fan of Zines? Well here's my fav from the 90's 🙂 , The "**Hacker's Manifesto**". Not every one will SEE it, but oh well, good luck.

Of course if you're SEEing this, here's the original one: https://phrack.org/issues/7/3.html

### Solution

The challenge presents the [Hacker's Manifesto](https://phrack.org/issues/7/3.html) in Braille, with the keyword `SEE` hinting at translation. 

![image](https://gist.github.com/user-attachments/assets/d4580fec-823b-4ab0-9425-4697a54c40c3)

Players were basically expected to decode the Braille using tools like CyberChef's [`From Braille`](https://gchq.github.io/CyberChef/#recipe=From_Braille()&input=4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qC/4qC/4qCP4qCT4qCX4qCB4qCJ4qCF4qCA4qCK4qCd4qCJ4qCo4qC/4qC/CgrioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioKfioJXioIfioKXioI3ioJHioIDioJXioJ3ioJHioKDioIDioIrioI7ioI7ioKXioJHioIDioLbioKDioIDioI/ioJPioIrioIfioJHioIDioJLioIDioJXioIvioIDioILioLQKCuKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgv%2BKgpOKgvwrioJ7ioJPioJHioIDioIvioJXioIfioIfioJXioLrioIrioJ3ioJvioIDioLrioIHioI7ioIDioLrioJfioIrioJ7ioJ7ioJHioJ3ioIDioI7ioJPioJXioJfioJ7ioIfioL3ioIDioIHioIvioJ7ioJHioJfioIDioI3ioL3ioIDioIHioJfioJfioJHioI7ioJ7ioKjioKjioKgKCuKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKgs%2BKgjOKgs%2BKgnuKgk%2BKgkeKggOKgieKgleKgneKgjuKgieKgiuKgkeKgneKgieKgkeKggOKgleKgi%2BKggOKggeKggOKgk%2BKggeKgieKgheKgkeKgl%2BKgjOKgs%2BKgjAoK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCD4qC9CgrioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioKzioKzioKzioJ7ioJPioJHioIDioI3ioJHioJ3ioJ7ioJXioJfioKzioKzioKwKCuKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKguuKgl%2BKgiuKgnuKgnuKgkeKgneKggOKgleKgneKggOKgmuKggeKgneKgpeKggeKgl%2BKgveKggOKgpuKgoOKggOKgguKglOKgpuKglgrioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL/ioKTioL8KCuKggOKggOKggOKggOKggOKggOKggOKggOKggeKgneKgleKgnuKgk%2BKgkeKgl%2BKggOKgleKgneKgkeKggOKgm%2BKgleKgnuKggOKgieKggeKgpeKgm%2BKgk%2BKgnuKggOKgnuKgleKgmeKggeKgveKgoOKggOKgiuKgnuKghOKgjuKggOKggeKgh%2BKgh%2BKggOKgleKgp%2BKgkeKgl%2BKggOKgnuKgk%2BKgkeKggOKgj%2BKggeKgj%2BKgkeKgl%2BKgjuKgqOKggOKggOKgkOKgnuKgkeKgkeKgneKggeKgm%2BKgkeKglwrioIHioJfioJfioJHioI7ioJ7ioJHioJnioIDioIrioJ3ioIDioInioJXioI3ioI/ioKXioJ7ioJHioJfioIDioInioJfioIrioI3ioJHioIDioI7ioInioIHioJ3ioJnioIHioIfioJDioKDioIDioJDioJPioIHioInioIXioJHioJfioIDioIHioJfioJfioJHioI7ioJ7ioJHioJnioIDioIHioIvioJ7ioJHioJfioIDioIPioIHioJ3ioIXioIDioJ7ioIHioI3ioI/ioJHioJfioIrioJ3ioJvioJDioKjioKjioKgK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCZ4qCB4qCN4qCd4qCA4qCF4qCK4qCZ4qCO4qCo4qCA4qCA4qCe4qCT4qCR4qC94qCE4qCX4qCR4qCA4qCB4qCH4qCH4qCA4qCB4qCH4qCK4qCF4qCR4qCoCgrioIDioIDioIDioIDioIDioIDioIDioIDioIPioKXioJ7ioIDioJnioIrioJnioIDioL3ioJXioKXioKDioIDioIrioJ3ioIDioL3ioJXioKXioJfioIDioJ7ioJPioJfioJHioJHioKTioI/ioIrioJHioInioJHioIDioI/ioI7ioL3ioInioJPioJXioIfioJXioJvioL3ioIDioIHioJ3ioJnioIDioILioJTioKLioLTioITioI7ioIDioJ7ioJHioInioJPioJ3ioJXioIPioJfioIHioIrioJ3ioKAK4qCR4qCn4qCR4qCX4qCA4qCe4qCB4qCF4qCR4qCA4qCB4qCA4qCH4qCV4qCV4qCF4qCA4qCD4qCR4qCT4qCK4qCd4qCZ4qCA4qCe4qCT4qCR4qCA4qCR4qC94qCR4qCO4qCA4qCV4qCL4qCA4qCe4qCT4qCR4qCA4qCT4qCB4qCJ4qCF4qCR4qCX4qC54qCA4qCA4qCZ4qCK4qCZ4qCA4qC94qCV4qCl4qCA4qCR4qCn4qCR4qCX4qCA4qC64qCV4qCd4qCZ4qCR4qCX4qCA4qC64qCT4qCB4qCeCuKgjeKggeKgmeKgkeKggOKgk%2BKgiuKgjeKggOKgnuKgiuKgieKgheKgoOKggOKguuKgk%2BKggeKgnuKggOKgi%2BKgleKgl%2BKgieKgkeKgjuKggOKgjuKgk%2BKggeKgj%2BKgkeKgmeKggOKgk%2BKgiuKgjeKgoOKggOKguuKgk%2BKggeKgnuKggOKgjeKggeKgveKggOKgk%2BKggeKgp%2BKgkeKggOKgjeKgleKgh%2BKgmeKgkeKgmeKggOKgk%2BKgiuKgjeKguQrioIDioIDioIDioIDioIDioIDioIDioIDioIrioIDioIHioI3ioIDioIHioIDioJPioIHioInioIXioJHioJfioKDioIDioJHioJ3ioJ7ioJHioJfioIDioI3ioL3ioIDioLrioJXioJfioIfioJnioKjioKjioKgK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCN4qCK4qCd4qCR4qCA4qCK4qCO4qCA4qCB4qCA4qC64qCV4qCX4qCH4qCZ4qCA4qCe4qCT4qCB4qCe4qCA4qCD4qCR4qCb4qCK4qCd4qCO4qCA4qC64qCK4qCe4qCT4qCA4qCO4qCJ4qCT4qCV4qCV4qCH4qCo4qCo4qCo4qCA4qCK4qCE4qCN4qCA4qCO4qCN4qCB4qCX4qCe4qCR4qCX4qCA4qCe4qCT4qCB4qCd4qCA4qCN4qCV4qCO4qCe4qCA4qCV4qCLCuKgnuKgk%2BKgkeKggOKgleKgnuKgk%2BKgkeKgl%2BKggOKgheKgiuKgmeKgjuKgoOKggOKgnuKgk%2BKgiuKgjuKggOKgieKgl%2BKggeKgj%2BKggOKgnuKgk%2BKgkeKgveKggOKgnuKgkeKggeKgieKgk%2BKggOKgpeKgjuKggOKgg%2BKgleKgl%2BKgkeKgjuKggOKgjeKgkeKgqOKgqOKgqArioIDioIDioIDioIDioIDioIDioIDioIDioJnioIHioI3ioJ3ioIDioKXioJ3ioJnioJHioJfioIHioInioJPioIrioJHioKfioJHioJfioKjioIDioIDioJ7ioJPioJHioL3ioITioJfioJHioIDioIHioIfioIfioIDioIHioIfioIrioIXioJHioKgKCuKggOKggOKggOKggOKggOKggOKggOKggOKgiuKghOKgjeKggOKgiuKgneKggOKgmuKgpeKgneKgiuKgleKgl%2BKggOKgk%2BKgiuKgm%2BKgk%2BKggOKgleKgl%2BKggOKgk%2BKgiuKgm%2BKgk%2BKggOKgjuKgieKgk%2BKgleKgleKgh%2BKgqOKggOKggOKgiuKghOKgp%2BKgkeKggOKgh%2BKgiuKgjuKgnuKgkeKgneKgkeKgmeKggOKgnuKgleKggOKgnuKgkeKggeKgieKgk%2BKgkeKgl%2BKgjuKggOKgkeKgreKgj%2BKgh%2BKggeKgiuKgnQrioIvioJXioJfioIDioJ7ioJPioJHioIDioIvioIrioIvioJ7ioJHioJHioJ3ioJ7ioJPioIDioJ7ioIrioI3ioJHioIDioJPioJXioLrioIDioJ7ioJXioIDioJfioJHioJnioKXioInioJHioIDioIHioIDioIvioJfioIHioInioJ7ioIrioJXioJ3ioKjioIDioIDioIrioIDioKXioJ3ioJnioJHioJfioI7ioJ7ioIHioJ3ioJnioIDioIrioJ7ioKjioIDioIDioJDioJ3ioJXioKDioIDioI3ioI7ioKgK4qCO4qCN4qCK4qCe4qCT4qCg4qCA4qCK4qCA4qCZ4qCK4qCZ4qCd4qCE4qCe4qCA4qCO4qCT4qCV4qC64qCA4qCN4qC94qCA4qC64qCV4qCX4qCF4qCo4qCA4qCA4qCK4qCA4qCZ4qCK4qCZ4qCA4qCK4qCe4qCA4qCK4qCd4qCA4qCN4qC94qCA4qCT4qCR4qCB4qCZ4qCo4qCo4qCo4qCQCuKggOKggOKggOKggOKggOKggOKggOKggOKgmeKggeKgjeKgneKggOKgheKgiuKgmeKgqOKggOKggOKgj%2BKgl%2BKgleKgg%2BKggeKgg%2BKgh%2BKgveKggOKgieKgleKgj%2BKgiuKgkeKgmeKggOKgiuKgnuKgqOKggOKggOKgnuKgk%2BKgkeKgveKghOKgl%2BKgkeKggOKggeKgh%2BKgh%2BKggOKggeKgh%2BKgiuKgheKgkeKgqAoK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCK4qCA4qCN4qCB4qCZ4qCR4qCA4qCB4qCA4qCZ4qCK4qCO4qCJ4qCV4qCn4qCR4qCX4qC94qCA4qCe4qCV4qCZ4qCB4qC94qCo4qCA4qCA4qCK4qCA4qCL4qCV4qCl4qCd4qCZ4qCA4qCB4qCA4qCJ4qCV4qCN4qCP4qCl4qCe4qCR4qCX4qCo4qCA4qCA4qC64qCB4qCK4qCe4qCA4qCB4qCA4qCO4qCR4qCJ4qCV4qCd4qCZ4qCg4qCA4qCe4qCT4qCK4qCO4qCA4qCK4qCOCuKgieKgleKgleKgh%2BKgqOKggOKggOKgiuKgnuKggOKgmeKgleKgkeKgjuKggOKguuKgk%2BKggeKgnuKggOKgiuKggOKguuKggeKgneKgnuKggOKgiuKgnuKggOKgnuKgleKgqOKggOKggOKgiuKgi%2BKggOKgiuKgnuKggOKgjeKggeKgheKgkeKgjuKggOKggeKggOKgjeKgiuKgjuKgnuKggeKgheKgkeKgoOKggOKgiuKgnuKghOKgjuKggOKgg%2BKgkeKgieKggeKgpeKgjuKgkeKggOKgigrioI7ioInioJfioJHioLrioJHioJnioIDioIrioJ7ioIDioKXioI/ioKjioIDioIDioJ3ioJXioJ7ioIDioIPioJHioInioIHioKXioI7ioJHioIDioIrioJ7ioIDioJnioJXioJHioI7ioJ3ioITioJ7ioIDioIfioIrioIXioJHioIDioI3ioJHioKjioKjioKgK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCV4qCX4qCA4qCL4qCR4qCR4qCH4qCO4qCA4qCe4qCT4qCX4qCR4qCB4qCe4qCR4qCd4qCR4qCZ4qCA4qCD4qC94qCA4qCN4qCR4qCo4qCo4qCoCuKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKggOKgleKgl%2BKggOKgnuKgk%2BKgiuKgneKgheKgjuKggOKgiuKghOKgjeKggOKggeKggOKgjuKgjeKggeKgl%2BKgnuKggOKggeKgjuKgjuKgqOKgqOKgqArioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioIDioJXioJfioIDioJnioJXioJHioI7ioJ3ioITioJ7ioIDioIfioIrioIXioJHioIDioJ7ioJHioIHioInioJPioIrioJ3ioJvioIDioIHioJ3ioJnioIDioI7ioJPioJXioKXioIfioJnioJ3ioITioJ7ioIDioIPioJHioIDioJPioJHioJfioJHioKjioKjioKgK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCZ4qCB4qCN4qCd4qCA4qCF4qCK4qCZ4qCo4qCA4qCA4qCB4qCH4qCH4qCA4qCT4qCR4qCA4qCZ4qCV4qCR4qCO4qCA4qCK4qCO4qCA4qCP4qCH4qCB4qC94qCA4qCb4qCB4qCN4qCR4qCO4qCo4qCA4qCA4qCe4qCT4qCR4qC94qCE4qCX4qCR4qCA4qCB4qCH4qCH4qCA4qCB4qCH4qCK4qCF4qCR4qCoCgrioIDioIDioIDioIDioIDioIDioIDioIDioIHioJ3ioJnioIDioJ7ioJPioJHioJ3ioIDioIrioJ7ioIDioJPioIHioI/ioI/ioJHioJ3ioJHioJnioKjioKjioKjioIDioIHioIDioJnioJXioJXioJfioIDioJXioI/ioJHioJ3ioJHioJnioIDioJ7ioJXioIDioIHioIDioLrioJXioJfioIfioJnioKjioKjioKjioIDioJfioKXioI7ioJPioIrioJ3ioJvioIDioJ7ioJPioJfioJXioKXioJvioJMK4qCe4qCT4qCR4qCA4qCP4qCT4qCV4qCd4qCR4qCA4qCH4qCK4qCd4qCR4qCA4qCH4qCK4qCF4qCR4qCA4qCT4qCR4qCX4qCV4qCK4qCd4qCA4qCe4qCT4qCX4qCV4qCl4qCb4qCT4qCA4qCB4qCd4qCA4qCB4qCZ4qCZ4qCK4qCJ4qCe4qCE4qCO4qCA4qCn4qCR4qCK4qCd4qCO4qCg4qCA4qCB4qCd4qCA4qCR4qCH4qCR4qCJ4qCe4qCX4qCV4qCd4qCK4qCJ4qCA4qCP4qCl4qCH4qCO4qCR4qCA4qCK4qCOCuKgjuKgkeKgneKgnuKggOKgleKgpeKgnuKgoOKggOKggeKggOKgl%2BKgkeKgi%2BKgpeKgm%2BKgkeKggOKgi%2BKgl%2BKgleKgjeKggOKgnuKgk%2BKgkeKggOKgmeKggeKgveKgpOKgnuKgleKgpOKgmeKggeKgveKggOKgiuKgneKgieKgleKgjeKgj%2BKgkeKgnuKgkeKgneKgieKgiuKgkeKgjuKggOKgiuKgjuKggOKgjuKgleKgpeKgm%2BKgk%2BKgnuKgqOKgqOKgqOKggOKggeKggOKgg%2BKgleKggeKgl%2BKgmeKggOKgiuKgjgrioIvioJXioKXioJ3ioJnioKgK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCQ4qCe4qCT4qCK4qCO4qCA4qCK4qCO4qCA4qCK4qCe4qCo4qCo4qCo4qCA4qCe4qCT4qCK4qCO4qCA4qCK4qCO4qCA4qC64qCT4qCR4qCX4qCR4qCA4qCK4qCA4qCD4qCR4qCH4qCV4qCd4qCb4qCo4qCo4qCo4qCQCuKggOKggOKggOKggOKggOKggOKggOKggOKgiuKggOKgheKgneKgleKguuKggOKgkeKgp%2BKgkeKgl%2BKgveKgleKgneKgkeKggOKgk%2BKgkeKgl%2BKgkeKgqOKgqOKgqOKggOKgkeKgp%2BKgkeKgneKggOKgiuKgi%2BKggOKgiuKghOKgp%2BKgkeKggOKgneKgkeKgp%2BKgkeKgl%2BKggOKgjeKgkeKgnuKggOKgnuKgk%2BKgkeKgjeKgoOKggOKgneKgkeKgp%2BKgkeKgl%2BKggOKgnuKggeKgh%2BKgheKgkeKgmeKggOKgnuKglQrioJ7ioJPioJHioI3ioKDioIDioI3ioIHioL3ioIDioJ3ioJHioKfioJHioJfioIDioJPioJHioIHioJfioIDioIvioJfioJXioI3ioIDioJ7ioJPioJHioI3ioIDioIHioJvioIHioIrioJ3ioKjioKjioKjioIDioIrioIDioIXioJ3ioJXioLrioIDioL3ioJXioKXioIDioIHioIfioIfioKjioKjioKgK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCZ4qCB4qCN4qCd4qCA4qCF4qCK4qCZ4qCo4qCA4qCA4qCe4qC94qCK4qCd4qCb4qCA4qCl4qCP4qCA4qCe4qCT4qCR4qCA4qCP4qCT4qCV4qCd4qCR4qCA4qCH4qCK4qCd4qCR4qCA4qCB4qCb4qCB4qCK4qCd4qCo4qCA4qCA4qCe4qCT4qCR4qC94qCE4qCX4qCR4qCA4qCB4qCH4qCH4qCA4qCB4qCH4qCK4qCF4qCR4qCo4qCo4qCoCgrioIDioIDioIDioIDioIDioIDioIDioIDioL3ioJXioKXioIDioIPioJHioJ7ioIDioL3ioJXioKXioJfioIDioIHioI7ioI7ioIDioLrioJHioITioJfioJHioIDioIHioIfioIfioIDioIHioIfioIrioIXioJHioKjioKjioKjioIDioLrioJHioITioKfioJHioIDioIPioJHioJHioJ3ioIDioI7ioI/ioJXioJXioJ3ioKTioIvioJHioJnioIDioIPioIHioIPioL3ioIDioIvioJXioJXioJnioIDioIHioJ4K4qCO4qCJ4qCT4qCV4qCV4qCH4qCA4qC64qCT4qCR4qCd4qCA4qC64qCR4qCA4qCT4qCl4qCd4qCb4qCR4qCX4qCR4qCZ4qCA4qCL4qCV4qCX4qCA4qCO4qCe4qCR4qCB4qCF4qCo4qCo4qCo4qCA4qCe4qCT4qCR4qCA4qCD4qCK4qCe4qCO4qCA4qCV4qCL4qCA4qCN4qCR4qCB4qCe4qCA4qCe4qCT4qCB4qCe4qCA4qC94qCV4qCl4qCA4qCZ4qCK4qCZ4qCA4qCH4qCR4qCe4qCA4qCO4qCH4qCK4qCPCuKgnuKgk%2BKgl%2BKgleKgpeKgm%2BKgk%2BKggOKguuKgkeKgl%2BKgkeKggOKgj%2BKgl%2BKgkeKgpOKgieKgk%2BKgkeKguuKgkeKgmeKggOKggeKgneKgmeKggOKgnuKggeKgjuKgnuKgkeKgh%2BKgkeKgjuKgjuKgqOKggOKggOKguuKgkeKghOKgp%2BKgkeKggOKgg%2BKgkeKgkeKgneKggOKgmeKgleKgjeKgiuKgneKggeKgnuKgkeKgmeKggOKgg%2BKgveKggOKgjuKggeKgmeKgiuKgjuKgnuKgjuKgoOKggOKgleKglwrioIrioJvioJ3ioJXioJfioJHioJnioIDioIPioL3ioIDioJ7ioJPioJHioIDioIHioI/ioIHioJ7ioJPioJHioJ7ioIrioInioKjioIDioIDioJ7ioJPioJHioIDioIvioJHioLrioIDioJ7ioJPioIHioJ7ioIDioJPioIHioJnioIDioI7ioJXioI3ioJHioJ7ioJPioIrioJ3ioJvioIDioJ7ioJXioIDioJ7ioJHioIHioInioJPioIDioIvioJXioKXioJ3ioJnioIDioKXioI7ioIDioLrioIrioIfioIfioKQK4qCK4qCd4qCb4qCA4qCP4qCl4qCP4qCK4qCH4qCO4qCg4qCA4qCD4qCl4qCe4qCA4qCe4qCT4qCV4qCO4qCR4qCA4qCL4qCR4qC64qCA4qCB4qCX4qCR4qCA4qCH4qCK4qCF4qCR4qCA4qCZ4qCX4qCV4qCP4qCO4qCA4qCV4qCL4qCA4qC64qCB4qCe4qCR4qCX4qCA4qCK4qCd4qCA4qCe4qCT4qCR4qCA4qCZ4qCR4qCO4qCR4qCX4qCe4qCoCgrioIDioIDioIDioIDioIDioIDioIDioIDioJ7ioJPioIrioI7ioIDioIrioI7ioIDioJXioKXioJfioIDioLrioJXioJfioIfioJnioIDioJ3ioJXioLrioKjioKjioKjioIDioJ7ioJPioJHioIDioLrioJXioJfioIfioJnioIDioJXioIvioIDioJ7ioJPioJHioIDioJHioIfioJHioInioJ7ioJfioJXioJ3ioIDioIHioJ3ioJnioIDioJ7ioJPioJHioIDioI7ioLrioIrioJ7ioInioJPioKDioIDioJ7ioJPioJEK4qCD4qCR4qCB4qCl4qCe4qC94qCA4qCV4qCL4qCA4qCe4qCT4qCR4qCA4qCD4qCB4qCl4qCZ4qCo4qCA4qCA4qC94qCV4qCl4qCX4qCA4qCL4qCH4qCB4qCb4qCA4qCK4qCO4qCA4qCX4qC04qC04qCee%2BKgkeKgmeKggeKgkeKglOKgluKghuKgguKghuKgouKgpuKgguKgsuKgmeKggeKgkeKgouKgsuKgi%2BKggeKgluKgkeKgsuKgg%2BKgluKghuKgpuKgi%2BKgmeKgluKgkeKgg33ioIDioKjioIDioLrioJHioIDioI3ioIHioIXioJHioIDioKXioI7ioJHioIDioJXioIvioIDioIHioIDioI7ioJHioJfioKfioIrioInioJHioIDioIHioIfioJfioJHioIHioJnioL3ioIDioJHioK3ioIrioI7ioJ7ioIrioJ3ioJvioIDioLrioIrioJ7ioJPioJXioKXioJ7ioIDioI/ioIHioL3ioIrioJ3ioJsK4qCL4qCV4qCX4qCA4qC64qCT4qCB4qCe4qCA4qCJ4qCV4qCl4qCH4qCZ4qCA4qCD4qCR4qCA4qCZ4qCK4qCX4qCe4qCk4qCJ4qCT4qCR4qCB4qCP4qCA4qCK4qCL4qCA4qCK4qCe4qCA4qC64qCB4qCO4qCd4qCE4qCe4qCA4qCX4qCl4qCd4qCA4qCD4qC94qCA4qCP4qCX4qCV4qCL4qCK4qCe4qCR4qCR4qCX4qCK4qCd4qCb4qCA4qCb4qCH4qCl4qCe4qCe4qCV4qCd4qCO4qCg4qCA4qCB4qCd4qCZCuKgveKgleKgpeKggOKgieKggeKgh%2BKgh%2BKggOKgpeKgjuKggOKgieKgl%2BKgiuKgjeKgiuKgneKggeKgh%2BKgjuKgqOKggOKggOKguuKgkeKggOKgkeKgreKgj%2BKgh%2BKgleKgl%2BKgkeKgqOKgqOKgqOKggOKggeKgneKgmeKggOKgveKgleKgpeKggOKgieKggeKgh%2BKgh%2BKggOKgpeKgjuKggOKgieKgl%2BKgiuKgjeKgiuKgneKggeKgh%2BKgjuKgqOKggOKggOKguuKgkeKggOKgjuKgkeKgkeKghQrioIHioIvioJ7ioJHioJfioIDioIXioJ3ioJXioLrioIfioJHioJnioJvioJHioKjioKjioKjioIDioIHioJ3ioJnioIDioL3ioJXioKXioIDioInioIHioIfioIfioIDioKXioI7ioIDioInioJfioIrioI3ioIrioJ3ioIHioIfioI7ioKjioIDioIDioLrioJHioIDioJHioK3ioIrioI7ioJ7ioIDioLrioIrioJ7ioJPioJXioKXioJ7ioIDioI7ioIXioIrioJ3ioIDioInioJXioIfioJXioJfioKAK4qC64qCK4qCe4qCT4qCV4qCl4qCe4qCA4qCd4qCB4qCe4qCK4qCV4qCd4qCB4qCH4qCK4qCe4qC94qCg4qCA4qC64qCK4qCe4qCT4qCV4qCl4qCe4qCA4qCX4qCR4qCH4qCK4qCb4qCK4qCV4qCl4qCO4qCA4qCD4qCK4qCB4qCO4qCo4qCo4qCo4qCA4qCB4qCd4qCZ4qCA4qC94qCV4qCl4qCA4qCJ4qCB4qCH4qCH4qCA4qCl4qCO4qCA4qCJ4qCX4qCK4qCN4qCK4qCd4qCB4qCH4qCO4qCoCuKgveKgleKgpeKggOKgg%2BKgpeKgiuKgh%2BKgmeKggOKggeKgnuKgleKgjeKgiuKgieKggOKgg%2BKgleKgjeKgg%2BKgjuKgoOKggOKgveKgleKgpeKggOKguuKggeKgm%2BKgkeKggOKguuKggeKgl%2BKgjuKgoOKggOKgveKgleKgpeKggOKgjeKgpeKgl%2BKgmeKgkeKgl%2BKgoOKggOKgieKgk%2BKgkeKggeKgnuKgoOKggOKggeKgneKgmeKggOKgh%2BKgiuKgkeKggOKgnuKgleKggOKgpeKgjgrioIHioJ3ioJnioIDioJ7ioJfioL3ioIDioJ7ioJXioIDioI3ioIHioIXioJHioIDioKXioI7ioIDioIPioJHioIfioIrioJHioKfioJHioIDioIrioJ7ioITioI7ioIDioIvioJXioJfioIDioJXioKXioJfioIDioJXioLrioJ3ioIDioJvioJXioJXioJnioKDioIDioL3ioJHioJ7ioIDioLrioJHioITioJfioJHioIDioJ7ioJPioJHioIDioInioJfioIrioI3ioIrioJ3ioIHioIfioI7ioKgKCuKggOKggOKggOKggOKggOKggOKggOKggOKgveKgkeKgjuKgoOKggOKgiuKggOKggeKgjeKggOKggeKggOKgieKgl%2BKgiuKgjeKgiuKgneKggeKgh%2BKgqOKggOKggOKgjeKgveKggOKgieKgl%2BKgiuKgjeKgkeKggOKgiuKgjuKggOKgnuKgk%2BKggeKgnuKggOKgleKgi%2BKggOKgieKgpeKgl%2BKgiuKgleKgjuKgiuKgnuKgveKgqOKggOKggOKgjeKgveKggOKgieKgl%2BKgiuKgjeKgkeKggOKgiuKgjgrioJ7ioJPioIHioJ7ioIDioJXioIvioIDioJrioKXioJnioJvioIrioJ3ioJvioIDioI/ioJHioJXioI/ioIfioJHioIDioIPioL3ioIDioLrioJPioIHioJ7ioIDioJ7ioJPioJHioL3ioIDioI7ioIHioL3ioIDioIHioJ3ioJnioIDioJ7ioJPioIrioJ3ioIXioKDioIDioJ3ioJXioJ7ioIDioLrioJPioIHioJ7ioIDioJ7ioJPioJHioL3ioIDioIfioJXioJXioIXioIDioIfioIrioIXioJHioKgK4qCN4qC94qCA4qCJ4qCX4qCK4qCN4qCR4qCA4qCK4qCO4qCA4qCe4qCT4qCB4qCe4qCA4qCV4qCL4qCA4qCV4qCl4qCe4qCO4qCN4qCB4qCX4qCe4qCK4qCd4qCb4qCA4qC94qCV4qCl4qCg4qCA4qCO4qCV4qCN4qCR4qCe4qCT4qCK4qCd4qCb4qCA4qCe4qCT4qCB4qCe4qCA4qC94qCV4qCl4qCA4qC64qCK4qCH4qCH4qCA4qCd4qCR4qCn4qCR4qCX4qCA4qCL4qCV4qCX4qCb4qCK4qCn4qCR4qCA4qCN4qCRCuKgi%2BKgleKgl%2BKgqAoK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCK4qCA4qCB4qCN4qCA4qCB4qCA4qCT4qCB4qCJ4qCF4qCR4qCX4qCg4qCA4qCB4qCd4qCZ4qCA4qCe4qCT4qCK4qCO4qCA4qCK4qCO4qCA4qCN4qC94qCA4qCN4qCB4qCd4qCK4qCL4qCR4qCO4qCe4qCV4qCo4qCA4qCA4qC94qCV4qCl4qCA4qCN4qCB4qC94qCA4qCO4qCe4qCV4qCP4qCA4qCe4qCT4qCK4qCO4qCA4qCK4qCd4qCZ4qCK4qCn4qCK4qCZ4qCl4qCB4qCH4qCgCuKgg%2BKgpeKgnuKggOKgveKgleKgpeKggOKgieKggeKgneKghOKgnuKggOKgjuKgnuKgleKgj%2BKggOKgpeKgjuKggOKggeKgh%2BKgh%2BKgqOKgqOKgqOKggOKggeKgi%2BKgnuKgkeKgl%2BKggOKggeKgh%2BKgh%2BKgoOKggOKguuKgkeKghOKgl%2BKgkeKggOKggeKgh%2BKgh%2BKggOKggeKgh%2BKgiuKgheKgkeKgqAoK4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCA4qCs4qCs4qCs4qCe4qCT4qCR4qCA4qCN4qCR4qCd4qCe4qCV4qCX4qCs4qCs4qCs) recipe. After decoding, the text can be compared to the original manifesto on Phrack.org. A difference in one paragraph reveals the hidden flag. 


![image](https://gist.github.com/user-attachments/assets/0c80a121-8f1a-425d-b022-bd4048e3443a)

`r00t{edae962125814dae54fa6e4b628fd6eb}`

## YAZ (Yet Another Zine)

### Challenge Overview: 

Yet Another Zine, damn. 🤦‍♂️ Anyway, I TWEETed about this zine no one seemed to relate. Can you maybe make sense out of it? 😅

For those who unlocked/redeemed the hint for 20 points 😭😭😭, It read:

> `flag: r00t{} , Flag starts with a 9 and ends with a 2 😎`

### Solution

The word `TWEET` has been stressed intentionally. Looking at the file given, it might looking visually similar to the original, although its not. A hidden message has been added to the text. If you do some research on google, you will learn of a steganography technique used to hide messages in tweets. There are many sites to decode such text. For example:

- https://holloway.nz/steg/
- https://wulfsige.com/crypto/

![image](https://gist.github.com/user-attachments/assets/55e97b7b-f332-4b7a-86e4-4d560e0d0a39)

`r00t{9333d0e7c6e1e085b9115d9d1971cce2}`


## SheetsNlayers-1

### Challenge Overview: 

What is flag 1?

### *Solution*

You are given a zip achieve containing`SheetsNlayers.vhdx`


> Simply put, "*This is  Windows hard disk image file. It acts like a real, physical hard drive, but is stored in a single file that's located on a physical disk like a hard drive. VHDX files can contain an entire operating system for purposes such as testing software or running older or newer software not compatible with the host operating system, or simply to hold files like any other storage container.*" ~Source: [Lifewire](https://www.lifewire.com/what-is-a-hard-disk-drive-2618152)
{: .prompt-tip }

On windows, there are multiple ways to access the image:
#### Method 1

- Double click on the file to mount it

![image](https://gist.github.com/user-attachments/assets/0a789779-fb44-45aa-8d1c-b6115903e85e)

![image](https://gist.github.com/user-attachments/assets/efc59d57-88fc-4a44-a590-821ef9cb60af)

Flag 1 can be found there:

![image](https://gist.github.com/user-attachments/assets/527ac663-af08-45fb-a08b-8fc18d0f648b)

However, it looks like base64 but strange. The trick was reversing the string and then base64 decoding.  [CyberChef Recipe](https://gchq.github.io/CyberChef/#recipe=Reverse('Character')From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=PTAzWWtOVE5sTnpZalpqWW1WR01tRldaNE1UTzFRV1kwQXpNME1tTmhkak16c0hkd0FqYw)

![image](https://gist.github.com/user-attachments/assets/f82ac2f2-447b-4719-96cd-6a3288d4c978)

`r00t{327a6c4304ad5938eaf0efb6cc3e53dc}`

#### Method 2

Alternatively, you could have opened the `vhdx` file with FTK Imager as follows:

![image](https://gist.github.com/user-attachments/assets/0ca2b7eb-8294-47af-be52-5d1e1c5fbba4)

![image](https://gist.github.com/user-attachments/assets/a3a3fbb1-c30a-4e4e-8bf7-6bfbf9bf1172)

![image](https://gist.github.com/user-attachments/assets/7b2afb7c-bc1f-4d69-a516-46dd3e44fefc)

![image](https://gist.github.com/user-attachments/assets/907f7f4a-5571-45e4-9184-0251e2465548)

#### Method 3 

- On start, search for `create and format hard disk partitions` or just hit `win+r` and search for `diskmgmt.msc`
- Click on `Action > Attach VHD`
- Choose Disk Location and click Ok as shown:

![image](https://gist.github.com/user-attachments/assets/0739d078-7145-4f48-bb01-ceac35489c8a)

![image](https://gist.github.com/user-attachments/assets/3810cff7-ba22-41f3-9e5a-bf8805e19708)


## SheetsNlayers-2
### Challenge Overview: 

What is flag 2?

### Solution

On the same disk file, there was a note.txt file with further instructions:

![image](https://gist.github.com/user-attachments/assets/9a75af72-f17c-41af-9cd3-c7538465cb0f)

Unzipping `moreflags.zip` requires you to have a password:

![image](https://gist.github.com/user-attachments/assets/211eaed0-e9c3-48bc-ada9-357664b33cab)

Get cracking with `john` & `zip2john` as shown:

```bash
➜  zip2john moreflags.zip > forjohn
ver 2.0 moreflags.zip/moreflags.xlsx PKZIP Encr: cmplen=10899, decmplen=15420, crc=F053B033 ts=7C3A cs=f053 type=8
➜  john -w=~/Desktop/rockyou.txt forjohn
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
ne.11.88         (moreflags.zip/moreflags.xlsx)
1g 0:00:00:03 DONE (2024-11-17 20:04) 0.3257g/s 1683Kp/s 1683Kc/s 1683KC/s neagupavel..nazir4u
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
{: .nolineno }

After getting the password (`ne.11.88`) you extract an `.xlsx` file.


> After that I later realized there were different ways to approach it...*probably an oversite on my end but oh well. 🤦‍♂️*
{: .prompt-info }

Opening the file on Excel, you get what looks like a blank workbook:

![image](https://gist.github.com/user-attachments/assets/9dbbcf33-9e51-4c59-abb4-47841c0be0be)

#### Unintended method 1 - Strings

Running strings on the file, you will notice that there are 4 sheets in the doc... Probably hidden?

```bash
➜  strings moreflags.xlsx

--- redacted ----

xl/worksheets/_rels/sheet1.xml.relsPK
xl/worksheets/_rels/sheet2.xml.relsPK
xl/worksheets/_rels/sheet3.xml.relsPK
xl/worksheets/_rels/sheet4.xml.relsPK

--- redacted ----
```
{: .nolineno }

#### Unintended method 2 - Online tools

If you are familiar with [IRIS-H Digital Forensics](https://iris-h.services/#)- an online site to analyze malicious documents, you could easily get hints to flag 2-4.

[Eg:](https://iris-h.services/pages/report/591123936fea0a43f13f68abecf99f87677ec700)

![image](https://gist.github.com/user-attachments/assets/896b405e-2d82-4ea8-9c46-631e933606c5)

![image](https://gist.github.com/user-attachments/assets/a9bd263d-29ef-491d-bf9f-31f9c021140d)

#### Intended

Check for hidden sheets by right clicking on the visible sheet and click on unhide: 

![image](https://gist.github.com/user-attachments/assets/5a4bade5-d7df-44c3-a0fc-2b77c7be19fe)

You will then get a list of hidden sheets.

![image](https://gist.github.com/user-attachments/assets/68cac92c-83fa-4b9c-8702-6074101c54ff)

Click on each to unhide. For flag 2, it's preety easy to locate it in cell `A1`.

![image](https://gist.github.com/user-attachments/assets/d0d39680-3780-4abc-acb7-e127aa6a6dc6)

However its not as straight foward, but rather a weirdly looking string `E\K1hHXp_E3+?<-2_nlQAi"$R2_m0G2.U3.AN;YW2e?DQ0ms` … So looking it up on [Cyberchef](https://gchq.github.io/CyberChef/#recipe=Magic(4,true,false,'')&input=RVxLMWhIWHBfRTMrPzwtMl9ubFFBaSIkUjJfbTBHMi5VMy5BTjtZVzJlP0RRMG1z&oeol=CR), you will get the flag [Base85](https://gchq.github.io/CyberChef/#recipe=From_Base85('!-u',true,'z')&oeol=CR) encoded.

![image](https://gist.github.com/user-attachments/assets/16a100f2-a374-4334-bbdf-0bb3e25ce123)

`r00t{df38bae72ccf3f172345dcee96a7ea21}`

## SheetsNlayers-3

### Challenge Overview

What is flag 3?

### Solution

Unhiding `Sheet 3` , it kinda looks blank, but the trick was more of searching/finding the exact cell containing the flag... 

![image](https://gist.github.com/user-attachments/assets/348cd3db-61f4-4603-9fe8-1aa07096563c)

In this case the flag was in cell `FLA3`. 

```bash
TVRFMElEUTRJRFE0SURFeE5pQXhNak1nTlRZZ05UVWdPVGtnT1RnZ09Ua2dPVGNnTlRRZ05Ea2dOVGNnTlRFZ05UVWdNVEF4SURFd01TQXhNREVnTlRRZ05UQWdORGdnTlRZZ05Ea2dOVEFnTVRBeUlEVTNJREV3TVNBMU5pQXhNREVnTlRNZ09Ua2dOVE1nTlRFZ01UQXdJRFE0SURRNElERXlOUT09
```
{: .nolineno }

Again, using [Cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)From_Base64('A-Za-z0-9%2B%5C%5C-%3D',true,false)From_Decimal('Space',false)&input=VFZSRk1FbEVVVFJKUkZFMFNVUkZlRTVwUVhoTmFrMW5UbFJaWjA1VVZXZFBWR3RuVDFSblowOVVhMmRQVkdOblRsUlJaMDVFYTJkT1ZHTm5UbFJGWjA1VVZXZE5WRUY0U1VSRmQwMVRRWGhOUkVWblRsUlJaMDVVUVdkT1JHZG5UbFJaWjA1RWEyZE9WRUZuVFZSQmVVbEVWVE5KUkVWM1RWTkJNVTVwUVhoTlJFVm5UbFJOWjA5VWEyZE9WRTFuVGxSRlowMVVRWGRKUkZFMFNVUlJORWxFUlhsT1VUMDk) to analyze the string, you will notice it is double base64 encoded. From the decoded string, you get another hex string which you also need you need to add to the recipe.

![image](https://gist.github.com/user-attachments/assets/88c63895-2078-401c-a243-43556954a452)


![image](https://gist.github.com/user-attachments/assets/8a6d14e9-c09d-48e8-80b1-450d097c0cd7)


`r00t{87cbca61937eee620812f9e8e5c53d00}`

## SheetsNlayers-4

### Challenge Overview

What is flag 3?

### Solution

Unhiding sheet `flag4`, you will realize a section of cells written `No comment`. I thought this was gonna be preety straight foward but oh well 😅

![image](https://gist.github.com/user-attachments/assets/f6471b1c-f35b-4563-a585-2383887df4e8)

So the trick here was basically to inspect the sheet for comments. Simply click on Info on the file tab:

![image](https://gist.github.com/user-attachments/assets/4126ce84-9c78-4e50-9365-aaecf2ae8aec)

Check for issues:

![image](https://gist.github.com/user-attachments/assets/8cbe0aab-6365-4cfb-a940-66cf05155d76)

![image](https://gist.github.com/user-attachments/assets/375042aa-3506-4020-8b07-af864e203ef6)

Make sure Comments is checked then click on  Inspect

![image](https://gist.github.com/user-attachments/assets/a5c4f908-1405-4cb0-995e-3e60fbcbb252)

You will notice that there is actually a comment.

![image](https://gist.github.com/user-attachments/assets/0538f83e-6431-4b74-b352-37365c7d254c)

To view the comment, navigate to the `Review` tab and click on `Show All Comments`

![image](https://gist.github.com/user-attachments/assets/ed8d341c-4267-48c2-9ad4-68ea102150b1)

You will then see a flag in cell B4. `C__ELgf4342e`hbf666ea_g`a7h6g6d4db5__N`

![image](https://gist.github.com/user-attachments/assets/e323bbcf-ca67-4afc-9cb9-9e961a7e2505)

Another wierd string 😑. Doing a quick lookup on [CyberChef](https://gchq.github.io/CyberChef/#recipe=ROT47(47)&input=Q19fRUxnZjQzNDJlYGhiZjY2NmVhX2dgYTdoNmc2ZDRkYjVfX04) or [dcode.fr](https://www.dcode.fr/rot-47-cipher), you get the flag.

![image](https://gist.github.com/user-attachments/assets/e173b9cd-54fd-4df8-802c-808986b2ec98)


`r00t{87cbca61937eee620812f9e8e5c53d00}`


That comes to the end of my writeup, I hope you enjoyed it or learnt something new (for the beginners). My team members from Pwnus did a [blog post](https://blog.pwnus.site/posts/P3rf3ctr00t-CTF/) on the rest of the challenges they managed to solve. Feel free to check it out. ✌ See you on my next post. 