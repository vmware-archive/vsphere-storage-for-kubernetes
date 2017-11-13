---
title: Customize roles and privileges for different user cases
---

With Kubernetes version 1.9.x vSphere cloud provider supports kubernetes cluster spanning across multiple vCenters. Make sure that all above privileges are correctly set for all vCenters.

In general vSphere user designated for vSphere cloud provider should have
* **Read access** on *parent entities of the node VMs* like folder, host, datacenter etc.
* **VirtualMachine.Inventory.Create/Delete** on the defined *resource pool*. Used to create/delete dummy VMs.

## Minimal set of vSphere roles/privileges required for static only persistent volume provisioning.

**Note:** Datastore.FileManagement is only required for the role `manage-k8s-volumes`, if PVC is created to bind with statically provisioned PV, and reclaim policy set to delete. When PVC is deleted, associated statically provisioned PV will also be deleted.

| Roles         | Privileges    | Entities  | Propagate to Children |
| ------------- |-------------  |-----------| ----------------------|
| manage-k8s-node-vms | VirtualMachine.Config.AddExistingDisk, VirtualMachine.Config.AddNewDisk, VirtualMachine.Config.AddRemoveDevice, VirtualMachine.Config.RemoveDisk | VM Folder | Yes |
| manage-k8s-volumes | Datastore.FileManagement (Low level file operations) | Datastore | No |
| Read-only (pre-existing default role) | System.Anonymous, System.Read, System.View | vCenter, Datacenter, Datastore Cluster, Datastore Storage Folder | No |

## Minimal set of vSphere roles/privileges required for dynamic persistent volume provisioning with storage policy based volume placement.

**Same as documented at
https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/existing.html**

| Roles         | Privileges    | Entities  | Propagate to Children |
| ------------- |-------------  |-----------| ----------------------|
| manage-k8s-node-vms | Resource.AssignVMToPool, VirtualMachine.Config.AddExistingDisk, VirtualMachine.Config.AddNewDisk, VirtualMachine.Config.AddRemoveDevice,  VirtualMachine.Config.RemoveDisk, VirtualMachine.Inventory.Create, VirtualMachine.Inventory.Delete | Cluster, Hosts, VM Folder | Yes |
| manage-k8s-volumes | Datastore.AllocateSpace, Datastore.FileManagement (Low level file operations) | Datastore | No |
| k8s-system-read-and-spbm-profile-view | StorageProfile.View (Profile-driven storage view) | vCenter | No |
| Read-only (pre-existing default role) | System.Anonymous, System.Read, System.View | Datacenter, Datastore Cluster, Datastore Storage Folder | No |


## Minimal set of vSphere roles/privileges required for dynamic volume provisioning without storage policy based volume placement.

| Roles         | Privileges    | Entities  | Propagate to Children |
| ------------- |-------------  |-----------| ----------------------|
| manage-k8s-node-vms | VirtualMachine.Config.AddExistingDisk, VirtualMachine.Config.AddNewDisk,  VirtualMachine.Config.AddRemoveDevice, VirtualMachine.Config.RemoveDisk | VM Folder | Yes |
| manage-k8s-volumes | Datastore.AllocateSpace, Datastore.FileManagement (Low level file operations) | Datastore | No |
| Read-only (pre-existing default role) | System.Anonymous, System.Read, System.View | vCenter, Datacenter, Datastore Cluster, Datastore Storage Folder | No |
