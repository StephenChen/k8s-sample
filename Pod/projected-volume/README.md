## 特殊的 Volume - Projected Volume

project volume 是 Kubernetes v1.11 后特性

为容器提供预先定义好的数据，这些信息被 Kubernetes 投射到容器当中。  
这种方式和环境变量的方式的对比，比环境变量更优(不自动更新)。

种类：

- Secret
- ConfigMap
- Downward API
- ServiceAccountToken

### Secret

加密数据存放在 Etcd 中，通过 Pod 容器挂载 Volume 来访问。

[secret.yaml](secret.yaml)  
[secret-test.yaml](secret-test.yaml)

### ConfigMap

保存不加密的信息

[configmap.yaml](configmap.yaml)

### Downward API

让 Pod 里容器能够获取到 Pod API 对象本身的信息。  
能获取到的信息，一定是 Pod 里的容器进程启动之前就确定的信息。

[downwardapi.yaml](downwardapi.yaml)

```text
1. 使用fieldRef可以声明使用:
spec.nodeName - 宿主机名字
status.hostIP - 宿主机IP
metadata.name - Pod的名字
metadata.namespace - Pod的Namespace
status.podIP - Pod的IP
spec.serviceAccountName - Pod的Service Account的名字
metadata.uid - Pod的UID
metadata.labels['<KEY>'] - 指定<KEY>的Label值
metadata.annotations['<KEY>'] - 指定<KEY>的Annotation值
metadata.labels - Pod的所有Label
metadata.annotations - Pod的所有Annotation

2. 使用resourceFieldRef可以声明使用:
容器的CPU limit
容器的CPU request
容器的memory limit
容器的memory request
```

### ServiceAccountToken

一个特殊的 Secret

`kubectl describe pod nginx-deployment-test`

挂载: `/var/run/secrets/kubernetes.io/serviceaccount`, 包含 `ca.crt namespace token`

**把 Kubernetes 客户端以容器方式运行在集群里， 使用 default ServiceAccount 自动授权的方式， 称为"InClusterConfig"，最为推荐的进行 Kubernetes API 编程的授权方式**

### 常用命令：

```shell
kubectl create secret generic user --from-file=./username.txt

kubectl create -f xxx.yaml

kubectl get secrets

kubectl exec -it projected-volume -- /bin/sh
ls /projected-volume/

kubectl create configmap ui-config --from-file=ui.properties
kubectl get configmaps ui-config -o yaml

kubectl create -f dapi-volume.yaml
kubectl logs test-downwardapi-volume
```
