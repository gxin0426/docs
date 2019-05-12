##部署Traefik

Traefik是一款开源的反向代理和负载均衡工具。最大的优点是能够与常见的微服务系统直接整合，可以实现自动化动态配置。目前支持Docker，swarm Mesos/Marathon k8s Consul Etcd Zoopkeeper。

- 创建Service account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata: 
  name: ingress
  namespace: kube-system
---
kind:ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: ingress
subjects: 
  - kind: ServiceAccount
    name: ingress
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

- 创建deployment(jimmysong)部署traefik（个人感觉最好使用daemonset方式部署或者使用hostNetwork：true副本数等于节点数的方式部署）

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: traefik-ingress-lb
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      restartPolicy: Always
      serviceAccountName: ingress
      containers:
      - image: traefik
        name: traefik-ingress-lb
        resources:
          limits:
            cpu: 200m
            memory: 30Mi
          requests:
            cpu: 100m
            memory: 20Mi
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8580
          hostPort: 8580
        args:
        - --web
        - --web.address=:8580
        - --kubernetes
```

- 创建nginx的ingress(已经部署nginx和service)

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata: 
  name: nginx-ing
  namespace: kube-system
spec: 
  rules:
  - host: nginx-ing.local
    http: 
      paths: 
      - path: /
        backend:
          serviceName: nginx-svc
          servicePort: 80
```

- 查看traefik部署在哪个节点使用curl命令验证是否部署成功

```shell
$ curl -H Host:nginx-ing.local http://10.2.22.64/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

