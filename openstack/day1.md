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
                          password: ansible
                            - name: add sudo to sysop no passwd
                  copy:
                          content: 'sysop ALL=NOPASSWD: ALL\n'
                          dest: /etc/sudoers.d/sysop
                          mode: 0400

```

#### Apt update and upgrade

```yml
                - name: apt update, upgrade, autoclean, autoremove, purge
                  apt:
                          autoclean: yes
                          autoremove: yes
                          purge: yes
                          update_cache: yes
                          cache_valid_time: 90000
                          upgrade: yes
```

#### Install git

```yml
                - name: install git
                  apt:
                        name: git
                        state: latest
```

#### Final version of the playbook

```yml
# openstack.yml
---
# global configuration
- 
        hosts: administration, openstack
        become: yes
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
        become: yes
        vars:
                devstack_password: devstack
        tasks:
                - name: add sysop user with sudo privilege
                  user:
                          name: sysop
                          comment: devstack user
                          groups: users,ssh,sudo
                          state: present
                          home: /home/sysop
                          password: ansible
                - name: add sudo to sysop no passwd
                  copy:
                          content: 'sysop ALL=NOPASSWD: ALL'
                          dest: /etc/sudoers.d/sysop
                          mode: 0400
                          force: no
                - name: apt update, upgrade, autoclean, autoremove, purge
                  apt:
                          autoclean: yes
                          autoremove: yes
                          purge: yes
                          update_cache: yes
                          cache_valid_time: 90000
                          upgrade: yes
                - name: install git
                  apt:
                          name: git
                          state: latest
                - name: clone devstack git directory
                  git:
                          accept_hostkey: true
                          clone: yes
                          dest: /opt/devstack/
                          repo: https://github.com/openstack/devstack.git
                          version: stable/stein
                          update: no
                - name: create stack user
                  shell: tools/create-stack-user.sh
                  args:
                          chdir: /opt/devstack
                - name: copy local.conf template to /opt/devstack
                  template:
                          src: templates/local.j2
                          dest: /opt/devstack/local.conf
-
        hosts: openstack
        become_user: stack
        tasks:
                - name: execute devstack installation script
                  shell: /opt/devstack/stack.sh
                  become_user: stack
                  args:
                          chdir: /opt/devstack
...
```

```yml #host.yml
---
all:
        vars:
                ansible_user: ansible
        hosts:
                ubuntu4:
                        ansible_become_pass: ansible
application:
        children:
                production:
                        hosts:
                                ubuntu[1:2]:
                                centos[1:2]:
                test:
                        hosts:
                                ubuntu[3:4]:
                                centos3:
administration:
        hosts:
                controller:
                        ansible_connection: local
openstack:
        hosts:
                devstack:
linux:
        children:
                ubuntu:
                        hosts:
                                ubuntu[1:4]:
                                controller:
                                devstack1:

                centos:
                        hosts:
                                centos[1:3]:
...

```

```yml
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

```yml
# local.j2

# Sample ``local.conf`` for user-configurable variables in ``stack.sh``

# NOTE: Copy this file to the root DevStack directory for it to work properly.

# ``local.conf`` is a user-maintained settings file that is sourced from ``stackrc``.
# This gives it the ability to override any variables set in ``stackrc``.
# Also, most of the settings in ``stack.sh`` are written to only be set if no
# value has already been set; this lets ``local.conf`` effectively override the
# default values.

# This is a collection of some of the settings we have found to be useful
# in our DevStack development environments. Additional settings are described
# in https://docs.openstack.org/devstack/latest/configuration.html#local-conf
# These should be considered as samples and are unsupported DevStack code.

# The ``localrc`` section replaces the old ``localrc`` configuration file.
# Note that if ``localrc`` is present it will be used in favor of this section.
[[local|localrc]]

# Minimal Contents
# ----------------

# While ``stack.sh`` is happy to run without ``localrc``, devlife is better when
# there are a few minimal variables set:

# If the ``*_PASSWORD`` variables are not set here you will be prompted to enter
# values for them by ``stack.sh``and they will be added to ``local.conf``.
ADMIN_PASSWORD={{devstack_password}}
DATABASE_PASSWORD={{devstack_password}}
RABBIT_PASSWORD={{devstack_password}}
SERVICE_PASSWORD={{devstack_password}}

# ``HOST_IP`` and ``HOST_IPV6`` should be set manually for best results if
# the NIC configuration of the host is unusual, i.e. ``eth1`` has the default
# route but ``eth0`` is the public interface.  They are auto-detected in
# ``stack.sh`` but often is indeterminate on later runs due to the IP moving
# from an Ethernet interface to a bridge on the host. Setting it here also
# makes it available for ``openrc`` to include when setting ``OS_AUTH_URL``.
# Neither is set by default.
HOST_IP={{ansible_default_ipv4.address}}
#HOST_IPV6=2001:db8::7


# Logging
# -------

# By default ``stack.sh`` output only goes to the terminal where it runs.  It can

# be configured to additionally log to a file by setting ``LOGFILE`` to the full
# path of the destination log file.  A timestamp will be appended to the given name.
LOGFILE=$DEST/logs/stack.sh.log

# Old log files are automatically removed after 7 days to keep things neat.  Change
# the number of days by setting ``LOGDAYS``.
LOGDAYS=2

# Nova logs will be colorized if ``SYSLOG`` is not set; turn this off by setting
# ``LOG_COLOR`` false.
#LOG_COLOR=False


# Using milestone-proposed branches
# ---------------------------------

# Uncomment these to grab the milestone-proposed branches from the
# repos:
#CINDER_BRANCH=milestone-proposed
#GLANCE_BRANCH=milestone-proposed
#HORIZON_BRANCH=milestone-proposed
#KEYSTONE_BRANCH=milestone-proposed
#KEYSTONECLIENT_BRANCH=milestone-proposed
#NOVA_BRANCH=milestone-proposed
#NOVACLIENT_BRANCH=milestone-proposed
#NEUTRON_BRANCH=milestone-proposed
#SWIFT_BRANCH=milestone-proposed

# Using git versions of clients
# -----------------------------
# By default clients are installed from pip.  See LIBS_FROM_GIT in
# stackrc for details on getting clients from specific branches or
# revisions.  e.g.
# LIBS_FROM_GIT="python-ironicclient"
# IRONICCLIENT_BRANCH=refs/changes/44/2.../1

# Swift
# -----

# Swift is now used as the back-end for the S3-like object store. Setting the
# hash value is required and you will be prompted for it if Swift is enabled
# so just set it to something already:
SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5

# For development purposes the default of 3 replicas is usually not required.
# Set this to 1 to save some resources:
SWIFT_REPLICAS=1

# The data for Swift is stored by default in (``$DEST/data/swift``),
# or (``$DATA_DIR/swift``) if ``DATA_DIR`` has been set, and can be
# moved by setting ``SWIFT_DATA_DIR``. The directory will be created
# if it does not exist.
SWIFT_DATA_DIR=$DEST/data
```