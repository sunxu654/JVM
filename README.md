<!-- TOC -->

- [1. JVM优化](#1-jvm%E4%BC%98%E5%8C%96)
  - [1.1. JVM简单结构图](#11-jvm%E7%AE%80%E5%8D%95%E7%BB%93%E6%9E%84%E5%9B%BE)
    - [1.1.1. 类加载子系统与方法区：](#111-%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%AD%90%E7%B3%BB%E7%BB%9F%E4%B8%8E%E6%96%B9%E6%B3%95%E5%8C%BA)
  - [1.2. Java堆](#12-java%E5%A0%86)
  - [1.3. 直接内存](#13-%E7%9B%B4%E6%8E%A5%E5%86%85%E5%AD%98)
  - [1.4. 垃圾回收系统](#14-%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%B3%BB%E7%BB%9F)
  - [1.5. Java栈](#15-java%E6%A0%88)
  - [1.6. 本地方法栈](#16-%E6%9C%AC%E5%9C%B0%E6%96%B9%E6%B3%95%E6%A0%88)
  - [1.7. PC寄存器](#17-pc%E5%AF%84%E5%AD%98%E5%99%A8)
  - [1.8. 执行引擎](#18-%E6%89%A7%E8%A1%8C%E5%BC%95%E6%93%8E)
- [2. 堆结构及对象分代](#2-%E5%A0%86%E7%BB%93%E6%9E%84%E5%8F%8A%E5%AF%B9%E8%B1%A1%E5%88%86%E4%BB%A3)
  - [2.1. 什么是分代，分代的必要性是什么](#21-%E4%BB%80%E4%B9%88%E6%98%AF%E5%88%86%E4%BB%A3%E5%88%86%E4%BB%A3%E7%9A%84%E5%BF%85%E8%A6%81%E6%80%A7%E6%98%AF%E4%BB%80%E4%B9%88)
  - [2.2. 分代的划分](#22-%E5%88%86%E4%BB%A3%E7%9A%84%E5%88%92%E5%88%86)
  - [2.3. 新生代（Young Generation）](#23-%E6%96%B0%E7%94%9F%E4%BB%A3young-generation)
    - [2.3.1. 老年代（Old Generationn）](#231-%E8%80%81%E5%B9%B4%E4%BB%A3old-generationn)
    - [2.3.2. 永久代（Permanent Generationn）](#232-%E6%B0%B8%E4%B9%85%E4%BB%A3permanent-generationn)
- [3. 垃圾回收算法及分代垃圾收集器](#3-%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95%E5%8F%8A%E5%88%86%E4%BB%A3%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8)
  - [3.1. 垃圾收集器的分类](#31-%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8%E7%9A%84%E5%88%86%E7%B1%BB)
    - [3.1.1. 次收集器](#311-%E6%AC%A1%E6%94%B6%E9%9B%86%E5%99%A8)
    - [3.1.2. 全收集器](#312-%E5%85%A8%E6%94%B6%E9%9B%86%E5%99%A8)
    - [3.1.3. 垃圾回收器的常规匹配](#313-%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8%E7%9A%84%E5%B8%B8%E8%A7%84%E5%8C%B9%E9%85%8D)
  - [3.2. 常见垃圾回收算法](#32-%E5%B8%B8%E8%A7%81%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95)
    - [3.2.1. 引用计数（Reference Counting）](#321-%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0reference-counting)
    - [3.2.2. 复制（Copying）](#322-%E5%A4%8D%E5%88%B6copying)
    - [3.2.3. 标记-清除（Mark-Sweep）](#323-%E6%A0%87%E8%AE%B0-%E6%B8%85%E9%99%A4mark-sweep)
    - [3.2.4. 标记-整理（Mark-Compact）](#324-%E6%A0%87%E8%AE%B0-%E6%95%B4%E7%90%86mark-compact)

<!-- /TOC -->

# 1. JVM优化
## 1.1. JVM简单结构图
### 1.1.1. 类加载子系统与方法区：
![](https://i.imgur.com/jMOTXPq.jpg)
类加载子系统负责从文件系统或者网络中加载Class信息，加载的类信息存放于一块称为方法区的内存空间。除了类的信息外，方法区中可能还会存放运行时常量池信息，包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）。
## 1.2. Java堆
java堆在虚拟机启动的时候建立，它是java程序最主要的内存工作区域。几乎所有的java对象实例都存放在java堆中。堆空间是所有线程共享的，这是一块与java应用密切相关的内存空间。
## 1.3. 直接内存
java的NIO库允许java程序使用直接内存。直接内存是在java堆外的、直接向系统申请的内存空间。通常访问直接内存的速度会优于java堆。因此出于性能的考虑，读写频繁的场合可能会考虑使用直接内存。由于直接内存在java堆外，因此它的大小不会直接受限于Xmx指定的最大堆大小，但是系统内存是有限的，java堆和直接内存的总和依然受限于操作系统能给出的最大内存。
## 1.4. 垃圾回收系统
垃圾回收系统是java虚拟机的重要组成部分，垃圾回收器可以对方法区、java堆和直接内存进行回收。其中，java堆是垃圾收集器的工作重点。和C/C++不同，java中所有的对象空间释放都是隐式的，也就是说，java中没有类似free()或者delete()这样的函数释放指定的内存区域。对于不再使用的垃圾对象，垃圾回收系统会在后台默默工作，默默查找、标识并释放垃圾对象，完成包括java堆、方法区和直接内存中的全自动化管理。
## 1.5. Java栈
每一个java虚拟机线程都有一个私有的java栈，一个线程的java栈在线程创建的时候被创建，java栈中保存着帧信息，java栈中保存着局部变量、方法参数，同时和java方法的调用、返回密切相关。
## 1.6. 本地方法栈
本地方法栈和java栈非常类似，最大的不同在于java栈用于方法的调用，而本地方法栈则用于本地方法的调用，作为对java虚拟机的重要扩展，java虚拟机允许java直接调用本地方法（通常使用C编写）
## 1.7. PC寄存器
PC（Program Counter）寄存器也是每一个线程私有的空间，java虚拟机会为每一个java线程创建PC寄存器。在任意时刻，一个java线程总是在执行一个方法，这个正在被执行的方法称为当前方法。如果当前方法不是本地方法，PC寄存器就会指向当前正在被执行的指令。如果当前方法是本地方法，那么PC寄存器的值就是undefined
## 1.8. 执行引擎
执行引擎是java虚拟机的最核心组件之一，它负责执行虚拟机的字节码，现代虚拟机为了提高执行效率，会使用即时编译(just in time)技术将方法编译成机器码后再执行。

Java HotSpot Client VM(-client)，为在客户端环境中减少启动时间而优化的执行引擎；本地应用开发使用。（如：eclipse）

Java HotSpot Server VM(-server)，为在服务器环境中最大化程序执行速度而设计的执行引擎。应用在服务端程序。（如：tomcat）

Java HotSpot Client模式和Server模式的区别

当虚拟机运行在-client模式的时候,使用的是一个代号为C1的轻量级编译器, 而-server模式启动的虚拟机采用相对重量级,代号为C2的编译器. C2比C1编译器编译的相对彻底,服务起来之后,性能更高

JDK安装目录/jre/lib/（x86、i386、amd32、amd64）/jvm.cfg

文件中的内容，-server和-client哪一个配置在上，执行引擎就是哪一个。如果是JDK1.5版本且是64位系统应用时，-client无效。

--64位系统内容
-server KNOWN
-client IGNORE

--32位系统内容
-server KNOWN
-client KNOWN

*注意：在部分JDK1.6版本和后续的JDK版本(64位系统)中，-client参数已经不起作用了，Server模式成为唯一

# 2. 堆结构及对象分代
## 2.1. 什么是分代，分代的必要性是什么
Java虚拟机根据对象存活的周期不同，把堆内存划分为几块，一般分为新生代、老年代和永久代（对HotSpot虚拟机而言），这就是JVM的内存分代策略。

堆内存是虚拟机管理的内存中最大的一块，也是垃圾回收最频繁的一块区域，我们程序所有的对象实例都存放在堆内存中。给堆内存分代是为了提高对象内存分配和垃圾回收的效率。试想一下，如果堆内存没有区域划分，所有的新创建的对象和生命周期很长的对象放在一起，随着程序的执行，堆内存需要频繁进行垃圾收集，而每次回收都要遍历所有的对象，遍历这些对象所花费的时间代价是巨大的，会严重影响我们的GC效率。

有了内存分代，情况就不同了，新创建的对象会在新生代中分配内存，经过多次回收仍然存活下来的对象存放在老年代中，静态属性、类信息等存放在永久代中，新生代中的对象存活时间短，只需要在新生代区域中频繁进行GC，老年代中对象生命周期长，内存回收的频率相对较低，不需要频繁进行回收，永久代中回收效果太差，一般不进行垃圾回收，还可以根据不同年代的特点采用合适的垃圾收集算法。分代收集大大提升了收集效率，这些都是内存分代带来的好处。
## 2.2. 分代的划分
Java虚拟机将堆内存划分为新生代、老年代和永久代，永久代是HotSpot虚拟机特有的概念（JDK1.8之后为metaspace替代永久代），它采用永久代的方式来实现方法区，其他的虚拟机实现没有这一概念，而且HotSpot也有取消永久代的趋势，在JDK 1.7中HotSpot已经开始了“去永久化”，把原本放在永久代的字符串常量池移出。永久代主要存放常量、类信息、静态变量等数据，与垃圾回收关系不大，新生代和老年代是垃圾回收的主要区域。

内存简图如下：
![](https://i.imgur.com/gV4LVPU.jpg)
 
## 2.3. 新生代（Young Generation）
新生成的对象优先存放在新生代中，新生代对象朝生夕死，存活率很低，在新生代中，常规应用进行一次垃圾收集一般可以回收70% ~ 95% 的空间，回收效率很高。

HotSpot将新生代划分为三块，一块较大的Eden（伊甸）空间和两块较小的Survivor（幸存者）空间，默认比例为8：1：1。划分的目的是因为HotSpot采用复制算法来回收新生代，设置这个比例是为了充分利用内存空间，减少浪费。新生成的对象在Eden区分配（大对象除外，大对象直接进入老年代），当Eden区没有足够的空间进行分配时，虚拟机将发起一次Minor GC。

GC开始时，对象只会存在于Eden区和From Survivor区，To Survivor区是空的（作为保留区域）。GC进行时，Eden区中所有存活的对象都会被复制到To Survivor区，而在From Survivor区中，仍存活的对象会根据它们的年龄值决定去向，年龄值达到年龄阀值（默认为15，新生代中的对象每熬过一轮垃圾回收，年龄值就加1，GC分代年龄存储在对象的header中）的对象会被移到老年代中，没有达到阀值的对象会被复制到To Survivor区。接着清空Eden区和From Survivor区，新生代中存活的对象都在To Survivor区。接着， From Survivor区和To Survivor区会交换它们的角色，也就是新的To Survivor区就是上次GC清空的From Survivor区，新的From Survivor区就是上次GC的To Survivor区，总之，不管怎样都会保证To Survivor区在一轮GC后是空的。GC时当To Survivor区没有足够的空间存放上一次新生代收集下来的存活对象时，需要依赖老年代进行分配担保，将这些对象存放在老年代中。
### 2.3.1. 老年代（Old Generationn）
在新生代中经历了多次（具体看虚拟机配置的阀值）GC后仍然存活下来的对象会进入老年代中。老年代中的对象生命周期较长，存活率比较高，在老年代中进行GC的频率相对而言较低，而且回收的速度也比较慢。
### 2.3.2. 永久代（Permanent Generationn）
永久代存储类信息、常量、静态变量、即时编译器编译后的代码等数据，对这一区域而言，Java虚拟机规范指出可以不进行垃圾收集，一般而言不会进行垃圾回收。
# 3. 垃圾回收算法及分代垃圾收集器
## 3.1. 垃圾收集器的分类
### 3.1.1. 次收集器
Scavenge GC，指发生在新生代的GC，因为新生代的Java对象大多都是朝生夕死，所以Scavenge GC非常频繁，一般回收速度也比较快。当Eden空间不足以为对象分配内存时，会触发Scavenge GC。

一般情况下，当新对象生成，并且在Eden申请空间失败时，就会触发Scavenge GC，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden去能尽快空闲出来。

当年轻代堆空间紧张时会被触发
相对于全收集而言，收集间隔较短
### 3.1.2. 全收集器
Full GC，指发生在老年代的GC，出现了Full GC一般会伴随着至少一次的Minor GC（老年代的对象大部分是Scavenge GC过程中从新生代进入老年代），比如：分配担保失败。Full GC的速度一般会比Scavenge GC慢10倍以上。当老年代内存不足或者显式调用System.gc()方法时，会触发Full GC。

当老年代或者持久代堆空间满了，会触发全收集操作
可以使用System.gc()方法来显式的启动全收集
全收集一般根据堆大小的不同，需要的时间不尽相同，但一般会比较长。
### 3.1.3. 垃圾回收器的常规匹配

![](https://i.imgur.com/2bWXOPA.jpg)

## 3.2. 常见垃圾回收算法
### 3.2.1. 引用计数（Reference Counting）
比较古老的回收算法。原理是此对象有一个引用，即增加一个计数，删除一个引用则减少一个计数。垃圾回收时，只用收集计数为0的对象。此算法最致命的是无法处理循环引用的问题。

### 3.2.2. 复制（Copying）
此算法把内存空间划为两个相等的区域，每次只使用其中一个区域。垃圾回收时，遍历当前使用区域，把正在使用中的对象复制到另外一个区域中。此算法每次只处理正在使用中的对象，因此复制成本比较小，同时复制过去以后还能进行相应的内存整理，不会出现“碎片”问题。当然，此算法的缺点也是很明显的，就是需要两倍内存空间。简图如下：
![](https://i.imgur.com/ZZ9ZVkE.jpg)

### 3.2.3. 标记-清除（Mark-Sweep）
此算法执行分两阶段。第一阶段从引用根节点开始标记所有被引用的对象，第二阶段遍历整个堆，把未标记的对象清除。此算法需要暂停整个应用，同时，会产生内存碎片。简图如下：

![](https://i.imgur.com/x6iQRHH.jpg)

### 3.2.4. 标记-整理（Mark-Compact）

此算法结合了“标记-清除”和“复制”两个算法的优点。也是分两阶段，第一阶段从根节点开始标记所有被引用对象，第二阶段遍历整个堆，把清除未标记对象并且把存活对象“压缩”到堆的其中一块，按顺序排放。此算法避免了“标记-清除”的碎片问题，同时也避免了“复制”算法的空间问题。简图如下：

![](https://i.imgur.com/LLbSfCd.jpg)

123



本地的修改
