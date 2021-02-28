---
title: Map
date: 2021-2-28
cover:
top_img:
categories: 
    - java
    - java集合框架
tags: 
mathjax: true
katex: true
---
# 1 HashMap和HashTable不同点
1. HashTable继承于Dictionary类，HashMap继承于AbstractMap。AbstractMap中提供的基础方法更多，并且实现了多个通用的方法，而在Dictionary中只有少量的接口，并且都是abstract类型。
2. HashMap可以允许存在一个为null的key和任意个为null的value,但是Hashtable中的key和value都不允许为null。当HashMap遇到为null的key时，会调用putForNullKey方法进行处理，对value没有处理，只要是对象就可以。而HashTable遇到null时，会直接抛出NullPointerException异常信息。
3. Hashtable是同步的， HashMap不是，Collections类中存在一个静态方法：synchronizedMap()，该方法也是一个线程安全的Map对象。HashMap内部没有实现任何线程同步的代码，性能相对要好，HashTable大部分对外的接口都使用synchronized包裹，所以是线程安全的，但是性能会差。
4. 算法不一样，HashMap的initialCapacity为16，而Hashtable的initialCapacity为11。HashMap中的初始容量必须是2的幂，如果初始化传入的initialCapacity不是2的幂，会自动调整为大于初始的initialCapacity最小的2的幂。HashMap使用自己的计算hash的方法（会依赖key的hashCode方法），HashTable则直接使用key的hashCode方法得到。


# 2 LinkedHashMap
- LinkedHashMap可以看做是HashMap和双向链表合二为一。LinkedHashMap是HashMap的子类，拥有HashMap所有特性。LinkedHashMap支持LRU算法。
- HashMap内部是无序的，LinkedHashMap通过维护内部的双向链表保证了迭代顺序。
![](http://note.youdao.com/yws/public/resource/6d570a4731e802c585e8b26b774f0e30/xmlnote/6470514937724277985DCD01037CCE04/12780)
1. 成员变量
- 与HashMap相比，linkedHashMap增加了两个属性用于保证迭代顺序，分别是双向链表头结点header和标志位accessOrder（值为true时，表示按照访问顺序迭代，值为false时，表示按照插入顺序迭代）
2. LinkedHashMap没有增加额外的方法，重写了部分方法。
![](http://note.youdao.com/yws/public/resource/6d570a4731e802c585e8b26b774f0e30/xmlnote/41963E5194674F00A49B1DEFB90DA22E/12798)
3. Entry
![](http://note.youdao.com/yws/public/resource/6d570a4731e802c585e8b26b774f0e30/xmlnote/C8685352761E46DAA3D51F769645CA90/12783)
4. put
<br>LinkedHashMap完全继承了HashMap的put方法，只是对put方法所调用的recordAccess方法和addEntry方法进行了重写。
```
public V put(K key, V value) {

    //当key为null时，调用putForNullKey方法，并将该键值对保存到table的第一个位置 
    if (key == null)
        return putForNullKey(value); 

    //根据key的hashCode计算hash值
    int hash = hash(key.hashCode());           

    //计算该键值对在数组中的存储位置（哪个桶）
    int i = indexFor(hash, table.length);              

    //在table的第i个桶上进行迭代，寻找 key 保存的位置
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {      
        Object k;
        //判断该条链上是否存在hash值相同且key值相等的映射，若存在，则直接覆盖 value，并返回旧value
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this); // LinkedHashMap重写了Entry中的recordAccess方法--- (1)    
            return oldValue;    // 返回旧值
        }
    }

    modCount++; //修改次数增加1，快速失败机制

    //原Map中无该映射，将该添加至该链的链头
    addEntry(hash, key, value, i);  // LinkedHashMap重写了HashMap中的createEntry方法 ---- (2)    
    return null;
}
```
- addEntry方法
```
/**
 * This override alters behavior of superclass put method. It causes newly
 * allocated entry to get inserted at the end of the linked list and
 * removes the eldest entry if appropriate.
 *
 * LinkedHashMap中的addEntry方法
 */
void addEntry(int hash, K key, V value, int bucketIndex) {   

    //创建新的Entry，并插入到LinkedHashMap中  
    createEntry(hash, key, value, bucketIndex);  // 重写了HashMap中的createEntry方法

    //双向链表的第一个有效节点（header后的那个节点）为最近最少使用的节点，这是用来支持LRU算法的
    Entry<K,V> eldest = header.after;  
    //如果有必要，则删除掉该近期最少使用的节点，  
    //这要看对removeEldestEntry的覆写,由于默认为false，因此默认是不做任何处理的。  
    if (removeEldestEntry(eldest)) {  
        removeEntryForKey(eldest.key);  
    } else {  
        //扩容到原来的2倍  
        if (size >= threshold)  
            resize(2 * table.length);  
    }  
} 

-------------------------------我是分割线------------------------------------

 /**
 * Adds a new entry with the specified key, value and hash code to
 * the specified bucket.  It is the responsibility of this
 * method to resize the table if appropriate.
 *
 * Subclass overrides this to alter the behavior of put method.
 * 
 * HashMap中的addEntry方法
 */
void addEntry(int hash, K key, V value, int bucketIndex) {
    //获取bucketIndex处的Entry
    Entry<K,V> e = table[bucketIndex];

    //将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry 
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);

    //若HashMap中元素的个数超过极限了，则容量扩大两倍
    if (size++ >= threshold)
        resize(2 * table.length);
}
```
- createEntry
```
void createEntry(int hash, K key, V value, int bucketIndex) { 
    // 向哈希表中插入Entry，这点与HashMap中相同 
    //创建新的Entry并将其链入到数组对应桶的链表的头结点处， 
    HashMap.Entry<K,V> old = table[bucketIndex];  
    Entry<K,V> e = new Entry<K,V>(hash, key, value, old);  
    table[bucketIndex] = e;     

    //在每次向哈希表插入Entry的同时，都会将其插入到双向链表的尾部，  
    //这样就按照Entry插入LinkedHashMap的先后顺序来迭代元素(LinkedHashMap根据双向链表重写了迭代器)
    //同时，新put进来的Entry是最近访问的Entry，把其放在链表末尾 ，也符合LRU算法的实现  
    e.addBefore(header);  
    size++;  
} 
```
5. resize
```
这一块是处理原来的HashMap冲突的，因为容量扩容了
	原来通过 & 运算出来的hash冲突的key,与新的容量 & 可能位置会发生改变
	比如之前的冲突的两个key （k1,k2）的hash 分别为 1101，11101
	初始化默认的容量是16 二进制就是 10000， 16-1的二进制就是 1111 与key进行& 运算结果都是 1101，但是与11111(36-1)进行 一个是1101 另一是11101
	所以需要重新计算下标，通过与oldCap 进行 & 运算 分为低位和高位，因为和oldCap &运算只有两种结果，一种是0 另一种是oldCap
	将结果=0的设置成低位，将等于oldCap的保存到高位，
	低位的hash值肯定小于oldCap，所以下标还是在原来的位置
	高位的hash一定大于oldCap，在二进制最左至少会多一个1（有可能还有前面还有其他二进制数字，但运算结果都是一样的）计算正好是原来位置加上新的容量
```
- LinkedHashMap完全继承了HashMap的resize()方法，只是对它所调用的transfer方法进行了重写。
```
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;

    // 若 oldCapacity 已达到最大值，直接将 threshold 设为 Integer.MAX_VALUE
    if (oldCapacity == MAXIMUM_CAPACITY) {  
        threshold = Integer.MAX_VALUE;
        return;             // 直接返回
    }

    // 否则，创建一个更大的数组
    Entry[] newTable = new Entry[newCapacity];

    //将每条Entry重新哈希到新的数组中
    transfer(newTable);  //LinkedHashMap对它所调用的transfer方法进行了重写

    table = newTable;
    threshold = (int)(newCapacity * loadFactor);  // 重新设定 threshold
}
```
Map扩容操作的核心在于重哈希。所谓重哈希是指重新计算原HashMap中的元素在新table数组中的位置并进行行复制处理的过程。鉴于性能和LinkedHashMap自身特点的考量，LinkedHashMap对重哈希过程(transfer方法)进行了重写：
```
/**
 * Transfers all entries to new table array.  This method is called
 * by superclass resize.  It is overridden for performance, as it is
 * faster to iterate using our linked list.
 */
void transfer(HashMap.Entry[] newTable) {
    int newCapacity = newTable.length;
    // 与HashMap相比，借助于双向链表的特点进行重哈希使得代码更加简洁
    for (Entry<K,V> e = header.after; e != header; e = e.after) {
        int index = indexFor(e.hash, newCapacity);   // 计算每个Entry所在的桶
        // 将其链入桶中的链表
        e.next = newTable[index];
        newTable[index] = e;   
    }
}
```
从上述源码可知，LinkedHashMap借助于自身维护的双向链表轻松实现了重哈希操作。
6. get对比
```
public V get(Object key) {
    // 根据key获取对应的Entry，若没有这样的Entry，则返回null
    Entry<K,V> e = (Entry<K,V>)getEntry(key); 
    if (e == null)      // 若不存在这样的Entry，直接返回
        return null;
    e.recordAccess(this);
    return e.value;
}

/**
     * Returns the entry associated with the specified key in the
     * HashMap.  Returns null if the HashMap contains no mapping
     * for the key.
     * 
     * HashMap 中的方法
     *     
     */
    final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : hash(key);
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
```
7. 总结
- LinkedHashMap相对于HashMap只是要额外维护一个双向链表用于保持迭代顺序。
- put操作上，虽然LinkedHashMap完全继承了HashMap的put操作，还增加了其他，比如在LinkedHashMap中向哈希表中插入新Entry的同时，还会通过Entry的addBefore方法将其链入到双向链表中。

# 3 LinkedHashMap与LRU（Least recently used， 最近最少使用）算法

1. LinkedHashMap通过双向链表头结点header和标志位accessOrder来保证迭代顺序。
- 当accessOrder标志位为true时，表示双向链表中的元素按照访问的先后顺序排列，具体是将当前访问的Entry(put进来的Entry或get出来的Entry)移到双向链表的尾部，这样会使得链表既符合插入顺序，又符合访问的先后顺序，因为这时该Entry也被访问了。
- 当标志位accessOrder的值为false时，表示双向链表中的元素按照Rntry插入LinkedHashMap中的先后顺序，即每次put到LinkedHashMap中的Entry都放在双向链表的尾部，这样遍历双向链表时，Entry的输出顺序便和插入的顺序一致，这也是默认的双向链表的存储顺序。
- 当标志位accessOrder的值为false时，虽然也会调用recordAccess方法，但不做任何操作。
```
/**
* This method is invoked by the superclass whenever the value
* of a pre-existing entry is read by Map.get or modified by Map.set.
* If the enclosing Map is access-ordered, it moves the entry
* to the end of the list; otherwise, it does nothing.
*/
void recordAccess(HashMap<K,V> m) {  
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;  
    //如果链表中元素按照访问顺序排序，则将当前访问的Entry移到双向循环链表的尾部，  
    //如果是按照插入的先后顺序排序，则不做任何事情。  
    if (lm.accessOrder) {  
        lm.modCount++;  
        //移除当前访问的Entry  
        remove();  
        //将当前访问的Entry插入到链表的尾部  
        addBefore(lm.header);  
      }  
  } 
```
再结合创建节点，就很清晰了
```
   //创建新的Entry，并插入到LinkedHashMap中  
    createEntry(hash, key, value, bucketIndex);  // 重写了HashMap中的createEntry方法

    //双向链表的第一个有效节点（header后的那个节点）为最近最少使用的节点，这是用来支持LRU算法的
    Entry<K,V> eldest = header.after;  
    //如果有必要，则删除掉该近期最少使用的节点，  
    //这要看对removeEldestEntry的覆写,由于默认为false，因此默认是不做任何处理的。  
    if (removeEldestEntry(eldest)) {  
        removeEntryForKey(eldest.key);  
    } else {  
        //扩容到原来的2倍  
        if (size >= threshold)  
            resize(2 * table.length);  
    }  
} 

void createEntry(int hash, K key, V value, int bucketIndex) { 
    // 向哈希表中插入Entry，这点与HashMap中相同 
    //创建新的Entry并将其链入到数组对应桶的链表的头结点处， 
    HashMap.Entry<K,V> old = table[bucketIndex];  
    Entry<K,V> e = new Entry<K,V>(hash, key, value, old);  
    table[bucketIndex] = e;     

    //在每次向哈希表插入Entry的同时，都会将其插入到双向链表的尾部，  
    //这样就按照Entry插入LinkedHashMap的先后顺序来迭代元素(LinkedHashMap根据双向链表重写了迭代器)
    //同时，新put进来的Entry是最近访问的Entry，把其放在链表末尾 ，也符合LRU算法的实现  
    e.addBefore(header);  
    size++;  
}
```
- 总结来说，使用LinkedHashMap实现LRU的必要前提是将accessOrder标志位设为true以便开启按访问顺序排序的模式。无论是put方法还是get方法，都会导致目标Entry成为最近访问的Entry，因此就把该Entry加入到了双向链表的末尾：get方法通过调用recordAccess方法来实现；
2. 使用LinkedHashMap实现LRU算法
- 需要将accessOrder设置为true
- 需要重写removeEldestEntry方法
```
public class LRU<K,V> extends LinkedHashMap<K, V> implements Map<K, V>{

    private static final long serialVersionUID = 1L;

    public LRU(int initialCapacity,
             float loadFactor,
                        boolean accessOrder) {
        super(initialCapacity, loadFactor, accessOrder);
    }

    /** 
     * @description 重写LinkedHashMap中的removeEldestEntry方法，当LRU中元素多余6个时，
     *              删除最不经常使用的元素
     * @author rico       
     * @created 2017年5月12日 上午11:32:51      
     * @param eldest
     * @return     
     * @see java.util.LinkedHashMap#removeEldestEntry(java.util.Map.Entry)     
     */  
    @Override
    protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) {
        // TODO Auto-generated method stub
        if(size() > 6){
            return true;
        }
        return false;
    }

    public static void main(String[] args) {

        LRU<Character, Integer> lru = new LRU<Character, Integer>(
                16, 0.75f, true);

        String s = "abcdefghijkl";
        for (int i = 0; i < s.length(); i++) {
            lru.put(s.charAt(i), i);
        }
        System.out.println("LRU中key为h的Entry的值为： " + lru.get('h'));
        System.out.println("LRU的大小 ：" + lru.size());
        System.out.println("LRU ：" + lru);
    }
}
```
3. LinkedHashMap实现有序性的原理
- 需要重写HashMap中的迭代器
```
    public void remove() {
        if (lastReturned == null)
        throw new IllegalStateException();
        if (modCount != expectedModCount)
        throw new ConcurrentModificationException();

            LinkedHashMap.this.remove(lastReturned.key);
            lastReturned = null;
            expectedModCount = modCount;
    }

    Entry<K,V> nextEntry() {        // 迭代输出双向链表各节点
        if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
            if (nextEntry == header)
                throw new NoSuchElementException();

            Entry<K,V> e = lastReturned = nextEntry;
            nextEntry = e.after;
            return e;
    }
```
4. JDK1.8的改动
- 删除了addentry, createentry方法。
- 提供实现
```
 void afterNodeAccess(Node<K,V> p) { }
    void afterNodeInsertion(boolean evict) { }
    void afterNodeRemoval(Node<K,V> p) { }
```
本质思想是一样的。

## 1.7和1.8重要区别
- 1.7 哈希表，头插法，2个线程put，同时扩容会出现链表成环
- 哈希表，当链表达到一定长度则转化为红黑树
尾插法，避免2个线程put扩容链表成环
[1.7和1.8区别](https://www.cnblogs.com/lankerenf3039/p/12128199.html)

[头插和尾插的区别](https://www.cnblogs.com/wen-he/p/11496050.html)