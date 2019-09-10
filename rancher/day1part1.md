# Day 1 : containers orchestration / part 1, docker

## Resources

+ Packt book "Docker orchestration" <https://www.packtpub.com/eu/virtualization-and-cloud/docker-orchestration>
+ Source code of the book : <https://gitlab.com/perlstalker/docker-orchestration-examples>

## A quick introduction to docker

### Overview

<https://docs.docker.com/engine/docker-overview/>

### Main components

| Name | Description |
|------|---------------|
| Swarm | Cluser management |
| Compose | Orchestration |
| Machine | Docker hosts management |
| Registry | Docker images publication |
| Hub | Public docker images publication |
| Networking | Network between docker containers |
| Service discovery | Service access between containers |

## Docker installation

### IP adresses

VMWare Workstation subnet : 192.168.126.0/24
DHCP : .128 to .254
Gateway : .2

| hostname | address |
|-----|---------|
| controller | 192.168.126.10 |
| ubuntu1 | 192.168.126.11 |
| ubuntu2 | 192.168.126.12 |
| ubuntu3 | 192.168.126.13 |
| devstack | 192.168.126.17 |
| centos1 | 192.168.126.33 |
| centos2 | 192.168.126.34 |
| centos3 | 192.168.126.35 |
| packstack | 192.168.126.65 |
| docker1 | 192.168.126.97 |

### General procedure

+ Clone the ansible ready VM
+ Give it a fixed IP address
+ Restart your ansible controller machine
+ Add the new VM to the inventory
+ Implement a playbook to install docker

### Add the new VM to the inventory

```bash
# hosts
[control]
controller ansible_connection=local

[docker]
docker1

[all:vars]
ansible_become=true
ansible_user=ansible
```

### Apply a linux configuration (hostname, ...)

We define ansible roles to manage the common configuration tasks. See <https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html>.

```yml
# roles/linux/tasks/main.yml
- name: modify hostname
  hostname:
    name: "{{inventory_hostname}}"

- name: load /etc/hosts
  template:
    src: hosts.j2
    dest: /etc/hosts

- name: apt update, upgrade, autoclean, autoremove, purge
  apt:
    autoclean: yes
    autoremove: yes
    purge: yes
    update_cache: yes
    cache_valid_time: 90000
    upgrade: yes
```

### Install docker

Configure installation sources :

```yml
# roles/docker/tasks/main.yml

# Configure apt
- name: check if docker gpg key already exists
  stat:
    path: "{{working_directory}}/docker.gpg"
  register: docker_key

- name: download docker gpg key
  get_url:
    url: https://download.docker.com/linux/ubuntu/gpg
    dest: "{{working_directory}}/docker.gpg"
  when: docker_key.stat.exists == False

- name: add docker to apt key ring
  apt_key:
    id: 0EBFCD88
    file: "{{working_directory}}/docker.gpg"
    state: present

- name: identify docker version
  shell: lsb_release -cs
  register: ubuntu_version

- name: add docker repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ubuntu_version.stdout}} stable"
    state: present

- name: update repository
  apt:
    update_cache: yes
    cache_valid_time: 90000
```

Install docker-CE and docker-compose :

```yml
# Install docker
- name: install docker CE
  apt:
    name: docker-ce
    state: latest

- name: install docker-compose
  apt:
    name: docker-compose
    state: latest
```

Install docker-machine :

```yml
# Install docker-machine
# See : https://docs.docker.com/machine/install-machine/
- name : set docker-machine os_family  variable
  shell: uname -s
  register: os_family

- name: check docker-machine installation variables
  debug:
    msg: "OS family : {{os_family.stdout}} Architecture: {{ansible_architecture}}"

- name: install docker machine
  get_url:
    url: "https://github.com/docker/machine/releases/download/v0.16.0/docker-machine-{{os_family.stdout}}-{{ansible_architecture}}"
    validate_certs: no
    dest: /usr/local/bin/docker-machine
    owner: root
    group: root
    mode: '0755'

```

Configure docker group:

Read this warning <https://docs.docker.com/install/linux/linux-postinstall/> before you process

```yml
# Configure docker
- name: Ensure group docker exists
  group:
    name: docker
    state: present

- name: add ansible user to docker group
  user:
    name: ansible
    state: present
    append: True
    groups: docker
```

### Link roles to docker1

```yml
# docker.yml
---
- hosts: docker
  vars:
    working_directory: /home/ansible
  roles:
    - linux
    - docker
...
```

## Docker getting started lab

+ Test the lab <https://docs.docker.com/get-started/>

## Deploying docker on an existing VM named docker2 with docker-machine

1) Clone the VM Ubuntu standard
2) Give it a static IP, add ssh key both from ansible controller and from docker1
3) Add it to your ansible inventory, and your ansible playbook role = linux
4) Deploy docker to docker2 using docker-machine

### Static ip

| hostname | address |
|-----|---------|
| docker2 | 192.168.126.98 |

### Docker2 ansible configuration

```yml
# docker.yml
---
- hosts: docker2, controller
  roles:
    - linux
- hosts: docker
  vars:
    working_directory: /home/ansible
  roles:
    - linux
    - docker
...
```

### Deploying docker on docker2 using docker-machine

Log into docker1 and execute the shell command below:

```bash
ansible@docker1:~$ docker-machine create --driver generic --generic-ip-address 192.168.126.98 --generic-ssh-key ~/.ssh/id_rsa --generic-ssh-user ansible docker2
Creating CA: /home/ansible/.docker/machine/certs/ca.pem
Creating client certificate: /home/ansible/.docker/machine/certs/cert.pem
Running pre-create checks...
Creating machine...
(docker2) Importing SSH key...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with ubuntu(systemd)...
Installing Docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env docker2
ansible@docker1:~$
```

Check that your new docker node is up :

```bash
ansible@docker1:~$ docker-machine ls
NAME      ACTIVE   DRIVER    STATE     URL                         SWARM   DOCKER     ERRORS
docker2   -        generic   Running   tcp://192.168.126.98:2376           v19.03.2
```

### Testing your new docker host docker2

Ssh into docker2 :

```bash
ansible@docker1:~$ docker-machine ssh docker2
Welcome to Ubuntu 19.04 (GNU/Linux 5.0.0-27-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Sep  9 13:40:43 UTC 2019

  System load:  0.02               Processes:              210
  Usage of /:   27.0% of 19.56GB   Users logged in:        0
  Memory usage: 19%                IP address for ens33:   192.168.126.98
  Swap usage:   0%                 IP address for docker0: 172.17.0.1

 * Congrats to the Kubernetes community on 1.16 beta 1! Now available
   in MicroK8s for evaluation and testing, with upgrades to RC and GA

     snap info microk8s

0 updates can be installed immediately.
0 of these updates are security updates.


Last login: Mon Sep  9 13:32:11 2019 from 192.168.126.97
```

Launch a container on docker2 : (basic example <https://www.macadamian.com/learn/docker-machine-basic-examples/>)

Run a terminal on docker1 :

```bash
ansible@docker1$ eval $(docker-machine env docker2)
ansible@docker1:~$ docker run -it alpine  /bin/sh
```

Run another terminal on docker1

```bash
# Running docker shell on docker 1
ansible@docker1:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                      NAMES
7c3083acc301        rancher/rancher     "entrypoint.sh"     3 days ago          Up About an hour    0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   beautiful_hypatia
# Switching to docker2
ansible@docker1:~$ eval $(docker-machine env docker2)
# Running docker ps on docker 2
ansible@docker1:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
f3c7da3e30fa        alpine              "/bin/sh"           About a minute ago   Up About a minute                       vigilant_lederberg
# Switching back to docker1...
ansible@docker1:~$ eval $(docker-machine env -u)
ansible@docker1:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                      NAMES
7c3083acc301        rancher/rancher     "entrypoint.sh"     3 days ago          Up About an hour    0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   beautiful_hypatia
```

## Deploying on virtual box with docker-machine

Self-study with the documentation about "installing docker with docker-machine using virtual box driver" : <https://docs.docker.com/machine/drivers/virtualbox/>

## Deploying on AWS with docker-machine

Self-study...

## Networking

+ Do the 4 tutorials about networking <https://docs.docker.com/network/>

## Managing application

+ Do the tutorial about nodejs application <https://github.com/docker/labs/tree/master/developer-tools/nodejs/porting/>

## Orchestrating containers with docker-compose

Test this example: <https://github.com/PacktPublishing/Docker-Orchestration/tree/master/Chapter02>

## Managing clusters with swarm

Follow this tutorial <https://docs.docker.com/engine/swarm/swarm-tutorial/>

## Service discovery

Read this article : <https://blog.octo.com/en/how-does-it-work-docker-part-3-load-balancing-service-discovery-and-security/>
