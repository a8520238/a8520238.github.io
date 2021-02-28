---
title: unsafe
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 17.1 Unsafe
> Unsafe 在Netty、Cassandra、Hadoop、Kafka中均有应用
- Unsafe 赋予java直接操控内存指针的能力，但只有主类加载器加载的类才能调用这个方法
```
1 public static Unsafe getUnsafe() {
2     Class var0 = Reflection.getCallerClass();
3     if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
4         throw new SecurityException("Unsafe");
5     } else {
6         return theUnsafe;
7     }
8 }
```

也可以通过手动设置bootclasspath参数来使用
```
1 Field f = Unsafe.class.getDeclaredField("theUnsafe");
2 f.setAccessible(true);
3 Unsafe unsafe = (Unsafe) f.get(null);
```
## 17.1.1 Unsafe简介
1. 内存管理。包括分配内存、释放内存等
该部分包括了allocateMemory（分配内存）、reallocateMemory（重新分配内存）、copyMemory（拷贝内存）、freeMemory（释放内存 ）、getAddress（获取内存地址）、addressSize、pageSize、getInt（获取内存地址指向的整数）、getIntVolatile（获取内存地址指向的整数，并支持volatile语义）、putInt（将整数写入指定内存地址）、putIntVolatile（将整数写入指定内存地址，并支持volatile语义）、putOrderedInt（将整数写入指定内存地址、有序或者有延迟的方法）等方法。getXXX和putXXX包含了各种基本类型的操作。
<br>利用copyMemory方法，我们可以实现一个通用的对象拷贝方法，无需再对每一个对象都实现clone方法，当然这通用的方法只能做到对象浅拷
2. 非常规的对象实例化
allocateInstance()方法提供了另一种创建实例的途径。通常我们可以用new或者反射来实例化对象，使用allocateInstance()方法可以直接生成对象，且无需调用构造方法和其他初始化方法。
3. 操作类、对象、变量
<br>包括了staticFieldOffset（静态域偏移）、defineClass（定义类）、defineAnonymousClass(定义匿名类)、ensureClassInitialized（确保类初始化）、objectFieldOffset（对象域偏移）等方法。
<br>通过上述方法我们可以获取对象的指针，通过对指针进行偏移，我们不仅可以直接修改指针指向的数据（即使是私有的），甚至可以获得JVM已经定为垃圾、可以进行回收的对象。
4. 数组操作
- arrayBaseOffset: 获取数组第一个元素的偏移地址
- arrayIndexScale: 获取数组中元素的增量地址
5. 多线程同步、包括锁机制、CAS操作
- 这部分包括了monitorEnter、tryMonitorEnter、monitorExit、compareAndSwapInt、compareAndSwap等方法。
- 其中monitorEnter、tryMonitorEnter、monitorExit已经被标记为deprecated，不建议使用。
- Unsafe类的CAS操作可能是用的最多的，它为Java的锁机制提供了一种新的解决办法，比如AtomicInteger等类都是通过该方法来实现的。compareAndSwap方法是原子的，可以避免繁重的锁机制，提高代码效率。这是一种乐观锁，通常认为在大部分情况下不出现竞态条件，如果操作失败，会不断重试直到成功
6. 挂起与恢复
- park 和 unpark
- LockSupport类中的park方法，最终调用的是Unsafe.park()方法
7. 内存屏障
- JAVA8引入了loadFence、storeFence、fullFence等方法，用于定义内存屏障，避免代码重排序
- loadFence表示该方法之前的所有load操作在内存屏障之前完成
- storeFence表示该方法之前的所有store操作在内存屏障之前完成
- fullFence表示该方法按之前的所有load、store操作在内存屏障之前完成

8. Unsafe
- Unsafe是java设计时留下的一个后门，提供了一些可以直接操控内存和线程的低层次操作。

# 17.2 LockSupport
1. LockSupport 中有基本的线程阻塞原语，可以做到join()、wait()/notifyAll()功能一样，使线程自由的阻塞、释放。
2. LockSupport提供park()和unpark()方法实现阻塞线程和解除线程阻塞，LockSupport和每个使用它的线程都与一个许可(permit)关联。
- permit相当于是1，0的开关，默认是0，调用一次unpark就加1变成1，调用一次park会消费permit，也就会将1变成0，同时park立即返回。再次调用park会变成block(因为permit为0了，会阻塞在这里，直到permit变为1)，这时调用unpark会把permit置为1.
- 下面举例说明：
```
public class LockSupportTest {
    public static void main(String[] args) throws InterruptedException {
      //买锅
//      Thread t1 = new Thread(new BuyGuo(Thread.currentThread()));
//      t1.start();

      //买菜
        Thread t2 = new Thread(new BuyCai(Thread.currentThread()));
        t2.start();
//        LockSupport.park();
//        System.out.println("锅买回来了...");
        LockSupport.park();
        System.out.println("菜买回来 了...");
        System.out.println("开始做饭");
    }
}
class BuyGuo implements Runnable{
    private Object threadObj;
    public BuyGuo(Object threadObj) {
        this.threadObj = threadObj;
    }

    @Override
    public void run() {
        System.out.println("去买锅...");
        LockSupport.unpark((Thread)threadObj);

    }
}
class BuyCai implements Runnable{
    private Object threadObj;
    public BuyCai(Object threadObj) {
        this.threadObj = threadObj;
    }

    @Override
    public void run() {
        System.out.println("买菜去...");
        LockSupport.unpark((Thread)threadObj);
    }
}
```
main方法启动后，主线程和买菜线程同时开始执行。因为两者同时进行，通过permit进行控制。
- 这里很重要的一点，要将park()与unpark()配对使用才高效。
```
//买锅
  Thread t1 = new Thread(new BuyGuo(Thread.currentThread()));
   t1.start();
  //买菜
    Thread t2 = new Thread(new BuyCai(Thread.currentThread()));
    t2.start();
    LockSupport.park();
    System.out.println("锅买回来了...");
    LockSupport.park();
    System.out.println("菜买回来了...");
    System.out.println("开始做饭");
```
- 调用两次park()和unpark()，发现有时候可以，有时候会使线程卡在那里，然后我又换了下顺序
```
 //买锅
  Thread t1 = new Thread(new BuyGuo(Thread.currentThread()));
   t1.start();
      LockSupport.park();
    System.out.println("锅买回来了...");
  //买菜
    Thread t2 = new Thread(new BuyCai(Thread.currentThread()));
    t2.start();
    LockSupport.park();
    System.out.println("菜买回来了...");
    System.out.println("开始做饭");
```
park()和unpark()既然是成对配合使用，通过标识permit来控制，可能会出现阻塞的情况原因。

当买锅的时候，通过unpark()将permit置为1，但是还没等到外边的main方法执行第一个park()，买菜的线程又调了一次unpark()，但是这时候permit还是从1变成了1，等回到主线程调用park()的时候，因为还有两个park()需要执行，也就是需要执行，也就是需要两个消费permit，因为permit只有1个人，所以，可能会剩下一个park()卡在那里了。

3. 与wait()/notifyAll()的比较LockSupport的park/unpark方法，虽然与平时object中wait/notify同样达到阻塞线程的效果。

面向的对象主体不同。
- LockSupport()操作的线程对象，直接传入的就是Thread，而park/unpark不需要获取对象的监视器。
- wait()属于具体对象，notifyAll()也是针对所有线程进行唤醒。wait/notify需要获取对象的监视器，即synchronized修饰
- 也就是说LockSupport阻塞的线程，notify/notifyAll没法唤醒。但是park之后，同样可以被中断(interrupt())