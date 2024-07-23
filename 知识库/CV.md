ssk集群管控平台

1. 不同公有云资源的管控
    
2. 中间件服务的分发部署
    

在此基础之上，基于karmada构建了一个联邦调度平台,应用的分发部署

1. 根据用户的服务画像（cpu，内存）和时间维度进行相关性分析，从而进行调度和部署优化，应用错峰
    
2. 精细化资源配置（任务优先级，cpu set化）
    
3. 亲和性和互斥性
    

### 自建机房cni以及kubernetes的优化和改造

#### kube-schedule

预选过程中的局部最优解

减少预选过程中参与的节点数量：percentageOfNodesToScore

#### kubelet改造

1. 收敛kubelet的自决策能力：
    
    1. pod的 QoS 等级
        
    2. PDB的设置
        
2. 增强容器，支持业务给自己的容器扩展io limit,swapniss,pid limit等参数

kubernetes优化