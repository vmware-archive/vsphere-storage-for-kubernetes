---
title: Prerequisites
---

Following is the list of prerequisites for running Kubernetes with vSphere Cloud Provider:

* Kubernetes version - v1.6.5 and above

Following table summarizes key features introduced in vSphere Cloud Provider in each Kubernetes release

| Kubernetes Release | vSphere Cloud Provider feature |
| ------ | ------ |
| v1.6.3 | Dynamic volume provisioning using vSAN storage capabilities |
| v1.6.5 | Integration with vSphere HA |
| v1.7.0 | Integration with vSphere Storage Policy Based Management(SPBM) for dynamic volume provisioning |
| v1.7.0 | Enhanced vSphere Cloud Provider debuggabilty via integration with metrices exposed for Kubernetes storage APIs |
| v1.8.0 | vSphere Cloud Provider refactoring for better debuggability, logging and code maintenance |
| v1.8.2 | Performance improvement for large scale deployment |
| v1.9.0 | Multi vCenter Support |

* vSphere version - 6.0.x and above

* Container Host operating systems -
    - Photon - v1.0 GA
    - Ubuntu - v16.04
    - CoreOs - v4.11.2
    - Centos  - v7.3
    - RHEL OS -  v7.4

* vSphere setup to deploy the Kubernetes cluster.
   - For Kubernetes version 1.9.x and above: vSphere Cloud Provider supports Kubernetes cluster spanning across multiple vCenters.
   - For Kubernetes version 1.8.x and below: vSphere Cloud Provider supports Kubernetes cluster deployed only in one vCenter.
* vSphere supported storage.
    - It can be HCI offering such as VMware vSAN or block and file storage offerings like VMFS and NFS.
    - With kuberneters version 1.9.x if user wants to use multiple vCenters then vSAN storage can not be used. vSAN is limited to one cluster in one vCenter deployment.
* vCenter user with required set of privileges.
* VMware Tools needs to be installed on the guest operating system on each Node VM. Please refer this [link](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.html.hostclient.doc/GUID-ED3ECA21-5763-4919-8947-A819A17980FB.html) for instruction on installing VMware tools.
* Node VM name requirements needed only until Kuberneters version 1.8.x.
    - VM names can not begin with numbers.
    - VM names can not have capital letters, any special characters except `.` and `-`.
    - VM names can not be shorter than 3 chars and longer than 63
* The disk.EnableUUID parameter must be set to "TRUE" for each Node VM. Please refer this [section.](/vsphere-storage-for-kubernetes/documentation/existing.html#enable-disk-uuid-on-node-virtual-machines)

