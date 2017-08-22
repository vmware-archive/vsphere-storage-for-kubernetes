## vSphere Cloud Provider Overview
Containers are stateless and ephemeral but applications are stateful and need persistent storage. vSphere adds this persistent storage support to Kubernetes through interface called Cloud Provider. Cloud provider is an interface which helps in extending Kubernetes with cluster of instances managed by virtualization technologies, public/private cloud platforms and required networking for these instances.
  
Kubernetes cloud provider is an interface to integrate various nodes (i.e. hosts), load balancers and networking routes. This interface allows extending Kubernetes to use various cloud and virtualization solutions as base infrastructure to run on. 

<img src="documentation/images/vSphere.png" width="70%" height="70%"/>

vSphere is one of the cloud providers of Kubernetes and thus allows Kubernetes Pods use enterprise grade storage. vSphere Storage (VMFS, vSAN, NFS) has proven features like policy based management, QoS, high availability and data reliability for containers using Kubernetes.

## Detailed documentation

Detailed documentation can be found on our [GitHub Documentation Page](http://vmware.github.io/vsphere-storage-for-kubernetes/documentation/).

## Contributing

The vsphere-storage-for-kubernetes project team welcomes contributions from the community. If you wish to contribute code and you have not
signed our contributor license agreement (CLA), our bot will update the issue when you open a Pull Request. For any
questions about the CLA process, please refer to our [FAQ](https://cla.vmware.com/faq). For more detailed information,
refer to [CONTRIBUTING.md](CONTRIBUTING.md).


## Contact Us
You can reach us via:

* [containers@vmware.com](containers@vmware.com)
* [Issues](https://github.com/vmware/kubernetes/issues)
* [VMware{code} Kubernetes Slack](https://vmwarecode.slack.com/messages/kubernetes) channel

