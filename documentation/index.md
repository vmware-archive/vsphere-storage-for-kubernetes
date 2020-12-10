---
title: vSphere Storage for Kubernetes
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Introduction

Containers have changed the way applications are packaged and deployed. Not only containers are efficient from an infrastructure utilization point of view, but they also provide strong isolation between process on same host. They are lightweight and once packaged can run anywhere. This user guide outlines integration of vSphere storage with Kubernetes.

## Persistent Storage in Container World

Although it is relatively easy to run stateless Microservices using container technology, stateful applications require slightly different treatment. There are multiple factors which need to be considered when handling persistent data using containers, such as:

* Containers are ephemeral by nature, so the data that needs to be persisted has to survive through the restart/re-scheduling of a container.
* When containers are re-scheduled, they can die on one host and might get scheduled on a different host. In such a case the storage should also be shifted and made available on the new host for the container to start gracefully.
* The application should not have to worry about the volume and data. The underlying infrastructure should handle the complexity of unmounting and mounting.
* Certain applications have a strong sense of identity (e.g.; Kafka, Elastic) and the disk used by a container with certain identity is tied to it. It is important that if a container with a certain ID gets re-scheduled for some reason then the disk associated with that ID is re-attached to the new container instance.

## Kubernetes Storage Primitives

Kubernetes provides abstractions to ensure that the storage details are separated from allocation and usage of storage.

Please refer [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/) for details.
