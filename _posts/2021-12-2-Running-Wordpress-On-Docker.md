---

title: "Running Wordpress on Docker"
date: 2021-12-2 01:09:33 +0300
author: oste
image: /assets/img/Posts/docker.png
categories: [DOCKER]
tags: [docker, docker containers, ubuntu, docker images]
---

Today i'm going to be showing you how easy it is to have wordpress up and running using Docker Compose in an isolated environment built with docker containers. If you are not conversant with Docker Compose, i would suggest going through Docker's official [documentation](https://docs.docker.com/compose/gettingstarted/) of the same.

However, before we get started, you need to have `Compose` installed. If you have not yet installed it, you can do so by running the following commands

`sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

`sudo chmod +x /usr/local/bin/docker-compose`

You can test is the installation was successful by checking the version of Compose installed as shown below:

```bash
root@docker:/var/www/wordpress# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

We are then going to create a directory called **wordpress** in `/var/www/` where will add resources necessary to build our image. In this case, we need to create a file called `docker-compose.yml` . You can also use `.yaml` extension.

_See the snippet attached below_ 👇🏾

```bash
docker@docker:~/Desktop$ cd /var/www/
docker@docker:/var/www$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Onk 18 22:57 .
drwxr-xr-x 15 root root 4096 Onk 18 22:57 ..
drwxr-xr-x  2 root root 4096 Onk 18 22:57 html
docker@docker:/var/www$ sudo su
[sudo] password for docker:
root@docker:/var/www# mkdir wordpress
root@docker:/var/www# cd wordpress/
root@docker:/var/www/wordpress# nano docker-file.yml
root@docker:/var/www/wordpress# cat docker-file.yml
version: "3.9"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

You can then proceed to build the project as follows:

`docker-compose up -d`

The command above basically runs in detached mode, pulls the needed Docker Images from the docker registry since we dont have them locally on our system and starts the wordpress and database containers.

```bash
root@docker:/var/www/wordpress# docker-compose up -d
Creating network "wordpress_default" with the default driver
Creating volume "wordpress_db_data" with default driver
Creating volume "wordpress_wordpress_data" with default driver
Pulling db (mysql:5.7)...
5.7: Pulling from library/mysql
a10c77af2613: Pull complete
b76a7eb51ffd: Pull complete
258223f927e4: Pull complete
2d2c75386df9: Pull complete
63e92e4046c9: Pull complete
f5845c731544: Pull complete
bd0401123a9b: Pull complete
2724b2da64fd: Pull complete
d10a7e9e325c: Pull complete
1c5fd9c3683d: Pull complete
2e35f83a12e9: Pull complete
Digest: sha256:7a3a7b7a29e6fbff433c339fc52245435fa2c308586481f2f92ab1df239d6a29
Status: Downloaded newer image for mysql:5.7
Pulling wordpress (wordpress:latest)...
latest: Pulling from library/wordpress
eff15d958d66: Pull complete
933427dc39f7: Pull complete
35bb08dc7ee2: Pull complete
58a3f26800d7: Pull complete
bba5c0e912ac: Pull complete
62e5c0d120c5: Pull complete
410a26230a3b: Pull complete
2880852a4de4: Pull complete
ea5136838c9b: Pull complete
e11691a8fc82: Pull complete
d85eae320cca: Pull complete
7fb6fb87e009: Pull complete
f47ba3377e25: Pull complete
b768e2589a07: Pull complete
27d9df1ded76: Pull complete
be48c3fa7409: Pull complete
ccb83b81b398: Pull complete
33d5041f0578: Pull complete
d1674b6f1114: Pull complete
65c90de6fe6b: Pull complete
69fce6e644d9: Pull complete
Digest: sha256:e7b40265f5d72fc74fb73683a60f5d3de9787a6467b87b78ee742645807d8463
Status: Downloaded newer image for wordpress:latest
Creating wordpress_db_1 ... done
Creating wordpress_wordpress_1 ... done
```

If you want to confirm if the docker images were pulled successfully, simply run `docker images` command as shown below:

```bash
root@docker:/var/www/wordpress# docker images
REPOSITORY                                      TAG            IMAGE ID       CREATED         SIZE
wordpress                                       latest         e3a452c0a154   13 days ago     616MB
mysql                                           5.7            8b43c6af2ad0   2 weeks ago     448MB
```

Since we also spin up the containers in detached mode and want to confirm if they are up and running, we can check the status by running `docker ps` command:

```bash
root@docker:/var/www/wordpress# docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS          PORTS                                       NAMES
fbeb6f8b9e6f   wordpress:latest             "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes    0.0.0.0:8000->80/tcp, :::8000->80/tcp       wordpress_wordpress_1
cc8a525a6502   mysql:5.7                    "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes    3306/tcp, 33060/tcp                         wordpress_db_1

```

In order to access our wordpress site, we can access it on port 8000 since we binded it to port 80 on the 0.0.0.0.
If we fire up our browser, we can reach it on `http://machine_ip:8000` as shown in the screenshots below.

Select your prefered language and hit `**Continue**`
![image](https://user-images.githubusercontent.com/58165365/144333009-cdaded7c-2a3c-4d86-adbd-411a11c369fc.png)

Provide Site Title, Username, password and an email and start the installation process which is very fast.
![image](https://user-images.githubusercontent.com/58165365/144333071-2c940f20-06fb-4ad2-a5fb-96111a91b4f7.png)

After the installation completes, simply login using the credentials configuredin the previous step and you should have access to the dashboard. Easy, right?😃💁🏼‍♂️
![image](https://user-images.githubusercontent.com/58165365/144333115-09d905ad-f28e-45ef-976b-bf0be00964ac.png)

If you want to shutdown the wordpress site, simply run `docker-compose down` which removes the containers created , default network created but preserve the WordPress database. You can also confirm the containers are not running by running `docker ps` command.

```bash
root@docker:/var/www/wordpress# docker-compose down
Stopping wordpress_wordpress_1 ... done
Stopping wordpress_db_1        ... done
Removing wordpress_wordpress_1 ... done
Removing wordpress_db_1        ... done
Removing network wordpress_default
root@docker:/var/www/wordpress# docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED       STATUS          PORTS

root@docker:/var/www/wordpress#
```
