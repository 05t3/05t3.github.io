---

title: "Installing Portainer on Docker"
date: 2022-02-04 01:09:33 +0300
author: oste
image: /assets/img/Posts/Portainer.jpg
categories: [HOMELAB]
tags:
  [raspberry pi, docker, ubuntu, portainer, docker containers, docker images]
---

# What is Portainer?

Portainer is an open source toolset that allows you to easily build and manage containers in Docker, Docker Swarm, Kubernetes and Azure ACI. Portainer hides the complexity of managing containers behind an easy-to-use UI. By removing the need to use the CLI, write YAML or understand manifests, Portainer makes deploying apps and troubleshooting problems so easy that anyone can do it.

Portainer is available in two models:

1. Portainer Community Edition (CE)
2. Portainer Business Edition (BE)

For the sake of this walkthrough, i will be using the CE which is free, open source and straightforward to install.There are three ways you can install the portainer server:

1. Docker Standalone
2. Docker swarm
3. Kubernetes cluster

To get started, you will need:

- The latest version of Docker installed and working
- sudo access on the machine that will host your Portainer Server instance

With that said, lets get started.

# Deployment

First, create the volume that Portainer Server will use to store its database. _The Portainer Server requires persistent storage in order to maintain the database and configuration information it needs to function._

`docker volume create portainer_data`

```bash
root@oste::/home/ubuntu/containers# docker volume create portainer_data
portainer_data
root@oste:/home/ubuntu/containers# docker volume ls
DRIVER    VOLUME NAME
local     portainer_data
```

Download and install the Portainer Server container:

```docker
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    cr.portainer.io/portainer/portainer-ce:2.9.3
```

Lets first understand the command above.

- `-d` We are running the container in the background in a “detached” mode
- `-p 8000:8000 -p 9443:9443` We are exposing ports 8000 & 9443 on the host machine( Your Pi or Linux Server)
- `--restart=always` We want to restart the container regardless of the exit status. When you specify always, the Docker daemon will try to restart the container indefinitely. The container will also always start on daemon startup, regardless of the current state of the container. \_For more information on the restart policy, feel free to read through the [official documentation](https://docs.docker.com/engine/reference/run/#restart-policies---restart)
- `--name portainer` Here, we are basically naming the container to be created for easier identification. If you do not assign a container name with the --name option, then the daemon generates a random string name for you, eg “evil_ptolemy”
- `cr.portainer.io/portainer/portainer-ce:2.9.3` This is the docker image we want to pull from the docker registry followed by the tag. Image and tags are separated by `:`

After running the command above, We can check if its actually up and running using the `docker ps` command

![image](https://user-images.githubusercontent.com/58165365/151363901-89e3bd47-98de-4ec5-ac58-144c8e24dcea.png)

# Logging In

Now that the installation is complete, you can log into your Portainer Server instance by opening a web browser and going to:

`https://IP:9443`

or

`https://FQDN:9443`

You will be presented with the initial setup page for Portainer Server.

> NB: By default, Portainer generates and uses a self-signed SSL certificate to secure port `9443`. Alternatively you can provide your own SSL certificate during installation or via the UI after installation has completed.

![image](https://user-images.githubusercontent.com/58165365/151365027-093467a6-3ad8-44a1-a6e9-4026d8cfe207.png)

After creating an account, we are welcomed with the dashboard

![image](https://user-images.githubusercontent.com/58165365/151365275-a349734c-e9c7-4671-9037-86a240d6dede.png)

# Using Portainer

Lets start with the Home tab

![image](https://user-images.githubusercontent.com/58165365/151367173-7a699a71-93c6-4530-b239-9bc9cca67d28.png)

This page provides an overview of your environments along with vital statistics about each. To manage an environment, click to select it.

![image](https://user-images.githubusercontent.com/58165365/151367513-c28ed62d-383d-4d07-a0b0-ebb220947838.png)

## Environment info

This section shows the environment name, its URL and port along with any tags. You can also see the number of CPU cores (and their available memory), the Docker version, and whether or not the Portainer Agent is installed.

## Summary tiles

This dashboard is made up of tiles showing the number of stacks, containers (including health and running-status metrics), images (and how much disk space they consume), volumes and networks.

---

If you are used to the CLI method of managing docker, let me give you a quick overview and hopefully get you hooked to portainer.🙂

### Stacks

A stack is a collection of services, usually related to one application or usage. For example, a WordPress stack definition may include a web server container (such as Apache) and a database container (such as MySQL).

![image](https://user-images.githubusercontent.com/58165365/151372265-50cab23a-02df-4b02-9064-0ddb7a1863c5.png)

_As at now, i dont have any stack deployed yet, but in the subsequent blogs, i will show you how deploy stuff as a stack._

### Images

Images are what is used to build containers. The Images section in Portainer lets you interact with the images in an environment.

![image](https://user-images.githubusercontent.com/58165365/151378897-f71dc655-a560-4a77-9c89-33ff06536131.png)

You can pull images from Docker Hub or any other registry as show below. You just need to type the image and tag name. If unsure of what tag you want to use, you can click the search button and you will be redirected to the docker registry where you can locate a specific image

![image](https://user-images.githubusercontent.com/58165365/151379934-19e77dea-0c21-4c89-95c1-05a0ecbce1f0.png)

You can also view a list of the images that are currently available in an environment, including their IDs, usage states, tags, sizes and creation dates.

![image](https://user-images.githubusercontent.com/58165365/151380220-5337188f-0649-4289-8e85-847af452541a.png)

There are many other options available like importing, exporting, Removing or building an image.

### Networks

Portainer lets you add, remove and manage networks in your environment.

![image](https://user-images.githubusercontent.com/58165365/151388562-69e651bc-ed9d-421c-8738-fd7c14e3b7c8.png)

### Volumes

A volume is a data storage area that can be mounted into a container to provide persistent storage. Unlike bind mounts, volumes are independent of the underlying OS and are fully managed by the Docker Engine.

In Portainer you can view a list of the volumes on your environment, add new volumes and remove existing volumes.

![image](https://user-images.githubusercontent.com/58165365/151391807-8842c359-70b1-4f01-ab12-7efb95dffa4a.png)

### Containers

Put simply, a container is a runnable instance of an image. Containers do not hold any persistent data and therefore can be destroyed and recreated as needed.

![image](https://user-images.githubusercontent.com/58165365/151392182-0e4e2c52-b74e-42fd-88e2-4dd6a68e1222.png)

Once a container has been created you can inspect it, edit or duplicate it, attach volumes, view logs and statistics, edit ownership, and access its console. For instance, lets inpect out portainer container.

![image](https://user-images.githubusercontent.com/58165365/151392963-f59db572-0fe6-4bef-894b-76a063df1feb.png)
![image](https://user-images.githubusercontent.com/58165365/151393145-69b393bc-628a-4796-a821-5c8978c80974.png)
![image](https://user-images.githubusercontent.com/58165365/151393406-29e56d65-0911-4826-a8c7-0977d7147536.png)

Awesome, right? 💁🏼‍♂️

---

You can do lots of stuff with portainer that i cannot exhaust on one blog😅. Consider this an intro that gives you a general overview of how it works or how it can be used. I'm working on Part 2 of this blog where i go into depth.

Reason why i decided to start with portainer as my first project is because as we progress, we will deploy lots of containers. You will definately want a good way to monitor what containers are running, resource usage etc. This is where portainer comes in handy. Well...someone will ask, why not use the CLI? 🤷🏼‍♂️ I'd say it all boils down to your personal preference. If you are comfortable working around the CLI 👍🏼

So what next? 🤔. On my next blog, i will show you how we can setup a dashboard that will contain links to the various apps we will deploy. I mean, who wants to remember the port of each and every app we'll deploy 🤷🏼‍♂️ This is where a dashboard comes in handy. In this case, i will be using Homer, super easy to use and modify.

This brings me to the end of my blog, thanks for coming this far. Feel free to share i you found it useful. You can also buzz me on twitter if you have any questions or recommendations. Stay safe ✌🏼
