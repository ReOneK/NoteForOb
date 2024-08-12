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
	