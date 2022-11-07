## Kubeflow  manifest部署教程

### git repo

https://github.com/kubeflow/manifests.git

### 工具

- kubernetes （1.19） 带一个storageclass
- kustomize （v4.4.1）
- Kubeflow (1.3.0)

### 默认邮箱密码

email (`user@example.com`) and password (`12341234`)

### 一条命令安装

```shell
while ! kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

### 分步安装

安装apps（官方组件）目录和common（通用服务）目录下的组件。

app目录

| Admission-webhook | Centraldashboard | Jupiter  | Katie       | kfp-tekton   | Kfserving    | Kubebench      | Mpi-job     |
| ----------------- | :--------------: | -------- | ----------- | ------------ | ------------ | -------------- | ----------- |
| Mxnet-job         |     Pipeline     | Profiles | Pytorch-job | Tensor board | Tf-trainning | Volume-app-web | Xgboost-job |

common目录

| Cert-manager       | Dex            | Istio           | Istio-1.9.0    | knative |
| ------------------ | -------------- | --------------- | -------------- | ------- |
| Kubeflow-namespace | Kubeflow-roles | Idc-authservice | User-namespace |         |

#### cert-manager

Cert-manager为kubeflow组件提供admiss-webhook需要的证书

```bash
kustomize build common/cert-manager/cert-manager-kube-system-resources/base | kubectl apply -f -
kustomize build common/cert-manager/cert-manager-crds/base | kubectl apply -f -
kustomize build common/cert-manager/cert-manager/overlays/self-signed | kubectl apply -f -
```

#### istio

Istio被许多kubeflow组件用来保护流量，网络认证和路由策略

```
kustomize build common/istio-1-9-0/istio-crds/base | kubectl apply -f -
kustomize build common/istio-1-9-0/istio-namespace/base | kubectl apply -f -
kustomize build common/istio-1-9-0/istio-install/base | kubectl apply -f -
```

#### Dex

用来身份认证（默认用户名密码：user，12341234）

```
kustomize build common/dex/overlays/istio | kubectl apply -f -
```

#### OIDC Authservice

OIDC AuthService扩展了你的Istio Ingress-Gateway能力，以便能够作为OIDC客户端运行

```
kustomize build common/oidc-authservice/base | kubectl apply -f -
```

#### Knative

KFServing官方Kubeflow组件使用Knative

```
kustomize build common/knative/knative-serving-crds/base | kubectl apply -f -
kustomize build common/knative/knative-serving-install/base | kubectl apply -f -
kustomize build common/istio-1-9-0/cluster-local-gateway/base | kubectl apply -f -
```

你可以选择安装Knative Eventing，它可以用于推理请求记录

```
kustomize build common/knative/knative-eventing-crds/base | kubectl apply -f -
kustomize build common/knative/knative-eventing-install/base | kubectl apply -f -
```

#### Kubeflow Namespace

Create kubeflow namespace

```
kustomize build common/kubeflow-namespace/base | kubectl apply -f -
```

#### Kubeflow Roles

创建需要的角色

创建Kubeflow ClusterRoles：kubeflow-view、kubeflow-edit和kubeflow-admin。Kubeflow组件将权限聚集到这些ClusterRoles

```
kustomize build common/kubeflow-roles/base | kubectl apply -f -
```

#### Kubeflow Istio Resources

创建kubeflow需要的istio网关

```
kustomize build common/istio-1-9-0/kubeflow-istio-resources/base | kubectl apply -f -
```

#### Kubeflow Pipelines

安装多用户pipeline

```
kustomize build apps/pipeline/upstream/env/platform-agnostic-multi-user | kubectl apply -f -
```

如果runtime不是docker，用pns executor代替

```
kustomize build apps/pipeline/upstream/env/platform-agnostic-multi-user-pns | kubectl apply -f -
```

**Multi-User Kubeflow Pipelines 依赖：**

- Istio + Kubeflow Istio Resources
- Kubeflow Roles
- OIDC Auth Service (or cloud provider specific auth service)
- Profiles + KFAM

也可以安装standalone模式

```
export PIPELINE_VERSION=1.5.0
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"
kubectl wait --for condition=established --timeout=60s crd/applications.app.k8s.io
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/dev?ref=$PIPELINE_VERSION"
```

- 不支持多用户分离
- 对这里提到的其他服务没有依赖性

#### Central dashboard

```
kustomize build apps/centraldashboard/upstream/overlays/istio | kubectl apply -f -
```

#### KFServing

```
kustomize build apps/kfserving/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Katib

```
kustomize build apps/katib/upstream/installs/katib-with-kubeflow | kubectl apply -f -
```

#### Admission Webhook

```
kustomize build apps/admission-webhook/upstream/overlays/cert-manager | kubectl apply -f -
```

#### Notebooks

```
kustomize build apps/jupyter/notebook-controller/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Profiles + KFAM

```
kustomize build apps/profiles/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Tensorboard

```
kustomize build apps/tensorboard/tensorboards-web-app/upstream/overlays/istio | kubectl apply -f -
# Tensorboard Controller 
kustomize build apps/tensorboard/tensorboard-controller/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Operator

```
kustomize build apps/tf-training/upstream/overlays/kubeflow | kubectl apply -f -
kustomize build apps/pytorch-job/upstream/overlays/kubeflow | kubectl apply -f -
kustomize build apps/mpi-job/upstream/overlays/kubeflow | kubectl apply -f -
kustomize build apps/mxnet-job/upstream/overlays/kubeflow | kubectl apply -f -
kustomize build apps/xgboost-job/upstream/overlays/kubeflow | kubectl apply -f -
```

#### User Namespace

```
kustomize build common/user-namespace/base | kubectl apply -f -
```

