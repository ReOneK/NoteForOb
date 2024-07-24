### kubelet 熔断机制

##### 背景
异常场景

##### 解决方案
1. 通过ansible第三方渠道放置 `/etc/shein_kubelet_stop` 占位文件，最长10s周期被kubelet读取 （可以用ansible/puppet/saltstack/terraform等工具）
2. 所有pod停止sync，除了特定标志位的pod
3. 移除占位文件，即可恢复调度最长10s周期被kubelet读取
4. terminating的pod被本机制阻止