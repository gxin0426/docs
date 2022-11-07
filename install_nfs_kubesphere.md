## install nfs

```
#yum install -y nfs-utils 
#mkdir /data/volumes/v1
#systemctl start rpcbind
#systemctl status rpcbind
#systemctl enable --now nfs-serve
#vim /etc/exports
/data/volumes/v1 *
#exportfs -arv
```

## install storageclass

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
  namespace: nfs-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-runner
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list", "watch", "create", "delete"]
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "update"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-provisioner
subjects:
- kind: ServiceAccount
  name: nfs-provisioner
   # replace with namespace where provisioner is deployed
  namespace: nfs-provisioner
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-provisioner
  # replace with namespace where provisioner is deployed
  namespace: nfs-provisioner
rules:
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-provisioner
subjects:
- kind: ServiceAccount
  name: nfs-provisioner
   # replace with namespace where provisioner is deployed
  namespace: nfs-provisioner
roleRef:
  kind: Role
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      annotations:
        storageclass.kubernetes.io/is-default-class: "true"
      name: nfs
    provisioner: example.com/nfs
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    ```

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-provisioner
  namespace: nfs-provisioner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-provisioner
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-provisioner
    spec:
      serviceAccount: nfs-provisioner
      containers:
        - name: nfs-provisioner
          image: registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: example.com/nfs
            - name: NFS_SERVER
              value: 192.168.11.186
            - name: NFS_PATH
              value: /data/volumes/v1
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.11.186
            path: /data/volumes/v1
```

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
  namespace: default
spec:
  storageClassName: nfs-storage
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256Mi
```

