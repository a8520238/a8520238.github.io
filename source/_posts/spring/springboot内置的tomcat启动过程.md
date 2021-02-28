---
title: springboot内置的tomcat启动过程
date: 2021-2-28
cover:
top_img:
categories: springboot
tags: 
mathjax: true
katex: true
---
# SpringBoot tomcat启动


## 内置的tomcat
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <version>2.1.6.RELEASE</version>
</dependency>
@SpringBootApplication
public class MySpringbootTomcatStarter{
    public static void main(String[] args) {
        Long time=System.currentTimeMillis();
        SpringApplication.run(MySpringbootTomcatStarter.class);
        System.out.println("===应用启动耗时："+(System.currentTimeMillis()-time)+"===");
    }
}

```

上面是main函数入口，两个关键点，分别是`SpringBootApplication`注解和`SpringApplication.run()`方法。

## 发布生产的tomcat

发布的时候，目前大多数的做法还是排除内置的tomcat，打瓦包（war）然后部署在生产的tomcat中。

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
更新main函数，主要是继承SpringBootServletInitializer，并重写configure()方法。

```@SpringBootApplication
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

从main函数说起

```
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class[]{primarySource}, args);
}

--这里run方法返回的是ConfigurableApplicationContext
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return (new SpringApplication(primarySources)).run(args);
}
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

既然我们想知道tomcat在SpringBoot中是怎么启动的，那么run方法中，重点关注创建应用上下文（createApplicationContext）和刷新上下文（refreshContext）。

[SpringBoot内置tomcat启动原理](https://www.cnblogs.com/sword-successful/p/11383723.html)