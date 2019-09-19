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

