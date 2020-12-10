---
title: Static Provisioning
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Introduction

In the case of Kubernetes `Volumes`, once the `Pod` is deleted the specification of the volume in the `Pod` is also lost. Even though `VMDK` file persists, but from Kubernetes perspective the volume is deleted.

The `PersistentVolumes` API solves this problem where `PVs` have a lifecycle independent of the `Pods` and are not dependant on a `Pod` to persist. `PVs` are units of storage provisioned in advance, they are Kubernetes objects backed by vSphere storage. `PVs` are created and deleted using `kubectl` commands.

In order to use these `PVs` the user needs to create `PersistentVolumeClaims` which is simply a request for some storage. A `PVC` must specify the `access mode` and `storage capacity`, once a `PVC` is created a `PV` is automatically bound to this claim. Kubernetes will bind a `PV` to `PVC` based on the `access mode` and `storage capacity`. The `PVC` can also mention `volume name`, `selectors` and `volume class` for a better match.

This design of `PV` to `PVC` mappings not only abstracts storage provisioning and consumption but also ensures security through access control.

## Manually provisioning volumes

Here is an example of how to use `PVs` and `PVCs` to add persistent storage to your `Pods`.

### Create vSphere VMDK

#### Via govc (preferred)

Requests a 2GB `VMDK` be created on the `datastore1` datastore in a folder called `volumes` with a name of `myDisk.vmdk`:

```sh
govc datastore.disk.create -ds datastore1 -size 2G volumes/myDisk.vmdk
```

#### Via ESXi CLI

SSH into an ESXi host as `root` and use following command to create a 2GB `VMDK` (this assumes your vSphere datastore is called `datastore1` and you want to create the `VMDK` in the folder `volumes`):

```sh
vmkfstools -c 2G /vmfs/volumes/datastore1/volumes/myDisk.vmdk
```

## Create a PersistentVolume

### Define the PersistentVolume

As you can see below, we call the `PV` "`pv0001`" and pass in the `VMDK` path to the `volumePath` parameter.

```yaml
$ cat vsphere-volume-pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  vsphereVolume:
    volumePath: "[datastore1] volumes/myDisk"
    fsType: ext4
```

In the above example, `datastore1` is located in the root folder of vCenter. If the datastore is the member of a `Datastore Cluster` or located in subfolder, then the folder path needs to be provided in the `volumePath` as shown below.

```yaml
vsphereVolume:
  volumePath: "[DatastoreCluster/datastore1] volumes/myDisk"
```

### Import the PersistentVolume

Import the `PersistentVolume` definition we created above into our Kubernetes cluster using `kubectl`:

```sh
kubectl create -f vsphere-volume-pv.yaml
```

### Verify the PersistentVolume

Verify the `PV` we imported above was created correctly:

```sh
$ kubectl describe pv pv0001
Name:		pv0001
Labels:		<none>
Status:		Available
Claim:
Reclaim Policy:	Retain
Access Modes:	RWO
Capacity:	2Gi
Message:
Source:
    Type:	vSphereVolume (a Persistent Disk resource in vSphere)
    VolumePath:	[datastore1] volumes/myDisk
    FSType:	ext4
No events.
```

## Create a PersistentVolumeClaim

### Define the PersistentVolumeClaim

Create a `PVC` to consume the `PV` created above - in this example the `PVC` simply requests 2GB of storage:

```yaml
$ cat vsphere-volume-pvc.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc0001
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

### Import the PersistentVolumeClaim

Import the `PVC` definition from above into our Kubernetes cluster:

```sh
kubectl create -f vsphere-volume-pvc.yaml
```

### Verify the PersistentVolumeClaim

Ensure the `PVC` we created has been assigned a `PV` - the `Status` should show `Bound` and the `Volume` field should show the `PV` name from above, in this case `pv0001`.

```sh
$ kubectl describe pvc pvc0001
Name:		pvc0001
Namespace:	default
Status:		Bound
Volume:		pv0001
Labels:		<none>
Capacity:	2Gi
Access Modes:	RWO
No events.
```

## Create a Pod using the PVC

### Define the Pod

Create a simple `Pod` the consumes the `PVC` created above by name `pvc0001`:

```yaml
$ cat vsphere-volume-pvcpod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pvpod
spec:
  containers:
  - name: test-container
    image: gcr.io/google_containers/test-webserver
    volumeMounts:
    - name: test-volume
      mountPath: /test-vmdk
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: pvc0001
```

### Import the Pod

Import the `Pod` definition from above into our Kubernetes cluster:

```sh
kubectl create -f vsphere-volume-pvcpod.yaml
```

### Verify the Pod

Make sure the `PV` was bound to the pod correctly by ensuring the `STATUS` is set to `Running`:

```sh
$ kubectl get pod pvpod
NAME      READY     STATUS    RESTARTS   AGE
pvpod       1/1     Running   0          48m
```

You have successfully manually provisioned a Kubernetes volume on vSphere.
