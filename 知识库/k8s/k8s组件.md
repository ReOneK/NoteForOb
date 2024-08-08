## apiserver

#### 基础流程
1. [[pod的生命周期管理#apiserver|基础概念]]  

#### default ns下面内置的kubernetes svc的作用
	1. 是由 kube-apiserver 中的 bootstrap controller 进行控制的
	2. 代表 Kubernetes API Server

	bootstrap controller的主要作用是初始化

