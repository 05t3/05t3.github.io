---

title: "Docker Commands"
date: 2021-12-2 01:09:33 +0300
author: oste
image: /assets/img/Posts/docker.png
categories: [DOCKER]
tags: [docker, docker containers, ubuntu, docker images]
---

Hey there and welcome back. This is a continuation of my previous article, where we will look at some of the basic docker commands that should get you started and use docker with ease.But first, lets define a few terminologies that we will mention from time to time:

- **Docker registries**: _A Docker registry stores Docker images. [Docker Hub](https://hub.docker.com/) is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default._
- **Images**: _An image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization. For example, you may build an image which is based on the ubuntu image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run._
- **Containers**: _A container is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state._
- **Docker daemon**: _The Docker daemon (dockerd) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes._

Lets get started, shall we 💁🏾‍♂️

Before anything else, to display the docker help options either run `docker` with no parameters or execute `docker help`

```bash
docker
```

To check the status, start or stop the docker daemon:

```bash
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl stop docker
```

To automatically start Docker and Containerd on boot, use the commands below:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

To list all your images

```bash
docker images
```

List currently running containers

```bash
docker ps
```

List all containers and commands run previously

```bash
docker ps -a
```

To view system-wide information about Docker

```bash
docker info
```

To List networks available.

```bash
docker network ls
```

By default, all containers run on bridged network (172.17.0.0/16)

To inspect a specific network

```bash
docker network inspect bridge
```

If you want to pull an image from the docker registry, simply run `docker pull <image_name>`

Lets pull our first image from docker registry since we don't have it locally

```bash
docker pull ubuntu
```

We can then run it using the command below:

```bash
docker run -it ubuntu:latest /bin/bash
exit
```

Lets first explain what the command above does.

When you run this command, it runs an ubuntu container, attaches interactively to your local command-line session, and runs /bin/bash. Instead of concatenating `-it` , you can also write it as `-i -t`.

1. Docker creates a new container, as though you had run a `docker container create` command manually.
2. Docker allocates a read-write filesystem to the container, as its final layer. This allows a running container to create or modify files and directories in its local filesystem.
3. Docker creates a network interface to connect the container to the default network, since we did not specify any networking options. This includes assigning an IP address to the container. By default, containers can connect to external networks using the host machine’s network connection.
4. Docker starts the container and executes `/bin/bash`. Because the container is running interactively and attached to your terminal (due to the `i` and `t` flags), you can provide input using your keyboard while the output is logged to your terminal.
5. When you type exit to terminate the /bin/bash command, the container stops but is not removed. You can start it again or remove it.

Say you want to search for a specific image on the docker registry, you can do so directly on [docker hub](https://hub.docker.com/), or the command line as shown below:

```bash
docker search <image_name>
```

For example:

```bash
docker@docker:~$ docker search centos
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                            The official build of CentOS.                   6906      [OK]
ansible/centos7-ansible           Ansible on Centos7                              135                  [OK]
consol/centos-xfce-vnc            Centos container with "headless" VNC session…   131                  [OK]
jdeathe/centos-ssh                OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   121                  [OK]
centos/systemd                    systemd enabled base container.                 105                  [OK]
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   92
imagine10255/centos6-lnmp-php56   centos6-lnmp-php56                              58                   [OK]
tutum/centos                      Simple CentOS docker image with SSH access      48
centos/postgresql-96-centos7      PostgreSQL is an advanced Object-Relational …   45
jdeathe/centos-ssh-apache-php     Apache PHP - CentOS.                            31                   [OK]
kinogmt/centos-ssh                CentOS with SSH                                 29                   [OK]
guyton/centos6                    From official centos6 container with full up…   10                   [OK]
nathonfowlie/centos-jre           Latest CentOS image with the JRE pre-install…   8                    [OK]
centos/tools                      Docker image that has systems administration…   7                    [OK]
drecom/centos-ruby                centos ruby                                     6                    [OK]
darksheer/centos                  Base Centos Image -- Updated hourly             3                    [OK]
mamohr/centos-java                Oracle Java 8 Docker image based on Centos 7    3                    [OK]
amd64/centos                      The official build of CentOS.                   2
miko2u/centos6                    CentOS6 日本語環境                                   2                    [OK]
dokken/centos-7                   CentOS 7 image for kitchen-dokken               2
blacklabelops/centos              CentOS Base Image! Built and Updates Daily!     1                    [OK]
mcnaughton/centos-base            centos base image                               1                    [OK]
myheritage/centos7-git-java       CentOs based docker image for Jenkins slave     1                    [OK]
starlabio/centos-native-build     Our CentOS image for native builds              0                    [OK]
smartentry/centos                 centos with smartentry                          0                    [OK]
```

Congrats for the far you have come. At this point, you know how to pull images,inspect networks and run containers. Lets go a step further.

By default, if you create a container without specifying what name you want to call it, docker assigns it random names. For instance in the snippet below, the names assigned were _epic_jackson_ & _tender_pike_:

```bash
docker@docker:~$ docker ps -a
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED          STATUS                      PORTS                                       NAMES
a8ff9e854a5d   ubuntu:latest                                          "/bin/bash"              7 seconds ago    Exited (0) 2 seconds ago                                                epic_jackson
24bcb9c20f67   ubuntu:latest                                          "/bin/bash"              37 seconds ago   Exited (0) 33 seconds ago                                               tender_pike

```

If you would like to give your containers meaningful names, you can do so by running your container as follows:

```bash
docker run -it --name <container_name> ubuntu:latest /bin/bash
```

If we check the name this time round, you will notice the difference.

```bash
docker@docker:~$ docker run -it --name my-application ubuntu:latest /bin/bash
root@a4d490a7347a:/# exit
exit
docker@docker:~$ docker ps -a
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED          STATUS                      PORTS                                       NAMES
a4d490a7347a   ubuntu:latest                                          "/bin/bash"              12 seconds ago   Exited (0) 7 seconds ago                                                my-application
```

You can also rename a container as follows:

```bash
docker rename <old_name> <new_name>
```

For instance:

```bash
docker@docker:~$ docker ps -a
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED          STATUS                      PORTS                                       NAMES
a8ff9e854a5d   ubuntu:latest                                          "/bin/bash"              42 minutes ago   Exited (0) 42 minutes ago                                               epic_jackson

docker@docker:~$ docker rename epic_jackson test-application

docker@docker:~$ docker ps -a
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED          STATUS                      PORTS                                       NAMES
a8ff9e854a5d   ubuntu:latest                                          "/bin/bash"              43 minutes ago   Exited (0) 43 minutes ago                                               test-application
```

Say you wanna jump back into the previously run container

```bash
docker start <container id>
# or
docker start <wierd_name>
```

The command above starts the container but to get a shell, simple used `attach`

```bash
docker attach <container id>
# or
docker attach  <wierd_name>
```

If you have a shell on a container and would like to exit without terminating it, you can simply background it as follows:

```bash
Ctrl+p , Ctrl+q
```

To remove the history of a container

```bash
docker rm <container_name>
```

To know more info about a container

```bash
docker inspect <container_name>
# or
docker container inspect <container_name>
```

To remove an image

```bash
docker rmi <image_id>
# or
docker rmi <repository_name>:<tag>
```

Say you want to delete an image and you have a running container... you'll get an error

```bash
docker rmi image
```

1. To fix the error, you need to first stop the container,

```bash
docker stop <image_id>
```

2. Remove the container

```bash
docker container rm <image_id>
```

3. Finally remove the image

```bash
docker rmi image
```

This would look something like this:

```bash

```

Say you want to save your docker image into a tarball

```bash
docker save -o xxxxx.tar ubuntu:latest
```

You can load the saved tar file in docker as follows:

```bash
docker load -i xxxxxx.tar
```

If you want to rename your images, you can do so by running

```bash
docker tag Ubuntu:latest ubuntu:v1
```
