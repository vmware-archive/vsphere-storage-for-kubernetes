---
title: Maximum scale supported by VCP
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Test Scenario

1. Create a kubernetes cluster with 1000 worker nodes and enabling vSphere Cloud Provider (VCP) on all of them.
2. Create a statefulset(nginx) with persistent volume dynamically provisioned where provisioner is vsphere-volume.

   Kubernetes StatefulSets allows to us verify 4 key workflows in VCP.

    * Create vSphere persistent volume
    * Attach vSphere persistent volume to a pod
    * Detach vSphere persistent volume from a pod
    * Delete vSphere persistent volume
3. Once the StatefulSet is created with replicas: 1000,  we see 1000 pods created with vSphere persistent volume attached for each pod. Each pod has a vSphere persistent volume attached to it. Each pod is provisioned on an individual kubernetes worker node.
4. The datastore used to create volumes is a shared VMFS datastore and disk type is thin.

The following infrastructure configuration is used to validate the scale limits for VCP.

## Node CPU and Memory Capacity Allocation

To handle workload for 1000 node Kubernetes cluster master node is configured to use 8 cpus and 8 GB memory.

```
Capacity:
 cpu:     8
 memory:  8174428Ki
```
**Note:** Here 8 CPU cores means 8000 milli-cores.

Worker nodes are allocated 2 GB and 2 CPUs.

**Note:** CPU/Memory allocation for nodes can be adjusted based on the workload requirement. Above is the minimum required allocation.


## Configure CPU and Memory Requests for Kubernetes System Pods

It is recommended to set guaranteed CPU and Memory reservation for Kubernetes System Pods so that these pods does not go out of resources.

CPU/Memory Requests is set in the Pod manifest files located at `/etc/kubernetes/manifests` as mentioned below.

```
containers:
    image: gcr.io/google_containers/etcd-amd64:3.0.17
    name: etcd
    resources:
      requests:
        cpu: 1200m
        memory: 1650Mi
```

CPU/Memory Requests set on system pods.

| Pod | CPU Requests | Memory Requests |
| ------ | ------ | ------ |
| apiserver | 1200m (15%) | 1650Mi |
| controller-manager | 1600m (20%) | 2200Mi |
| scheduler | 800m (10%) | 1100Mi |
| etcd | 1200m (15%) | 1650Mi |

**Note:** Here resource `requests` is set on system pods and not the `limit`, so that if Pod requires more CPU/Memory then available CPU/Memory can be utilized during the peak time.

For rest of the Pods (Proxy, DNS and flannel etc.) on the master node 40% cpu is available.

Refer [Managing Compute Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) to know more about CPU and Memory Requests in Kubernetes.

Below is the summary of master node resources captured during scale testing 1000 node Kubernetes cluster.

```
Namespace                  Name                               CPU Requests  CPU Limits  Memory Requests  Memory Limits
---------                  ----                               ------------  ----------  ---------------  -------------
kube-system                etcd-node-01                       1200m (15%)   0 (0%)      1650Mi (20%)     0 (0%)
kube-system                kube-apiserver-node-01             1200m (15%)   0 (0%)      1650Mi (20%)     0 (0%)
kube-system                kube-controller-manager-node-01    1600m (20%)   0 (0%)      2200Mi (27%)     0 (0%)
kube-system                kube-dns-545bc4bfd4-ghdjf          260m (3%)     0 (0%)      110Mi (1%)       170Mi (2%)
kube-system                kube-flannel-ds-7b5tp              0 (0%)        0 (0%)      0 (0%)           0 (0%)
kube-system                kube-proxy-mk8jp                   0 (0%)        0 (0%)      0 (0%)           0 (0%)
kube-system                kube-scheduler-node-01             800m (10%)    0 (0%)      1100Mi (13%)     0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ------------  ----------  ---------------  -------------
  5060m (63%)   0 (0%)      6710Mi (85%)     170Mi (2%)
```

## Flannel Network Configuration

Most of the deployment uses 17.0.0.0/24 as the default flannel network setting. This allows registering only 254 nodes. To deploy more than 254 nodes flannel network should be set to 17.0.0.0/8.
