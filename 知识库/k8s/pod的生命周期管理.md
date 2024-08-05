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
- webhook认证（使用外部 Webhook 服务进行认证）

> [!NOTE] 
> 认证成功后，Authorization头信息将会从请求中移除，用户信息会添加到请求的上下文信息中。这样后续步骤就可以访问到认证阶段确定的请求用户的信息了。

#### 鉴权

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