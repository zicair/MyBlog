---

title: SpringBoot启动原理(运行)
tags:
  - SpringBoot
categories:
  - SpringBoot
date: 2020-07-31 13:34:03
---

通过断点调试，阅读源码查看SpringBoot的启动原理，启动过程分为创建对象和运行。这里介绍启动时，SpringBoot的创建出对象后执行run方法的运行过程。

<!--more-->

# 1. 启动流程

从main方法出设置断点，然后debug运行

![image-20200729153624362](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(运行)/image-20200729153624362.png)

调用SpringApplication的静态方法run，然后，对参数再包装再次调用重载run方法

![image-20200729153845855](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(运行)/image-20200729153845855.png)

调用SpringApplication的静态方法run，使用构造函数生成对象

![image-20200729153942629](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(运行)/image-20200729153942629.png)

主要流程：使用SpringApplication的静态方法，创建对象，执行run方法：

1. 创建`SpringApplication`对象
2. 执行`SpringApplication`对象的run方法

# 2. 创建对象

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	// 参数为resourceLoader=null primarySources=Application.class
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 2.1. 检查是不是web应用
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 2.2. 设置初始化器
    // 2.2.2 从类路径下META-INF/spring.factories搜索所有ApplicationContextInitializer
    setInitializers((Collection) getSpringFactoriesInstances(
        ApplicationContextInitializer.class));
    // 2.3. 设置监听器
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 2.4. 从所有的配置类中找到主配置类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

SpringBoot启动后，会先创建`SpringApplication`对象，创建过程参考上一篇文章：{% post_link SpringBoot启动原理(创建) %}

具体来说，创建过程分为以下几步，每一步又有各自的运行流程：

**1. 检查是不是web应用**

通过静态方法检查当前类加载器静态属性中是否包含web相应的关键字

**2. 设置初始化器**

从类路径下META-INF/spring.factories搜索所有ApplicationContextInitializer，并加入list中返回，传给当前的`this.initializers`

**3. 设置监听器**

即从类路径"META-INF/spring.factories"获取所有的监听器，并加入list中返回，传给当前类的`this.listeners`

**4. 从所有的配置类中找到主配置累**

将找到的主配置类传给当前的`this.mainApplicationClass`

![image-20200731145839407](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(运行)/image-20200731145839407.png)

# 3.  创建出的对象执行run()方法运行

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    // 3.1 获取SpringApplicationRunListeners；从类路径下META-INF/spring.factories
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 3.2 回调所有的获取SpringApplicationRunListeners。starting()方法
    listeners.starting();
    try {
        // 封装命令行参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
        // 3.3 准备环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners,                                                       applicationArguments);
        // 创建环境完成后回调SpringApplicationRunListeners.environmentPrepared(),表示环境准备完成
        
        configureIgnoreBeanInfo(environment);
        // 控制台打印图标
        Banner printedBanner = printBanner(environment);
        
        // 3.4 创建ApplicationContext；决定创建web的ioc还是普通的ioc，利用反射创建
        context = createApplicationContext();
        exceptionReporters = getSpringFactoriesInstances(
            SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);
        
        // 3.5 准备上下文环境；将environment保存到ioc中，并且applyInitializers();
        // applyInitializers();回调之前保存的所有ApplicationContextInitializer的initialize方法
        // 回调所有的SpringApplicationRunListener的contextPrepared();
        prepareContext(context, environment, listeners, applicationArguments,
                       printedBanner);
        // prepareContext运行完以后回调所有的SpringApplicationRunListener的contextLoaded();
        
        // 3.6 刷新IOC容器
        refreshContext(context);
        // 空方法
        afterRefresh(context, applicationArguments);
        // 所有的SpringApplicationRunListener回调finished()方法
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                .logStarted(getApplicationLog(), stopWatch);
        }
        // 所有的SpringApplicationRunListener回调started()方法
        listeners.started(context);
        
        // 回调ApplicationRunner， 回调CommandLineRunner
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    // 返回IOC容器
    return context;
}
```

## 3.1 获取SpringApplicationRunListeners

```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
    Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
    return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
        SpringApplicationRunListener.class, types, this, args));
}
```

其中的getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args)也是通过类路径下去找到names，根据names去创建实例，与前面不同的是，前面获取的是ApplicationContextInitializer和ApplicationListener，这里获取的是SpringApplicationRunListener

```java
SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners) {
    this.log = log;
    this.listeners = new ArrayList<>(listeners);
}
```

## 3.2 listeners.starting()

查看SpringApplicationRunListener接口有哪些方法：

![image-20200731155719953](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(运行)/image-20200731155719953.png)

对于获得的所有SpringApplicationRunListener对象，都执行相应的staring()方法；

```java
public void starting() {
    for (SpringApplicationRunListener listener : this.listeners) {
        listener.starting();
    }
}
```
## 3.3 准备环境

创建环境完成后回调SpringApplicationRunListeners.environmentPrepared(),表示环境准备完成

```java
	private ConfigurableEnvironment prepareEnvironment(
			SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// Create and configure the environment
		ConfigurableEnvironment environment = getOrCreateEnvironment();
		configureEnvironment(environment, applicationArguments.getSourceArgs());
        // 回调SpringApplicationRunListeners.environmentPrepared方法
		listeners.environmentPrepared(environment);
		bindToSpringApplication(environment);
        // 判断是不是web引用，是的话进行转换
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader())
					.convertEnvironmentIfNecessary(environment, deduceEnvironmentClass());
		}
		ConfigurationPropertySources.attach(environment);
		return environment;
	}
```

## 3.4 创建ApplicationContext(IOC)

```java
	protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
                // 判断是否创建为web IOC容器
				switch (this.webApplicationType) {
				case SERVLET:
					contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
					break;
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, "
								+ "please specify an ApplicationContextClass",
						ex);
			}
		}
        // 利用反射创建
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}
```

## 3.5 准备上下文环境

```java
	private void prepareContext(ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
		context.setEnvironment(environment);
		postProcessApplicationContext(context);
        
        // applyInitializers();回调之前保存的所有ApplicationContextInitializer的initialize方法
		applyInitializers(context);
        
        // 回调所有的SpringApplicationRunListener的contextPrepared();
		listeners.contextPrepared(context);
        
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[0]));
        
        // 回调所有的SpringApplicationRunListener的contextLoaded();
		listeners.contextLoaded(context);
	}
```

## 3.6 刷新IOC容器

refreshContext(context)：完成IOC的刷新，扫描、刷新、创建IOC组件
afterRefresh(context, applicationArguments)：空方法
listeners.started(context)：所有的SpringApplicationRunListener回调started()方法

    // 从容器中获取并回调ApplicationRunner， 回调CommandLineRunner
    callRunners(context, applicationArguments);
```java
	private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<>();
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
		runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
		AnnotationAwareOrderComparator.sort(runners);
		for (Object runner : new LinkedHashSet<>(runners)) {
			if (runner instanceof ApplicationRunner) {
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}
```

