### 1.两个pod公用一个卷通信

~~~yaml
apiVersion: v1
kind: Pod
metadata: 
  name: two-containers
  labels: 
    app: test
spec: 
  restartPolicy: Never
  volumes:
  - name: share-data
    emptyDir: {}
  containers: 
  - name: nginx-container
    image: nginx
    volumeMounts: 
    - name: share-data
      mountPath: /usr/share/nginx/html
  - name: debian
    image: debian
    volumeMounts: 
    - name: share-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c","echo hello >/pod-data/index.html"]
~~~

### 2.标准pod

~~~yaml
apiVersion: v1
kind: Pod
metadata: 
  name: nginx
  labels: 
    app: nginx
spec: 
  containers: 
  - name: nginx
    image: nginx
    ports: 
    - containerPort: 80
~~~

### 3.标准service

~~~yaml
apiVersion: v1
kind: Service
metadata: 
  name: nginx-service
spec: 
  type: NodePort
  ports: 
  - port: 80
    nodePort: 30001
  selector:
    app: nginx
~~~

### 4.deployment

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: hello-dep
  labels: 
    app: hello
spec: 
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata: 
      labels:  
        app: hello  
    spec: 
      containers:  
      - name: hello  
        image: hello  
        ports:  
        - name: http 
          containerPort: 80 
~~~

### 5. statefulset 带 ceph pvc

~~~yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      storageClassName: ceph-rbd-storage
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
~~~

- apps/v1

~~~yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  namespace: redis
  name: redis-app
spec:
  serviceName: "redis-service"
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
        labels:
          app: redis
    spec:
      terminationGracePeriodSeconds: 20
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - redis
              topologyKey: kubernetes.io/hostname
      containers:
      - name: redis
        image: "registry.cn-qingdao.aliyuncs.com/gold-faas/gold-redis:1.0"
        command:
          - "redis-server"
        args:
          - "/etc/redis/redis.conf"
          - "--protected-mode"
          - "no"
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
        ports:
            - name: redis
              containerPort: 6379
              protocol: "TCP"
            - name: cluster
              containerPort: 16379
              protocol: "TCP"
        volumeMounts:
          - name: "redis-conf"
            mountPath: "/etc/redis"
          - name: "redis-data"
            mountPath: "/var/lib/redis"
      volumes:
      - name: "redis-conf"
        configMap:
          name: "redis-conf"
          items:
            - key: "redis.conf"
              path: "redis.conf"
      - name: "redis-data"
        emptyDir: {}
~~~





### 6.声明pvc和测试pvc的pod

~~~yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim2
  annotations:
    volume.beta.kubernetes.io/storage-class: "ceph-rbd-storage"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Mi
#pod
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox:1.24
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-claim
~~~

