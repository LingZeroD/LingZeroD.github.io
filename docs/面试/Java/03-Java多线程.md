## 问题？

### Java 并发 - 知识体系

![java-concurrent-overview-1](https://picture.lingzero.cn/202207141535286.png)

### Java 并发 - 理论基础

####  多线程的出现是要解决什么问题的?

  CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

  - CPU 增加了缓存，以均衡与内存的速度差异；// 导致 `可见性`问题

  - 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；// 导致 `原子性`问题

  - 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。// 导致 `有序性`问题

#### 线程不安全是指什么? 举例说明

    如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

#### 并发出现线程不安全的本质什么? 

    可见性（CPU缓存引起），原子性（分时复用）和有序性（重排序）。

####  Java是怎么解决并发问题的? 

  3个关键字，JMM和8个Happens-Before

  - volatile、synchronized 和 final 三个关键字
  - Java 内存模型规范了 JVM 如何提供按需禁用缓存和编译优化的方法

####  线程安全是不是非真即假? 
  不是，一个类在可以被多个线程安全调用时就是线程安全的。线程安全不是一个非真即假的命题，可以将共享数据按照安全程度的强弱顺序分成以下五类: 不可变、绝对线程安全、相对线程安全、线程兼容和线程对立。

####  线程安全有哪些实现思路?

  1. 互斥同步

    synchronized 和 ReentrantLock。
  2. 非阻塞同步

    **CAS**、**AtomicInteger**，**ABA**

  3. 无同步方法

     栈封闭（多线程访问同一个方法的局部变量）

     线程本地存储（Thread Local Storage）

     可重入代码（Reentrant Code）

#### 如何理解并发和并行的区别?

  **并发**：两个及两个以上的作业在同一 **时间段** 内执行。

  **并行**：两个及两个以上的作业在同一 **时刻** 执行。

### Java 并发 - 线程基础

####  线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?

  ![Java 线程的状态 ](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

  ![img](https://picture.lingzero.cn/202207141449533.png)

####  通常线程有哪几种使用方式?

  - 实现 Runnable 接口；
  - 实现 Callable 接口；
  - 继承 Thread 类。

####  基础线程机制有哪些?

  1. Executor

     Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

     主要有三种 Executor:

     1. CachedThreadPool: 一个任务创建一个线程；
     2. FixedThreadPool: 所有任务只能使用固定大小的线程；
     3. SingleThreadExecutor: 相当于大小为 1 的 FixedThreadPool。

  2. Daemon

     守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

     当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

     main() 属于非守护线程。

     使用 setDaemon() 方法将一个线程设置为守护线程。

  3. Sleep()

  4. yield()

####  线程的中断方式有哪些?

  InterruptedException、 interrupted()、Executor 的中断操作（调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法）

####  线程的互斥同步方式有哪些? 如何比较和选择?

  第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

  **1. 锁的实现**

  synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

  **2. 性能**

  新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

  **3. 等待可中断**

  当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

  ReentrantLock 可中断，而 synchronized 不行。

  **4. 公平锁**

  公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

  synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

  **5. 锁绑定多个条件**

  一个 ReentrantLock 可以同时绑定多个 Condition 对象。

- 线程之间有哪些协作方式?

  join()、wait() notify() notifyAll()、await() signal() signalAll()

### Java并发 - Java中所有的锁

![img](https://www.pdai.tech/_images/thread/java-lock-1.png)


### 关键字: synchronized详解

####  Synchronized可以作用在哪里? 分别通过对象锁和类锁进行举例。

  对象锁：方法锁(默认锁对象为this,当前实例对象)和同步代码块锁(自己指定锁对象)

  类锁：指synchronize修饰静态的方法或指定锁对象为Class对象

####  Synchronized本质上是通过什么保证线程安全的? 分三个方面回答：加锁和释放锁的原理，可重入原理，保证可见性原理。

- 加锁和释放锁的原理

  ![执行 monitorenter 获取锁](https://picture.lingzero.cn/202207141519493.png)

  ![执行 monitorexit 释放锁](https://picture.lingzero.cn/202207141520190.png)

  

- 可重入原理

  **可重入锁**：又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。

  加锁次数计数器

- 保证可见行原理

  内存模型和happens-before规则

####  Synchronized由什么样的缺陷?  Java Lock是怎么弥补这些缺陷的。

JVM中monitorenter和monitorexit字节码依赖于底层的操作系统的Mutex Lock来实现的，但是由于使用Mutex Lock需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的；

- `锁粗化(Lock Coarsening)`：也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。

- `锁消除(Lock Elimination)`：通过运行时JIT编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本的Stack上进行对象空间的分配(同时还可以减少Heap上的垃圾收集开销)。

- `轻量级锁(Lightweight Locking)`：这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态(即单线程执行环境)，在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在monitorenter和monitorexit中只需要依靠一条CAS原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒(具体处理步骤下面详细讨论)。

- `偏向锁(Biased Locking)`：是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令，因为CAS原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本的延迟。

- `适应性自旋(Adaptive Spinning)`：当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁(mutex semaphore)前会进入忙等待(Spinning)然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore(即互斥锁)进入到阻塞状态。

####  Synchronized和Lock的对比，和选择?



####  Synchronized在使用时有何注意事项?

####  Synchronized修饰的方法在抛出异常时,会释放锁吗?

####  多个线程等待同一个snchronized锁的时候，JVM如何选择下一个获取锁的线程?

####  Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?

####  我想更加灵活地控制锁的释放和获取(现在释放锁和获取锁的时机都被规定死了)，怎么办?

####  什么是锁的升级和降级? 什么是JVM里的偏斜锁、轻量级锁、重量级锁?

####  不同的JDK中对Synchronized有何优化?

### 关键字: volatile详解

####  volatile关键字的作用是什么?
####  volatile能保证原子性吗?
####  之前32位机器上共享的long和double变量的为什么要用volatile? 现在64位机器上是否也要设置呢?
####  i++为什么不能保证原子性?
####  volatile是如何实现可见性的?  内存屏障。
####  volatile是如何实现有序性的?  happens#### before等
####  说下volatile的应用场景?

### 关键字: final详解

####  所有的final修饰的字段都是编译期常量吗?
####  如何理解private所修饰的方法是隐式的final?
####  说说final类型的类如何拓展? 比如String是final类型，我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?
####  final方法可以被重载吗? 可以
####  父类的final方法能不能够被子类重写? 不可以
####  说说final域重排序规则?
####  说说final的原理?
####  使用 final 的限制条件和局限性?
####  看本文最后的一个思考题

### JUC锁: LockSupport详解

####  为什么LockSupport也是核心基础类? AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)
####  写出分别通过wait/notify和LockSupport的park/unpark实现同步?
####  LockSupport.park()会释放锁资源吗? 那么Condition.await()呢?
####  Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park()的区别? 重点
####  如果在wait()之前执行了notify()会怎样?
####  如果在park()之前执行了unpark()会怎样?

### JUC锁: 锁核心类AQS详解

####  什么是AQS? 为什么它是核心?
####  AQS的核心思想是什么? 它是怎么实现的? 底层数据结构等
####  AQS有哪些核心的方法?
####  AQS定义什么样的资源获取方式? AQS定义了两种资源获取方式：`独占`(只有一个线程能访问执行，又根据是否按队列的顺序分为`公平锁`和`非公平锁`，如`ReentrantLock`) 和`共享`(多个线程可同时访问执行，如`Semaphore`、`CountDownLatch`、 `CyclicBarrier` )。`ReentrantReadWriteLock`可以看成是组合式，允许多个线程同时对某一资源进行读。
####  AQS底层使用了什么样的设计模式? 模板
####  AQS的应用示例?

### JUC锁: ReentrantLock详解

####  什么是可重入，什么是可重入锁? 它用来解决什么问题?
####  ReentrantLock的核心是AQS，那么它怎么来实现的，继承吗? 说说其类内部结构关系。
####  ReentrantLock是如何实现公平锁的?
####  ReentrantLock是如何实现非公平锁的?
####  ReentrantLock默认实现的是公平还是非公平锁?
####  使用ReentrantLock实现公平和非公平锁的示例?
####  ReentrantLock和Synchronized的对比?

### JUC锁: ReentrantReadWriteLock详解

####  为了有了ReentrantLock还需要ReentrantReadWriteLock?
####  ReentrantReadWriteLock底层实现原理?
####  ReentrantReadWriteLock底层读写状态如何设计的? 高16位为读锁，低16位为写锁
####  读锁和写锁的最大数量是多少?
####  本地线程计数器ThreadLocalHoldCounter是用来做什么的?
####  缓存计数器HoldCounter是用来做什么的?
####  写锁的获取与释放是怎么实现的?
####  读锁的获取与释放是怎么实现的?
####  RentrantReadWriteLock为什么不支持锁升级?
####  什么是锁的升降级? RentrantReadWriteLock为什么不支持锁升级?



### JUC集合: ConcurrentHashMap详解

####  为什么HashTable慢? 它的并发度是什么? 那么ConcurrentHashMap并发度是什么?
####  ConcurrentHashMap在JDK1.7和JDK1.8中实现有什么差别? JDK1.8解決了JDK1.7中什么问题
####  ConcurrentHashMap JDK1.7实现的原理是什么? 分段锁机制
####  ConcurrentHashMap JDK1.8实现的原理是什么? 数组+链表+红黑树，CAS
####  ConcurrentHashMap JDK1.7中Segment数(concurrencyLevel)默认值是多少? 为何一旦初始化就不可再扩容?
####  ConcurrentHashMap JDK1.7说说其put的机制?
####  ConcurrentHashMap JDK1.7是如何扩容的? rehash(注：segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry<K,V>[] 进行扩容)
####  ConcurrentHashMap JDK1.8是如何扩容的? tryPresize
####  ConcurrentHashMap JDK1.8链表转红黑树的时机是什么? 临界值为什么是8?
####  ConcurrentHashMap JDK1.8是如何进行数据迁移的? transfer

### JUC集合: CopyOnWriteArrayList详解

####  请先说说非并发集合中Fail#### fast机制?
####  再为什么说ArrayList查询快而增删慢?
####  对比ArrayList说说CopyOnWriteArrayList的增删改查实现原理? COW基于拷贝
####  再说下弱一致性的迭代器原理是怎么样的? `COWIterator<E>`
####  CopyOnWriteArrayList为什么并发安全且性能比Vector好?
####  CopyOnWriteArrayList有何缺陷，说说其应用场景?

### JUC集合: ConcurrentLinkedQueue详解

####  要想用线程安全的队列有哪些选择? Vector，`Collections.synchronizedList( List<T> list)`, ConcurrentLinkedQueue等
####  ConcurrentLinkedQueue实现的数据结构?
####  ConcurrentLinkedQueue底层原理?  全程无锁(CAS)
####  ConcurrentLinkedQueue的核心方法有哪些? offer()，poll()，peek()，isEmpty()等队列常用方法
####  说说ConcurrentLinkedQueue的HOPS(延迟更新的策略)的设计?
####  ConcurrentLinkedQueue适合什么样的使用场景?

### JUC集合: BlockingQueue详解

####  什么是BlockingDeque?
####  BlockingQueue大家族有哪些? ArrayBlockingQueue, DelayQueue, LinkedBlockingQueue, SynchronousQueue...
####  BlockingQueue适合用在什么样的场景?
####  BlockingQueue常用的方法?
####  BlockingQueue插入方法有哪些? 这些方法(`add(o)`,`offer(o)`,`put(o)`,`offer(o, timeout, timeunit)`)的区别是什么?
####  BlockingDeque 与BlockingQueue有何关系，请对比下它们的方法?
####  BlockingDeque适合用在什么样的场景?
####  BlockingDeque大家族有哪些?
####  BlockingDeque 与BlockingQueue实现例子?

### JUC线程池: FutureTask详解

####  FutureTask用来解决什么问题的? 为什么会出现?
####  FutureTask类结构关系怎么样的?
####  FutureTask的线程安全是由什么保证的?
####  FutureTask结果返回机制?
####  FutureTask内部运行状态的转变?
####  FutureTask通常会怎么用? 举例说明。

### JUC线程池: ThreadPoolExecutor详解

####  为什么要有线程池?
####  Java是实现和管理线程池有哪些方式?  请简单举例如何使用。
####  为什么很多公司不允许使用Executors去创建线程池? 那么推荐怎么使用呢?
####  ThreadPoolExecutor有哪些核心的配置参数? 请简要说明
####  ThreadPoolExecutor可以创建哪是哪三种线程池呢?
####  当队列满了并且worker的数量达到maxSize的时候，会怎么样?
####  说说ThreadPoolExecutor有哪些RejectedExecutionHandler策略? 默认是什么策略?
####  简要说下线程池的任务执行机制? execute –> addWorker –>runworker (getTask)
####  线程池中任务是如何提交的?
####  线程池中任务是如何关闭的?
####  在配置线程池的时候需要考虑哪些配置因素?
####  如何监控线程池的状态?

### JUC线程池: ScheduledThreadPool详解

####  ScheduledThreadPoolExecutor要解决什么样的问题?
####  ScheduledThreadPoolExecutor相比ThreadPoolExecutor有哪些特性?
####  ScheduledThreadPoolExecutor有什么样的数据结构，核心内部类和抽象类?
####  ScheduledThreadPoolExecutor有哪两个关闭策略? 区别是什么?
####  ScheduledThreadPoolExecutor中scheduleAtFixedRate 和 scheduleWithFixedDelay区别是什么?
####  为什么ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?
####  Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?

### JUC线程池: Fork/Join框架详解

####  Fork/Join主要用来解决什么样的问题?
####  Fork/Join框架是在哪个JDK版本中引入的?
####  Fork/Join框架主要包含哪三个模块? 模块之间的关系是怎么样的?
####  ForkJoinPool类继承关系?
####  ForkJoinTask抽象类继承关系? 在实际运用中，我们一般都会继承 RecursiveTask 、RecursiveAction 或 CountedCompleter 来实现我们的业务需求，而不会直接继承 ForkJoinTask 类。
####  整个Fork/Join 框架的执行流程/运行机制是怎么样的?
####  具体阐述Fork/Join的分治思想和work#### stealing 实现方式?
####  有哪些JDK源码中使用了Fork/Join思想?
####  如何使用Executors工具类创建ForkJoinPool?
####  写一个例子: 用ForkJoin方式实现1+2+3+...+100000?
####  Fork/Join在使用时有哪些注意事项? 结合JDK中的斐波那契数列实例具体说明。****

### JUC工具类: CountDownLatch详解

####  什么是CountDownLatch?
####  CountDownLatch底层实现原理?
####  CountDownLatch一次可以唤醒几个任务? 多个
####  CountDownLatch有哪些主要方法? await(),countDown()
####  CountDownLatch适用于什么场景?
####  写道题：实现一个容器，提供两个方法，add，size 写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束? 使用CountDownLatch 代替wait notify 好处。

### JUC工具类: CyclicBarrier详解

####  什么是CyclicBarrier?
####  CyclicBarrier底层实现原理?
####  CountDownLatch和CyclicBarrier对比?
####  CyclicBarrier的核心函数有哪些?
####  CyclicBarrier适用于什么场景?

### JUC工具类: Semaphore详解

####  什么是Semaphore?
####  Semaphore内部原理?
####  Semaphore常用方法有哪些? 如何实现线程同步和互斥的?
####  Semaphore适合用在什么场景?
####  单独使用Semaphore是不会使用到AQS的条件队列?
####  Semaphore中申请令牌(acquire)、释放令牌(release)的实现?
####  Semaphore初始化有10个令牌，11个线程同时各调用1次acquire方法，会发生什么?
####  Semaphore初始化有10个令牌，一个线程重复调用11次acquire方法，会发生什么?
####  Semaphore初始化有1个令牌，1个线程调用一次acquire方法，然后调用两次release方法，之后另外一个线程调用acquire(2)方法，此线程能够获取到足够的令牌并继续运行吗?
####  Semaphore初始化有2个令牌，一个线程调用1次release方法，然后一次性获取3个令牌，会获取到吗?

### JUC工具类: Phaser详解

####  Phaser主要用来解决什么问题?
####  Phaser与CyclicBarrier和CountDownLatch的区别是什么?
####  如果用CountDownLatch来实现Phaser的功能应该怎么实现?
####  Phaser运行机制是什么样的?
####  给一个Phaser使用的示例?

### JUC工具类: Exchanger详解

####  Exchanger主要解决什么问题?
####  对比SynchronousQueue，为什么说Exchanger可被视为 SynchronousQueue 的双向形式?
####  Exchanger在不同的JDK版本中实现有什么差别?
####  Exchanger实现机制?
####  Exchanger已经有了slot单节点，为什么会加入arena node数组? 什么时候会用到数组?
####  arena可以确保不同的slot在arena中是不会相冲突的，那么是怎么保证的呢?
####  什么是伪共享，Exchanger中如何体现的?
####  Exchanger实现举例

### Java 并发 ThreadLocal详解

####  什么是ThreadLocal? 用来解决什么问题的?
####  说说你对ThreadLocal的理解
####  ThreadLocal是如何实现线程隔离的?
####  为什么ThreadLocal会造成内存泄露? 如何解决
####  还有哪些使用ThreadLocal的应用场景?

## 基础

### **说说进程和线程的区别？**

进程是程序的一次执行，是系统进行资源分配和调度的独立单位，他的作用是是程序能够并发执行提高资源利用率和吞吐率。

由于进程是**资源分配和调度的基本单位**，因为进程的创建、销毁、切换产生大量的时间和空间的开销，进程的数量不能太多，而**线程是比进程更小的能独立运行的基本单位**，他是进程的一个实体，可以减少程序并发执行时的时间和空间开销，使得操作系统具有更好的并发性。

线程**基本不拥有系统资源**，只有一些运行时必不可少的资源，比如**程序计数器、寄存器和栈**，进程则**占有堆、栈**。

> **从 JVM 角度说进程和线程之间的关系**
>
> 下图是 Java 内存区域，通过下图我们从 JVM 的角度来说一下线程和进程之间的关系。
>
> ![img](https://picture.lingzero.cn/202207071323293.png)
>
> 从上图可以看出：一个进程中可以有多个线程，多个线程共享进程的**堆**和**方法区 (JDK1.8 之后的元空间)**资源，但是每个线程有自己的**程序计数器、虚拟机栈** 和 **本地方法栈**。
>
> **总结：** **线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反。**

扩展:

>### 程序计数器为什么是私有的?
>
>程序计数器主要有下面两个作用：
>
>1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
>2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。
>
>需要注意的是，如果执行的是 native 方法，那么程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址。
>
>所以，程序计数器私有主要是为了**线程切换后能恢复到正确的执行位置**。
>
>### 虚拟机栈和本地方法栈为什么是私有的)虚拟机栈和本地方法栈为什么是私有的?
>
>- **虚拟机栈：** 每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。
>- **本地方法栈：** 和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。
>
>所以，为了**保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的
>
>### 一句话简单了解堆和方法区
>
>堆和方法区是所有线程共享的资源，其中堆是进程中最大的一块内存，主要用于存放新创建的对象 (几乎所有对象都在这里分配内存)，方法区主要用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

### 说说并发与并行的区别?

- **并发**：两个及两个以上的作业在同一 **时间段** 内执行。
- **并行**：两个及两个以上的作业在同一 **时刻** 执行。

### 线程的生命周期?

Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态（图源《Java 并发编程艺术》4.1.4 节）。

![Java 线程的状态 ](https://picture.lingzero.cn/202207071341459.png)

线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。Java 线程状态变迁如下图所示（图源 [issue#736open in new window](https://github.com/Snailclimb/JavaGuide/issues/736)）：

![img](https://picture.lingzero.cn/202207071340809.png)

> 订正(来自[issue736open in new window](https://github.com/Snailclimb/JavaGuide/issues/736))：原图中 wait 到 runnable 状态的转换中，`join`实际上是`Thread`类的方法，但这里写成了`Object`。

由上图可以看出：线程创建之后它将处于 **NEW（新建）** 状态，调用 `start()` 方法后开始运行，线程这时候处于 **READY（可运行）** 状态。可运行状态的线程获得了 CPU 时间片（timeslice）后就处于 **RUNNING（运行）** 状态。

> 在操作系统中层面线程有 READY 和 RUNNING 状态，而在 JVM 层面只能看到 RUNNABLE 状态（图源：[HowToDoInJavaopen in new window](https://howtodoinjava.com/)：[Java Thread Life Cycle and Thread Statesopen in new window](https://howtodoinjava.com/Java/multi-threading/Java-thread-life-cycle-and-thread-states/)），所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。
>
> **为什么 JVM 没有区分这两种状态呢？** （摘自：[java线程运行怎么有第六种状态？ - Dawell的回答open in new window](https://www.zhihu.com/question/56494969/answer/154053599) ） 现在的**时分**（time-sharing）**多任务**（multi-task）操作系统架构通常都是用所谓的“**时间分片**（time quantum or time slice）”方式进行**抢占式**（preemptive）轮转调度（round-robin式）。这个时间分片通常是很小的，一个线程一次最多只能在 CPU 上运行比如 10-20ms 的时间（此时处于 running 状态），也即大概只有 0.01 秒这一量级，时间片用后就要被切换下来放入调度队列的末尾等待再次调度。（也即回到 ready 状态）。线程切换的如此之快，区分这两种状态就没什么意义了。

![RUNNABLE-VS-RUNNING](https://picture.lingzero.cn/202207071341954.png)

当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）** 状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而 **TIMED_WAITING(超时等待)** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将 Java 线程置于 TIMED_WAITING 状态。当超时时间到达后 Java 线程将会返回到 RUNNABLE 状态。当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到 **BLOCKED（阻塞）** 状态。线程在执行 Runnable 的`run()`方法之后将会进入到 **TERMINATED（终止）** 状态。

相关阅读：[挑错 |《Java 并发编程的艺术》中关于线程状态的三处错误open in new window](https://mp.weixin.qq.com/s/UOrXql_LhOD8dhTq_EPI0w) 

### 什么是上下文切换?

线程在执行过程中会有自己的运行条件和状态（也称上下文），比如上文所说到过的程序计数器，栈信息等。当出现如下情况的时候，线程会从占用 CPU 状态中退出。

- 主动让出 CPU，比如调用了 `sleep()`, `wait()` 等。
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用CPU导致其他线程或者进程饿死。
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。
- 被终止或结束运行

这其中前三种都会发生线程切换，线程切换意味着需要保存当前线程的上下文，留待线程下次占用 CPU 的时候恢复现场。并加载下一个将要占用 CPU 的线程上下文。这就是所谓的 **上下文切换**。

上下文切换是现代操作系统的基本功能，因其每次需要保存信息恢复信息，这将会占用 CPU，内存等系统资源进行处理，也就意味着效率会有一定损耗，如果频繁切换就会造成整体效率低下。

### 如何预防和避免线程死锁?

**如何预防死锁？** 破坏死锁的产生的必要条件即可：

1. **破坏请求与保持条件** ：一次性申请所有的资源。

2. **破坏不剥夺条件** ：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。

3. **破坏循环等待条件** ：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

**如何避免死锁？**

避免死锁就是在资源分配时，借助于算法（比如银行家算法）对资源分配进行计算评估，使其进入安全状态。

> **安全状态** 指的是系统能够按照某种线程推进顺序（P1、P2、P3.....Pn）来为每个线程分配所需资源，直到满足每个线程对资源的最大需求，使每个线程都可顺利完成。称<P1、P2、P3.....Pn>序列为安全序列。

### 说说 sleep() 方法和 wait() 方法区别和共同点?

- 两者最主要的区别在于：**`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- 两者都可以暂停线程的执行。
- `wait()` 通常被用于线程间交互/通信，`sleep() `通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify() `或者 `notifyAll()` 方法。`sleep() `方法执行完成后，线程会自动苏醒。或者可以使用 `wait(long timeout)` 超时后线程会自动苏醒。

### 为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？

这是另一个非常经典的 Java 多线程面试问题，而且在面试中会经常被问到。很简单，但是很多人都会答不上来！

new 一个 Thread，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**

## 进阶

### synchronized

#### **知道 synchronized 原理吗？**

synchronized 是 java 提供的原子性内置锁，这种内置的并且使用者看不到的锁也被称为**监视器锁**，使用 synchronized 之后，会在编译之后在同步的代码块前后加上 monitorenter 和 monitorexit 字节码指令，他依赖操作系统底层互斥锁实现。他的作用主要就是实现原子性操作和解决共享变量的内存可见性问题。

执行 monitorenter 指令时会尝试获取对象锁，如果对象没有被锁定或者已经获得了锁，锁的计数器 + 1。此时其他竞争锁的线程则会进入等待队列中。

执行 monitorexit 指令时则会把计数器 - 1，当计数器值为 0 时，则锁释放，处于等待队列中的线程再继续竞争锁。

synchronized 是排它锁，当一个线程获得锁之后，其他线程必须等待该线程释放锁后才能获得锁，而且由于 Java 中的线程和操作系统原生线程是一一对应的，线程被阻塞或者唤醒时时会从用户态切换到内核态，这种转换非常消耗性能。

从内存语义来说，加锁的过程会清除工作内存中的共享变量，再从主内存读取，而释放锁的过程则是将工作内存中的共享变量写回主内存。

_实际上大部分时候我认为说到 monitorenter 就行了，但是为了更清楚的描述，还是再具体一点_。

如果再深入到源码来说，synchronized 实际上有两个队列 waitSet 和 entryList。

1.  当多个线程进入同步代码块时，首先进入 entryList
2.  有一个线程获取到 monitor 锁后，就赋值给当前线程，并且计数器 + 1
3.  如果线程调用 wait 方法，将释放锁，当前线程置为 null，计数器 - 1，同时进入 waitSet 等待被唤醒，调用 notify 或者 notifyAll 之后又会进入 entryList 竞争锁
4.  如果线程执行完毕，同样释放锁，计数器 - 1，当前线程置为 null

![](https://picture.lingzero.cn/202207071311497.jpeg)

#### 锁的优化机制了解吗？

从 JDK1.6 版本之后，synchronized 本身也在不断优化锁的机制，有些情况下他并不会是一个很重量级的锁了。优化机制包括自适应锁、自旋锁、锁消除、锁粗化、轻量级锁和偏向锁。

锁的状态从低到高依次为**无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁**，升级的过程就是从低到高，降级在一定条件也是有可能发生的。

**自旋锁**：由于大部分时候，锁被占用的时间很短，共享变量的锁定时间也很短，所有没有必要挂起线程，用户态和内核态的来回上下文切换严重影响性能。自旋的概念就是让线程执行一个忙循环，可以理解为就是啥也不干，防止从用户态转入内核态，自旋锁可以通过设置 - XX:+UseSpining 来开启，自旋的默认次数是 10 次，可以使用 - XX:PreBlockSpin 设置。

**自适应锁**：自适应锁就是自适应的自旋锁，自旋的时间不是固定时间，而是由前一次在同一个锁上的自旋时间和锁的持有者状态来决定。

**锁消除**：锁消除指的是 JVM 检测到一些同步的代码块，完全不存在数据竞争的场景，也就是不需要加锁，就会进行锁消除。

**锁粗化**：锁粗化指的是有很多操作都是对同一个对象进行加锁，就会把锁的同步范围扩展到整个操作序列之外。

**偏向锁**：当线程访问同步块获取锁时，会在对象头和栈帧中的锁记录里存储偏向锁的线程 ID，之后这个线程再次进入同步块时都不需要 CAS 来加锁和解锁了，偏向锁会永远偏向第一个获得锁的线程，如果后续没有其他线程获得过这个锁，持有锁的线程就永远不需要进行同步，反之，当有其他线程竞争偏向锁时，持有偏向锁的线程就会释放偏向锁。可以用过设置 - XX:+UseBiasedLocking 开启偏向锁。

**轻量级锁**：JVM 的对象的对象头中包含有一些锁的标志位，代码进入同步块的时候，JVM 将会使用 CAS 方式来尝试获取锁，如果更新成功则会把对象头中的状态位标记为轻量级锁，如果更新失败，当前线程就尝试自旋来获得锁。

整个锁升级的过程非常复杂，我尽力去除一些无用的环节，简单来描述整个升级的机制。

简单点说，偏向锁就是通过对象头的偏向线程 ID 来对比，甚至都不需要 CAS 了，而轻量级锁主要就是通过 CAS 修改对象头锁记录和自旋来实现，重量级锁则是除了拥有锁的线程其他全部阻塞。

![](https://picture.lingzero.cn/202207071312721.jpeg)

#### 对象头具体都包含哪些内容？

在我们常用的 Hotspot 虚拟机中，对象在内存中布局实际包含 3 个部分：

1.  对象头
2.  实例数据
3.  对齐填充

而对象头包含两部分内容，Mark Word 中的内容会随着锁标志位而发生变化，所以只说存储结构就好了。

1.  对象自身运行时所需的数据，也被称为 Mark Word，也就是用于轻量级锁和偏向锁的关键点。具体的内容包含对象的 hashcode、分代年龄、轻量级锁指针、重量级锁指针、GC 标记、偏向锁线程 ID、偏向锁时间戳。
2.  存储类型指针，也就是指向类的元数据的指针，通过这个指针才能确定对象是属于哪个类的实例。

_如果是数组的话，则还包含了数组的长度_

![](https://picture.lingzero.cn/202207071312430.jpeg)

#### 对于加锁，那再说下 ReentrantLock 原理？他和 synchronized 有什么区别？

相比于 synchronized，ReentrantLock 需要显式的获取锁和释放锁，相对现在基本都是用 JDK7 和 JDK8 的版本，ReentrantLock 的效率和 synchronized 区别基本可以持平了。他们的主要区别有以下几点：

1.  等待可中断，当持有锁的线程长时间不释放锁的时候，等待中的线程可以选择放弃等待，转而处理其他的任务。
2.  公平锁：synchronized 和 ReentrantLock 默认都是非公平锁，但是 ReentrantLock 可以通过构造函数传参改变。只不过使用公平锁的话会导致性能急剧下降。
3.  绑定多个条件：ReentrantLock 可以同时绑定多个 Condition 条件对象。

ReentrantLock 基于 AQS(**AbstractQueuedSynchronizer 抽象队列同步器**) 实现。别说了，我知道问题了，AQS 原理我来讲。

AQS 内部维护一个 state 状态位，尝试加锁的时候通过 CAS(CompareAndSwap) 修改值，如果成功设置为 1，并且把当前线程 ID 赋值，则代表加锁成功，一旦获取到锁，其他的线程将会被阻塞进入阻塞队列自旋，获得锁的线程释放锁的时候将会唤醒阻塞队列中的线程，释放锁的时候则会把 state 重新置为 0，同时当前线程 ID 置为空。

![](https://picture.lingzero.cn/202207071312417.jpeg)

### CAS

#### **CAS 的原理呢？**

CAS 叫做 CompareAndSwap，比较并交换，主要是通过处理器的指令来保证操作的原子性，它包含三个操作数：

1.  变量内存地址，V 表示
2.  旧的预期值，A 表示
3.  准备设置的新值，B 表示

当执行 CAS 指令时，只有当 V 等于 A 时，才会用 B 去更新 V 的值，否则就不会执行更新操作。

#### **那么 CAS 有什么缺点吗？**

CAS 的缺点主要有 3 点：

**ABA 问题**：ABA 的问题指的是在 CAS 更新的过程中，当读取到的值是 A，然后准备赋值的时候仍然是 A，但是实际上有可能 A 的值被改成了 B，然后又被改回了 A，这个 CAS 更新的漏洞就叫做 ABA。只是 ABA 的问题大部分场景下都不影响并发的最终效果。

Java 中有 AtomicStampedReference 来解决这个问题，他加入了预期标志和更新后标志两个字段，更新时不光检查值，还要检查当前的标志是否等于预期标志，全部相等的话才会更新。

**循环时间长开销大**：自旋 CAS 的方式如果长时间不成功，会给 CPU 带来很大的开销。

**只能保证一个共享变量的原子操作**：只对一个共享变量操作可以保证原子性，但是多个则不行，多个可以通过 AtomicReference 来处理或者使用锁 synchronized 实现。

### volatile 

#### **volatile 原理知道吗？**

相比 synchronized 的加锁方式来解决共享变量的内存可见性问题，volatile 就是更轻量的选择，他没有上下文切换的额外开销成本。使用 volatile 声明的变量，可以确保值被更新的时候对其他线程立刻可见。volatile 使用内存屏障来保证不会发生指令重排，解决了内存可见性的问题。

我们知道，线程都是从主内存中读取共享变量到工作内存来操作，完成之后再把结果写回主内存，但是这样就会带来可见性问题。举个例子，假设现在我们是两级缓存的双核 CPU 架构，包含 L1、L2 两级缓存。

1.  线程 A 首先获取变量 X 的值，由于最初两级缓存都是空，所以直接从主内存中读取 X，假设 X 初始值为 0，线程 A 读取之后把 X 值都修改为 1，同时写回主内存。这时候缓存和主内存的情况如下图。

![](https://pica.zhimg.com/v2-5d13d1d458bcaa84af429a3fc7c9aef6_r.jpg?source=1940ef5c)

1.  线程 B 也同样读取变量 X 的值，由于 L2 缓存已经有缓存 X=1，所以直接从 L2 缓存读取，之后线程 B 把 X 修改为 2，同时写回 L2 和主内存。这时候的 X 值入下图所示。  
    那么线程 A 如果再想获取变量 X 的值，因为 L1 缓存已经有 x=1 了，所以这时候变量内存不可见问题就产生了，B 修改为 2 的值对 A 来说没有感知。  

![](https://picx.zhimg.com/v2-38e8b03e3ef612411a9b384bd32de0c1_r.jpg?source=1940ef5c)

那么，如果 X 变量用 volatile 修饰的话，当线程 A 再次读取变量 X 的话，CPU 就会根据缓存一致性协议强制线程 A 重新从主内存加载最新的值到自己的工作内存，而不是直接用缓存中的值。

再来说内存屏障的问题，volatile 修饰之后会加入不同的内存屏障来保证可见性的问题能正确执行。这里写的屏障基于书中提供的内容，但是实际上由于 CPU 架构不同，重排序的策略不同，提供的内存屏障也不一样，比如 x86 平台上，只有 StoreLoad 一种内存屏障。

1.  StoreStore 屏障，保证上面的普通写不和 volatile 写发生重排序
2.  StoreLoad 屏障，保证 volatile 写与后面可能的 volatile 读写不发生重排序
3.  LoadLoad 屏障，禁止 volatile 读与后面的普通读重排序
4.  LoadStore 屏障，禁止 volatile 读和后面的普通写重排序

![](https://picture.lingzero.cn/202207071316253.jpeg)

### ThreadLocal

#### **说说 ThreadLocal 原理？**

ThreadLocal 可以理解为线程本地变量，他会在每个线程都创建一个副本，那么在线程之间访问内部副本变量就行了，做到了线程之间互相隔离，相比于 synchronized 的做法是用空间来换时间。

ThreadLocal 有一个静态内部类 ThreadLocalMap，ThreadLocalMap 又包含了一个 Entry 数组，Entry 本身是一个弱引用，他的 key 是指向 ThreadLocal 的弱引用，Entry 具备了保存 key value 键值对的能力。

弱引用的目的是为了防止内存泄露，如果是强引用那么 ThreadLocal 对象除非线程结束否则始终无法被回收，弱引用则会在下一次 GC 的时候被回收。

但是这样还是会存在内存泄露的问题，假如 key 和 ThreadLocal 对象被回收之后，entry 中就存在 key 为 null，但是 value 有值的 entry 对象，但是永远没办法被访问到，同样除非线程结束运行。

但是只要 ThreadLocal 使用恰当，在使用完之后调用 remove 方法删除 Entry 对象，实际上是不会出现这个问题的。

![](https://picture.lingzero.cn/202207071354123.jpeg)

### 线程池

**使用线程池的好处**：

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

#### **线程池原理知道吗？**

首先线程池有几个核心的参数概念：

1.  最大线程数 maximumPoolSize
2.  核心线程数 corePoolSize
3.  活跃时间 keepAliveTime
4.  阻塞队列 workQueue
5.  拒绝策略 RejectedExecutionHandler

当提交一个新任务到线程池时，具体的执行流程如下：

1.  当我们提交任务，线程池会根据 corePoolSize 大小创建若干任务数量线程执行任务
2.  当任务的数量超过 corePoolSize 数量，后续的任务将会进入阻塞队列阻塞排队
3.  当阻塞队列也满了之后，那么将会继续创建 (maximumPoolSize-corePoolSize) 个数量的线程来执行任务，如果任务处理完成，maximumPoolSize-corePoolSize 额外创建的线程等待 keepAliveTime 之后被自动销毁
4.  如果达到 maximumPoolSize，阻塞队列还是满的状态，那么将根据不同的拒绝策略对应处理

![](https://picture.lingzero.cn/202207071354499.jpeg)

#### **拒绝策略有哪些？**

主要有 4 种拒绝策略：

1.  AbortPolicy：直接丢弃任务，抛出异常，这是默认策略
2.  CallerRunsPolicy：只用调用者所在的线程来处理任务
3.  DiscardOldestPolicy：丢弃等待队列中最近的任务，并执行当前任务
4.  DiscardPolicy：直接丢弃任务，也不抛出异常

###  Atomic 原子类

#### 介绍一下 Atomic 

`Atomic` 翻译成中文是原子的意思。在化学上，我们知道原子是构成一般物质的最小单位，在化学反应中是不可分割的。在我们这里 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

所以，所谓原子类说简单点就是具有原子/原子操作特征的类。

并发包 `java.util.concurrent` 的原子类都存放在`java.util.concurrent.atomic`下,如下图所示。

![JUC原子类概览](https://picture.lingzero.cn/202207071410157.png)

#### JUC 包中的原子类是哪 4 类?

**基本类型**

使用原子的方式更新基本类型

- `AtomicInteger`：整型原子类
- `AtomicLong`：长整型原子类
- `AtomicBoolean`：布尔型原子类

**数组类型**

使用原子的方式更新数组里的某个元素

- `AtomicIntegerArray`：整型数组原子类
- `AtomicLongArray`：长整型数组原子类
- `AtomicReferenceArray`：引用类型数组原子类

**引用类型**

- `AtomicReference`：引用类型原子类
- `AtomicStampedReference`：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
- `AtomicMarkableReference` ：原子更新带有标记位的引用类型

**对象的属性修改类型**

- `AtomicIntegerFieldUpdater`：原子更新整型字段的更新器
- `AtomicLongFieldUpdater`：原子更新长整型字段的更新器
- `AtomicReferenceFieldUpdater`：原子更新引用类型字段的更新器

#### 简单介绍一下 AtomicInteger 类的原理

AtomicInteger 线程安全原理简单分析

AtomicInteger 类的部分源码：

```java
// setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

CAS 的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址，返回值是 valueOffset。另外 value 是一个 volatile 变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。

关于 Atomic 原子类这部分更多内容可以查看我的这篇文章：并发编程面试必备：[JUC 中的 Atomic 原子类总结](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484834&idx=1&sn=7d3835091af8125c13fc6db765f4c5bd&source=41#wechat_redirect)

### AQS





### 参考:

- 知乎: https://www.zhihu.com/question/443280657/answer/1785592611
- JavaGuide:https://javaguide.cn/