# Day 1 : containers orchestration

## Resources

+ Packt book "Docker orchestration" <https://www.packtpub.com/eu/virtualization-and-cloud/docker-orchestration>
+ Source code of the book : <https://gitlab.com/perlstalker/docker-orchestration-examples>

## A quick introduction to docker

### Overview

<https://docs.docker.com/engine/docker-overview/>

### Main components

| Name | Description |
|------|---------------|
| Swan |  |
| Compose |  |
| Machine |   |
| Registry |   |
| Hub | |
| Networking |  |
| Service discovery | |

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

## Docker examples

### Get started labs

+ Test the lab <https://docs.docker.com/get-started/>

### Networking

+ Do the 4 tutorials about networking <https://docs.docker.com/network/>

### Managing application

+ Do the tutorial about nodejs application <https://github.com/docker/labs/tree/master/developer-tools/nodejs/porting/>

## Docker deployment

### Deploying on an existing VM with docker-machine



### Deploying on virtual box with docker-machine

Installing using virtual box driver <https://docs.docker.com/machine/drivers/virtualbox/>

### Deploying on AWS with docker-machine

### Orchestrating containers with docker-compose

### Managing clusters with swarm

## Introducing Rancher OS

<https://rancher.com/docs/os/v1.x/en/quick-start-guide/>

Downloading rancher-os ISO for VMWare : <https://releases.rancher.com/os/latest/vmware/rancheros.iso>

## Running docker containers on rancher OS


## Rancher

### Introducing Rancher and Kubernetes

### Installing rancher

### Deploying Rancher Server

### Deploying workload


