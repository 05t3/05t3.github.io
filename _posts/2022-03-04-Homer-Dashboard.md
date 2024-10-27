---

title: "Dashboards - Homer"
date: 2022-03-04 01:09:33 +0300
author: oste
image: /assets/img/Posts/Homer.png
categories: [HOMELAB]
tags:
  [
    raspberry pi,
    docker,
    ubuntu,
    portainer,
    docker containers,
    docker images,
    homer,
    dashboard,
  ]
---

# Coming Soon

<!-- ![image](https://user-images.githubusercontent.com/58165365/151720953-a67781fc-e803-4ea4-b36e-1c28ed3bfa29.png)
Monitoring

# Homer Dashboard

So we now have our server up and runnning, installed docker and running portainer to manage our docker containers. At this point we now need a dashboard that contains a list of services we are running on our server. Here is where Homer comes in handy. Homer is basically a simple static **HOMepage for your servER** to keep your services on hand, from a simple _yaml_ configuration file.

You can get the project on github at [bastienwirtz/homer](https://github.com/bastienwirtz/homer)

# Installation

To get started, you can run this container using the CLI as:

```bash
docker run -d \
  -p 8080:8080 \
  -v </your/local/assets/>:/www/assets \
  --restart=always \
  b4bz/homer:latest
```

Default assets are automatically installed in the `/www/assets` directory. Use `UID` and/or `GID` env var to change the assets owner (`docker run -e "UID=1000" -e "GID=1000" [...]`).

In our case, we are going to make this interesting by creating a container on portainer.

But first, we need to create a directory which we will map to the `/www/assets` directory of the container. In this case, i created mine in `/home/ubuntu/containers/homer`

With that done, we can jump to portainer and follow the steps outlined below:

- Name your container
- Add the docker image to use: `b4bz/homer:latest`
- Homer runs on port 8080, so we need to map it to a port which is not in use on our host machine. in this case, i used `8000`.

![image](https://user-images.githubusercontent.com/58165365/157215971-628237ec-533e-4866-b984-a9260e5f117b.png)

- Create a bind volume volume which will be stored on our `/home/ubuntu/containers/homer` directory we created earlier.

![image](https://user-images.githubusercontent.com/58165365/155893225-cb87cdcd-8889-4c5d-8d2e-781216f23973.png)

- We need to set the containers restart policy as `always`.

![image](https://user-images.githubusercontent.com/58165365/155893254-72a21f0d-b486-424a-aea1-ba2387937d99.png)

- Add `GID` & `UID` environment variables.

![image](https://user-images.githubusercontent.com/58165365/157215446-18b11aa7-f68e-43e1-aebe-a303737bfcb7.png)

For your case you can confirm the values above by running the `id` command:

```bash
ubuntu@oste:~$ id
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),115(netdev),118(lxd)
```

With that done, we can deploy the container and visit the dashboard on the port assigned as follows:

![image](https://user-images.githubusercontent.com/58165365/151720928-73244beb-d78e-4515-be57-0b8fb046979d.png)

By default, this is how the Homer dashboard looks like. How about we tweak the code a little and tailor it to meet our need?

Using your favourite text editor, you can navigate to the volume mounted and find the `config.yml` file. This is the file we need to make some changes.

```bash
root@oste:/home/ubuntu/containers/homer# ls -la
total 12
drwxr-xr-x 3 root   root   4096 Jan 30 23:26 .
drwxr-xr-x 5 root   root   4096 Feb 27 17:54 ..
drwxr-xr-x 4 ubuntu ubuntu 4096 Jan 30 23:26 assets
root@oste:/home/ubuntu/containers/homer# cd assets/
root@oste:/home/ubuntu/containers/homer/assets# ls -la
total 44
drwxr-xr-x 4 ubuntu ubuntu 4096 Jan 30 23:26 .
drwxr-xr-x 3 root   root   4096 Jan 30 23:26 ..
-rw-r--r-- 1 ubuntu ubuntu 1263 Jan 30 23:26 additionnal-page.yml.dist
-rw-r--r-- 1 ubuntu ubuntu 6535 Jan 31 01:49 config.yml
-rw-r--r-- 1 ubuntu ubuntu 2948 Jan 30 23:26 config.yml.dist
-rw-r--r-- 1 ubuntu ubuntu 2010 Jan 30 23:26 config.yml.dist.sample-sui
-rw-r--r-- 1 ubuntu ubuntu  176 Jan 30 23:26 custom.css.sample
drwxr-xr-x 2 ubuntu ubuntu 4096 Jan 31 00:22 icons
-rw-r--r-- 1 ubuntu ubuntu  655 Jan 30 23:26 manifest.json
drwxr-xr-x 2 ubuntu ubuntu 4096 Jan 31 00:31 tools
root@oste:/home/ubuntu/containers/homer/assets#
```

In my case, i Created different categories such as :

- Cloud
- Docker Management
- Home Servers
- Monitoring services
- Remote Connection
- WiFi routers
- Web Labs.

In the end, the site would look something like this.

// screeshot.

There are many other dashboards you can use such as:

- [Heimdal](https://github.com/linuxserver/Heimdall)
- [Organizr](https://github.com/causefx/Organizr)
- [Dashy](https://github.com/Lissy93/dashy)
- [DashMachine](https://github.com/rmountjoy92/DashMachine)
- [Homepage](https://github.com/tomershvueli/homepage)

I will probably cover some of them in future blog posts. I hope you enjoyed this read and looking forward to hearing your thoughts on the same. Lemmy know what you have been using in the comment section. Unti next time, take care and keep safe. -->
