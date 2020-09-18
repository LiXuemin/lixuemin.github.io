---
title: JVM常用知识点
date: 2020-01-05 12:03:37
categories: Java基础
tags:
- JVM
---
#### 1. JVM运行时数据区

这个图相信很多人都看过N多次了，自己再做一次加深下记忆
![JVM Runtime](/images/jvm/JVMRuntime.png)

其中方法区和堆是由所有线程共享的数据区，而我们绝大多数的JVM调优都是针对方法区和堆。
其它都是线程隔离的数据区。
想要理解JVM，有3个比较重要的概念需要了解清楚，分别是：

* 类加载机制
* 虚拟机栈
* 堆
<!--more-->
#### 2. 类加载器

##### 2.1 类加载机制

虚拟机把描述类的数据从class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

##### 2.2 类的生命周期

类从被加载到虚拟机内存中开始，到卸载出内存为止，整个生命周期包括：
![类的生命周期](/images/jvm/ClassLoader.png)

其中类的加载、验证、准备、初始化、卸载这5个阶段是按照顺序开始的，**解析**阶段则不一定，某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定。

##### 2.3 类加载器

绝大部分Java程序都会使用到以下三种系统提供的类加载器：

* 启动类加载器(Bootstrap Classloader)： 加载${JAVA_HOME}/lib目录中的，并且虚拟机可识别的
* 扩展类加载器(Extension Classloader): 加载${JAVA_HOME}/lib/ext目录中的扩展类
* 应用程序类加载器(Application Classloader): 加载ClassPath上所指定的类库，可以理解为加载所有开发者自己写的类

如果有必要，还可以加入自己定义的类加载器。

##### 2.4 双亲委派模型(Parents Delegation Model)

![双亲委派模型](/images/jvm/ParentsDelegationModel.png)

1. 类加载器收到了加载类的请求，它首先不会自己加载这个类，而是把这个请求委派给父类加载器完成，每一个层次的类加载器都是如此。
2. 一直向上委派，因此所有的加载请求都应该传送到顶层的启动类加载器中。
3. 启动类加载器会检测当前类能否被加载，如果可以则加载；如果不可以则向下传递，反馈给子加载器，让其尝试自己进行加载。

有了双亲委派模型，才保证了Java类型体系中最基础的行为。

##### 2.5 OSGi

OSGi中的类加载器并不符合传统的双亲委派的类加载器，但其对类加载器的使用是很值得学习的。

#### 3. 虚拟机栈(VM Stack)

了解虚拟机栈对于我们理解JVM中方法的运行有极大帮助，通过原理，我们可以了解线程中的方法调用链，知道为什么生产环境中不建议甚至禁止使用递归。
![Stack Frame](/images/jvm/StackFrame.png)

一个线程中的方法调用链可能会很长，很多方法同时处于执行状态。
对于执行引擎来讲，在活动线程中，只有位于栈顶的栈帧才是有效的，称为当前栈帧(Current Stack Frame),与之相关联的方法称为当前方法(Current Method)。

如果线程请求的栈深度大于虚拟机所允许的深度，将抛出`StackOverFlowError`异常；
如果虚拟机栈可以动态扩展，若扩展时无法申请到足够的内存，就会抛出`OutOfMemoryError`异常。

#### 4. 堆(Heap)

堆的重要性是因为我们平时所需要的性能调优，GC等都发生在此内存区域。
三种常见的JVM：HotSpot,JRockit,j9vm。而平时用的几乎都是HotSpot，我们来看看它的实现。

##### 4.1 堆内存的划分

由于现在GC收集器基本都采用分代收集算法，所以Java堆中可以细分为：

* 新生代(Young Generation) : 99%的对象生命周期
  * 伊甸区(Eden Space) : 创建对象
  * 幸存者区(Survivor Space) : 对象活过GC就进入
* 老年代(Old Generation)
  * 养老区(Tenured Space) ： 幸存者区活过多次GC进入；大对象直接进入
* <del>永久代(Perm Gen)</del>  元空间(MetaSpace,支持动态扩展，since Java8)

![JVM heap structure](/images/jvm/JVM_Structure.png)

##### 4.2 GC分类

* Partial GC：并不收集整个GC堆的模式
  * Young GC：只收集young gen的GC
  * Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
  * Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式
* Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。

假如伊甸区和养老区都满了，就会发生`OutOfMemoryError`.

##### 4.3 GC算法

* 复制算法

* 标记-清除 & 标记-整理算法

* 分代收集算法

没有什么新的思想，只是因为在实际场景中，并没有一种算法能够同时在所有场景下保证最优。
所以针对每个内存区域，对象的存活率不同，可以对不同内存（新生代，老年代）采用最适当的手机算法。

* 新生代 每次GC都会有大批对象死去，对象存活率低  选用复制算法
* 老年代 对象存活率高 选用标记-清除，标记-整理算法

##### 4.4 HeapDump

加参数 -XX:+HeapDumpOnOutOfMemoryError 在发生OOM时生成堆快照
