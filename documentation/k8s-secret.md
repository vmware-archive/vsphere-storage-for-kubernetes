---
  title: Securing vSphere credentials
  summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Introduction

vSphere credentials can be protected from being exposed as clear text by using Kubernetes `secrets` instead of plain-text usernames and passwords in your `vsphere.conf`.

## Config file changes

Create a `vsphere.conf` file with `secret-name` and `secret-namespace` defined instead of `username` and `password` as is detailed [here](/vsphere-storage-for-kubernetes/documentation/existing.html#supported-parameters).

```sh
[Global]
secret-name = "vcconf"
secret-namespace = "kube-system"
port = "443"
insecure-flag = "1"

[VirtualCenter "10.0.1.200"]
datacenters = "DC-1"

[Workspace]
server = "10.0.1.200"
datacenter = "DC-1"
default-datastore = "vsanDatastore"
resourcepool-path = "ClusterNameHere/Resources"
folder = "kubernetes"

[Disk]
scsicontrollertype = pvscsi
```

## Encode the username and password

Launch the Kubernetes cluster with vSphere Cloud Provider configured as above.

Create `base64` encoded strings for the username and password:

```sh
$ echo -n 'Administrator@vsphere.local' | base64
QWRtaW5pc3RyYXRvckB2c3BoZXJlLmxvY2Fs

$ echo -n 'password' | base64
cGFzc3dvcmQ=
```

## Create a K8s secret

Next, create a Kubernetes `secret` with the `base64` encoded vCenter credentials.

Create a `credentials.yaml` as shown below - replacing your vCenter IP address and `base64` strings from above, as appropriate. In the below example, the `secret` name is `vcconf` and matches what we put in the `secret-name` section of `vsphere.conf` file above.

```sh
apiVersion: v1
kind: Secret
metadata:
 name: vcconf
type: Opaque
data:
   10.0.1.200.username: QWRtaW5pc3RyYXRvckB2c3BoZXJlLmxvY2Fs
   10.0.1.200.password: cGFzc3dvcmQ=
```

Import the secret definition into Kubernetes (note, we are importing into the `kube-system` namespace, which must match the `secret-namespace` section in the `vsphere.conf` file from above):

```sh
kubectl create -f credentials.yaml --namespace=kube-system
```

The vSphere Cloud Provider will authenticate with the vCenter using the Kubernetes managed secret.

## Using SAML auth

Note: _vSphere credentials can also be encrypted using **SAML token authentication**, Please refer documentation for [SAML token authentication](/vsphere-storage-for-kubernetes/documentation/saml-token-authentication.md) using vCenter SSO API._
