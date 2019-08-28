# Ansible, day 2

## Add centos 1 to 3 to the inventory
### IP addresses
| hostname | IP |
|------|----|
| centos1 | 192.168.126.33 |
| centos2 | 192.168.126.34 |
| centos3 | 192.168.126.35 |

### Network configuration

Example of configuration procedure : http://www.mustbegeek.com/configure-static-ip-address-in-centos/

1. Edit the network configuration file :

```bash
$su
$vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

eth0 must be replaced by the id of your interface card.

2. Reboot

3. Edit the resolv.conf file

```bash
$su
$vi /etc/resolv.conf
nameserver 8.8.8.8
:wq
```

### SSH

1. Install openssh server

Nota : ssh server is already installed if you've got a default cofiguration of centos

```bash
$su
$yum install -y openssh-server
```

2. Test connection from controller

```bash
ansible@controller:~/deployment$ ssh ansible@192.168.126.33
The authenticity of host '192.168.126.33 (192.168.126.33)' can't be established.
ECDSA key fingerprint is SHA256:lJU+RaEF8d6RFzNyD/7nIDubAAVnCCgrfPR6qYFrht4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.126.33' (ECDSA) to the list of known hosts.
ansible@192.168.126.33's password: 
Last login: Wed Aug 28 11:35:46 2019
[ansible@localhost ~]$ exit
déconnexion
Connection to 192.168.126.33 closed.
```

3. Add you ssh key to the centos host

```bash
ansible@controller:~/deployment$ ssh-copy-id ansible@192.168.126.33
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ansible/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
ansible@192.168.126.33's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ansible@192.168.126.33'"
and check to make sure that only the key(s) you wanted were added.

ansible@controller:~/deployment$ ssh ansible@192.168.126.33
Last login: Wed Aug 28 09:40:23 2019 from 192.168.126.10
[ansible@localhost ~]$ exit
déconnexion
Connection to 192.168.126.33 closed.
```

4. Sudo without password for ansible user

Tuto : https://www.cyberciti.biz/faq/how-to-sudo-without-password-on-centos-linux/

```bash
$su
$visudo -f /etc/sudoers.d/ansible
ansible ALL=NOPASSWD: ALL
```


### Inventaire

Add centos1 to /etc/hosts on controller

Add centos1 to yaml file :

```yaml
---
all:
        vars:
                ansible_become: true
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
                localhost:
                        ansible_connection: local
linux:
        children:
                ubuntu:
                        hosts:
                                ubuntu[1:4]:
                                localhost:
                centos:
                        hosts:
                                centos[1:3]:
...
```

## Core functionalities

### Modules

+ Tutorial : https://github.com/spurin/masteringansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/02%20-%20Ansible%20Modules
+ Documentation : https://docs.ansible.com/ansible/latest/user_guide/modules.html
+ Documentation : https://docs.ansible.com/ansible/latest/user_guide/modules_intro.html

#### Modules in a nutshell

Modules are like libraries or extensions to a language. They provide an easy and abstracted access to functionalities such as :

+ Gathering facts
+ Installing software
+ Configuring devices
+ Running shell commands
+ Manipulating files and directories

You can call modules either from the ansible shell or from a playbook. Examples follow.

#### Gathering facts

```bash
ansible ubuntu3 -m setup
```

#### Running command line shell

Reboot

```bash
ansible ubuntu1 -m command -a "/sbin/reboot -t now"
```

#### Controlling service

Start cron

```bash
ansible ubuntu2 -m service -a "name=cron state=started"
```

#### Copying files

```bash
touch test
ansible ubuntu2 -m copy -a "src=test dest=/home/ansible/"
```

### YAML

#### YAML resources

+ Tutorial : https://github.com/spurin/masteringansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/03%20-%20YAML
+ Specifications : https://yaml.org/spec/1.2/spec.html

#### Yaml in a nutshell

YAML includes :

+ Structure (document)
+ Lists
+ Dictionaries
+ Key-value pairs
+ Type inference

#### Examples

Download files in the tutorial and run the python script on each example :

```bash
chmod u+X show_yaml_python.py
show_yaml_python.py
```

### Playbooks

#### Playbooks resources

Tutorial : [Introduction to playbooks]("https://github.com/spurin/masteringansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/04%20-%20Ansible%20Playbooks%2C%20Breakdown%20of%20sections")

#### To run the tutorial

Clone the repository into your ansible controller

```bash
git clone https://github.com/spurin/masteringansible.git
```

Goto the playbook subdirectories

```bash
cd "masteringansible/02 - Ansible Architecture and Design/04 - Ansible Playbooks, Breakdown of sections/02"
```

Remove the local (default) ansible configuration file 

```bash
mv ansible.cfg ansible.cfg.old
```

Run the example


```bash
ansible-playbook -i hosts motd-playbook.yml
```

### Variables

#### Variables resources

+ Tutorial : [playbook variables]("https://github.com/PacktPublishing/Mastering-Ansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/05%20-%20Ansible%20Playbooks%2C%20Variables")
+ Documentation : https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html

#### Variables in action

We want to update /etc/hostname and /etc/hosts on our 8 stations so that hostname value is the real hostname and /etc/hosts contains every existing hosts.

##### Changing the hostname in /etc/hostname

Documentation : https://docs.ansible.com/ansible/latest/modules/hostname_module.html

Run the following playbook :

```yml
---
-
        hosts: all
        tasks:
                - name: show host name
                  debug:
                          msg: "{{inventory_hostname}}"
                - name: update /etc/hostname
                  copy:
                          content: "{{inventory_hostname}}\n"
                          dest: /etc/hostname
                - name: modifiy hostname on the system
                  hostname:
                          name: "{{inventory_hostname}}"
...
```




### Facts

+ Tutorial : [playbook facts]("https://github.com/PacktPublishing/Mastering-Ansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/06%20-%20Ansible%20Playbooks%2C%20Facts")
+ Documentation : https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variables-discovered-from-systems-facts

### Jinja2

+ Tutorial : [jinja2]("https://github.com/PacktPublishing/Mastering-Ansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/07%20-%20Templating%20with%20Jinja2")
+ Documentation : https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html

### Wrap up

+ Tutorial : [Wrap up of day1](https://github.com/PacktPublishing/Mastering-Ansible/tree/master/02%20-%20Ansible%20Architecture%20and%20Design/08%20-%20Ansible%20Playbooks%2C%20Creating%20and%20Executing)


## Ansible functionalities

### Patterns

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html

### Gathering facts

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html#gathering-facts


## Playbook functionalities


### Handlers

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#handlers-running-operations-on-change

### Using variables

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html
+ Tutorial : [Hosts vars and groups vars](https://github.com/spurin/masteringansible/tree/master/03%20-%20Ansible%20Playbooks%2C%20Advanced%20Topics/01%20-%20Ansible%20Playbook%20Modules)
+ Variables and inventory : https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html

### Conditionals

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html

### Loops

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html

### Blocks

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/playbooks_blocks.html

### Templating and filters with Jinja2

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html

## Structuring Ansible playbooks

### Include and imports

+ Tutorial : [Include and import](https://github.com/spurin/masteringansible/tree/master/04%20-%20Structuring%20Ansible%20Playbooks/01%20-%20Using%20Include%20and%20Import)

### Tags

+ Tutorial : [Tags](https://github.com/spurin/masteringansible/tree/master/04%20-%20Structuring%20Ansible%20Playbooks/02%20-%20Using%20Tags)

### Roles

+ Tutorial : [Roles](https://github.com/spurin/masteringansible/tree/master/04%20-%20Structuring%20Ansible%20Playbooks/03%20-%20Using%20Roles)

## Advanced topics

### Parallelism

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html#parallelism-and-shell-commands

### Tests

+ Tutorial : https://docs.ansible.com/ansible/latest/reference_appendices/test_strategies.html

### Rolling updates

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html

### Dynamic inventory

+ Documentation : https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html
+ Tutorial : [dynamic inventory](https://github.com/spurin/masteringansible/tree/master/03%20-%20Ansible%20Playbooks%2C%20Advanced%20Topics/02%20-%20Dynamic%20Inventories)

### Privilege escalation

+ Tutorial : https://docs.ansible.com/ansible/latest/user_guide/become.html

## Quizz

https://forms.gle/KdzcAhrcugeoitqVA

## END OF DAY 2
