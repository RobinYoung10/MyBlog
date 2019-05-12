---
title: Java网络编程学习（二）——————Socket的使用
date: 2019-05-10 14:48:41
tags: java网络编程
---

### 一、服务器Socket

ServerSocket类包含了使用Java编写服务器所需的全部内容，它的生命周期如下：

1. 使用一个ServerSocket()构造函数在一个指定端口创建一个ServerSocket。
2. 使用accept()方法监听入站连接。此方法会一直阻塞，直到一个客户端建立连接，此时将返回一个连接客户端和服务器的Socket对象。
3. 调用Socket的getInputStream()方法或getOutputStream()方法，获得客户间通信的输入输出流。
4. 服务器和客户端根据协议交互，直到关闭连接。
5. 服务器或客户端（或两者）关闭连接。
6. 服务器返回步骤2，等待下一次连接。

如下代码实现一个返回当前时间的服务器Socket:

```java
public class DaytimeServer {
    public final static int PORT = 13;

    public static void main(String[] args) {
        try(ServerSocket serverSocket = new ServerSocket(PORT)) {
            while(true) {
                try(Socket connection = serverSocket.accept()) {
                    Writer writer = new OutputStreamWriter(connection.getOutputStream());
                    Date now = new Date();
                    writer.write(now.toString() + "\r\n");
                    writer.flush();
                    connection.close();
                } catch (Exception e) {}
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

首先`new ServerSocket(13)`创建一个监听端口13的服务器Socket，`serverSocket.accept()`接受连接并返回连接Socket对象实例，调用`getOutputStream()`获得服务器Socket输出流，最后通过这个输出流写入数据到客户端，关闭连接。

##### 多线程服务器

上面的服务器socket是单线程socket，每次只能有一个客户端socket连接，新的客户端连接需要等待前一个客户端断开连接后才能连接，而多线程服务器socket能解决这个问题，它为每一个连接提供自己的一个线程，这样就可以防止慢客户端阻塞所有其他客户端，如下代码所示：

```java
public class MultithreadDaytimeServer {

    public final static int PORT = 10086;

    public static void main(String[] args) {
        try(ServerSocket server = new ServerSocket(PORT)) {
            while(true) {
                try {
                    Socket connection = server.accept();
                    Thread task = new DaytimeThread(connection);
                    task.start();
                } catch (Exception e) {
                    System.err.println(e);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static class DaytimeThread extends Thread {
        private Socket connection;

        DaytimeThread(Socket connection) {
            this.connection = connection;
        }

        @Override
        public void run() {
            try {
                Reader reader = new InputStreamReader(connection.getInputStream(), "UTF-8");
                for(int r = reader.read(); r != -1; r = reader.read()) {
                    System.out.print((char)r);
                }
                connection.shutdownInput();
                Writer writer = new OutputStreamWriter(connection.getOutputStream());
                Date now = new Date();
                writer.write(now.toString() + "\r\n");
                writer.flush();
            } catch (Exception e) {
                System.err.println(e);
            } finally {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

这样设计服务器有可能发生一种拒绝服务攻击，因为每个连接生成一个新线程，大量的入站连接会导致它生成大量线程，耗尽内存而崩溃。解决办法就是使用一个固定的线程池限制资源的使用，当线程到达最大量时，它可能会拒绝连接，但起码不会崩溃：

```java
public class LoggingDaytimeServer {

    public final static int PORT = 13;
    private final static Logger auditLogger = Logger.getLogger("requests");
    private final static Logger errorLogger = Logger.getLogger("errors");

    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(50);

        try (ServerSocket server = new ServerSocket(PORT)) {
            while (true) {
                try {
                    Socket connection = server.accept();
                    Callable<Void> task = new DaytimeTask(connection);
                    pool.submit(task);
                } catch (IOException e) {
                    errorLogger.log(Level.SEVERE, "accept error", e);
                } catch (RuntimeException e) {
                    errorLogger.log(Level.SEVERE, "unexpected error" + e.getMessage(), e);
                }
            }
        } catch (IOException e) {
            errorLogger.log(Level.SEVERE, "Couldn't start server", e);
        } catch (RuntimeException e) {
            errorLogger.log(Level.SEVERE, "Couldn't start server:" + e.getMessage(), e);
        }
    }

    private static class DaytimeTask implements Callable<Void> {

        private Socket connection;

        DaytimeTask(Socket connection) {
            this.connection = connection;
        }

        @Override
        public Void call() {
            try {
                Date now = new Date();
                auditLogger.info(now + " " + connection.getRemoteSocketAddress());
                Writer out = new OutputStreamWriter(connection.getOutputStream());
                out.write(now.toString() + "\r\n");
                out.flush();
            } catch (IOException e) {

            } finally {
                try {
                    connection.close();
                } catch (IOException e) {

                }
            }
            return null;
        }
    }
}
```

### 二、客户端Socket

客户端Socket有以下四个基本操作：

1. 连接远程机器
2. 发送数据
3. 接收数据
4. 关闭连接

创建一个Socket对象，它会在网络上建立连接，获得一个客户端Socket，格式如下：

```java
Socket socket = new Socket("localhost", 10086);
```

上面代码尝试连接端口号10086的localhost，具体的例子如下：

```java
public class SocketTest1 {
    public static void main(String[] args) throws Exception {
        Socket socket = new Socket("localhost", 10086);
        socket.setSoTimeout(15000);
        Writer out = new OutputStreamWriter(socket.getOutputStream());
        out.write("from client\r\n");
        out.flush();
        socket.shutdownOutput();
        InputStream in = socket.getInputStream();
        StringBuilder time = new StringBuilder();
        InputStreamReader reader = new InputStreamReader(in, "UTF-8");
        for(int c = reader.read(); c != -1; c = reader.read()) {
            time.append((char) c);
        }
        System.out.println(time);
    }
}
```

这个客户端Socket做了写入数据到服务器和从服务器接收数据的操作，分别通过输出流和输入流完成。

java.net.Socket类时Java完成客户端TCP操作的基础类，其他建立TCP网络连接的面向客户端的类（如URL、URLConnection等）最终都会调用这个类的方法。