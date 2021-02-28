---
title: IOC
date: 2021-2-28
cover:
top_img:
categories: spring
tags: 
mathjax: true
katex: true
---
# IOC 原理

## 1 工厂
> IOC的核心就是工厂

- spring相比于EJB是一个轻量级的框架，它整合了工厂，代理，模板，策略等多个设计模式。
- 工厂模式的关键在与解耦合，针对设计模式中的打开扩展，关闭修改。工厂模式将源码中的赋值操作提取出到配置文件中。

### 1.1 对象的创建方式

1. 直接使用new来创建
2. 通过反射的方式创建
```
Class clazz = Class.forName(env.getProperty("userDAO"));
userDAO = (UserDAO) clazz.newInstance();
```

### 1.2 简单工厂和工厂设计模式

简单工厂对于每一个对象有自己的工厂。而每一个工厂中的try，catch块等代码存在，造成了一种冗余。

工厂设计模式可以解决这种冗余，通过创建一种方法getObject,返回Object类型对象，因为所有对象都是Object子类。这样一个工厂可以创建所有对象，解决了冗余。

Spring本质：工厂 ApplicationContext(applicationContext.xml)

## 2 Spring中工厂的使用

1. ApplicationContext(Spring中的工厂)（接口）
    - 非web环境： ClassPathXmlApplicationContext
    - 注解形式：AnnotationConfigApplicationContext
    - web环境：XmlWebApplicationContext
2. ApplicationContext是重量级资源，一个应用只会创建一个工厂对象，一定是线程安全的（多线程并发访问）
3. beanDefinitionNames获取的是Spring工厂配置文件中所有bean标签的id值
    ```
    String[] beanDefinitionNames =
    ctx.getBeanDefinitionNames();
    ```
## 3 注入

### 3.1 配置文件注入
注入也就是通过Spring工厂及配置文件，为所创建对象的成员变量赋值。

1. 使用编码的方式比如set对成员变量赋值存在耦合。
2. 通过Spring配置文件赋值
```
<bean id="person"
    class="com.baizhiedu.basic.Person
    ">
    <property name="id">
    <value>10</value>
    </property>
    <property name="name">
    <value>xiaojr</value>
    </property>
</bean>
```
![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/EC465B4348FD4559A3268BCE1E30801B/4828)

### 3.2 Set注入详解

![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/944309719CDA4598B24E24EAEAE7431D/4835)

1. JDK内置类型
- String及8中基本类型
```
    <value>suns</value>
```
- 数组
```
<list>
<value>suns@zparkhr.com.cn</value>
<value>liucy@zparkhr.com.cn</value>
<value>chenyn@zparkhr.com.cn</value>
</list>
```
- Set集合
```
<set>
    <value>11111</value>
    <value>112222</value>
</set>
<set>
    <ref bean
</set>
```
- list集合
```
<list>
    <value>11111</value>
    <value>2222</value>
</list>
```
- Map集合
```
<map>
    <entry>
        <key><value>suns</value></key>
        <value>3434334343</value>
    </entry>
    <entry>
        <key><value>chenyn</value></key>
    </entry>
</map>
```
- Properites 是一种特殊的Map key=String value=String
```
<props>
    <prop key="key1">value1</prop>
    <prop key="key2">value2</prop>
</props>
```
- 复杂的JDK类型（Date）
需要程序员自定义类型转换器，处理。
2. 用户自定义类型
- 为成员变量提供set get方法，在配置文件中进行注入
```
<bean id="userService" class="xxxx.UserServiceImpl">
    <property name="userDAO">
        <bean class="xxx.UserDAOImpl"/>
    </property>
</bean>
```
- 上面的方法存在缺陷，配置文件代码冗余，被注入对象多次创建，浪费内存资源。使用下面的方法注入引用。
```
<bean id="userDAO" class="xxx.UserDAOImpl"/>
<bean id="userService" class="xxx.UserServiceImpl">
    <property name="userDAO">
        <ref bean="userDAO"/>
    </property>
</bean>
```
3. 基于p命名空间简化，也就是不用<property>标签
- 基本类型
    + <bean id="person" class="xxx.Person" p:name="suns"/>
- 自定义类型
    + <bean id="userService" class="xxx.UserServiceImpl"
 p:userDAO-ref="userDAO"/>

### 3.3 构造注入

注⼊：通过Spring的配置⽂件，为成员变量赋值

Set注⼊：Spring调⽤Set⽅法 通过配置⽂件为成员变量赋值

构造注⼊：Spring调⽤构造⽅法 通过配置⽂件为成员变量赋值

- 首先为实例对象提供构造器方法，再配置配置文件如下
```
<bean id="customer" class="com.baizhiedu.basic.constructer.Customer">
    <constructor-arg>
        <value>suns</value>
    </constructor-arg>
    <constructor-arg>
        <value>102</value>
    </constructor-arg>
</bean>
```
- 构造方法重载
    + 参数个数不同时：通过控制<constructor-arg>标签的数量进⾏区分
    + 构造参数个数相同时：通过在标签引⼊ type属性 进⾏类型的区分<constructor-arg type="">
### 3.4 注入的总结
在实战中，应用set注入更多，因为构造方法麻烦（重载），spring底层框架大量应用set注入。

## 4 反转控制与依赖注入

### 4.1 反转控制（IOC Inverse of Control）
- 控制：对于成员变量赋值的控制权
- 反转控制：把对于成员变量赋值的控制权，从代码反转（转移）到Spring工厂和配置文件中完成
- 好处：松耦合
- 底层实现：工厂设计模式

### 4.2 依赖注入（Dependency Injection DI）
- 注入：通过Spring的⼯⼚及配置⽂件，为对象（bean，组件）的成员变量赋值
- 依赖注入：：当⼀个类需要另⼀个类时，就意味着依赖，⼀旦出现依赖，就可以把另⼀个类作为本类的成员变量，最终通过Spring配置⽂件进⾏注⼊(赋值)。
- 好处：解耦合

## 5 Spring工厂创建复杂对象

### 5.1 什么是复杂对象
复杂对象：指的是不能直接通过new构造方法创建的对象

比如 Connection SqlSessionFactory
### 5.2 Spring工厂创建复杂对象的3种方法

1. FactoryBean 接口
    + 首先实现FactoryBean接口
    ![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/E4764668C2A1402BA5B96CE03B3FB176/4955)
    + Spring配置文件配置
        - 如果Class中指定的类型 是FactoryBean接⼝的实现类，那么通过id值获得的是这个类所创建的复杂对象Connection,也就是getObject获得的对象。
        - <bean id="conn" class="com.baizhiedu.factorybea n.ConnectionFactoryBean"/>
> 如果就想获得FactoryBean类型的对象ctx.getBean("&conn") 获得就是ConnectionFactoryBean对象.

> isSingleton⽅法 返回 true 只会创建⼀个复杂对象。一般来说SqlSessionFactory每次都要创建一个true。而Connection是false。

2. 实例工厂
作用：避免Spring框架的侵入，整合遗留系统

```
<bean id="connFactory"
class="com.baizhiedu.factorybean.ConnectionFactory"></bean>
<bean id="conn" factorybean=" connFactory" factorymethod="
getConnection"/>
```
3. 静态工厂
```
<bean id="conn"
class="com.baizhiedu.factorybean.
StaticConnectionFactory" factorymethod="
getConnection"/>
```

## 6 工厂生命周期

![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/7E396C782C8241AD8E3C8328DC8C388A/4992)

工厂的生命周期包括对象的创建、存活、销毁。

### 6.1 创建阶段

工厂创建的时机：
1. 如果scope="singleton",spring工厂创建的同时创建对象，要注意也可以使用懒加载，也就是如下:
<bean lazy-init="true"/>
2. 如果scope="prototype"，spring工厂会在获取对象的同时，创建对象。也就是ctx.getBean("")的时候。

### 6.2 初始化阶段

- Spring⼯⼚在创建完对象后，调⽤对象的初
始化⽅法，完成对应的初始化操作
- 初始化⽅法提供：程序员根据需求，提供
初始化⽅法，最终完成初始化操作
- 初始化⽅法调⽤：Spring⼯⼚进⾏调⽤

1. InitializingBean接⼝
```
//程序员根据需求，实现的⽅法，完成初
始化操作
public void
afterProperitesSet(){
}
```
2. 对象中提供一个普通的方法
```
public void myInit(){
}
<bean id="product"
class="xxx.Product" initmethod="
myInit"/>
```
3. 顺序

- 如果上述方法同时使用，InitializingBean在普通初始化方法之前。
- 注入在初始化方法之前。

### 6.3 销毁阶段

Spring销毁对象前，会调⽤对象的销毁⽅法，完成销毁操作
- Spring什么时候销毁所创建的对象？ ctx.close();
- 销毁⽅法：程序员根据⾃⼰的需求，定义销毁⽅法，完成销毁操作。调⽤：Spring⼯⼚完成调⽤

1. 使用
```
public void myDestroy()throws
Exception{
}
<bean id="" class="" initmethod=""
destroymethod="
myDestroy"/>
```
2. 分析
- 销毁⽅法的操作只适⽤于 scope="singleton"
- 资源的释放操作
io.close()
connection.close();

## 7 property配置

jdbc.driverClassName = com.mysql.jdbc.Driver

jdbc.url = jdbc:mysql://localhost:3306/suns?useSSL=false

jdbc.username = root

jdbc.password = 123456

> Spring的配置⽂件与⼩配置⽂件进⾏整合
```
applicationContext.xml
<context:property-placeholder location="classpath:/db.properties"/>
```

在Spring配置文件中通过${key}获取小配置文件中对应的值
![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/B26F81FBAE2B42318EC9FEE72D2A2A84/5060)

## 8 自定义类型转换器

1. 类型转换器
    作用：Spring通过类型转换器把配置文件中字符串类型的数据，转换成了对象中成员变量对应类型的数据，进而完成了注入。

![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/3FAAA19BFB204F749AAAD7FCED8F6453/5077)
2. 自定义类型转换器

原因：当Spring内部没有提供特定类型转换器时，而程序员在应用的过程中还需要使用，就需要自定义。

比如类implement Converter接口

```
public class MyDateConverter implements Converter<String, Date> {
//convert⽅法作⽤：String ---> Date
    SimpleDateFormat sdf = new SimpleDateFormat();
    sdf.parset(String) ---> Date
    param:source 代表的是配置⽂件中 ⽇期字符串 <value>2020-10-11</value>
    return : 当把转换好的Date作为convert⽅法的返回值后，Spring⾃动的为birthday属性进⾏注⼊（赋值）
    @Override
    public Date convert(String source) {
        Date date = null;
        try {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            date = sdf.parse(source);
            } catch (ParseException e) {
            e.printStackTrace();
            }
        return date;
        }
    }
```
然后在Spring的配置文件中进行配置
- MyDateConverter对象创建出来
```
<bean id="myDateConverter"
class="xxxx.MyDateConverter"/>
```
- 类型转换器的注册
```
⽬的：告知Spring框架，我们所创建的MyDateConverter是⼀个类型转换器
<!--⽤于注册类型转换器-->
<bean id="conversionService" class="org.springframework.context
.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <ref bean="myDateConverter"/>
        </set>
    </property>
</bean>
```
3. 细节
- MyDateConverter中的⽇期的格式，通过依赖注⼊的⽅式，由配置⽂件完成赋值。
```
public class MyDateConverter implements Converter<String, Date> {
    private String pattern;
    public String getPattern() {
    return pattern;
}
public void setPattern(String pattern) {
    this.pattern = pattern;
}
/*
convert⽅法作⽤：String ---> Date
SimpleDateFormat sdf = new SimpleDateFormat();
sdf.parset(String) ---> Date
param:source 代表的是配置⽂件中 ⽇期字符串 <value>2020-10-11</value>
return : 当把转换好的Date作为convert⽅法的返回值后，Spring⾃动的为birthday属性进⾏注⼊（赋值）
*/
@Override
public Date convert(String source) {
    Date date = null;
    try {
        SimpleDateFormat sdf = new SimpleDateFormat(pattern);
    date = sdf.parse(source);
    } catch (ParseException e) {
        e.printStackTrace();
    }
    return date;
    }
}
<!--Spring创建MyDateConverter类型
对象-->
<bean id="myDateConverter" class="com.baizhiedu.converter
.MyDateConverter">
<property name="pattern" value="yyyy-MM-dd"/>
</bean>
```
> ConversionSeviceFactoryBean 定义 id属性 值必须conversionService

> pring框架内置⽇期类型的转换器, ⽇期格式：2020/05/01 (不⽀持 ：2020-05-01)

## 9 后置处理Bean

BeanPostProcessor作⽤：对Spring⼯⼚所创建的对象，进⾏再加⼯。本质上讲是一种AOP。

![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/4E5A9EAC8DD24C5BA29DB46FDF53E9BA/5079)

程序员实现BeanPostProcessor规定接⼝中的⽅法：

Object postProcessBeforeInitiallization(Object bean String beanName)

作⽤：Spring创建完对象，并进⾏注⼊后，可以运⾏Before⽅法进⾏加⼯。获得Spring创建好的对象 ：通过⽅法的参数
最终通过返回值交给Spring框架。

```
Object postProcessAfterInitiallization(
Object bean String beanName)
```
作⽤：Spring执⾏完对象的初始化操作后，可以运⾏After⽅法进⾏加⼯获得Spring创建好的对象 ：通过⽅法的参数最终通过返回值交给Spring框架。

实战中：很少处理Spring的初始化操作：没有必要区分Before After。只需要实现其中的⼀个After⽅法即可

注意：
```
postProcessBeforeInitiallization
return bean对象
```

[IOC源码解读](https://blog.csdn.net/nuomizhende45/article/details/81158383)