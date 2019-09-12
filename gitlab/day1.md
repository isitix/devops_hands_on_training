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

Many versions with different licenses <https://www.howtoforge.com/tutorial/how-to-install-and-configure-gitlab-on-ubuntu-1804/>

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
```

## Running gitlab in a container

Test this procedure <https://docs.gitlab.com/omnibus/docker/>

## Browsing the web interface

+ Connect to gitlab with your browser http://<PUBLIC_IP>
+ Customize the password of the root account (login name = root)
+ Login
