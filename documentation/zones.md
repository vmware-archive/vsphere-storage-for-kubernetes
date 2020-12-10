---
title: Zone support in vSphere Cloud Provider
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Introduction

In the vSphere infrastructure, Kubernetes nodes are run as VMs on an ESX host and Volumes are created as VMDK files on a datastore. The Volume is always placed on a datastore that is shared among all the node VMs of the Kubernetes cluster. This is done so that the Volume can be attached to a pod scheduled on any of the node VMs. However, this implies that there should be a shared datastore among the node VMs in vSphere infrastructure to create Volumes. It also implies that the datastore capacity of any other non-shared datastores cannot be utilized for creating Volumes.

With the introduction of zones in Kubernetes, the cloud provider can express the topology of the cloud infrastructure as zones. Kubernetes nodes and Volumes have labels denoting the zone information. The Kubernetes pod scheduler is zone aware in placing pods on nodes in the required zone.

vSphere Cloud Provider added Zones support since Kubernetes version 1.14.

## vCenter Setup

### Tag Zones and Regions in vCenter

Zones and Regions are marked by creating and applying Tags in vCenter. The Tags can be applied to a Datacenter, Cluster or ESX Host. Tags are inherited from the parent and can be overridden on the child in this hierarchy. So a Tag applied to a Datacenter is inherited by all Clusters and Hosts in that Datacenter, and a Tag applied to a Cluster overrides any Tags that it might inherit from its parent Datacenter.

See the vCenter documentation [here](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vcenterhost.doc/GUID-E8E854DD-AA97-4E0C-8419-CE84F93C4058.html) for details on how to create and associate Tags.

Let us say we have a vCenter inventory with two Clusters. To mark the two clusters as two zones, "zone-a" and "zone-b", and to mark the entire Datacenter as a region, "vc1-region", the steps would look like this.

```text
Datacenter (vc1-region)
    |
    |-- Cluster-1 (zone-a)
         |-- Host-1
               |-- k8s-master
         |-- Host-2
               |-- k8s-node-1
               |-- hostLocalDatastore-2
         |-- Host-3
               |-- k8s-node-2
         |-- vsanDatastore-1
    |-- Cluster-2 (zone-b)
         |-- Host-4
               |-- k8s-node-3
         |-- Host-5
               |-- k8s-node-4
         |-- Host-6
         |-- vsanDatastore-2
    |-- sharedVMFSDatastore
```

1. Create two tag categories, say "k8s-zone" and "k8s-region"
2. Create zone tags, say "zone-a" and "zone-b" in the "k8s-zone" tag category
3. Create region tags, say "vc1-region" in the "k8s-region" tag category
4. Apply the "vc1-region" tag to the Datacenter
5. Apply the "zone-a" tag to Cluster-1 and "zone-b" tag to Cluster-2.

This completes the steps required to mark zones in vCenter. The second part is to configure each Kubernetes node to look for these zone labels at startup.

## Kubernetes Setup

Add the zone and region vSphere `Tag Category` names in `vsphere.conf` of each Kubernetes node. Note that the value is the `Tag Category` names, "k8s-region" and "k8s-zone" and not the actual `Tag` names, the tag names themselves are: `vc1-region`, `zone-a` or `zone-b`.

```sh
/etc/kubernetes/vsphere.conf

[Labels]
    region = "k8s-region"
    zone = "k8s-zone"
```

* Restart all the services on Kubernetes master.
* Restart kubelet on all the Kubernetes nodes.

The Kubernetes nodes now have labels showing the zone to which it belongs.

```sh
$ kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\tregion: "}{.metadata.labels.failure-domain\.beta\.kubernetes\.io/region}{"\tzone: "}{.metadata.labels.failure-domain\.beta\.kubernetes\.io/zone}{"\n"}{end}'

k8s-master	region: vc1-region	zone: zone-a
k8s-node1	region: vc1-region	zone: zone-a
k8s-node2	region: vc1-region	zone: zone-a
k8s-node3	region: vc1-region	zone: zone-b
k8s-node4	region: vc1-region	zone: zone-b
```

**Note:**

The [Labels] section with region/zone entries in `vsphere.conf` file acts as a feature flag for zone support in vSphere Cloud Provider. If these entries are not present in the `vsphere.conf` file then vSphere Cloud Provider does not recognize Zones in vCenter.

## Zone for Volume

When a Volume is created in such an environment it gets automatically labelled with the zone information. The labels specify all the zones from which a pod can access the Volume.

In the above sample vCenter inventory, when a Volume is created on vsanDatastore-2, it gets the "zone-b" label associated with it.

```yaml
$ cat vsphere-volume-pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: vol-1
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  vsphereVolume:
    volumePath: "[vsanDatastore-2] volumes/myDisk.vmdk"
    fsType: ext4
```

```sh
kubectl create -f vsphere-volume-pv.yaml
```

**Note** the "zone-b" label applied to the Persistent Volume

```sh
$ kubectl describe pv vol-1
Name:       vol-1
Labels:	    failure-domain.beta.kubernetes.io/region=vc1-region
            failure-domain.beta.kubernetes.io/zone=zone-b
Status:     Available
Claim:
Reclaim Policy:	Retain
Access Modes:	RWO
Capacity:	2Gi
Message:
Source:
    Type:	vSphereVolume (a Persistent Disk resource in vSphere)
    VolumePath:	[vsanDatastore-2] volumes/myDisk.vmdk
    FSType:	ext4
```

A [dynamically created Volume](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/storageclass.html) in such an environment will always be placed on a datastore that is shared across all the Kubernetes nodes. In the sample vCenter inventory shown above, the Volume is placed on sharedVMFSDatastore. Since the Volume is visible for pods in both the zones, it gets the zone label of "zone-a__zone-b" as shown here.

```yaml
$ cat vsphere-volume-sc-fast.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/vsphere-volume
parameters:
  diskformat: thin
  fstype: ext3
```

```sh
kubectl create -f vsphere-volume-sc-fast.yaml
```

```yaml
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

```sh
kubectl create -f vsphere-volume-pvcsc.yaml
```

```sh
$ kubectl describe pvc pvcsc001
Name:           pvcsc001
Namespace:      default
StorageClass:   fast
Status:         Bound
Volume:         pvc-1ee83e2b-4b9b-11e9-ab0c-0050569a14a9
Labels:         <none>
Capacity:       2Gi
Access Modes:   RWO
No events.
```

**Note** the "zone-a__zone-b" label for the dynamically created volume

```sh
$ kubectl describe pv pvc-1ee83e2b-4b9b-11e9-ab0c-0050569a14a9
Name:		pvc-1ee83e2b-4b9b-11e9-ab0c-0050569a14a9
Labels:		failure-domain.beta.kubernetes.io/region=vc1-region
            failure-domain.beta.kubernetes.io/zone=zone-a__zone-b
Status:		Bound
Claim:      default/pvcsc001
Reclaim Policy:	Delete
Access Modes:	RWO
Capacity:	2Gi
Message:
Source:
    Type:	vSphereVolume (a Persistent Disk resource in vSphere)
    VolumePath:	[sharedVMFSDatastore] kubevols/k8s-dynamic-pvc-1ee83e2b-4b9b-11e9-ab0c-0050569a14a9.vmdk
    FSType:	ext4
```

## Specifying the Zone for a Volume

When creating a Volume dynamically, the required zone for the Volume can be specified as `allowedTopologies` in a Storage Class.

```yaml
$ cat dynamic.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc_zone_a
provisioner: kubernetes.io/vsphere-volume
parameters:
    diskformat: thin
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - zone-a
```

When a Persistent Volume is created based on this storage class, it is placed on a datastore that is accessible to all nodes in zone-a. In the sample vCenter inventory shown above, this Volume gets placed on vsanDatastore-1.

```sh
$ kubectl describe pv pvc-3264d346-4b9f-11e9-ab0c-0050569a14a9
Name:		pvc-3264d346-4b9f-11e9-ab0c-0050569a14a9
Labels:		failure-domain.beta.kubernetes.io/region=vc1-region
            failure-domain.beta.kubernetes.io/zone=zone-a
Annotations:	kubernetes.io/createdby=vsphere-volume-dynamic-provisioner
		pv.kubernetes.io/bound-by-controller=yes
		pv.kubernetes.io/provisioned-by=kubernetes.io/vsphere-volume
StorageClass:	sc_zone_a
Status:		Bound
Claim:      default/pvc-3
Reclaim Policy:	Delete
Access Modes:	RWO
Capacity:	1Mi
Message:
Source:
    Type:	vSphereVolume (a Persistent Disk resource in vSphere)
    VolumePath:	[vsanDatastore-1] e4d0895c-3a18-ef1e-cbee-020037a3a334/k8s-dynamic-pvc-3264d346-4b9f-11e9-ab0c-0050569a14a9.vmdk
    FSType:	ext4
```

**Note:**
In this example, the Volume could also get placed on sharedVMFSDatastore since it will still be accessible to all nodes in zone-a.

**Note:**
When a pod that uses such a Persistent Volume Claim is created, the Kubernetes pod controller automatically schedules it on one of the Kubernetes nodes in the Volume's zone. This is described more in the Kubernetes documentation of [topology aware scheduling.](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/#topology-awareness)
