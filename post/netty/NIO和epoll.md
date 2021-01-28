# NIO和epoll

下面的内容将详细介绍网络编程的演变，涉及的知识点包括：

* BIO
* NIO
* select和epoll

## BIO

BIO是传统的网络编程模型，B（Blocking）主要体现在accept和read方法。

当没有任何客户端连接，accept会一直阻塞；如果客户端没有发送数据，那么read函数会一直阻塞。

简易版的客户端：

```java
import java.net.Socket;

public class Client {
    public static void main(String[] args) throws Exception{
        Socket socket = new Socket("127.0.0.1", 8888);
        Thread.sleep(10000);
        try{
            socket.getOutputStream().write("hello".getBytes());
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            socket.close();
        }
    }
}
```

单线程版服务器：

```java
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    static byte[] bs = new byte[1024];
    public static void main(String[] args) throws Exception{
        //backlog?
        ServerSocket serverSocket = new ServerSocket(8888);
        while (true){
            //如果没有客户端连接，accept会阻塞
            //clientSocket是建立了三次握手之后返回处理的socket
            Socket clientSocket = serverSocket.accept();
            InputStream inputStream = clientSocket.getInputStream();
            StringBuilder sb = new StringBuilder();
            int len;
            //如果socket里没有数据，read会阻塞
            //如果客户端关闭了socket，返回-1
            //如果socket里有数据，read返回的是数据的大小
            while ((len = inputStream.read(bs)) != -1) {
                //注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
                sb.append(new String(bs, 0, len,"UTF-8"));
            }
            System.out.println(sb);
        }
    }
}
```

存在的明显问题就是不能支持并发，当一个客户端连接之后，必须处理完成之后才能处理另外的客户端。

多线程服务器：

```java
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    static byte[] bs = new byte[1024];
    public static void main(String[] args) throws Exception{
        ServerSocket serverSocket = new ServerSocket(8888);
        while (true){
            Socket clientSocket = serverSocket.accept();
            System.out.println(clientSocket);
            new Thread(new Runnable() {
                @Override
                public void run(){
                    try{
                        InputStream inputStream = clientSocket.getInputStream();
                        StringBuilder sb = new StringBuilder();
                        int len;
                        while ((len = inputStream.read(bs)) != -1) {
                            sb.append(new String(bs, 0, len,"UTF-8"));
                        }
                        System.out.println(sb);
                    }catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}
```

虽然可以通过多线程的方式提供并发，但是多线程明显的缺陷就是消耗过多的系统资源，尤其是当只有多个连接却发送数据不多的情况下，这对系统资源是极大的浪费。

## NIO

NIO作为非阻塞的IO，就是操作系统提供了系统函数，可以在执行accept和read函数时候直接返回而不阻塞。在java中，这是全新的java包`java.nio.*`提供支持。

* read=-1，说明客户端发送了socket.close()[四次挥手]
* read=0，说明socket里没有数据可读
* read>0，说明socket里有数据

单线程NIO服务器：

```java
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class NIOServer {

    static List<SocketChannel> channelList = new ArrayList<>();
    public static void main(String[] args) throws Exception{

        ServerSocketChannel serverSocket = ServerSocketChannel.open();

        serverSocket.bind(new InetSocketAddress("0.0.0.0", 8000));

        serverSocket.configureBlocking(false);

        while (true){
            //不阻塞
            SocketChannel clientSocket = serverSocket.accept();
            if (clientSocket == null){
//                System.out.println("没有客户端连接！");
            }else{
                System.out.println(clientSocket);
                clientSocket.configureBlocking(false);
                channelList.add(clientSocket);
            }

            ByteBuffer buffer = ByteBuffer.allocateDirect(4096);

            Iterator<SocketChannel> iterator = channelList.iterator();
            while (iterator.hasNext()){
                SocketChannel socketChannel = iterator.next();
                int read = socketChannel.read(buffer);

                if (read > 0){
                    buffer.flip();
                    byte[] bs = new byte[buffer.limit()];
                    buffer.get(bs);
                    System.out.println(new String(bs));
                    buffer.clear();
                }else if(read == -1){
                    //客户端执行了socket.close()
//                    System.out.println(socketChannel + "数据读完了");
                    iterator.remove();  // 删除已经关闭的socket
                }else if (read == 0){
//                    System.out.println(socketChannel + "没有数据啦");
                }
            }
        }
    }
}
```

上面编写的服务器具有高并发处理的能力。当然缺点也很明显，如果轮询的是上万个不活跃的socket，那么效率是很低下的，此外，每次遍历都会去系统调用。我们可以将这部分轮询过程交给操作系统去完成，操作系统内核提供了一个select函数，完成的就是轮询的功能，JVM只需调用系统函数select即可，一次性将所有的文件描述符或者socket丢给系统调用。

## select和epoll

以下介绍socket和文件描述符是不区分的。

早起处理并发确实是用select来解决，但是select仍存在弊端：

1.轮询的socket个数是受限的，由linux内核决定

2.每次循环，jvm调用select函数都要将**所有**socket从用户区复制到内核区

3.轮询本身就是低效的

解决2的方法就是内核开辟空间保存socket，不需要每次循环都copy所有的socket

解决3的方式是通过中断+callback

### epoll

内核提供的三个epoll函数：

* `epoll_create()`

* `epoll_ctl(int epdf, int op, int fd, struct epoll_event *event)`
* * op：`EPOLL_CTL_ADD`，`EPOLL_CTL_MOD`，`EPOLL_CTL_DEL`
  * event：读写事件
* `epoll_wait(int epdf, struct epoll_event *event, int maxevents, int timeout)`

![image-20200916142431977](https://tva1.sinaimg.cn/large/007S8ZIlly1gish21oi99j30d20dcmxv.jpg)

* 程序开始的时候通过epoll_create创建一个文件描述符fd7，这个文件描述符可以理解成一个容器（红黑树实现）来专门存放socket。
* 当有socket（fd3）连接的时候，程序调用epoll_ctl，将fd3放到fd7里面。
* 当fd3来了数据后会触发中断，此时fd3会copy一份到队列中
* 调用epoll_wait的时候，内核会把队列里的准备好的文件描述符给应用程序，程序依次的去处理这些socket。
* epoll_wait可以传超时时间，如果在超时时间内里没有返回的描述符，则一直阻塞

```java
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class SelectServer {
    private static Selector selector = null;

    public static void main(String[] args) throws Exception{
        ServerSocketChannel server = ServerSocketChannel.open();
        server.configureBlocking(false);
        server.bind(new InetSocketAddress(8000));

        //如果在epoll模型下，相当于是epoll_create->fd7
        selector = Selector.open();

        //epoll_ctl(fd7, ADD, fd3, ACCEPT)
        server.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("服务器开始启动了...");

        while (true){
            Set<SelectionKey> keys = selector.keys();
            System.out.println("keys有哪些" + keys);
            //epoll_wait(500)
            while (selector.select(500) > 0){
                //返回有状态的fd集合
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                System.out.println("有状态的fd集合" + selectionKeys);
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()){
                    SelectionKey key = iterator.next();
                    iterator.remove();
                    if (key.isAcceptable()){
                        acceptHandler(key);
                    }else if(key.isReadable()){
                        readHandler(key);
                    }else if(key.isWritable()){
                    }else{
                    }
                }
            }
        }
    }

    public static void acceptHandler(SelectionKey key) throws Exception{
        ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
        SocketChannel client = ssc.accept();
        client.configureBlocking(false);
        ByteBuffer bb = ByteBuffer.allocate(1024);
        client.register(selector, SelectionKey.OP_READ, bb);
        System.out.println("新客户端连接" + client);
    }

    public static void readHandler(SelectionKey key) throws Exception{
        SocketChannel clientChannel = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer)key.attachment();
        int read = 0;
        while ((read = clientChannel.read(buffer)) > 0){
            buffer.flip();
            System.out.println(new String(buffer.array(),0,read));
            buffer.clear();

            ByteBuffer outBuffer = ByteBuffer.wrap("好的".getBytes());
            clientChannel.write(outBuffer);
        }
        if (read == -1){
            System.out.println("客户端关闭");
            key.cancel();
        }
    }
}
```

有一点必须要记住：不管是select还是epoll，返回给程序的都是可读可写的socket**状态**，必须由程序员自己手动的读写这些socket（内核空间和用户空间的copy），**因此说select和epoll都是同步操作**。（如果程序自己读取IO，那么这个IO模型，无论BIO，NIO，多路复用，都是同步模型。但windows系统中，IOCP内核有线程会拷贝到程序的内存空间的）。


