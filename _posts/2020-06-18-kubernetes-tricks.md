---
title: "Kubernetes tricks. How to get a shell into a Kubernetes Node?"
date: 2020-06-18
---

# Kubernetes tricks. How to get a shell into a Kubernetes Node?
##  **Question**

I don't have SSH access to Kubernnetes nodes. How I can get into?  

##  **Answer**

###  Get nodes names

```shell
# kubectl get nodes -o wide
NAME                                 STATUS   ROLES   AGE   VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
aks-agentpool1-81885468-vmss000000   Ready    agent   37d   v1.15.10   172.16.1.152   <none>        Ubuntu 16.04.6 LTS   4.15.0-1077-azure   docker://3.0.7
aks-agentpool1-81885468-vmss000001   Ready    agent   37d   v1.15.10   172.16.1.183   <none>        Ubuntu 16.04.6 LTS   4.15.0-1075-azure   docker://3.0.7
aks-agentpool1-81885468-vmss00000r   Ready    agent   16d   v1.15.10   172.16.0.206   <none>        Ubuntu 16.04.6 LTS   4.15.0-1075-azure   docker://3.0.7
```

###  Create YAML file

Update `nodeName` to name of the node you want access

```yaml
# helper.yaml
apiVersion: v1
kind: Pod
metadata:
  name: helper
spec:
  nodeName: aks-agentpool1-81885468-vmss000000
  hostPID: true
  containers:
  - name: helper
    securityContext: 
      privileged: true
    image: alpine
    stdin: true
    stdinOnce: true
    tty: true
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["nsenter"]
    args: [ "--target", "1", "--mount", "--uts", "--ipc", "--net", "--pid", "--", "bash", "-l" ]
```
###  Run the pod
```shell
kubectl apply -f .\helper.yaml
```

#  Attach to the pod
```shell
kubectl attach helper -i
``` 

#  Delete pod after you finish
```shell
`kubectl delete pod/helper
```
