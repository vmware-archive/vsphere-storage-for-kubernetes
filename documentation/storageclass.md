---
title: Dynamic Provisioning and StorageClass API
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

With PV and PVCs one can only provision storage statically i.e. PVs first needs to be created before a Pod claims it. However, with StorageClass API Kubernetes enables dynamic volume provisioning. This avoids pre-provisioning of storage and storage is provisioned automatically when a user requests it.

StorageClass API object specifies a provisioner and parameters  which are used to decide which volume plugin to be used and provisioner specific parameters.
Provisioner could be AWS EBS, vSphere, OpenStack and so on.

vSphere is one of the provisioners and it allows following parameters:

* **diskformat** which can be thin(default), zeroedthick and eagerzeroedthick

* **datastore** is an optional field which can be VMFSDatastore or VSANDatastore. This allows user to select the datastore to provision PV from, if not specified the default datastore from vSphere config file is used.

* **storagePolicyName** is an optional field which is the name of the SPBM policy to be applied. The newly created persistent volume will have the SPBM policy configured with it.

* **VSAN Storage Capability Parameters** (cacheReservation, diskStripes, forceProvisioning, hostFailuresToTolerate, iopsLimit and objectSpaceReservation) are supported by vSphere provisioner for vSAN storage. The persistent volume created with these parameters will have these vSAN storage capabilities configured with it. For more detail about these parameters go to [Storage Policy Management section](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/policy-based-mgmt.html).

**Note:**

All the example yamls can be found [here](https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere) unless otherwise specified. Please download these examples.

Let us look at an example of how to use StorageClass for dynamic provisioning.

**Note:** With StorageClass, VMDK need not be created manually for Persistent Volume. VMDK will be created dynamically by `vsphere-volume` provisioner.

**Create Storage Class**

```
$ cat vsphere-volume-sc-fast.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
  datastore: VSANDatastore
  diskformat: thin
  fstype: ext3
```

For Kubernetes version 1.8.X or older, if datastore is a member of datastore Cluster or within some subfolder, the datastore folder path needs to be provided in the datastore parameter as below.

```
   datastore:	DatastoreCluster/VSANDatastore
```
For Kubernetes version 1.9.X datastore name is sufficient to identify the datastore.

**Create the Storageclass**

```
$ kubectl create -f examples/volumes/vsphere/vsphere-volume-sc-fast.yaml
```

**Verify storage class is created**

```
$ kubectl describe storageclass fast
Name:           fast
IsDefaultClass: No
Annotations:    <none>
Provisioner:    kubernetes.io/vsphere-volume
Parameters:     diskformat=thin,fstype=ext3,datastore=VSANDatastore
No events.
```

**Create Persistent Volume Claim**

```
$ cat vsphere-volume-pvcsc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvcsc001
  annotations:
    volume.beta.kubernetes.io/storage-class: fast
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

**Create the persistent volume claim**

```
$ kubectl create -f examples/volumes/vsphere/vsphere-volume-pvcsc.yaml
```

**Verify persistent volume claim is created**

```
$ kubectl describe pvc pvcsc001
Name:           pvcsc001
Namespace:      default
StorageClass:   fast
Status:         Bound
Volume:         pvc-83295256-f8e0-11e6-8263-005056b2349c
Labels:         <none>
Capacity:       2Gi
Access Modes:   RWO
Events:
  FirstSeen LastSeen Count  From        SubObjectPath   Type  Reason Message
  -----------------------------------------------------------
  1m          1m      1   persistentvolume-controller  Normal  Provisioning Succeeded

  Successfully provisioned volume pvc-83295256-f8e0-11e6-8263-005056b2349c using kubernetes.io/vsphere-volume
```

Persistent Volume is automatically created and is bounded to this pvc.

**Verify persistent volume is created**

```
$ kubectl describe pv pvc-83295256-f8e0-11e6-8263-005056b2349c
Name:           pvc-83295256-f8e0-11e6-8263-005056b2349c
Labels:         <none>
StorageClass:   fast
Status:         Bound
Claim:          default/pvcsc001
Reclaim Policy: Delete
Access Modes:   RWO
Capacity:       2Gi
Message:
Source:
    Type:       vSphereVolume (a Persistent Disk resource in vSphere)
    VolumePath: [VSANDatastore] kubevols/kubernetes-dynamic-pvc-83295256-f8e0-11e6-8263-005056b2349c.vmdk
    FSType:     ext3
No events.
```

**Note:** VMDK is created inside kubevols folder in datastore which is mentioned in 'vsphere' cloudprovider configuration. The cloudprovider config is created during setup of Kubernetes cluster on vSphere.

**Create Pod which uses Persistent Volume Claim with storage class.**

```
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
      mountPath: /test-vmdk
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: pvcsc001
```

**Create the pod**

```
$ kubectl create -f vsphere-volume-pvcscpod.yaml
```

**Verify pod is created**

```
$ kubectl get pod pvpod
NAME      READY     STATUS    RESTARTS   AGE
pvpod       1/1      Running   0          48m
```
