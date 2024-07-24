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
	- 在NewMainKubelet

5. 监控以及验证