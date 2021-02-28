---
title: java关键字
date: 2021-2-28
cover:
top_img:
categories: 
    - java
    - javaSE
tags: 
mathjax: true
katex: true
---
# 1 compareTo

两个字符串比较字典序，str1.compareTo(str2)
如果str比str2大，输出正数，反之负数，相等为0.

# 2 final

对于final，编译期间直接进行了常量替换（而对于非final字段则是在运行期进行赋值处理的）。下面是例子
```
final String str1=”ja”;
final String str2=”va”;
String str3=str1+str2;
```
在编译时，直接替换成了String str3=”ja”+”va”，根据第三条规则，再次替换成String str3=”JAVA”.

# 3 transient

> Java语言的关键字，变量修饰符，如果用transient声明一个实例变量，当对象存储时，它的值不需要维持。这里的对象存储是指，Java的serialization提供的一种持久化对象实例的机制。当一个对象被序列化的时候，transient型变量的值不包括在序列化的表示中，然而非transient型的变量是被包括进去的。使用情况是：当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。

简单点说，就是被transient修饰的成员变量，在序列化的时候其值会被忽略，在被反序列化后， transient 变量的值被设为初始值， 如 int 型的是 0，对象型的是 null。

- ArrayList使用transient关键字
- 但是ArrayList是支持序列化的，通过调用方法 writeObject() 和 readObject() 完成。
- 在 writeObject() 方法中，for循环按需序列化，用了几个下标序列化几个对象。读取的时候也是一样的，有几个读几个，开发人员真是优化到极致！

# 4 instanceof 

instanceof 是 Java 的一个二元操作符，类似于 ==，>，< 等操作符。

instanceof 是 Java 的保留关键字。它的作用是测试它左边的对象是否是它右边的类的实例，返回 boolean 的数据类型。

以下实例创建了 displayObjectClass() 方法来演示 Java instanceof 关键字用法：

```
public static void displayObjectClass(Object o) {
  if (o instanceof Vector)
     System.out.println("对象是 java.util.Vector 类的实例");
  else if (o instanceof ArrayList)
     System.out.println("对象是 java.util.ArrayList 类的实例");
  else
    System.out.println("对象是 " + o.getClass() + " 类的实例");
}
```

# 5 volatile

volatile通常被比喻成"轻量级的synchronized"，也是Java并发编程中比较重要的一个关键字。和synchronized不同，volatile是一个变量修饰符，只能用来修饰变量。无法修饰方法及代码块等。

volatile的用法比较简单，只需要在声明一个可能被多线程同时访问的变量时，使用volatile修饰就可以了。

```
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}  
```

如以上代码，是一个比较典型的使用双重锁校验的形式实现单例的，其中使用volatile关键字修饰可能被多个线程同时访问到的singleton。

## 5.1 volatile的原理

对于volatile变量，当对volatile变量进行写操作的时候，JVM会向处理器发送一条lock前缀的指令，将这个缓存中的变量回写到系统主存中。

但是写到了内存，其他处理器缓存（|线程私有内存）中的值还是旧的，再执行计算操作会有问题，所以在多处理器下，为保证各个处理器的缓存是一致的，就会实现缓存一致性协议。

缓存一致性协议：每个处理器通过探嗅在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。

如果一个变量被volatile所修饰的话，在每次数据变化之后，其值都会被强制刷入主存。而其他处理器的缓存由于遵守了缓存一致性协议，也会把这个变量的值从主存加载到自己的缓存中。这就保证了一个volatile在并发编程中，其值在多个缓存中是可见的。

## 5.2 volatile与可见性（缓存一致性协议保证）

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

Java内存模型规定了都有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的 工作内存和主存之间进行数据同步。所以，就可能出现线程1改了某个变量的值就，但是线程2不可见的情况。

volatile修饰的变化量可以立即同步到主内存，被其修饰的变量在每次使用之前都从主内存刷新。因此，可以使用volatile来保证多线程操作时变量的可见性。

## 5.3 volatile与有序性（内存屏障保证）

有序性即程序执行的顺序按照代码的先后顺序执行。

除了引入了时间片以外，由于处理器优化和指令重排等，CPU还可能对输入代码进行乱序执行，比如load->add->save有可能被优化成load->save->add。这就是可能存在有序性问题。

而volatile除了可以保证数据的可见性之外，还有一个强大的功能，那就是它可以进制指令重排优化等。

普通的变量仅仅会保证在该方法的执行过程中所依赖的赋值结果的地方都能获得正确的结果，而不能保证变量的赋值操作的顺序与程序代码中的执行顺序一致。

volatile可以禁止指令重排，这就保证了代码的先后顺序执行。这就保证了有序性。被volatile修饰的变量的操作，会严格按照代码顺序执行，load->add->save的执行顺序就是：load、add、save。

## 5.4 volatile与原子性

原子性是指一个操作是不可中断的，要全部执行完成，要不就都不执行。

线程是CPU调度的基本单位。CPU有时间片的概念，会根据不同的调度算法进行线程调度。当一个线程获得时间片之后开始执行，在时间片耗尽之后，就会失去CPU使用权。所以在多线程场景下，由于时间片在线程间切换，就会发生原子性问题。

在synchronized中，为了保证原子性，需要通过字节码指令monitorenter和monitorexit，但是volatile和这两个指令之间是没有任何关系的。

所以volatile是不能保证原子性的。

**所以在以下两个场景中可以使用volatile来代替synchronized:**
1. 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程会修改变量的值。
2. 变量不需要与其他状态变量共同参与不变约束。

除以上场景外，都需要使用其他方式来保证原子性，如synchronized或者concurrent包。

volatile和原子性的例子：
```
public class Test {
    public volatile int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }

        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```
以上代码比较简单，就是创建10个线程，然后分别执行1000次i++操作。正常情况下，程序的输出结果应该是10000，但是，多次执行的结果都小于10000。这其实就是volatile无法满足原子性的原因。

为什么会出现这种情况呢，那就是因为虽然volatile可以保证inc在多个线程之间的可见性。但是无法inc++的原子性。

# 6 synchronized

synchronized在需要原子性、可见性和有序性这三种特性的时候都可以作为其中一种解决方案。

## 6.1 synchronized的用法

synchronized既可以修饰方法也可以修饰代码块。其修饰的区域，在同一时间只能被单个线程访问。

```
public class SynchronizedDemo {
     //同步方法
    public synchronized void doSth(){
        System.out.println("Hello World");
    }

    //同步代码块
    public void doSth1(){
        synchronized (SynchronizedDemo.class){
            System.out.println("Hello World");
        }
    }
}
```
## 6.2 synchronized的实现原理

- 对于同步方法，JVM采用ACC_SYNCHRONIZED标记来实现同步。
    + 方法级的同步是隐式的。同步方法的常量池中会有一个ACC_SYNCHRONIZED标志。当某个线程要访问某个方法的时候，会检查是否有ACC_SYNCHRONIZED，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放。
- 对于同步代码块。JVM采用monitorenter、monitorexit两个指令来实现同步。
    + 同步代码块使用monitorenter和monitorexit两个指令实现。可以把执行monitorenter指令理解为加锁，执行monitorexit理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行monitorenter）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行monitorexit指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。

ACC_SYNCHRONIZED和monitorenter、monitorexit都是基于Monitor实现的，在Java虚拟机（HotSpot）中，Monitor是基于C++实现的，由ObjectMonitor实现。

在该类中，有几个方法，如enter、exit、wait、notify、notifyAll。sychronized加锁的时候，会调用objectMonitor的enter方法，解锁的时候调用exit方法。

## 6.3 synchronized原子性

保证原子性的高级字节码指令：
- monitorenter
- monitorexit
这两个字节码对应的关键字：synchronized

保证原子性：执行monitorenter指令时，该线程的CPU时间片用完了，但是没有解锁。由于synchronized的锁是可重入的，下一个时间片还是只能被自己获得，继续执行代码。直到所有代码完成。

## 6.4 synchronized可见性

保证synchronized可见性：被其修饰的代码，在开始执行时会加锁，执行完之后会解锁。对一个变量解锁之前，必须把此变量同步回主存。这样解锁后，后续线程就可以访问到被修改后的变量。

## 6.5 synchronized有序性

> synchronized是无法禁止指令重排和处理器优化的。

synchronized保证有序性：
- 有序性概念扩展：Java中的天然有序性可以总结为：如果在本线程内观察，所有操作都是天然有序的。如果在一个线程中观察另一个线程，所有的操作都是无序的。
- as-if-serial语义的意思指：不管怎么重排序（编译器和处理器为了提高并行度），单线程程序的执行结果都不能被改变。编译器和处理器无论如何优化，都必须遵守as-if-serial语义。

所以，synchronized修饰额代码，同一时间只能被同一个线程访问，单线程执行，可以保证有序性。

## 6.6 synchronized锁优化

1. 在JDK1.6之前，synchronized的实现会直接调用ObjectMonitor的enter和exit，这种锁被称之为重量级锁。
2. 在JDK1.6中，对锁进行了很多优化，出现了轻量级锁，偏向锁，锁消除，锁消除，适应性自旋锁，锁粗化(自旋锁在1.4就有，只不过默认的是关闭的，jdk1.6是默认开启的)，进行了锁优化。

# 7 final

final表示被final修饰的部分是无法被修改的。
final可以定义：变量、方法、类。

## 7.1 final变量
如果将变量设置为final，则不能更改final变量的值（它将是常量）。无法被修改。

## 7.2 final方法
声明为final的方法，不能被覆盖，也就是子类不能够覆盖final修饰的方法。

## 7.3 final类

final类不能被继承

# 8 static

static表示静态的意思，用来修饰成员变量和成员方法，也可以形成static代码块。

1. 静态变量

声明该变量属于class对象

2. 静态方法
- 静态方法属于类而不是实例。
- java8开始也可以有接口类型的静态方法了。
3. 静态代码块
    - java的静态块是一组指令在类装载的时候在内存中由Java类加载器执行。
    - 静态块常用于初始化类的静态变量
    - Java不允许在静态块中使用非静态变量。静态块只在类加载时执行一次。
4. 静态类
    - Java可以嵌套使用静态类，但是静态类不能用于嵌套的顶层。
    - 静态嵌套类的使用与其他顶层类一样，嵌套只是为了便于项目打包。
    - static修饰的内部类可以直接做为一个普通类来使用，而不需要实例一个外部类。
    - 静态内部类跟静态方法一样，只能访问静态的成员变量和方法，不能访问非静态的方法和属性，但是普通内部类可以访问任意外部类的成员变量和方法
    - 静态内部类可以声明普通成员变量和方法，而普通内部类不能声明static成员变量和方法。
    - 静态内部类可以单独初始化:`Inner i = new Outer.Inner();`
    - 普通内部类初始化：`Outer o = new Outer();
Inner i = o.new Inner();`
    - 静态内部类使用场景一般是当外部类需要使用内部类，而内部类无需外部类资源，并且内部类可以单独创建的时候会考虑采用静态内部类的设计。


[静态内部类](https://blog.csdn.net/xihuanyuye/article/details/84713527)