# SpeCon

Specon is a noval container scheduler that is optimized for short-lived deep learning applications. SpeCon analyzes the common characteristics of training. We design a suite of algorithm to monitor the progress of the trainning and speculatively migrate the slow-growing models to release resources for fast-growing ones.

## Getting Started

Specon relies on root privilege and customized Kubernetes configuration and required NFS file system to store logs for all workers. SpeCon is only actively tested on Ubuntu 18.04 LTS and Kubernetes 1.17

### Build

#### Specon-configuration

build PersistentVolume(PV) and PV Claim(PVC) in both default namespace and kube-system to /mnt/linuxidx which is a shared folder for all workers.

PV example
```buildoutcfg
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv1-nfs-system
  namespace: kube-system
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /mnt/linuxidc
    server: master_ip
```
PVC example
```buildoutcfg
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc1-nfs-system
  namespace: kube-system
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
```
Grant delete and create roles to all pods resource by edit kube-scheduler config
```buildoutcfg
kubectl edit clusterrole system:kube-scheduler
```
Modify config to
```buildoutcfg
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - delete
  - get
  - list
  - watch
  - create
```
Create service account
```buildoutcfg
kubectl apply -f Specon-configuration/role.yaml
```

#### NFS 
Build NFS system for workers and mount files to /mnt/linuxidc

####
Config Specon system by changing ./Specon-system/Specon.yaml
Firstly changing all node-name in Spec.nodename and args. Node name is the shown in kubernetes cluster. 0.1 and 10 are two parameters in the paper.
```buildoutcfg
nodeName: node0
```
```buildoutcfg
containers:
    - name: worker1
      image: fuyuqi1995/mig-worker
      command: ["python"]
      args: ["worker.py","node1","0.1","10"]
```
Then start Specon system
```buildoutcfg
kubectl apply -f Spcon-system.Specon.yaml
```
### Example

An example yaml to deploy bidirectional rnn job in SpeCon system.
```buildoutcfg
kubectl apply -f examples/job.yaml
```
### Evaluation
All logs from Specon are stored in /mnt/linuxidc. Both master and workers generate logs. And migrate events would be summarized in distribution.csv. As an example
```buildoutcfg
| time               | job   | orginal_node                           | to_node                                |
|--------------------|-------|----------------------------------------|----------------------------------------|
| 1578177799.674656  | job1  | node-1.mig.shield-pg0.utah.cloudlab.us | node-3.mig.shield-pg0.utah.cloudlab.us |
| 1578177802.5349672 | job2  | node-1.mig.shield-pg0.utah.cloudlab.us |                                        |
| 1578177832.698994  | job4  | node-2.mig.shield-pg0.utah.cloudlab.us |                                        |
| 1578177832.7767122 | job5  | node-4.mig.shield-pg0.utah.cloudlab.us |                                        |
| 1578177891.2359834 | job3  | node-1.mig.shield-pg0.utah.cloudlab.us | node-2.mig.shield-pg0.utah.cloudlab.us |
```
There are two migration events happened: at time 1578177799.674656, job 1 migrate from node 1 to node 3; at time 1578177891.2359834, job 3 migrate from node 1 to node 2.



