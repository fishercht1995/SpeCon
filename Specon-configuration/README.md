

##### Build Persistent Volumes 
build PV and PVC in both default namespace and kube-system namespace. Templates are shown in pv-build folder
##### Add roles to pods
modify clusterrole system:kube-scheduler and add pod create to our own scheduler
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
##### config roles and serve account
```buildoutcfg
kubectl apply -f role.yaml
```
##### build NFS
build nfs system and create a shared folder as working directionary.
##### initialize SpeCon migration pod
modify node name in mig.yaml and deploy mig pod
```buildoutcfg
kubectl apply -f mig.yaml
```
If mig pod deployed successfully, mig pod should be in the kube-system namespace
```
kubectl get pods --namespace=kube-system

kube-system   kube-proxy-ct4q8                                                       1/1     Running       0          39m
kube-system   kube-proxy-k44wb                                                       1/1     Running       0          26m
kube-system   kube-scheduler-node-0.cluster002.flsys-pg0.utah.cloudlab.us            1/1     Running       0          39m
kube-system   mig                                                                    2/2     Running       0          36s
kube-system   weave-net-9hlrm                                                        2/2     Running       1          39m
kube-system   weave-net-vss7x                                                        2/2     Running       1          26m
```

