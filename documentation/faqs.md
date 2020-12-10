---
title: FAQs
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

{{site.data.alerts.warning}}Following FAQs are only applicable to in-tree vSphere Cloud Provider and in-tree vSphere Volume Plugin. For information about vSphere CSI Driver please visit https://vsphere-csi-driver.sigs.k8s.io/ {{site.data.alerts.end}}

## What is the biggest Kubernetes cluster it has been tested for?
It has been tested on 1000 node Kubernetes cluster so far. Please refer to this [section](/vsphere-storage-for-kubernetes/documentation/maximum-scale-limit.html) for details about recommended configuration.

## Can vSphere Cloud Provider support Kubernetes Cluster spanning across multiple vCenters, ESXi Clusters and Datacenters?
With Kubernetes version 1.9.x vSphere cloud provider supports Kubernetes cluster spanning across multiple ESXi clusters, vSphere Datacenters and vCenters. All Kubernetes node VMs must have access to shared storage.

## Can Kubernetes Cluster span across multiple vCenters?
With Kubernetes version 1.9.x vSphere cloud provider supports Kubernetes cluster spanning across multiple ESXi clusters, vSphere Datacenters and vCenters. All Kubernetes node VMs must have access to shared storage.

## Which Roles and Privileges are required for the vCenter User, used by vSphere Cloud Provider?
 Please refer to this [section](/vsphere-storage-for-kubernetes/documentation/existing.html#create-roles-add-privileges-to-roles-and-assign-them-to-the-vsphere-cloud-provider-user-and-vsphere-entities).

## Which vSphere Datastore types are supported by vSphere Cloud Provider?
VMFS, NFS, vSAN and VVOl datastore types are supported by vSphere Cloud Provider.

## Can SPBM be used with dynamic volume provisioning?
Please refer to this [section](/vsphere-storage-for-kubernetes/documentation/policy-based-mgmt.html).
Also refer to [Storage Policy Management inside kubernetes](https://github.com/kubernetes/examples/blob/master/staging/volumes/vsphere/README.md#storage-policy-management-inside-kubernetes) for more details.

## Can single node cluster deployed on laptop?
Yes, as long as laptop supports nested virtualization single node cluster can be deplyed on the laptop.

## Which Kubernetes distribution is supported?
vSphere Cloud Provider is available in vanilla Kubernetes and all distributions using Kubernetes v1.5 and above should support it. Please refer this [section](/vsphere-storage-for-kubernetes/documentation/prerequisites.html).

## Can multiple Kubernetes Clusters be deployed on one vCenter Server?
Yes, Using resource pool and maintaining the VMs from each Kubernetes cluster in their respective VM folder, multiple Kubernetes clusters can be deployed on one vCenter server.

## Can Kubernetes Cluster access storage from another vCenter?
Yes. vSphere supports shared storage across multiple vCenters. User can use shared storage in multiple Kubernetes Clusters.

## Does vSphere Cloud Provider support Kubernetes Zones?
Yes. See [zone support](/vsphere-storage-for-kubernetes/documentation/zones.html) for more details.

## Which Operating System are supported?
VCP is validated on Photon, Ubuntu, Core OS, RHEL. please check this [section](/vsphere-storage-for-kubernetes/documentation/prerequisites.html) for details.

## How Kubernetes volumes can be made resilient to failures on vSAN datastore?
Please check the [HA section](/vsphere-storage-for-kubernetes/documentation/high-availability.html) for details.

## Is SDRS supported on VMs hosting kubernetes cluster?
No.

## Can Reclaim Policy be set to `Retain` on dynamic PVs once volumes are provisioned with default reclaim policy `delete`?
If the volume was dynamically provisioned, then the default reclaim policy is set to “delete”. This means by default, when the PVC is deleted, the underlying PV and storage asset will also be deleted.
If volume needs to be retained, then reclaim policy needs to be changed from “delete” to “retain”. Please refer this [link](https://kubernetes.io/docs/tasks/administer-cluster/change-pv-reclaim-policy/) for more details.

## How to resize dynamic volumes once provisioned?
Support for resizing existing dynamic volume is not yet there.
Proposal is available [here](https://github.com/gnufied/community/blob/91b41028182a5291b4eccbf88f8065f66b2b7eed/contributors/design-proposals/grow-volume-size.md). vSphere Cloud Provider does not support resize of volume. Please track this [issue](https://github.com/vmware/kubernetes/issues/168).

## Is ReadWriteMany volume supported with vSphere Cloud Provider?
ReadWriteMany volume is not supported by vSphere Cloud Provider.
ReadWriteMany volumes are supported using the Container Storage Interface (CSI) as the new standard/driver that sits between vSphere and Kubernetes distributions: https://vsphere-csi-driver.sigs.k8s.io/

## Is it mandatory to have all the node VMs in the same datastore?
It is not mandatory to keep all node VMs on the same datastore. But make sure node VM has access to volumes datastores.

## How can disk.EnableUUID enabled on Node VMs?
To enable disk.EnableUUID option for the Node VM, Select VM and select `Edit Settings`. Find
Customize hardware -> VM Options -> Configuration Parameters -> Edit Configuration. Check to see if the parameter disk.EnableUUID is set, if it is there then make sure it is set to TRUE. If the parameter is not there, select Add Row and add it. This can be set on the template VM before cloning node VMs from the template.

## Is it mandatory to have all Kubernetes Cluster VMs in the same directory?
Kubernetes release 1.8 and below requires Master and worker node VMs to be present under one VM folder. Each Kubernetes Cluster deployed in vSphere, should be placed in their respective VM folder, else under root folder.

Kubernetes release 1.9 and above does not require all Kubernetes Cluster VMs to be present in the same directory.
