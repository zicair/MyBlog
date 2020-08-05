---
title: SpringBoot启动原理(创建)
tags:
  - SpringBoot
categories:
  - SpringBoot
date: 2020-07-29 15:31:48

---

通过断点调试，阅读源码查看SpringBoot的启动原理，启动过程分为创建对象和运行。这里介绍启动时，SpringBoot的创建对象的过程。

<!--more-->

# 1. 启动流程

从main方法出设置断点，然后debug运行

![image-20200729153624362](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729153624362.png)

调用SpringApplication的静态方法run，然后，对参数再包装再次调用重载run方法

![image-20200729153845855](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729153845855.png)

调用SpringApplication的静态方法run，使用构造函数生成对象

![image-20200729153942629](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729153942629.png)

主要流程：使用SpringApplication的静态方法，创建对象，执行run方法：

1. 创建`SpringApplication`对象
2. 执行`SpringApplication`对象的run方法

# 2. 创建`SpringApplication`对象

前面首先是通过`SpringApplication`的静态方法，new出对象再执行run，这里查看创建`SpringApplication`对象时做了哪些事情

首先是new的过程中显示对类的属性初始化，然后在构造器中调用`SpringApplication`重载的构造器

![image-20200729160142458](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729160142458.png)

进入重载的构造器中：

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

## 2.1 检查是不是web应用

调用`WebApplicationType`的静态方法判断当前应用是不是web应用

![image-20200729162148263](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729162148263.png)



## 2.2 设置初始化器

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

设置初始化器先是通过getSpringFactoriesInstances(ApplicationContextInitializer.class)获得结果，然后将结果保存到this.initializers

```java
public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {
    this.initializers = new ArrayList<>();
    this.initializers.addAll(initializers);
}
```

### 2.2.1 调用getSpringFactoriesInstances(ApplicationContextInitializer.class)

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}
```

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = getClassLoader();
    // Use names and ensure unique to protect against duplicates
    // 2.2.2 先获取names
    Set<String> names = new LinkedHashSet<>(
        SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 2.2.3 通过names创建初始化器，排序返回
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
                                                       classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

### 2.2.2 获取names

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}
```

![image-20200729164241607](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729164241607.png)

上面类加载器搜索的路径`FACTORIES_RESOURCE_LOCATION`为：

```java
public final class SpringFactoriesLoader {
	public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```

可以看出他是从类路径下META-INF/spring.factories搜索所有ApplicationContextInitializer

### 2.2.3 通过names创建初始化器，排序返回

![image-20200729170931608](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729170931608.png)

## 2.3 设置监听器

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

设置监听器和设置初始化器的步骤类似，调用getSpringFactoriesInstances(ApplicationListener.class)方法，然后将得到的结果存在this.listeners中

```java
public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
    this.listeners = new ArrayList<>();
    this.listeners.addAll(listeners);
}
```

### 2.3.1 调用getSpringFactoriesInstances(ApplicationListener.class)

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}
```

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = getClassLoader();
    // Use names and ensure unique to protect against duplicates
    // 2.3.2 获取names
    Set<String> names = new LinkedHashSet<>(
        SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 2.3.3 根据names创建监听器，排序返回
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
                                                       classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

### 2.3.2 获取names

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}
```

![image-20200729170513634](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729170513634.png)

这里的FACTORIES_RESOURCE_LOCATION="META-INF/spring.factories"

```java
public final class SpringFactoriesLoader {
    public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```

即从类路径"META-INF/spring.factories"获取所有的监听器

### 2.3.3 根据names创建监听器，排序返回

同样调用createSpringFactoriesInstances()创建实例

![image-20200729171246665](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/SpringBoot启动原理(创建)/image-20200729171246665.png)

## 2.4 从所有的配置类中找到主配置类

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

private Class<?> deduceMainApplicationClass() {
    try {
        StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
        for (StackTraceElement stackTraceElement : stackTrace) {
            if ("main".equals(stackTraceElement.getMethodName())) {
                return Class.forName(stackTraceElement.getClassName());
            }
        }
    }
    catch (ClassNotFoundException ex) {
        // Swallow and continue
    }
    return null;
}
```

到这里就完成了SpringBoot启动时创建对象的过程，通过创建出的这个对象，执行它的run()方法。参见：{% post_link SpringBoot启动原理(运行) %}

