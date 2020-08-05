---
title: Java中的synchronized
date: 2020-06-12 21:12:44
tags: [Java, Synchronized]
categories:
- [Java, Synchronized]
---

Java并发编程中的Synchronized的性质，并比较和Lock的区别。

<!--more-->

# 1. 并发编程中的三个问题

- 可见性：一个线程对共享变量的修改，另一个线程不能立即得到最新值。
- 原子性：在一次或多次操作中，要么所有的操作都执行并且不受其他因素干扰而中断，要么所有的操作都不执行。
- 有序性：是指程序中的代码执行顺序，Java在编译 时和运行时会对代码进行优化，导致程序最终的执行顺序不一定就是我们编写代码的顺序。

# 2. Java内存模型

Java Memory Model （Java内存模型/JMM）是java虚拟机规范中定义的一种内存模型，java内存模型是标准化的，屏蔽掉了底层不同计算机的区别。

java内存模型是一套规范，描述了java程序中各种变量（线程共享变量）的访问规则，以及在jvm中将变量存储到内存和从内存中读取变量这样的底层细节。具体如下：

- 主内存：是所有线程共享的，都能访问的。所有的共享变量都存储在主内存。

- 工作内存：每一个线程都有自己的工作内存，工作内存只存储该线程对共享变量的副本。线程对变量的所有操作都必须在工作内存中完成，而不能直接读取主内存中的变量。不同线程之间也不能直接访问对方工作内存中的变量。

  ![Java中的synchronized_1](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Java中的synchronized/Java中的synchronized_1.png)

# 3. synchronized的特性

- 可重入特性：同步代码块中可继续包含带有synchronized的同步代码块。锁对象有一个计数器（recursions变量）会记录线程获得几次锁。可重入特性可避免死锁且能更好地封装代码（同步代码块中可以直接调用带有synchronized的方法）。
- 不可中断特性：一个线程获得锁后，另一个线程想要获得锁必须处于阻塞或者等待，如果第一个线程不释放锁，第二个线程会一直等待或阻塞，不可被中断。

要看synchronized的原理，但是synchronized是一个关键字，看不到源码，可以将编译好的class文件进行反汇编。

JDK自带的一个工具：`javap`，对字节码进行反汇编，查看字节码指令。

在DOS命令行输入命令：![Java中的synchronized_2](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Java中的synchronized/Java中的synchronized_2.png)

`-p`显示所有的包括私有的、`v`详细显示![Java中的synchronized_3](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Java中的synchronized/Java中的synchronized_3.png)

**由源码可以看出，synchronized出现异常时会自动释放锁：**

![Java中的synchronized_4](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Java中的synchronized/Java中的synchronized_4.png)

# 4. synchronized和Lock的区别

1. synchronized是关键字，而Lock是一个接口；
2. synchronized会自动释放锁，而Lock必须手动释放锁；
3. synchronized是不可中断的，Lock可以中断（lock.tryLock）也可以不中断（lock）；
4. 通过Lock可以知道线程有没有拿到锁（lock.tryLock返回Boolean值），而synchronized不能；
5. synchronized可以锁住方法和代码块，而Lock只能锁住代码块；
6. Lock可以使用读锁提高多线程读效率（Lock的实现类ReentrantReadWriteLock）；
7. synchronized是非公平锁，释放锁时随机唤醒。Lock = ReentrantLock（带参）可以控制是否是公平的；

# 5. 使用synchronized的建议：

- **减少synchronized的范围**

  同步代码块中尽量短，减少同步代码块中代码的执行时间，减少锁的竞争。

- **降低synchronized锁的粒度**

  将一个锁拆分为多个锁提高并发度。`Hashtable`锁定整个哈希表，一次只能一个线程对其操作，效率低；`ConcurrentHashMap`局部锁定，只锁定当前桶，其他线程可以访问别的桶中数据。

- **读写分离**

  读取时不加锁，写入和删除时加锁。

  `ConcurrentHashMap`， `CopyOnWriteArrayList`和`CopyOnWriteArraySet`

# 6. 扩展：

`java.util.Hashtable` and `Collections.synchronizedMap(new HashMap())` are synchronized. But [`ConcurrentHashMap`](ConcurrentHashMap.html) is "concurrent". A concurrent collection is thread-safe, but not governed by a single exclusion lock.