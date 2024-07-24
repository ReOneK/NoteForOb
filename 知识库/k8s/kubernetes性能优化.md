### kubelet 熔断机制

##### 背景
异常场景下（集群升级失败导致api版本的不兼容）控制面和数据面资源版本不一致，导致kubelet会大量terminating pod

##### 解决方案
1. 通过ansible第三方渠道放置 `/etc/shein_kubelet_stop` 占位文件，最长10s周期被kubelet读取 （可以用ansible/puppet/saltstack/terraform等工具）
2. 所有pod停止sync，除了特定标志位的pod
3. 移除占位文件，即可恢复调度最长10s周期被kubelet读取
4. terminating的pod被本机制阻止