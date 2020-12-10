---
title: Prerequisites
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

Following is the list of prerequisites for running Kubernetes with vSphere Cloud Provider. Refer to [the versions list](/vsphere-storage-for-kubernetes/documentation/versions.html) for features supported in each version.

## Kubernetes

* Kubernetes version - v1.6.5+

## vSphere

* vSphere version - 6.0.x (Virtual Hardware 11) and above. (**Note:** Standalone ESX is not supported.)
* vSAN, VMFS and NFS supported.
  * vSAN support is limited to one cluster in one vCenter.

### Permissions

* vCenter user with required [set of privileges](/vsphere-storage-for-kubernetes/documentation/vcp-roles.html).

## VMs

* All node VMs must be placed in a vSphere VM folder.
  * Create a VM folder following the instructions mentioned in this [link](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.vcenterhost.doc/GUID-031BDB12-D3B2-4E2D-80E6-604F304B4D0C.html) and move Kubernetes Node VMs to this folder.

* The `disk.EnableUUID` parameter must be set to `TRUE` for each Node VM.
  * This step is necessary so that the VMDK always presents a consistent UUID to the VM, thus allowing the disk to be mounted properly. For each of the virtual machine nodes that will be participating in the Kubernetes cluster, follow the steps below using [govc.](#tooling)
  * Set disk.EnableUUID to true for all VMs:

    ```sh
    govc vm.change -e="disk.enableUUID=1" -vm='vm name here'
    ```

_Note: If Kubernetes node VMs are created from a template VM then `disk.EnableUUID=1` can be set on the template VM - VMs cloned from this template, will automatically inherit this property._

### Operating System

* Kubernetes host operating systems:
  * Photon OS - v1.0 GA
  * Ubuntu - 16.04, 18.04
  * CoreOS - v4.11.2
  * Centos  - v7.3
  * RHEL OS -  v7.4
  * SLES - v12 SP3

* VMware Tools needs to be installed on the guest operating system on each Node VM. Please refer this [link](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.html.hostclient.doc/GUID-ED3ECA21-5763-4919-8947-A819A17980FB.html) for instructions on installing VMware tools.

## Tooling

* [govc](https://github.com/vmware/govmomi/releases) v0.18 and above

The [govc CLI](https://github.com/vmware/govmomi/tree/master/govc#govc) can be used for additional configuration and troubleshooting.  Example govc
configuration:

        export GOVC_URL='vCenter IP OR FQDN'
        export GOVC_USERNAME='vCenter User'
        export GOVC_PASSWORD='vCenter Password'
        export GOVC_INSECURE=1
