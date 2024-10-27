---

title: "Docker Installation"
date: 2021-12-2 01:09:33 +0300
author: oste
image: /assets/img/Posts/docker.png
categories: [DOCKER]
tags: [docker, docker containers, ubuntu, docker images]
---

In this walkthrough, i am going to show you how you can install docker on [Ubuntu 20.04](https://ubuntu.com/download/desktop) in just 10 steps. However, minimal and cloud native OS are the preferred choice for a docker host since they are designed and configured to run containers and given their small footprint, they consume relatively fewer resources that their counterparts. Some of these Operating systems Include:

- [RancherOS](https://rancher.com/docs/os/v1.x/en/)
- [Ubuntu Core](https://ubuntu.com/core)
- [Core OS](https://getfedora.org/en/coreos?stream=stable)

Feel free to explore the above and compare the difference. Let me explain about the docker architecture briefly before we go further.

> Docker uses a client-server architecture. The Docker _client_ talks to the Docker _daemon_, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate using a **REST API**, over UNIX sockets or a network interface. Another Docker client is **Docker Compose**, that lets you work with applications consisting of a set of containers.
> Source: _[Docker Documentation](https://docs.docker.com/get-started/overview/)_

![image](https://user-images.githubusercontent.com/58165365/144400225-5feef0f8-5afe-461e-a01d-363b9199bac8.png)

If you had previously installed docker on your system and you happened to encounter some problems for some reason, you can follow along but first you will need to run the following command in order to uninstall older versions, for example `docker`, `docker.io`, or `docker-engine`

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

The contents of `/var/lib/docker/`, including images, containers, volumes, and networks, are preserved. If you do not need to save your existing data, and want to start with a clean installation, simple run the following commands

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

With that done, lets get started.

1. We'll start by updating and upgrading our system using the following command

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

2. We also need to install a few prerequisite packages which let apt use packages over HTTPS

```bash
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

3. Add Docker’s official GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

4. Add the Docker repository to APT sources:

```bash
 echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5. Update the package database with the Docker packages from the newly added repo:

```bash
sudo apt-get update
```

6. To make sure you are about to install from the Docker repo instead of the default Ubuntu repo:

```bash
apt-cache policy docker-ce
```

You’ll see output like this, although the version number for Docker may be different:

```bash
docker-ce:
  Installed: (none)
  Candidate: 5:20.10.7~3-0~ubuntu-focal
  Version table:
     5:20.10.7~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:20.10.6~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:20.10.5~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:20.10.4~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:20.10.3~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:20.10.2~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:20.10.1~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:20.10.0~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.15~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.14~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.13~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.12~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.11~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.10~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.9~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
```

7. Update the apt package index, and install the latest version of `Docker Engine` and `containerd`

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

8. Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:

```bash
sudo systemctl status docker
```

The output should be similar to the following, showing that the service is active and running:

```bash
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2021-07-30 13:52:36 EAT; 3min 37s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 6836 (dockerd)
      Tasks: 8
     Memory: 53.4M
     CGroup: /system.slice/docker.service
             └─6836 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
Ado 30 13:52:34 docker dockerd[6836]: time="2021-07-30T13:52:34.834129261+03:00" level=warning msg="Your kernel does not support CPU realtime >
Ado 30 13:52:34 docker dockerd[6836]: time="2021-07-30T13:52:34.835495233+03:00" level=warning msg="Your kernel does not support cgroup blkio >
Ado 30 13:52:34 docker dockerd[6836]: time="2021-07-30T13:52:34.835857823+03:00" level=warning msg="Your kernel does not support cgroup blkio >
Ado 30 13:52:34 docker dockerd[6836]: time="2021-07-30T13:52:34.836753497+03:00" level=info msg="Loading containers: start."
Ado 30 13:52:35 docker dockerd[6836]: time="2021-07-30T13:52:35.486328397+03:00" level=info msg="Default bridge (docker0) is assigned with an >
Ado 30 13:52:35 docker dockerd[6836]: time="2021-07-30T13:52:35.989845144+03:00" level=info msg="Loading containers: done."
Ado 30 13:52:36 docker dockerd[6836]: time="2021-07-30T13:52:36.855617392+03:00" level=info msg="Docker daemon" commit=b0f5bc3 graphdriver(s)=>
Ado 30 13:52:36 docker dockerd[6836]: time="2021-07-30T13:52:36.856528926+03:00" level=info msg="Daemon has completed initialization"
Ado 30 13:52:36 docker systemd[1]: Started Docker Application Container Engine.
Ado 30 13:52:37 docker dockerd[6836]: time="2021-07-30T13:52:37.201020910+03:00" level=info msg="API listen on /run/docker.sock"
```

9. We can check the docker version by simply running the following command

```bash
docker@docker:~/Desktop$ docker --version
Docker version 20.10.7, build f0df350
docker@docker:~/Desktop$
```

10. To verify that Docker Engine is installed correctly by running the hello-world image.

```bash
docker@docker:~/Desktop$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:df5f5184104426b65967e016ff2ac0bfcd44ad7899ca3bbcf8e44e4461491a9e
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
This message shows that your installation appears to be working correctly.
To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash
Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/
For more examples and ideas, visit:
 https://docs.docker.com/get-started/
docker@docker:~/Desktop/
```

That was easy and smooth, right? So "_Now we have docker installed, what next?_" In the next article, I'm going to be taking through some of the basic docker commands that will get you started. Until then, thanks for reading and following through. If you have any question about the same, feel free to ping me on twitter [@oste_ke](https://twitter.com/oste_ke)
