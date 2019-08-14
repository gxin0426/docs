### 1.operator 加入etcd

```yaml
#创建etcd的secret
kubectl -n monitoring create secret generic etcd-certs --from-file=/etc/kubernetes/ssl/ca.pem --from-file=/etc/kubernetes/ssl/etcd.pem --from-file=/etc/kubernetes/ssl/etcd-key.pem

#查看secret
kubectl -n monitoring get secrets etcd-certs

#使Prometheus Operator接入secret
vim manifests/prometheus/prometheus-k8s.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: k8s
  labels:
    prometheus: k8s
spec:
  replicas: 2
  secrets:
  - etcd-certs
  version: v2.2.1

#创建Service、Endpoints和ServiceMonitor服务
vim manifests/prometheus/prometheus-etcd.yaml 
apiVersion: v1
kind: Service
metadata:
  name: etcd-k8s
  labels:
    k8s-app: etcd
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: api
    port: 2379
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: etcd-k8s
  labels:
    k8s-app: etcd
subsets:
- addresses:
  - ip: 192.168.100.102
    nodeName: etcd1
  - ip: 192.168.100.103
    nodeName: etcd2
  - ip: 192.168.100.104
    nodeName: etcd3
  ports:
  - name: api
    port: 2379
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: etcd-k8s
  labels:
    k8s-app: etcd-k8s
spec:
  jobLabel: k8s-app
  endpoints:
  - port: api
    interval: 30s
    scheme: https
    tlsConfig:
      caFile: /etc/prometheus/secrets/etcd-certs/ca.pem
      certFile: /etc/prometheus/secrets/etcd-certs/etcd.pem
      keyFile: /etc/prometheus/secrets/etcd-certs/etcd-key.pem
      #use insecureSkipVerify only if you cannot use a Subject Alternative Name
      insecureSkipVerify: true 
  selector:
    matchLabels:
      k8s-app: etcd
  namespaceSelector:
    matchNames:
    - monitoring




```

### 2.设置 Kubernetes 资源限制 and 压测工具

https://www.jianshu.com/p/352909c038f8



