# Openstack day 2

## Back to devstack installation (shell mode)

1. Add resources to your centos VM : > 4GB RAM, compute
2. Clone it
3. Execute the following procedure : <http://devopspy.com/cloud-computing/openstack-all-in-one-setup-centos/>

### IP address

| Hostname | IP |
|------|-----|
| packstack | 192.168.126.65 |

### Executing the procedure

Packstack documentation : <https://wiki.openstack.org/wiki/Packstack>

Message at the end of the procedure :

```bash
Additional information:
 * Parameter CONFIG_NEUTRON_L2_AGENT: You have choosen OVN neutron backend. Note that this backend does not support LBaaS, VPNaaS or FWaaS services. Geneve will be used as encapsulation method for tenant networks
 * A new answerfile was created in: /root/packstack-answers-20190830-100934.txt
 * Time synchronization installation was skipped. Please note that unsynchronized time on server instances might be problem for some OpenStack components.
 * File /root/keystonerc_admin has been created on OpenStack client host 192.168.126.65. To use the command line tools you need to source the file.
 * To access the OpenStack Dashboard browse to http://192.168.126.65/dashboard .
Please, find your login credentials stored in the keystonerc_admin in your home directory.
 * The installation log file is available at: /var/tmp/packstack/20190830-100933-4TP3Ag/openstack-setup.log
 * The generated manifests are available at: /var/tmp/packstack/20190830-100933-4TP3Ag/manifests
```

### Openstack modules

Modules list, book p18 (ch1)

## Chapter 2 and chapter 3 of Manning book "Openstack in action"

### Backup

Take a snapshot of the VM.

### Tempest test

#### Tempest test use cases

+ Validating development on Openstack
+ Benchmarking your Openstack instance (design, performance, hardware architecture)
+ Defining and running acceptance tests for an Openstack integration project

#### Hands on Tempest

Documentation available here :

+ Ch2
+ <https://docs.openstack.org/tempest/latest/>
+ <https://docs.openstack.org/tempest/latest/field_guide/index.html>

Tutorial : <https://blogs.rdoproject.org/2016/09/running-tempest-on-rdo-openstack-newton/>

XXXX Problem with the tutorial to launch tests

### Access to Openstack Dashboard (Horizon)

+ Log into the dashboard
+ Ch2.3

To get the admin credentials :

```bash
sudo cat /root/keystonerc_admin
```

Then open a browser and goto the URL <http://PACKSTACKIP/dashboard>

### Create and launch your first VM

+ Ch2.4

XXXX network error while provisioning the new VM

### Test the Openstack CLI

To configure the CLI, follow the instructions here : <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/4/html/End_User_Guide/cli_openrc.html>

Defaults credentials are stored in the following files :

+ admin user : /root/keystonerc_admin
+ demo user : /root/keystonerc_demo
+ Ch3.1

### Configure a project

+ Ch3.3

XXXX tenant is deprecated. It has been replaced by project.

To create a new project:

```bash
openstack project create NEW_PROJECT_NAME
```

## Ansible ready version

+ Translate the devstack shell installation procedure into an ansible playbook.
+ Test and validate your playbook

## Openstack on OVH

### Activate Openstack on OVH

+ Goto your management console
+ Activate public cloud
+ Create a new project
+ Goto Users&Roles
+ Add a user with the desired role

### Connect to Horizon

+ Go to OVH Horizon Web portail : <https://horizon.cloud.ovh.net>
+ Use the login and password defined previously

### Create an instance

+ Test instance creation on OVH public cloud
+ ssh into the instance using your public-private keys pair

### Install the environment to use Openstack api

<https://docs.ovh.com/gb/en/public-cloud/prepare_the_environment_for_using_the_openstack_api/>

### Connect to OVH Public Cloud using the CLI

The procedure is explained in the section "Test the Openstack CLI".

### Install and test Heat on OVH

+ <https://docs.ovh.com/gb/en/public-cloud/deploy-infrastructure-with-openstack-heat/>

XXX TODO

## Bonus : Openstack on Docker with Ansible

Test the installation of Kolla Openstack

+ <https://gitlab.inria.fr/discovery/kolla-ansible>
+ <https://docs.openstack.org/kolla-ansible/stein/user/>

## QCM Openstack

<https://forms.gle/CUrzSLwn8aS9GGGK9>

## END OF DAY 2
