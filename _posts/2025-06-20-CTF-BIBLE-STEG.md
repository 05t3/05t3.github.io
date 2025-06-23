---
title: "CTF BIBLE - STEGANOGRAPHY"
date: 2025-06-20 01:09:33 +0300
author: oste
description: Collection of Tools, Cheats sheets I use for solving various kinds of CTF Challenges
image: /assets/img/Posts/CTF BIBLE - Steganography.png
categories: [Cheat Sheets, CTFs]
tags:
  [steg, steganography, forensics, OSINT, malware, rev, web]
---

> This Cheat Sheet is still work in progress and will be constantly updated.
{: .prompt-info }

# Steganography

## Tool Summary


| Tool                                                            | Description                                                                                                                                                                                                                                                                         | Supported Formats     |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| [stegosuite](https://www.kali.org/tools/stegosuite/)            | Stegosuite is a graphical steganography tool to easily hide information in image files. It allows the embedding of text messages and multiple files of any type. In addition, the embedded data is encrypted using AES.                                                             | BMP, GIF, JPG and PNG |
| [stegcracker](https://www.kali.org/tools/stegcracker/)          | StegCracker is steganography brute-force utility to uncover hidden data inside files.                                                                                                                                                                                               |                       |
| [stegsnow](https://www.kali.org/tools/stegsnow/)                | This utility can conceal messages in ASCII text by appending whitespaces to the end of lines.                                                                                                                                                                                       |                       |
| [steghide](https://www.kali.org/tools/steghide/)                | Steghide is steganography program which hides bits of a data file in some of the least significant bits of another file in such a way that the existence of the data file is not visible and cannot be proven.                                                                      |                       |
| [stegoVeritas](https://github.com/bannsec/stegoVeritas)         | Yet another Stego Tool                                                                                                                                                                                                                                                              | gif,jpeg,png,tiff,bmp |
| [zsteg](https://github.com/zed-0xff/zsteg)                      | detect stegano-hidden data in PNG & BMP                                                                                                                                                                                                                                             | PNG & BMP             |
| [Secret Messages](https://holloway.nz/steg/)                    | Put hidden messages in the text you write.                                                                                                                                                                                                                                          |                       |
| [outguess](https://www.kali.org/tools/outguess/)                | OutGuess is a universal tool for steganography that allows the insertion of hidden information into the redundant bits of data sources.                                                                                                                                             | JPEG, PPM and PNM     |
| [SilentEye](https://github.com/achorein/silenteye)              | SilentEye is a cross-platform application design for an easy use of steganography, in this case hiding messages into pictures or sounds.                                                                                                                                            |                       |
| [htstego](https://github.com/efeciftci/htstego)                 | This is a steganography utility for generating stego images in halftone format and extracting payloads from such generated images.                                                                                                                                                  |                       |
| [iSteg](https://github.com/rafiibrahim8/iSteg)                  | iSteg can hide your file inside an image using LSB method                                                                                                                                                                                                                           |                       |
| [DeepSound](https://github.com/Jpinsoft/DeepSound)              | DeepSound is a steganography tool and audio converter that hides secret data into audio files. The application also enables you to extract secret files directly from audio files or audio CD tracks.                                                                               |                       |
| [OpenStego](https://github.com/syvaidya/openstego)              | OpenStego is a steganography application that provides two functionalities:<br>1. **Data Hiding**: *It can hide any data within an image file.*<br>2. **Watermarking**: *Watermarking image files with an invisible signature. It can be used to detect unauthorized file copying.* |                       |
| [Binwalk](https://www.kali.org/tools/binwalk/)                  | Binwalk is a tool for searching a given binary image for embedded files and executable code.                                                                                                                                                                                        |                       |
| **[wav-steg-py](https://github.com/pavanchhatpar/wav-steg-py)** | A python script to hide information over an audio file in .wav format                                                                                                                                                                                                               |                       |
| [stego-lsb](https://github.com/ragibson/Steganography)          | - `wavsteg` - WavSteg uses least significant bit steganography to hide a file in the samples of a .wav file.                                                                                                                                                                        | .wav                  |
|                                                                 | - `steglsb` - LSBSteg uses least significant bit steganography to hide a file in the color information of an RGB image (.bmp or .png).                                                                                                                                              | .bmp or .png          |
|                                                                 | - `stegdetect` - StegDetect provides one method for detecting simple steganography in images.                                                                                                                                                                                       |                       |
| [stegano](https://pypi.org/project/stegano/)                    | Stegano only hide messages, without encryption and is a pure Python Steganography module.                                                                                                                                                                                           |                       |
| [OpenPuff](https://openpuff.en.lo4d.com/windows)                | Securely hide your sensitive information within other files and protect your privacy with this powerful steganography tool.                                                                                                                                                         |                       |
| [Sonic Visualiser](https://www.sonicvisualiser.org/)            | Sonic Visualiser is a free, open-source application for Windows, Linux, and Mac, designed to be the first program you reach for when want to study a music recording closely.                                                                                                       |                       |
| [Audacity](https://www.audacityteam.org/)                       | It allows users to visualize audio signals as spectrograms, which can reveal hidden messages embedded within the audio.                                                                                                                                                             |                       |


## Tool installation

```bash
sudo apt install outguess stegosuite stegsnow stegcracker steghide binwalk python3-binwalk

gem install zsteg

pip3 install stegoveritas
stegoveritas_install_deps

pip install stego-lsb stegano
```
{: .nolineno }