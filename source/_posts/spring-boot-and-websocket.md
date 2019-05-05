---
title: Spring Boot整合WebSocket
date: 2019-04-29 17:55:19
tags: Spring boot
---

### 一、Maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

### 二、配置WebSocket

这步很简单，只要几行代码就可以搞定

```java
@Configuration
public class WebSocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

### 三、WebSocket服务类

在服务类上声明注解@ServerEndpoint就可以将类变成websocket服务类，加上@Component注解让spring可以自动加载。在类里面，需要实现的方法主要有：

* 连接建立成功调用的方法（@OnOpen注解声明）
* 连接关闭调用的方法（@OnClose注解声明）
* 收到客户端消息后调用的方法（@OnMessage注解声明）

如下代码：

```java
@ServerEndpoint("/websocket/{sid}")
@Component
public class WebSocketServer {

    protected static Logger log = LoggerFactory.getLogger(WebSocketServer.class);
    //静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    private static int onlineCount = 0;
    //concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
    private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<WebSocketServer>();

    //与某个客户端的连接会话，需要通过它来给客户端发送数据
    private Session session;

    //接收sid
    private String sid="";
    /**
     * 连接建立成功调用的方法*/
    @OnOpen
    public void onOpen(Session session,@PathParam("sid") String sid) {
        this.session = session;
        webSocketSet.add(this);     //加入set中
        addOnlineCount();           //在线数加1
        log.info("有新窗口开始监听:"+sid+",当前在线人数为" + getOnlineCount());
        this.sid=sid;
        try {
            sendMessage("连接成功");
        } catch (IOException e) {
            log.error("websocket IO异常");
        }
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        webSocketSet.remove(this);  //从set中删除
        subOnlineCount();           //在线数减1
        log.info("有一连接关闭！当前在线人数为" + getOnlineCount());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息*/
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("收到来自窗口"+sid+"的信息:"+message);
        //群发消息
        for (WebSocketServer item : webSocketSet) {
            try {
                item.sendMessage(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     *
     * @param session
     * @param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }
    /**
     * 实现服务器主动推送
     */
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }


    /**
     * 群发自定义消息
     * */
    public static void sendInfo(String message,@PathParam("sid") String sid) throws IOException {
        log.info("推送消息到窗口"+sid+"，推送内容:"+message);
        for (WebSocketServer item : webSocketSet) {
            try {
                //这里可以设定只推送给这个sid的，为null则全部推送
                if(sid==null) {
                    item.sendMessage(message);
                }else if(item.sid.equals(sid)){
                    item.sendMessage(message);
                }
            } catch (IOException e) {
                continue;
            }
        }
    }

    public static synchronized int getOnlineCount() {
        return onlineCount;
    }

    public static synchronized void addOnlineCount() {
        WebSocketServer.onlineCount++;
    }

    public static synchronized void subOnlineCount() {
        WebSocketServer.onlineCount--;
    }
}
```

从onOpen方法可以看出，我们将接收的路径/{sid}中sid的值设置为每个session会话的唯一标识，用来识别每个客户端。

在这个类里面，我们自定义了一个sendMessage方法用来实现从服务器推送消息至客户端，实现的代码为：

```java
this.session.getBasicRemote().sendText(message);
```

### 自定义Controller接收消息并调用WebSocketServer

我们可以写一个controller来接收http连接，并在方法里调用WebSocketServer：

```java
@Controller
public class WebSocketController {

    //推送数据接口
    @ResponseBody
    @RequestMapping("/socket/push/{cid}")
    public boolean pushToWeb(@PathVariable String cid, String message) {
        try {
            WebSocketServer.sendInfo(message,cid);
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
}
```

### 页面发起Socket请求

我们首先配置Spring Boot的试图模板，这里用到thymeleaf模板，在pom.xml引入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

接着在配置文件application.properties声明：

```
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.thymeleaf.cache=false
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.servlet.content-type=text/html
```

然后编写html页面文件，新建index.html，编写如下代码:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
hello index

<script>
    var socket;
    if(typeof(WebSocket) == "undefined") {
        console.log("您的浏览器不支持WebSocket");
    }else{
        console.log("您的浏览器支持WebSocket");
        //实现化WebSocket对象，指定要连接的服务器地址与端口  建立连接
        //等同于socket = new WebSocket("ws://localhost:8083/checkcentersys/websocket/20");
        socket = new WebSocket("ws://localhost:8888/websocket/sj11");
        //打开事件
        socket.onopen = function() {
            console.log("Socket 已打开");
            socket.send("这是来自客户端的消息" + location.href + new Date());
        };
        //获得消息事件
        socket.onmessage = function(msg) {
            console.log(msg.data);
            //发现消息进入    开始处理前端触发逻辑
        };
        //关闭事件
        socket.onclose = function() {
            console.log("Socket已关闭");
        };
        //发生了错误事件
        socket.onerror = function() {
            alert("Socket发生了错误");
            //此时可以尝试刷新页面
        }
        //离开页面时，关闭socket
        //jquery1.8中已经被废弃，3.0中已经移除
        // $(window).unload(function(){
        //     socket.close();
        //});
    }
</script>
</body>
</html>
```

通过新建WebSocket对象并配置连接地址，逐个声明打开连接、接收消息、关闭连接和发生错误事件，即可完成配置。这里我的WebSocket链接是ws://localhost:8888/websocket/sj11，最后一个sj11就会成为服务里面的sid。

在Controller里面配置这个页面地址，启动Spring Boot，访问页面，按f12查看输出：

```
您的浏览器支持WebSocket
Socket 已打开
连接成功
这是来自客户端的消息http://localhost:8888/indexMon Apr 29 2019 17:51:41 GMT+0800 (中国标准时间)
```

打开新窗口，访问http://localhost:8888/socket/push/sj11?message=hhasdasdf，可以index页面看到输出：

```
hhasdasdf
```