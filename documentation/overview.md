---
title: vSphere Cloud Provider
---

## Overview
Containers are stateless and ephemeral but applications are stateful and need persistent storage. vSphere adds this persistent storage support to Kubernetes through interface called Cloud Provider.

 ![Image](images/vSphere.png)
 
 
Kubernetes cloud provider is an interface to integrate various nodes (i.e. hosts), load balancers and networking routes. This interface allows extending Kubernetes to use various cloud and virtualization solutions as base infrastructure to run on. 
 
Cloud provider provides following interfaces to effectively integrate cloud platforms:

* **Instances** - interface for virtual machine management
* **Load Balancers** - interface to integrate with load balancer provided by cloud platform
* **Routes** - interface to add new routing rules of cloud platform
* **Zones** - integrate with zones if implemented by cloud platform


## vSphere Storage Concepts
vSphere Storage for Kubernetes offers a Cloud Provider **(vSphere Cloud Provider - VCP)** for Kubernetes which allows Pods to use enterprise grade persistent storage. vSphere has a proven Software Defined Storage(SDS) platform that integrates with block, file and hyperConverged offerings such as VMware VSAN. These storage offerings can be exposed as VMFS, NFS, VVol and VSAN datastores. SDS platform in vSphere has enterprise grade features like Storage Policy Based Management(SPBM) which enables customers to guarantee QoS requested by their business critical applications and enforce SLAs. vSphere provides various data services out of the box such high availability and data reliability for containers using Kubernetes.
 
Datastores is an abstraction which hides storage details and provide uniform interface for storing persistent data. Datastores enables simplified storage management with features like grouping them in folders. Depending on the backend storage used, the datastores can be of the type vSAN, VMFS, NFS & VVol.
 
* vSAN is a hyper-converged infrastructure storage which provides excellent performance as well as reliability. vSAN advantage is simplified storage management with features like policy driven administration. 
 
* VMFS (Virtual Machine File System) is a cluster file system that allows virtualization to scale beyond a single node for multiple VMware ESX servers. VMFS increases resource utilization by providing shared access to pool of storage.
 
* NFS (Network File System) is a distributed file protocol to access storage over network like local storage. vSphere supports NFS as backend to store virtual machines files.

* VVol(Virtual Volumes) - Virtual Volumes datastore represents a storage container in vCenter Server and vSphere Web Client. 

## Kubernetes Storage support
VCP supports every storage primitive exposed by Kubernetes -

* [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
* [Persistent Volumes (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* [Persistent Volumes Claims (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
* [Storage Class](https://kubernetes.io/docs/concepts/storage/storage-classes/#vsphere)
* [Stateful Sets](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)

Persistent volumes requested by stateful containerized applications can be provisioned on vSAN, VVol, VMFS or NFS datastores.

Kubernetes volumes are defined in Pod specification. They reference VMDK files and these VMDK files are mounted as volumes when the container is running. When the Pod is deleted the Kubernetes volume is unmounted and the data in VMDK files persists.


![Image](images/Picture1.png)
 
 
  
