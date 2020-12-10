---
title: Permissions
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Introduction

With Kubernetes version 1.9.x vSphere Cloud Provider supports Kubernetes cluster spanning across multiple vCenters. Make sure that all above privileges are correctly set for all vCenters if you are using multiple vCenters.

Please refer [vSphere Documentation Center](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.security.doc/GUID-18071E9A-EED1-4968-8D51-E0B4F526FDA3.html) to find out how to create a `Custom Role`, `User`, and `Role Assignment`.

## Overview

In general, the vSphere user designated to the vSphere Cloud Provider should have the following permissions:

* `Read` permission on the *parent entities* of the node VMs such as `folder`, `host`, `datacenter`, `datastore folder`, `datastore cluster`, etc.
* `VirtualMachine.Inventory.Create/Delete` permission on the `vsphere.conf` defined `resource pool` - this is used to create/delete dummy VMs.

## Static Provisioning

| Roles         | Privileges    | Entities  | Propagate to Children |
| ------------- |-------------  |-----------| ----------------------|
| manage-k8s-node-vms | VirtualMachine.Config.AddExistingDisk, VirtualMachine.Config.AddNewDisk, VirtualMachine.Config.AddRemoveDevice, VirtualMachine.Config.RemoveDisk | VM Folder | Yes |
| manage-k8s-volumes | Datastore.FileManagement (Low level file operations) | Datastore | No |
| Read-only (pre-existing default role) | System.Anonymous, System.Read, System.View | vCenter, Datacenter, Datastore Cluster, Datastore Storage Folder, Cluster, Hosts | No |

## Dynamic Provisioning

| Roles         | Privileges    | Entities  | Propagate to Children |
| ------------- |-------------  |-----------| ----------------------|
| manage-k8s-node-vms | Resource.AssignVMToPool, VirtualMachine.Config.AddExistingDisk, VirtualMachine.Config.AddNewDisk, VirtualMachine.Config.AddRemoveDevice,  VirtualMachine.Config.RemoveDisk, VirtualMachine.Inventory.Create, VirtualMachine.Inventory.Delete, VirtualMachine.Config.Settings | Cluster, Hosts, VM Folder | Yes |
| manage-k8s-volumes | Datastore.AllocateSpace, Datastore.FileManagement (Low level file operations) | Datastore | No |
| k8s-system-read-and-spbm-profile-view | StorageProfile.View (Profile-driven storage view) | vCenter | No |
| Read-only (pre-existing default role) | System.Anonymous, System.Read, System.View | Datacenter, Datastore Cluster, Datastore Storage Folder | No |
