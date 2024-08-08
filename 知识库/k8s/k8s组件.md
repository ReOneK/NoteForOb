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

