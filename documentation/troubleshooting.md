---
title: Troubleshooting
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---
To troubleshoot issues with Kuberenetes cluster deployment, please visit [https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

## Logs to inspect
Following logs are needed to debug VCP issues

 * Controller-Manager logs
 * API-server logs
 * Kubelet logs

### Controller-Manager Logs
Login to the master node and execute the following command.

```
# kubectl describe pod <Controller-Manager Pod Name> --namespace=kube-system | grep "Container ID"
    Container ID:  docker://74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a
```
Note: For the above command name of the Controller-Manager Pod can be retrived using `kubectl get pods --namespace=kube-system` command. If the controller-manager pod is not found, use `docker ps -a` as below to grab the `Container ID` of the controller-manager pod.

```
# docker ps | grep controller-manager
74a15e75d136        gcr.io/google_containers/kube-controller-manager-amd64   "kube-controller-m..."   23 minutes ago      Up 23 minutes                           k8s_kube-controller-manager_kube-controller-manager-kubeadm-master_kube-system_22d908089b353bf8749a89843022bff3_0
12df71e6d4a8        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 23 minutes ago      Up 23 minutes                           k8s_POD_kube-controller-manager-kubeadm-master_kube-system_22d908089b353bf8749a89843022bff3_0

# docker inspect 74a15e75d136 | grep Id
        "Id": "74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a",
```

Once the `Container ID` for controller manager is obtained, logs can be obtained from the `/var/lib/docker/containers/` directory.

```
# ls /var/lib/docker/containers/74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a
74a15e75d1365164cec2c005030a7957ddb3d3c9ee487c6b40df35b0a4e4f95a-json.log  checkpoints  config.v2.json  hostconfig.json
```

**Note:** On the `systemd-based` setup, `journalctl` should be used.
- Below is the command to collect controller-manager logs.
  ```
  journalctl -u kube-controller-manager.service > controller-manager.log
  ```

### API Server Logs
Login to the master node and execute the following command.

```
# kubectl describe pod <API Server Pod Name> --namespace=kube-system | grep "Container ID"
    Container ID:  docker://b6fdc5d41e50b22406c411d709f64cda7442545f5d5872145f27ba0fc4dd501c
```
Note: For the above command name of the API server Pod can be retrived using `kubectl get pods --namespace=kube-system` command.
If the API Server pod is not found, use `docker ps -a` as below to grab the `Container ID` of the API-Server pod.

```
# docker ps -a | grep apiserver
b6fdc5d41e50        gcr.io/google_containers/kube-apiserver-amd64            "kube-apiserver --..."   5 hours ago         Up 5 hours                              k8s_kube-apiserver_kube-apiserver-kubeadm-master_kube-system_16e5059388b998793f4191a20f2de9c2_1
abe6e03e5bd5        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 5 hours ago         Up 5 hours                              k8s_POD_kube-apiserver-kubeadm-master_kube-system_16e5059388b998793f4191a20f2de9c2_1


# docker inspect b6fdc5d41e50 | grep Id
        "Id": "b6fdc5d41e50b22406c411d709f64cda7442545f5d5872145f27ba0fc4dd501c",
```

Once the `Container ID` for API server is obtained, logs can be obtained from the `/var/lib/docker/containers/` directory.

```
# ls /var/lib/docker/containers/b6fdc5d41e50b22406c411d709f64cda7442545f5d5872145f27ba0fc4dd501c
b6fdc5d41e50b22406c411d709f64cda7442545f5d5872145f27ba0fc4dd501c-json.log  checkpoints  config.v2.json  hostconfig.json
```

### Kubelet Logs
Kubelet runs as the service and not in the container or Pod in the Kubernetes cluster. Kubelet logs can be obtained using `journalctl` as shown below.

```
journalctl -u kubelet > kubelet.log
```


## Modify log level
Log levels can be adjusted using `--v` option in the pod manifests files. To help debug vSphere Cloud Provider, It is recommended to increase the log level for Controller-Manager. Manifest files are generally located at `/etc/kubernetes/manifests/`.

```
# cd /etc/kubernetes/manifests/
# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

Open YAML/JSON file and find `--v` option in the container's command. Increase value for this option to 9. If this option is not present then add --v=9 to the container's command for the controller-manager. This helps debug issues in the vSphere Cloud Provider.

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
After increasing the log level, Kubelet needs to be restarted. When Kubelet is restarted, the API server and Controller-Manager Pods are also restarted with updated manifest files. To restart the Kubelet use following command.

```
systemctl restart kubelet
```

## Details of a specific Resource

In addition to logs, the output of `kubectl describe` command on targeted resources like pod, pvc, pv can be captured. It helps to narrow down the problem quickly.

```
kubectl describe pod <podname> --namespace=<namespace_name>
kubectl describe pvc <pvcname> --namespace=<namespace_name>
kubectl describe pv <pvname> --namespace=<namespace_name>
```
