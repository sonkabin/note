# BIO、NIO、AIO

IO包括两个过程：1）等待就绪；2）操作。对于读函数，分为等待系统可读和真正的读；同理写函数分为等待网卡可写和真正的写。等待就绪的阻塞不使用CPU，真正的读写操作使用CPU但速度很快。

阻塞的意思：不使用CPU，也不进行其他操作

## 适用场景

1. BIO适用于连接数目少且固定的架构，对服务器资源要求高
2. NIO适用于**连接数目多且连接比较短**（轻操作）的架构，比如聊天服务器、服务间通讯，JDK1.4开始支持
3. AIO适用于**连接数目多且连接比较长**（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，JDK1.7开始支持

## BIO模型

```java
{
    ExecutorService executor = Excutors.newFixedThreadPollExecutor(100);//线程池

    ServerSocket serverSocket = new ServerSocket();
    serverSocket.bind(8088);
    while(!Thread.currentThread.isInturrupted()){//主线程死循环等待新连接到来
        Socket socket = serverSocket.accept();
        executor.submit(new ConnectIOnHandler(socket));//为新的连接创建新的线程
}

class ConnectIOnHandler extends Thread{
    private Socket socket;
    public ConnectIOnHandler(Socket socket){
       this.socket = socket;
    }
    public void run(){
      while(!Thread.currentThread.isInturrupted()&&!socket.isClosed()){死循环处理读写事件
          String someThing = socket.read()....//读取数据
          if(someThing!=null){
             ......//处理数据
             socket.write()....//写数据
          }
      }
    }
}
```



使用多线程，主要原因在于socket.accept()、socket.read()、socket.write()三个主要函数都是同步阻塞的，**当一个连接在处理IO时，系统是阻塞的而CPU是空闲的，单线程不能利用CPU**

用多线程的原因是：

1. 利用多核；
2. 当I/O阻塞系统，但CPU空闲的时候，可以利用多线程使用CPU资源

BIO非常依赖于线程，当面对十万甚至百万级连接的时候，传统的BIO模型是无能为力的。

## NIO模型

通道（Channel）表示打开到IO设备（如文件、套接字）的连接，对数据的处理在缓冲区（Buffer）进行

NIO的读、写、注册和接收函数，在等待就绪阶段都是非阻塞的，真正的I/O操作是同步阻塞的。所以，**如果一个连接不能读写（读写函数返回0），可以立即返回，只需要把这件事在Selector上注册，然后切换到其他就绪的连接（Channel）继续进行读写。**

Selector的select是阻塞的，因此可以在死循环中调用这个函数，不用担心CPU空转。当有新事件到来时，会在Selector上标识可读、可写、有连接到来



### Buffer

既可以读又可以写，是一个内存块。支持Scattering和Gathering

属性：`capacity, limit, position, mark`。`mark <= position <= limit <= capacity`

方法：

1. `get()`：获取缓冲区数据
2. `put()`：存数据到缓冲区
3. `flip()`：切换到读模式，`limit=position, position=0, mark=-1`
4. `rewind()`：从开头开始重复读，`position=0，mark=-1`
5. `clear()`：缓冲区的属性恢复到最初状态，数据仍然在缓冲区中，但是处于被遗忘状态。`position=0, limit=capacity, mark=-1`
6. `mark()`：将mark标记到position位置，后续通过`reset()`可以再次从该位置开始

```java
public static void main(String[] args) {
    ByteBuffer buf = ByteBuffer.allocate(1024);
    String str = "abc";
    buf.put(str.getBytes());
    // 切换到读
    buf.flip();
    byte[] dst = new byte[buf.limit()];
    buf.get(dst);
    System.out.println(new String(dst, 0, buf.limit()));
    // 恢复缓冲区状态
    buf.clear();
    // 可以读缓冲区
    System.out.println(new String(dst, 0, 1));
    // 读过之后必须要恢复缓冲区状态，才可以进行写
    buf.clear();
    
    // 再次开始写和读
    str = "ddd";
    buf.put(str.getBytes());
    buf.flip();
    buf.get(dst);
    System.out.println(new String(dst, 0, buf.limit()));
}
```



`allocate()`申请的是JVM堆上的缓冲区，称为非直接缓冲区；而`allocateDirect()`申请的是操作系统物理内存的缓冲区，称为直接缓冲区。非直接缓冲区的效率低于直接缓冲区，因为它的数据需要在内核空间和用户之间互相拷贝。

### Channel

Channel是双向的

包含4个：

1. FileChannel：

   1）通过`map()`方法可以获得直接缓冲区

   ```java
   // 直接缓冲区的方式
   MappedByteBuffer inMappedBuf = inChannel.map(MapMode.READ_ONLY, 0, inChannel.size());
   // 参数1：MapMode.READ_WRITE，读写模式；参数2：可以直接修改的起始位置；参数3：映射到内存的大小，即将文件的多少个字节映射到内存
   MappedByteBuffer outMappedBuf = outChannel.map(MapMode.READ_WRITE, 0, inChannel.size());
   byte[] dst = new byte[inMappedBuf.limit()];
   inMappedBuf.get(dst);
   outMappedBuf.put(dst);
   
   // 非直接缓冲区的方式
   while(inChannel.read(buf) != -1){
       buf.flip();
       outChannel.write(buf);
       buf.clear();
   }
   ```

   

   2）通过`transferTo和transferFrom`在通道之间传输数据

   ```java
   inChannel.transferTo(0, inChannel.size(), outChannel);
   ```

   

2. SocketChannel；

3. ServerSocketChannel；

4. DatagramChannel

通道获取：1）对支持通道的类提供了getChannel()方法；2）NIO2中为每个通道提供了静态方法open()；3）NIO2的Files工具类的newByteChannel()方法

### 直接缓冲区与非直接缓冲区的选择

直接缓冲区最好分配给那些大型的、持久的缓冲区，最好只在直接缓冲区能在程序性能方面带来明显的好处时才分配。直接缓冲区的缺点是：程序将数据写入缓冲区后，将由操作系统来决定何时写入（可能导致垃圾回收器不能及时清理该缓冲区）；开辟一块直接缓冲区消耗较大。

### 非阻塞式

```java
public static void main(String[] args) {
    // Server
    new Thread(() -> {
        ServerSocketChannel ssc = null;
        try{
            ssc = ServerSocketChannel.open();
            ssc.bind(new InetSocketAddress(9090));
            // 配置非阻塞模式
            ssc.configureBlocking(false);

            Selector selector = Selector.open();
            // 注册Channel到Selector上，监听"Accept"事件
            ssc.register(selector, SelectionKey.OP_ACCEPT);
            // 等于0会阻塞，>0表示有事件就绪，有类似的方法：select(timeout), selectNow()，没有就绪事件立即返回0
            while(selector.select() > 0){
                Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                while(it.hasNext()){
                    SelectionKey sk = it.next();
                    if(sk.isAcceptable()){
                        SocketChannel sc = ssc.accept();
                        // 配置非阻塞
                        sc.configureBlocking(false);
                        // 注册Channel到Selector上，监听"Read/Write"事件
                        sc.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
                    }else if(sk.isReadable()){
                        SocketChannel sc = (SocketChannel) sk.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        int len = 0;
                        while((len = sc.read(buffer)) != -1){
                            buffer.flip();
                            System.out.println(new String(buffer.array(), 0, len));
                            buffer.clear();
                        }
                    }
                    // 需要移除，避免重复操作SelectionKey
                    it.remove();
                }
            }
        }catch(IOException e){
            e.printStackTrace();
        }finally{
            if(ssc != null){
                try{
                    ssc.close();
                }catch(IOException e){
                    e.printStackTrace();
                }
            }
        }
    }).start();
    try{
        Thread.sleep(100);
    }catch(Exception e){
        e.printStackTrace();
    }
    // Client
    new Thread(() -> {
        SocketChannel sc = null;
        try{
            sc = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9090));
            // 配置非阻塞
            sc.configureBlocking(false);
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put("hello".getBytes());
            buffer.flip();
            sc.write(buffer);
            buffer.clear();

        }catch(IOException e){
            e.printStackTrace();
        }finally{
            if(sc != null){
                try{
                    sc.close();
                }catch(IOException e){
                    e.printStackTrace();
                }
            }
        }
    }).start();
}
```



## BIO和NIO的区别

|        BIO         |         NIO          |
| :----------------: | :------------------: |
|       面向流       |  面向缓冲区 Buffer   |
| 阻塞IO（网络通信） | 非阻塞IO（网络通信） |
|                    | 选择器（Selectors）  |

