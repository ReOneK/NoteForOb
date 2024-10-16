### 主要干了哪些事情
1. 提供Kubernetes API,包括认证授权，数据校验以及集群状态变更
2. 代理集群中的一些附加组件，如kubernetes UI,metrics-server,npd（node problem detect?）
3. 创建k8s服务，apiserver service以及kubernetes service
4. 资源在不同版本之间的转换

### 一次请求的具体流程
#### 访问接口
curl -k https://<masterIP>:6443

#### 1.Decode
	
#### 2.认证
	apiserver在第一次启动的时候，会根据用户提供的命令行参数组装一个合适的认证器列表
	每一个请求都会逐一经过这个请求列表，直到有一个认证通过
	- 客户端证书认证（x509）
	- HTTP Basic（静态密码文件）
	- Bearer Token（静态Token文件）
	- webhook认证（使用外部 webhook 服务进行认证）

> [!NOTE] 
> 认证成功后，Authorization头信息将会从请求中移除，用户信息会添加到请求的上下文信息中。这样后续步骤就可以访问到认证阶段确定的请求用户的信息了。

#### 3.鉴权
	kube-apiserver需要基于用户提供的命令行参数，来组装一个合适的鉴权器列表来处理每一个请求。当所有的鉴权器都拒绝该请求时，请求会终止
	通过指定--authorization-mode参数设置授权机制，至少需要指定一个
	- AlwaysAllow
	- AlwaysDeny
	- ABAC
	- Webhook
	- RBAC
	- Node

#### 4.Admission control
	持久化（存到etcd之前）的最后一道保障
	- 变更准入控制器（Mutating Admission Controller）用于变更信息，能够修改用户提交的资源对象信息
	- 验证准入控制器（Validating Admission Controller）用于身份验证，能够验证用户提交的资源对象信息

#### 5.etcd
	kube-apiserver将反序列化HTTP请求（解码），构造运行时对象（runtime object），并将它持久化到etcd。

> [!NOTE] apiserver如何找到每一个资源对应的操作Handler
>  
> - 1.API注册。当 kube-apiserver 启动时，它会加载所有已注册的资源类型和它们的处理函数。这些资源和它们的处理逻辑通常在 Kubernetes 源代码中定义，并在编译时注册到 API Server 中。
> 2.REST处理器
> - 每个资源类型都有一个对应的 REST 处理器，该处理器用于处理 HTTP 请求。REST 处理器实现了资源的创建、读取、更新、删除等（CRUD）操作的具体逻辑。


### 

