# Ansible, day 1

## Recommended resources

+ Ansible online reference guide : https://docs.ansible.com/
+ Source code of Packt ansible course : https://github.com/spurin/masteringansible

The Packt source code comes with a video course : https://subscription.packtpub.com/video/virtualization-and-cloud/9781788629515

## Course track

+ Introduction
+ Installation
+ Quick start
+ Configuration
+ Inventory
+ Modules
+ Playbook
+ Managing playbooks
+ Advanced topics

## Introduction

Watch the ansible introduction video : https://www.ansible.com/resources/videos/quick-start-video

## Prerequisites

VM images in a virtualization environment (virtual box, vmware) :

+ Ubuntu
+ Centos

## Installation

Many options :

+ Package included in distributions (centos, ubuntu) : apt, yum
+ Adding the official ansible repository to your apt source list
+ Installing from source (github)
+ Installing in a default python environment using pip
+ Installing in a python virtual environment (pyenv, miniconda, ...) using pip

### Procedure

1. Install Ubuntu (VM)
2. Stop Ubuntu instance
3. Take a snapshot
4. Install ansible using apt

We install following Ansible official installation guide for Ubuntu : https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-node. The complete reference documentation is here : https://docs.ansible.com/ansible/latest/installation_guide/index.html

### SSH authentication

Reference documentation : https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server

## Getting started

Tutorial : https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html

### Adding a local machine to the inventory

1. Browse /etc/ansible directory on the controler
2. Edit hosts file (sudo)
3. Go to the end of the file
4. Add line *localhost ansible_connection=local*
5. Save and quit
6. Type the shell command in the controller : *ansible all -m ping*

### Accessing a remote machine *ubuntu1*

To get a machine ansible ready, see : https://github.com/isitix/isitix.github.io/blob/master/_posts/2018-06-19-debian-ansible-ready.markdown

#### Setting a new VM

+ Clone your ubuntu VM (snapshot before ansible)
+ Edit /etc/hosts and /etc/hostname to modify hostname 

#### 1) Configure a fixed Ip address
To configure a fixed ip address in Ubuntu, see : https://www.configserverfirewall.com/ubuntu-linux/configure-ubuntu-server-static-ip-address/

+ Go to /etc/netplan
+ Edit 50-cloud-init.yaml file

Add the following lines to the yaml file : 

```yaml
network:
    ethernets:
        ens33:
            addresses:
                - 192.168.126.10/24
            gateway4: 192.168.126.1
            nameservers:
                addresses: [8.8.8.8, 4.4.4.4]
    version: 2
```

Then goto a shell on the controller and type :
```shell
sudo netplan apply
ip add
```

#### 2) Add a ssh key pair to the controler and export it t

```bash
controler$ssh-keygen
controler$ssh-copy-id ansible@192.168.126.10
```

#### 3) Test the connection to ubuntu1

```bash
controler$ssh -i id_rsa ansible@192.168.126.10
```

#### 4) Add ubuntu1 to the inventory

On controller :

+ Edit /etc/ansible/hosts
+ Add line *ubuntu1* at the end of the file
+ Edit /etc/hosts
+ Add line *192.168.126.10 ubuntu1*

Test the connection from the controller to all ansible stations :

```bash
controler$ansible all -m ping
```

#### 5) Add ansible user to sudo without password

On controler and on ubuntu1 :

```bash
$sudo visudo -f /etc/sudoers.d/ansible
ansible ALL=NOPASSWD: ALL
CTRL+S
CTRL+X
$sudo chmod 400 /etc/sudoers.d/ansible
```

Type Ctl + O to save the file to /etc/sudoers.d/ansible

#### 6) Clone ubuntu1 to get ubuntu2 and ubuntu3

Network addresses :

| hostname | address |
|-----|---------|
| controler | 192.168.126.10 |
| ubuntu1 | 192.168.126.11 |
| ubuntu2 | 192.168.126.12 |
| ubuntu3 | 192.168.126.13 |

## Local configuration

Move the configuration file from /etc/ansible to your home directory (/home/ansible) :

```bash
controller$cp /etc/ansible/ansible.cfg /home/ansible/.ansible.cfg
controller$cd ~
controller$mkdir deployment
controller$cp /etc/ansible/hosts /home/ansible/deployment/
controller$vim .ansible.cfg
inventory      = /home/ansible/deployment/hosts
:wq
```

## Static inventories

+ Tutorial : https://github.com/spurin/masteringansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/01%20-%20Ansible%20Inventories
+ Documentation : https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html

### In a nutshell

In an inventory :

+ you can assign a machine to a group
+ you can design a hierarchy of groups (parent groups and children groups) with many levels
+ you can proceed any operation (facts gathering, vars setting, task, trigger) on a specific group
+ you can define a range based on machine names
+ you can apply variables (ssh port, ansible user, become password, ...) to every machines or a group of machines

Static inventory file format may be :

+ yaml
+ json
+ text

### Hands on

+ Add a new ubuntu server, ubuntu4 (192.168.126.14)
+ Delete sudoer no password configuration on ubuntu4
+ Add ubuntu4 to the inventory
+ Define the following group hierarchy :

| Group name | Parent group | Hosts |
|------|------|------------|
| production | application | ubuntu1, ubuntu2 |
| test | application | ubuntu3, ubuntu4 |
| application | NA | ubuntu 1 to 4 |
| administration | NA | controller |
| ubuntu | linux | controller, ubuntu 1 to 4 |

### Yaml inventory file

```yaml
---
all:
        vars:
                become: true
application:
        children:
                production:
                        hosts:
                                ubuntu[1:2]:
                test:
                        hosts:
                                ubuntu[3:4]:
administration:
        hosts:
                localhost:
                        ansible_connection: local
linux:
        children:
                ubuntu:
                        hosts:
                                ubuntu[1:4]:
                                localhost:
...
```

To test the yaml inventory file from a controller terminal :
```bash
controller$ansible all -i hosts.yml -m ping
```

## END OF DAY 1