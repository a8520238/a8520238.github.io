---
title: AOP
date: 2021-2-28
cover:
top_img:
categories: spring
tags: 
mathjax: true
katex: true
---
# AOP面向切面编程

AOP可以分为静态代理和动态代理

## 1 静态代理

目的： 通过代理类，为原始类（⽬标）增加额外的功能

好处：利于原始类(⽬标)的维护

代理类 = 目标类 + 额外功能 + 原始类实现相同的接口

静态代理存在的问题是要为每一个类创建代理类。静态文件数量过多存在冗余。

## 2 动态代理

### 2.1 概念

概念：通过代理类为原始类（目标类）增加额外的功能
好处：利于原始类的维护

### 2.2 步骤

1. 创建原始对象
2. 额外功能，(MethodBeforeAdviceJ接口 MethodInteceptor)
3. 定义切入点
- 切⼊点：额外功能加⼊的位置
- ⽬的：由程序员根据⾃⼰的需要，决定额外功能加⼊给那个原始⽅法，如 register login
- 简单的测试：所有⽅法都做为切⼊点，都加⼊额外的功能。
```
<aop:config>
    <aop:pointcut id="pc" expression="execution(* *(..))"/>
</aop:config>
```
4. 组装
```
<aop:advisor adviceref="before" pointcutref="pc"/>
```
5. 调用

### 2.3 细节

1. Spring创建的动态代理列在哪

- Spring框架在运⾏时，通过动态字节码技术，在JVM创建的，运⾏在JVM内部，等程序结束后，会和JVM⼀起消失
2. 什么叫动态字节码技术
- 通过第三个动态字节码框架，在JVM中创建对应类的字节码，进⽽创建对象，当虚拟机结束，动态字节码跟着消失。主要分为JDK和CGlib两种

## 3 Spring 动态代理详解


### 3.1 MethodBeforeAdvice

这个接口不常用，但需要理解，具体步骤是实现MethodBeforeAdvice接口，重新其中的before方法。

```
public class Before1 implements
MethodBeforeAdvice {
/*
    作⽤：需要把运⾏在原始⽅法执⾏之前运⾏的额外功能，书写在before⽅法中
Method: 额外功能所增加给的那个原始⽅法,比如
login⽅法
register⽅法
showOrder⽅法
Object[]: 额外功能所增加给的那个原始⽅法的参数。String
name,String password
Object: 额外功能所增加给的那
个原始对象 UserServiceImpl
OrderServiceImpl
*/
@Override
public void before(Method method, Object[] args, Object
target) throws Throwable {
        System.out.println("-----new method before advice log------");
    }
}
```

### 3.2 MethodInterceptor（方法拦截器）

> methodinterceptor接⼝：额外功能可以根据需要运⾏在原始⽅法执⾏ 前、后、前后。

```
public class Arround implements MethodInterceptor {
/*
invoke⽅法的作⽤:额外功能书写在invoke额外功能
原始⽅法之前
原始⽅法之后
原始⽅法执⾏之前 之后
确定：原始⽅法怎么运⾏
参数：MethodInvocation（Method):额外功能所增加给的那个原始⽅法login, 
register
invocation.proceed() ---> login运⾏
register运⾏
返回值：Object: 原始⽅法的返回值

示例
@Override
public Object invoke(MethodInvocation invocation) throws Throwable {
    System.out.println("-----额外功能 log----");
    Object ret = invocation.proceed();
    return ret;
    }
}
```

### 3.3 细节

1. 执行顺序
    - 事务要执行在原始方法之前
    - 额外功能也可以在异常中使用
2. 影响原始方法的返回值
    - 要想影响额外方法的返回值，不能直接返回原始方法的返回结果。

## 4 切入点详解

切入点决定了额外功能加入的位置

```
<aop:pointcut id="pc" expression="execution(* *(..))"/>
exection(* *(..)) ---> 匹配了所有⽅法
```

### 4.1 切入点表达式

```
* *(..) --> 所有⽅法
* ---> 修饰符 返回值
* ---> ⽅法名
()---> 参数表
..---> 对于参数没有要求 (参数有没
有，参数有⼏个都⾏，参数是什么类型的
都⾏)
```

- 注意非java.lang包中的类型，必须要写全限定名。
- 类切入点：指定特定类为切入点，为类中所有方法加入额外功能。 ```com.baizhiedu.proxy.UserServi
ceImpl.*(..) ```
- 忽略包指定类，其中的..代表多级包的意思，
```* *..UserServiceImpl.*(..) ```
- 切入点及当前子包都生效，```* com.baizhiedu.proxy..*.*
(..) ```

### 4.2 切入点函数

1. execution
最为重要的切⼊点函数，功能最全。用于执⾏⽅法切⼊点表达式、类切⼊点表达式和包切⼊点表达式。
- 弊端：execution执⾏切⼊点表达式 ，书写麻烦
```execution(*com.baizhiedu.proxy..*.*(..))```
注意：其他的切⼊点函数 简化是execution书写复杂度，功能上完全⼀致。
2. args
- 作用：主要用于函数（方法）参数的匹配。
- 示例切入点为方法参数必须是两个字符串类型额参数

```execution(* *(String,String)) args(String,String) ```
3. within
- 作用：主要用于进行类、包切入点表达式的匹配
- 切入点：UserServiceImpl这个类，
```
execution(* *..UserServiceImpl.*(..))
within(*..UserServiceImpl)

execution(* com.baizhiedu.proxy..*.*(..))
within(com.baizhiedu.proxy..*)
```
4. @annotion(自定义注解)
- 作用：为具有特殊注解的方法加入额外功能
```
<aop:pointcut id="" expression="@annotation(com.baizhie
du.Log)"/>
```

5. 切入点函数的逻辑运算
- 指的是整合多个切入点函数一起配合工作，进而完成更为复杂的需求。
- and与操作 login 同时 参数 2个字符串
```
execution(* login(String,String))
execution(* login(..)) and args(String,String)
```
与操作不能用于同种类型的切入点函数。
- or 或操作
案例：register⽅法 和 login⽅法作为切⼊点
```
execution(* login(..)) or execution(* register(..))
```
## 5 AOP编程

### 5.1 概念

AOP (Aspect Oriented Programing) ⾯向切⾯编程 = Spring动态代理开发，以切⾯为基本单位的程序开发，通过切⾯间的彼此协同，相互调⽤，完成程序的构建

切⾯ = 切⼊点 + 额外功能

本质：Spring的动态代理开法，通过代理类为原始类增加额外功能。

核心问题：
- AOP如何创建动态代理类（动态字节码技术）
- Spring工厂如何加工创建代理对象（通过原始对象的id值，获得的是代理对象）

### 5.2 JDK动态代理

![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/AADB64BF9EF64ED0AC5619C2A9678CD8/5358)
![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/0BBC89D8BAE545A1B6F70B9CFAA13F01/5366)

### 5.3 CGlib的动态代理

CGlib创建动态代理的原理：⽗⼦继承关系创建代理对象，原始类作为⽗类，代理类作为⼦类，这样既可以保证2者⽅法⼀致，同时在代理类中提供新的实现(额外功能+原始⽅法)

![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/0BBC89D8BAE545A1B6F70B9CFAA13F01/5366)

> 总结：JDK动态代理Proxy.newProxyInstance()通过接⼝创建代理的实现类, Cglib动态代理Enhancer通过继承⽗类创建的代理类

### 5.4 Spring工厂如何加工原始对象

![](http://note.youdao.com/yws/public/resource/1f4682a8c41f4bbfefa91f24f452c92e/xmlnote/63A5C3AD55C4480F96FF67C4783BE9BE/5375)

## 6 基于注解的AOP编程

1. 原始对象
2. 额外功能
3. 切入点
4. 组装切面
```
# 通过切⾯类 定义了 额外功能
@Around
定义了 切⼊点
@Around("execution(*login(..))")
@Aspect 切⾯类
package com.baizhiedu.aspect;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;

/*
1. 额外功能
public class MyArround implements MethodInterceptor{
    public Object invoke(MethodInvocation invocation){
        Object ret = invocation.proceed();
        return ret;
    }
}
2. 切⼊点
<aop:config
<aop:pointcut id="" expression="execution(* login(..))"/>
*/
@Aspect
public class MyAspect {

@Around("execution(*login(..))")
public Object arround(ProceedingJoinPoint
joinPoint) throws Throwable {
    System.out.println("----aspect log ------");
    Object ret = joinPoint.proceed();
    return ret;
    }
}

<bean id="userService" class="com.baizhiedu.aspect.UserServiceImpl"/>

<bean id="arround" class="com.baizhiedu.aspect.MyAspect"/>
<!--告知Spring基于注解进⾏AOP编程-->
<aop:aspectj-autoproxy />
注意如果不使用xml，而是配置bean的话，需要在@aspect注解上加入@component("name")


可以使用@Pointcut注解

@Aspect
public class MyAspect {
    @Pointcut("execution(*login(..))")
    public void myPointcut() {}
    @Around(value="myPointcut()")
    public Object arround(ProceedingJoinPoint
joinPoint) throws Throwable {}
```