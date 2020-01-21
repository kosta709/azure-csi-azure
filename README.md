# CSI Demo

### Start cluster
```
RESOURCE_GROUP=demo-csi
az group create --name $RESOURCE_GROUP --location westcentralus
aks-engine generate --output-directory aks/ -m aks/param-model.json --debug
az group deployment create --name demo-csi-$(date +"%F-%H%M%S") -g $RESOURCE_GROUP --template-file aks/azuredeploy.json --parameters aks/azuredeploy.parameters.json
```

### Cloud Controller manager

https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/
https://github.com/kubernetes-sigs/cloud-provider-azure

Old code is in https://github.com/kubernetes/kubernetes/tree/master/staging/src/k8s.io/legacy-cloud-providers 

```
[kosta@localhost demo-csi-azure]$ kubectl get pods -A | grep cloud
kube-system   cloud-controller-manager-k8s-master-17068304-0       1/1     Running   0          109m
kube-system   cloud-node-manager-b2f7w                             1/1     Running   0          110m
kube-system   cloud-node-manager-c9mlg                             1/1     Running   0          110m
kube-system   cloud-node-manager-nzl2b                             1/1     Running   0          110m
```

```
kubectl -nkube-system get pods  cloud-controller-manager-k8s-master-17068304-0 -oyaml
...
  - args:
    - --allocate-node-cidrs=false
    - --cloud-config=/etc/kubernetes/azure.json
    - --cloud-provider=azure
    - --cluster-cidr=10.240.0.0/12
    - --cluster-name=demo-csi-1
    - --configure-cloud-routes=false
    - --controllers=*
    - --kubeconfig=/var/lib/kubelet/kubeconfig
    - --leader-elect=true
    - --route-reconciliation-period=10s
    - --v=2
...
  volumes:
  - hostPath:
      path: /etc/kubernetes
      type: ""
    name: etc-kubernetes
  - hostPath:
      path: /etc/ssl
      type: ""
    name: etc-ssl
  - hostPath:
      path: /var/lib/kubelet
      type: ""
    name: var-lib-kubelet
  - hostPath:
      path: /var/lib/waagent/ManagedIdentity-Settings
      type: ""
    name: msi
```


Create pod with load balancer and see logs + see lb on azure
```
kubectl create -f cloud-controller-example/
```

### CSI 

https://github.com/kubernetes-sigs/azuredisk-csi-driver
https://medium.com/@kosta709/kubernetes-csi-in-action-d5cf59857eee


CSI API Resources  
```
[kosta@localhost Internal shared storage]$ kubectl api-resources | grep -E "^NAME|csi|storage|PersistentVolume"
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
csidrivers                                     csi.storage.k8s.io             false        CSIDriver
csinodeinfos                                   csi.storage.k8s.io             false        CSINodeInfo
volumesnapshotclasses                          snapshot.storage.k8s.io        false        VolumeSnapshotClass
volumesnapshotcontents                         snapshot.storage.k8s.io        false        VolumeSnapshotContent
volumesnapshots                                snapshot.storage.k8s.io        true         VolumeSnapshot
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment


[kosta@localhost Internal shared storage]$ kubectl get crd | grep storage
csidrivers.csi.storage.k8s.io                    2020-01-21T11:54:26Z
csinodeinfos.csi.storage.k8s.io                  2020-01-21T11:54:26Z
volumesnapshotclasses.snapshot.storage.k8s.io    2020-01-21T11:54:52Z
volumesnapshotcontents.snapshot.storage.k8s.io   2020-01-21T11:54:52Z
volumesnapshots.snapshot.storage.k8s.io          2020-01-21T11:54:52Z
```

CSI Pods  
```
kubectl get pods -A | grep 
```

