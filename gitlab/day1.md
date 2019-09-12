# Day 1 : installing and setting up gitlab

## Resources

+ Book <https://www.packtpub.com/eu/virtualization-and-cloud/gitlab-quick-start-guide>

## Introducing Gitlab

+ Version control
+ Ticketing
+ Project management, Kanban
+ CI/CD

Watch the introduction video <https://www.youtube.com/watch?v=yjxrBSllNGo>

Features shared with Gihub :

+ Version control
+ Ticketing
+ Kanban

Available both as SAAS or as self-hosted

Many versions with different licenses <https://about.gitlab.com/pricing/>

## Version flow

See <https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow>

Flow specific to Gitlab <https://docs.gitlab.com/ee/workflow/gitlab_flow.html>

## Installation requirements

+ Basic : 1 CPU, 2GB of RAM
+ Ubuntu

## Installation on Ubuntu with Ansible

Without Ansible, installation procedure is available here : <https://about.gitlab.com/install/#ubuntu>

+ Add a role gitlab on the ansible controller
+ Add the required tasks to the role
+ Clone the Ubuntu VM 18 LTS
+ Configure a static IP address
+ Install Gitlab

```yml
# roles/gitlab/tasks/main.yml
# gitlab installation and configuration
# for an installation procedure, read https://about.gitlab.com/install/#ubuntu
# We don't install postfix because we don't need mail notifications
- name: install gitlab dependencies
  apt:
    name: "{{packages}}"
    state: latest
  vars:
    packages:
      - curl
      - openssh-server
      - ca-certificates

- name: check if repository shell script is already on the VM
  stat:
    path: "{{working_directory}}/script.deb.sh"
  register: gitlab_script

- name: dowload repo shell script
  get_url:
    url: https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
    dest: "{{working_directory}}/script.deb.sh"
    mode: '0755'

# We install cosmic version (Ubuntu 19) but our target is Ubuntu 19 (not supported)...
- name: execute the script to add gitlab repository
  shell:
    cmd: "{{working_directory}}/script.deb.sh"
  when: gitlab_script.stat.exists == False

- name: update apt
  apt:
    update_cache: yes
  when: gitlab_script.stat.exists == False

- name : install gitlab
  apt:
    name: gitlab-ce
    state: latest
```

To configure gitlab, download the configuration file template:

```bash
wget https://gitlab.com/gitlab-org/omnibus-gitlab/raw/master/files/gitlab-config-template/gitlab.rb.template
```

Move it to roles/gitlab/templates/gitlab.rb.j2 and customize the web URL.

Then add the following task to the role:

```yml
- name: set basic configuration gitlab.rb file
  template:
    src: gitlab.rb.j2
    dest: /etc/gitlab/gitlab.rb
  notify: reconfigure gitlab
```

and add the corresponding handler:

```yml
# /roles/gitlab/handlers/main.yml
- name: refonfigure gitlab
  shell: gitlab-ctl reconfigure
  listen: reconfigure gitlab
```

## Running gitlab in a container

Test this procedure <https://docs.gitlab.com/omnibus/docker/>

## Browsing the web interface

+ Connect to gitlab with your browser <http://<PUBLIC_IP>>
+ Customize the password of the root account (login name = root)
+ Login

## Adding https access to gitlab

### Generating self-signed certificate using 192.168.126.113.xip.io DNS name :

```bash
ansible@gitlab1:~$ sudo openssl genrsa -out "/etc/gitlab/ssl/gitlab.key" 2048
```

```bash
Generating RSA private key, 2048 bit long modulus (2 primes)
.................................................+++++
....................................................................+++++
e is 65537 (0x010001)
```

```bash
ansible@gitlab1:~$ sudo openssl req -new -key "/etc/gitlab/ssl/gitlab.key" -out "/etc/gitlab/ssl/gitlab.csr" 
```

```bash
Can't load /home/ansible/.rnd into RNG
139690523541952:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/home/ansible/.rnd
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:FR
Locality Name (eg, city) []:FR
Organization Name (eg, company) [Internet Widgits Pty Ltd]:FR
Organizational Unit Name (eg, section) []:FR 
Common Name (e.g. server FQDN or YOUR name) []:192.168.126.113.xip.io
Email Address []:mikael.dautrey@isitix.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

```bash
ansible@gitlab1:~$ sudo openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.csr" -signkey "/etc/gitlab/ssl/gitlab.key"  -out "/etc/gitlab/ssl/gitlab.crt"
```

```bash
Signature ok
subject=C = FR, ST = FR, L = FR, O = FR, OU = FR, CN = 192.168.126.113.xip.io, emailAddress = mikael.dautrey@isitix.com
Getting Private key
```

### Updating gitlab configuration template

```yml
# ansible playbook
- hosts: gitlab1
  vars:
    external_url: "https://192.168.126.113.xip.io"
    gitlab_crt: "/etc/gitlab/ssl/gitlab.crt"
    gitlab_key: "/etc/gitlab/ssl/gitlab.key"
  roles:
    - linux
    - gitlab
```

```yml
# roles/gitlab/templates/gitlab.rb.j2
external_url '{{external_url}}'

nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "{{gitlab_crt}}"
nginx['ssl_certificate_key'] = "{{gitlab_key}}"

```

## Initialisation de Gitlab

+ Add https and ssh to gitlab server
+ Add a user account (imie)
+ Connect to gitlab with the new user account

## Test an example of application

See <https://docs.gitlab.com/ee/ci/examples/test-clojure-application.html>
