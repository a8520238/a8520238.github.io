---
title: AQS并发工具类
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 1 CountDownLatch

## 1.1 使用场景

假设我们有 N ( N > 0 ) 个任务，那么我们会用 N 来初始化一个 CountDownLatch，然后将这个 latch 的引用传递到各个线程中，在每个线程完成了任务后，调用 latch.countDown() 代表完成了一个任务。

调用latch.await()的方法的线程会阻塞，直到所有任务完成。

```
class Driver2 { // ...
    void main() throws InterruptedException {
        CountDownLatch doneSignal = new CountDownLatch(N);
        Executor e = Executors.newFixedThreadPool(8);

        // 创建 N 个任务，提交给线程池来执行
        for (int i = 0; i < N; ++i) // create and start threads
            e.execute(new WorkerRunnable(doneSignal, i));

        // 等待所有的任务完成，这个方法才会返回
        doneSignal.await();           // wait for all to finish
    }
}

class WorkerRunnable implements Runnable {
    private final CountDownLatch doneSignal;
    private final int i;

    WorkerRunnable(CountDownLatch doneSignal, int i) {
        this.doneSignal = doneSignal;
        this.i = i;
    }

    public void run() {
        try {
            doWork(i);
            // 这个线程的任务完成了，调用 countDown 方法
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```
当我们将一个较大的任务进行拆分，然后开启多个线程执行，等所有线程都执行完了以后，再往下执行其他操作。

下面的列子，用了两个CountDownLatch
```
class Driver { // ...
    void main() throws InterruptedException {
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(N);

        for (int i = 0; i < N; ++i) // create and start threads
            new Thread(new Worker(startSignal, doneSignal)).start();

        // 这边插入一些代码，确保上面的每个线程先启动起来，才执行下面的代码。
        doSomethingElse();            // don't let run yet
        // 因为这里 N == 1，所以，只要调用一次，那么所有的 await 方法都可以通过
        startSignal.countDown();      // let all threads proceed
        doSomethingElse();
        // 等待所有任务结束
        doneSignal.await();           // wait for all to finish
    }
}

class Worker implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;

    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
    }

    public void run() {
        try {
            // 为了让所有线程同时开始任务，我们让所有线程先阻塞在这里
            // 等大家都准备好了，再打开这个门栓
            startSignal.await();
            doWork();
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```
整体来说，就是有m个线程是做任务的，有n个线程在某个栅栏上等待这m个线程做完任务，直到所有m个任务完成后，n个线程同时同时通过栅栏。
## 1.2 源码分析
[源码分析](https://github.com/h2pl/Java-Tutorial/blob/master/docs/java/currency/Java%E5%B9%B6%E5%8F%91%E6%8C%87%E5%8D%979%EF%BC%9AAQS%E5%85%B1%E4%BA%AB%E6%A8%A1%E5%BC%8F%E4%B8%8E%E5%B9%B6%E5%8F%91%E5%B7%A5%E5%85%B7%E7%B1%BB%E7%9A%84%E5%AE%9E%E7%8E%B0.md)

# 2 CyclicBarrier

- CyclicBarrier意思是周期性栅栏、
- CountDownLatch是基于AQS的共享模式的使用，而CyclicBarrier是基于Condition来实现

![](http://note.youdao.com/yws/public/resource/5cd10a62158ca44fb1f7fbe48671fb51/xmlnote/93D8DE706E794E0D9AEA7493D09A07E1/11264)

- 由上图可知CyclicBarrier的源码最重要的是await()方法

## 2.1 源码分析

```
public class CyclicBarrier {
    // 我们说了，CyclicBarrier 是可以重复使用的，我们把每次从开始使用到穿过栅栏当做"一代"，或者"一个周期"
    private static class Generation {
        boolean broken = false;
    }

    /** The lock for guarding barrier entry */
    private final ReentrantLock lock = new ReentrantLock();

    // CyclicBarrier 是基于 Condition 的
    // Condition 是“条件”的意思，CyclicBarrier 的等待线程通过 barrier 的“条件”是大家都到了栅栏上
    private final Condition trip = lock.newCondition();

    // 参与的线程数
    private final int parties;

    // 如果设置了这个，代表越过栅栏之前，要执行相应的操作
    private final Runnable barrierCommand;

    // 当前所处的“代”
    private Generation generation = new Generation();

    // 还没有到栅栏的线程数，这个值初始为 parties，然后递减
    // 还没有到栅栏的线程数 = parties - 已经到栅栏的数量
    private int count;

    public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }

    public CyclicBarrier(int parties) {
        this(parties, null);
    }
```
1. 开启新一代
```
// 开启新的一代，当最后一个线程到达栅栏上的时候，调用这个方法来唤醒其他线程，同时初始化“下一代”
private void nextGeneration() {
    // 首先，需要唤醒所有的在栅栏上等待的线程
    trip.signalAll();
    // 更新 count 的值
    count = parties;
    // 重新生成“新一代”
    generation = new Generation();
}
```
> 开启新一代，类似于重新实例化一个CyclicBarrier实例
2. 打破一个栅栏

```
private void breakBarrier() {
    // 设置状态 broken 为 true
    generation.broken = true;
    // 重置 count 为初始值 parties
    count = parties;
    // 唤醒所有已经在等待的线程
    trip.signalAll();
}
```
3. await方法

```
// 不带超时机制
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
// 带超时机制，如果超时抛出 TimeoutException 异常
public int await(long timeout, TimeUnit unit)
    throws InterruptedException,
           BrokenBarrierException,
           TimeoutException {
    return dowait(true, unit.toNanos(timeout));
}
```

```
private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
    final ReentrantLock lock = this.lock;
    // 先要获取到锁，然后在 finally 中要记得释放锁
    // 如果记得 Condition 部分的话，我们知道 condition 的 await() 会释放锁，被 signal() 唤醒的时候需要重新获取锁
    lock.lock();
    try {
        final Generation g = generation;
        // 检查栅栏是否被打破，如果被打破，抛出 BrokenBarrierException 异常
        if (g.broken)
            throw new BrokenBarrierException();
        // 检查中断状态，如果中断了，抛出 InterruptedException 异常
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }
        // index 是这个 await 方法的返回值
        // 注意到这里，这个是从 count 递减后得到的值
        int index = --count;

        // 如果等于 0，说明所有的线程都到栅栏上了，准备通过
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                // 如果在初始化的时候，指定了通过栅栏前需要执行的操作，在这里会得到执行
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                // 如果 ranAction 为 true，说明执行 command.run() 的时候，没有发生异常退出的情况
                ranAction = true;
                // 唤醒等待的线程，然后开启新的一代
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    // 进到这里，说明执行指定操作的时候，发生了异常，那么需要打破栅栏
                    // 之前我们说了，打破栅栏意味着唤醒所有等待的线程，设置 broken 为 true，重置 count 为 parties
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        // 如果是最后一个线程调用 await，那么上面就返回了
        // 下面的操作是给那些不是最后一个到达栅栏的线程执行的
        for (;;) {
            try {
                // 如果带有超时机制，调用带超时的 Condition 的 await 方法等待，直到最后一个线程调用 await
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                // 如果到这里，说明等待的线程在 await（是 Condition 的 await）的时候被中断
                if (g == generation && ! g.broken) {
                    // 打破栅栏
                    breakBarrier();
                    // 打破栅栏后，重新抛出这个 InterruptedException 异常给外层调用的方法
                    throw ie;
                } else {
                    // 到这里，说明 g != generation, 说明新的一代已经产生，即最后一个线程 await 执行完成，
                    // 那么此时没有必要再抛出 InterruptedException 异常，记录下来这个中断信息即可
                    // 或者是栅栏已经被打破了，那么也不应该抛出 InterruptedException 异常，
                    // 而是之后抛出 BrokenBarrierException 异常
                    Thread.currentThread().interrupt();
                }
            }

              // 唤醒后，检查栅栏是否是“破的”
            if (g.broken)
                throw new BrokenBarrierException();

            // 这个 for 循环除了异常，就是要从这里退出了
            // 我们要清楚，最后一个线程在执行完指定任务(如果有的话)，会调用 nextGeneration 来开启一个新的代
            // 然后释放掉锁，其他线程从 Condition 的 await 方法中得到锁并返回，然后到这里的时候，其实就会满足 g != generation 的
            // 那什么时候不满足呢？barrierCommand 执行过程中抛出了异常，那么会执行打破栅栏操作，
            // 设置 broken 为true，然后唤醒这些线程。这些线程会从上面的 if (g.broken) 这个分支抛 BrokenBarrierException 异常返回
            // 当然，还有最后一种可能，那就是 await 超时，此种情况不会从上面的 if 分支异常返回，也不会从这里返回，会执行后面的代码
            if (g != generation)
                return index;

            // 如果醒来发现超时了，打破栅栏，抛出异常
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```

- 使用场景
```
public class CyclicBarrierDemo {
    static class PreTaskThread implements Runnable {

        private String task;
        private CyclicBarrier cyclicBarrier;

        public PreTaskThread(String task, CyclicBarrier cyclicBarrier) {
            this.task = task;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            // 假设总共三个关卡
            for (int i = 1; i < 4; i++) {
                try {
                    Random random = new Random();
                    Thread.sleep(random.nextInt(1000));
                    System.out.println(String.format("关卡%d的任务%s完成", i, task));
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                cyclicBarrier.reset(); // 重置屏障
            }
        }
    }

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
            System.out.println("本关卡所有前置任务完成，开始游戏...");
        });

        new Thread(new PreTaskThread("加载地图数据", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载人物模型", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载背景音乐", cyclicBarrier)).start();
    }
}
```

# 3 Semaphore

- Semaphore类似一个资源池（可以类比线程池），每个线程需要调用acquire()方法获取资源，然后才能执行，需要release资源，让给其他的线程用。
- 总结来说， Semaphore是AQS中共享锁的使用，因为每个线程共享一个池。
- Semaphore同时支持公平锁和非公平锁。

1. acquire
```
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
public void acquireUninterruptibly() {
    sync.acquireShared(1);
}
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}
public void acquireUninterruptibly(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireShared(permits);
}
```

```
public void acquireUninterruptibly() {
    sync.acquireShared(1);
}
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

2. 对比公平和非公平的tryAcquireShared方法
```
// 公平策略：
protected int tryAcquireShared(int acquires) {
    for (;;) {
        // 区别就在于是不是会先判断是否有线程在排队，然后才进行 CAS 减操作
        if (hasQueuedPredecessors())
            return -1;
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
// 非公平策略：
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```
```
//如果tryAcquireShared(arg) 返回小于 0 ， 说明 state 已经小于 0 了（没资源了），此时 acquire 不能立马拿到资源，需要进入到阻塞队列等待
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```
```
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            //从这里醒来后，会尝试获得锁，如果能获得，继续向后传递，不过不能则阻塞。
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
3. release
```
// 任务介绍，释放一个资源
public void release() {
    sync.releaseShared(1);
}
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        int next = current + releases;
        // 溢出，当然，我们一般也不会用这么大的数
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
}
```
```
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

# 4 ReentrantReadWriterLock

- 一个缓存的实例
```
// 这是一个关于缓存操作的故事
class CachedData {
    Object data;
    volatile boolean cacheValid;
    // 读写锁实例
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

    void processCachedData() {
        // 获取读锁
        rwl.readLock().lock();
        if (!cacheValid) { // 如果缓存过期了，或者为 null
            // 释放掉读锁，然后获取写锁 (后面会看到，没释放掉读锁就获取写锁，会发生死锁情况)
            rwl.readLock().unlock();
            rwl.writeLock().lock();

            try {
                if (!cacheValid) { // 重新判断，因为在等待写锁的过程中，可能前面有其他写线程执行过了
                    data = ...
                    cacheValid = true;
                }
                // 获取读锁 (持有写锁的情况下，是允许获取读锁的，称为 “锁降级”，反之不行。)
                rwl.readLock().lock();
            } finally {
                // 释放写锁，此时还剩一个读锁
                rwl.writeLock().unlock(); // Unlock write, still hold read
            }
        }

        try {
            use(data);
        } finally {
            // 释放读锁
            rwl.readLock().unlock();
        }
    }
}
```
- ReentrantReadWriterLock分为读锁和写锁两个实例，读锁时共享锁，可被多个线程同时使用，写锁是独占锁。持有写锁的线程可以继续获取读锁，反之不行。

## 4.1 源码分析-属性
```
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 下面这块说的就是将 state 一分为二，高 16 位用于共享模式，低16位用于独占模式
    static final int SHARED_SHIFT   = 16;
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
    // 取 c 的高 16 位值，代表读锁的获取次数(包括重入)
    static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
    // 取 c 的低 16 位值，代表写锁的重入次数，因为写锁是独占模式
    static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

    // 这个嵌套类的实例用来记录每个线程持有的读锁数量(读锁重入)
    static final class HoldCounter {
        // 持有的读锁数
        int count = 0;
        // 线程 id
        final long tid = getThreadId(Thread.currentThread());
    }

    // ThreadLocal 的子类
    static final class ThreadLocalHoldCounter
        extends ThreadLocal<HoldCounter> {
        public HoldCounter initialValue() {
            return new HoldCounter();
        }
    }
    /**
      * 组合使用上面两个类，用一个 ThreadLocal 来记录当前线程持有的读锁数量
      */ 
    private transient ThreadLocalHoldCounter readHolds;

    // 用于缓存，记录"最后一个获取读锁的线程"的读锁重入次数，
    // 所以不管哪个线程获取到读锁后，就把这个值占为已用，这样就不用到 ThreadLocal 中查询 map 了
    // 算不上理论的依据：通常读锁的获取很快就会伴随着释放，
    //   显然，在 获取->释放 读锁这段时间，如果没有其他线程获取读锁的话，此缓存就能帮助提高性能
    private transient HoldCounter cachedHoldCounter;

    // 第一个获取读锁的线程(并且其未释放读锁)，以及它持有的读锁数量
    private transient Thread firstReader = null;
    private transient int firstReaderHoldCount;

    Sync() {
        // 初始化 readHolds 这个 ThreadLocal 属性
        readHolds = new ThreadLocalHoldCounter();
        // 为了保证 readHolds 的内存可见性
        setState(getState()); // ensures visibility of readHolds
    }
    ...
}
```
1. state 的高 16 位代表读锁的获取次数，包括重入次数，获取到读锁一次加 1，释放掉读锁一次减 1
2. state 的低 16 位代表写锁的获取次数，因为写锁是独占锁，同时只能被一个线程获得，所以它代表重入次数
3. 每个线程都需要维护自己的 HoldCounter，记录该线程获取的读锁次数，这样才能知道到底是不是读锁重入，用 ThreadLocal 属性 readHolds 维护
4. cachedHoldCounter 有什么用？其实没什么用，但能提示性能。将最后一次获取读锁的线程的 HoldCounter 缓存到这里，这样比使用 ThreadLocal 性能要好一些，因为 ThreadLocal 内部是基于 map 来查询的。但是 cachedHoldCounter 这一个属性毕竟只能缓存一个线程，所以它要起提升性能作用的依据就是：通常读锁的获取紧随着就是该读锁的释放。我这里可能表达不太好，但是大家应该是懂的吧。
5. firstReader 和 firstReaderHoldCount 有什么用？其实也没什么用，但是它也能提示性能。将"第一个"获取读锁的线程记录在 firstReader 属性中，这里的第一个不是全局的概念，等这个 firstReader 当前代表的线程释放掉读锁以后，会有后来的线程占用这个属性的。firstReader 和 firstReaderHoldCount 使得在读锁不产生竞争的情况下，记录读锁重入次数非常方便快速
6. 如果一个线程使用了 firstReader，那么它就不需要占用 cachedHoldCounter
7. 个人认为，读写锁源码中最让初学者头疼的就是这几个用于提升性能的属性了，使得大家看得云里雾里的。主要是因为 ThreadLocal 内部是通过一个 ThreadLocalMap 来操作的，会增加检索时间。而很多场景下，执行 unlock 的线程往往就是刚刚最后一次执行 lock 的线程，中间可能没有其他线程进行 lock。还有就是很多不怎么会发生读锁竞争的场景。

## 4.2 读锁

1. 读锁ReadLock的lock流程
```
// ReadLock
public void lock() {
    sync.acquireShared(1);
}
// AQS
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

- 下面这段代码，有两种获取读锁的场景，一种是新来的，一种是排队排到了
```
protected final int tryAcquireShared(int unused) {

    Thread current = Thread.currentThread();
    int c = getState();

    // exclusiveCount(c) 不等于 0，说明有线程持有写锁，
    //    而且不是当前线程持有写锁，那么当前线程获取读锁失败
    //         （另，如果持有写锁的是当前线程，是可以继续获取读锁的）
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;

    // 读锁的获取次数
    int r = sharedCount(c);

    // 读锁获取是否需要被阻塞，稍后细说。为了进去下面的分支，假设这里不阻塞就好了
    if (!readerShouldBlock() &&
        // 判断是否会溢出 (2^16-1，没那么容易溢出的)
        r < MAX_COUNT &&
        // 下面这行 CAS 是将 state 属性的高 16 位加 1，低 16 位不变，如果成功就代表获取到了读锁
        compareAndSetState(c, c + SHARED_UNIT)) {

        // =======================
        //   进到这里就是获取到了读锁
        // =======================

        if (r == 0) {
            // r == 0 说明此线程是第一个获取读锁的，或者说在它前面获取读锁的都走光光了，它也算是第一个吧
            //  记录 firstReader 为当前线程，及其持有的读锁数量：1
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            // 进来这里，说明是 firstReader 重入获取读锁（这非常简单，count 加 1 结束）
            firstReaderHoldCount++;
        } else {
            // 前面我们说了 cachedHoldCounter 用于缓存最后一个获取读锁的线程
            // 如果 cachedHoldCounter 缓存的不是当前线程，设置为缓存当前线程的 HoldCounter
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0) 
                // 到这里，那么就是 cachedHoldCounter 缓存的是当前线程，但是 count 为 0，
                // 大家可以思考一下：这里为什么要 set ThreadLocal 呢？(当然，答案肯定不在这块代码中)
                //   既然 cachedHoldCounter 缓存的是当前线程，
                //   当前线程肯定调用过 readHolds.get() 进行初始化 ThreadLocal
                readHolds.set(rh);

            // count 加 1
            rh.count++;
        }
        // return 大于 0 的数，代表获取到了共享锁
        return 1;
    }
    // 往下看
    return fullTryAcquireShared(current);
}
```
- readerShouldBlock() 返回 true，2 种情况：
    + 在 FairSync 中说的是 hasQueuedPredecessors()，即阻塞队列中有其他元素在等待锁。也就是说，公平模式下，新来的要排队
    + 在 NonFairSync 中说的是 apparentlyFirstQueuedIsExclusive()，即判断阻塞队列中 head 的第一个后继节点是否是来获取写锁的，如果是的话，让这个写锁先来，避免写锁饥饿。
    > 作者给写锁定义了更高的优先级，所以如果碰上获取写锁的线程马上就要获取到锁了，获取读锁的线程不应该和它抢。

    > 如果 head.next 不是来获取写锁的，那么可以随便抢，因为是非公平模式，大家比比 CAS 速度
- compareAndSetState(c, c + SHARED_UNIT) 这里 CAS 失败，存在竞争。可能是和另一个读锁获取竞争，当然也可能是和另一个写锁获取操作竞争。

随后进入fullTryAcquireShared中尝试：
```
/**
 * 1\. 刚刚我们说了可能是因为 CAS 失败，如果就此返回，那么就要进入到阻塞队列了，
 *    想想有点不甘心，因为都已经满足了 !readerShouldBlock()，也就是说本来可以不用到阻塞队列的，
 *    所以进到这个方法其实是增加 CAS 成功的机会
 * 2\. 在 NonFairSync 情况下，虽然 head.next 是获取写锁的，我知道它等待很久了，我没想和它抢，
 *    可是如果我是来重入读锁的，那么只能表示对不起了
 */
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    // 别忘了这外层有个 for 循环
    for (;;) {
        int c = getState();
        // 如果其他线程持有了写锁，自然这次是获取不到读锁了，乖乖到阻塞队列排队吧
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        } else if (readerShouldBlock()) {
            /**
              * 进来这里，说明：
              *  1\. exclusiveCount(c) == 0：写锁没有被占用
              *  2\. readerShouldBlock() 为 true，说明阻塞队列中有其他线程在等待
              *
              * 既然 should block，那进来这里是干什么的呢？
              * 答案：是进来处理读锁重入的！
              * 
              */

            // firstReader 线程重入读锁，直接到下面的 CAS
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        // cachedHoldCounter 缓存的不是当前线程
                        // 那么到 ThreadLocal 中获取当前线程的 HoldCounter
                        // 如果当前线程从来没有初始化过 ThreadLocal 中的值，get() 会执行初始化
                        rh = readHolds.get();
                        // 如果发现 count == 0，也就是说，纯属上一行代码初始化的，那么执行 remove
                        // 然后往下两三行，乖乖排队去
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    // 排队去。
                    return -1;
            }
            /**
              * 这块代码我看了蛮久才把握好它是干嘛的，原来只需要知道，它是处理重入的就可以了。
              * 就是为了确保读锁重入操作能成功，而不是被塞到阻塞队列中等待
              *
              * 另一个信息就是，这里对于 ThreadLocal 变量 readHolds 的处理：
              *    如果 get() 后发现 count == 0，居然会做 remove() 操作，
              *    这行代码对于理解其他代码是有帮助的
              */
        }

        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");

        if (compareAndSetState(c, c + SHARED_UNIT)) {
            // 这里 CAS 成功，那么就意味着成功获取读锁了
            // 下面需要做的是设置 firstReader 或 cachedHoldCounter

            if (sharedCount(c) == 0) {
                // 如果发现 sharedCount(c) 等于 0，就将当前线程设置为 firstReader
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                // 下面这几行，就是将 cachedHoldCounter 设置为当前线程
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh;
            }
            // 返回大于 0 的数，代表获取到了读锁
            return 1;
        }
    }
}
```
> firstReader 是每次将读锁获取次数从 0 变为 1 的那个线程。
> 能缓存到 firstReader 中就不要缓存到 cachedHoldCounter 中。

2. 读锁释放
```
// ReadLock
public void unlock() {
    sync.releaseShared(1);
}
```
```
// Sync
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared(); // 这句代码其实唤醒 获取写锁的线程，往下看就知道了
        return true;
    }
    return false;
}

// Sync
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        if (firstReaderHoldCount == 1)
            // 如果等于 1，那么这次解锁后就不再持有锁了，把 firstReader 置为 null，给后来的线程用
            // 为什么不顺便设置 firstReaderHoldCount = 0？因为没必要，其他线程使用的时候自己会设值
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        // 判断 cachedHoldCounter 是否缓存的是当前线程，不是的话要到 ThreadLocal 中取
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();

        int count = rh.count;
        if (count <= 1) {

            // 这一步将 ThreadLocal remove 掉，防止内存泄漏。因为已经不再持有读锁了
            readHolds.remove();

            if (count <= 0)
                // 就是那种，lock() 一次，unlock() 好几次的逗比
                throw unmatchedUnlockException();
        }
        // count 减 1
        --rh.count;
    }

    for (;;) {
        int c = getState();
        // nextc 是 state 高 16 位减 1 后的值
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // 如果 nextc == 0，那就是 state 全部 32 位都为 0，也就是读锁和写锁都空了
            // 此时这里返回 true 的话，其实是帮助唤醒后继节点中的获取写锁的线程
            return nextc == 0;
    }
}
```
## 4.3 写锁

1. 写锁获取
- 写锁是独占锁
- 如果有读锁被占用，写锁获取是要进入到阻塞队列中等待的
```
// WriteLock
public void lock() {
    sync.acquire(1);
}
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        // 如果 tryAcquire 失败，那么进入到阻塞队列等待
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

// Sync
protected final boolean tryAcquire(int acquires) {

    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {

        // 看下这里返回 false 的情况：
        //   c != 0 && w == 0: 写锁可用，但是有线程持有读锁(也可能是自己持有)
        //   c != 0 && w !=0 && current != getExclusiveOwnerThread(): 其他线程持有写锁
        //   也就是说，只要有读锁或写锁被占用，这次就不能获取到写锁
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;

        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");

        // 这里不需要 CAS，仔细看就知道了，能到这里的，只可能是写锁重入，不然在上面的 if 就拦截了
        setState(c + acquires);
        return true;
    }

    // 如果写锁获取不需要 block，那么进行 CAS，成功就代表获取到了写锁
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```
- writerShouldBlock()判定
```
static final class NonfairSync extends Sync {
    // 如果是非公平模式，那么 lock 的时候就可以直接用 CAS 去抢锁，抢不到再排队
    final boolean writerShouldBlock() {
        return false; // writers can always barge
    }
    ...
}
static final class FairSync extends Sync {
    final boolean writerShouldBlock() {
        // 如果是公平模式，那么如果阻塞队列有线程等待的话，就乖乖去排队
        return hasQueuedPredecessors();
    }
    ...
}
```
2. 写锁释放
```
// WriteLock
public void unlock() {
    sync.release(1);
}

// AQS
public final boolean release(int arg) {
    // 1\. 释放锁
    if (tryRelease(arg)) {
        // 2\. 如果独占锁释放"完全"，唤醒后继节点
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

// Sync 
// 释放锁，是线程安全的，因为写锁是独占锁，具有排他性
// 实现很简单，state 减 1 就是了
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    // 如果 exclusiveCount(nextc) == 0，也就是说包括重入的，所有的写锁都释放了，
    // 那么返回 true，这样会进行唤醒后继节点的操作。
    return free;
}
```
## 4.4 锁降级
- 如果线程持有读锁，那么写锁也需要等待

- 在非公平模式下，为了提高吞吐量，lock的时候会先CAS竞争一下，能成功就代表读锁获取成功了，但是如果发现head.next是获取写锁的线程，就不会去做CAS操作。
- 持有写锁的线程，去获取读锁的过程称为锁降级。这样，线程就同时持有写锁和读锁。
- 锁升级是不被允许的，线程持有读锁的话，在没释放的情况不能获取写锁，因为会发生死锁。

> 如果线程 a 先获取了读锁，然后获取写锁，那么线程 a 就到阻塞队列休眠了，自己把自己弄休眠了，而且可能之后就没人去唤醒它了。

# 5 Exchanger

[java exchanger必知必会](https://www.jianshu.com/p/990ae2ab1ae0)