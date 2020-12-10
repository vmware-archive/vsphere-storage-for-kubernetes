---
  title: vSphere Cloud Provider Configuration
  summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

## Prerequisites

Please refer to the [prerequisites](/vsphere-storage-for-kubernetes/documentation/prerequisites.html) page for the required environment setup piror to following the steps below. After the prerequisites are met, follow below-mentioned steps to enable vSphere Cloud Provider for Kubernetes.

## Permissions

The first step is to create and assign roles to the vSphere Cloud Provider user and vSphere entities. Please refer Roles and Privileges documented [here](/vsphere-storage-for-kubernetes/documentation/vcp-roles.html).

_Note: If the vCenter Administrator account is going to be used by vSphere Cloud Provider, then this step can be skipped._

## Kubernetes Configuration

### Config file location

The vSphere Cloud Provider config file (`vsphere.conf`) needs to be created in a directory which should be accessible from the `kubelet`, `controller-manager`, and `API server` services. In general - we advise placing it in `/etc/kubernetes`.

### Config file

For Kubernetes versions 1.9.x and above your `vsphere.conf` file should be placed only on the Kubernetes master nodes.

#### Template config file

Below is an example `vsphere.conf` without any inline comments to base your configuration on.

```sh
[Global]
user = "administrator@vsphere.local"
password = "password"
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

#### Supported Parameters

Below is the summary of supported parameters in the `vsphere.conf` file

`[Global]`

* `user` is the vCenter username for vSphere Cloud Provider.
* `password` is the password for vCenter user specified with `user`.

*Note: If you don't want to store your password and username in plain text, see [this section](/vsphere-storage-for-kubernetes/documentation/existing.html#securing-vsphere-username-and-password) and use the below two parameters instead*

* `secret-name` K8s secret name - used if you wish to store your vSphere credentials in a K8s `secret`
* `secret-namespace` K8s secret namepsace - used if you wish to store your vSphere credentials in a K8s `secret`
* `server` is the vCenter Server IP or FQDN
* `port` is the vCenter Server Port. The default is 443 if not specified.
* `insecure-flag` is set to 1 if vCenter used a self-signed certificate.
* `datacenter` is the name of the vCenter Datacenter on which Kubernetes node VMs are deployed.
* `datastore` is the default datastore to use for provisioning volumes using storage classes/dynamic provisioning.

`[VirtualCenter ""]` - Multiple `VirtualCenter` sections are supported in K8s 1.9 and above

* Any of the params from `[Global]` section above are supported here.

`[Workspace]` - Defines the temporary location of shadow VMs provisioned to create volumes

* `server` is the vCenter Server IP or FQDN
* `datacenter` - the Datacenter to provision temporary VMs to for volume provisioning
* `default-datastore` is the default datastore used for dynamic volume provisioning.
  * If the datastore is located in storage folder or datastore is the member of datastore cluster, make sure to specify full datastore path.
    * For a datastore located in a datastore cluster, specify the datastore as shown below:

        ```sh
        datastore = "DatastoreCluster/datastore1"
        ```

    * For a datastore located in a storage folder, specify datastore as shown below

        ```sh
        datastore = "DatastoreStorageFolder/datastore1"
        ```
* `resourcepool-path` - the resource pool to provision temporary VMs to for volume provisioning (by default, all environments have a `Resources` resource pool, even without DRS)
* `folder` - the VM folder your Kubernetes VMs are in, in vCenter

`[Disk]`

* `scsicontrollertype` almost always set to `pvscsi`

#### Examples

##### Single vCenter

Below is an annotated example configuration for a Kubernetes cluster for which all Kubernetes nodes are located on a single vCenter:

```sh
[Global] # Anything in the Global section will apply to all vCenters in the config file unless overridden in the VirtualCenter section

[VirtualCenter "10.0.1.200"] # The IP address of your vCenter server
user = "Administrator@vsphere.local" # The vCenter user to authenticate with
password = "password" # The password for the above vCenter user
port = "443" # The vCenter API port - usually 443
insecure-flag = "1" # Do not verify the SSL cert on the vCenter
datacenters = "DC-1" # # Your Datacenter name within vCenter where the K8s nodes reside

[Workspace] # This defines where the container storage will be provisioned (usually the same as your above vCenter) when using SPBM
server = "10.0.1.200" # The IP address of your vCenter server for storage provisioning operations
datacenter = "DC-1" # The Datacenter to provision temporary VMs to for volume provisioning
default-datastore = "vsanDatastore" # The default datastore to provision temporary VMs to for volume provisioning
resourcepool-path = "Cluster01/Resources" # The resource pool to provision temporary VMs to for volume provisioning
folder = "kubernetes" # The VM folder your Kubernetes VMs are in, in vCenter

[Disk] # This section is mandatory
scsicontrollertype = pvscsi # Defines the SCSI controller in use on the VMs - leave this as is
```

##### Multiple vCenters

Below is an annotated example configuration for a Kubernetes cluster for which all Kubernetes nodes are located on multiple vCenters/Datacenters:

```sh
[Global] # Anything in the Global section will apply to all vCenters in the config file unless overridden in the VirtualCenter section
user = "Administrator@vsphere.local" # The vCenter user to authenticate with
password = "password" # The password for the above vCenter user
port = "443" # The vCenter API port - usually 443
insecure-flag = "1" # Do not verify the SSL cert on the vCenter
datacenters = "DC-1, DC-2" # Your Datacenter names within vCenter where the K8s nodes reside

[VirtualCenter "10.0.1.200"] # The IP address of your first vCenter server
user = "Administrator123@vsphere.local" # Override the global vCenter user to authenticate with
password = "password123" # Override the global password for the above vCenter user
datacenters = "DC-1" # Your Datacenter name within vCenter

[VirtualCenter "10.0.100.200"] # The IP address of your second vCenter server
user = "Administrator456@vsphere.local" # Override the global vCenter user to authenticate with
password = "password456" # Override the global password for the above vCenter user
datacenters = "DC-2" # Your Datacenter name within vCenter

[VirtualCenter "172.16.0.100"] # A vCenter server that uses the global username and password from above
[VirtualCenter "192.168.0.100"] # Another vCenter server that uses the global username and password from above

[Workspace] # This defines where the container storage will be provisioned (currently only available in a single vCenter) when using SPBM
server = "10.0.1.200" # The IP address of your vCenter server for storage provisioning operations
datacenter = "DC-1" # The Datacenter to provision temporary VMs to for volume provisioning
default-datastore = "vsanDatastore" # The default datastore to provision temporary VMs to for dynamic volume provisioning
resourcepool-path = "Cluster01/Resources" # The resource pool to provision temporary VMs to for dynamic volume provisioning
folder = "kubernetes" # The VM folder your Kubernetes VMs are in, in vCenter

[Disk] # This section is mandatory
scsicontrollertype = pvscsi # Defines the SCSI controller in use on the VMs - leave this as is
```

## Enable the vSphere Cloud Provider

### On the Kubernetes masters

Add following flags to the `kubelet` service configuration (usually in the `systemd` config file), as well as the `controller-manager` and `api-server` container manifest files on the master node (usually in `/etc/kubernetes/manifests`).

```sh
--cloud-provider=vsphere
--cloud-config=/etc/kubernetes/vsphere.conf
```

#### Systemd services

The `kubelet` service usually runs as a `systemd` service and it's file can be found at `/etc/systemd/system/kubelet.service`. You will need to add the following to the `kubelet` daemon arguments:

```sh
--cloud-provider=vsphere --cloud-config=/etc/kubernetes/vsphere.conf
```

An example configuration of the complete `systemd` service file would be like this:

```yaml
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/kubernetes/kubernetes
Requires=docker.service network-online.target
After=docker.service network-online.target

[Service]
ExecStartPre=/bin/mkdir -p /var/lib/kubelet
ExecStartPre=/bin/mount --bind /var/lib/kubelet /var/lib/kubelet
ExecStartPre=/bin/mount --make-shared /var/lib/kubelet
ExecStart=/usr/bin/docker run \
        --net=host \
        --pid=host \
        --privileged \
        -v /dev:/dev \
        -v /sys:/sys:ro \
        -v /var/run:/var/run:rw \
        -v /var/lib/docker/:/var/lib/docker:rw \
        -v /var/lib/kubelet/:/var/lib/kubelet:shared \
        -v /var/log:/var/log:shared \
        -v /srv/kubernetes:/srv/kubernetes:ro \
        -v /etc/kubernetes:/etc/kubernetes:ro \
        cns-docker-local.artifactory.eng.vmware.com/hyperkube-amd64:20190305200017 \
        /hyperkube kubelet --address=0.0.0.0 --allow-privileged=true --enable-server --enable-debugging-handlers --kubeconfig=/srv/kubernetes/kubeconfig.json --cluster-dns=10.0.0.10 --cluster-domain=cluster.local --v=4 --register-schedulable=false --pod-manifest-path=/etc/kubernetes/manifests --cloud-provider=vsphere --cloud-config=/etc/kubernetes/vsphere.conf
Restart=always
StartLimitInterval=0
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

#### Container manifests

We need to add the `cloud` config flags into the container manifests for the API Server and the Controller Manager.

Depending on your kubernetes version, the files you need to edit are either `json` or `yaml`, but the path is the same.

They will usually located at `/etc/kubernetes/manifests/kube-apiserver.json`/`.yaml` and `/etc/kubernetes/manifests/kube-controller-manager.json`/`.yaml`. You will need to add the following to the `command` section:

`JSON:`

```json
"--cloud-provider=vsphere",
"--cloud-config=/etc/kubernetes/vsphere.conf"
```

`YAML:`

```json
  - --cloud-provider=vsphere
  - --cloud-config=/etc/kubernetes/vsphere.conf
```

You can see the sections added to a `JSON` configuration below:

```json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "labels": {
            "component": "kube-apiserver",
            "tier": "control-plane"
        },
        "name": "kube-apiserver",
        "namespace": "kube-system"
    },
    "spec": {
        "containers": [
            {
                "command": [
                    "/hyperkube",
                    "apiserver",
                    "--address=127.0.0.1",
                    "--etcd-servers=http://127.0.0.1:2379",
                    "--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,ResourceQuota",
                    "--service-cluster-ip-range=10.0.0.0/16",
                    "--client-ca-file=/srv/kubernetes/ca.pem",
                    "--tls-cert-file=/srv/kubernetes/apiserver.pem",
                    "--tls-private-key-file=/srv/kubernetes/apiserver-key.pem",
                    "--secure-port=443",
                    "--storage-backend=etcd3",
                    "--allow-privileged",
                    "--v=6",
                    "--authorization-mode=AlwaysAllow",
                    "--cloud-provider=vsphere",
                    "--cloud-config=/etc/kubernetes/vsphere.conf"
                ],
```

### On the Kubernetes workers

Add following flags to the `kubelet` service configuration (usually in the `systemd` config file).

```sh
--cloud-provider=vsphere
```

On worker nodes, unless using `zones`, we do not require the `vsphere.conf` config file hence the `--cloud-config=` flag is not needed.

#### Systemd services

The `kubelet` service usually runs as a `systemd` service and it's file can be found at `/etc/systemd/system/kubelet.service`. You will need to add the following to the `kubelet` daemon arguments:

```sh
--cloud-provider=vsphere
```

An example configuration would be like this:

```yaml
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/kubernetes/kubernetes
Requires=docker.service network-online.target
After=docker.service network-online.target

[Service]
ExecStartPre=/bin/mkdir -p /var/lib/kubelet
ExecStartPre=/bin/mount --bind /var/lib/kubelet /var/lib/kubelet
ExecStartPre=/bin/mount --make-shared /var/lib/kubelet
ExecStart=/usr/bin/docker run \
        --net=host \
        --pid=host \
        --privileged \
        -v /dev:/dev \
        -v /sys:/sys:ro \
        -v /var/run:/var/run:rw \
        -v /var/lib/docker/:/var/lib/docker:rw \
        -v /var/lib/kubelet/:/var/lib/kubelet:shared \
        -v /var/log:/var/log:shared \
        -v /srv/kubernetes:/srv/kubernetes:ro \
        -v /etc/kubernetes:/etc/kubernetes:ro \
        cns-docker-local.artifactory.eng.vmware.com/hyperkube-amd64:20190305200017 \
        /hyperkube kubelet --address=0.0.0.0 --allow-privileged=true --enable-server --enable-debugging-handlers --kubeconfig=/srv/kubernetes/kubeconfig.json --cluster-dns=10.0.0.10 --cluster-domain=cluster.local --v=4 --hairpin-mode=promiscuous-bridge --cloud-provider=vsphere
Restart=always
StartLimitInterval=0
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

### Update all node ProviderID fields

We need to make sure the K8s `ProviderID` is set on all nodes in the cluster to ensure volumes are mounted into the right node.

This can be checked by running the below:

```sh
kubectl get nodes -o json | jq '.items[]|[.metadata.name, .spec.providerID, .status.nodeInfo.systemUUID]'
```

If the output is empty, you will need to run the following script:

On a machine with [`govc`](https://github.com/vmware/govmomi/tree/master/govc)  `jq`, and `kubectl` installed (usually the master node), run the following script to set the `ProviderID` from vCenter to each node:

```sh
#!/bin/bash

export GOVC_USERNAME='<user>'
export GOVC_INSECURE=1
export GOVC_PASSWORD='<password>'
export GOVC_URL='<server>'
DATACENTER='Coop'
FOLDER='<path>'
# In my case I'm using a prefix for the VM's, so grep'ing is necessary.
# You can remove it if the folder you are using only contains the machines you need.
VM_PREFIX='<prefix>'
IFS=$'\n'
for vm in $(./govc ls "/$DATACENTER/vm/$FOLDER" | grep $VM_PREFIX); do
  MACHINE_INFO=$(./govc vm.info -json -dc=$DATACENTER -vm.ipath="/$vm" -e=true)
  # My VMs are created on vmware with upper case names, so I need to edit the names with awk
  VM_NAME=$(jq -r ' .VirtualMachines[] | .Name' <<< $MACHINE_INFO | awk '{print tolower($0)}')
  # UUIDs come in lowercase, upper case then
  VM_UUID=$( jq -r ' .VirtualMachines[] | .Config.Uuid' <<< $MACHINE_INFO | awk '{print toupper($0)}')
  echo "Patching $VM_NAME with UUID:$VM_UUID"
  # This is done using dry-run to avoid possible mistakes, remove when you are confident you got everything right.
  kubectl patch node $VM_NAME -p "{\"spec\":{\"providerID\":\"vsphere://$VM_UUID\"}}"
done
```

At this point, if you query the `ProviderID` again, they should be populated, test with the following command:

```sh
kubectl get nodes -o json | jq '.items[]|[.metadata.name, .spec.providerID, .status.nodeInfo.systemUUID]'
```

This time you should see the `ProviderID` output for each node.

### Restart the services

Reload the `systemd` unit files using:

```sh
systemctl daemon-reload
```

#### Kubelet

Restart the `kubelet` service using:

```sh
systemctl restart kubelet.service
```

#### Controller-manager

##### Running as a container

If the `controller-manager` is running in a container, restart the `controller-manager` container:

```sh
$ docker ps | grep controller | grep -v POD
8350dcd5ccd1        "/hyperkube controlle"   About an hour ago   Up About an hour   k8s_kube-controller-manager_kube-controlle-manager-kubernetes-master_kube-system_...

$ docker stop 8350dcd5ccd1
8350dcd5ccd1

$ docker ps | grep controller | grep -v POD
298f6d3edd5a        "/hyperkube controlle"   4 seconds ago       Up 4 seconds       k8s_kube-controller-manager_kube-controlle-manager-kubernetes-master_kube-system_...
```

##### Running as a systemd service

If the `controller-manager` is running as a `systemd` service, restart the service for `controller-manager`:

```sh
systemctl restart kube-controller-manager.service
```

#### API Server

##### Running as a container

If the Kubernetes `API server` is running in a container, restart the `API server` container:

```sh
$ docker ps | grep apiserver | grep -v POD
4b76d2fb16be        "/hyperkube apiserver"   40 hours ago         Up 40 hours        k8s_kube-apiserver_kube-apiserver-kubernetes-master_kube-system_...

$ docker stop 4b76d2fb16be
4b76d2fb16be

$ docker ps | grep apiserver | grep -v POD
e33dc8074088        "/hyperkube apiserver"   3 seconds ago        Up 2 seconds       k8s_kube-apiserver_kube-apiserver-kubernetes-master_kube-system_...
```

##### Running as a systemd service

If API server is running as service, restart service for API server.

```sh
systemctl restart kube-apiserver.service
```

## Securing vSphere username and password

If exposing vSphere username and password in plain text is a security concern, the username and password can be put in a [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/). This feature is available as of Kubernetes release v1.11 and more information is [available here](/vsphere-storage-for-kubernetes/documentation/k8s-secret.html).
