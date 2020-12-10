---
title: Upgrading clusters deployed with kubeadm
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Before you begin

Before proceeding:

* You need to have a functional Kubernetes cluster deployed using kubeadm running version 1.7.0 or higher with vSphere Cloud Provider configured in order to use the process described here.
* Make sure swap is disabled. If the host is running a linux based distribution, run:

        swapoff -a

* kubeadm upgrade allows you to upgrade etcd. kubeadm upgrade will also upgrade etcd to 3.1.10 as part of upgrading from v1.8 to v1.9 by default. This is due to the fact that etcd 3.1.10 is the officially validated etcd version for Kubernetes v1.9. The upgrade is handled automatically by kubeadm.
* Note that kubeadm upgrade will not touch any of your workloads, only Kubernetes-internal components. As a best-practice you should back up what’s important to you. For example, any app-level state, such as a database an app might depend on (like MySQL or MongoDB) must be backed up beforehand.

## Upgrade Kubernetes cluster

### Upgrade the control plane

* Install the most recent version of kubeadm as mentioned in this [link](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm-upgrade-1-9/)

* On the master node, run the following

        kubeadm upgrade plan

  This checks that your cluster is upgradeable and fetches the versions available to upgrade to in an user-friendly way.

* Now, pick a version to upgrade to and run.

        kubeadm upgrade apply v1.9.2

  You can refer the [link](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm-upgrade-1-9/) for detailed instructions.

* Now that your control plane is upgraded, you need to upgrade your kubelets as well.

### Upgrading master and node packages

* For each host (referred to as $HOST below) in your cluster, upgrade kubelet by executing the following commands:

        kubectl drain $HOST --ignore-daemonsets

* Upgrade the Kubernetes package versions on the $HOST node by using a Linux distribution-specific package manager. You can refer this [link](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm-upgrade-1-9/) for distro specific instructions.

### Update kubelet configuration on workers

* Remove the **--cloud-config** paramater from kubelet configuration file.
* Remove the vsphere.conf (vSphere configuration file) from /etc/kubernetes
* Restart the kubelet

        systemctl status kubelet

## Update vSphere Cloud Provider

The upgrade instructions from Kubernetes 1.7/1.8 to 1.9 using kubeadm as described above works fine for a Kubernetes cluster deployed on a single vCenter.

In 1.9, vSphere Cloud Provider has support for a single Kubernetes cluster deployed across multiple vCenters. So, if one needs to upgrade from Kubernetes cluster 1.7/1.8 deployed on single vCenter to Kubernetes cluster 1.9 deployed across multiple vCenters, few additional steps need to be performed apart from those mentioned above.

**Note:** Do not trigger any workflows that involve volume related operations unless all the instructions mentioned below are executed.

* Identify the nodes that needs to be moved to other vCenter. Migrate the node(s) from current vCenter to new vCenter. Make sure all the nodes in the Kubernetes cluster have access to atleast one shared datastore.
* After successful migration, check the status of the kubelet on the node(s).

        systemctl status kubelet

* Update the vsphere.conf on master node to let vSphere Cloud Provider know about the nodes that are deployed in multiple vCenters. Follow the instructions mentioned in the [link](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/existing.html#step-5-create-the-vsphere-cloud-config-file-vsphereconf) to update the vsphere.conf file. This file is located at /etc/kubernetes directory.
* Restart the kubelet on master.

        systemctl restart kubelet

* Verify all the nodes are ready and running.

        kubectl get nodes
