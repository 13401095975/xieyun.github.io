## Netty

1. netty是由jboss提供的一个java开源框架
2. netty是一个异步的、基于时间驱动的网络应用框架，用以快速开发高性能、高可靠性的网络IO程序
3. netty主要针对在TCP协议下，面对clients端的高并发应用，或者peer-to-peer场景下的大量数据持续传输的应用
4. netty本质是一个NIO框架

netty

nio

java原生编程

tcp/ip



### IO模型

- io模型就是用什么样的通道进行数据的发送和接收，很大程序的决定了程序通信的性能
- java支持3种网络编程模型：BIO NIO AIO



### NIO

- Channel(通道)  FileChannel  DatagramChannel (UDP) ServerSocketChannel SocketChannel
- Buffer（缓冲区） ByteBuffer
- Selector（选择器）

#### FileChannel

```java
public int read(ByteBuffer dst); //将通道中数据读入缓冲区
public int write(ByteBuffer src); //将缓冲区数据写入通道
public long transferFrom(ReadableByteChannel src, long position, long count); //将目标通道数据复制到当前通道
public long transerTo(long position, long count, WritableByteChannel target); //将当前通道数据复制给目标通道
```



#### Selector

- Selector能够检测多个注册的通道上是否有事件发生

```java
selector.select();//阻塞
selector.select(1000);//阻塞1000ms
selector.wakeup();//唤醒
selector.selectNow();//不阻塞，立马返回
```



- 当客户端连接时，会通过ServerSocketChannel得到对应的SocketChannel
- 将SocketChannel注册到Selector上，registor(Selector sel, int ops)
- 注册后返回一个SelectionKey, 会和该Selector关联
- Selector进行监听select方法，返回有事件发生的通道的个数
- 得到各个SelectionKey（有事件发生的）
- 再通过SelectionKey反向获取channel
- 通过channel完成读写



SelectionKey：OP_CONNECT OP_ACCTPT OP_READ OP_WRITE

```java
selector();
channel();
attachment();
isAcceptable();
isReadable();
isWritable();
```

ServerSocketChannel:

```java
open();
bind();
configureBlocking(bool);
accept();
register(Selector sel, int ops);

```

零拷贝是指不经过CPU拷贝

- mmap
- sendFile, linux2.4后支持很好，windows一次只支持8M



线程模型

- 阻塞I/O
- Reactor模式

#### Reactor模式图解

![image-20200626202624895](D:\data\note\开发\netty.assets\image-20200626202624895.png)

### Reactor模式细分

#### 单Reactor单线程，一个线程处理容易造成瓶颈

![image-20200626202837494](D:\data\note\开发\netty.assets\image-20200626202837494.png)



#### 单Reactor多线程

![image-20200626204353328](D:\data\note\开发\netty.assets\image-20200626204353328.png)

- 优点：充分利用多核CPU的处理能力。
- 缺点：多线程数据共享和访问比较复杂，reactor处理所有的事件监听和响应，在单线程运行高并发场景容易出现性能瓶颈。

#### 主从Reactor多线程（Netty依据此模型进行了该进）

![image-20200626205509827](D:\data\note\开发\netty.assets\image-20200626205509827.png)

- 注：一个Reactor主线程可以对应多个Reactor子线程
- 缺点：编程复杂度高

#### Netty模型

![image-20200626212109924](D:\data\note\开发\netty.assets\image-20200626212109924.png)



