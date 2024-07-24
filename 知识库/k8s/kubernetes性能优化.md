### kubelet 熔断机制

##### 背景
	跨大版本升级可能带来 Kubernetes API 版本的变化，如果控制平面（API Server 和 Controller Manager等）已经升级到新的版本，而 Kubelet 尚未完成升级，Kubelet 可能会因为不兼容的 API 版本无法正确同步资源列表，可能会表现为删除旧的pod，或者无法调度/管理新的pod

##### 解决方案
1. 逐步升级
2. 升级过程的协调
		确保 API Server 和 Kubelet 的版本兼容，先升级控制平面组件，然后逐步升级节点上的 Kubelet，kube-proxy
3. 使用熔断机制
	1. 基于监控告警触发
	2. 通过ansible放置/etc/shein_kubelet_stop的占位文件
	3. 在每个节点上运行定时脚本去检查是否存在占位文件
	4. 如果存在则通过 `kubectl get pods --all-namespaces -o json | jq -r 
	5. 停止所有调度

	- 另外一个方案是直接修改kubelet的代码
	- 在NewMainKubelet这边添加一个go func去check 占位文件
	- 如果存在则将标识设立为true，并在killpod这个函数的时候去进行判断

5. 监控以及验证

### 大规模apiserver优化

##### 背景
1. 内存消耗来源
	1. apiserver在watchCache中缓存了集群所有云数据，并且为每种资源缓存了历史watchCacheEvent
	2. 一些controller在没有指定resourceVersion的情况发起list请求，会导致apiServer在内存中进行大量的数据深拷贝，这些深拷贝的数据无法被直接GC