---
title: Volumes
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---
A Pod can specify vsphereVolume as Kubernetes Volumes and then vSphere VMDK is mounted as Volume into your Pod. The contents of a volume are preserved when it is unmounted. It supports both VMFS and VSAN datastores.

All the example yamls can be found [here](https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere) unless otherwise specified. Please download these examples.

Here is an example of how to create a VMDK file and how a Pod can use it.

**Create VMDK**

First ssh into ESX and then use following command to create vmdk on datastore1

```
vmkfstools -c 2G /vmfs/volumes/datastore1/volumes/myDisk.vmdk
```

Or use govc:

```
govc datastore.disk.create -ds datastore1 -size 2G volumes/myDisk.vmdk
```

**Create Pod which uses vSphere Volume 'myDisk.vmdk'**

```
#vsphere-volume-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test-vmdk
spec:
  containers:
  - image: gcr.io/google_containers/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-vmdk
      name: test-volume
  volumes:
  - name: test-volume
    # This VMDK volume must already exist.
    vsphereVolume:
      volumePath: "[datastore1] volumes/myDisk.vmdk"
      fsType: ext4

```

**Create the pod**

```
$ kubectl create -f vsphere-volume-pod.yaml
```

**Verify that pod is running**

```
$ kubectl get pods test-vmdk
NAME      READY     STATUS    RESTARTS   AGE
test-vmdk   1/1     Running   0          48m
```
