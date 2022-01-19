### 水平扩展

```yaml
kubectl scale deployment nginx-deployment --replicas=4
```

### 滚动更新

`kubectl get deployments`  
nginx-deployment 创建后状态信息的四个状态字段：

- DESIRED：用户期望的 Pod 副本个数(spec.replicsa)；
- CURRENT：当前处于 Running 状态的 Pod 的个数；
- UP-TO-DATE：当前处于最新版本的 Pod 的个数，所谓最新版本指的是 Pod 的 Spec 部分与 Deployment 里 Pod 模板里定义的完全一致；
- AVAILABLE：当前已经可用的 Pod 的个数，即：既是 Running 状态，又是最新版本，并且已经处于 Ready（健康检查正确）状态的 Pod 的个数。

`kubectl rollout status deployment/nginx-deployment`  
实时查看 Deployment 对象的状态变化

`kubectl get rs`  
查看 Deployment 所控制的 ReplicaSet  
ReplicaSet 的名字，由 Deployment 的名字和 pod-template-hash。  
ReplicaSet 会把这个随机字符串加在控制的所有 Pod 的标签里，保证不会和其他 Pod 混淆。

Deployment 只是在 ReplicaSet 的基础上，添加了 UP-TO-DATE 这个跟版本有关的状态字段。

```shell

$ kubectl describe deployment nginx-deployment
...
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
...
  Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 1
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 2
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 2
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 1
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 3
  Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 0
```

新ReplicaSet(hash=1764197365)。

在 Age=24s，Deployment Controller 将新 ReplicaSet 所控制的 Pod 副本加1，"水平扩展"一个副本，  
在 Age=22s，将旧 ReplicaSet 所控制的 Pod 副本减1，"水平收缩"一个副本，以此类推。

将一个集群中正在运行的多个 Pod 版本，交替地逐一升级的过程，即"滚动更新"。

"滚动更新"完成后，可查看新旧两个 ReplicaSet 最终状态。

```shell
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1764197365   3         3         3       6s
nginx-deployment-3167673210   0         0         0       30s
```

好处：

- 升级刚开始，只有一个新版本 Pod，不会因为启动失败而影响服务。
- 使用 Pod 的 Health Check 机制检查应用的运行状态，而不是依赖容器的 Running 状态。
- Deployment Controller 确保在任何时间窗口内，只有指定比例的 Pod 处于离线状态，只有指定比例的新 Pod 被创建。这两个比例可以配置，默认是 DESIRED 的值 25%。

```yaml
kubectl create -f nginx-deployment.yaml --record # 记录每次操作所执行的命令

kubectl get deployments

kubectl rollout status deployment/nginx-deployment

kubectl get rs
```

Canary Deployment & Blue-Green Deployment

- 