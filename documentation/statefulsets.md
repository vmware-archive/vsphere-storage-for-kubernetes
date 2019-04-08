---
title: StatefulSets
---

StatefulSets are valuable for applications which require any stable identifiers or stable storage. vSphere Cloud Provider supports StatefulSets and vSphere volumes can be consumed by StatefulSets.

**Note:**

All the example yamls can be found [here](https://github.com/Kubernetes/Kubernetes/tree/master/examples/volumes/vsphere) unless otherwise specified. Please download these examples.

**Create a storage class that will be used by the volumeClaimTemplates of a Stateful Set**

```
#simple-storageclass.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: thin-disk
provisioner: kubernetes.io/vsphere-volume
parameters:
    diskformat: thin
```

**Create a Stateful set that consumes storage from the Storage Class created**

```
#simple-statefulset.yaml

---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---


apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 14
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.beta.kubernetes.io/storage-class: thin-disk
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

This will create Persistent Volume Claims for each replica and provision a volume for each claim if an existing volume could not be bound to the claim.
