---
title: AQS
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 12AQS（AbstractQueuedSynchronizer）原理分析

- AQS是很多同步器的基础框架， ReentrantLock、CountDownLatch和Semaphore都是基于AQS实现的，AQS还可以定制同步器。
- AQS的使用方式通常是通过内部类继承AQS实现同步功能，通过继承AQS，可以简化同步器的实现。

## 12.1 原理简介
- 在AQS内部，维护了一个FIFO队列来管理多线程高的排队工作。
- 无法获取锁的线程将会被封装成一个节点，置于队列尾部。
    - 入队的线程将通过自旋的方式获取同步状态
    - 若在有限次的尝试，仍未获取成功，线程会被阻塞住
- 当头结点释放同步状态后，此时头结点线程将会唤醒后继节点阻塞的线程。后续节点线程恢复运行并获取同步状态后，会将旧的头结点从队列中移除，并将自己设为头结点。
- JDK中几种常见使用了AQS的同步组件：
    + ReentrantLock:使用了AQS的独占获取和释放，用state变量记录某个线程获取独占锁的次数，获取锁时+1，释放锁时-1，在获取时会校验线程是否可以获取锁。
    + Semaphore：使用了AQS的共享获取和释放，用state变量做为计数器，只有在大于0时允许线程进入。获取锁+1，释放锁-1.
    + CountDownLatch: 使用了AQS的共享获取和释放，用state变量做为计数器，在初始化时指定。只要state还大于0，获取共享锁会因为失败而阻塞，直到计数器的值为0时，共享锁才允许获取，所有等待线程会被逐一唤醒。

## 12.2 节点源码分析

```
static final class Node {
    /** 共享类型节点，标记节点在共享模式下等待 */
    static final Node SHARED = new Node();
    /** 独占类型节点，标记节点在独占模式下等待 */
    static final Node EXCLUSIVE = null;
    /** 等待状态 - 取消 */
    static final int CANCELLED =  1;
    //等待状态 - 通知。某个节点是处于该状态，当该节点释放同步状态后，会通知后继节点线程，使之可以恢复运行 
    static final int SIGNAL    = -1;
    /** 等待状态 - 条件等待。表明节点等待在 Condition 上 */
    static final int CONDITION = -2;
    // 等待状态 - 传播。表示无条件向后传播唤醒动作
    static final int PROPAGATE = -3;
   
    volatile int waitStatus;
    // 前驱节点
    volatile Node prev;
    //后继节点
    volatile Node next;
    //对应的线程
    volatile Thread thread;
    //下一个等待节点，用在 ConditionObject 中
    Node nextWaiter;
    //判断节点是否是共享节点
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
    // 获取前驱节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
    Node() {    // 用于建立初始标头或共享标头
    }
    /** addWaiter 方法会调用该构造方法 */
    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }
    /** Condition 中会用到此构造方法 */
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```
int waitStatus等待状态：
1. CANCELLED，值为1，由于同步队列中等待的线程等待超时或者被中断，需要从同步队列中取消等待，节点进入该状态将不会变化。
2. SIGHAL， 值为-1，后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行
3. CONDITION，值为-2，节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用了signal()方法后，该节点会从等待队列中转移到同步队列中，加入到对同步状态的获取中
4. PROPAGATE, 值为-3，表示下一次共享式同步状态获取将会无条件被传播下去。
5. INITIAL，值为0，初始状态
> AQS中0状态和CONDITION状态为始态，CANCELLED状态为终态。

> AQS中包含了head和tail两个Node引用，head节点实际上是一个虚节点，含义是当前持有锁的线程。head和tail在AQS中是延迟初始化的，意味着所有线程都能获取到锁的情况下，队列中的head和tail都会是null。

## 12.3 独占锁的实现

ReentrantLock类使用了AQS的独占锁的获取和释放

### 12.3.1 获取同步状态
1. 初始时，第一个线程调用lock获得了锁，首先会通过CAS的方式将state更新为1，并将线程持有者设置为该线程。
```
//ReentrantLock -> NofairSync -> lock（非公平锁）

final void lock() {
    if (compareAndSetState(0, 1))
        //setExclusiveOwnerThread是AbstractOwnableSynchronizer的方法，AQS继承了AbstractOwnableSynchronizer
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```
2. 这个时候有另一个线程2来尝试获得锁，同样调用lock方法，尝试通过CAS的方式将state更新为1，这时线程2通过CAS更新state失败。这时就会调用acquire(1)方法，（该方法是AQS提供的模板方法，它会调用子类的tryAcquire方法）

```
//AQS的模板方法会调用tryAcquire方法，tryAcquire又调用了Sync的nonfairTryAcquire的方法。

//NofairSync
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
final boolean nonfairTryAcquire(int acquires) {
    //（1）获取当前线程
    final Thread current = Thread.currentThread();
    //（2）获得当前同步状态state
    int c = getState();
    //（3）如果state==0，表示没有线程获取
    if (c == 0) {
        //（3-1）那么就尝试以CAS的方式更新state的值
        if (compareAndSetState(0, acquires)) {
            //（3-2）如果更新成功，就设置当前独占模式下同步状态的持有者为当前线程
            setExclusiveOwnerThread(current);
            //（3-3）获得成功之后，返回true
            return true;
        }
    }
    //（4）这里是重入锁的逻辑
    else if (current == getExclusiveOwnerThread()) {
        //（4-1）判断当前占有state的线程就是当前来再次获取state的线程之后，就计算重入后的state
        int nextc = c + acquires;
        //（4-2）这里是风险处理
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        //（4-3）通过setState无条件的设置state的值，（因为这里也只有一个线程操作state的值，即
        //已经获取到的线程，所以没有进行CAS操作）
        setState(nextc);
        return true;
    }
    //（5）没有获得state，也不是重入，就返回false
    return false;
}
```
3. 如果上述tryAcquire的过程失败返回false，我们需要将线程2构造成一个Node节点加入同步队列中。构造入口如下：
```
//(1)tryAcquire，这里thread2执行返回了false，那么就会执行addWaiter将当前线程构造为一个结点加入同步队列中
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
4. 下面就是addWaiter方法的执行流程
```
private Node addWaiter(Node mode) {
    //(1)将当前线程以及阻塞原因(是因为SHARED模式获取state失败还是EXCLUSIVE获取失败)构造为Node结点
    Node node = new Node(Thread.currentThread(), mode);
    //(2)这一步是快速将当前线程插入队列尾部
    Node pred = tail;
    if (pred != null) {
        //(2-1)将构造后的node结点的前驱结点设置为tail
        node.prev = pred;
        //(2-2)以CAS的方式设置当前的node结点为tail结点
        if (compareAndSetTail(pred, node)) {
            //(2-3)CAS设置成功，就将原来的tail的next结点设置为当前的node结点。这样这个双向队
            //列就更新完成了
            pred.next = node;
            return node;
        }
    }
    //(3)执行到这里，说明要么当前队列为null，要么存在多个线程竞争失败都去将自己设置为tail结点，
    //那么就会有线程在上面（2-2）的CAS设置中失败，就会到这里调用enq方法
    enq(node);
    return node;
}
```
- 将要去获取锁的线程和独占模式封装为一个node对象
- 通过CAS的方式在尾部插入当前节点
- 如果插入失败，进入enq方法
5. enq的实现
```
//通过CAS + 自旋的方式插入节点到队尾
private Node enq(final Node node) {
    for (;;) {
        //(4)还是先获取当前队列的tail结点
        Node t = tail;
        //(5)如果tail为null，表示当前同步队列为null，就必须初始化这个同步队列的head和tail（建
        //立一个哨兵结点）
        if (t == null) { 
            //（5-1）初始情况下，多个线程竞争失败，在检查的时候都发现没有哨兵结点，所以需要CAS的
            //设置哨兵结点
            if (compareAndSetHead(new Node()))
                tail = head;
        } 
        //(6)tail不为null
        else {
            //(6-1)直接将当前结点的前驱结点设置为tail结点
            node.prev = t;
            //(6-2)前驱结点设置完毕之后，还需要以CAS的方式将自己设置为tail结点，如果设置失败，
            //就会重新进入循环判断一遍
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
- 如果tail是null,设置哨兵节点
- 循环CAS修改尾结点为当前节点

6.addWaiter之后，外层方法是`acquireQueued`,这个方法的主要作用是在同步队列中嗅探自己的前驱节点，如果前驱节点是头结点的话，就会尝试获取同步状态。
```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        //在这样一个循环中尝试tryAcquire同步状态
        for (;;) {
            //获取前驱结点
            final Node p = node.predecessor();
            //(1)如果前驱结点是头节点，就尝试取获取同步状态，这里的tryAcquire方法相当于还是调
            //用NofairSync的tryAcquire方法，在上面已经说过
            if (p == head && tryAcquire(arg)) {
                //如果前驱结点是头节点并且tryAcquire返回true，那么就重新设置头节点为node
                setHead(node);
                p.next = null; //将原来的头节点的next设置为null，交由GC去回收它
                failed = false;
                return interrupted;
            }
            //(2)如果不是头节点,或者虽然前驱结点是头节点但是尝试获取同步状态失败就会将node结点
            //的waitStatus设置为-1(SIGNAL),并且park自己，等待前驱结点的唤醒。至于唤醒的细节
            //下面会说到 p节点是node节点的前驱节点，node节点是添加到队列的末尾。
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);//（3）如果没能成功重新获取锁，则取消获取
    }
}
```
这个方法会自旋，线程2的前驱节点为线程1，线程1的state被占有，所以tryAcquire失败，转到shouldParkAfterFailedAcquire方法。
7. shouldParkAfterFailedAcquire和parkAndCheckInterrupt
```
//方法名称的解释为：停止失败的获取
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //（1）获取前驱结点的waitStatus
    int ws = pred.waitStatus;
    //（2）如果前驱结点的waitStatus为SINGNAL，就直接返回true
    if (ws == Node.SIGNAL)
        //前驱结点的状态为SIGNAL，那么该结点就能够安全的调用park方法阻塞自己了。
        return true;
    if (ws > 0) {
        //（3）这里就是将所有的前驱结点状态为CANCELLED的都移除
        do {
            node.prev = pred = pred.prev; //将node的前节点pred的前节点赋值给node的前节点，pred从中删除，因为pred.waitStatus > 0
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        //CAS操作将这个前驱节点设置成SIGHNAL。
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```
这个方法第一次会将前驱节点的waitStatus改为-1，第二次循环使，前驱结点已经是-1了，所以会返回true,然后acquireQueued方法中就会接着执行parkAndCheckInterrupt将自己park阻塞挂起。
**todo**
```
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```
8. 如果没能成功获取锁，该node将取消获取锁
```
private void cancelAcquire(Node node) {
   if (node == null)
       return;
   node.thread = null;
   // 遍历并更新节点前驱，把node的prev指向前部第一个非取消节点。
   Node pred = node.prev;
   while (pred.waitStatus > 0)
       node.prev = pred = pred.prev;

   // 记录pred节点的后继为predNext，后续CAS会用到。
   Node predNext = pred.next;

   // 直接把当前节点的等待状态置为取消,后继节点即便也在cancel可以跨越node节点。
   node.waitStatus = Node.CANCELLED;

   /*
    * 如果CAS将tail从node置为pred节点了
    * 则剩下要做的事情就是尝试用CAS将pred节点的next更新为null以彻底切断pred和node的联系。
    * 这样一来就断开了pred与pred的所有后继节点，这些节点由于变得不可达，最终会被回收掉。
    * 由于node没有后继节点，所以这种情况到这里整个cancel就算是处理完毕了。
    *
    * 这里的CAS更新pred的next即使失败了也没关系，说明有其它新入队线程或者其它取消线程更新掉了。
    */
   if (node == tail && compareAndSetTail(node, pred)) {
       compareAndSetNext(pred, predNext, null);
   } else {
       // 如果node还有后继节点，这种情况要做的事情是把pred和后继非取消节点拼起来。
       int ws;
       if (pred != head &&
           ((ws = pred.waitStatus) == Node.SIGNAL ||
            (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
           pred.thread != null) {
           Node next = node.next;
           /* 
            * 如果node的后继节点next非取消状态的话，则用CAS尝试把pred的后继置为node的后继节点
            * 这里if条件为false或者CAS失败都没关系，这说明可能有多个线程在取消，总归会有一个能成功的。
            */
           if (next != null && next.waitStatus <= 0)
               compareAndSetNext(pred, predNext, next);
       } else {
           /*
            * 这时说明pred == head或者pred状态取消或者pred.thread == null
            * 在这些情况下为了保证队列的活跃性，需要去唤醒一次后继线程。
            * 举例来说pred == head完全有可能实际上目前已经没有线程持有锁了，
            * 自然就不会有释放锁唤醒后继的动作。如果不唤醒后继，队列就挂掉了。
            * 
            * 这种情况下看似由于没有更新pred的next的操作，队列中可能会留有一大把的取消节点。
            * 实际上不要紧，因为后继线程唤醒之后会走一次试获取锁的过程，
            * 失败的话会走到shouldParkAfterFailedAcquire的逻辑。
            * 那里面的if中有处理前驱节点如果为取消则维护pred/next,踢掉这些取消节点的逻辑。
            */
           unparkSuccessor(node);
       }
       
       /*
        * 取消节点的next之所以设置为自己本身而不是null,
        * 是为了方便AQS中Condition部分的isOnSyncQueue方法,
        * 判断一个原先属于条件队列的节点是否转移到了同步队列。
        *
        * 因为同步队列中会用到节点的next域，取消节点的next也有值的话，
        * 可以断言next域有值的节点一定在同步队列上。
        *
        * 在GC层面，和设置为null具有相同的效果。
        */
       node.next = node; 
   }
}

/**
 * 唤醒后继线程。
 */
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    // 尝试将node的等待状态置为0,这样的话,后继争用线程可以有机会再尝试获取一次锁。
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    /*
     * 这里的逻辑就是如果node.next存在并且状态不为取消（就是waitStatus！=1），则直接唤醒s即可
     * 否则需要从tail开始向前找到node之后最近的非取消节点。
     *
     * 这里为什么要从tail开始向前查找也是值得琢磨的:
     * 如果读到s == null，不代表node就为tail，参考addWaiter以及enq函数中的我的注释。
     * 不妨考虑到如下场景：
     * 1. node某时刻为tail
     * 2. 有新线程通过addWaiter中的if分支或者enq方法添加自己
     * 3. compareAndSetTail成功
     * 4. 此时这里的Node s = node.next读出来s == null，但事实上node已经不是tail，它有后继了!
     */
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
9. 这时加入线程3竞争state,梳理整个流程
- 首先调用ReentrantLock.lock()方法尝试加锁
- 因为是非公平锁，所以就会转到NoFairSync.Lock(()方法
- 在NoFairSync.Lock()方法中，会首先尝试设置state的值，因为已经被占有那么肯定是失败的。这时候会调用AQS.acquire(1).
- 在AQS的模板方法acquire(1)中，实际首先会调用的是子类的tryAcquire()方法，而在非公平锁的实现中即Sync.nofairTryAcquire()方法。
- 显然tryAcquire()会返回false，所以acquire()继续执行，即调用AQS.addWaiter()，就会将当前线程构造称为一个Node结点,初始状况下waitStatus为0。
- 在addWaiter方法中，会首先尝试直接将构建的node结点以CAS的方式(存在多个线程尝试将自己设置为tail)设置为tail结点，如果设置成功就直接返回，失败的话就会进入一个自旋循环的过程。即调用enq()方法。最终保证自己成功被添加到同步队列中。
- 加入同步队列之后，就需要将自己挂起或者嗅探自己的前驱结点是否为头结点以便尝试获取同步状态。即调用acquireQueued()方法。
- 在这里thread3的前驱结点不是head结点，所以就直接调用shouldParkAfterFailedAcquire()方法，该方法首先会将刚刚的thread2线程结点中的waitStatue的值改变为-1(初始的时候是没有改变这个waitStatus的，每个新节点的添加就会改变前驱结点的waitStatus值)。
- thread2所在结点的waitStatus改变后，shouldParkAfterFailedAcquire方法会返回false。所以之后还会在acquireQueued中进行第二次循环。并再次调用shouldParkAfterFailedAcquire方法，然后返回true。最终调用parkAndCheckInterrupt()将自己挂起。

## 12.3.2 释放同步状态

1. 释放同步状态，实际上是调用了AQS中的release()方法
```
public void unlock() {
    sync.release(1); //这里ReentrantLock的unlock方法调用了AQS的release方法
}
public final boolean release(int arg) {
    //这里调用了子类的tryRelease方法，即ReentrantLock的内部类Sync的tryRelease方法
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
2. release方法首先会调用ReentrantLock的内部类Sync的tryRelease方法。
```
protected final boolean tryRelease(int releases) {
    //（1）获取当前的state，然后减1，得到要更新的state
    int c = getState() - releases;
    //（2）判断当前调用的线程是不是持有锁的线程，如果不是抛出IllegalMonitorStateException
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    //（3）判断更新后的state是不是0
    if (c == 0) {
        free = true;
        //（3-1）将当前锁持者设为null
        setExclusiveOwnerThread(null);
    }
    //（4）设置当前state=c=getState()-releases
    setState(c);
    //（5）只有state==0，才会返回true
    return free;
}
```
3. 当tryReleasea返回true后，会继续执行if语句块
```
if (tryRelease(arg)) {
    //（1）获取当前队列的头节点head
    Node h = head;
    //（2）判断头节点不为null，并且头结点的waitStatus不为0(CACCELLED)
    if (h != null && h.waitStatus != 0)
        //（3-1）调用下面的方法唤醒同步队列head结点的后继结点中的线程
        unparkSuccessor(h);
    return true;
}
```
4. 执行unparkSuccessor
```
private void unparkSuccessor(Node node) {
    //（1）获得node的waitStatus
    int ws = node.waitStatus;
    //（2）判断waitStatus是否小于0
    if (ws < 0)
        //（2-1）如果waitStatus小于0需要将其以CAS的方式设置为0
        compareAndSetWaitStatus(node, ws, 0);

    //（2）获得s的后继结点，这里即head的后继结点
    Node s = node.next;
    //（3）判断后继结点是否已经被移除，或者其waitStatus==CANCELLED
    if (s == null || s.waitStatus > 0) {
        //（3-1）如果s！=null，但是其waitStatus=CANCELLED需要将其设置为null
        s = null;
        //（3-2）会从尾部结点开始寻找，找到离head最近的不为null并且node.waitStatus的结点小于0的
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //（4）node.next!=null或者找到的一个离head最近的结点不为null
    if (s != null)
        //（4-1）唤醒这个结点中的线程
        LockSupport.unpark(s.thread);
}
```
- 从上面的代码可以总结：unparkSuccessor主要做了两件事
    + 获取head节点的waitStatus，如果小于0，就通过CAS操作将其修改为0.
    + 寻找head节点的下一个节点，如果这个节点的waitStatus小于0，就唤醒这个节点，否则遍历下去，找到第一个waitStatus<=0的节点，并唤醒。
5. 下面分析释放掉state之后，唤醒同步队列中的节点之后程序如何运行。
- 线程1（获取到锁的线程）调用unlock方法之后，最终执行到unparkSuccessor方法会唤醒线程2节点。所以线程2倍unpark.
- 当时线程2在调用acquireQueued方法之后继续执行acquireAndCheckInterrupt里面被park阻塞挂起了，所以线程2倍唤醒之后继续执行acquireQueued方法中的for循环。
- for循环中的第一件事是查看自己的前驱节点是不是头结点（应该是满足的）；
- 前驱节点是head节点，就会调用tryAcquire方法查实获取state,因为线程1已经释放了state,即state=0,所以线程2调用tryAcquire方法的时候，以CAS方式将state从0更新为1是成功的，线程2获取了锁。
- 线程2获取state成功，就会从acquireQueued方法中退出。注意这时候的acquireQueued返回值为false，所以在AQS的模板方法的acquire中会直接从if条件退出，最后执行自己锁住的代码块中的程序。


## 12.3.3 公平锁和非公平锁的区别
1. 非公平锁在调用lock后，首先就会调用CAS进行一次抢锁，如果这个时候锁没有被占用，那么直接就获取到锁返回了。
2. 非公平锁在CAS失败后，和公平锁一样都会进入到tryAcquire方法，在tryAcquire方法中，如果发现锁这个时候被释放了（state==0）,非公平锁会直接CAS抢锁，但是公平锁会判断等待队列是否有线程处于等待状态，如果有则不去抢锁，拍到后面。
3. 公平锁和非公平锁就这两点区别，如果两次CAS都不成功，那么后面非公平锁和公平锁是一样的。
# 12.4 共享锁的实现

共享模式下，同一时刻会有多个线程获取共享同步状态。
使用tryAcquireShared方法时注意
- 返回负数表示获取失败
- 返回0表示成功，但是后继争用线程不会成功
- 返回正数表示获取成功，并且后继争用线程可也能成功。

## 12.4.1 获取共享锁的实现
```
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                // 一旦共享获取成功，设置新的头结点，并且唤醒后继线程
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

/**
 * 这个函数做的事情有两件:
 * 1. 在获取共享锁成功后，设置head节点
 * 2. 根据调用tryAcquireShared返回的状态以及节点本身的等待状态来判断是否要需要唤醒后继线程。
 */
private void setHeadAndPropagate(Node node, int propagate) {
    // 把当前的head封闭在方法栈上，用以下面的条件检查。
    Node h = head;
    setHead(node);
    /*
     * propagate是tryAcquireShared的返回值，这是决定是否传播唤醒的依据之一。
     * h.waitStatus为SIGNAL或者PROPAGATE时也根据node的下一个节点共享来决定是否传播唤醒，
     * 这里为什么不能只用propagate > 0来决定是否可以传播在本文下面的思考问题中有相关讲述。
     */
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}

/**
 * 这是共享锁中的核心唤醒函数，主要做的事情就是唤醒下一个线程或者设置传播状态。
 * 后继线程被唤醒后，会尝试获取共享锁，如果成功之后，则又会调用setHeadAndPropagate,将唤醒传播下去。
 * 这个函数的作用是保障在acquire和release存在竞争的情况下，保证队列中处于等待状态的节点能够有办法被唤醒。
 */
private void doReleaseShared() {
    /*
     * 以下的循环做的事情就是，在队列存在后继线程的情况下，唤醒后继线程；
     * 或者由于多线程同时释放共享锁由于处在中间过程，读到head节点等待状态为0的情况下，
     * 虽然不能unparkSuccessor，但为了保证唤醒能够正确稳固传递下去，设置节点状态为PROPAGATE。
     * 这样的话获取锁的线程在执行setHeadAndPropagate时可以读到PROPAGATE，从而由获取锁的线程去释放后继等待线程。
     */
    for (;;) {
        Node h = head;
        // 如果队列中存在后继线程。
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h);
            }
            // 如果h节点的状态为0，需要设置为PROPAGATE用以保证唤醒的传播。
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        // 检查h是否仍然是head，如果不是的话需要再进行循环。
        if (h == head)
            break;
    }
}
```
## 12.4.2 释放共享锁

```
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        // doReleaseShared的实现上面获取共享锁已经介绍
        doReleaseShared();
        return true;
    }
    return false;
}
```
## 12.5 AQS中的条件队列

ConditionObject是AQS中定义的内部类，实现了Condition接口，ConditionObject是基于Lock实现的，在其内部通过链表维护等待队列（条件队列）。

Conditon必须在lock的同步控制块中使用，调用Condition的signal方法并不代表线程就可以马上执行，signal方法的作用是将线程所在的节点从等待队列中移除，然后加入到同步队列中，线程的执行始终都需要根据同步状态（即线程是否站有锁）。

### 12.5.1 与Lock之间的实现关系

在ReentrantLock中创建
```
public Condition newCondition() {
    return sync.newCondition();
}
```
sync中的实现
```
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;
    final ConditionObject newCondition() {
        return new ConditionObject();
    }
}
```
### 12.5.2 ConditionObject

两个成员变量
```
//条件（等待）队列的第一个节点
private transient Node firstWaiter;
//条件（等待）队列的最后一个节点
private transient Node lastWaiter;
```

### 12.5.3 await()

```
public final void await() throws InterruptedException {
     //当前线程被中断则抛异常
     if (Thread.interrupted()) throw new InterruptedException();
     //添加进条件队列
     Node node = addConditionWaiter();
     //释放当前线程持有的锁
     int savedState = fullyRelease(node);
     int interruptMode = 0;
     //在这里一直查找创建的Node节点在不在Sync队列，不在就一直禁用当前线程
     while (!isOnSyncQueue(node)) {
         //park当前线程，直到唤醒
         LockSupport.park(this);
         //如被中断也跳出循环
         if ((interruptMode = checkInterruptWhileWaiting(node)) != 0) 
         break;
     }
     //此时已被唤醒，获取之前被释放的锁，
     if (acquireQueued(node, savedState) && interruptMode != THROW_IE)   
       interruptMode = REINTERRUPT;
     //移除节点，释放内存
     if (node.nextWaiter != null) // clean up if cancelled
       unlinkCancelledWaiters();
     //被中断后的操作
     if (interruptMode != 0)
       reportInterruptAfterWait(interruptMode);
}
```
本质上来说，await()方法就是为当前线程创建一个Node节点，加入到Condition队列并释放锁，之后就一直查看这个节点是否在Sync队列中（signal()方法将它移到Sync队列），如果在的话就唤醒此线程，重新获取锁。
```
/**
 *  添加一个Node节点到Condition队列中
 */
private Node addConditionWaiter() {
  Node t = lastWaiter;
  //如果尾节点被取消，就清理掉
  if (t != null && t.waitStatus != Node.CONDITION) {
      unlinkCancelledWaiters();
      t = lastWaiter;
  }
  //新建状态为的CONDITION节点，并添加在尾部
  Node node = new Node(Thread.currentThread(), Node.CONDITION);
  if (t == null)
      firstWaiter = node;
  else
      t.nextWaiter = node;
  lastWaiter = node;
  return node;
}
/**
*  释放当前线程的state，实际还是调用tryRelease方法。考虑一下这里的 savedState。

如果在 condition1.await() 之前，假设线程先执行了 2 次 lock() 操作，那么 state 为 2，

我们理解为该线程持有 2 把锁，这里 await() 方法必须将 state 设置为 0，

然后再进入挂起状态，这样其他线程才能持有锁。当它被唤醒的时候，它需要重新持有 2 把锁，才能继续下去。
*/
// 首先，我们要先观察到返回值 savedState 代表 release 之前的 state 值
// 对于最简单的操作：先 lock.lock()，然后 condition1.await()。
//         那么 state 经过这个方法由 1 变为 0，锁释放，此方法返回 1
//         相应的，如果 lock 重入了 n 次，savedState == n
// 如果这个方法失败，会将节点设置为"取消"状态，并抛出异常 IllegalMonitorStateException
final int fullyRelease(Node node) {
  boolean failed = true;
    try {
       int savedState = getState();
       if (release(savedState)) {
           failed = false;
           return savedState;
       } else {
           throw new IllegalMonitorStateException();
       }
     } finally {
        if (failed)
           node.waitStatus = Node.CANCELLED;
     }
}

/*
考虑一下，如果一个线程在不持有 lock 的基础上，就去调用 condition1.await() 方法，它能进入条件队列，但是在上面的这个方法中，由于它不持有锁，release(savedState) 这个方法肯定要返回 false，进入到异常分支，然后进入 finally 块设置 node.waitStatus = Node.CANCELLED，这个已经入队的节点之后会被后继的节点”请出去“。
*/


/**
*  检查当前节点在不在Sync队列
*/
final boolean isOnSyncQueue(Node node) {
    //如果当前节点状态为CONDITION，一定还在Condition队列
    //如果Sync队列的前置节点为null，则表明当前节点一定还在Condition队列
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    //有后继节点，也有前置节点，那么一定在Sync队列
    if (node.next != null) // If has successor, it must be on queue
        return true;
    //倒查Node节点，前置节点不能为null，第一个if已经做了判断，其前置节点为non-null,但是当前节点也不在Sync也是可能的,因为CAS操作将其加入队列也可能失败，所以我们需要从尾部开始遍历确保其在队列
    return findNodeFromTail(node);

    // 下面这个方法从阻塞队列的队尾开始从后往前遍历找，如果找到相等的，说明在阻塞队列，否则就是不在阻塞队列

    // 可以通过判断 node.prev() != null 来推断出 node 在阻塞队列吗？答案是：不能。
    // 这个可以看上篇 AQS 的入队方法，首先设置的是 node.prev 指向 tail，
    // 然后是 CAS 操作将自己设置为新的 tail，可是这次的 CAS 是可能失败的。
}
```
### 12.5.4 signal()将等待时间最长的线程（如果存在），从Condition队列中移动到拥有锁的Sync队列

```
public final void signal() {
     //当前线程非独占线程，报非法监视器状态异常
     if (!isHeldExclusively())
         throw new IllegalMonitorStateException();
     //头节点是等待时间最长的节点
     Node first = firstWaiter;
     if (first != null)
         doSignal(first);
}
private void doSignal(Node first) {
      do {
          //如果头节点的下一节点为null，则将Condition的lastWaiter置为null
          if ( (firstWaiter = first.nextWaiter) == null)
              lastWaiter = null;
          //将头结点的下一个节点设为null
          first.nextWaiter = null;
          //被唤醒并且头节点不为null则结束循环
      } while (!transferForSignal(first) &&
              (first = firstWaiter) != null);
 }
final boolean transferForSignal(Node node) {
    //如果无法改变节点状态，说明节点已经被唤醒
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
    //将当前节点添加到Sync队列尾部，并设置前置节点的waitStatus为SIGNAL，表明后继有节点（可能）将被唤醒，如果取消或者设置waitStatus失败，会唤醒重新同步操作，这时候waitStatus是瞬时的，出现错误也是无妨的
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}

```
### 12.5.6 singalAll将所有线程从Condition队列移动到拥有锁Sync队列中

```
public final void signalAll() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignalAll(first);
}
/**
 * 遍历所有节点，加入到拥有锁的Sync队列
*/
private void doSignalAll(Node first) {
    lastWaiter = firstWaiter = null;
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;
        transferForSignal(first);
        first = next;
    } while (first != null);
}
```
### 12.5.7 interruptMode
1. interruptMode 可以取值为 REINTERRUPT（1），THROW_IE（-1），0
- REINTERRUPT： 代表 await 返回的时候，需要重新设置中断状态
- THROW_IE： 代表 await 返回的时候，需要抛出 InterruptedException 异常
- 0 ：说明在 await 期间，没有发生中断
2. 有以下三种情况会让 LockSupport.park(this); 这句返回继续往下执行：
- 常规路径。signal -> 转移节点到阻塞队列 -> 获取了锁（unpark）
- 线程中断。在 park 的时候，另外一个线程对这个线程进行了中断
- signal 的时候我们说过，转移以后的前驱节点取消了，或者对前驱节点的CAS操作失败了
- 假唤醒。这个也是存在的，和 Object.wait() 类似，都有这个问题
3. 线程唤醒后第一步是调用 checkInterruptWhileWaiting(node) 这个方法，此方法用于判断是否在线程挂起期间发生了中断，如果发生了中断，是 signal 调用之前中断的，还是 signal 之后发生的中断。
```
// 1\. 如果在 signal 之前已经中断，返回 THROW_IE
// 2\. 如果是 signal 之后中断，返回 REINTERRUPT
// 3\. 没有发生中断，返回 0
private int checkInterruptWhileWaiting(Node node) {
    return Thread.interrupted() ?
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
}
```
> Thread.interrupted()：如果当前线程已经处于中断状态，那么该方法返回 true，同时将中断状态重置为 false，所以，才有后续的 重新中断（REINTERRUPT） 的使用。

4. 怎么判断是 signal 之前还是之后发生的中断：
```
// 只有线程处于中断状态，才会调用此方法
// 如果需要的话，将这个已经取消等待的节点转移到阻塞队列
// 返回 true：如果此线程在 signal 之前被取消，
final boolean transferAfterCancelledWait(Node node) {
    // 用 CAS 将节点状态设置为 0 
    // 如果这步 CAS 成功，说明是 signal 方法之前发生的中断，因为如果 signal 先发生的话，signal 中会将 waitStatus 设置为 0
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        // 将节点放入阻塞队列
        // 这里我们看到，即使中断了，依然会转移到阻塞队列
        enq(node);
        return true;
    }

    // 到这里是因为 CAS 失败，肯定是因为 signal 方法已经将 waitStatus 设置为了 0
    // signal 方法会将节点转移到阻塞队列，但是可能还没完成，这边自旋等待其完成
    // 当然，这种事情还是比较少的吧：signal 调用之后，没完成转移之前，发生了中断
    while (!isOnSyncQueue(node))
        Thread.yield();
    return false;
}
```
> 即使发生了中断，节点依然会转移到阻塞队列。

- lock方法不会响应中断
- lockInterruptibly会抛出InterruptedException异常
