---
title: 分别使用 BIO 和 NIO 的实现简易群聊系统
date: 2022-08-29 17:13:25
categories: Java
tags:
    - BIO
    - NIO
    - Netty
    - SpringBoot
cover: https://img-blog.csdnimg.cn/9de0e63deef14190b01100372a0e1c73.png
---
# 前言
`下面是BIO和NIO的原理结构图，可以看出使用BIO时，每个客户端都会独占一个线程，而使用NIO时，一个Selector选择器独占一个线程，一个选择器下面可以连接多个客户端，然后Selector开始轮询下面的每一个客户端，这就提高了线程的复用，所以叫非阻塞IO`

![在这里插入图片描述](https://img-blog.csdnimg.cn/9de0e63deef14190b01100372a0e1c73.png)

---

# 一、BIO
>下面的代码已经完成了一个 BIO 服务端的编写，启动下面的代码后，可以用 cmd 的 telnet 命令来连接到服务端（telnet怎么安装可以自己bing一下哦）。

```bash
telnet 127.0.0.1 6666
```
>然后回车，就可以输入要发送的文字信息了，有些电脑可能回车以后还需要按一下 Ctrl+J 才能开始输入文字，这时，我们的cmd就变成了客户端，可以向服务端发送信息。

---

##  1、代码（含服务端、客户端）
```java
package pers.xuyijie.bio;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author 徐一杰
 * @date 2022/8/18 16:40
 * @description BIO Demo
 */
public class BIOServer {
    public static void main(String[] args) throws IOException {
        //创建一个线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        //创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        System.out.println("服务器已启动");
        while (true){
            System.out.println("等待连接...");
            //监听，等待客户端连接
            final Socket socket = serverSocket.accept();
            System.out.println("连接到一个客户端");
            //创建一个线程，与之通信
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        handler(socket);
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                }
            });
        }
    }

    /**
     * @author 徐一杰
     * @date 2022/8/23
     * @description 和客户端通信
    */
    public static void handler(Socket socket) throws IOException {
        System.out.println("线程信息：id=" + Thread.currentThread().getId() + " name=" + Thread.currentThread().getName());
        byte[] bytes = new byte[1024];
        //通过Socket，获取输入流
        InputStream inputStream = socket.getInputStream();
        //循环读取客户端发送的数据
        while (true){
            System.out.println("正在等待客户端信息...");
            int read = inputStream.read(bytes);
            if (read != -1){
                System.out.println(new String(bytes, 0, read));
            }else {
                break;
            }
        }
        try {
            socket.close();
            System.out.println("客户端连接已关闭");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```
---

##  2、演示

![在这里插入图片描述](https://img-blog.csdnimg.cn/42717364a5b14d5799b4476a18f77382.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/047af4efbb5c4e1f8252a1b4150a5178.png)

---

# 二、NIO
>下面的代码分为 Server 和 Client，将idea设置成同时启动多个相同类（看我下面的截图），可以启动多个客户端，而且Server有转发功能，收到任意一个客户端发来的消息，都可以把消息转发给其他所有客户端，也就是说任意一个客户端发送的消息，其他客户端和服务端都可以看到，可以作为一个简易的群聊系统。

`首先Edit Configurations`

![在这里插入图片描述](https://img-blog.csdnimg.cn/5c739589de8b49aca93b8e4e9f47e43a.png)
`然后选择要启动多个的类，我这里是 NIOClient，把Allow multiple instance勾上，确定就可以了`

![在这里插入图片描述](https://img-blog.csdnimg.cn/c01bb4b783224d27934982aac4e5e80b.png)

---

##  1、代码
### 服务端

`NIOServer.java，这个是服务端，只需要启动一个`
```java
package pers.xuyijie.nio.groupchat;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;

/**
 * @author 徐一杰
 * @date 2022/8/23 20:10
 * @description
 */
public class NIOServer {
    private Selector selector;
    private ServerSocketChannel serverSocketChannel;
    private static final int PORT = 6667;

    /**
     * 构造器完成初始化工作
     */
    public NIOServer(){
        try {
            //得到选择器
            selector = Selector.open();
            //开启通道
            serverSocketChannel = ServerSocketChannel.open();
            //绑定端口
            serverSocketChannel.socket().bind(new InetSocketAddress(PORT));
            //设置非阻塞
            serverSocketChannel.configureBlocking(false);
            //把通道注册到选择器
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e){
            e.printStackTrace();
        }
    }

    /**
     * 监听
     */
    public void listen(){
        try {
            //循环处理
            while (true){
                int count = selector.select(2000);
                if (count > 0){
                    //有事件处理
                    //遍历得到selectionKey集合
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while (iterator.hasNext()){
                        SelectionKey selectionKey = iterator.next();
                        if (selectionKey.isAcceptable()){
                            SocketChannel socketChannel = serverSocketChannel.accept();
                            socketChannel.configureBlocking(false);
                            //将通道注册到选择器
                            socketChannel.register(selector, SelectionKey.OP_READ);
                            System.out.println(socketChannel.getRemoteAddress() + "上线了！");
                        }
                        if (selectionKey.isReadable()){
                            readData(selectionKey);
                        }
                        //从当前keys中删除，防止重复操作
                        iterator.remove();
                    }
                }else {
                    System.out.println("正在监听...");
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {

        }
    }

    /**
     * 读取客户端消息
     * @param selectionKey
     */
    public void readData(SelectionKey selectionKey) throws IOException {
        //得到关联的SocketChannel
        SocketChannel socketChannel = null;
        try {
            socketChannel = (SocketChannel) selectionKey.channel();
            //创建buffer
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
            int count = socketChannel.read(byteBuffer);
            if (count > 0){
                //将buffer中的数据转成字符串
                String msg = new String(byteBuffer.array());
                System.out.println("客户端：" + msg);
                //向其他客户端转发消息
                sendMsgToOtherClients(msg, socketChannel);
            }
        } catch (IOException e) {
            System.out.println(socketChannel.getRemoteAddress() + "离线了！");
            //取消注册
            selectionKey.cancel();
            //关闭通道
            socketChannel.close();
        }
    }

    /**
     * 转发消息
     * @param msg
     * @param socketChannel
     * @throws IOException
     */
    public void sendMsgToOtherClients(String msg, SocketChannel socketChannel) throws IOException {
        System.out.println("服务器转发消息");
        //遍历所有注册到选择器上的SocketChannel，并排除自己
        for (SelectionKey selectionKey : selector.keys()){
            //通过key取出相应的通道
            Channel channel = selectionKey.channel();
            //排除自己
            if (channel instanceof SocketChannel && channel != socketChannel){
                SocketChannel socketChannel1 = (SocketChannel) channel;
                //将msg存储到buffer
                ByteBuffer byteBuffer = ByteBuffer.wrap(msg.getBytes());
                //将buffer的数据写入通道
                socketChannel1.write(byteBuffer);
            }
        }
    }

    public static void main(String[] args) {
        //创建一个服务器对象
        NIOServer nioServer = new NIOServer();
        nioServer.listen();
    }

}

```
---

### 客户端

`NIOClient.java，这时客户端，设置好以后，可以点击多次启动，启动多个`

```java
package pers.xuyijie.nio.groupchat;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Scanner;

/**
 * @author 徐一杰
 * @date 2022/8/23 20:39
 * @description
 */
public class NIOClient {
    private final String HOST = "127.0.0.1";
    private final int PORT = 6667;
    private final Selector selector;
    private final SocketChannel socketChannel;
    private final String username;

    /**
     * 构造器完成初始化工作
     */
    public NIOClient() throws IOException {
        //得到选择器
        selector = Selector.open();
        //连接服务器
        socketChannel = SocketChannel.open(new InetSocketAddress(HOST, PORT));
        //设置非阻塞
        socketChannel.configureBlocking(false);
        //将通道注册到选择器
        socketChannel.register(selector, SelectionKey.OP_READ);
        //得到username
        username = socketChannel.getLocalAddress().toString().substring(1);
        System.out.println(username + "已连接");
    }

    /**
     * 向服务端发送消息
     * @param msg
     * @throws IOException
     */
    public void sendMsg(String msg) throws IOException {
        msg = username + "：" + msg;
        socketChannel.write(ByteBuffer.wrap(msg.getBytes()));
    }

    /**
     * 接收从其他客户端转发的消息
     * @throws IOException
     */
    public void readMsg() throws IOException {
        int readChannel = selector.select();
        if (readChannel > 0){
            //有可用通道
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()){
                SelectionKey selectionKey = iterator.next();
                if (selectionKey.isReadable()){
                    //得到相关通道
                    SocketChannel socketChannel1 = (SocketChannel) selectionKey.channel();
                    //得到buffer
                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                    //读取
                    socketChannel1.read(byteBuffer);
                    //把buffer的数据转换成字符串
                    String msg = new String(byteBuffer.array());
                    System.out.println(msg.trim());
                }else {
                    //没有可用通道
                    System.out.println("没有可用通道");
                }
            }
        }
    }

    public static void main(String[] args) throws IOException {
        //启动客户端
        NIOClient nioClient = new NIOClient();
        //启动一个线程
        new Thread(){
            @Override
            public void run() {
                while (true){
                    try {
                        nioClient.readMsg();
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                    try {
                        sleep(3000);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
            }
        }.start();
        //发送数据给服务端
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()){
            String msg = scanner.nextLine();
            nioClient.sendMsg(msg);
        }
    }

}

```

---

## 2、演示
>启动两个文件，先启动 NIOServer.java，再启动 NIOClient.java，NIOClient我启动了两个，因为服务器有转发功能，相当于一个群聊系统.

![在这里插入图片描述](https://img-blog.csdnimg.cn/9efa8e81e838424e929e690d8db201b0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e1859a97710644b6ad64e813de508b83.png)

![!\[在这里插入图片描述\](https://img-blog.csdnimg.cn/faead29b143f](https://img-blog.csdnimg.cn/1b00e21e59264746b1a656b5aeb312a6.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d38faa8883c24c75bfb0764f50b6289f.png)




---

# 总结
以上是 JDK 的原生 BIO 和 NIO 的使用 demo，代码的意思在注释里面基本都有写，但是写的不是很详细，因为我也没有完全弄懂，这东西要慢慢自己断点调试来研究，接下来会写 NIO 的框架——Netty


