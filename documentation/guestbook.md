---
title: Running Stateful application - Guestbook App
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

Guestbook is PHP application with Redis as backend. In this section, we will demonstrate how to use  Kubernetes deployed on vSphere to run Guestbook Application with persistent storage. At the end of this demo, a sample guestbook app will be available inside Kubernetes where the data is resident inside VMDKs managed by vSphere.

The data in the VMDK is independent of the lifecycle of the pods and persists even if pods are deleted.

## Guestbook with Dynamic Provisioning (Recommended)

Download the yaml files for storage class based guestbook from [here](https://github.com/vmware/kubernetes/tree/kube-examples/kube-examples/guestbook/guestbook-storageclass)

Create the storage class

```sh
kubectl create -f redis-sc.yaml
```

Create the claims that use the storage class

```sh
kubectl create -f redis-master-claim.yaml create -f redis-slave-claim.yaml
```

This should trigger creation of storage volumes on the datastore configured for the vSphere Cloud Provider.
All the dynamically provisioned volumes will be created in the kubevols directory inside the datastore.

Create the guestbook app consuming the claims

```sh
kubectl create -f guestbook-all-in-one.yaml
```

Verify Guestbook Application
The following steps verify that the Pods, storage and the application are running correctly

Wait till the pods are ready

```sh
$ kubectl get pods
NAME                         READY STATUS   RESTARTS AGE
frontend-88237173-77e6n       1/1   Running    0     43m
redis-master-1145698066-2hbbk 1/1   Running    0     43m
redis-slave-5845680045-3ffdg  1/1   Running    0     43m
```

Get the port on which the front end is accessible

```sh
$ kubectl describe service frontend | grep NodePort
Type: NodePort
NodePort: <unset> 31531/TCP
```

Check the nodes on which the frontend is running

Get the IP address of one of the nodes (This can also be obtained from vCenter)
Also possible from vCenter or the console output kube-up

```sh
$ kubectl describe node kubernetes-minion-1|grep -i Address
Addresses: 10.20.105.59
```

Combine the IP and Port and head to the URL (Ex: http://10.20.105.59:31531 ). Try out the app and leave a few messages.
To check the attach status of VMDKs on ESX head to vCenter (Settings page or Recent Tasks window)

## Guestbook with Static Provisioning

### Storage Setup

The backing VMDK files are needed when dynamic provisioning is not used. The VMDK files need to exist before creating the service in k8s that uses them.

```sh
govc datastore.mkdir -ds vsanDatastore kubevols
govc datastore.disk.create -ds vsanDatastore -size 2G kubevols/redis-slave.vmdk
govc datastore.disk.create -ds vsanDatastore -size 2G kubevols/redis-master.vmdk
```

### App Deployment

In this example we will provision PVs and Guestbook pods will claim these PVs

Download all files in the PVC PV Demo from [here](https://github.com/vmware/kubernetes/tree/kube-examples/kube-examples/guestbook/guestbook-pvc)

Create the persistent volume

```sh
kubectl create -f redis-master-pv.yaml create -f redis-slave-pv.yaml
```

Check the persistent volume records

```sh
kubectl get pv
```

Create the PVC

```sh
kubectl create -f redis-slave-claim.yaml
kubectl create -f redis-master-claim.yaml
```

Check that the volumes are bound to the claims

```sh
$ kubectl get pv

NAME                CAPACITY ACCESSMODES RECLAMIPOLICY STATUS CLAIM REASON AGE
redis-master-pv 2Gi RWO           Retain    Bound     default/redis-master-claim     34s
redis-slave-pv 2Gi    RWO         Retain    Bound     default/redis-slave-claim     34s
```

Create the guestbook

```sh
kubectl create -f guestbook-all-in-one.yaml
```

## Lifecycle of a Volume

To test the persistence, delete the guestbook and recreate it using the same volumes, the old messages on the guest book should show up.

```sh
kubectl delete -f guestbook-all-in-one.yaml
kubectl create -f guestbook-all-in-one.yaml
```

After create, the port mapping will change.

```sh
# Get the new port
$ kubectl describe service frontend| grep NodePort
Type:                 NodePort
NodePort:      <unset> 32400/TCP
```
