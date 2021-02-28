# Java 堆中的对象内存

## 1 java 中的对象指向问题

### 1.1 方法区指向堆

`private static User user=new User();` 

+ 引用变量user是静态的，所以存放在方法区中，而真实的user对象存放在堆中。所以，这就形成了方法区指向堆。

+ 或者类变量是静态的，也会指向堆。
+ 或者是final修饰的类常量。

### 1.2 虚拟机线程栈指向堆

类的方法中的属性存放在虚拟机栈的局部变量表 中，对象实例同样存在在堆中。

## 2 堆中对象的实例的布局

### 2.1 对象内存模型

java 中的对象内存布局：

- 对象头 Header
    + mark word ： 8 字节
    + class pointer ： 未指针压缩时 8 字节，压缩后 4 字节
    + 数组 length （数组对象特有）： 4 字节
- 对象实例数据 Instance Data ：
- 对齐填充 Padding ： 填充为 8 字节的整数倍

![java 中的对象内存布局](http://note.youdao.com/yws/public/resource/bfce0e3d92cf4516094fe684a07f9b39/xmlnote/88CB4B6F16FA4A71A606DA0E62EC8BE5/9123)

### 2.2 Object obj = new Object() 占用的字节

实例数据为空，mark word 占 8 字节，其他：

- 若开启了类指针压缩 则 class pointer 占4字节，然后 padding 填充 4 字节
- 若没开启，则 class pointer 占 8 字节，padding 不填充

所以，一共占用 16 字节。

## 3 堆中对象访问

### 3.1 句柄访问

堆中划分出一块空间来存放 句柄池 , 即 方法区/栈帧 中的对象引用是存放的句柄地址，通过句柄池再在堆中查找对象的实际地址。

![句柄访问](http://note.youdao.com/yws/public/resource/7839ea220156efcfc3c18b40b088fedd/xmlnote/7EA3C812D0484F4583883EAF2DCE4445/2708)

### 3.2 直接指针访问 (hot spot)

堆中直接存放对象的实例数据，即 方法区/栈帧 中的对象引用直接指向堆实例的地址。

![直接指针访问](http://note.youdao.com/yws/public/resource/7839ea220156efcfc3c18b40b088fedd/xmlnote/0F7750F4ACE14A5BB5B93239A0203931/2707)