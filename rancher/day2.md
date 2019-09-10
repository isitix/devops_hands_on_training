# Day 2 : containers orchestration, rancher

## Introducing Rancher OS

<https://rancher.com/docs/os/v1.x/en/quick-start-guide/>

Downloading rancher-os ISO for VMWare : <https://releases.rancher.com/os/latest/vmware/rancheros.iso>

## Running docker containers on RancherOS

### Documentation

+ <https://rancher.com/rancher-os>
+ <https://rancher.com/docs/os/v1.x/en/>
+ <https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/>
+ <https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/server/install-to-disk/>
+ <https://rancher.com/docs/os/v1.x/en/installation/boot-process/>
+ <https://sdbrett.com/BrettsITBlog/2017/01/rancheros-installing-to-hard-disk/>

### Building a rancher VM from the ISO file

+ For VMWare, boot a VM on the ISO file and then follow the procedure to install to disk <https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/server/install-to-disk/>
+ For VirtualBox, either try the same procedure as VMWare or follow the procedure using docker-machine <https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/workstation/docker-machine/>

| hostname | address |
|-----|---------|
| rancher1 | 192.168.126.105 |

#### Howto

We need an http server to download the configuration file. See this tutorial, for example : <https://www.tecmint.com/install-apache-web-server-in-a-docker-container/>

We start the http server on the VM docker1 with the following shell command:

```bash
docker1$ docker run -dit --name apacheweb -p 8070:80 -v /home/ansible/webcontent/:/usr/local/apache2/htdocs/ httpd:2.4
```

We then check that we can upload a file from this server to the rancheros VM with wget :

```bash
ansible@docker1:~$ cd webcontent/
ansible@docker1:~/webcontent$ touch cloud-config.yml
# on ranchervm
wget 192.168.126.97:8070/cloud-config.yml
```

We define our cloud-config.yml file as follows :

```yml
# cloud-config.yml

hostname: rancher1

rancher:
  network:
    interfaces:
      eth*:
        dhcp: false
      eth0:
        address: 192.168.126.105/24
        gateway: 192.168.126.2
      dns:
        nameservers:
          - 8.8.8.8

ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAADA...ZgWmdMJtB5MuC5fH5V2l3zkA5 ansible@docker1
```

Install rancherOS :

```bash
rancher$ sudo ros install -c cloud-config.yml -d /dev/sda
```

We can then ssh into the rancher vm :

```bash
ansible@docker1:~/webcontent$ ssh rancher@192.168.126.105
The authenticity of host '192.168.126.105 (192.168.126.105)' can't be established.
ECDSA key fingerprint is SHA256:s2npj2IvaW6AVl8RAa4I/MUHKc2bPmSnIPCFrh1NAXo.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.126.105' (ECDSA) to the list of known hosts.
```

### RancherOS quick start guide

<https://rancher.com/docs/os/v1.x/en/quick-start-guide/>

## Rancher

### Introducing Rancher

Read the following documentation :

+ <https://rancher.com/what-is-rancher/overview/>

### Introducing Kubernetes

Read the following documentation :

+ <https://kubernetes.io/>
+ <https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/>
+ <https://kubernetes.io/docs/concepts/overview/components/>
+ <https://kubernetes.io/docs/concepts/architecture/nodes>
+ <https://en.wikipedia.org/wiki/Kubernetes>

### Installing rancher

<https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/quickstart-manual-setup/>

### Creating our first cluster

+ Before you proceed, snapshot your environment

Open a browser and go to rancher API GUI and click on "create a cluster".

| Node | Role | Node name |
|------|------|------|
| docker1 | etcd, controller | ranchercontroller1 |
| docker2 | worker | rancherworker1 |
| rancher1 | worker | rancherworker2 |

Follow the procedure. Cut and paste the generated shell command into a shell on the corresponding host (either docker1, docker 2 or rancher1).

Wait for the cluster to become active.

### Deploying workload

Read this article : <https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0>

Follow the instructions here : <https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/workload/quickstart-deploy-workload-ingress/>

## Introducing K3S

## Deploying K3S

## Using K3OS
