### kubelet 熔断机制

##### 背景
跨大版本升级可能带来 Kubernetes API 版本的变化。如果 Kubelet 不能理解新的 API 版本，会导致其无法正确管理资源。升级过程中，Kubelet 可能无法识别 Pod 的 API 版本，从而导致意外行为如删除 Pod。

##### 解决方案
1. 通过ansible第三方渠道放置 `/etc/shein_kubelet_stop` 占位文件，最长10s周期被kubelet读取 （可以用ansible/puppet/saltstack/terraform等工具）
2. 所有pod停止sync，除了特定标志位的pod
3. 移除占位文件，即可恢复调度最长10s周期被kubelet读取
4. terminating的pod被本机制阻止