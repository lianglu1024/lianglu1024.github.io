# NIO介绍

NIO可以理解成NoBlocking IO，也可以理解为New IO，是java02年引入的包，存放在`package java.nio`。

NIO有三个最重要的概念，Channel，Buffer和Selector，**为什么引入了这三个概念，就可以比传统的BIO效率高呢**？我们要带着这样的思考往下看。

`Channel`：表示操作 I/O （如读取或写入）的连接。类似于Stream，只不过Stream是单向的，channel是双向的，可读可写。负责buffer进行传输数据。

`Buffer`：是一个用于存储特定基本类型数据的容器。除了boolean外，其余每种基本类型都有一个对应的buffer类。Buffer类的子类有ByteBuffer，CharBuffer， DoubleBuffer， FloatBuffer,，IntBuffer,，LongBuffer，ShortBuffer。

`Selector`：Selector（选择器）用于监听多个通道的事件（比如：连接打开，数据到达）。

Buffer有四个重要参数：

* capacity：buffer的容量，一旦定义不可改变
* limit：表示缓存区中可以操作数据的大小（limit后数据不能读写）
* position：缓存区中正在操作的位置
* mark：标记当前position的位置，可以通过reset恢复到mark的位置

```java
import org.junit.Test;

import java.nio.ByteBuffer;

public class Test02 {

    @Test
    public void test01(){
        ByteBuffer buf = ByteBuffer.allocate(1024);
        System.out.println(buf.position());
        System.out.println(buf.limit());

        String str = "abc";
        buf.put(str.getBytes());
        System.out.println(buf.position());  //3
        System.out.println(buf.limit());  //1024

        buf.flip();//position=0，limit=flip之前的position
        System.out.println(buf.position()); //0
        System.out.println(buf.limit()); //3
        System.out.println((char) buf.get());  // a
        System.out.println(buf.position());  //1

        buf.rewind();  //可重复读，
        System.out.println(buf.position()); //0
        System.out.println(buf.limit());  //3

        buf.clear(); //position和limit复位，数据不清除
        System.out.println(buf.position()); //0
        System.out.println(buf.limit());  //1024
        System.out.println((char) buf.get());  //a
    }
}
```

直接缓冲区和非直接缓存区

![image-20200918174155355](https://tva1.sinaimg.cn/large/007S8ZIlly1giuy0mpccuj30z40m4aep.jpg)

![image-20200918174636264](https://tva1.sinaimg.cn/large/007S8ZIlly1giuy4wt2fvj30xs0pgtf3.jpg)

![image-20200918175057622](https://tva1.sinaimg.cn/large/007S8ZIlly1giuy9gokppj30zw0nsnb6.jpg)

### Channel

![image-20200918175410066](https://tva1.sinaimg.cn/large/007S8ZIlly1giuycru0s2j313i0lu798.jpg)

通道的主要实现类：

* FileChannel
* SocketChannel
* ServerSocketChanel
* DatagramChannel

获取通道的方式

* getChannel方法，例如FileXxxStream，RandomAccessFile，Socket，ServerSocket





### 分散(Scatter)与聚集(Gather)

分散读取(Scattering Reads)：将通道中的数据分散到多个缓冲区中

聚集写入(Gathering Writes)：将多个缓冲区中的数据聚集到通道中



https://blog.csdn.net/weixin_43815050/article/details/95218893

















