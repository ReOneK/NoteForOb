#### G-M-P分别代表：

- G - Goroutine，Go协程，是参与调度与执行的最小单位
- M - Machine，指的是系统级线程
- P - Processor，指的是逻辑处理器，P关联了的本地可运行G的队列(也称为LRQ)，最多可存放256个G

#### 调度流程

![[GMP调度流程.png]]

1. 每个P有个局部队列，`局部队列`保存待执行的goroutine(流程2)，
2. 当M绑定的P的的局部队列已经满了之后就会把goroutine放到`全局队列`(流程2-1)
3. `每个P和一个M绑定`，M是真正的执行P中goroutine的实体(流程3)，M从绑定的P中的局部队列获取G来执行
4. 当M绑定的P的局部队列为空时，M会从全局队列获取到本地队列来执行G(流程3.1)，
5. 当从全局队列中没有获取到可执行的G时候，M会从其他P的局部队列中偷取G来执行(流程3.2)，这种从其他P偷的方式称为`work stealing`
6. 当G因系统调用(syscall)阻塞时会阻塞M，此时P会和M解绑即`hand off`，并寻找新的idle的M，若没有idle的M就会新建一个M(流程5.1)。
7. 当G因channel或者network I/O阻塞时，不会阻塞M，M会寻找其他runnable的G；当阻塞的G恢复后会重新进入runnable进入P队列等待执行(流程5.3)

#### 调度过程中阻塞

- I/O，select
- block on syscall
- channel
- 等待锁
- runtime.Gosched()


> [!NOTE] 关于阻塞
> - 用户态阻塞不会导致M阻塞，仅阻塞G（阻塞则跳过，等就绪再加入）
> - 系统调用阻塞会导致M阻塞，此时M可被抢占（）
