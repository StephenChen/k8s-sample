## 容器健康检查和恢复机制

定义健康检查"探针"(Probe)，是生产环境中保证应用健康存活的重要手段。

通过 Pod 的 Events 和 Status 可以看到异常和 RESTARTS 增加。

但 Kubernetes 没有 Docker 的 Stop 语义，所以虽然是 RESTART 但实际上是重新创建容器。

此为 Kubernetes 中的 Pod 恢复机制，restartPolicy，是 Pod Spec 的一个标准字段(pod.spec.restartPolicy)，默认为 Always。

Pod 的恢复过程，永远发生在当前节点(Node)，除非绑定发生变化(pod.sepc.node)，意味着宿主机宕机，Pod 也不会主动迁移到其他节点。

如果要使 Pod 出现在其他节点，需使用 Deployment 来管理 Pod。

- restartPolicy:
    - Always: 任何情况，只要容器不在运行状态，就自动重启容器
    - OnFailure: 只在容器异常时才重启容器
    - Never: 从不重启容器

比如关心容器退出后的上下文环境(日志、文件、目录)，就需要设置 Never。因为如果重建，内容可能丢失(被垃圾回收)。

Kubernetes 官方文档，有 restartPolicy 和 Pod 及 Pod
里容器状态做了对应，[Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#example-states)

只要记住**以下两个基本设计原理：**

- 只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器(比如：Always)，那么这个 Pod 就会保持 Running 状态，并进行容器重启。否则，Pod 就会进入 Failed 状态。
- 对于包含多个容器的 Pod，只有它里面的所有容器都进入异常状态后，Pod 才会进入 Failed 状态。在此之前，Pod 都是 Running 状态。此时，Pod 的 READY 字段会显示正常容器的个数。

#### livenessProbe

除了指定命令外，也可以定义为发起 HTTP 或 TCP 请求的方式。

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    prot: 8080
    httpHeaders:
      - name: X-Custom-Header
        value: Awesome
    initialDelaySeconds: 3
    periodSeconds: 3
```

```yaml
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```

使用命令：

```shell
kubectl create -f test-liveness-exec.yaml

kubectl get pod

kubectl describe pod test-liveness-exec
```