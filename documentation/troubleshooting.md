---
title: Troubleshooting Kubernetes vSphere Cloud Provider
---
This document is aimed to help debug issues in the Kuberentes vSphere Cloud Provider.

For general Kubernetes Issue, Please visit [https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

## What logs can I inspect?
For digging deeper and help gather data for the technical support we need to see the log files.

For vSphere Cloud Provider most often we need to look into Controller-Manager, Kubelet and API-server logs.

Logs for Controller-Manager and API server should be available on the Master node VM and Logs for Kubelet is located on both Master and Worker Nodes.

To locate the log files for Controller-Manager and API server, we need to find the `Container ID` from Pods. `Container ID` can be obtained by inspecting Pod running in the kube-system namespace or using `docer ps -a` command

Login into the master node and execute following command.

```
# kubectl describe pod kube-controller-manager-kubeadm-master --namespace=kube-system | grep "Container ID"
    Container ID:  docker://74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a
```

if you are unable to find controller-manager pod, use `docker ps -a` as below to grab `Container ID`.

```
# docker ps | grep controller-manager
74a15e75d136        gcr.io/google_containers/kube-controller-manager-amd64   "kube-controller-m..."   23 minutes ago      Up 23 minutes                           k8s_kube-controller-manager_kube-controller-manager-kubeadm-master_kube-system_22d908089b353bf8749a89843022bff3_0
12df71e6d4a8        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 23 minutes ago      Up 23 minutes                           k8s_POD_kube-controller-manager-kubeadm-master_kube-system_22d908089b353bf8749a89843022bff3_0



# docker inspect 74a15e75d136 | grep Id
        "Id": "74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a",
```
Once the `Container ID` is obtained, logs can be obtained from `/var/lib/docker/containers/` directory. Docker creates the directory for each container and keeps logs in that directory.

```
# ls /var/lib/docker/containers/74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a
74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a-json.log  checkpoints  config.v2.json  hostconfig.json
```

**Note:** On `systemd-based` setup, you may need to use `journalctl` instead. You need to know the service name to get the logs for controller-manager.
- Here is the example.
  ```
  journalctl -u kube-controller-manager.service > controller-manager.log
  ```

Kubelet generally runs as the service and not in the container or Pod in the Kubernetes cluster. Kubelet logs can be obtained using `journalctl` as shown below.

```
journalctl -u kubelet > kubelet.log
```

## How can I modify the log level?

Log levels can be adjusted using `--v` option in the pod manifests files. To help debug vSphere Cloud Provider, we recommended increasing log level for Controller-Manager. Manifest files are generally located at `/etc/kubernetes/manifests/`.

```
# cd /etc/kubernetes/manifests/
# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

Open YAML/JSON file and find `--v` option in the container's command. Increase it to 9. if this option is not present then add --v=9 to the container's command for the controller-manager, so that we can help debug issues in the vSphere Cloud Provider.

```
# cat kube-controller-manager.yaml
apiVersion: v1
kind: Pod
metadata:
.
.
spec:
  containers:
  - command:
    - kube-controller-manager
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    .
    .
    .
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --v=9
```
After increasing the log level, you need to restart the `kubelet`, which will restart the API server and Controller-Manager with updated pod manifest file.


## What information is needed to provide to technical support for analysis?

We recommend to provide controller-manager log with increased log level, kubelet logs and Kubernetes server version when you file an issue at [https://github.com/vmware/kubernetes/issues](https://github.com/vmware/kubernetes/issues)