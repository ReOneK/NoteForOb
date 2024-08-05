## pod创建的过程
### kubectl create

#### 客户端参数检查验证
	 镜像名称
	 镜像拉取策略校验
#### 对象生成
	获取pod默认生成器
	生成运行时对象
> [!NOTE] api groups和version 
> k8s的api带版本号并且被划分为不同的api groups
> k8s一般支持多版本的api groups,为了找到最合适的api,会调用apiserver进行获取
> 但是为了提高性能，一般会在本地~/.kube/cache/discovery目录缓存这些schema文件。

#### 按照正确的格式输出创建的对象
#### 客户端的认证支持
	-  -- kubeconfig

### apiserver
#### 认证
	apiserver在第一次启动的时候，会根据用户提供的命令行参数组装一个合适的认证器列表
	每一个请求都会逐一经过这个请求列表，直到有一个认证通过
- 客户端证书认证（x509）
- HTTP Basic（静态密码文件）
- Bearer Token（静态Token文件）
- webhook认证（使用外部 webhook 服务进行认证）

> [!NOTE] 
> 认证成功后，Authorization头信息将会从请求中移除，用户信息会添加到请求的上下文信息中。这样后续步骤就可以访问到认证阶段确定的请求用户的信息了。

#### 鉴权
	kube-apiserver需要基于用户提供的命令行参数，来组装一个合适的鉴权器列表来处理每一个请求。当所有的鉴权器都拒绝该请求时，请求会终止
	通过指定--authorization-mode参数设置授权机制，至少需要指定一个
- AlwaysAllow
- AlwaysDeny
- ABAC
- Webhook
- RBAC
- Node

#### Admission control
	持久化（存到etcd之前）的最后一道保障
- 变更准入控制器（Mutating Admission Controller）用于变更信息，能够修改用户提交的资源对象信息
- 验证准入控制器（Validating Admission Controller）用于身份验证，能够验证用户提交的资源对象信息

#### etcd
	kube-apiserver将反序列化HTTP请求（解码），构造运行时对象（runtime object），并将它持久化到etcd。

> [!NOTE] apiserver如何找到每一个资源对应的操作Handler
> 注册机制
>  1.API注册
> - 当 kube-apiserver 启动时，它会加载所有已注册的资源类型和它们的处理函数。这些资源和它们的处理逻辑通常在 Kubernetes 源代码中定义，并在编译时注册到 API Server 中。
> 2.REST处理器
> - 每个资源类型都有一个对应的 REST 处理器，该处理器用于处理 HTTP 请求。REST 处理器实现了资源的创建、读取、更新、删除等（CRUD）操作的具体逻辑。
> 
> 1.当apiserver启动时，会创建一个server chain,允许apiserver进行聚合
> 2.这个时候会创建一个默认的通用apiserver
> 3.同时默认生成的OpenAPI信息会填充到apiserver的配置中
> - 为每个api组配置一个存储服务提供器
> - 为每一个不同版本的API组添加REST路由映射信息

## pod删除的过程
1. **发出删除请求**：
    
    - 当用户使用 `kubectl delete pod` 命令删除一个 Pod 时，这个请求首先被发送到 Kubernetes API 服务器。
    - API 服务器会将这个删除请求记录下来，并将 Pod 的状态标记为 "Terminating"。
2. **通知 kubelet**：
    
    - API 服务器随后会通知运行该 Pod 的节点上的 kubelet，要求其执行 Pod 的删除操作。
    - 这个通知通常是通过 `gRPC` 或 HTTP 请求发送的。
3. **执行优雅终止**：
    
    - kubelet 接收到删除请求后，会向 Pod 中的容器发送一个 `SIGTERM` 信号，触发优雅终止过程。此时，应用程序有机会进行清理和保存数据。
    - kubelet 会等待一段时间（按照 `terminationGracePeriodSeconds` 设置）以给容器足够时间关闭。
4. **强制终止**：
    
    - 如果容器在优雅终止时间内没有退出，kubelet 将发送 `SIGKILL` 信号，以强制终止容器。
5. **反馈状态**：
    
    - 在容器终止后，kubelet 会将 Pod 的最终状态反馈给 API 服务器。在 API 服务器确认 Pod 已彻底删除后，会将其从集群状态中移除。  ^54a0c9

### 一个长期处于terminating的pod如何查找原因

#### 排查问题的方法
- kubectl describe pod <pod_name>
- kubectl logs pod-name -n namespace
- kubectl debug pod/pod-name -n namespace --image=busybox sh
- journalctl -u kubelet

#### 可能存在的原因
1. 挂载的持久卷资源未卸载
		- kubelet删除的时候会check
2. 优雅时间过长
		- 容器一个优雅的终止时间 terminationGracePeriodSeconds
		- 在此时间内，Pod 会一直处于 "Terminating" 状态
		- 容器可能需要较长时间来处理终止信号
			[[#容器可能需要长时间处理SIGTERM信号的原因]]
3. kubelet与apiserver通信问题
		[[#^54a0c9]]


#### 容器可能需要长时间处理SIGTERM信号的原因
1. **应用程序逻辑**：
    
    - 应用程序在收到 SIGTERM 信号后，需要进行较为复杂的数据清理或保存状态的操作。
    - 一些数据库或状态保存服务需要在关闭前完成事务清理或者数据同步。
2. **外部依赖**：
    
    - 容器可能依赖某些外部服务进行清理操作，这些服务的响应时间影响了容器的终止时间。
3. **资源密集型任务**：
    
    - 容器正在处理资源密集型任务（如大规模的文件处理或数据分析），这些任务需要时间完成