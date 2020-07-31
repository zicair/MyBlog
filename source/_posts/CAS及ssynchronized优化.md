---
title: CAS及ssynchronized优化
date: 2020-07-14 09:47:28
tags: [Java, Synchronized]
categories:
- [Java, Synchronized]
---

CAS作用和原理，ssynchronized锁的优化: 偏向锁、轻量级锁、自旋锁、锁消除、锁粗化

<!--more-->

# CAS及ssynchronized优化

## 1. CAS作用和原理

创建五个线程，对类的变量自增：

![image-20200714100246436](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/CAS及ssynchronized优化/image-20200714100246436.png)

![image-20200714100403011](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/CAS及ssynchronized优化/image-20200714100403011.png)

**CAS：**Compare And Swap(比较相同再交换)。是现代CPU广泛支持的一种对内存中的共享数据进行操作的一种特殊指令。
**CAS的作用：**CAS可以将比较和交换转换为原子操作，这个原子操作直接由CPU保证。CAS可以保证共享变量赋值时的原子操作。CAS操作依赖3个值：内存中的值V，旧值X，新值B，如果旧值X等于内存中的值V，就将新值B保存到内存中。 
**CAS原理：**通过AtomicInteger的源码可以看到，Unsafe类提供了原子操作。
**Unsafe：**类使Java拥有了像C语言的指针一样操作内存空间的能力，同时也带来了指针的问题。过度的使Unsafe类会使得出错的几率变大，因此Java官方并不建议使用的，官方文档也几乎没有。Unsafe对象不能直接调用，只能通过反射获得。 

CAS和volatile实现无锁并发 ：

![image-20200714101817624](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/CAS及ssynchronized优化/image-20200714101817624.png)

AtomicInteger的incrementAndGet()调用过程：

---

![image-20200714102611238](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/CAS及ssynchronized优化/image-20200714102611238.png)

**乐观锁和悲观锁**

- 悲观锁从悲观的角度出发：
  总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这
  样别人想拿这个数据就会阻塞。因此synchronized我们也将其称之为悲观锁。JDK中的ReentrantLock
  也是一种悲观锁。性能较差！

- 乐观锁从乐观的角度出发:
  总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，就算改了也没关系，再重试即可。所
  以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去修改这个数据，如何没有人修改则更
  新，如果有人修改则重试。
  CAS这种机制我们也可以将其称之为乐观锁。综合性能较好！  

  ```
  CAS获取共享变量时，为了保证该变量的可见性，需要使用volatile修饰。结合CAS和volatile可以
  实现无锁并发，适用于竞争不激烈、多核CPU的场景下。
  1. 因为没有使用 synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一。
  2. 但如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响。  
  ```



## 2. Java对象的布局

**锁的升级过程：**无锁 > 偏向锁 > 轻量级锁 > 重量级锁 。这些锁的状态存储在java对象的布局中。

在JVM中，对象在内存中的布局分为三块区域：对象头、实例数据和对齐填充。如下图所示：  

![image-20200714105802613](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/CAS及ssynchronized优化/image-20200714105802613.png)

### 2.1 对象头

当一个线程尝试访问synchronized修饰的代码块时，它首先要获得锁，那么这个锁到底存在哪里呢？是
存在锁对象的对象头中的。
HotSpot采用instanceOopDesc和arrayOopDesc来描述对象头，arrayOopDesc对象用来描述数组类
型。instanceOopDesc的定义的在Hotspot源码的 instanceOop.hpp 文件中，另外，arrayOopDesc
的定义对应 arrayOop.hpp 。  

```c++
class instanceOopDesc : public oopDesc {
 public:
  // aligned header size.
  static int header_size() { return sizeof(instanceOopDesc)/HeapWordSize; }
  // If compressed, the offset of the fields of the instance may not be aligned.
  static int base_offset_in_bytes() {
    // offset computation code breaks if UseCompressedClassPointers
    // only is true
    return (UseCompressedOops && UseCompressedClassPointers) ?
			klass_gap_offset_in_bytes() :
			sizeof(instanceOopDesc);
  } 
  static bool contains_field_offset(int offset, int nonstatic_field_size) {
    int base_in_bytes = base_offset_in_bytes();
    return (offset >= base_in_bytes &&
			(offset-base_in_bytes) < nonstatic_field_size * heapOopSize);
  }
};
```

从instanceOopDesc代码中可以看到 instanceOopDesc继承自oopDesc，oopDesc的定义载Hotspot
源码中的 oop.hpp 文件中。  

```c++
class oopDesc {
  friend class VMStructs;
 private:
  volatile markOop _mark;
  union _metadata {
	Klass* _klass;
	narrowKlass _compressed_klass;
  } _metadata;
  // Fast access to barrier set. Must be initialized.
  static BarrierSet* _bs;
  // 省略其他代码
};
```

在普通实例对象中，oopDesc的定义包含两个成员，分别是 `_mark` 和 `_metadata`
_mark 表示对象标记、属于markOop类型，也就是接下来要讲解的Mark World，它记录了对象和锁有
关的信息
_metadata 表示类元信息，类元信息存储的是对象指向它的类元数据(Klass)的首地址，其中Klass表示
普通指针、 _compressed_klass 表示压缩类指针。
对象头由两部分组成，一部分用于存储自身的运行时数据，称之为 Mark Word，另外一部分是类型指
针，及对象指向它的类元数据的指针  

#### Mark Word

Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、
线程持有的锁、偏向线程ID、偏向时间戳等等，占用内存大小与虚拟机位长一致。

#### klass pointer

这一部分用于存储对象的类型指针，该指针指向它的类元数据，JVM通过这个指针确定对象是哪个类的
实例。该指针的位长度为JVM的一个字大小，即32位的JVM为32位，64位的JVM为64位。 如果应用的对
象过多，使用64位的指针将浪费大量内存，统计而言，64位的JVM将会比32位的JVM多耗费50%的内
存。为了节约内存可以使用选项 -XX:+UseCompressedOops 开启指针压缩，其中，oop即ordinary
object pointer普通对象指针。开启该选项后，下列指针将压缩至32位：

1. 每个Class的属性指针（即静态变量）
2. 每个对象的属性指针（即对象变量）
3. 普通对象数组的每个元素指针

当然，也不是所有的指针都会压缩，一些特殊类型的指针JVM不会优化，比如指向PermGen的Class对
象指针(JDK8中指向元空间的Class对象指针)、本地变量、堆栈元素、入参、返回值和NULL指针等。
对象头 = Mark Word + 类型指针（未开启指针压缩的情况下）
在32位系统中，Mark Word = 4 bytes (32bits)，类型指针 = 4bytes (32bits)，对象头 = 64 bits；  

### 2.2 实例数据

就是类中定义的成员变量。

### 2.3 对齐填充

对齐填充并不是必然存在的，也没有什么特别的意义，他仅仅起着占位符的作用，由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头正好是8字节的倍数，因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。  



## 3. 偏向锁

### 3.1 什么是偏向锁

偏向锁是JDK 6中的重要引进，因为HotSpot作者经过研究实践发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低，引进了偏向锁。
偏向锁的“偏”，是这个锁会偏向于第一个获得它的线程，会在对象头存储锁偏向的线程ID，以后该线程进入和退出同步块时只需要检查是否为偏向锁、锁标志位以及ThreadID即可。因此偏向锁是在只有一个线程执行同步块时进一步提高性能，适用于一个线程反复获得同一锁的情况。偏向锁可以提高带有同步但无竞争的程序性能。  

不过一旦出现多个线程竞争时必须撤销偏向锁，所以撤销偏向锁消耗的性能必须小于之前节省下来的
CAS原子操作的性能消耗，不然就得不偿失。

### 3.2 偏向锁原理

当线程第一次访问同步块并获取锁时，偏向锁处理流程如下：

```
1. 虚拟机将会把对象头中的标志位设为“01”，即偏向模式。
2. 同时使用CAS操作把获取到这个锁的线程的ID记录在对象的Mark Word之中 ，如果CAS操作
成功，持有偏向锁的线程以后每次进入这个锁相关的同步块时，虚拟机都可以不再进行任何
同步操作，偏向锁的效率高。
```

  ![image-20200714114312320](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/CAS及ssynchronized优化/image-20200714114312320.png)

### 3.3 偏向锁的撤销

1. 偏向锁的撤销动作必须等待全局安全点
2. 暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态
3. 撤销偏向锁，恢复到无锁（标志位为 **01**）或轻量级锁（标志位为 **00**）的状态

偏向锁在Java 6之后是默认启用的，但在应用程序启动几秒钟之后才激活，可以使用 `-XX:BiasedLockingStartupDelay=0` 参数关闭延迟，如果确定应用程序中所有锁通常情况下处于竞争状态，可以通过 `XX:-UseBiasedLocking=false` 参数关闭偏向锁。    



## 4. 轻量级锁

### 4.1 什么是轻量级锁

轻量级锁是JDK 6之中加入的新型锁机制，它名字中的“轻量级”是相对于使用monitor的传统锁而言的，因此传统的锁机制就称为“重量级”锁。首先需要强调一点的是，轻量级锁并不是用来代替重量级锁的。
**引入轻量级锁的目的：**在多线程交替执行同步块的情况下，尽量避免重量级锁引起的性能消耗，但是如果多个线程在同一时刻进入临界区，会导致轻量级锁膨胀升级重量级锁，所以轻量级锁的出现并非是要替代重量级锁。

### 4.2 轻量级锁原理

当关闭偏向锁功能或者多个线程竞争偏向锁导致偏向锁升级为轻量级锁，则会尝试获取轻量级锁，其步骤如下： 

**获取锁**  

1. 判断当前对象是否处于无锁状态（hashcode、0、01），如果是，则JVM首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝（官方把这份拷贝加了一个Displaced前缀，即Displaced Mark Word），将对象的Mark Word复制到栈帧中的Lock Record中，将Lock Reocrd中的owner指向当前对象。
2. JVM利用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，如果成功表示竞争到
   锁，则将锁标志位变成00，执行同步操作。
3. 如果失败则判断当前对象的Mark Word是否指向当前线程的栈帧，如果是则表示当前线程已经持
   有当前对象的锁，则直接执行同步代码块；否则只能说明该锁对象已经被其他线程抢占了，这时轻量级锁需要膨胀为重量级锁，锁标志位变成10，后面等待的线程将会进入阻塞状态。

**释放锁**
轻量级锁的释放也是通过CAS操作来进行的，主要步骤如下：

1. 取出在获取轻量级锁保存在Displaced Mark Word中的数据。
2. 用CAS操作将取出的数据替换当前对象的Mark Word中，如果成功，则说明释放锁成功。    
3. 如果CAS操作替换失败，说明有其他线程尝试获取该锁，则需要将轻量级锁需要膨胀升级为重量级锁。



## 5. 锁自旋

### 5.1 什么是锁自旋

前面我们讨论monitor实现锁的时候，知道monitor会阻塞和唤醒线程，线程的阻塞和唤醒需要CPU从用户态转为核心态，频繁的阻塞和唤醒对CPU来说是一件负担很重的工作，这些操作给系统的并发性能带来了很大的压力。同时，虚拟机的开发团队也注意到在许多应用上，共享数据的锁定状态只会持续很短的一段时间，为了这段时间阻塞和唤醒线程并不值得。如果物理机器有一个以上的处理器，能让两个或以上的线程同时并行执行，我们就可以让后面请求锁的那个线程“稍等一下”，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需让线程执行一个忙循环(自旋) , 这项技术就是所谓的自旋锁。  



自旋锁在JDK 1.4.2中就已经引入 ，只不过默认是关闭的，可以使用-XX:+UseSpinning参数来开启，在JDK 6中 就已经改为默认开启了。自旋等待不能代替阻塞，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，因此，如果锁被占用的时间很短，自旋等待的效果就会非常好，反之，如果锁被占用的时间很长。那么自旋的线程只会白白消耗处理器资源，而不会做任何有用的工作，反而会带来性 能上的浪费。因此，自旋等待的时间必须要有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程了。自旋次数的默认值是10次，用户可以使用参数-XX : PreBlockSpin来更改。  



### 5.2 适应性锁自旋

在JDK 6中引入了自适应的自旋锁。自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100次循环。另外，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能省略掉自旋过程，以避免浪费处理器资源。有了自适应自旋，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测就会越来越准确，虛拟机就会变得越来越“聪明”了。  



## 6. 锁消除

锁消除是指虚拟机即时编译器（JIT）在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持，如果判断在一段代码中，堆上的所有数据都不会逃逸出去从而被其他线程访问到，那就可以把它们当做栈上数据对待，认为它们是线程私有的，同步加锁自然就无须进行。变量是否逃逸，对于虚拟机来说需要使用数据流分析来确定，但是程序员自己应该是很清楚的，怎么会在明知道不存在数据争用的情况下要求同步呢?实际上有许多同步措施并不是程序员自己加入的，同步的代码在Java程序中的普遍程度也许超过了大部分读者的想象。下面这段非常简单的代码仅仅是输出3个字符串相加的结果，无论是源码字面上还是程序语义上都没有同步。  

```java
public class Demo01 {
	public static void main(String[] args) {
		contactString("aa", "bb", "cc");
	} 
	public static String contactString(String s1, String s2, String s3) {
		return new StringBuffer().append(s1).append(s2).append(s3).toString();
	}
}
```

StringBuffer的append ( ) 是一个同步方法，锁就是this也就是(new StringBuilder())。虚拟机发现它的
动态作用域被限制在concatString( )方法内部。也就是说, new StringBuilder()对象的引用永远不会“逃
逸”到concatString ( )方法之外，其他线程无法访问到它，因此，虽然这里有锁，但是可以被安全地消除
掉，在即时编译之后，这段代码就会忽略掉所有的同步而直接执行了。  



## 6. 锁粗化

原则上，我们在编写代码的时候，总是推荐将同步块的作用范围限制得尽量小，只在共享数据的实际作
用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变小，如果存在锁竞争，那等待锁的线
程也能尽快拿到锁。大部分情况下，上面的原则都是正确的，但是如果一系列的连续操作都对同一个对
象反复加锁和解锁，甚至加锁操作是出现在循环体中的，那即使没有线程竞争，频繁地进行互斥同步操
作也会导致不必要的性能损耗 。

```java
public class Demo01 {
	public static void main(String[] args) {
		StringBuffer sb = new StringBuffer();
		for (int i = 0; i < 100; i++) {
			sb.append("aa");
		} 
		System.out.println(sb.toString());
	}
}
```



## **7. 平时写代码对synchronized的优化**

- 减少synchronized的范围
  同步代码块中尽量短，减少同步代码块中代码的执行时间，减少锁的竞争。
- 降低synchronized锁的粒度
  将一个锁拆分为多个锁提高并发度(ashtable锁整个表、ConcurrentHashMap锁列)
- 读写分离
  读取时不加锁，写入和删除时加锁  