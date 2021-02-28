---
title: CAS
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
## 10.0 CAS

1. 缓存行：Cache line 缓存的最小操作单位
2. 比较并交换：Compare and Swap, CAS操作需要输入两个数值，一个期望值和一个新值，只有当期望值与主存实际存在的旧值相同时，才能修改。

## 10.1 CAS如何保证个原子性

对如下代码：
```
if (this == expect) {
    this = update
    return true;
} else {
    return false;
}
```
比较this == expect，替换this = update，compareAndSwapInt如何这两个步骤的原子性呢？

### 10.1.1 CAS底层实现
1. Lock锁住总线，确保对内存的读写改操作原子执行时，其他处理器暂时无法访问总线。
2. 通过缓存锁定（利用cpu缓存一致性） 
- 缓存锁定,(解决多个cpu缓存同时修改主存中的同一个缓存行而引起的并发修改问题)
- 缓存一致性(MESI),（解决多个cpu缓存如何从主存同步数据的一致性问题），某个cpu写回数据后，会通知其他cpu中的缓存失效。

[CAS原理详解](https://blog.csdn.net/Hsuxu/article/details/9467651?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf)

## 10.2 volatile与cas区别与联系

- volatile不能保证原子性
- CAS指令在Intel CPU上称为CMPXCHG指令，它的作用是将指定内存地址的内容与所给的某个值相比，如果相等，则将其内容替换为指令中提供的新值，如果不相等，则更新失败。这一比较并交换的操作是原子的，不可以被中断
- 硬件特征上可以保证同一时刻只能有一个线程可修改同一个内存上的值。
##  几个问题
1. 既然cas保证了更新语句的原子性，volatile保证了各个CPU间的可见性，缓存锁定扮演了什么角色？
    + l3缓存锁是共享的
2. java5后提供了CAS特性保证了可以直接调用计算机底层的CAS特性，而不用手动实现。
3. 手动实现的CAS是否能保证原子性
```

public final int incrementAndGet() {    
 
    for (;;) {// 这样优于while(true)    
        int current = get();// 获取当前值    
        int next = current + 1;// 设置更新值    
        if (compareAndSet(current, next))    
            return next;    
    }    
} 

compareAndSwapObject源码如何提现调用了底层的CAS？
   1:  UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapObject(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jobject e_h, jobject x_h))
   2:    UnsafeWrapper("Unsafe_CompareAndSwapObject");
   3:    oop x = JNIHandles::resolve(x_h); //待更新的新值，也就是UpdateValue
   4:    oop e = JNIHandles::resolve(e_h); //期望值,也就是ExpectValue 
   5:    oop p = JNIHandles::resolve(obj); //待操作对象
   6:    HeapWord* addr = (HeapWord *)index_oop_from_field_offset_long(p, offset);//根据操作的对象和其在内存中的offset，计算出内存中具体位置
   7:    oop res = oopDesc::atomic_compare_exchange_oop(x, addr, e, true);// 如果操作对象中的值和e期望值一致，则更新存储值为x，反之不更新
   8:    jboolean success  = (res == e); 
   9:    if (success) //满足更新条件
  10:        update_barrier_set((void*)addr, x); // 更新存储值为x
  11:    return success;
  12:  UNSAFE_END
```

## 10.3 ABA问题
- 通过版本号解决

## 10.4 多变量问题
- CAS能完成对单个变量的原子操作，但不能保证多个变量同时操作的原子性
    + 可以通过把多个变量封装成引用类型，通过AtomicReference进行CAS操作
    + 或者使用锁，锁住同步代码块
