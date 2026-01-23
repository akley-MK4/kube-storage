# Local Path Provisioner
This project is about the use and configuration of the Local Path Provisioner.
Compared to Kubernetes 'Local Volume provisioner', it can dynamically create PV.

## Reference
https://github.com/rancher/local-path-provisioner

## Install
1. Create the local-path-storage namespace and install the provisioner  
Download the official stable version and install it.
```console
wget https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.34/deploy/local-path-storage.yaml
kubectl apply -f local-path-storage.yaml
```

2. Set the paths of the volumes for the nodes of cluster in the local-path-storage.yaml file
```json
{
    "nodePathMap":[
        {
            // Set the path of the volume for unspecified nodes
            "node":"DEFAULT_PATH_FOR_NON_LISTED_NODES",
            "paths":["/mnt/local-storage-default"]
        },
        {
            // Set up the paths to the two mounted disks on node1
            "node":"node1",
            "paths":["/mnt/ssd", "/mnt/hdd"]
        }
    ]
}
```

3. Adjust other configurations as needed, such as adjusting StorageClass in the local-path-storage.yaml file
To use different mounting volume paths, you can add different StorageClass in the yaml file.
```yaml
# Set this StorageClass as a default StorageClass for the cluster
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path-default
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer #WaitForFirstConsumer, Immediate
reclaimPolicy: Retain   #Retain,Delete
allowVolumeExpansion: true
```

```yaml
# Using the PVC of this StorageClass will create a PV on node1, which is based on the path '/mnt/ssd/{{ .PVC.Namespace }}/{{ .PVC.Name }}/' of the node, and the pod that mounts the PVC will be scheduled to node1.
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer #WaitForFirstConsumer, Immediate
reclaimPolicy: Retain   #Retain,Delete
allowVolumeExpansion: true
parameters:
  nodePath: "/mnt/ssd"
  pathPattern: "{{ .PVC.Namespace }}/{{ .PVC.Name }}/"
```

### Upgrade
n/a

### Uninstall
```console
kubectl delete -f local-path-storage.yaml
```

### Usage


#### Use the provision's StorageClass to dynamically create PV.
1. Use the yaml file below to create a PVC that references a StorageClass from the local-path-storage.yaml file.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-local-path
  labels:
    type: local
  annotations: {}
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  # The name needs to be consistent with the name of StorageClass in the local-path-storage.yaml file
  # local-path-default: Use any configured volume on any node
  # local-path-ssd:  Use SSD Volume on Node1
  storageClassName: local-path-default
```