### 说说 HashMap 原理吧？

HashMap 主要由数组和链表组成，他不是线程安全的。核心的点就是 put 插入数据的过程，get 查询数据以及扩容的方式。JDK1.7 和 1.8 的主要区别在于头插和尾插方式的修改，头插容易导致 HashMap 链表死循环，并且 1.8 之后加入红黑树对性能有提升。

**put 插入数据流程**

往 map 插入元素的时候首先通过对 key hash 然后与数组长度 - 1 进行与运算 ((n-1)&hash)，都是 2 的次幂所以等同于取模，但是位运算的效率更高。找到数组中的位置之后，如果数组中没有元素直接存入，反之则判断 key 是否相同，key 相同就覆盖，否则就会插入到链表的尾部，如果链表的长度超过 8，则会转换成红黑树，最后判断数组长度是否超过默认的长度 * 负载因子也就是 12，超过则进行扩容。

![](https://picture.lingzero.cn/202207071315490.jpeg)

**get 查询数据**

查询数据相对来说就比较简单了，首先计算出 hash 值，然后去数组查询，是红黑树就去红黑树查，链表就遍历链表查询就可以了。

**resize 扩容过程**

扩容的过程就是对 key 重新计算 hash，然后把数据拷贝到新的数组。

### 多线程环境怎么使用 Map 呢？ConcurrentHashmap 了解过吗？

多线程环境可以使用 Collections.synchronizedMap 同步加锁的方式，还可以使用 HashTable，但是同步的方式显然性能不达标，而 ConurrentHashMap 更适合高并发场景使用。

ConcurrentHashmap 在 JDK1.7 和 1.8 的版本改动比较大，1.7 使用` Segment+HashEntry `分段锁的方式实现，1.8 则抛弃了 Segment，改为使用 `CAS+synchronized+Node `实现，同样也加入了红黑树，避免链表过长导致性能的问题。

**1.7 分段锁**

从结构上说，1.7 版本的 ConcurrentHashMap 采用分段锁机制，里面包含一个 Segment 数组，Segment 继承于 ReentrantLock，Segment 则包含 HashEntry 的数组，HashEntry 本身就是一个链表的结构，具有保存 key、value 的能力能指向下一个节点的指针。

实际上就是相当于每个 Segment 都是一个 HashMap，默认的 Segment 长度是 16，也就是支持 16 个线程的并发写，Segment 之间相互不会受到影响。

![](https://picture.lingzero.cn/202207071315081.jpeg)

**put 流程**

其实发现整个流程和 HashMap 非常类似，只不过是先定位到具体的 Segment，然后通过 ReentrantLock 去操作而已，后面的流程我就简化了，因为和 HashMap 基本上是一样的。

1.  计算 hash，定位到 segment，segment 如果是空就先初始化
2.  使用 ReentrantLock 加锁，如果获取锁失败则尝试自旋，自旋超过次数就阻塞获取，保证一定获取锁成功
3.  遍历 HashEntry，就是和 HashMap 一样，数组中 key 和 hash 一样就直接替换，不存在就再插入链表，链表同样

![](https://picture.lingzero.cn/202207071315528.jpeg)

**get 流程**

get 也很简单，key 通过 hash 定位到 segment，再遍历链表定位到具体的元素上，需要注意的是 value 是 volatile 的，所以 get 是不需要加锁的。

**1.8CAS+synchronized**

1.8 抛弃分段锁，转为用 CAS+synchronized 来实现，同样 HashEntry 改为 Node，也加入了红黑树的实现。主要还是看 put 的流程。

![](https://picture.lingzero.cn/202207071315397.jpeg)

**put 流程**

1.  首先计算 hash，遍历 node 数组，如果 node 是空的话，就通过 CAS + 自旋的方式初始化
2.  如果当前数组位置是空则直接通过 CAS 自旋写入数据
3.  如果 hash==MOVED，说明需要扩容，执行扩容
4.  如果都不满足，就使用 synchronized 写入数据，写入数据同样判断链表、红黑树，链表写入和 HashMap 的方式一样，key hash 一样就覆盖，反之就尾插法，链表长度超过 8 就转换成红黑树

![](https://picture.lingzero.cn/202207071315144.jpeg)

**get 查询**

get 很简单，通过 key 计算 hash，如果 key hash 相同就返回，如果是红黑树按照红黑树获取，都不是就遍历链表获取。

### 参考:

- 知乎: https://www.zhihu.com/question/443280657/answer/1785592611
- 