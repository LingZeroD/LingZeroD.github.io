# AQS探索

自上而下：从应用到原理

**自下而上：从原理到应用**

问题引入
--------------------

虽然说 Java 的底层已经在 unsafe 这个类中封装了 compareAndSwap 方法，支持了对 CAS 原语的调用。但是对于上层业务来说，怎么才能无感调用？在业务代码中相同步的往往是某项被竞争的资源，这种资源常常以对象的形式进行封装，而 CAS 只是能够原始的去修改内存中的一个值，那么如何用 CAS 去同步对象？

### 如何设计**同步管理框架**？

1、通用性，下层实现透明的同步机制，同时与上层业务解耦。

2、利用 CAS，原子的修改共享标记

3、等待队列

AQS 结构
------------------------

```java
private transient volatile Node head;


private transient volatile Node tail;




private volatile int state;




private transient Thread exclusiveOwnerThread;
```

怎么样，看样子应该是很简单的吧，毕竟也就四个属性啊。

> 独占模式：一旦被占用，其他线程都不能占用
> 
> 共享模式：一旦被占用，其他共享模式下的线程能占用

AbstractQueuedSynchronizer 的等待队列示意如下所示，注意了，之后分析过程中所说的 queue，也就是阻塞队列**不包含 head，不包含 head，不包含 head**。

![](https://www.javadoop.com/blogimages/AbstractQueuedSynchronizer/aqs-0.png)

等待队列中每个线程被包装成一个 Node 实例，数据结构是链表，一个 FIFO 双向链表

```java
static final class Node {
    
    static final Node SHARED = new Node();
    
    static final Node EXCLUSIVE = null;

    
    
    
    static final int CANCELLED =  1;
    
    
    static final int SIGNAL    = -1;
    
    
    static final int CONDITION = -2;
    



    
    static final int PROPAGATE = -3;
    
    
    
    
    volatile int waitStatus;
    
    volatile Node prev;
    
    volatile Node next;
    
    volatile Thread thread;
}
```

Node 的数据结构其实也挺简单的，就是 thread + waitStatus + pre + next 四个属性而已，大家先要有这个概念在心里。

![](http://www.darker-wxl.com/images/image-20220324094924874.png)

### tryAcquire 和 acquire

那么内部是怎么来写的呢

![](https://picture.lingzero.cn/202207071420148.png)

**很明显，AQS 中的 tryAcquire 是需要 override 由子类重写来实现的**

![](https://picture.lingzero.cn/202207071420713.png)

而 acquire 中不需要重写，即他很确定，通过 acquire 一定能获得锁

#### addWaiter 方法

那么我们看看 addWaiter 方法，很明显，就是将当前线程封装成一个 node，然后加入等待队列，返回值为当前的节点。

![](https://picture.lingzero.cn/202207071420621.png)

后面的 enq 是需要保证完整入队的，作者为了保证性能会先进行快速入队操作

![](https://picture.lingzero.cn/202207071420595.png)

#### [](#acquireQueued方法 "acquireQueued方法")acquireQueued 方法

failed 变量

![](https://picture.lingzero.cn/202207071420610.png)

那么考虑一下？为什么要将线程挂起，直接自旋，直到当前线程获得锁不好嘛？

> 自旋是一个 CPU 的操作，如果大量的线程在自选等待那么一定会出现性能问题，所以理想情况下，我们需要将那些还没有轮到他出队的那个线程挂起，再在适合的时间将他们唤醒，提升性能。

![](https://picture.lingzero.cn/202207071420718.png)

**执行挂起操作**

![](https://picture.lingzero.cn/202207071420035.png)

### [](#大致步骤 "大致步骤")大致步骤

![](https://picture.lingzero.cn/202207071420968.png)

### release 和 tryRelease

![](https://picture.lingzero.cn/202207071420704.png)

还是需要子类去实现 tryRelease 方法，如果没有实现，抛出异常

### unparkSucccessor 方法

![](https://picture.lingzero.cn/202207071420811.png)

从队列的**尾节点**开始搜索，找到除了 head 节点之外的最靠前的，且的节点，并进行唤醒该线程的操作

![](https://picture.lingzero.cn/202207071420283.png)

注意：

> 这里的中断，并不会像 wait，sleep 时中断一样会抛出异常，这里只会更改一个状态值。
> 
> 简单的来说，就是当线程处于等待队列中时，无法响应外部的中断请求，只有当这个线程拿到锁之后，然后在进行中断响应，一个细节

在来扒一扒 RTL
---------------------------------

![](https://picture.lingzero.cn/202207071420502.png)

看看源码可以直到，RTL 是继承了 Lock 的，那么意义是什么？

> Lock 的意义在于提供了区别于 Synchronized 的另一种具有更多广泛操作的同步方式，它能支持更多灵活的结构，并且可以关联多个 Condition 对象

Java 中段机制
---------------------------------

![](https://picture.lingzero.cn/202207071420970.png)

Lock 源码
---------------------------

![](https://picture.lingzero.cn/202207071420192.png)

1、lock：用于获取锁，如果当前锁被其他线程占用，即等待，直到获取位置

2、lockInterruptibly：和 lock 的区别在于，假如当前线程在等待锁的过程中被中断那么会退出等待并抛出，中断异常

3、trylock(无参)：尝试获取锁并立即返回

4、trylock(time)：假如期间被中断，抛出中断异常

5、unlock：释放锁

6、newCondition：新建一个绑定在当前 Lock 上的 Condition 对象 (等待状态)

RTL 源码
------------------------

核心：

![](https://picture.lingzero.cn/202207071420298.png)

**首先 Sync 继承了 AQS**，他有两个子类，即 FairSync 和 unFairSync

### nonfairTryAcquire

![](https://picture.lingzero.cn/202207071420207.png)

c 为 AQS 中的 state 状态

如果没被占用，直接用 CAS 原子更改 state，将当前线程置为独占线程

然后判断当前线程是否的独占线程，这里就是重入锁的一个实现了，我们想想，如果自己和自己抢自己的锁，那么自己应该怎么办？

> 可重入性：单个线程执行时，重新进入一个子程序仍然是线程安全的

这里就会用重入锁，state+1 表示重入锁的次数，将来释放也要释放这么多个锁

### tryRelease

![](https://picture.lingzero.cn/202207071420165.png)

注意，这里返回的是，**是否完全释放该锁**，而不是释放成功或失败，注意。

其他的方法都是一些简单的操作了

### NonfairSync

![](https://picture.lingzero.cn/202207071420549.png)

lock 中，上来就是一个 CAS，也不管其他的线程是否在等待，即非公平，但是只有一次机会，即这次 cas 没有获得锁，那就调用 Acquire 方法，即内部调用一次 tryAcquire，如果失败，则进入 FIFO 等待，即变成公平的。

tryAcquire 直接调用了父类的方法，在上面。。

### FairSync

![](https://picture.lingzero.cn/202207071420324.png)

如果锁空闲，而且 FIFO 里没有排在当前线程之前的线程，尝试获取锁，返回 true