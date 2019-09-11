# Day 1 : installing and setting up gitlab

## Resources

+ Book <https://www.packtpub.com/eu/virtualization-and-cloud/gitlab-quick-start-guide>

## Introducing Gitlab

+ Version control
+ Ticketing
+ Project management, Kanban
+ CI/CD

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

Without Ansible, installation procedure is available here : <https://www.howtoforge.com/tutorial/how-to-install-and-configure-gitlab-on-ubuntu-1804/>

+ Add a role gitlab on the ansible controller
+ Add the required tasks to the role
+ Clone the Ubuntu VM
+ Configure a static IP address
+ Install Gitlab

