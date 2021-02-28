---
title: 阻塞队列
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 14.1 BlockingQueue

阻塞队列含义：
- 是一个先进先出的队列
- 获取队列元素但是队列为空，会阻塞直到队列有元素时返回
- 添加元素时，如果队列已满，那么等到队列可以放新元素时再放入。

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/2B03B09825504490A52F0F66BD6F0FF6/11439)
- 上表中put和take是带有阻塞的
- BlockingQueue不接受null值得插入，相应的方法在碰到null的插入时会抛出NullPointerException异常。null值在这里通常用于作为特殊值返回，代表poll失败。
- BlockingQueue是设计用来实现生产者-消费者队列的
- BlockingQueue的实现都是线程安全的，但是批量的集合操作如`addALL`,`containsAll`,`retainAll`,`removeAll`和`removeAll`不一定是原子操作。如 addAll(c) 有可能在添加了一些元素后中途抛出异常，此时 BlockingQueue 中已经添加了部分元素，这个是允许的，取决于具体的实现。
- BlockingQueue不支持close和shutdown等关闭操作，因为开发者可能希望不会有新的元素添加进去。
- BlockingQueue 在生产者-消费者的场景中，是支持多消费者和多生产者的，说的其实就是线程安全问题。

阻塞队列的由来：

- 为了消除生产者类与消费者类之间的代码依赖性
- 将生产数据的过程与使用数据的过程解耦简化负载
- 自己实现多个线程操作共享变量，容易引发重复消费和死锁等问题。

# 14.2 BlockingQueue实现之ArrayBlockingQueue

ArrayBlockingQueue是BlockingQueue接口的有界队列实现类，底层采用数组来实现。

其并发控制可以采用可重入锁来控制，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。

ArrayBlockingQueue 共有以下几个属性：
```
// 用于存放元素的数组
final Object[] items;
// 下一次读取操作的位置
int takeIndex;
// 下一次写入操作的位置
int putIndex;
// 队列中的元素数量
int count;

// 以下几个就是控制并发用的同步器
final ReentrantLock lock;
private final Condition notEmpty;
private final Condition notFull;
```

ArrayBlockingQueue 实现并发同步的原理就是，读操作和写操作都需要获取到 AQS 独占锁才能进行操作。如果队列为空，这个时候读操作的线程进入到读线程队列排队，等待写线程写入新的元素，然后唤醒读线程队列的第一个等待线程。如果队列已满，这个时候写操作的线程进入到写线程队列排队，等待读线程将队列元素移除腾出空间，然后唤醒写线程队列的第一个等待线程。

# 14.3 BlockingQueue 实现之 LinkedBlockingQueue

- 底层基于单向链表的阻塞队列，可以当做无界队列也可以当做有界队列使用。
```
// 传说中的无界队列
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}

// 传说中的有界队列
public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
    last = head = new Node<E>(null);
}
```
- 类内属性
```
// 队列容量
private final int capacity;

// 队列中的元素数量
private final AtomicInteger count = new AtomicInteger(0);

// 队头
private transient Node<E> head;

// 队尾
private transient Node<E> last;

// take, poll, peek 等读操作的方法需要获取到这个锁
private final ReentrantLock takeLock = new ReentrantLock();

// 如果读操作的时候队列是空的，那么等待 notEmpty 条件
private final Condition notEmpty = takeLock.newCondition();

// put, offer 等写操作的方法需要获取到这个锁
private final ReentrantLock putLock = new ReentrantLock();

// 如果写操作的时候队列是满的，那么等待 notFull 条件
private final Condition notFull = putLock.newCondition();
```

> LinkedBlockingQueue 源码与ArrayBlockingQueue相似，较为简单

# 14.4 BlockingQueue 实现之 SynchronousQueue

它是一个特殊的队列，它的名字其实就蕴含了它的特征 - - 同步的队列。为什么说是同步的呢？这里说的并不是多线程的并发问题，而是因为当一个线程往队列中写入一个元素时，写入操作不会立即返回，需要等待另一个线程来将这个元素拿走；同理，当一个读线程做读操作的时候，同样需要一个相匹配的写线程的写操作。这里的 Synchronous 指的就是读线程和写线程需要同步，一个读线程匹配一个写线程。

我们比较少使用到 SynchronousQueue 这个类，不过它在线程池的实现类 ThreadPoolExecutor 中得到了应用。

## 14.4.1 源码
```
// 构造时，我们可以指定公平模式还是非公平模式，区别之后再说
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue() : new TransferStack();
}
abstract static class Transferer {
    // 从方法名上大概就知道，这个方法用于转移元素，从生产者手上转到消费者手上
    // 也可以被动地，消费者调用这个方法来从生产者手上取元素
    // 第一个参数 e 如果不是 null，代表场景为：将元素从生产者转移给消费者
    // 如果是 null，代表消费者等待生产者提供元素，然后返回值就是相应的生产者提供的元素
    // 第二个参数代表是否设置超时，如果设置超时，超时时间是第三个参数的值
    // 返回值如果是 null，代表超时，或者中断。具体是哪个，可以通过检测中断状态得到。
    abstract Object transfer(Object e, boolean timed, long nanos);
}
```
Tranferer有两个内部实现类，公平模式意味着，所有线程遵守先来先到，FIFO，对应TranferQueue。非公平模式对应TransferStack。
```
// 写入值
public void put(E o) throws InterruptedException {
    if (o == null) throw new NullPointerException();
    if (transferer.transfer(o, false, 0) == null) { // 1
        Thread.interrupted();
        throw new InterruptedException();
    }
}
// 读取值并移除
public E take() throws InterruptedException {
    Object e = transferer.transfer(null, false, 0); // 2
    if (e != null)
        return (E)e;
    Thread.interrupted();
    throw new InterruptedException();
}
```
- 写操作 put(E o) 和读操作 take() 都是调用 Transferer.transfer(…) 方法，区别在于第一个参数是否为 null 值。
- 设计思路如下
    + 当调用这个方法的时候，如果队列是空的，或者队列中的节点和当前的线程操作类型一致（如当前操作是put操作，而队列中的元素也都是写线程）。这种情况下，将当前线程加入到等待队列即可。
    + 如果队列中有等待节点，而且与当前操作可以匹配（如队列中都是读操作线程，当前线程都是写操作线程，反之亦然）。这种情况下，匹配等待队列的队头，出队，返回数据。


这里有个隐含条件，队列如果不为空，肯定都是同种类型的节点，要么都是读操作，要么都是写操作。要看是读线程积压，还是写线程积压。

1. QNode
```
static final class QNode {
    volatile QNode next;          // 可以看出来，等待队列是单向链表
    volatile Object item;         // CAS'ed to or from null
    volatile Thread waiter;       // 将线程对象保存在这里，用于挂起和唤醒
    final boolean isData;         // 用于判断是写线程节点(isData == true)，还是读线程节点

    QNode(Object item, boolean isData) {
        this.item = item;
        this.isData = isData;
    }
}
```
2. transfer
```
/**
 * Puts or takes an item.
 */
Object transfer(Object e, boolean timed, long nanos) {

    QNode s = null; // constructed/reused as needed
    boolean isData = (e != null);

    for (;;) {
        QNode t = tail;
        QNode h = head;
        if (t == null || h == null)         // saw uninitialized value
            continue;                       // spin

        // 队列空，或队列中节点类型和当前节点一致，
        // 即我们说的第一种情况，将节点入队即可。读者要想着这块 if 里面方法其实就是入队
        if (h == t || t.isData == isData) { // empty or same-mode
            QNode tn = t.next;
            // t != tail 说明刚刚有节点入队，continue 即可
            if (t != tail)                  // inconsistent read
                continue;
            // 有其他节点入队，但是 tail 还是指向原来的，此时设置 tail 即可
            if (tn != null) {               // lagging tail
                // 这个方法就是：如果 tail 此时为 t 的话，设置为 tn
                advanceTail(t, tn);
                continue;
            }
            // 
            if (timed && nanos <= 0)        // can't wait
                return null;
            if (s == null)
                s = new QNode(e, isData);
            // 将当前节点，插入到 tail 的后面
            if (!t.casNext(null, s))        // failed to link in
                continue;

            // 将当前节点设置为新的 tail
            advanceTail(t, s);              // swing tail and wait
            // 看到这里，请读者先往下滑到这个方法，看完了以后再回来这里，思路也就不会断了
            Object x = awaitFulfill(s, e, timed, nanos);
            // 到这里，说明之前入队的线程被唤醒了，准备往下执行
            if (x == s) {                   // wait was cancelled
                clean(t, s);
                return null;
            }

            if (!s.isOffList()) {           // not already unlinked
                advanceHead(t, s);          // unlink if head
                if (x != null)              // and forget fields
                    s.item = s;
                s.waiter = null;
            }
            return (x != null) ? x : e;

        // 这里的 else 分支就是上面说的第二种情况，有相应的读或写相匹配的情况
        } else {                            // complementary-mode
            QNode m = h.next;               // node to fulfill
            if (t != tail || m == null || h != head)
                continue;                   // inconsistent read

            Object x = m.item;
            if (isData == (x != null) ||    // m already fulfilled
                x == m ||                   // m cancelled
                !m.casItem(x, e)) {         // lost CAS
                advanceHead(h, m);          // dequeue and retry
                continue;
            }

            advanceHead(h, m);              // successfully fulfilled
            LockSupport.unpark(m.waiter);
            return (x != null) ? x : e;
        }
    }
}

void advanceTail(QNode t, QNode nt) {
    if (tail == t)
        UNSAFE.compareAndSwapObject(this, tailOffset, t, nt);
}
```
```
// 自旋或阻塞，直到满足条件，这个方法返回
Object awaitFulfill(QNode s, Object e, boolean timed, long nanos) {

    long lastTime = timed ? System.nanoTime() : 0;
    Thread w = Thread.currentThread();
    // 判断需要自旋的次数，
    int spins = ((head.next == s) ?
                 (timed ? maxTimedSpins : maxUntimedSpins) : 0);
    for (;;) {
        // 如果被中断了，那么取消这个节点
        if (w.isInterrupted())
            // 就是将当前节点 s 中的 item 属性设置为 this
            s.tryCancel(e);
        Object x = s.item;
        // 这里是这个方法的唯一的出口
        if (x != e)
            return x;
        // 如果需要，检测是否超时
        if (timed) {
            long now = System.nanoTime();
            nanos -= now - lastTime;
            lastTime = now;
            if (nanos <= 0) {
                s.tryCancel(e);
                continue;
            }
        }
        if (spins > 0)
            --spins;
        // 如果自旋达到了最大的次数，那么检测
        else if (s.waiter == null)
            s.waiter = w;
        // 如果自旋到了最大的次数，那么线程挂起，等待唤醒
        else if (!timed)
            LockSupport.park(this);
        // spinForTimeoutThreshold 这个之前讲 AQS 的时候其实也说过，剩余时间小于这个阈值的时候，就
        // 不要进行挂起了，自旋的性能会比较好
        else if (nanos > spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanos);
    }
}
```
## 14.5 BlockingQueue 实现之 PriorityBlockingQueue

- PriorityBlockingQueue 只能指定初始的队列大小，后面插入元素的时候，如果空间不够的话会自动扩容）。
- 是PriorityQueue 的线程安全版本。不可以插入 null 值，同时，插入队列的对象必须是可比较大小的（comparable），否则报 ClassCastException 异常。它的插入操作 put 方法不会 block，因为它是无界队列（take 方法在队列为空的时候会阻塞）

1. 属性
```
// 构造方法中，如果不指定大小的话，默认大小为 11
private static final int DEFAULT_INITIAL_CAPACITY = 11;
// 数组的最大容量
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

// 这个就是存放数据的数组
private transient Object[] queue;

// 队列当前大小
private transient int size;

// 大小比较器，如果按照自然序排序，那么此属性可设置为 null
private transient Comparator<? super E> comparator;

// 并发控制所用的锁，所有的 public 且涉及到线程安全的方法，都必须先获取到这个锁
private final ReentrantLock lock;

// 这个很好理解，其实例由上面的 lock 属性创建
private final Condition notEmpty;

// 这个也是用于锁，用于数组扩容的时候，需要先获取到这个锁，才能进行扩容操作
// 其使用 CAS 操作
private transient volatile int allocationSpinLock;

// 用于序列化和反序列化的时候用，对于 PriorityBlockingQueue 我们应该比较少使用到序列化
private PriorityQueue q;
```
2. 构造方法
```
// 默认构造方法，采用默认值(11)来进行初始化
public PriorityBlockingQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}
// 指定数组的初始大小
public PriorityBlockingQueue(int initialCapacity) {
    this(initialCapacity, null);
}
// 指定比较器
public PriorityBlockingQueue(int initialCapacity,
                             Comparator<? super E> comparator) {
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.lock = new ReentrantLock();
    this.notEmpty = lock.newCondition();
    this.comparator = comparator;
    this.queue = new Object[initialCapacity];
}
// 在构造方法中就先填充指定的集合中的元素
public PriorityBlockingQueue(Collection<? extends E> c) {
    this.lock = new ReentrantLock();
    this.notEmpty = lock.newCondition();
    // 
    boolean heapify = true; // true if not known to be in heap order
    boolean screen = true;  // true if must screen for nulls
    if (c instanceof SortedSet<?>) {
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
        this.comparator = (Comparator<? super E>) ss.comparator();
        heapify = false;
    }
    else if (c instanceof PriorityBlockingQueue<?>) {
        PriorityBlockingQueue<? extends E> pq =
            (PriorityBlockingQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        screen = false;
        if (pq.getClass() == PriorityBlockingQueue.class) // exact match
            heapify = false;
    }
    Object[] a = c.toArray();
    int n = a.length;
    // If c.toArray incorrectly doesn't return Object[], copy it.
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, n, Object[].class);
    if (screen && (n == 1 || this.comparator != null)) {
        for (int i = 0; i < n; ++i)
            if (a[i] == null)
                throw new NullPointerException();
    }
    this.queue = a;
    this.size = n;
    if (heapify)
        heapify();
}
```
3. 自动扩容
```
private void tryGrow(Object[] array, int oldCap) {
    // 这边做了释放锁的操作
    lock.unlock(); // must release and then re-acquire main lock
    Object[] newArray = null;
    // 用 CAS 操作将 allocationSpinLock 由 0 变为 1，也算是获取锁
    if (allocationSpinLock == 0 &&
        UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset,
                                 0, 1)) {
        try {
            // 如果节点个数小于 64，那么增加的 oldCap + 2 的容量
            // 如果节点数大于等于 64，那么增加 oldCap 的一半
            // 所以节点数较小时，增长得快一些
            int newCap = oldCap + ((oldCap < 64) ?
                                   (oldCap + 2) :
                                   (oldCap >> 1));
            // 这里有可能溢出
            if (newCap - MAX_ARRAY_SIZE > 0) {    // possible overflow
                int minCap = oldCap + 1;
                if (minCap < 0 || minCap > MAX_ARRAY_SIZE)
                    throw new OutOfMemoryError();
                newCap = MAX_ARRAY_SIZE;
            }
            // 如果 queue != array，那么说明有其他线程给 queue 分配了其他的空间
            if (newCap > oldCap && queue == array)
                // 分配一个新的大数组
                newArray = new Object[newCap];
        } finally {
            // 重置，也就是释放锁
            allocationSpinLock = 0;
        }
    }
    // 如果有其他的线程也在做扩容的操作
    if (newArray == null) // back off if another thread is allocating
        Thread.yield();
    // 重新获取锁
    lock.lock();
    // 将原来数组中的元素复制到新分配的大数组中
    if (newArray != null && queue == array) {
        queue = newArray;
        System.arraycopy(array, 0, newArray, 0, oldCap);
    }
}
```
4 put
```
public void put(E e) {
    // 直接调用 offer 方法，因为前面我们也说了，在这里，put 方法不会阻塞
    offer(e); 
}
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    // 首先获取到独占锁
    lock.lock();
    int n, cap;
    Object[] array;
    // 如果当前队列中的元素个数 >= 数组的大小，那么需要扩容了
    while ((n = size) >= (cap = (array = queue).length))
        tryGrow(array, cap);
    try {
        Comparator<? super E> cmp = comparator;
        // 节点添加到二叉堆中
        if (cmp == null)
            siftUpComparable(n, e, array);
        else
            siftUpUsingComparator(n, e, array, cmp);
        // 更新 size
        size = n + 1;
        // 唤醒等待的读线程
        notEmpty.signal();
    } finally {
        lock.unlock();
    }
    return true;
}
```
```
// 这个方法就是将数据 x 插入到数组 array 的位置 k 处，然后再调整树
private static <T> void siftUpComparable(int k, T x, Object[] array) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        // 二叉堆中 a[k] 节点的父节点位置
        int parent = (k - 1) >>> 1;
        Object e = array[parent];
        if (key.compareTo((T) e) >= 0)
            break;
        array[k] = e;
        k = parent;
    }
    array[k] = key;
}
```
5. take
```
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    // 独占锁
    lock.lockInterruptibly();
    E result;
    try {
        // dequeue 出队
        while ( (result = dequeue()) == null)
            notEmpty.await();
    } finally {
        lock.unlock();
    }
    return result;
}
```
```
private E dequeue() {
    int n = size - 1;
    if (n < 0)
        return null;
    else {
        Object[] array = queue;
        // 队头，用于返回
        E result = (E) array[0];
        // 队尾元素先取出
        E x = (E) array[n];
        // 队尾置空
        array[n] = null;
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            siftDownComparable(0, x, array, n);
        else
            siftDownUsingComparator(0, x, array, n, cmp);
        size = n;
        return result;
    }
}
```
```
private static <T> void siftDownComparable(int k, T x, Object[] array,
                                           int n) {
    if (n > 0) {
        Comparable<? super T> key = (Comparable<? super T>)x;
        // 这里得到的 half 肯定是非叶节点
        // a[n] 是最后一个元素，其父节点是 a[(n-1)/2]。所以 n >>> 1 代表的节点肯定不是叶子节点
        // 下面，我们结合图来一行行分析，这样比较直观简单
        // 此时 k 为 0, x 为 17，n 为 9
        int half = n >>> 1; // 得到 half = 4
        while (k < half) {
            // 先取左子节点
            int child = (k << 1) + 1; // 得到 child = 1
            Object c = array[child];  // c = 12
            int right = child + 1;  // right = 2
            // 如果右子节点存在，而且比左子节点小
            // 此时 array[right] = 20，所以条件不满足
            if (right < n &&
                ((Comparable<? super T>) c).compareTo((T) array[right]) > 0)
                c = array[child = right];
            // key = 17, c = 12，所以条件不满足
            if (key.compareTo((T) c) <= 0)
                break;
            // 把 12 填充到根节点
            array[k] = c;
            // k 赋值后为 1
            k = child;
            // 一轮过后，我们发现，12 左边的子树和刚刚的差不多，都是缺少根节点，接下来处理就简单了
        }
        array[k] = key;
    }
}
```

# 总结
- ArrayBlockingQueue 底层是数组，有界队列，如果我们要使用生产者-消费者模式，这是非常好的选择。
- LinkedBlockingQueue 底层是链表，可以当做无界和有界队列来使用，所以大家不要以为它就是无界队列。
- SynchronousQueue 本身不带有空间来存储任何元素，使用上可以选择公平模式和非公平模式。
- PriorityBlockingQueue 是无界队列，基于数组，数据结构为二叉堆，数组第一个也是树的根节点总是最小值。