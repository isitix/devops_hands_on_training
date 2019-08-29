# Openstack day 1

## Resources

+ Book "Openstack in action"  <https://www.manning.com/books/openstack-in-action> and <https://livebook.manning.com/book/openstack-in-action/about-this-book/38>
+ Code source of the book <https://github.com/codybum/OpenStackInAction>
+ Openstack documentation <https://docs.openstack.org>

## Introduction

+ Datacenter architecture
+ Cloud architecture
+ Openstack

### Architecture

+ VMware virtual data center architecture : <https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.introduction.doc_50%2FGUID-2E7DB290-2A07-4F54-9199-B68FCB210BBA.html>
+ Openstack general compute cloud : <https://docs.openstack.org/arch-design/use-cases/use-case-general-compute.html>
+ Openstack storage cloud : <https://docs.openstack.org/arch-design/use-cases/use-case-storage.html>
+ Openstack network virtual function cloud : <https://docs.openstack.org/arch-design/use-cases/use-case-nfv.html>
+ Openstack design : <https://docs.openstack.org/arch-design/design.html>

### Modules

+ <https://docs.openstack.org/puppet-openstack-guide/latest/contributor/module-list.html>

### Openstack operation

+ <https://docs.openstack.org/operations-guide/index.html>

### Openstack deployment using ansible

#### Kolla-ansible (containers for openstack)

+ <https://gitlab.inria.fr/discovery/kolla-ansible>
+ <https://docs.openstack.org/kolla-ansible/stein/user/>

#### Openstack-ansible (bare metal)

+ <https://docs.openstack.org/openstack-ansible/stein/user/>

### Products

+ Cisco ACI : <https://www.cisco.com/c/en/us/solutions/data-center-virtualization/application-centric-infrastructure/index.html#~software>
+ VMWare NSX : <https://www.vmware.com/products/nsx.html>
+ Nutanix : <https://www.nutanix.com/en>

### Papers

+ Google data center network architecture : <https://conferences.sigcomm.org/sigcomm/2017/files/program/topic-preview-2-2.pdf>
+ Data center, past, present and future : <http://wikibon.org/wiki/v/The_Data_Center:_Past,_Present_and_Future>
+ Data center environment : <https://en.wikipedia.org/wiki/Data_center>

## Getting started with openstack (Chapter 2 and 3 of Manning book)

### Instructions

+ Read chapter 2 and chapter 3 of the book
+ Follow instructions
+ Translate shell command into ansible tasks

### Additional resources

+ <https://docs.openstack.org/devstack/latest/index.html>

### Hands on devstack

#### Clone ubuntu1 vm to devstack

Modify the IP address of the clone: 

| hostname | IP address |
|-----|-----|
| devstack | 192.168.126.17 |

#### Add to inventory on the controller

Add the following line to your hosts.yml inventory file :

```yml
# hosts.yml
openstack:
        hosts:
                devstack:
```

Add devstack hosts to /etc/hosts on the controller

Add devstack to the hosts template file :

```j2
# templates/hosts.j2
127.0.0.1 localhost
127.0.1.1 {{inventory_hostname}}
192.168.126.10 controller
192.168.126.11 ubuntu1
192.168.126.12 ubuntu2
192.168.126.13 ubuntu3
192.168.126.14 ubuntu4

192.168.126.17 devstack

192.168.126.33 centos1
192.168.126.34 centos2
192.168.126.35 centos3

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

#### Create a new playbook

Create a new playbook to configure devstack

```yml
# openstack.yml
---
-
        hosts: administration, openstack
        tasks:
                - name: modifiy hostname
                  hostname:
                          name: "{{inventory_hostname}}"
                - name: load /etc/hosts
                  template:
                          src: templates/hosts.j2
                          dest: /etc/hosts
...
```

Run the new playbook from the controller :

```bash
$ansible-playbook openstack.yml
PLAY [administration, openstack] ***********************************************

TASK [Gathering Facts] *********************************************************
ok: [controller]
ok: [devstack]

TASK [modifiy hostname] ********************************************************
ok: [controller]
ok: [devstack]

TASK [load /etc/hosts] *********************************************************
ok: [controller]
ok: [devstack]

PLAY RECAP *********************************************************************
controller                 : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
devstack                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

#### Add the devstack configuration to the playbook

```yml
# openstack.yml
---
# global configuration
- 
        hosts: administration, openstack
        tasks:
                - name: modifiy hostname
                  hostname:
                          name: "{{inventory_hostname}}"
                - name: load /etc/hosts
                  template:
                          src: templates/hosts.j2
                          dest: /etc/hosts
-
# devstack VM configuration
  # Creer un utilisateur sysop avec sudo sans mot de pass               
        hosts: openstack
        tasks:
                - name: add sysop user with sudo privilege
                  debug:
                          msg: "TODO"
...
```

#### Add sysop user

Documentation : <https://docs.ansible.com/ansible/latest/modules/user_module.html>

```yml
                - name: add sysop user with sudo privilege
                  user:
                          name: sysop
                          comment: devstack user
                          groups: users,ssh,sudo
```

#### Apt update and upgrade

TODO
