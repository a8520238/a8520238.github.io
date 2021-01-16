---
title: BIO_NIO_AIO总结
date: 2020-10-29 16:06:54
cover:
top_img:
categories: IO
tags: 
mathjax: true
katex: true
---

# 1 同步异步阻塞非阻塞

## 1.1 同步与异步

同步和异步描述的是一种消息通知的机制，主动等待消息返回还是被动接受消息。

- 同步：同步IO指的是调用方通过主动等待获取调用返回的结果来获取消息通知。
- 异步：异步IO指的是被调用方通过某种方式（如，回调函数）来通知调用方获取消息。

## 1.2 阻塞和非阻塞

阻塞和非阻塞描述的是调用方在获取消息过程中的状态，阻塞等待还是立刻返回。

- 阻塞IO指的是调用方在获取消息的过程中会挂起阻塞，直到获取到消息。
- 非阻塞IO指的是调用方在获取IO的过程中会立刻返回而不进行挂起。

# 2 BIO
同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。

## 2.1 传统BIO

![BIO](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/1B620F8437B04888AAB5AA2638CD9558/3792)

> IO过程中涉及两个过程，到网卡的数据通过内核调用到达内核区内存，用户程序再将内核区数据拷到用户区，再读入到程序中。在上图中accept和read两个过程都是阻塞的。

- 采用 BIO 通信模型 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般通过在while(true) 循环中服务端会调用 accept() **linux native方法，建立连接，也就是把数据读到内核区** 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如上图所示。
- 无论是NIO还是BIO，读取输入流的过程都是阻塞的，即从内存中读取数据写数据的过程。即多个socket连接只能有一个进行读写操作。


```
//客户端
public class IOClient {

  public static void main(String[] args) {
    // TODO 创建多个线程，模拟多个客户端连接服务端
    new Thread(() -> {
      try {
        Socket socket = new Socket("127.0.0.1", 3333);
        while (true) {
          try {
            socket.getOutputStream().write((new Date() + ": hello world").getBytes());
            Thread.sleep(2000);
          } catch (Exception e) {
          }
        }
      } catch (IOException e) {
      }
    }).start();

  }

}


//服务端
public class IOServer {

  public static void main(String[] args) throws IOException {
    // TODO 服务端处理客户端连接请求
    ServerSocket serverSocket = new ServerSocket(3333);

    // 接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理
    new Thread(() -> {
      while (true) {
        try {
          // 阻塞方法获取新的连接
          Socket socket = serverSocket.accept();

          // 每一个新的连接都创建一个线程，负责读取数据
          new Thread(() -> {
            try {
              int len;
              byte[] data = new byte[1024];
              InputStream inputStream = socket.getInputStream();
              // 按字节流方式读取数据(阻塞)
              while ((len = inputStream.read(data)) != -1) {
                System.out.println(new String(data, 0, len));
              }
            } catch (IOException e) {
            }
          }).start();

        } catch (IOException e) {
        }

      }
    }).start();

  }

}
```

## 2.2 BIO改进，伪异步IO

![](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/470934604B5343609AB712EE118A9B3A/3795)

- 如上图所示，每accept一个socket，就会产生一个新的线程。这样就会支持多个socket同时读写，通过线程池可以改善性能，在并发量不大的情况在可行。

- 对BIO进行多线程上的改进，当有新的客户端接入时，将客户端的 Socket 封装成一个Task（该任务实现java.lang.Runnable接口）投递到后端的线程池中进行处理，JDK 的线程池维护一个消息队列和 N个活跃线程，对消息队列中的任务进行处理。由于线程池可以设置消息队列的大小和最大线程数，因此，它的资源占用是可控的，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。


## 2.3 当客户端并发访问量增加后这种模型会出现什么问题？

在 Java 虚拟机中，线程是宝贵的资源，线程的创建和销毁成本很高，除此之外，线程的切换成本也是很高的。尤其在 Linux 这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的系统函数。如果并发访问量增加会导致线程数急剧膨胀可能会导致线程堆栈溢出、创建新线程失败等问题，最终导致进程宕机或者僵死，不能对外提供服务。

# 3 NIO

- NIO，可以看做是通过selector进行IO多路复用
- Java NIO和IO之间最大的区别是IO是面向流（Stream）的，NIO是面向块（buffer）

![](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/7D92B0EF4C404E2DB7DC55684F75A058/3755)

如上图所示，从网卡调到内核空间的过程是非阻塞的调用函数为，epoll() 和 recvfrom() 等native方法。也就是数据准备阶段是非阻塞的。

从源码上看，select通过轮训选择有响应的socket，做读写操作是阻塞的，要注意select的过程是阻塞的，直到至少有一个已注册的事件发生。当一个或者更多的事件发生时，select() 方法将返回所发生的事件的数量,可以用分布式多线程提升性能。
> 注意,为了提升性能，有时候会使用公共内存，减少数据从内核拷贝到用户区消耗的性能。


IO流是阻塞的，NIO流是不阻塞的。也就是说select到的channel即表明数据已经全部存到buffer中，只需要从buffer中取即可。同步的含义是需要轮训判断是否读取完毕。读写过程中可以做别的事情，但如果读写的线程只有一个，性能较低，可以开多个线程进行读写。

NIO 包含下面几个核心的组件：

+ Channel(通道)
+ Buffer(缓冲区)
+ Selector(选择器)

NIO与IO区别：
- NIO少了1次从内核空间到用户空间的拷贝。（零拷贝）
ByteBuffer.allocateDirect()分配的内存使用的是本机内存而不是Java堆上的内存，和网络或者磁盘交互都在操作系统的内核空间中发生。allocateDirect()的区别在于这块内存不由java堆管理, 但仍然在同一用户进程内。
- NIO以块处理数据，IO以流处理数据
- 非阻塞，NIO1个线程可以管理多个输入输出通道


缓冲区的存在导致读入内核空间的过程是非阻塞的。缓冲区实际上是一个容器对象，更直接的说，其实就是一个数组，在NIO库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的； 在写入数据时，它也是写入到缓冲区中的；任何时候访问 NIO 中的数据，都是将它放到缓冲区中。而在面向流I/O系统中，所有数据都是直接写入或者直接将数据读取到Stream对象中。

```
public class NIOServer {
  public static void main(String[] args) throws IOException {
    // 1. serverSelector负责轮询是否有新的连接，服务端监测到新的连接之后，不再创建一个新的线程，
    // 而是直接将新连接绑定到clientSelector上，这样就不用 IO 模型中 1w 个 while 循环在死等
    Selector serverSelector = Selector.open();
    // 2. clientSelector负责轮询连接是否有数据可读
    Selector clientSelector = Selector.open();

    new Thread(() -> {
      try {
        // 对应IO编程中服务端启动
        ServerSocketChannel listenerChannel = ServerSocketChannel.open();
        listenerChannel.socket().bind(new InetSocketAddress(3333));
        listenerChannel.configureBlocking(false);
        listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

        while (true) {
          // 监测是否有新的连接，这里的1指的是阻塞的时间为 1ms
          if (serverSelector.select(1) > 0) {
            Set<SelectionKey> set = serverSelector.selectedKeys();
            Iterator<SelectionKey> keyIterator = set.iterator();

            while (keyIterator.hasNext()) {
              SelectionKey key = keyIterator.next();

              if (key.isAcceptable()) {
                try {
                  // (1)
                  // 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                  SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                  clientChannel.configureBlocking(false);
                  clientChannel.register(clientSelector, SelectionKey.OP_READ);
                } finally {
                  keyIterator.remove();
                }
              }

            }
          }
        }
      } catch (IOException ignored) {
      }
    }).start();
    new Thread(() -> {
      try {
        while (true) {
          // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为 1ms
          if (clientSelector.select(1) > 0) {
            Set<SelectionKey> set = clientSelector.selectedKeys();
            Iterator<SelectionKey> keyIterator = set.iterator();

            while (keyIterator.hasNext()) {
              SelectionKey key = keyIterator.next();

              if (key.isReadable()) {
                try {
                  SocketChannel clientChannel = (SocketChannel) key.channel();
                  ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                  // (3) 面向 Buffer
                  clientChannel.read(byteBuffer);
                  byteBuffer.flip();
                  System.out.println(
                      Charset.defaultCharset().newDecoder().decode(byteBuffer).toString());
                } finally {
                  keyIterator.remove();
                  key.interestOps(SelectionKey.OP_READ);
                }
              }

            }
          }
        }
      } catch (IOException ignored) {
      }
    }).start();

  }
}
```

# 4 几个有助理解的图

![](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/2341F85D9DAA47B0B4BA52E27C073D93/3777)
![](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/584105CFDCB145A0A387ECB07AA5B1F7/3779)
![BIO](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/51931963C88A45D5968E96D61666E379/3831)
<center>BIO</center> 

![NIO](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/F3C2D8748A7948EE85665B2489C34202/3833)
<center>NIO</center> 

# 5 Netty

Java 的开源框架  网络服务器客户端框架
使用netty的优点 java开源框架 原始nio有bug存在。

## 5.1 Reactor线程模型

1. Reactor线程模型：
    - 单线程模型：所有的IO操作都由同一个NIO线程处理，仅限于一些小型应用场景。但在高负载、高并发等情况下使用单线程肯定就不太合理，主要是因为NIO的一个线程同时要去处理成千上万的请求 的时候，在性能上会支撑不了，即便CPU负载100%，对于海量消息的处理，编码解码以及读取、发送消息等情况，依然满足不了。
    - 当NIO的线程负载过重之后，整体服务性能处理就会变慢，结果就是导致客户端在向服务端发起请求、链接就会超时，由于客户端一般都会有一种超时机制，反复地向服务端再次发起请求，此时就相当于陷入了死循环，更加加重了服务器负载。

![](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/B625A11242CB43F083E51B956D643D32/3871)

2. 多线程模型：由一组NIO线程处理IO操作
3. 主从线程模型：一组线程池接受请求，一组线程池处理IO

![](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/60BD876D2259409CB7E63A33998D2511/3869)

多线程：一组nio线程处理io操作
![](http://note.youdao.com/yws/public/resource/347e6a5d67e0fd43b8f066d5fb52a900/xmlnote/997E77E15F7D45E8BC40AAEA8A73B5AB/3867)

重点概念：线程池  等待队列
主从线程模型：一组线程池接受请求，一组线程池，处理io

https://blog.csdn.net/quxing10086/article/details/80296245


# 6 epoll（Todo）

https://blog.csdn.net/songchuwang1868/article/details/89877739

# 7 sync