---

title: "Cheat Sheets"
date: 2022-06-14 01:09:33 +0300
author: oste
image: /assets/img/Posts/cheats.png
categories: [Cheats]
tags: [wget]
---

# TOOLING

These are among the common Linux command line utilities that can be used by T1 Analysts

## **UTILITIES**

- **dig**
- **ping**
- **whois**
- **strings**
- **file**
- **curl**
- **wget**

<details>
<summary>Wget</summary>
<pre>

- Installation

`sudo apt install wget`

or

`sudo yum install wget`

- Download a File

`wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.17.2.tar.xz`

- To save the downloaded file under a different name

`wget -O latest-hugo.zip https://github.com/gohugoio/hugo/archive/master.zip`

- Downloading a File to a Specific Directory

`wget -P /mnt/iso http://mirrors.mit.edu/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1804.iso`

- If you want to limit the Download Speed

`wget --limit-rate=1m https://dl.google.com/go/go1.10.3.linux-amd64.tar.gz`

where: in the above we are limiting the download speed to 1MB

- `k` for kilobytes
- `m` for megabytes
- `g` for gigabytes
- When Downloading a file in the Background

`wget -b https://download.opensuse.org/tumbleweed/iso/openSUSE-Tumbleweed-DVD-x86_64-Current.iso`

Output is redirected to `wget-log`. To watch the status of the download, use the `tail` command

`tail -f wget-log`

- Changing the Wget User-Agent

`wget --user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0" http://wget-forbidden.com/`

- Downloading Multiple Files

`wget -i linux-distros.txt`

where:

```bash
http://mirrors.edge.kernel.org/archlinux/iso/2018.06.01/archlinux-2018.06.01-x86_64.iso
https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.4.0-amd64-netinst.iso
https://download.fedoraproject.org/pub/fedora/linux/releases/28/Server/x86_64/iso/Fedora-Server-dvd-x86_64-28-1.1.iso
```

- To download a file from a password-protected FTP server

`wget --ftp-user=FTP_USERNAME --ftp-password=FTP_PASSWORD ftp://ftp.example.com/filename.tar.gz`

- Creating a Mirror of a Website

`wget -m https://example.com`

- Creating a Mirror of a Website for local browsing

`wget -m -k -p https://example.com`

- Skipping Certificate Check

`wget --no-check-certificate https://domain-with-invalid-ss.com`

- Downloading to the Standard Output

`wget -q -O - "http://wordpress.org/latest.tar.gz" | tar -xzf - -C /var/www`

In the following example, wget will quietly ( flag `-q`) download and output the latest WordPress version to stdout ( flag `-O`) and pipe it to the tar utility, which will extract the archive to the /var/www directory.

</pre>
</details>
