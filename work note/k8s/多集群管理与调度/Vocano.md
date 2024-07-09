#### 要解决什么样的问题

1. 分组调用（kube-batch）
    
    1. 分组调度基于容器组（jobs）,算法检查每个job并判断是否可以调度整个容器组，每个组中的容器称为task（任务）
        
2. 资源自动优化配置
    
3. 支持一系列高级调度场景
    
    1. 域资源公平性（DRF）：对需要资源较少的作业进行优先排序，从而执行更多的作业（执行更多的任务）
        
    2. binpack：确保任何被占有的节点都被尽可能的完全占用（节点尽可能的占满）
        
    3. 队列比例算法：对集群的整体资源进行分配，对预期资源利用率最低的队列进行优先排序
        

#### 作业提交流程

Informer--->list watch--->Cache--- resources snapshot --->Session--- open ssn ---> Plugin ---> bind

1. 通过list&&watch对资源cache
    
2. 调度不是监听到事件就立马执行，而是周期性的(open ssn)
    
3. 默认1s执行一次调度，期间会运行plugin插件，最终绑定到node
    

#### 相关资源定义

- Session ：对应于k8s的Framework
    
- Action ：相当于preFilter、Fileter、PostFilter、PreBind、Bind、PostBind等这些动作。
    
    - Enqueue：入队判断资源是否合适
        
    - Allocate：执行调度操作
        
    - Preempt：抢占资源
        
    - Backfill：回填操作，处理未指明资源使用量的pod调度
        
    - Reclaim：根据队列权重回收队列中的资源
        
    - Shuffle：根据资源状况重新分配节点
        
- VolcanoJob: 自定义的cr，主要用于创建podGroup资源,并被Volcano Schedule调度
    
- PodGroup: 一组强关联的Pod集合
    
- Queue: 一个PodGroup的队列，用来实现PodGroup的排队以及优先级的控制