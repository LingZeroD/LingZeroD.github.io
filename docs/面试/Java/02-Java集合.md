### Arraylist 与 LinkedList 区别?

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）
3. **插入和删除是否受元素位置的影响：** ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**
4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
5. **内存空间占用：** `ArrayList` 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 `LinkedList` 的空间花费则体现在它的每一个元素都需要消耗比 `ArrayList` 更多的空间（因为要存放直接后继和直接前驱以及数据）。

### CopyOnWriteArrayList 

线程安全的list,读不加锁,写通过加锁+copy+volatile关键字(修饰Object[])保证线程安全

读源码

```java
 /** The array, accessed only via getArray/setArray. */
    private transient volatile Object[] array;
    public E get(int index) {
        return get(getArray(), index);
    }
    @SuppressWarnings("unchecked")
    private E get(Object[] a, int index) {
        return (E) a[index];
    }
    final Object[] getArray() {
        return array;
    }
```

写源码

```java
/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();//加锁
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);//拷贝新数组
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();//释放锁
        }
    }
```

### 说说 HashMap 原理吧？

HashMap 主要由数组和链表组成，他不是线程安全的。核心的点就是 put 插入数据的过程，get 查询数据以及扩容的方式。JDK1.7 和 1.8 的主要区别在于头插和尾插方式的修改，头插容易导致 HashMap 链表死循环，并且 1.8 之后加入红黑树对性能有提升。

**put 插入数据流程**

往 map 插入元素的时候首先通过对 key hash 然后与数组长度 - 1 进行与运算 ((n-1)&hash)，都是 2 的次幂所以等同于取模，但是位运算的效率更高。找到数组中的位置之后，如果数组中没有元素直接存入，反之则判断 key 是否相同，key 相同就覆盖，否则就会插入到链表的尾部，如果链表的长度超过 8，则会转换成红黑树，最后判断数组长度是否超过默认的长度 * 负载因子也就是 12，超过则进行扩容。

![](https://picture.lingzero.cn/202207071315490.jpeg)

**get 查询数据**

查询数据相对来说就比较简单了，首先计算出 hash 值，然后去数组查询，是红黑树就去红黑树查，链表就遍历链表查询就可以了。

**resize 扩容过程**

扩容的过程就是对 key 重新计算 hash，然后把数据拷贝到新的数组。

### HashMap为什么线程不安全?

1、HashMap线程不安全原因：

原因：

- **JDK1.7 中，由于多线程对HashMap进行扩容，调用了HashMap#transfer()，具体原因：某个线程执行过程中，被挂起，其他线程已经完成数据迁移，等CPU资源释放后被挂起的线程重新执行之前的逻辑，数据已经被改变，造成死循环、数据丢失。**
- **JDK1.8 中，由于多线程对HashMap进行put操作，调用了HashMap#putVal()，具体原因：假设两个线程A、B都在进行put操作，并且hash函数计算出的插入下标是相同的，当线程A执行完第六行代码后由于时间片耗尽导致被挂起，而线程B得到时间片后在该下标处插入了元素，完成了正常的插入，然后线程A获得时间片，由于之前已经进行了hash碰撞的判断，所有此时不会再进行判断，而是直接进行插入，这就导致了线程B插入的数据被线程A覆盖了，从而线程不安全。**

改善：

- 数据丢失、死循环已经在在JDK1.8中已经得到了很好的解决，如果你去阅读1.8的源码会发现找不到HashMap#transfer()，因为JDK1.8直接在HashMap#resize()中完成了数据迁移。

2、HashMap线程不安全的体现：

- JDK1.7 HashMap线程不安全体现在：死循环、数据丢失
- JDK1.8 HashMap线程不安全体现在：数据覆盖

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

>ConcurrentHashMap 在 JDK7 和 JDK8的区别？
>（1）数据结构：JDK7 的数据结构是 Segment数组 + HashEntry数组 + 链表，JDK8 的数据结构是 HashEntry数组 + 链表 + 红黑树，当链表的长度超过8时，链表就会转换成红黑树，从而降低时间复杂度（由O(n) 变成了 O(logN)），提高了效率
>
>（2）锁的实现：JDK7的锁是segment，是基于ReentronLock实现的，包含多个HashEntry；而JDK8 降低了锁的粒度，采用 table 数组元素作为锁，从而实现对每行数据进行加锁，进一步减少并发冲突的概率，并使用 synchronized 来代替 ReentrantLock，因为在低粒度的加锁方式中，synchronized 并不比 ReentrantLock 差，在粗粒度加锁中ReentrantLock 可以通过 Condition 来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了。
>
>（3）统计集合中元素个数 size 的方式：JDK7 是先尝试 2次通过不锁住 segment 的方式来统计各个 segment 大小，如果统计的过程中，容器的 count 发生了变化，则再采用加锁的方式来统计所有Segment的大小；在 JDK8 中，对于size的计算，在扩容和 addCount() 方法中就已经有处理了，等到调用 size() 时直接返回元素的个数
>

### 参考:

- 知乎: https://www.zhihu.com/question/443280657/answer/1785592611
- 