---
title: springmvc 0xml配置
date: 2021-2-28
cover:
top_img:
categories: springmvc
tags: 
mathjax: true
katex: true
---
# 0xml非springboot配置springmvc

## 1 JavaConfig

```
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    //web容器会在启动的时候去调用onStartup(),web容器就是tomcat。ServletContext,web容器的上下文。web组件包括filter,servlet,listener。
    
    //servlet3.0 版本之后，提出的新规范SPI
    //简单来说，项目里面如果有某些类或者某些方法，需要启动的时候，被Tomcat调用的话，首先在项目的根目录的META-INF/services/javax.servlet.ServletContainerInitializer目录下建立一个文件。
    @Override
    public void onStartup(ServletContext servletCxt) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        //注册appconfig.class
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

通过Spring-web 启动onStartup的原理
1. 在spring-web包的META-INF下有，javax.servlet.ServletConrainerInitializer文件。这里有org.springframework.web.SpringServletContainerInitializer，也就是Spring用这个类实现的启动初始。
2. 在org.springframework.web.SpringServletContainerInitializer中，SpringServletContainerInitializer实现了ServletContainerInitializer接口。这里面有onStartup方法。在实现方法总，有一个Set<Class<?> webAppInitializerClasses>，这时候在实现类上有@HandlesTypes(WebApplicationInitial.class)注解，会把所有这个接口的实现类扫到集合中，spring会遍历set集合，new对象存到list集合中，再调用使用onstarup方法。
> 也就是说spring实现了servlet的接口，达到了tomcat启动时的调用过程。

## 2 jar包和war包

jar包内置tomcat

war包没有内置tomcat


## 3 Spring MVC 找Controller流程
1. 扫描整个项目（Spring已经做了）
2. 拿到所有加了@Controller注解的类
3. 遍历类里面所有的方法对象
4. 判断方法是否加了@RequestMapping注解
5. 把RequestMapping注解的value 做为map集合的key ,put进去，把method对象(反射)作为value放入map集合
6. 根据用户发送的请求，拿到请求中的URI
7. 使用请求的uri 最为map的key 去map里get看是否有返回值，返回一个方法对象。

## 4 controller的定义方式

有2种类型和3种实现

- 2种类型 BeanName类型 和 @Controller 类型
- 3种实现 HttpRequestHandler 实现Controller 加@Controller 获得适配器


赋值参数过程中使用了策略模式

![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/CBC2BCBAF1274D92A9EB4D03C535A0FF/5646)

![](http://note.youdao.com/yws/public/resource/8c1c4f1ba9be828413b85b4a9353e358/xmlnote/30D96177AE364F05AB57125D1F3457DD/5648)

# todo

spi