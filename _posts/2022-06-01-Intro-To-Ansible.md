---

title: "Intro To Ansible"
date: 2022-06-01 01:09:33 +0300
author: oste
image: /assets/img/Posts/ansible1.png
categories: [Automation, Ansible]
tags: [ansible, automation, ansible playbook]
---

Hey guys, and welcome back to this new segment of automation.

Today i'll be taking you through some fundamentals that will get you started.

# What is ansible?

Imagine a scenario where you are managing more than 50 servers, some on the cloud & some on-prem and want to do an update/upgrade Or , say you have your homelab setup with freshly installed OS and need to install a necessary tools which are'nt installed by default. In this case, you'd typically have to ssh into each and every server and manually run commands , which could be tedious. 😪 Here's where ansible comes to the rescue.

Simply put, you can think of it as an Open source DevOps tool/IT automation engine that automates the provisioning, application deployment, cofiguration management, orchestaration and many other IT processes.

Ansible has been built with a strong focus on security and reliability and relies on ssh protocol to communicate with your hosts

# Where can you install Ansible?

As mentioned, you only need to install it on one control node and then manage an entire fleet of remote machines. Ansible community package can be installed on a variety of operating systems including:

- RHEL, CentOS, or Fedora
- Ubuntu
- Debian
- Gentoo with portage
- FreeBSD
- MacOS
- Solaris
- Arch Linux
- Slackware Linux
- Clear Linux
- Windows (WSL)

> _Before installing ansible , your control node should have Python 3.8 or newer installed._

You can install ansible by running the following commands:

On CentOS:

```
$ sudo yum install epel-release
$ sudo yum install ansible
```

On Ubuntu:

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

At the time of writing this, i am using ansible version 2.9.6

```bash
ubuntu@oste:~$ ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
ubuntu@oste:~$
```

Please note that Ansible cannot run on a Windows host natively, though it can run under the Windows Subsystem for Linux (WSL). You can read more about this on [Matt Davis’s blog post](https://blog.rolpdog.com/2020/03/why-no-ansible-controller-for-windows.html).

# Terminologies

- `Control Node` - _Any machine with Ansible installed and configured to connect and execute commands on hosts._
- `Managed Nodes/hosts` - _The network devices (and/or servers) you manage with Ansible. Managed nodes are also sometimes called “hosts”._
- `Inventory/hostfile` - _A list of managed nodes which specifies like IP address or FQDNs for each managed node. It is typically located at `/etc/ansible/hosts`._
- `Modules` - _The units of code Ansible executes. You can invoke a single module with a task, or invoke several different modules in a playbook._
- `Tasks` - _The units of action in Ansible._
- `Playbooks` - _Ordered lists of tasks, saved so you can run those tasks in that order repeatedly. Playbooks are written in YAML and are easy to read, write, share and understand._
- `Collections` - _Collections are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins._
- `Role` - _a collection of playbooks and other files that are relevant to a goal such as installing a web server._
- `Play` - _a full Ansible run. A play can have several playbooks and roles, included from a single playbook that acts as entry point._

At this stage, you have an idea of what ansible is and what it can do. If you still hanging in there and not yet acquinted with concepts discussed above, i got you covered. Check out my next segment which is more hands on.
