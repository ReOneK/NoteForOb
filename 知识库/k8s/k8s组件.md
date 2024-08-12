## apiserver

#### 基础流程
1. [[pod的生命周期管理#apiserver|基础概念]]  

#### default ns下面内置的kubernetes svc的作用
	1. 是由 kube-apiserver 中的 bootstrap controller 进行控制的
	2. 代表 Kubernetes API Server

	bootstrap controller的主要作用：
		- 创建 kubernetes service；
		- 创建 default、kube-system 和 kube-public 命名空间
		- 提供基于 Service ClusterIP 的修复及检查功能
		- 提供基于 Service NodePort 的修复及检查功能

#### 多个apiserver实例是如何统一管理的
1. 一个集群中 apiserver 的所有实例会在 etcd 中的对应目录下创建 key，并定期更新这个 key 来上报自己的心跳信息。
2. ReconcileEndpoints 会从 etcd 中获取 apiserver 的实例信息并更新 endpoint。

## NodeLifecycleController
	定期监控node的状态并根据node的condition添加对应的taint标签或者直接驱逐node上的pod
#### taint
##### 使用效果
- `PreferNoSchedule`：调度器尽量避免把 pod 调度到具有该污点的节点上，如果不能避免(如其他节点资源不足)，pod 也能调度到这个污点节点上，已存在于此节点上的 pod 不会被驱逐；
- `NoSchedule`：不容忍该污点的 pod 不会被调度到该节点上，通过 kubelet 管理的 pod(static pod)不受限制，之前没有设置污点的 pod 如果已运行在此节点(有污点的节点)上，可以继续运行；
- `NoExecute`：不容忍该污点的 pod 不会被调度到该节点上，同时会将已调度到该节点上但不容忍 node 污点的 pod 驱逐掉；


> [!NOTE] 避免因网络等问题引起的pod驱逐行为
>  NodeLifecycleController 会为 node 进行分区并会为每个区设置不同的驱逐速率，即实际上会以 rate-limited 的方式添加 taint，在某些情况下可以避免 pod 被大量驱逐。

## garbage collector controller
#### 删除策略
- `Orphan` 策略：非级联删除，删除对象时，不会自动删除它的依赖或者是子对象，这些依赖被称作是原对象的孤儿对象，例如当执行以下命令时会使用 `Orphan` 策略进行删除，此时 ds 的依赖对象 `controllerrevision` 不会被删除

- `Background` 策略：在该模式下，kubernetes 会立即删除该对象，然后垃圾收集器会在后台删除这些该对象的依赖对象；

- `Foreground` 策略：在该模式下，对象首先进入“删除中”状态，即会设置对象的 `deletionTimestamp` 字段并且对象的 `metadata.finalizers` 字段包含了值 “foregroundDeletion”，此时该对象依然存在，然后垃圾收集器会删除该对象的所有依赖对象，垃圾收集器在删除了所有“Blocking” 状态的依赖对象（指其子对象中 `ownerReference.blockOwnerDeletion=true`的对象）之后，然后才会删除对象本身；

#### finalizer机制
	finalizer 是在删除对象时设置的一个 hook，其目的是为了让对象在删除前确认其子对象已经被完全删除，
	k8s 中默认有两种 finalizer：OrphanFinalizer 和 ForegroundFinalizer，finalizer 存在于对象的 ObjectMeta 中，当一个对象的依赖对象被删除后其对应的 finalizers 字段也会被移除，只有 finalizers 字段为空时，apiserver 才会删除该对象。