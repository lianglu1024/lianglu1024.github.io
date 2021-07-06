# JVM内存模型

jvm内存模型分为程序计数器、java虚拟机栈、本地方法栈、java堆和方法区

- 程序计数器：当前线程所执行的字节码的行号指示器，用于记录正在执行的虚拟机字节指令地址，线程私有
- Java虚拟机栈：存放局部变量表、对象的引用、方法出口等，线程私有。编译阶段就可以确定每个方法栈帧的大小，所以运行阶段是不会发生改变
- 本地方法栈：和虚拟栈相似，只不过它服务于Native方法，线程私有。
- Java堆：java内存最大的一块，所有对象实例、数组都存放在java堆，GC回收的地方，线程共享。
- 方法区：存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码数据等。方法区只是一个概念，由各个厂家去实现。1.6版本实现方式是永久代，使用堆内存的一部分作为方法区，且由JVM 管理。1.8版本实现方式是元空间，存放在系统内存中，受操作系统管理，不受gc处理

# JVM 为什么使用元空间替换了永久代？

永久代和元空间都是存放Class对象，使用动态代理的时候，可能使得空间暴涨

永久代是可以设置大小，而元空间用的是系统内存，设置和不设置没啥用

1.7字符串常量存在永久代中，而1.8是存在堆内存中。**常量池和字符串常量池是方法区的概念，但是实现可能各有不同**

- 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
- 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

# 新生代和老年代

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12826881/1625146771183-b9f78eb3-ae72-46b1-a4a3-da53801c6eaf.png?x-oss-process=image%2Fresize%2Cw_1500)

对象创建到回收的流程：

- 创建的对象首先进入新生代的eden区，如果该对象太大eden区无法承载，直接进入老年代
- 当eden区快满了触发一次Minor GC（YGC），通过拷贝算法，即将存活对象从eden区拷贝都s1区
- 之后eden区又快满了触发Minor GC，将eden中的存活对象和s1中的对象拷贝到s2区
- 之后又触发触发Minor GC，将eden中的存活对象和s2中的对象拷贝到s1区
- 当s区的对象达到年龄之后或者s空间不足时，将新生代存活对象复制到老年代
- 当老年代空间不足会触发Full GC，即扫描整个堆空间（新生代+老年代）



YGC的效率很高，单纯的拷贝；而FGC扫描整个堆内存，而且还要标记压缩，效率低。所以JVM调优就是减少FGC，FGC还会导致STW，使工作线程暂停

# 强软弱虚引用是什么以及区别？

- 强引用：我们平时new了一个对象就是强引用，例如 Object obj = new Object();即使在内存不足的情况下，JVM宁愿抛出OutOfMemory错误也不会回收这种对象。
- 软引用：如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。使用`SoftReference`来创建软引用对象，应用场景是缓存
- 弱引用：具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。使用`WeakReference`来创建弱引用对象，应用场景是缓存
- 虚引用：如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。虚引用主要用来跟踪对象被垃圾回收器回收的活动。使用`PhantomReference`来创建虚引用对象，必须搭配引用队列`ReferenceQueue`使用，应用场景是堆外内存的回收

# JVM垃圾回收的时候如何确定垃圾？是否知道什么是GC Roots

垃圾是内存中不再使用到的空间，判断一个对象是否是垃圾可以用**引用计数法**和**可达性分析，**java中使用可达性分析算法判断对象是否是垃圾，只要沿着GC Roots对象可以追溯到对象，该对象就不是垃圾。

哪些对象可以作为GC Roots的对象

![image.png](https://tva1.sinaimg.cn/large/008i3skNly1gs79u5yedtj30yy0i6wtl.jpg)

- 虚拟机栈中局部变量（也叫局部变量表）中引用的对象
- 方法区中类的静态变量、常量引用的对象
- 本地方法栈中 JNI (Native方法)引用的对象 

```
public class GCRootDemo {
    private byte[] byteArray = new byte[100 * 1024 * 1024];
    private static GCRootDemo gc2;
    private static final GCRootDemo gc3 = new GCRootDemo();
    public static void m1(){
        GCRootDemo gc1 = new GCRootDemo();
        System.gc();
        System.out.println("第一次GC完成");
    }
    public static void main(String[] args) {
        m1();
    }
}
```

解释：

```
gc1:是虚拟机栈中的局部变量
gc2:是方法区中类的静态变量
gc3:是方法区中的常量
都可以作为GC Roots 的对象。
```

# 垃圾回收算法

引用计数：只要有一个引用指向对象，就给引用加1，如果引用数为0，那么这个对象就会被回收。容易出现循环引用

标记-清除：标记出垃圾对象并清除。缺点是容易造成内存碎片

拷贝算法：内存分为两块，一块用来放分配的对象，等垃圾回收的时候，将不是垃圾的对象复制到另一块内存上连续存放。缺点是浪费空间

标记-压缩：结合上面两种方式，标记出垃圾对象清除之后将存活的对象移动拼凑成连续占用的内存空间。没有内存碎片，但是效率低下

# 常量池和字符串常量池

# volatile关键字

volatile本意是不稳定的，易变的

volatile关键字解决的是下面两个问题：

- 可以使变量对所有的线程立即可见（某一线程如果修改了工作内存中的变量副本，那么加上volatile之后，该变量就会立刻同步到其他线程的工作内存中）
- 禁止指令的重排序优化

volatile关键字不能保证原子性，如多线程下对一个变量进行自增操作



# volatile的原理和实现机制

“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”

lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：

　　1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；

　　2）它会强制将对缓存的修改操作立即写入主存；

　　3）如果是写操作，它会导致其他CPU中对应的缓存行无效。

# happens-before

单线程happen-before原则：在同一个线程中，书写在前面的操作happen-before后面的操作。 锁的happen-before原则：同一个锁的unlock操作happen-before此锁的lock操作。

volatile的happen-before原则：对一个volatile变量的写操作happen-before对此变量的任意操作(当然也包括写操作了)。

happen-before的传递性原则：如果A操作 happen-before B操作，B操作happen-before C操作，那么A操作happen-before C操作。

线程启动的happen-before原则：同一个线程的start方法happen-before此线程的其它方法。

线程中断的happen-before原则 ：对线程interrupt方法的调用happen-before被中断线程的检测到中断发送的代码。

线程终结的happen-before原则： 线程中的所有操作都happen-before线程的终止检测。

对象创建的happen-before原则： 一个对象的初始化完成先于他的finalize方法调用。

# 说说你知道的几种主要的JVM参数

堆栈配置相关

```
-Xms1024m：设置初始堆大小为1024m
-Xmx1024m：设置最大堆大小为1024m
-Xmn2g：设置年轻代大小为2g
-Xss128k：设置每个线程的栈空间为128k
-XX:MetaspaceSize=20m：设置元空间大小为20m
-XX:NewRatio=4：年轻代与老年代在堆中的占比，年轻代1份，老年代4份。默认是2
-XX:SurvivorRatio=8：年轻代中eden:s0:s1=8:1:1。s0和s1占比相同。默认是8
-XX:MaxTenuringThread=15：设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。
```

垃圾收集器相关

```
-XX:+UseParallelGC：选择垃圾收集器为并行收集器。
-XX:ParallelGCThreads=20：配置并行收集器的线程数
-XX:+UseConcMarkSweepGC：设置年老代为并发收集。
-XX:CMSFullGCsBeforeCompaction：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
```

辅助信息相关

```
-XX:+PrintGCDetails：打印gc回收详情
```

# 说说你遇到过的OOM

`java.lang.StackOverflowerError`：栈空间溢出

`java.lang.OutOfMemoryError:Java heap space`：堆空间溢出

`java.lang.OutOfMemoryError:GC overhead limit exceeded`：98%的时间都在GC并且回收不到2%的堆内存。现象就是机器CPU100%

`java.lang.OutOfMemoryError:Direct buffer memory`：直接内存溢出

`java.lang.OutOfMemoryError:unable to create new native thread:`创建的线程数太多。linux下最大运行数为1024个线程

`java.lang.OutOfMemoryError:Metaspace`：元空间溢出

# 内存溢出和内存泄露的区别？

内存溢出 out of memory，是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory

内存泄露 memory leak，内存泄露本意是申请的内存空间没有被正确释放，导致后续程序里这块内存被永远占用（不可达），而且指向这块内存空间的指针不再存在时，这块内存也就永远不可达了，内存空间就这么一点点被蚕食，借用别人的比喻就是：比如有10张纸，本来一人一张，画完自己擦了还回去，别人可以继续画，现在有个坏蛋要了纸不擦不还，然后还跑了找不到人了，如此就只剩下9张纸给别人用了，这样的人多起来后，最后大家一张纸都没有了。`ThreadLocal`类如果使用完之后没有remove也会造成内存泄露（只要线程一直存活，Entry里key对应的value对象会一直被强引用而得到不释放）

memory leak会最终会导致out of memory！

# 简单说说你了解的类加载器，可以打破双亲委派么，怎么打破。

 **什么是类加载器？**

类加载器就是根据指定全限定名称将class文件加载到JVM内存，转为Class对象。

- 启动类加载器（Bootstrap ClassLoader）：由C++语言实现（针对HotSpot）,负责将存放在<JAVA_HOME>\lib目录或-Xbootclasspath参数指定的路径中的类库加载到内存中。
- 扩展类加载器（Extension ClassLoader）：负责加载<JAVA_HOME>\lib\ext目录或java.ext.dirs系统变量指定的路径中的所有类库。
- 应用程序类加载器（Application ClassLoader）。负责加载用户类路径（classpath）上的指定类库，我们可以直接使用这个类加载器。一般情况，如果我们没有自定义类加载器默认就是用这个加载器。

**双亲委派模型**

如果一个类加载器收到类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器完成。每个类加载器都是如此，只有当父加载器在自己的搜索范围内找不到指定的类时（即ClassNotFoundException），子加载器才会尝试自己去加载。

**为什么需要双亲委派模型？**

在这里，先想一下，如果没有双亲委派，那么用户是不是可以自己定义一个java.lang.Object的同名类，java.lang.String的同名类，并把它放到ClassPath中,那么类之间的比较结果及类的唯一性将无法保证，因此，为什么需要双亲委派模型？防止内存中出现多份同样的字节码

**怎么打破双亲委派模型？**

打破双亲委派机制则不仅要继承ClassLoader类，还要重写loadClass和findClass方法。