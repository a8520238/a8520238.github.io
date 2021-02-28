---
title: SPI
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
# SPI

- API (Application Programming Interface) 大多数情况下，都是实现方来指定接口并完成对接口的不同实现，调用方仅仅依赖却无权选择不同实现。
- SPI (Service Provider Interface) 而如果是调用方来制定接口，实现方来针对接口来实现不同的实现。调用方来选择自己需要的实现方。

## 1 如何定义SPI

1. 定义一组接口（假如是org.foo.demo.IShout）,并写出接口的一个或多个实现，假设是Dog和Cat

```
public interface IShout {
    void shout();
}
public class Cat implements IShout {
    @Override
    public void shout() {
        System.out.println("miao miao");
    }
}
public class Dog implements IShout {
    @Override
    public void shout() {
        System.out.println("wang wang");
    }
}
```
2. 在 src/main/resources/ 下建立 /META-INF/services 目录， 新增一个以接口命名的文件 (org.foo.demo.IShout文件)，内容是要应用的实现类（这里是org.foo.demo.animal.Dog和org.foo.demo.animal.Cat，每行一个类）。

```
org.foo.demo.animal.Dog
org.foo.demo.animal.Cat
```

3. 使用 ServiceLoader 来加载配置文件中指定的实现。
```
public class SPIMain {
    public static void main(String[] args) {
        ServiceLoader<IShout> shouts = ServiceLoader.load(IShout.class);
        for (IShout s : shouts) {
            s.shout();
        }
    }
}
```
## 2 SPI的实现原理
ServiceLoader类
```
public final class ServiceLoader<S> implements Iterable<S>{
private static final String PREFIX = "META-INF/services/";

    // 代表被加载的类或者接口
    private final Class<S> service;

    // 用于定位，加载和实例化providers的类加载器
    private final ClassLoader loader;

    // 创建ServiceLoader时采用的访问控制上下文
    private final AccessControlContext acc;

    // 缓存providers，按实例化的顺序排列
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // 懒查找迭代器
    private LazyIterator lookupIterator;

    ......
}
```
流程：
1. 应用程序调用ServiceLoader.load方法。ServiceLoader.load方法内先创建一个新的ServiceLoader，并实例化该类中的成员变量，包括：
- loader(ClassLoader类型，类加载器)
- acc(AccessControlContext类型，访问控制器)
- providers(LinkedHashMap类型，用于缓存加载成功的类)
- lookupIterator(实现迭代器功能)
2. 应用程序通过迭代器接口获取对象实例
- ServiceLoader先判断成员变量providers对象中(LinkedHashMap类型)是否有缓存实例对象，如果有缓存，直接返回。 如果没有缓存，执行类的装载：
    + 读取META-INF/services/下的配置文件，获得所有能被实例化的类的名称
    + 通过反射方法Class.forName()加载类对象，并用instance()方法将类实例化
    + 把实例化后的类缓存到providers对象中(LinkedHashMap类型）
    + 返回实例对象。
## 3 SPI的两种使用方法

```
public class Test {
    public static void main(String[] args) {    
        Iterator<SPIService> providers = Service.providers(SPIService.class);
        ServiceLoader<SPIService> load = ServiceLoader.load(SPIService.class);

        while(providers.hasNext()) {
            SPIService ser = providers.next();
            ser.execute();
        }
        System.out.println("--------------------------------");
        Iterator<SPIService> iterator = load.iterator();
        while(iterator.hasNext()) {
            SPIService ser = iterator.next();
            ser.execute();
        }
    }
}
```
```
//输出
SpiImpl1.execute()
SpiImpl2.execute()
--------------------------------
SpiImpl1.execute()
SpiImpl2.execute()
```

[深入理解SPI机制](https://www.jianshu.com/p/3a3edbcd8f24)