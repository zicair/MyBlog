---
title: synchronized底层monitor原理
date: 2020-07-13 22:23:06
tags: [Java, Synchronized]
categories:
- [Java, Synchronized]
---

synchronized底层使用monitor来控制锁的活动。了解monitor中的各个属性值的含义，锁的竞争流程。

<!--more-->

# synchronized底层monitor原理

### 1. monitor监视器锁

无论是synchronized代码块还是synchronized方法，其线程安全的语义实现最终依赖于monitor。

在HotSpot虚拟机中，monitor是由ObjectMonitor实现的。源码由C++实现，位于HotSpot虚拟机源码ObjectMonitor.hpp文件中(src/share/vm/runtime/objectMonitor.hpp)。ObjectMonitor主要数据结构如下：

![image-20200713223837588](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713223837588.png)

- _owner: 初始时为NULL。当有线程占有该monitor时，owner标记为该线程的唯一标识。当线程释放monitor时，owner又恢复为NULL。owner是一个临界资源，JVM是通过CAS操作来保证其线程安全。
- _cxq: 竞争队列，所有请求锁的线程首先会被放在这个队列中(单向列表)。cxq是一个临界资源，JVM通过CAS原子指令来修改cxq队列。修改前cxq的旧值填入了node的next字段，cxq指向新值(新线程)。因此cxq是一个后进先出的stack（栈）。
- _EntryList：cxq队列中有资格成为候选资源的线程会被移动到该队列中。 
- _WaitSet：因为调用wait方法而被阻塞的线程会被放在该队列中。

---

每一个Java对象都可以与一个监视器monitor关联，我们可以把它理解成为一把锁，当一个线程想要执行一段被synchronized圈起来的同步方法或者代码块时，该线程得先获取到synchronized修饰的对象对应的monitor。

我们的Java代码里不会显示地去创造这么一个monitor对象，我们也无需创建，事实上可以这么理解： monitor并不是随着对象创建而创建的。我们是通过synchronized修饰符告诉JVM需要为我们的某个对象创建关联的monitor对象。每个线程都存在两个ObjectMonitor对象列表，分别为free和used列表。 同时JVM中也维护着global locklist。当线程需要ObjectMonitor对象时，首先从线程自身的free表中申请，若存在则使用若不存在则从global list中申请。

ObjectMonitor的数据结构、以及关系：

![image-20200713224950456](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713224950456.png)

### 2. monitor竞争

1. 执行monitorenter时，会调用InterpreterRuntime.cpp (src/share/vm/interpreter/interpreterRuntime.cpp) 的 InterpreterRuntime::monitorenter函数。具体代码可参见HotSpot源码。

   ![image-20200713231543477](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713231543477.png)

2. 对于重量级锁，monitorenter函数中会调用 ObjectSynchronizer::slow_enter

3. 最终调用 ObjectMonitor::enter（src/share/vm/runtime/objectMonitor.cpp），源码如下：

   ![image-20200713232235386](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713232235386.png)

   ![image-20200713232302247](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713232302247.png)

此处省略锁的自旋优化等操作。以上代码的具体流程概括如下：

- 通过CAS尝试把monitor的owner字段设置为当前线程。
- 如果设置之前的owner指向当前线程，说明当前线程再次进入monitor，即重入锁，执行 recursions ++ ，记录重入的次数。
- 如果当前线程是第一次进入该monitor，设置recursions为1，_owner为当前线程，该线程成功获得锁并返回。
- 如果获取锁失败，则等待锁的释放。使用EnterI()方法

### 3. monitor等待

竞争失败等待调用的是ObjectMonitor对象的EnterI方法（src/share/vm/runtime/objectMonitor.cpp），源码如下所示：

![image-20200713233845928](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713233845928.png)

![image-20200713233808086](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713233808086.png)

![image-20200713233919806](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713233919806.png)

当该线程被唤醒时，会从挂起的点继续执行，通过 ObjectMonitor::TryLock 尝试获取锁，TryLock方
法实现如下：  

![image-20200713234011498](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713234011498.png)

以上代码的具体流程概括如下：

- 当前线程被封装成ObjectWaiter对象node，状态设置成ObjectWaiter::TS_CXQ。
- 在for循环中，通过CAS把node节点push到_cxq列表中，同一时刻可能有多个线程把自己的node
  节点push到cxq列表中。
- node节点push到cxq列表之后，通过自旋尝试获取锁，如果还是没有获取到锁，则通过park将当
  前线程挂起，等待被唤醒。
- 当该线程被唤醒时，会从挂起的点继续执行，通过 ObjectMonitor::TryLock 尝试获取锁  

### 4. monitor释放

当某个持有锁的线程执行完同步代码块时，会进行锁的释放，给其它线程机会执行同步代码，在
HotSpot中，通过退出monitor的方式实现锁的释放，并通知被阻塞的线程，具体实现位于
ObjectMonitor的exit方法中。（src/share/vm/runtime/objectMonitor.cpp），源码：  

![image-20200713234402744](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713234402744.png)

![image-20200713234423895](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713234423895.png)

![image-20200713234455316](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713234455316.png)

- 退出同步代码块时会让recursions减1，当recursions的值减为0时，说明线程释放了锁。
- 根据不同的策略（由QMode指定），从cxq或EntryList中获取头节点，通过ObjectMonitor::ExitEpilog 方法唤醒该节点封装的线程，唤醒操作最终由unpark完成：  

![image-20200713234643397](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713234643397.png)

被唤醒的线程，会回到 void ATTR ObjectMonitor::EnterI (TRAPS) 的第600行，继续执行monitor
的竞争。

### 5. monitor是重量级锁

可以看到ObjectMonitor的函数调用中会涉及到Atomic::cmpxchg_ptr，Atomic::inc_ptr等内核函数，
执行同步代码块，没有竞争到锁的对象会park()被挂起，竞争到锁的线程会unpark()唤醒。这个时候就
会存在操作系统用户态和内核态的转换，这种切换会消耗大量的系统资源。所以synchronized是Java语
言中是一个重量级(Heavyweight)的操作。
要了解用户态和内核态需要先了解Linux系统的体系架构：  

![image-20200713235504342](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/synchronized底层monitor原理/image-20200713235504342.png)

从上图可以看出，Linux操作系统的体系架构分为：用户空间（应用程序的活动空间）和内核。
**内核：**本质上可以理解为一种软件，控制计算机的硬件资源，并提供上层应用程序运行的环境。
**用户空间：**上层应用程序活动的空间。应用程序的执行必须依托于内核提供的资源，包括CPU资源、存
储资源、I/O资源等。
**系统调用：**为了使上层应用能够访问到这些资源，内核必须为上层应用提供访问的接口：即系统调用。  

所有进程初始都运行于用户空间，此时即为用户运行状态（简称：用户态）；但是当它调用系统调用执
行某些操作时，例如 I/O调用，此时需要陷入内核中运行，我们就称进程处于内核运行态（或简称为内
核态）。 系统调用的过程可以简单理解为：

1. 用户态程序将一些数据值放在寄存器中， 或者使用参数创建一个堆栈， 以此表明需要操作系统提
   供的服务。
2. 用户态程序执行系统调用。
3. CPU切换到内核态，并跳到位于内存指定位置的指令。
4. 系统调用处理器(system call handler)会读取程序放入内存的数据参数，并执行程序请求的服务。
5. 系统调用完成后，操作系统会重置CPU为用户态并返回系统调用的结果。

由此可见用户态切换至内核态需要传递许多变量，同时内核还需要保护好用户态在切换时的一些寄存器
值、变量等，以备内核态切换回用户态。这种切换就带来了大量的系统资源消耗，这就是在
synchronized未优化之前，效率低的原因。  