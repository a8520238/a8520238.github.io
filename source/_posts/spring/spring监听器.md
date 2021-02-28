---
title: Spring监听器
date: 2021-2-28
cover:
top_img:
categories: spring
tags: 
mathjax: true
katex: true
---
# 1 事件监听机制

有三个角色： 事件， 事件监听器，事件发布器

事件发布器是事件监听器的包装

1.  事件及事件源:对应于观察者模式中的主题
2.  事件监听器:对应于观察者模式中的观察者。监听器监听特定事件,并在内部定义了事件发生后的响应逻辑。
3.  事件发布器:事件监听器的容器,对外提供发布事件和增删事件监听器的接口,维护事件和事件监听器之间的映射关系,并在事件发生时负责通知相关监听器。

事件发布器调用内部函数，将事件派发到需要相应的监听器

# 2 Spring中的监听器

1. 事件
   1. Spring为容器内事件定义了一个抽象类ApplicationEvent,该类继承了JDK中的事件基类EventObject。因而自定义容器内事件除了需要继承ApplicationEvent之外,还要传入事件源作为构造参数。

2. 事件监听器

   1. Spring定义了一个ApplicationListener接口作为为事件监听器的抽象,接口定义如下

      ```
      public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
      
         /**
          * Handle an application event.
          * @param event the event to respond to
          */
         void onApplicationEvent(E event);
      
      }
      ```

      

3. ApplicationContext接口继承了ApplicationEventPublisher接口,从而提供了对外发布事件的能, 而真正的事件发布器是ApplicationContext中的ApplicationEventMulticaster,默认实现类是SimpleApplicationEventMulticaster，该组件会在容器启动的时候自动创建，单例，管理所有事件监听器，提供对所有容器内事件的发布功能。

# 3 Spring中的监听

1. 自定义事件

   ```
   public class TaskFinishEvent2 extends ApplicationEvent {
       /**
        * Create a new ApplicationEvent.
        *
        * @param source the object on which the event initially occurred (never {@code null})
        */
       public TaskFinishEvent2(Object source) {
           super(source);
       }
   }
   ```

   

2. 自定义事件监听器并向容器注册

   ```
   @Component
   public class MailTaskFinishListener2 implements ApplicationListener<TaskFinishEvent2> {
   
       private String emial="takumiCX@163.com";
       
       @Override
       public void onApplicationEvent(TaskFinishEvent2 event) {
           
           System.out.println("Send Emial to "+emial+" Task:"+event.getSource());
           
       }
   ```

   

3. 发布事件

   ```
   @Component
   public class EventPublisher implements ApplicationContextAware {
   
       private ApplicationContext applicationContext;
   
       //发布事件
       public void publishEvent(ApplicationEvent event){
   
           applicationContext.publishEvent(event);
       }
   
       @Override
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           this.applicationContext=applicationContext;
       }
   }
   ```

   

# 4 Spring事件监听源码解析

> Spring事件监听机制离不开容器IOC特性提供的支持,比如容器会自动创建事件发布器,自动识别用户注册的监听器并进行管理,在特定的事件发布后会找到对应的事件监听器并对其监听方法进行回调。

1. 事件发布器ApplicationEventMulticaster是何时被初始化的,初始化过程中都做了什么？
2.  注册事件监听器的过程是怎样的,容器怎么识别出它们并进行管理?
3. 容器发布事件的流程是怎样的?它如何根据发布的事件找到对应的事件监听器,事件和由该事件触发的监听器之间的匹配规则是怎样的？

## 4.1 初始化事件发布器

真正的事件发布器是ApplicationEventMulticaster,它定义在AbstractApplicationContext中,并在ApplicationContext容器启动的时候进行初始化。在容器启动的refrsh()方法中可以找到初始化事件发布器的入口方法

在refresh中的initApplicationEventMulticaster()方法进行初始化

```
protected void initApplicationEventMulticaster() {
  ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  // 判断beanFactory里是否定义了id为applicationEventMulticaster的bean,默认是没有的
  if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
   this.applicationEventMulticaster =
     beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
   if (logger.isDebugEnabled()) {
    logger.debug("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
   }
  }
  else {
       //一般情况会走这里,创建一个SimpleApplicationEventMulticaster并交由容器管理
   this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
   beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
   if (logger.isDebugEnabled()) {
    logger.debug("Unable to locate ApplicationEventMulticaster with name '" +
      APPLICATION_EVENT_MULTICASTER_BEAN_NAME +
      "': using default [" + this.applicationEventMulticaster + "]");
   }
  }
 }
```

> Executor和错误处理器ErrorHandler的功能,当设置Executor为线程池时,则会以异步的方式对事件监听器进行回调,而ErrorHandler允许我们在回调方法执行错误时进行自定义处理。默认情况下，
> 这两个变量都为null。

```
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {

   private Executor taskExecutor;

   private ErrorHandler errorHandler;
```

```
public abstract class AbstractApplicationEventMulticaster
      implements ApplicationEventMulticaster, BeanClassLoaderAware, BeanFactoryAware {

   private final ListenerRetriever defaultRetriever = new ListenerRetriever(false);

   final Map<ListenerCacheKey, ListenerRetriever> retrieverCache =
         new ConcurrentHashMap<ListenerCacheKey, ListenerRetriever>(64);

   private ClassLoader beanClassLoader;

   private BeanFactory beanFactory;

   private Object retrievalMutex = this.defaultRetriever;
```

之后会调用beanFactory.registerSingleton方法将创建的SimpleApplicationEventMulticaster实例注册为容器的单实例bean。

初始化事件发布器的工作非常简单,一句话总结:由容器实例化用户自定义的事件发布器或者由容器帮我们创建一个简单的事件发布器并交由容器管理。



## 4.2 注册事件监听器流程

在OnRefresh()后的registerListeners()中进行注册

```
// uninitialized to let post-processors apply to them!
		//获取实现ApplicationListener接口的所有bean的beanName
		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
		for (String listenerBeanName : listenerBeanNames) {
			//将监听器的beanName保存到事件发布器中
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		}
```

```
@Override
public void addApplicationListenerBean(String listenerBeanName) {
   synchronized (this.retrievalMutex) {
   	  //保存所有监听器的beanName
      this.defaultRetriever.applicationListenerBeans.add(listenerBeanName);
      //清除维护事件和监听器映射关系的缓存
      this.retrieverCache.clear();
   }
}
```

defaultRetriever如下

```
	/**
	 * Helper class that encapsulates a specific set of target listeners,
	 * allowing for efficient retrieval of pre-filtered listeners.
	 * <p>An instance of this helper gets cached per event type and source type.
	 */
	private class ListenerRetriever {

		//保存所有事件监听器
		public final Set<ApplicationListener<?>> applicationListeners;

		//保存所有事件监听器的beanName
		public final Set<String> applicationListenerBeans;
```

上述只是找到了类名， 获得对象如下：

ApplicationListenerDetector实现了BeanPostProcessor接口,可以在容器级别对所有bean的生命周期过程进行增强。这里主要是为了能够在初始化所有bean后识别出所有的事件监听器bean并
将其注册到事件发布器中

```
@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) {

		//判断该bean是否实现了ApplicationListener接口
		if (this.applicationContext != null && bean instanceof ApplicationListener) {
			// potentially not detected as a listener by getBeanNamesForType retrieval
			Boolean flag = this.singletonNames.get(beanName);
			if (Boolean.TRUE.equals(flag)) {
				// singleton bean (top-level or inner): register on the fly
				//将实现了ApplicationListener接口的bean注册到事件发布器applicationEventMulticaster中
				this.applicationContext.addApplicationListener((ApplicationListener<?>) bean);
			}
			else if (Boolean.FALSE.equals(flag)) {
				if (logger.isWarnEnabled() && !this.applicationContext.containsBean(beanName)) {
					// inner bean with other scope - can't reliably process events
					logger.warn("Inner bean '" + beanName + "' implements ApplicationListener interface " +
							"but is not reachable for event multicasting by its containing ApplicationContext " +
							"because it does not have singleton scope. Only top-level listener beans are allowed " +
							"to be of non-singleton scope.");
				}
				this.singletonNames.remove(beanName);
			}
		}
		return bean;
	}
```



## 4.3 容器事件发布流程

在publishEvent方法中调用核心方法multicastEvent()，需要传入事件event和事件类型
eventType作为参数。

```
@Override
	public void multicastEvent(final ApplicationEvent event, ResolvableType eventType) {
		//获取事件类型
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		//遍历所有和事件匹配的事件监听器
		for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			//获取事件发布器内的任务执行器,默认该方法返回null
			Executor executor = getTaskExecutor();
			if (executor != null) {
                                //异步回调监听方法
				executor.execute(new Runnable() {
					@Override
					public void run() {
						invokeListener(listener, event);
					}
				});
			}
			else {
				//同步回调监听方法
				invokeListener(listener, event);
			}
		}
	}
```

获取匹配的监听器getApplicationListensers方法

```
protected Collection<ApplicationListener<?>> getApplicationListeners(
			ApplicationEvent event, ResolvableType eventType) {
        //获取事件中的事件源对象
		Object source = event.getSource();
		//获取事件源类型
		Class<?> sourceType = (source != null ? source.getClass() : null);
		//以事件类型和事件源类型为参数构建一个cacheKey,用于从缓存map中获取与之匹配的监听器列表
		ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);

		// Quick check for existing entry on ConcurrentHashMap...
		ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
		if (retriever != null) {
		    //从缓存中获取监听器列表
			return retriever.getApplicationListeners();
		}

		if (this.beanClassLoader == null ||
				(ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) &&
						(sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
			// Fully synchronized building and caching of a ListenerRetriever
			synchronized (this.retrievalMutex) {
				retriever = this.retrieverCache.get(cacheKey);
				if (retriever != null) {
					return retriever.getApplicationListeners();
				}
				retriever = new ListenerRetriever(true);
				//查找所有与发布事件匹配的监听器列表
				Collection<ApplicationListener<?>> listeners =
						retrieveApplicationListeners(eventType, sourceType, retriever);
				//将匹配结果缓存到map中
				this.retrieverCache.put(cacheKey, retriever);
				return listeners;
			}
		}
		else {
			// No ListenerRetriever caching -> no synchronization necessary
			return retrieveApplicationListeners(eventType, sourceType, null);
		}
	}
```

如果没有缓存则遍历

```
* Actually retrieve the application listeners for the given event and source type.
	 * @param eventType the event type
	 * @param sourceType the event source type
	 * @param retriever the ListenerRetriever, if supposed to populate one (for caching purposes)
	 * @return the pre-filtered list of application listeners for the given event and source type
	 */
	private Collection<ApplicationListener<?>> retrieveApplicationListeners(
			ResolvableType eventType, Class<?> sourceType, ListenerRetriever retriever) {

		//这是存放匹配的监听器的列表
		LinkedList<ApplicationListener<?>> allListeners = new LinkedList<ApplicationListener<?>>();
		Set<ApplicationListener<?>> listeners;
		Set<String> listenerBeans;
		synchronized (this.retrievalMutex) {
			listeners = new LinkedHashSet<ApplicationListener<?>>(this.defaultRetriever.applicationListeners);
			listenerBeans = new LinkedHashSet<String>(this.defaultRetriever.applicationListenerBeans);
		}
		//遍历所有的监听器
		for (ApplicationListener<?> listener : listeners) {
			//判断该事件监听器是否匹配
			if (supportsEvent(listener, eventType, sourceType)) {
				if (retriever != null) {
					retriever.applicationListeners.add(listener);
				}
				//将匹配的监听器加入列表
				allListeners.add(listener);
			}
		}
		//这部分可以跳过
		if (!listenerBeans.isEmpty()) {
			BeanFactory beanFactory = getBeanFactory();
			for (String listenerBeanName : listenerBeans) {
				try {
					Class<?> listenerType = beanFactory.getType(listenerBeanName);
					if (listenerType == null || supportsEvent(listenerType, eventType)) {
						ApplicationListener<?> listener =
								beanFactory.getBean(listenerBeanName, ApplicationListener.class);
						if (!allListeners.contains(listener) && supportsEvent(listener, eventType, sourceType)) {
							if (retriever != null) {
								retriever.applicationListenerBeans.add(listenerBeanName);
							}
							allListeners.add(listener);
						}
					}
				}
				catch (NoSuchBeanDefinitionException ex) {
					// Singleton listener instance (without backing bean definition) disappeared -
					// probably in the middle of the destruction phase
				}
			}
		}
		//对匹配的监听器列表进行排序
		AnnotationAwareOrderComparator.sort(allListeners);
		return allListeners;
	}
```

判断监听器是否匹配的逻辑在supportsEvent(listener, eventType, sourceType)中,

```
protected boolean supportsEvent(ApplicationListener<?> listener, ResolvableType eventType, Class<?> sourceType) {
		//对原始的ApplicationListener进行一层适配器包装成为GenericApplicationListener
		GenericApplicationListener smartListener = (listener instanceof GenericApplicationListener ?
				(GenericApplicationListener) listener : new GenericApplicationListenerAdapter(listener));
				//判断监听器是否支持该事件类型以及该事件源类型
		return (smartListener.supportsEventType(eventType) && smartListener.supportsSourceType(sourceType));
	}
```

首先对原始的ApplicationListener进行一层适配器包装成GenericApplicationListener,便于后面使用该接口中定义的方法判断监听器是否支持传入的事件类型或事件源类型

```
public interface GenericApplicationListener extends ApplicationListener<ApplicationEvent>, Ordered {

   /**
    * Determine whether this listener actually supports the given event type.
    */
   boolean supportsEventType(ResolvableType eventType); //判断是否支持该事件类型

   /**
    * Determine whether this listener actually supports the given source type.
    */
   boolean supportsSourceType(Class<?> sourceType);   //判断是否支持该事件源类型

}
```

declaredEventType是监听器泛型的实际类型,而eventType是发布的事件的类型
declaredEventType.isAssignableFrom(eventType)当以下两种情况返回true

- 1.declaredEventType和eventType类型相同
- 2.declaredEventType是eventType的父类型
  只要监听器泛型的实际类型和发布的事件类型一样或是它的父类型,则该监听器将被成功匹配。

而对于事件源类型而言,通常默认会直接返回true,也就是说事件源的类型通常对于判断匹配的监听器没有意义。

这里查找到所有匹配的监听器返回后会将匹配关系缓存到retrieverCache这个map中

```
里查找到所有匹配的监听器返回后会将匹配关系缓存到retrieverCache这个map中

				Collection<ApplicationListener<?>> listeners =
						retrieveApplicationListeners(eventType, sourceType, retriever);
				//将匹配结果缓存到map中
				this.retrieverCache.put(cacheKey, retriever);
				return listeners;

```

