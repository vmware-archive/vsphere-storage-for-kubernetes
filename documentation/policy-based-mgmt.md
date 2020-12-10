---
title: Dynamic Provisioning
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Overview

One of the most important features of vSphere is `Storage Policy based Management` (`SPBM`). `SPBM` provides a single unified control plane across a broad range of data services and storage solutions. `SPBM` enables vSphere administrators to overcome upfront storage provisioning challenges, such as capacity planning, differentiated service levels and managing capacity headroom.

Kubernetes `StorageClasses` allow the creation of `PersistentVolumes` on-demand without having to create storage and mount it into K8s nodes upfront. `StorageClasses` specifiy a `provisioner` and `parameters` which are used to define the intended policy for a `PersistentVolume` which will be dynamically provisioned.

Existing `SPBM` policies can be used to configure a `PersistentVolume` with a specific policy via the `storagePolicyName` parameter.

## Kubernetes StorageClass

### VMFS and NFS

The admin specifies the SPBM policy - `gold` as part of `StorageClass` definition for dynamic volume provisioning. When a `PVC` is created, the `PersistentVolume` will be provisioned on a compatible datastore with the most free space that satisfies the `gold` storage policy requirements.

```yaml
$ cat vsphere-volume-spbm-policy.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
    diskformat: zeroedthick
    storagePolicyName: gold
```

The admin can also specify a custom datastore where he wants the volume to be provisioned along with the `SPBM` policy name. When a `PVC` is created, the vSphere Cloud Provider checks if the user specified datastore satisfies the `gold` storage policy requirements. If it does, the vSphere Cloud Provider will provision the `PersistentVolume` on the user specified datastore. If not, it will create an error telling the user that the specified datastore is not compatible with `gold` storage policy requirements.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
    diskformat: zeroedthick
    storagePolicyName: gold
    datastore: Datastore-123
```

### vSAN

The Kubernetes user will have the ability to specify custom vSAN Storage Capabilities during dynamic volume provisioning. K8s admins can now define storage requirements, such as performance and availability, in the form of storage capabilities during dynamic volume provisioning. The storage capability requirements are converted into a vSAN policy which is then pushed down to the virtual disk layer when a `PersistentVolume` (virtual disk) is created. The virtual disk is distributed across the vSAN datastore to meet the policy requirements.

The official [vSAN policy documentation](https://www.youtube.com/watch?v=e0wkMPDvKPQ) describes in detail about each of the individual storage capabilities that are supported by vSAN. The user can specify these storage capabilities as part of `StorageClass` definition based on his application needs.

For vSAN policies, a few additional parameters can be specified in the `StorageClass` if desired or you can use a pre-existing policy by simply specifying `storagePolicyName` as above:

* `cacheReservation:` Flash capacity reserved as read cache for the container object. Specified as a percentage of the logical size of the virtual machine disk (vmdk) object. Reserved flash capacity cannot be used by other objects. Unreserved flash is shared fairly among all objects. Use this option only to address specific performance issues.
* `diskStripes:` The minimum number of capacity devices across which each replica of a object is striped. A value higher than 1 might result in better performance, but also results in higher use of system resources. Default value is 1. Maximum value is 12.
* `forceProvisioning:` If the option is set to Yes, the object is provisioned even if the Number of failures to tolerate, Number of disk stripes per object, and Flash read cache reservation policies specified in the storage policy cannot be satisfied by the datastore
* `hostFailuresToTolerate:` Defines the number of host and device failures that a virtual machine object can tolerate. For n failures tolerated, each piece of data written is stored in n+1 places, including parity copies if using RAID 5 or RAID 6.
* `iopsLimit:` Defines the IOPS limit for an object, such as a VMDK. IOPS is calculated as the number of I/O operations, using a weighted size. If the system uses the default base size of 32 KB, a 64-KB I/O represents two I/O operations
* `objectSpaceReservation:` Percentage of the logical size of the virtual machine disk (vmdk) object that must be reserved, or thick provisioned when deploying virtual machines. Default value is 0%. Maximum value is 100%.

#### vSAN StorageClass

In this example `PersistentVolumes` will be created via the `StorageClass` using the SPBM policy: `vSAN Default Storage Policy` which is already created within vCenter:

```yaml
$ cat vsphere-volume-sc-policyname.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
    storagePolicyName: "vSAN Default Storage Policy"
    datastore: vsanDatastore
```

Here, `PersistentVolumes` will be created with the vSAN capabilities - `hostFailuresToTolerate` set to `2` and `cachereservation` is `20%` - even if no `SPBM` policy exists within vCenter.

```yaml
$ cat vsphere-volume-sc-vsancapabilities.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
    hostFailuresToTolerate: "2"
    cachereservation: "20"
```

The K8s user can also specify the datastore in the `StorageClass` as shown in the below example. The volume will be created on the datastore specified in the `StorageClass`. This field is optional, if not specified as shown in the above example, the volume will be created on the datastore specified in the `vsphere.conf` config file used to [initialize the vSphere Cloud Provider](/vsphere-storage-for-kubernetes/documentation/existing.html).

```yaml
$ cat vsphere-volume-sc-vsancapabilities.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
    datastore: vsanDatastore
    hostFailuresToTolerate: "2"
    cachereservation: "20"
```

_Note: If the storage policy is not set during dynamic provisioning on a vSAN datastore, default vSAN SPBM policy is applied to the volume._

## Create a StorageClass

### Define the StorageClass

```yaml
$ cat vsphere-volume-sc-vsancapabilities.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
    datastore: vsanDatastore
    hostFailuresToTolerate: "2"
    cachereservation: "20"
```

### Import the StorageClass

Import the `StorageClass` definition into Kubernetes using `kubectl`:

```sh
kubectl create -f vsphere-volume-sc-vsancapabilities.yaml
```

### Verify the StorageClass is created

Make sure that the `StorageClass` we just created was received correctly and has the correct `parameters` set:

```sh
$ kubectl describe storageclass fast
Name:		fast
Annotations:	<none>
Provisioner:	kubernetes.io/vsphere-volume
Parameters:	hostFailuresToTolerate="2", cachereservation="20"
No events.
```

## Create a PersistentVolumeClaim

### Define the PersistentVolumeClaim

See below, we are creating a `PVC` called `pvcsc-vsan` and etting the `volume.beta.kubernetes.io/storage-class` key to `fast` so it will use the `StorageClass` we just created and we are requesting a `2Gi` (2GB) `PersistentVolume`.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvcsc-vsan
  annotations:
    volume.beta.kubernetes.io/storage-class: fast
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

### Import the PersistentVolumeClaim

Import the `PVC` definition from above into Kubernetes using `kubectl`:

```sh
kubectl create -f vsphere-volume-pvcsc.yaml
```

### Verify the PersistentVolumeClaim was created

Check to see if the `PVC` we just imported was created and has a `PersistentVolume` attached to it. The `Status` section below should show `Bound` if it worked and the `Volume` field should be populated. A `PersistentVolume` is automatically created and is bound to this `PVC`.

```sh
$ kubectl describe pvc pvcsc-vsan
Name:		pvcsc-vsan
Namespace:	default
Status:		Bound
Volume:		pvc-80f7b5c1-94b6-11e6-a24f-005056a79d2d
Labels:		<none>
Capacity:	2Gi
Access Modes:	RWO
No events.
```

### Verify a PersistentVolume was created

Next, let's check if a `PV` was successfully created for the `PVC` we defined above. If it has worked you should have a `PV` show up in the output and you should see that the `VolumePath` key is populated, as is the case below and the `Status` should say `Bound`. You can also see the `Claim` is set to the above `PVC` name `pvcsc-vsan`.

```sh
$ kubectl describe pv
Name:		pvc-80f7b5c1-94b6-11e6-a24f-005056a79d2d
Labels:		<none>
Status:		Bound
Claim:		default/pvcsc-vsan
Reclaim Policy:	Delete
Access Modes:	RWO
Capacity:	2Gi
Message:
Source:
    Type:	vSphereVolume (a Persistent Disk resource in vSphere)
    VolumePath:	[VSANDatastore] kubevols/kubernetes-dynamic-pvc-80f7b5c1-94b6-11e6-a24f-005056a79d2d.vmdk
    FSType:	ext4
No events.
```

_VMDKs are created inside a folder named `kubevols` in the datastore which is mentioned in the `vsphere.conf` vSphere Cloud Provider configuration file. The vSphere Cloud Provider config file is created [during setup](/vsphere-storage-for-kubernetes/documentation/existing.html) of Kubernetes cluster on vSphere._

## Define a Pod using the PVC

Below is a `Pod` config that will use the "`pvcsc-vsan`" `PVC` that was created above, and thereby the `PV` associated with it.

```sh
$ cat vsphere-volume-pvcscpod.yaml

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
      mountPath: /test
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: pvcsc-vsan
```

### Create the Pod

Import the `Pod` definition using `kubectl`:

```sh
kubectl create -f vsphere-volume-pvcscpod.yaml
```

### Verify the Pod is created

```sh
$ kubectl get pod pvpod
NAME      READY     STATUS    RESTARTS   AGE
pvpod       1/1     Running   0          48m
```

You are now successfully creating `PersistentVolumes` on demand with `StorageClasses` on vSphere.
