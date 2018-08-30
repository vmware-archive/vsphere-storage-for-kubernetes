---
  title: Configurations on Existing Kubernetes Cluster
---
**Prerequisites**

* All node VMs must be placed in vSphere VM folder. Create a VM folder following the instructions mentioned in this [link](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.vcenterhost.doc/GUID-031BDB12-D3B2-4E2D-80E6-604F304B4D0C.html) and move Kubernetes Node VMs to this folder.
* The disk UUID on the node VMs must be enabled: the `disk.EnableUUID` value must be set to `True`. This step is necessary so that the VMDK always presents a consistent UUID to the VM, thus allowing the disk to be mounted properly. For each of the virtual machine nodes that will be participating in the cluster, follow the steps below using [govc](/vsphere-storage-for-kubernetes/documentation/prerequisites.html)
   * Find Node VM Paths
        `govc ls /datacenter/vm/<vm-folder-name>`
   * Set disk.EnableUUID to true for all VMs
        `govc vm.change -e="disk.enableUUID=1" -vm='VM Path'`

     Note: If Kubernetes Node VMs are created from template VM then `disk.EnableUUID=1` can be set on the template VM. VMs cloned from this template, will automatically inherit this property.

**Prerequisites for Kubernetes version is 1.8.x or below.**

* Node host names must comply with the regex `[a-z](([-0-9a-z]+)?[0-9a-z])?(\.[a-z0-9](([-0-9a-z]+)?[0-9a-z])?)*` and must also comply with these restrictions:
  * They must not begin with numbers.
  * They must not use capital letters.
  * They must not have any special characters except `.` and `-`.
  * They must contain at least three characters but no more than 63 characters.

After prerequisites are met, follow below-mentioned steps to enable vSphere Cloud Provider.

## 1. Create and assign roles to the vSphere Cloud Provider user and vSphere entities.

The first step is to create and assign roles to the vSphere Cloud Provider user and vSphere entities. Please refer Roles and Privileges documented [here](/vsphere-storage-for-kubernetes/documentation/vcp-roles.html).

If vCenter Administrator account is going to be used by vSphere Cloud Provider, then this step can be skipped. Please refer [vSphere Documentation Center](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.security.doc/GUID-18071E9A-EED1-4968-8D51-E0B4F526FDA3.html) to know about steps for creating a Custom Role, User, and Role Assignment.


## 2. Create the vSphere cloud config file (vsphere.conf).
vSphere Cloud Provider config file needs to be placed in the shared directory which should be accessible from kubelet, controller-manager, and API server.

### vSphere cloud config file for Kubernetes version 1.9.x and above.
For version 1.9.x and above `vsphere.conf` file should be placed only on master nodes.

* Sample configuration for Kubernetes Cluster for which all Kubernetes nodes are located on the single vCenter.

```
[Global]
user = "Administrator1@vsphere.local"
password = "password"
port = "443"
insecure-flag = "1"
datacenters = "us-east"

[VirtualCenter "1.1.1.1"]

[Workspace]
server = "1.1.1.1"
datacenter = "us-east"
default-datastore="sharedVmfs-0"
resourcepool-path=""
folder = "kubernetes"

[Disk]
scsicontrollertype = pvscsi

[Network]
public-network = "VM Network"
```

* Sample configuration for Kubernetes Cluster for which Kubernetes nodes are located on multiple vCenter/Datacenters.

```
[Global]
user = "Administrator@vsphere.local"
password = "password"
port = "443"
insecure-flag = "1"
datacenters = "us-east, us-west"

[VirtualCenter "1.1.1.1"]
user = "Administrator2@vsphere.local"
password = "password2"
datacenters = "us-east"

[VirtualCenter "1.1.1.2"]
user = "Administrator3@vsphere.local"
password = "password3"
datacenters = "us-west"

[VirtualCenter "1.1.1.3"]
[VirtualCenter "1.1.1.4"]

[Workspace]
server = "1.1.1.1"
datacenter = "us-east"
default-datastore="sharedVmfs-0"
resourcepool-path=" "
folder = "kubernetes"

[Disk]
scsicontrollertype = pvscsi

[Network]
public-network = "VM Network"
```

Below is the summary of supported parameters in the `vsphere.conf` file for Kubernetes version 1.9.x

* Properties in ```Global``` section will be used for all specified vCenters unless overridden in ```VirtualCenter``` section.
* ```port``` is the vCenter Server Port. The default is 443 if not specified.
* ```insecure-flag``` should be set to 1 if the vCenter uses a self-signed cert.
* ```datacenters``` should be the list of all comma separated datacenters where Kubernetes node VMs are present.
* ```default-datastore``` is the default datastore to use for provisioning volumes using storage classes/dynamic provisioning.
* ```Workspace``` Workspace is used by vSphere Cloud Provider to know which virtual center, datacenter, folder, resourcepool pool to use for dynamic provisioning volumes using SPBM profile.
   * ```server``` is the virtual center server on which the dummy/shadow VM for storage profile based volumes should be created.
   * ```datacenter``` is the name of datacenter on which the dummy/shadow VM should be created.
   * ```folder``` is the virtual center VM folder path under which the dummy VMs should be placed.
   * ```resourcepool-path``` is the path to resource pool where dummy VMs for Storage Profile Based volume provisioning should be created.

If exposing vsphere username and password in plain text, is the security concern, username and password can be put in the [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/).
This feature is available as of Kubernetes release 1.11.

Steps for storing vSphere credentials in the Kubernetes Secret.

* Create vsphere.conf file with secret-name and secret-namespace.

```
[Global]
insecure-flag = 1
secret-name = "vcconf"
secret-namespace = "kube-system"

[VirtualCenter "1.1.1.1"]
port = 443
datacenters = k8s-dc-1

[Workspace]
server = 1.1.1.1
datacenter = datacenter
default-datastore = shareddatastore
folder = kubernetes
```

* Launch Kubernetes cluster with vSphere Cloud Provider Configured.
* Create secret with vCenter credentials.
   * Create base64 encoding for username and password

```
$ echo -n 'Administrator@vsphere.local' | base64
QWRtaW5pc3RyYXRvckB2c3BoZXJlLmxvY2Fs

$ echo -n 'password' | base64
cGFzc3dvcmQ=
```

   * Create a vccredentials.yaml as mentioned below.

```
apiVersion: v1
kind: Secret
metadata:
 name: vcconf
type: Opaque
data:
   1.1.1.1.username: QWRtaW5pc3RyYXRvckB2c3BoZXJlLmxvY2Fs
   1.1.1.1.password: cGFzc3dvcmQ=
```

   * Execute ```kubectl create -f vccredentials.yaml --namespace=kube-system```


vSphere credentials can also be encrypted using **SAML token authentication**, Please refer documentation for [SAML token authentication](/vsphere-storage-for-kubernetes/documentation//vsphere-storage-for-kubernetes/documentation/saml-token-authentication.md) using vCenter SSO API.


### vSphere cloud config file for Kubernetes version 1.8.x or below

vSphere Cloud Provider config file needs to be placed in the shared directory which should be accessible from the kubelet container, controller-manager pod, and API server pod.

**```vsphere.conf``` for Master Node:**

```
[Global]
        user = "vCenter username for cloud provider"
        password = "password"
        server = "IP/FQDN for vCenter"
        port = "443" #Optional
        insecure-flag = "1" #set to 1 if the vCenter uses a self-signed cert
        datacenter = "Datacenter name"
        datastore = "Datastore name" #Datastore to use for provisioning volumes using storage classes/dynamic provisioning
        working-dir = "vCenter VM folder path in which node VMs are located"
        vm-name = "VM name of the Master Node" #Optional
        vm-uuid = "UUID of the Node VM" # Optional
[Disk]
    scsicontrollertype = pvscsi
```

Note: **```vm-name``` parameter is introduced in 1.6.4 release.** Both ```vm-uuid``` and ```vm-name``` are optional parameters. If ```vm-name``` is specified then ```vm-uuid``` is not used. If both are not specified then kubelet will get vm-uuid from `/sys/class/dmi/id/product_serial` and query vCenter to find the Node VM's name.

**```vsphere.conf``` for Worker Nodes:** (Only Applicable to 1.6.4 release and above. For older releases this file should have all the parameters specified in Master node's ```vSphere.conf``` file)

```
[Global]
        vm-name = "VM name of the Worker Node"
```

Below is the summary of supported parameters in the `vsphere.conf` file

* ```user``` is the vCenter username for vSphere Cloud Provider.
* ```password``` is the password for vCenter user specified with `user`.
* ```server``` is the vCenter Server IP or FQDN
* ```port``` is the vCenter Server Port. The default is 443 if not specified.
* ```insecure-flag``` is set to 1 if vCenter used a self-signed certificate.
* ```datacenter``` is the name of the datacenter on which Node VMs are deployed.
* ```datastore``` is the default datastore to use for provisioning volumes using storage classes/dynamic provisioning.
* ```vm-name``` This is optional parameter. When this parameter is present, ```vsphere.conf``` file on the worker node does not need vCenter credentials.

  **Note:** ```vm-name``` is added in the release 1.6.4. Prior releases do not support this parameter.

* ```working-dir``` can be set to empty ( working-dir = ""), if Node VMs are located in the root VM folder.
* ```vm-uuid``` is the VM Instance UUID of virtual machine. ```vm-uuid``` can be set to empty (```vm-uuid = ""```). If set to empty, this will be retrieved from /sys/class/dmi/id/product_serial file on virtual machine (requires root access).
  * ```vm-uuid``` needs to be set in this format - ```423D7ADC-F7A9-F629-8454-CE9615C810F1```
  * ```vm-uuid``` can be retrieved from Node Virtual machines using the following command. This will be different on each node VM.

        cat /sys/class/dmi/id/product_serial | sed -e 's/^VMware-//' -e 's/-/ /' | awk '{ print toupper($1$2$3$4 "-" $5$6 "-" $7$8 "-" $9$10 "-" $11$12$13$14$15$16) }'

* `datastore` is the default datastore used for provisioning volumes using storage classes. If datastore is located in storage folder or datastore is the member of datastore cluster, make sure to specify full datastore path. Make sure vSphere Cloud Provider user has Read Privilege set on the datastore cluster or storage folder to be able to find datastore.
  * For datastore located in the datastore cluster, specify datastore as mentioned below
        ```
        datastore = "DatastoreCluster/datastore1"
        ```
  * For datastore located in the storage folder, specify datastore as mentioned below
        ```
        datastore = "DatastoreStorageFolder/datastore1"
        ```

## 3. Add flags to controller-manager, API server and Kubelet

### For Kubernetes version 1.9.x

* Add following flags to the kubelet configuration, controller-manager manifest file and API server manifest file on the master node.

    ```
    --cloud-provider=vsphere
    --cloud-config=<Path of the vsphere.conf file>
    ```

* Add following flags to kubelet running on each worker node. On the worker node, we do not require vsphere.conf file hence `--cloud-config=` flag should not be set for Kubelet for worker nodes.

    ```
    --cloud-provider=vsphere
    ```

### For Kubernetes version 1.8.x or below

* Add following flags to kubelet running on all nodes and controller-manager's and API server's manifest file on the master node.

    ```
    --cloud-provider=vsphere
    --cloud-config=<Path of the vsphere.conf file>
    ```

## 4. Restart Controller-Manager, API Server and Kubelet on all nodes.

* Reload kubelet systemd unit file using ```systemctl daemon-reload```
* Restart kubelet service using ```systemctl restart kubelet.service```

If Controller-Manager and API Server is running in the containers, kill and restart Controller-Manager and API Server containers after restarting kubelet on the node.

**Note: For Kubernetes version 1.8.x or below, after enabling the vSphere Cloud Provider, Node names will be set to the VM names from the vCenter Inventory**
