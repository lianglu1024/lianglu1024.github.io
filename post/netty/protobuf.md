## protobuf练习

![image-20201022185433911](https://tva1.sinaimg.cn/large/0081Kckwly1gjyb65gcnjj30ku0amjva.jpg)

![image-20201022142744342](https://tva1.sinaimg.cn/large/0081Kckwly1gjy3gidmgxj30xe0gijz7.jpg)





## 装饰模式？FileInputStream和FilterInputStream

## 通道是双向的么？

![image-20201022212122103](https://tva1.sinaimg.cn/large/0081Kckwly1gjyfezt27mj30nj0eowl9.jpg)

## allocate和allocateDirect的区别

我们的程序在跟硬件打交道的时候，都必须要内核介入，比如读取磁盘数据，网络发送信息等。数据的流动首先在用户空间，然后copy到内核空间，最后流转到对应的硬件设备上。数据从用户空间到内核空间的互相copy是来自于系统调用（system call），例如read，write等。

allocate底层调用的是HeapByteBuffer，就是在堆内分配了一个字节数组byte[] hb

allocateDirect底层调用的是DirectByteBuffer，里面调用了本地native方法unsafe.allocateMemory(size)，分配的内存放在堆外内存上。DirectByteBuffer内部用一个address变量指向了该堆外内存。

DirectByteBuffer 自身是（Java）堆内的，它背后真正承载数据的buffer是在（Java）堆外——native memory中的。这是 malloc() 分配出来的内存，是用户态的。

传统的BIO模式下向磁盘写数据

![image-20201023214800210](https://tva1.sinaimg.cn/large/0081Kckwly1gjzlsxqdtnj30cr08ejri.jpg)

使用NIO模式的直接内存向磁盘写数据可以省略堆内内存向堆外内存的数据copy，数据直接写到堆外内存。

补充： 

> 狭义的堆外内存
>
> 而作为java开发者，我们常说的堆外内存溢出了，其实是狭义的堆外内存，这个主要是指
>
> java.nio.DirectByteBuffer在创建的时候分配内存，我们这篇文章里也主要是讲狭义的堆外内存，因为它和我
>
> 们平时碰到的问题比较密切
>
> JDK/JVM里DirectByteBuffer的实现 
>
> DirectByteBuffer通常用在通信过程中做缓冲池，在mina，netty等nio框架中屡见不鲜
>
> 通过上面的代码我们知道可以通过-XX:MaxDirectMemorySize来指定最大的堆外内存
>
> DirectByteBuffer在创建的时候会通过Unsafe的native方法来直接使用malloc分配一块内存，这块内存是
>
> heap 之外的，那么自然也不会对gc造成什么影响(System.gc除外)，因为gc耗时的操作主要是操作heap之内
>
> 的对象，对这块内存的操作也是直接通过 Unsafe的native方法来操作的，相当于DirectByteBuffer仅仅是一
>
> 个壳，还有我们通信过程中如果数据是在Heap里的，最终也还是会copy一份到堆外，然后再进行发送，所以
>
> 为什么不直接使用堆外内存呢。对于需要频繁操作的内存，并且仅仅是临时存在一会的，都建议使用堆外内
>
> 存，并且做成缓冲池，不断循环利用这块内存。
>
> 如果我们大面积使用堆外内存并且没有限制，那迟早会导致内存溢出，毕竟程序是跑在一台资源受限的机器上，因为这块内存的回收不是你直接能控制的。
>
> 正常情况下，JVM创建一个缓冲区的时候，实际上做了如下几件事：
>
> 1. JVM确保Heap区域内的空间足够，如果不够则使用触发GC在内的方法获得空间;
> 2. 获得空间之后会找一组堆内的连续地址分配数组, 这里需要注意的是，在物理内存上，这些字节是不一定连续的;
>
> 对于不涉及到IO的操作，这样的处理没有任何问题，但是当进行IO操作的时候就会出现一点性能问题.
>
> 所有的IO操作都需要操作系统进入内核态才行，而JVM进程属于用户态进程, 当JVM需要把一个缓冲区写到某个Channel或Socket的时候，需要切换到内核态.
>
> 而内核态由于并不知道JVM里面这个缓冲区存储在物理内存的什么地址，并且这些物理地址并不一定是连续的(或者说不一定是IO操作需要的块结构)，所以在切换之前JVM需要把缓冲区复制到物理内存一块连续的内存上, 然后由内核去读取这块物理内存，整合成连续的、分块的内存.
>
> 为了解决这个问题, Java的某些版本会把物理区域分配好的部分内存做缓存就不用每次都开辟一块空间，但效果还不够好，毕竟复制的部分是少不了的.
>
> JDK1.4之后引入了NIO, 提供了一种内存映射技术, 让我们可以直接从Java代码中创建DirectBuffer，这种Buffer在创建的时候直接就在物理内存中分配一块连续内存，当需要使用的时候不再需要复制，内核直接调用即可. 但缺点也是显而易见的，就是每次分配都比较昂贵一点，同时由于分配的内存不在Java Heap中，所以也不会受用户设置的堆大小的限制.
>
> 通常情况下，大量使用IO操作的时候使用内存映射是非常值得的

### 参考

[关于JVM堆外内存的一切](https://juejin.im/post/6844903710766661639)



## 零拷贝

![image-20201026131055305](https://tva1.sinaimg.cn/large/0081Kckwly1gk2npsuxe3j30rm0t20wq.jpg)