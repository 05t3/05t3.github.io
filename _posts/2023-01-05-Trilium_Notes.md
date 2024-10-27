---
title: Trilium Notes (Self-Hosted)
description: Trilium Notes (Self-Hosted)
author: oste
date: 2023-01-05 01:09:33 +0300
categories: [HOMELAB]
tags: [docker, trilium, notes, ubuntu, self-hosted]
image: /assets/img/Posts/trilium.png
#   path: assets/img/Posts/forest.png
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---


Hello and happy new year. Welcome to my first blog in '23 where i will be showing you how easy you can self-host your note-taking application at home. I feel like the search for a good notetaking platform is never-ending. I've been using Github Gists, Obsidian and Notion for preety much 2 years now but this year i decided to try something new. Trilium Notes, a project by [zadam](https://github.com/zadam/trilium/) is a hierarchical note taking application with focus on building large personal knowledge bases.

It is provided as either desktop application (Linux and Windows) or web application hosted on your server (Linux). Mac OS desktop build is available, but it is [unsupported](https://github.com/zadam/trilium/wiki/FAQ#mac-os-support).

With that said, lets get started.🙂 I'm hosting it on my home network on a Raspberry-Pi, running Ubuntu server. Deployment method is a docker container.😁 I followed [zadam's](https://github.com/zadam/trilium/wiki/Docker-server-installation) knowledge base.

I used this docker-compose file to bring up the container:

```yaml
version: "3.3"
services:
  trilium:
    image: zadam/trilium:latest
    restart: unless-stopped
    ports:
      - "9999:8080"
    volumes:
      - /home/ubuntu/containers/trillium:/home/node/trilium-data
```

A lil about the compose file.

- We are using compose version 3.3
- We are using the latest build image from [docker hub](https://hub.docker.com/r/zadam/trilium/tags)
- We are setting the container restart policy to always keep the instance running unless the container is stopped.
- By default, trilium runs on port 8080. Since i have another different container using the same port, i assigned mine 9999.
- Trilium needs a directory where it can store its data, this then needs to be mounted into the docker container. The container needs to runs as a root to be able to access it in write mode.. In my case, `/home/ubuntu/containers/trillium` is the directory created.

From there we can go ahead and compose the file as show below:

![image](https://user-images.githubusercontent.com/58165365/210444836-24005ec2-2a9c-42b2-a215-62179790ab8c.png)

After getting similar output, access the trilium web editor by opening a browser and going to the server IP/FQDN followed by the port specificed earlier. Here we find the initial setup guide:

![image](https://user-images.githubusercontent.com/58165365/210438263-507006f5-7d75-410a-82e1-156e655dee54.png)

After selecting the first option, we proceed to setup a password.

![image](https://user-images.githubusercontent.com/58165365/210438297-06d01d18-44e4-4c58-9e67-61a88400976c.png)

Once set, you will be prompted to login.

![image](https://user-images.githubusercontent.com/58165365/210438324-163c46c4-675b-49a4-8ec6-57823195e39e.png)

Upon a successful login, we get access to a beautiful homepage. The UI honestly looks preety awesome 😍.

> Trillium has been written in JavaScript btw.

![image](https://user-images.githubusercontent.com/58165365/210438363-42ac1d9d-ec35-4abb-8c89-5974489c43f4.png)

Before we even start diving in, lets look at how much resources are consumed by the container.

![image](https://user-images.githubusercontent.com/58165365/210713664-352fb6c8-59ac-4d32-a715-54b9ecdf7d9d.png)

_Very minimal. 👌_

---

Poking around the templates on the left navigation pane, we get a variety of good looking and helpful teamplates. For example:

_[Weight Tracker](https://github.com/zadam/trilium/wiki/Weight-tracker)_

![image](https://user-images.githubusercontent.com/58165365/210438779-3332945f-56a9-442f-a7d0-d03708c953fe.png)

---

_**Statistics**_

![image](https://user-images.githubusercontent.com/58165365/210439070-ced0b3e8-407f-4687-b4db-053a6df0f5f2.png)

---

_**Mermaid Diagrams**_

![image](https://user-images.githubusercontent.com/58165365/210439430-60e0bffd-f616-4a4f-93a7-a6d5a1d24740.png)

---

_**Canvas Notes**_

![image](https://user-images.githubusercontent.com/58165365/210439531-dc4807e8-55dc-43e5-932d-e9f149b40c13.png)

---

_**Formatting Options**_

![image](https://user-images.githubusercontent.com/58165365/210439672-2f08f6bd-2c7e-42c1-848b-34678501557b.png)

---

_**Family Trees**_

![image](https://user-images.githubusercontent.com/58165365/210439965-3796c9af-4bbb-4ee0-9a1b-db091d819ccb.png)

---

Ohh, and its search functionality is powerfull too 🙌

![image](https://user-images.githubusercontent.com/58165365/210440459-53f7db98-5b05-488c-84ca-37fff0cdf0d8.png)

Trilium also has a switch for changing the web layout to `Mobile View` as shown:

![image](https://user-images.githubusercontent.com/58165365/210446150-6f0aacd1-ca0c-4f03-9a5e-8e593cfd011c.png)

Moving on, clicking on the icon on the top left, shows you more options you can tinker with.

![image](https://user-images.githubusercontent.com/58165365/210713602-7fa10d7c-4145-47e4-aeee-8492bc24e1d3.png)

For instance, You can use the SQL Console to query your notes from the Database.

![image](https://user-images.githubusercontent.com/58165365/210446194-8707f43f-709c-4de3-b11c-aeaaa51402f3.png)

I also tried creating a dummy text as shown:

![image](https://user-images.githubusercontent.com/58165365/210447633-01a3d02b-9398-4106-b458-d2e344aba79d.png)

# Review

- It looks preety awesome and powerful and cant wait to unleash its full potential overtime. However, i feel like its abit complex 🙈. _Time will tell_ .
- Can I compare it to Notion? 🙅‍♂️, Notion is quite powerful with tons of integrations (GitHub, Figma, Trello, GoogleDrive, CodePen. Slack, Zoom, Asana). I'm yet to see if Trilium accepts integrations or installing plugins.😁
- Can you import or export notes on Trilium? Trilium can import ENEX files which are used by Evernote for backup/export and also markdown. You can also export notes as MD, HTML and OPML. Compared to notion, i feel like notion is King, in the sense that i can import notes from/as: Evernote, Trello, Asana, Confluence, txt, MD, CSV, HTML, Word, Google Docs, Dropbox paper, Quip and Workflowy. Exporting? PDF,HTML, MD & CSV. Notion wins on this one.
- Lets talk about synchronization. Trilium supports synchronization between desktop client and a server instance. In my case I am using a server instance. If i had a desktop client on a different workstation and wanted to sync notes with my server instance, I would select that option when doing the initial installation setup as shown in the third option below:

![image](https://user-images.githubusercontent.com/58165365/210438263-507006f5-7d75-410a-82e1-156e655dee54.png)

More information about its sync capabilities can be found [here](https://github.com/zadam/trilium/wiki/Synchronization). Obsidian doesn't have that sync feature. Notion is not self-hosted but you can sync your notes from the browser, mobile application and also desktop application.

From their Github README, they boast of the following features:

![image](https://user-images.githubusercontent.com/58165365/210756687-9752ad56-c213-49f4-84ad-6e7578b43c0d.png)

Since i haven't tested them all, i will do a proper review of the same in subsequent blogs. Thanks for reading up to this point, lemmy know your thought on the same should you try it out. If you have any other #OpenSource deployable solutions or fun projects that you would want me to cover, feel free to reach out on the same. Cheers ✌🏼
