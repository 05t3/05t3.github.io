---

title: "Lab Setup on Raspberry Pi / VM"
date: 2022-02-04 01:09:33 +0300
author: oste
image: /assets/img/Posts/ubunturasp.jpg
categories: [HOMELAB]
tags:
  [raspberry pi, docker, ubuntu, portainer, docker containers, docker images]
---

# Initial Ubuntu Server Setup

Hello and welcome to this segment where i will be taking you through some of the best self hosted applications you can run in your home lab on your RaspberryPi or an Ubuntu Server. In this case i will be using:

- Raspberry Pi 4B
- [Ubuntu 20.04.3 for Raspberry Pi](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04.3&architecture=server-arm64+raspi)
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/).
  Alternatively, you can use : [balenaEtcher](https://www.balena.io/etcher/) or [Rufus](https://rufus.ie/en/)
- [PuTTY](https://www.putty.org/), (I love [MobaXterm Home Edition](https://mobaxterm.mobatek.net/download.html))
- Sandisk Ultra 64GB Micro SDXC Memory Card Class 10 80mb/S
- Ethernet Cable

If you choose to vitualize your Ubuntu server on VirtualBox or Vmware:

- [Download Ubuntu Server 20.04 LTS](https://ubuntu.com/download/server)
- [VirtualBox 6.1.32](https://www.virtualbox.org/wiki/Downloads) / [VMware Workstation Player](https://www.vmware.com/products/workstation-player.html)
- Follow this awesome walkthrough by [Ubuntu](https://docs.github.com/en/get-started/quickstart/hello-world) to get started. Once done, feel free to jump to this [section](https://05t3.github.io/posts/Raspberry-Pi-Setup/#:~:text=the%20new%20password.-,Configuring%20Static%20IP,-Next%20thing%20we).

Lets get started... 😎🤏🏼

It should take a minute to download the iso file if you are using fast internet

![image](https://user-images.githubusercontent.com/58165365/151159687-2f92ba49-c420-49b7-9e09-67df330198d3.png)

Download and install the pi imager, launch it and follow this steps.

1. Plug your memory card to a mem card reader
2. Choose OS

![image](https://user-images.githubusercontent.com/58165365/151137360-f4c3b0bb-7834-472e-9ddc-2161c6192050.png)

3. Select "Use Custom" and choose the .iso file you downloaded.

![image](https://user-images.githubusercontent.com/58165365/151137546-89aed5db-f388-46b2-8418-480af5c971bf.png)

4. Choose Storage and select the Memory card

![image](https://user-images.githubusercontent.com/58165365/151137949-abd90583-e449-4673-a0a8-a019441e341e.png)

5. Select **Write** and grab yourself a cup of coffee before the installation completes. ☕

![image](https://user-images.githubusercontent.com/58165365/151138262-9a4632a5-f350-474b-9263-15601c1c52e6.png)

In less than 2 minutes, the process should be done.

![image](https://user-images.githubusercontent.com/58165365/151139116-daa46835-7130-4d4f-ac79-3c6e6ee29602.png)

## Configuring Wireless Adapter (Method 1)

Close the raspberry pi imager and go to the root of the memory card on the file explorer.

Locate a file called `network-config` and open it with any text editor. It should look something of the sort.

![image](https://user-images.githubusercontent.com/58165365/152559517-d3fc87a2-fb9f-4a69-b5d6-05a32d559365.png)

![image](https://user-images.githubusercontent.com/58165365/152559303-a532acb4-1183-46f0-afca-0aa5f49200c3.png)

For now, i will show you how to configure your wireless adapter if you are planning on using it.

1. Uncomment the wifi section as shown below.
2. Modify the highlighted section as shown. Add your Home Router's name and Password.

> NB: They both need to be inside `" "`

![image](https://user-images.githubusercontent.com/58165365/152560081-d0529338-64ed-41b6-9bcb-88d342927767.png)

3. You can delete the other section or let it remain as comments. I.e

![image](https://user-images.githubusercontent.com/58165365/152561054-7c1e1806-828b-4258-a37a-5e5ddab6e51f.png)

With that done, we should be able to use Wi-Fi without needing to connect an ethernet cable. But we also need to know our username. In this case, the ISO file i downloaded is already pre-configured with a user called `ubuntu`. How do i know that?🤔 Still in the root directory of the memory card, locate `user-data` file. you need to ensure this two options are set to `true`.

![image](https://user-images.githubusercontent.com/58165365/152562847-2bcbab4a-ca10-44ee-b3d7-7e3e67e7fa3a.png)

![image](https://user-images.githubusercontent.com/58165365/152562489-21478973-41c5-405d-96a2-82ba47606808.png)

As you can see, we have a user called ubuntu and we should change the password on first boot as the default is set to `ubuntu`. `ssh_pwauth: true` simply means that we can be able to authenticate using passwords in ssh sessions.

Eject the SD card and plug it in your Raspberry PI and start it. Since we are doing a `headless configuration`, we are going to ssh into the server using MobaXterm . But wait, we need an Ip Address for this server. You can log into your home router and get the IP address or you can do a quick nmap scan on your network to determine what IP's have been assigned to your Pi. In my case, i got `192.168.1.19` & `192.168.1.20`

> A headless server is a computer without a monitor, keyboard, mouse, or other peripherals.Headless computers are normally controlled over the network.

![image](https://user-images.githubusercontent.com/58165365/151242292-59ed3bef-80ac-4dd5-aeb3-38711d6f61b4.png)

You will get a prompt, asking you to change your password.

![image](https://user-images.githubusercontent.com/58165365/151242665-a82b872e-4a4d-4f57-8d6f-00e41876f218.png)

Once changed, we can log back in with the new password.

![image](https://user-images.githubusercontent.com/58165365/151243366-0c20bbc8-0b91-4079-8f13-506a84427849.png)

## Configuring Static IP

Next thing we need to do is configure a static IP address on interface `eth0`. Since this server uses netplan as its default network management tool, we can get the configuration file in `/etc/netplan`.

> It is advisable to create a backup copy of the configuration file. Incase you make any mistakes, you can always role back to your backup file.

> In my case, the config file is named `50-cloud-init.yaml`. If you are following along on a VM, you are likely to get your config file named `00-installer-config.yaml`. Simply modify the ethernets section and ignore Wifi.

> Please note: if you are using a linux server on VM, your interface name may not be `eth0`, most likely `enp0s3`. take note and work with that.

> We could have configured the ethernet settings earlier in the `network-config` file but i wanted to show you two methods which you can use.

![image](https://user-images.githubusercontent.com/58165365/151323667-8abf2464-6d17-4609-a273-f3b013fefc5b.png)

Modify the config file with the following lines as is. I've made it easier for you to avoid indentation issues. 😉

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
  wifis:
    wlan0:
      access-points:
        "OSTE VLAN":
          password: "@oste1234"
      dhcp4: true
      optional: true
```

You then need to run the following command for changes to apply.

`sudo netplan apply`

This command parses and applies the configuration to the system. Configuration written to disk under `/etc/netplan/` will persist between reboots.

_If you want to know more about netplan, feel free to read their [official documentation](https://netplan.io/examples/)_

If we now check our IP address, `eth0` has a static IP of 192.168.1.100

![image](https://user-images.githubusercontent.com/58165365/151326996-8a343ae9-cd58-4f8d-bf1b-cd96822999d1.png)

## Configuring Hostname

To check your hostname, simply run `hostname`. To modify the existing hostname, edit `/etc/hostname` and reboot the system for changes to take place.

```bash
root@ubuntu:/etc/netplan# hostname
ubuntu
root@ubuntu:/etc/netplan# nano /etc/hostname

//reboot system

root@oste:/home/ubuntu# hostname
oste.com
```

We then need to add this hostname to the hosts file located in `/etc/hosts`

```bash
root@ubuntu:/etc/netplan# cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 ubuntu
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
root@ubuntu:/etc/netplan# nano /etc/hosts
root@ubuntu:/etc/netplan# cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 oste.com
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

One final step is to add our server's IP and hostname to our windws hosts file which is located in `C:\Windows\System32\drivers\etc`. You need administrative privilege to modify the hostfile.

![image](https://user-images.githubusercontent.com/58165365/151255279-d31e9191-c70a-4d8a-a48a-86673ab3c863.png)

## Installing Docker

In my Docker segment, '[Docker Installation](https://05t3.github.io/posts/Docker-Installation/)' , i've covered how to install docker step-by-step. Feel free to check it out.

---

Congrats on reaching this far, we have successfully prepared our homelab envirnment. 🥳 Hope this article helped you in one way or another. So what next? 🤔 We have docker installed, right? I am going to take you through the first app/solution that we'll be using to manage, create , monitor and deploy our apps, this is [Portainer](https://05t3.github.io/posts/Portainer/).

Stay tuned and thanks for following till this point. Keep safe and share the article if you found it helpful. 😁
