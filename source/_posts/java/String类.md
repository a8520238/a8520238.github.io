---
title: String类
date: 2021-2-28
cover:
top_img:
categories: javaSE
tags: 
mathjax: true
katex: true
---
# 1 String str= "" 和 new String()区别

## String str1= “abc”

在编译期，JVM会去常量池来查找是否存在“abc”，

如果不存在，就在常量池中开辟一个空间来存储“abc”；如果存在，就不用新开辟空间。

然后在栈内存中开辟一个名字为str1的空间，来存储“abc”在常量池中的地址值。

## String str2 = new String("abc")

在编译阶段JVM先去常量池中查找是否存在“abc”，如果过不存在，则在常量池中开辟一个空间存储“abc”。

在运行时期，通过String类的构造器在堆内存中new了一个空间，然后将String池中的“abc”复制一份存放到该堆空间中，在栈中开辟名字为str2的空间，存放堆中new出来的这个String对象的地址值。

也就是说，前者在初始化的时候可能创建了一个对象，也可能一个对象也没有创建；后者因为new关键字，至少在内存中创建了一个对象，也有可能是两个对象。

![](http://note.youdao.com/yws/public/resource/cf631267d2ad6409b8796f1bae308c36/xmlnote/F7F6A16B9D124D96806319258EE3D80E/6690)


## 特殊创建中，常量池中没有，堆中有

```
String s1 = new String("he")+new String("llo"); 
```
1. 这个代码中，首先，new String("he")，先在常量池中看，发现没有这个"he"常量，于是建一个，然后再在堆中创建一个String的对象（但没引用，很快被gc的）。
2. 加法，暗中new了StringBuilder，调用append方法。
3. new String("llo")一样的道理，堆中一个String对象，常量池中"llo"常量对象。
4. StringBuilder的append方法搞定后，调用toString()方法，具体是new一个String对象，也就是现在是一个堆中的String对象，内容是"hello"，但注意这个hello没有在常量池中创建！！其实可以理解因为没有出现过一次"hello"，拼接是通过StringBuilder的append方法完成的。
 

> 总之：对于所有包含new方式新建对象（包括null）和变量形式 的“+”连接表达式，它所产生的新对象都不会被加入字符串池中。

# 2 String为什么是不可变的？

效率和安全两方面考虑

1).不可变对象可以提高String Pool的效率和安全性。如果你知道一个对象是不可变的，那么需要拷贝这个对象的内容时，就不用复制它的本身而只是复制它的地址，复制地址（通常一个指针的大小）需要很小的内存效率也很高。对于同时引用这个“ABC”的其他变量也不会造成影响。

2).不可变对象对于多线程是安全的，因为在多线程同时进行的情况下，一个可变对象的值很可能被其他进程改变，这样会造成不可预期的结果，而使用不可变对象就可以避免这种情况。

>> 注意字符串引用名相加，相当于new一个String

>> https://blog.csdn.net/u010887744/article/details/50844525

# 3 字符串常量池的位置

在JDK 7以前的版本中，字符串常量池是放在永久代中的。

因为按照计划，JDK会在后续的版本中通过元空间来代替永久代，所以首先在JDK 7中，将字符串常量池先从永久代中移出，暂时放到了堆内存中。

在JDK 8中，彻底移除了永久代，使用元空间替代了永久代，于是字符串常量池再次从堆内存移动到永久代中

# 4 JDK6和JDK7中substring的原理和区别

1. JDK6 中的substring
- String是通过字符数组实现的。在jdk 6 中，String类包含三个成员变量：char value[]， int offset，int count。他们分别用来存储真正的字符数组，数组的第一个位置索引以及字符串中包含的字符个数。
- 当调用substring方法的时候，会创建一个新的string对象，但是这个string的值仍然指向堆中的同一个字符数组。这两个对象中只有count和offset 的值是不同的。

2. JDK6中substring导致的问题：

- 如果你有一个很长很长的字符串，但是当你使用substring进行切割的时候你只需要很短的一段。这可能导致性能问题，因为你需要的只是一小段字符序列，但是你却引用了整个字符串（因为这个非常长的字符数组一直在被引用，所以无法被回收，就可能导致内存泄露）。
- 在JDK 6中，一般用以下方式来解决该问题，原理其实就是生成一个新的字符串并引用他。
```
x = x.substring(x, y) + ""
```
3. JDK7中的substring

上面提到的问题，在jdk 7中得到解决。在jdk 7 中，substring方法会在堆内存中创建一个新的数组。其使用new String创建了一个新字符串，避免对老字符串的引用。从而解决了内存泄露问题。

# 5 replaceFirst、replaceAll、replace

replace(CharSequence target, CharSequence replacement) ，用replacement替换所有的target，两个参数都是字符串。

replaceAll(String regex, String replacement) ，用replacement替换所有的regex匹配项，regex很明显是个正则表达式，replacement是字符串。
```
//文字替换（全部） 
Pattern pattern = Pattern.compile("正则表达式"); 
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World"); 
// Java Hello World,Java Hello World
System.out.println(matcher.replaceAll("Java"));
```

replaceFirst(String regex, String replacement) ，基本和replaceAll相同，区别是只替换第一个匹配项。

```
//文字替换（首次出现字符） 
Pattern pattern = Pattern.compile("正则表达式"); 
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World"); 
//Java Hello World,正则表达式 Hello World
System.out.println(matcher.replaceFirst("Java")); 
```

# 6 Pattern和Matcher

1. Pattern是创建一个正则匹配串，只能通过
```
Pattern pattern = Pattern.compile("正则表达式"); 
```
来创建，因为他的构造方法是私有的。其作用是编译正则表达式后创建一个匹配模式.

常用方法介绍：
- Pattern complie(String regex)，由于Pattern的构造函数是私有的,不可以直接创建,所以通过静态方法compile(String regex)方法来创建,将给定的正则表达式编译并赋予给Pattern类。
- String pattern() 返回正则表达式的字符串形式,其实就是返回Pattern.complile(String regex)的regex参数。
- Pattern.matcher(CharSequence input) 对指定输入的字符串创建一个Matcher对象。
- String[] split(CharSequence input, int limit)
功能和String[] split(CharSequence input)相同,增加参数limit目的在于要指定分割的段数。
```
String regex = "\\?|\\*";
Pattern pattern = Pattern.compile(regex);
String[] splitStrs = pattern.split("123?123*456*456");//123 123 456 456
String[] splitStrs2 = pattern.split("123?123*456*456", 2);// 123 123*456*456
```

2. Matcher类使用Pattern实例提供的模式信息对正则表达式进行匹配

- boolean matches() 最常用方法:尝试对整个目标字符展开匹配检测,也就是只有整个目标字符串完全匹配时才返回真值.
```
Pattern pattern = Pattern.compile("\\?{2}");
Matcher matcher = pattern.matcher("??");
boolean matches = matcher.matches();//true
```
- boolean lookingAt() 对前面的字符串进行匹配,只有匹配到的字符串在最前面才会返回true.
```
Pattern p = Pattern.compile("\\d+");
Matcher m = p.matcher("22bb23");
boolean match = m.lookingAt();//true
```
- boolean find() 对字符串进行匹配,匹配到的字符串可以在任何位置
```
Pattern p = Pattern.compile("\\d+");
Matcher m = p.matcher("22bb23");
m.find();// 返回true
Matcher m2 = p.matcher("aa2223");
m2.find();// 返回true
Matcher m3 = p.matcher("aa2223bb");
m3.find();// 返回true
Matcher m4 = p.matcher("aabb");
m4.find();// 返回false
```
- group，返回匹配到的字符串
```
Pattern p = Pattern.compile("\\d+");
Matcher m = p.matcher("aa22bb23");
m.find();
int start = m.start();//2
String group = m.group();//22
int end = m.end();//4
```

3. Pattern模式 
- Pattern.MULTILINE模式的用法

正则表达式中出现了^或者$, 默认只会匹配第一行. 设置了Pattern.MULTILINE模式,会匹配所有行。例如，
```
Pattern p1 = Pattern.compile("^.*b.*$");
//输出false,因为正则表达式中出现了^或$，默认只会匹配第一行，第二行的b匹配不到。
System.out.println(p1.matcher("a\nb").find());
Pattern p2 = Pattern.compile("^.*b.*$",Pattern.MULTILINE);
//输出true,指定了Pattern.MULTILINE模式，就可以匹配多行了。
System.out.println(p2.matcher("a\nb").find());
```
- Pattern.DOTALL模式的用法

默认情况下, 正则表达式中点(.)不会匹配换行符, 设置了Pattern.DOTALL模式, 才会匹配所有字符包括换行符。例如，
```
Pattern p1 = Pattern.compile("a.*b");
//输出false，默认点(.)没有匹配换行符
System.out.println(p1.matcher("a\nb").find());
Pattern p2 = Pattern.compile("a.*b", Pattern.DOTALL);
//输出true,指定Pattern.DOTALL模式，可以匹配换行符。
System.out.println(p2.matcher("a\nb").find());
```

- 同时指定Pattern.MULTILINE和Pattern.DOTALL模式

实际情况中要是比较复杂的情况，可能Pattern.MULTILINE模式和Pattern.DOTAL模式需要同时指定来匹配多行，下面看一下，
```
Pattern p1 = Pattern.compile("^a.*b$");
//输出false
System.out.println(p1.matcher("cc\na\nb").find());
Pattern p2 = Pattern.compile("^a.*b$", Pattern.DOTALL);
//输出false,因为有^或&没有匹配到下一行
System.out.println(p2.matcher("cc\na\nb").find());
Pattern p3 = Pattern.compile("^a.*b$", Pattern.MULTILINE);
//输出false，匹配到下一行，但.没有匹配换行符
System.out.println(p3.matcher("cc\na\nb").find());
//指定多个模式，中间用|隔开
Pattern p4 = Pattern.compile("^a.*b$", Pattern.DOTALL|Pattern.MULTILINE);
//输出true
System.out.println(p4.matcher("cc\na\nb").find());
```
# 7 String s = a + b + c + d + e，共创建了多少对象

1. 如果是String s = "a" + "b" + "c" + "d" + "e"
- 会创建1个对象，赋值符号右边的"a"、"b"、"c"、"d"、"e"都是常量
对于常量，编译时就直接存储它们的字面值而不是它们的引用
在编译时就直接讲它们连接的结果提取出来变成了"abcde"
该语句在class文件中就相当于String s = "abcde"
然后当JVM执行到这一句的时候， 就在String pool里找
如果没有这个字符串，就会产生一个。
2. String s = a+b+c+d+e; 其中各个字母都是引用

- 3个对象，但只有一个String对象。由于编译器的优化，最终代码为通过StringBuilder完成。
- 总结：三个对象分别为

1 StringBuilder

2 new char[capacity]

3 new String(value,0,count);

> 两个String 相加， 会转成StringBuilder.append

# 8 String连接

- \+
- concat
- StringBuilder
- StringBuffer
- StringUtils.join

> 如果不是在循环体中进行字符串拼接的话，直接使用+就好了。

> 如果在并发场景中进行字符串拼接的话，要使用StringBuffer来代替StringBuilder。

# 9 String.valueOf和Integer.toString的区别

```
1.int i = 5;
2.String i1 = "" + i;
3.String i2 = String.valueOf(i);
4.String i3 = Integer.toString(i);
```
第三行和第四行没有区别，因为String.valueOf(i)也是调用Integer.toString(i)来实现的。

第二行代码其实是String i1 = (new StringBuilder()).append(i).toString(); 首先创建一个StringBuilder对象。

# 10 intern

在每次赋值的时候使用 String 的 intern 方法，如果常量池中有相同值，就会重复使用该对象，返回对象引用。

```
Q：下列程序的输出结果：
String s1 = “abc”;
String s2 = “abc”;
System.out.println(s1 == s2);
A：true，均指向常量池中对象。

Q：下列程序的输出结果：
String s1 = new String(“abc”);
String s2 = new String(“abc”);
System.out.println(s1 == s2);
A：false，两个引用指向堆中的不同对象。

Q：下列程序的输出结果：
String s1 = “abc”;
final String s2 = “a”;
final String s3 = “bc”;
String s4 = s2 + s3;
System.out.println(s1 == s4);
A：true，因为final变量在编译后会直接替换成对应的值，所以实际上等于s4=”a”+”bc”，而这种情况下，编译器会直接合并为s4=”abc”，所以最终s1==s4。

Q：下列程序的输出结果：
String s1 = “abc”;
String s2 = “a”;
String s3 = “bc”;
String s4 = s2 + s3;
System.out.println(s1 == s4);
A：false，因为s2+s3实际上是使用StringBuilder.append来完成，会生成不同的对象。


String a = "abc";
String b = new String("abc");
System.out.println(a == b);
fasle 一个返回常量池，一个返回堆中
```

```
Q：下列程序的输出结果：
String s = new String(“abc”);
String s1 = “abc”;
String s2 = new String(“abc”);
System.out.println(s == s1.intern());
System.out.println(s == s2.intern());
System.out.println(s1 == s2.intern());
A：false，false，true。
```

## 10.1 1.6和1.7中的intern

```
public static void main(String[] args) {
    String str1 = new StringBuilder("计算机").append("软件").toString();
    System.out.println(str1.intern() == str1);

    String str2 = new StringBuilder("ja").append("va").toString();
    System.out.println(str2.intern() == str2);
}
```
这段代码在JDK1.6中，会得到两个false，在JDK1.7中运行，会得到一个true和一个false。

- 书上说，产生差异的原因是：在JDK1.6中，intern()方法会把首次遇到的字符串实例复制到永久代中，返回的也是永久代中这个字符串实例的引用，而由StringBuilder创建的字符串实例在Java堆上，所以必然不是同一个引用，将返回false。

- 而JDK1.7的intern()不会再复制实例，只是在常量池中记录首次出现的实例的引用，因此intern()返回的引用和StringBuilder创建的那个字符串的实例是同一个。对str2比较返回false是因为"java"这个字符串在执行StringBuilder.toString()之前就已经出现过，字符串常量池中已经有它的引用了，不符合“首次出现”的原则，而“计算机软件”这个字符串则是首次出现的，因此返回true。

## 10.2 stringTable

这里先再提一下字符串常量池，实际上，为了提高匹配速度，也就是为了更快地查找某个字符串是否在常量池中，Java在设计常量池的时候，还搞了张stringTable，这个有点像我们的hashTable，根据字符串的hashCode定位到对应的桶，然后遍历数组查找该字符串对应的引用。如果找得到字符串，则返回引用，找不到则会把字符串常量放到常量池中，并把引用保存到stringTable了里面。

在JDK7、8中，可以通过-XX:StringTableSize参数StringTable大小

## 10.3 jdk1.6及其之前的intern()方法

在JDK6中，常量池在永久代分配内存，永久代和Java堆的内存是物理隔离的，执行intern方法时，如果常量池不存在该字符串，虚拟机会在常量池中复制该字符串，并返回引用；如果已经存在该字符串了，则直接返回这个常量池中的这个常量对象的引用。所以需要谨慎使用intern方法，避免常量池中字符串过多，导致性能变慢，甚至发生PermGen内存溢出。

**当然，这个常量池和堆是物理隔离的。**

![](http://note.youdao.com/yws/public/resource/cf631267d2ad6409b8796f1bae308c36/xmlnote/9953192D405E40A490141C053FAEEC0F/6710)

## 10.4 jdk1.7的intern()方法 (1.8之后)

JDK 1.7后，intern方法还是会先去查询常量池中是否有已经存在，如果存在，则返回常量池中的引用，这一点与之前没有区别，区别在于，如果在常量池找不到对应的字符串，则不会再将字符串拷贝到常量池，而只是在常量池中生成一个对原字符串的引用。简单的说，就是往常量池放的东西变了：原来在常量池中找不到时，复制一个副本放到常量池，1.7后则是将在堆上的地址引用复制到常量池。

当然这个时候，常量池被从方法区中移出来到了堆中。

看一个特殊的例子

```
String str2 = new String("str")+new String("01");
str2.intern();
String str1 = "str01";
System.out.println(str2==str1);//true

```
这个返回true的原因也一样，str2的时候，只有一个堆的String对象，然后调用intern，常量池中没有“str01”这个常量对象，于是常量池中生成了一个对这个堆中string对象的引用。

然后给str1赋值的时候，因为是带引号的，所以去常量池中找，发现有这个常量对象，就返回这个常量对象的引用，也就是str2引用所指向的堆中的String对象的地址。

所以str2和str1指向的是同一个东西，所以为true。

![](http://note.youdao.com/yws/public/resource/cf631267d2ad6409b8796f1bae308c36/xmlnote/98CB3B6F74BF45DD83A656A08DF48060/6712)

# 11 字符串长度的限制

- 编译期
1. 常量池字面量限制65536
2. javac的Gen类中限制不能>=65535

所以编译期String最长65534
- 运行期
运行期长度的限制是不能超过Int的范围