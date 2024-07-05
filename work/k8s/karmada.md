
#### 为什么选择karmada，能够带来什么价值

1. 统一多集群管理，跨集群的资源部署，管理和同步
    
2. 面向开源社区
    
3. 提高资源利用率，根据负载和资源成本动态调整应用部署位置
    
4. 提升应用可靠性，跨多个集群部署实现高可用性和灾备
    
5. 灵活的策略驱动调度
    
    1. 集群亲和性/反亲和行
        
    2. 地域分布
        
    3. 资源配额/成本控制
        
    4. 负载均衡
        

#### karmada和其它调度框架对比，有什么优势和劣势

1. Openshift : 内置ci/cd流水线，直接从源码构建容器镜像，商业化
    
2. Google Anthos ： 跨云平台，集成服务网格，声明式配置模型
    
3. Rancher : 从裸机到k8s集群的完整管理能力


#### karmada需要解决的问题

1. 生产级别的灾备方案
    
2. namespace资源对象创建的时候无法指定集群（现在状况是会同步到所有集群创建相同的namespace）
    
3. cni的网络感知（调度器无法感知固定ip的cni）
    
4. 对有状态对象的状态收集
    

#### Karmada failover

1. clusterStatus controller:同步集群状态
    
2. cluster controller : 根据集群当前状态下的conditions判断是否需要为集群打上不可调度和不可执行的污点taint
    
3. taint-manager controller : 感知集群拥有effect:NoExecute,获取该集群上所有的resourceBinding资源，然后放入对应的驱逐处理队列，驱逐消费者worker会判断resourceBingding和对应的pp(propagationPolicy)是否存在污点容忍，容忍则跳过，否则开启优雅驱逐，即 rb.spec.gracefulEvictionTasks 中增加一条优雅驱逐任务。
    
4. Karmada schedule: 在taint-manager controller 中由于集群故障会将资源从故障集群上驱逐了，即移除了rb.spec.clusters中的故障集群，导致 rb.spec.clusters 下所有的副本之和小于 rb.spec.replicas，所以这是一次扩容操作。
    
5. gracefulEviction controller: 新的副本在符合要求的集群上 Ready 后， 再移除优雅驱逐任务，也就是才能删除调度到故障集群的旧资源
    

#### 基于karmada的跨集群管控平台

1. **统一管理**：实现对多个Kubernetes集群的统一视图和管理界面，简化跨集群管理流程。
    
2. **资源优化**：在多个集群间自动调度工作负载，优化资源分配，实现成本效益最大化。
    
3. **可靠性提升**：通过跨集群备份和容灾，提高系统的总体可靠性和可用性。