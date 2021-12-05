# Selector

## select

监控所有注册的通道，当IO操作就绪时，返回对应的`SelectionKey`。通过`SelectionKey`可以得到IO操作就绪的`Channel`（`channel()`方法）和注册的`Selector`（`selector()`方法)

1. `select()`：阻塞的，直到至少有一个事件发生才返回
2. `select(timeout)`：阻塞的，超时后或者有事件发生返回
3. `selectNow()`：非阻塞的，如果没有事件发生返回0
4. `wakeup()`：唤醒阻塞的Selector

## NIO非阻塞网络编程原理

1. `ServerSocketChannel`注册到`Selector`上
2. 当客户端连接时，通过`ServerSocketChannel`得到`SocketChannel`
3. `SocketChannel`注册到`Selector`上
4. 注册后返回`SelectionKey`
5. `Selector`通过`select`方法监听事件，返回有事件就绪的通道个数
6. 进一步得到`SelectionKey`，从而得到`SocketChannel`
7. 通过`SocketChannel`完成业务处理

## NIO聊天室实例

**Server端**

```java
public class GroupChatServer {
    private static final int PORT = 9090;
    private Selector selector;
    private ServerSocketChannel ssc;

    public GroupChatServer(){
        try {
            selector = Selector.open();
            ssc = ServerSocketChannel.open();
            ssc.bind(new InetSocketAddress(PORT));
            ssc.configureBlocking(false);
            // 注册到Selector里
            ssc.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void listen(){
        try {
            while(true){
                if(selector.select(2000) > 0){
                    Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
                    while(keyIterator.hasNext()){
                        SelectionKey key = keyIterator.next();
                        if(key.isAcceptable()){
                            SocketChannel socketChannel = ssc.accept();
                            socketChannel.configureBlocking(false);
                            socketChannel.register(selector, SelectionKey.OP_READ);
                            System.out.println("客户端" + socketChannel.getRemoteAddress() + "上线");
                        }else if(key.isReadable()){
                            // 读取客户端发送过来的数据，并转发到其他客户端
                            readDataAndDispatcher(key);
                        }

                        keyIterator.remove();
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                ssc.close();
                selector.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void readDataAndDispatcher(SelectionKey key) {
        // 读取数据
        SocketChannel socketChannel = null;
        try {
            socketChannel = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            StringBuilder msg = new StringBuilder();
            int len;
            while ((len = socketChannel.read(buffer)) != 0) {
                msg.append(new String(buffer.array(), 0, len));
                buffer.rewind();
            }
            System.out.println("从客户端收到消息: " + msg);
            // 转发数据
            dispatcher(msg.toString(), socketChannel);
        } catch (IOException e) {
            // 出现异常说明连接断开
            try {
                System.out.println(socketChannel.getRemoteAddress() + " 离线了");
                // 取消注册
                key.cancel();
                // 关闭连接
                socketChannel.close();
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        }
    }

    private void dispatcher(String msg, Channel exclusive) throws IOException {
        for (SelectionKey key : selector.keys()) {
            Channel channel = key.channel();
            if(channel instanceof SocketChannel && channel != exclusive){
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                ((SocketChannel) channel).write(buffer);
            }
        }
    }

    public static void main(String[] args) {
        GroupChatServer server = new GroupChatServer();
        server.listen();
    }
}
```



**Client端**

```java
public class GroupChatClient {
    private static final String HOST = "127.0.0.1";
    private static final int PORT = 9090;
    private SocketChannel socketChannel;
    private Selector selector;
    private String username;

    public GroupChatClient(){
        try {
            InetSocketAddress address = new InetSocketAddress(HOST, PORT);
            selector = Selector.open();
            socketChannel = SocketChannel.open(address);
            socketChannel.configureBlocking(false);
            socketChannel.register(selector, SelectionKey.OP_READ);

            username = socketChannel.getLocalAddress().toString().substring(1);
            System.out.println(username + " is ok");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void sendData(String msg){
        try {
            msg = username + " 说：" + msg;
            ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
            socketChannel.write(buffer);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void receiveData(){
        try {
            if(selector.select(3000) > 0){
                Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
                while(keyIterator.hasNext()){
                    SelectionKey key = keyIterator.next();
                    if(key.isReadable()){
                        SocketChannel channel = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        StringBuilder msg = new StringBuilder();
                        int len;
                        while ((len = channel.read(buffer)) != 0) {
                            msg.append(new String(buffer.array(), 0, len));
                            buffer.rewind();
                        }
                        System.out.println(msg.toString().trim());
                    }

                    keyIterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        GroupChatClient client = new GroupChatClient();
        // 开一个线程用于接收数据，因为输入一直会等待控制台，因此会阻塞
        new Thread(() -> {
            while(true){
                client.receiveData();
            }

        }).start();
        Scanner scanner = new Scanner(System.in);
        while(scanner.hasNext()){
            String msg = scanner.nextLine();
            client.sendData(msg);
        }
    }
}
```

