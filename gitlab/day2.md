# Day 2 : running gitlab examples

## Debugging kubernetes integration with Gitlab

See <https://docs.gitlab.com/ee/user/project/clusters/index.html>

Kubernetes API :

```bash
ansible@docker1:~$ kubectl cluster-info | grep 'Kubernetes master' | awk '/http/ {print $NF}'
https://192.168.126.97/k8s/clusters/c-jvbnh
```

Secret tokens:

```bash
ansible@docker1:~$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
dc-8g2hc              kubernetes.io/dockerconfigjson        1      31h
default-token-6978z   kubernetes.io/service-account-token   3
```

CA certificate :

```bash
kubectl get secret default-token-6978z -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
-----BEGIN CERTIFICATE-----
MIICwjCCAaqgAwIBAgIBADANBgkqhkiG9w0BAQsFADASMRAwDgYDVQQDEwdrdWJl
LWNhMB4XDTE5MDkxMTExNTkwMFoXDTI5MDkwODExNTkwMFowEjEQMA4GA1UEAxMH
a3ViZS1jYTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKnBIKuGobAT
2tbHSFS0fdinWqsf4YQ1nM17vVaKLHK2sigeUkq7non5NZ5YJc7pKcM0pMifC51F
QX76tyWMF071SDGPpzQRF7U8gurmwZOcWHVw6/Ky0rw15OhhAHpa+t94p4QDdvcw
urRlrMxyyDYBNqifx6pHyNHm/FcrrIWz9fqvDrVoJ/a1sML6pN9Uf90J2rpzTl3m
IGntzVWFWSzF2q1qpssvwQffVTZKo6qsuIuczyn/JFbW++dF1jkFemZWgqanOJuf
7yCgsqGkWbObgxxPSuhzkX/vlVA96zfjPq1vNp+hS4Uc72AFYrYVawp0hH5Przf/
Ee5mH+p+LuUCAwEAAaMjMCEwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMB
Af8wDQYJKoZIhvcNAQELBQADggEBADZl4V08I4tfH9D1u3pS5aW7Jn3VlfSHKVwg
z6p5D3i8zD1nxQ5cDnl0Vk7u3CjwUzMipeTV+7Wtuh0nLTepHWswfVG7ksSEOQ4d
DB6oWIdxo5XMF76YGHuvoZVI4CYg/oSl4sfmxz6Wc+iQDTa6UBTmPGffFyy5VGzU
/YEkkbM/EQkd87hKsH+xd9S9SUxR8znTgmT5LE8C0G4gW7B+Br5LokK6AJcuHWGN
JAIitP2jMmVs/SdB4Nxg1P/Qqqn4aL4wRBKLFxFhi733+hO/yT1EtM1+Bndk+M6z
BZOVjkLfjNAxkFev3mc39k8FgjpZosxnusMnMq4yuIVxPWmm+v0=
-----END CERTIFICATE-----
```

Create a service account

```bash
ansible@docker1:~/k8s$ vim gitlab-admin-service-account.yaml
ansible@docker1:~/k8s$ kubectl apply -f gitlab-admin-service-account.yaml
serviceaccount/gitlab-admin created
clusterrolebinding.rbac.authorization.k8s.io/gitlab-admin created
```

Retrieve the token for gitlab-admin :

```bash
ansible@docker1:~/k8s$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab-admin | awk '{print $1}')
Name:         gitlab-admin-token-6p6dz
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: gitlab-admin
              kubernetes.io/service-account.uid: 4fedae2e-d5a1-11e9-8b57-000c29631f28

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1017 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJnaXRsYWItYWRtaW4tdG9rZW4tNnA2ZHoiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZ2l0bGFiLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNGZlZGFlMmUtZDVhMS0xMWU5LThiNTctMDAwYzI5NjMxZjI4Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmdpdGxhYi1hZG1pbiJ9.Fnn-rlyY7lYJZ1c4s6el-4jzG1TTuNmFuOMmtQA7OQGeBd38CcCI6Z6kdca-3-IMwbzW08nTGCAfEM3WXJYJDPYOXmYfevX2AE2roTotqTdxDGnfxDDTcgTojHgG8IVVALkjTkq7I0RcSuuAVThC6lAoXlIzuC9KDTGFZoPUorGuY-Q0zKqTJb2zvcDNcJQ1TCZY4bUdceFqq-Lbm_kWiCNJLYA3Ff8IQveKM4GA1_V6W5-G_nBZsz9BwWnxZnMEmenfmDmn7pErFJ0KF5fBbP1P7eeSLKYHPMaK3MXCx9KcgEl3APVD7OP90ERTA5nB_RfWtXSd4Gt-4JaO38JmIg
```

Copy these values in the form on gitlab web UI.

*Pb: API URL is blocked: Requests to the local network are not allowed*

Solution <https://gitlab.com/gitlab-org/gitlab-ce/issues/57948>

Go to the page https://<GITLAB SERVER IP>/admin/application_settings/network and check the box "Allow requests to the local network from system hooks".

*Certificate validation problem*

<https://gitlab.com/gitlab-org/gitlab-ce/issues/63470>

<https://rancher.com/blog/2019/connecting-gitlab-autodevops-authorized-cluster-endpoints/>

## Adding your K8S cluster following rancher procedure

1. Login as root on gitlab

2. Allow requests to the local network

Go to the page https://<GITLAB SERVER IP>/admin/application_settings/network and check the box "Allow requests to the local network from system hooks".

3. Get the CA certificate (the gitlab way rather than the rancher way)

```bash
ansible@docker1:~$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
dc-8g2hc              kubernetes.io/dockerconfigjson        1      31h
default-token-6978z   kubernetes.io/service-account-token   3
```

CA certificate :

```bash
kubectl get secret default-token-6978z -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
-----BEGIN CERTIFICATE-----
MIICwjCCAaqgAwIBAgIBADANBgkqhkiG9w0BAQsFADASMRAwDgYDVQQDEwdrdWJl
LWNhMB4XDTE5MDkxMTExNTkwMFoXDTI5MDkwODExNTkwMFowEjEQMA4GA1UEAxMH
a3ViZS1jYTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKnBIKuGobAT
2tbHSFS0fdinWqsf4YQ1nM17vVaKLHK2sigeUkq7non5NZ5YJc7pKcM0pMifC51F
QX76tyWMF071SDGPpzQRF7U8gurmwZOcWHVw6/Ky0rw15OhhAHpa+t94p4QDdvcw
urRlrMxyyDYBNqifx6pHyNHm/FcrrIWz9fqvDrVoJ/a1sML6pN9Uf90J2rpzTl3m
IGntzVWFWSzF2q1qpssvwQffVTZKo6qsuIuczyn/JFbW++dF1jkFemZWgqanOJuf
7yCgsqGkWbObgxxPSuhzkX/vlVA96zfjPq1vNp+hS4Uc72AFYrYVawp0hH5Przf/
Ee5mH+p+LuUCAwEAAaMjMCEwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMB
Af8wDQYJKoZIhvcNAQELBQADggEBADZl4V08I4tfH9D1u3pS5aW7Jn3VlfSHKVwg
z6p5D3i8zD1nxQ5cDnl0Vk7u3CjwUzMipeTV+7Wtuh0nLTepHWswfVG7ksSEOQ4d
DB6oWIdxo5XMF76YGHuvoZVI4CYg/oSl4sfmxz6Wc+iQDTa6UBTmPGffFyy5VGzU
/YEkkbM/EQkd87hKsH+xd9S9SUxR8znTgmT5LE8C0G4gW7B+Br5LokK6AJcuHWGN
JAIitP2jMmVs/SdB4Nxg1P/Qqqn4aL4wRBKLFxFhi733+hO/yT1EtM1+Bndk+M6z
BZOVjkLfjNAxkFev3mc39k8FgjpZosxnusMnMq4yuIVxPWmm+v0=
-----END CERTIFICATE-----
```

Save the certificate string into a file ca.crt and check that the certificat is valid :

```bash
mdautrey@databox:~$ openssl x509 -in ca.crt -noout -subject -issuer
subject= /CN=kube-ca
issuer= /CN=kube-ca
```

4. Get the service endpoint and the service token (the rancher way)

+ Get the kube config file of the cluster on Rancher GUI
+ Get the SECOND end point parameters

```yml
cluster-name : imie1
server: "https://192.168.126.97:6443"
user token: "kubeconfig-user-gpzkj.c-jvbnh:r56nd4xcbmgcp8jhnmkwrfd8wccq54sfxdk9k955fdd2p4csjrwjgs"
```

5. Set the corresponding parameters in the kubernetes config page of gitlab

## Adding a runner to execute your integration pipeline

A runner is a runtime that processes integration pipelines. You must add runners to your gitlab environment before you execute integration pipelines.

See <https://docs.gitlab.com/runner/>

## Installing a runner on kubernetes

PB!!!

## Installing a runner on Ubuntu

Follow the steps :

+ Install
+ Configure
+ Register

in the documentation <https://docs.gitlab.com/runner/>

### Installing a runner on a dedicated VM, gitlabrunner1, with ansible

Warning : gitlab is not compatible with the latest version of Ubuntu, Ubuntu 19.

Ajout d'une nouvelle machine :

| Hostname | IP |
|---------|-------|
| gitlabrunner1 | 192.168.126.114 |

Task:

```yml
- hosts: gitlabrunner1
  roles:
    - linux
    - gitlab-runner
```

Role:

```yml
# roles/gitlab-runner/tasks/main.yml
# TODO : refactorize with gitlab role

# gitlab installation and configuration
# for an installation procedure, read https://about.gitlab.com/install/#ubuntu 
# We don't install postfix because we don't want to setup mail alert
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

- name: install gitlab-runner
  apt:
    name: gitlab-runner
    state: present
```

### Registering the Ubuntu runner

The runner calls a runner executor, either ssh, bash, docker, ... To run any kind of work on our runner, we install docker on the runner via docker-machine.

```bash
ansible@docker1:~$ docker-machine create --driver generic --generic-ip-address 192.168.126.114 --generic-ssh-key ~/.ssh/id_rsa --generic-ssh-user ansible gitlabrunner1
```

Then we get our url and token on gitlab web GUI project page, settings->CI/CD->Runner

```txt
URL : https://192.168.126.113.xip.io/
Token : 5usmKyjUfxEKac_4-EJ_
```

We execute the register command : 

```bash
ansible@gitlabrunner1:~$ sudo gitlab-runner register
Running in system-mode.                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://192.168.126.113.xip.io/
Please enter the gitlab-ci token for this runner:
5usmKyjUfxEKac_4-EJ_
Please enter the gitlab-ci description for this runner:
[gitlabrunner1]: gitlabrunner1
Please enter the gitlab-ci tags for this runner (comma separated):

Whether to lock the Runner to current project [true/false]:
[true]: 
ERROR: Registering runner... failed                 runner=5usmKyjU status=couldn't execute POST against https://192.168.126.113.xip.io/api/v4/runners: Post https://192.168.126.113.xip.io/api/v4/runners: x509: certificate signed by unknown authority
PANIC: Failed to register this runner. Perhaps you are having network problems 
```

We have a problem with the self-signed certificate. Solution found here : <https://docs.gitlab.com/runner/configuration/tls-self-signed.html#supported-options-for-self-signed-certificates>

Solution :

```bash
ansible@gitlabrunner1:/$ sudo mkdir /etc/gitlab-runner/certs
ansible@gitlabrunner1:/$ sudo scp ansible@gitlab1:/etc/gitlab/ssl/gitlab.crt /etc/gitlab-runner/certs/192.168.126.113.xip.io.crt
```

Then, play the register process again : 

```bash
ansible@gitlabrunner1:/srv$ sudo gitlab-runner register
Running in system-mode.                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://192.168.126.113.xip.io/
Please enter the gitlab-ci token for this runner:
5usmKyjUfxEKac_4-EJ_
Please enter the gitlab-ci description for this runner:
[gitlabrunner1]: 
Please enter the gitlab-ci tags for this runner (comma separated):

Whether to lock the Runner to current project [true/false]:
[true]: 
Registering runner... succeeded                     runner=5usmKyjU
Please enter the executor: kubernetes, docker, parallels, ssh, virtualbox, docker-ssh+machine, docker-ssh, shell, docker+machine:
docker
Please enter the default Docker image (e.g. ruby:2.1):
alpine
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

### Auto devops quick start guide

Auto-devops generate default pipelines for applications that meet standard configuration.

<https://gitlab.com/help/topics/autodevops/quick_start_guide.md>

## Deploying your first application following Rancher instruction

Tutorial source : <https://rancher.com/blog/2019/connecting-gitlab-autodevops-authorized-cluster-endpoints/>

1. Install Helm

See <https://v3.helm.sh/>

2. Install Ingress

See <https://kubernetes.io/docs/concepts/services-networking/ingress/>

3. Create a project based on NodeJS gitlab template

4. Set it up as required by Rancher

5. Test

### Configuring pipelines

See <https://docs.gitlab.com/ee/ci/pipelines.html>

## Testing other examples

+ <https://docs.gitlab.com/ee/ci/examples/test_phoenix_app_with_gitlab_ci_cd/index.html>
+ <https://docs.gitlab.com/ee/ci/examples/deploy_spring_boot_to_cloud_foundry/index.html>
+ <https://docs.gitlab.com/ee/ci/examples/artifactory_and_gitlab/index.html>
+ <https://docs.gitlab.com/ee/ci/examples/php.html>

## Quizz final

<https://forms.gle/RMy4kwru9tmGrTp37>