---
title: "No Title"
date: 2020-06-18
---

###  Kubernetes tricks. How to get shell into a Kubernetes Node?

  


###  **Question**

### 

**  
**

I don't have SSH access to Kubernnetes nodes. How I can get into?  
  


###  **Answer**

### 

  


#  Get nodes names[](https://arkadiumarena.visualstudio.com/Department_Base_Engineers/_wiki/wikis/Department_Base_Engineers.wiki/5670/SSH-to-node-v2?anchor=get-nodes-names)

### 
    
    
    kubectl get nodes -o wide
    NAME                                 STATUS   ROLES   AGE   VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
    aks-agentpool1-81885468-vmss000000   Ready    agent   37d   v1.15.10   172.16.1.152   <none>        Ubuntu 16.04.6 LTS   4.15.0-1077-azure   docker://3.0.7
    aks-agentpool1-81885468-vmss000001   Ready    agent   37d   v1.15.10   172.16.1.183   <none>        Ubuntu 16.04.6 LTS   4.15.0-1075-azure   docker://3.0.7
    aks-agentpool1-81885468-vmss00000r   Ready    agent   16d   v1.15.10   172.16.0.206   <none>        Ubuntu 16.04.6 LTS   4.15.0-1075-azure   docker://3.0.7
    

#  Create YAML file[](https://arkadiumarena.visualstudio.com/Department_Base_Engineers/_wiki/wikis/Department_Base_Engineers.wiki/5670/SSH-to-node-v2?anchor=update-yaml)

### 

Update `nodeName` to name of the node you want access

  


helper.yaml

  

    
    
    

#  Run pod[](https://arkadiumarena.visualstudio.com/Department_Base_Engineers/_wiki/wikis/Department_Base_Engineers.wiki/5670/SSH-to-node-v2?anchor=run-pod)

### 

`kubectl apply -f .\helper.yaml`  
`  
`

#  Attach to the pod[](https://arkadiumarena.visualstudio.com/Department_Base_Engineers/_wiki/wikis/Department_Base_Engineers.wiki/5670/SSH-to-node-v2?anchor=attach-to-the-pod)

### 

`kubectl attach helper -i`  
`  
`

#  Delete pod after you finish[](https://arkadiumarena.visualstudio.com/Department_Base_Engineers/_wiki/wikis/Department_Base_Engineers.wiki/5670/SSH-to-node-v2?anchor=delete-pod-after-you-finish)

### 

`kubectl delete pod/helper`
