# nfs-ganesha-server-and-external-provisioner
This project is about the use and configuration of the NFS Server Provisioner.

## Reference
https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner/blob/master/README.md 

## Install
1. Create a namespace for the nfs-server-provisioner controller
```console
kubectl create namespace nfs-server-provisioner
```

2. Install NFS service on the nodes
```console
sudo apt-get update && sudo apt-get install -y nfs-common
```

3. Create the required PVs and PVCs for the replica set  
Manually create PV and PVC corresponding to the name of nfs-provisioner pod on the nodes of the cluster, create corresponding PV and PVC for each of the nfs-provisioner pod replicas you have.
```console
# Create specified PV and PVC for nfs-provisioner-0 pod on node1
kubectl apply -f ./pv-0.yaml
kubectl apply -f ./pvc-0.yaml -n nfs-server-provisioner
```

4. Add the repository of project  
```console
helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/  
helm repo update  
helm search repo nfs-ganesha-server-and-external-provisioner --versions
```
5. Select a version and export the file 'values.yaml' to the current directory  
```console
helm show values nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner --version 1.8.0 > ./values.yaml
```

6. Installing the Chart

```console
helm install nsp nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner --version 1.8.0 -n nfs-server-provisioner -f ./values.yaml
```

Replace Docker images that cannot be pulled
```console
docker pull k8s.nju.edu.cn/sig-storage/nfs-provisioner:v4.0.8
docker tag k8s.nju.edu.cn/sig-storage/nfs-provisioner:v4.0.8 registry.k8s.io/sig-storage/nfs-provisioner:v4.0.8
```

### Upgrade
```console
helm upgrade nsp nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner -n nfs-server-provisioner -f ./values.yaml
```

### Uninstall
helm uninstall nsp -n nfs-server-provisioner

### Cross cluster usage
Examples in the reference directory 'cross-cluster-example'
