---
title: Customize roles and privileges for different user cases
---

## Minimal set of vSphere roles/privileges required for static only persistent volume provisioning.

**Note Datastore.FileManagement is only required for the role `manage-k8s-volumes`, if PVC is created to bind with statically provisioned PV, and reclaim policy set to delete. When PVC is deleted, associated statically provisioned PV will also be deleted.**

<table>
<thead>
<tr>
  <th>Roles</th>
  <th>Privileges</th>
  <th>Entities</th>
  <th>Propagate to Children</th>
</tr>
</thead>
<tbody><tr>
  <td>manage-k8s-node-vms</td>
  <td>System.Anonymous<br> System.Read<br> System.View<br> VirtualMachine.Config.AddExistingDisk<br> VirtualMachine.Config.AddNewDisk<br> VirtualMachine.Config.AddRemoveDevice<br> VirtualMachine.Config.RemoveDisk<br>
  <td>VM Folder</td>
  <td>Yes</td>
</tr>
<tr>
  <td>manage-k8s-volumes</td>
  <td>Datastore.FileManagement (Low level file operations
)<br> System.Anonymous<br> System.Read<br> System.View<br>
</td>
  <td>Datastore</td>
  <td>No</td>
</tr>
<tr>
  <td>ReadOnly</td>
  <td>System.Anonymous<br>System.Read<br>System.View</td>
  <td>vCenter,<br> Datacenter,<br> Datastore Cluster,<br> Datastore Storage Folder</td>
  <td>No</td>
</tr>
</tbody>
</table>

## Minimal set of vSphere roles/privileges required for dynamic persistent volume provisioning with storage policy based volume placement.

**Same as documented at
https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/existing.html**

<table>
<thead>
<tr>
  <th>Roles</th>
  <th>Privileges</th>
  <th>Entities</th>
  <th>Propagate to Children</th>
</tr>
</thead>
<tbody><tr>
  <td>manage-k8s-node-vms</td>
  <td>Resource.AssignVMToPool<br> System.Anonymous<br> System.Read<br> System.View<br> VirtualMachine.Config.AddExistingDisk<br> VirtualMachine.Config.AddNewDisk<br> VirtualMachine.Config.AddRemoveDevice<br> VirtualMachine.Config.RemoveDisk<br> VirtualMachine.Inventory.Create<br> VirtualMachine.Inventory.Delete</td>
  <td>Cluster,<br> Hosts,<br> VM Folder</td>
  <td>Yes</td>
</tr>
<tr>
  <td>manage-k8s-volumes</td>
  <td>Datastore.AllocateSpace<br> Datastore.FileManagement (Low level file operations)<br> System.Anonymous<br> System.Read<br> System.View</td>
  <td>Datastore</td>
  <td>No</td>
</tr>
<tr>
  <td>k8s-system-read-and-spbm-profile-view</td>
  <td>StorageProfile.View<br> System.Anonymous<br> System.Read<br> System.View</td>
  <td>vCenter</td>
  <td>No</td>
</tr>
<tr>
  <td>ReadOnly</td>
  <td>System.Anonymous<br>System.Read<br>System.View</td>
  <td>Datacenter,<br> Datastore Cluster,<br> Datastore Storage Folder</td>
  <td>No</td>
</tr>
</tbody>
</table>

## Minimal set of vSphere roles/privileges required for dynamic volume provisioning without storage policy based volume placement.

<table>
<thead>
<tr>
  <th>Roles</th>
  <th>Privileges</th>
  <th>Entities</th>
  <th>Propagate to Children</th>
</tr>
</thead>
<tbody><tr>
  <td>manage-k8s-node-vms</td>
  <td>System.Anonymous<br> System.Read<br> System.View<br> VirtualMachine.Config.AddExistingDisk<br> VirtualMachine.Config.AddNewDisk<br> VirtualMachine.Config.AddRemoveDevice<br> VirtualMachine.Config.RemoveDisk<br></td>
  <td>VM Folder</td>
  <td>Yes</td>
</tr>
<tr>
  <td>manage-k8s-volumes</td>
  <td>Datastore.AllocateSpace<br> Datastore.FileManagement (Low level file operations)<br> System.Anonymous<br> System.Read<br> System.View</td>
  <td>Datastore</td>
  <td>No</td>
</tr>
<tr>
  <td>ReadOnly</td>
  <td>System.Anonymous<br>System.Read<br>System.View</td>
  <td>vCenter,<br>Datacenter,<br> Datastore Cluster,<br> Datastore Storage Folder</td>
  <td>No</td>
</tr>
</tbody>
</table>
