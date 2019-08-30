# Openstack day 2

## Back to devstack installation (shell mode)

1. Add resources to your centos VM : > 4GB RAM, compute
2. Clone it
3. Execute the following procedure : <http://devopspy.com/cloud-computing/openstack-all-in-one-setup-centos/>

### Openstack modules

Modules list, book p18 (ch1)

## Chapter 2 and chapter 3 of Manning book "Openstack in action"

### Backup

Take a snapshot of the VM.

### Tempest test

Run Openstack Tempest test :

+ Ch2
+ <https://docs.openstack.org/tempest/latest/>
+ <https://docs.openstack.org/tempest/latest/field_guide/index.html>

### Access to Openstack Dashboard (Horizon)

+ Log into the dashboard
+ Ch2.3

### Create and launch your first VM

+ Ch2.4

### Test the Openstack CLI

+ Ch3.1

### Test the Openstack Rest API

+ Ch3.2

### Configure a tenant

+ Ch3.3

### Configure quotas

+ Ch3.4

## Ansible ready version

+ Translate the devstack shell installation procedure into an ansible playbook.
+ Test and validate your playbook

## Openstack on OVH

### Connect to Horizon

+ <https://horizon.cloud.ovh.net>

### Test the previous tutorial on OVH Horizon

+ Ch2.3
+ Ch2.4

### Install the environment to use Openstack api

<https://docs.ovh.com/gb/en/public-cloud/prepare_the_environment_for_using_the_openstack_api/>

### Test the Openstack CLI on OVH

+ Ch3.1

### Test the Openstack Rest API on OVH

+ Ch3.2

### Install and test Heat on OVH

+ <https://docs.ovh.com/gb/en/public-cloud/deploy-infrastructure-with-openstack-heat/>

### Ansible playbook for OVH Openstack environment

Write an Ansible playbook to install OVH Openstack environment

## Bonus : Openstack on Docker with Ansible

Test the installation of Kolla Openstack

+ <https://gitlab.inria.fr/discovery/kolla-ansible>
+ <https://docs.openstack.org/kolla-ansible/stein/user/>


## END OF DAY 2

