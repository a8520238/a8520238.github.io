---
title: springboot配置tomcat
date: 2021-2-28
cover:
top_img:
categories: 
    - java
    - javaEE
tags: 
mathjax: true
katex: true
---
# SpringBoot内置tomcat

## 1 依赖
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <version>2.1.6.RELEASE</version>
</dependency>

```

## 2 发布生产时的依赖
- 发布生产时要使用外置tomcat
- 大多数做法是排除内置的tomcat,打war包，然后部署在生产的tomcat中。打包时的处理如下：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 移除嵌入式tomcat插件 -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--添加servlet-api依赖--->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```
同时更新main函数，主要是继承SpringBootServletInitializer,并重写configure()方法。

```
@SpringBootApplication
public class MySpringbootTomcatStarter extends SpringBootServletInitializer {
    public static void main(String[] args) {
        Long time=System.currentTimeMillis();
        SpringApplication.run(MySpringbootTomcatStarter.class);
        System.out.println("===应用启动耗时："+(System.currentTimeMillis()-time)+"===");
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(this.getClass());
    }
}
```

## 3 入口
```
@SpringBootApplication
public class MySpringbootTomcatStarter{
    public static void main(String[] args) {
        Long time=System.currentTimeMillis();
        SpringApplication.run(MySpringbootTomcatStarter.class);
        System.out.println("===应用启动耗时："+(System.currentTimeMillis()-time)+"===");
    }
}
```
这里要注意@SpringBootApplication注解和SpringApplication.run()方法。

1. run方法

- 入口run方法

```
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class[]{primarySource}, args);
}

--这里run方法返回的是ConfigurableApplicationContext
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return (new SpringApplication(primarySources)).run(args);
}
```
- 入口run方法调用的实例run方法

```
public ConfigurableApplicationContext run(String... args) {
	ConfigurableApplicationContext context = null;
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
	this.configureHeadlessProperty();
	SpringApplicationRunListeners listeners = this.getRunListeners(args);
	listeners.starting();

	Collection exceptionReporters;
	try {
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
		this.configureIgnoreBeanInfo(environment);
		
		//打印banner，这里你可以自己涂鸦一下，换成自己项目的logo
		Banner printedBanner = this.printBanner(environment);
		
		//创建应用上下文
		context = this.createApplicationContext();
		exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);

		//预处理上下文
		this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
		
		//刷新上下文
		this.refreshContext(context);
		
		//再刷新上下文
		this.afterRefresh(context, applicationArguments);
		
		listeners.started(context);
		this.callRunners(context, applicationArguments);
	} catch (Throwable var10) {
		
	}

	try {
		listeners.running(context);
		return context;
	} catch (Throwable var9) {
		
	}
}
```
