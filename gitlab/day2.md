# Day 2 : running gitlab examples

## Debugging our first examples

### Add kubernetes cluster

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

Pb: API URL is blocked: Requests to the local network are not allowed

Solution <https://gitlab.com/gitlab-org/gitlab-ce/issues/57948>

Certificate validation problem

<https://gitlab.com/gitlab-org/gitlab-ce/issues/63470>
<https://rancher.com/blog/2019/connecting-gitlab-autodevops-authorized-cluster-endpoints/>

### Auto devops quick start guide

<https://gitlab.com/help/topics/autodevops/quick_start_guide.md>

### Installing runner

See <https://docs.gitlab.com/runner/>

+ Install gitlab-runner

```bash
apt install gitlab-runner
```

### Configuring pipelines

See <https://docs.gitlab.com/ee/ci/pipelines.html>

## Testing other examples

+ <https://docs.gitlab.com/ee/ci/examples/test_phoenix_app_with_gitlab_ci_cd/index.html>
+ <https://docs.gitlab.com/ee/ci/examples/deploy_spring_boot_to_cloud_foundry/index.html>
+ <https://docs.gitlab.com/ee/ci/examples/artifactory_and_gitlab/index.html>
+ <https://docs.gitlab.com/ee/ci/examples/php.html>
