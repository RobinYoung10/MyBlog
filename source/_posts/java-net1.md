---
title: Java网络编程学习（一）
date: 2019-04-17 13:17:11
tags: java网络编程
---

### 一、URL类

java.net.URL类市对统一资源定位符的抽象。它是一个final类。URL是不可变的。构造一个URL对象后，其字段不再改变。

##### 创建新的URL

最简单的URL构造函数接受一个字符串形式的绝对URL作为唯一参数，如下：

```java
URL u = new URL("https://www.google.com/");
```

还可以通过指定协议、主机名和文件来构建URL：

```java
URL u = new URL("https", "github.com", "/RobinYoung10");
```

上面代码没有指定端口，则会使用该协议的默认端口。有时候默认端口不正确时，需要显式指定端口，如下：

```java
URL u = new URL("https", "github.com", 8000, "/RobinYoung10");
```

可以根据相对URL和基础URL构建URL：

```java
URL u1 = new URL("https://github.com/");
URL u2 = new URL(u1, "/RobinYoung10");
```

##### 从URL获取数据

1.openStream()方法，返回一个InputStream，可以从这个流读取数据。如下代码：

```java
public class URLTest {
    public static void main(String[] args) {

        InputStream in = null;
        try {
            //打开url进行读取
            URL U = new URL("https://github.com/");
            URL u = new URL(U, "/RobinYoung10");
            in = u.openStream();
            //缓冲输入提高性能
            in = new BufferedInputStream(in);
            //将InputStream串链到一个Reader
            Reader r = new InputStreamReader(in);
            StringBuilder stringBuilder = new StringBuilder();
            int c;
            while((c = r.read()) != -1) {
                //System.out.print((char) c);
                stringBuilder.append((char) c);
            }
            String str = stringBuilder.toString();
            System.out.println(str);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

上述代码获取网页的代码，并放入String对象中打印出来。

2.openConnection()方法，返回一个URLConnection对象。

其他URL类的方法具体参考Java文档。

### URLConnection

URLConnection是一个抽象类，表示指向URL指定资源的活动链接。URLConnection可以检查服务器发送的首部和内容，并相应的做出响应。还可以用post、put和其他http方法向服务器发回数据。

##### 1.打开URLConnection

上一部分最后的内容就说到，构造一个URL对象，调用这个URL对象的openConnection()方法获取一个URLConnection对象。如下：

```java
URL u = new URL("https://github.com");
URLConnection uc = u.openConnection();
```

##### 2.读取服务器的数据

使用URLConnection的getInputStream()方法，获取一个InputStream，可以读取和解析服务器发送的数据。这与上一部分URL获取数据差不多，而URL的openStream()方法就是从它自己的URLConnection对象返回的一个InputStream。

##### 3.获取首部字段

* public String getContentType()

返回响应主题的MIME内容类型，如text/html、image/jpeg。

* public int getContentLength()

告诉你内容有多少字节。如果发现资源的大小超过最大的int值（约21亿字节）时，会返回-1。还有一个getContentLengthLong()方法，返回的是long类型。

* public String getHeaderField(String name)

该方法返回指定首部字段的值。如：

```java
String contentType = uc.getHeaderField("content-type");
String date = uc.getHeaderField("date");
```

其他方法具体参考Java文档。

以下是URLConnection使用示例：

```java
public class URLConnectionTest2 {
    public static void main(String[] args) {
        try {
            URL u = new URL("http://114.115.168.26:8080/ncpsy/static/img/banner.jpg");
            URLConnection uc = u.openConnection();
            String contentType = uc.getContentType();
            System.out.println(contentType);
            int contentLength = uc.getContentLength();
            System.out.println(contentLength);
            byte data[] = new byte[contentLength];
            int len = 0;
            try (InputStream in = uc.getInputStream()) {
                FileOutputStream fos = new FileOutputStream("D:\\banner.jpg");
                while((len = in.read(data)) != -1) {
                    fos.write(data, 0, len);
                }
                in.close();
                fos.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

##### 4.配置连接

URLConnection类有7个保护的实例字段，定义了客户端如何向服务器做出请求，有些字段有getter和setter方法，用于获取和设置。字段包括:

* protected URL url

连接的url，此字段不能改变，没有setter方法，只有getter方法。

* protected boolean connected

如果连接打开，为true，连接关闭，则为false。没有getter和setter方法。

* protected boolean allowUserInteraction

表示是否允许用户交互。

* protected boolean doInput

表示是否可以用来读取服务器，默认为true。

* protected boolean doOutput

表示是否可以将输出发回服务器。例如程序需要使用post方法向服务器发送数据，可通过URLConnection获取输出流来完成。默认为false。为一个http URL将doOutput设置为true时，请求方法会有get转为post。

* protected boolean ifModifiedSince

判断服务器是否发生改变。

* protected boolean useCaches

是否使用缓存，默认为true。

##### 5.配置客户端请求HTTP首部

在打开连接之前，可以使用setRequestProperty()方法设置HTTP首部字段，如下：

```java
uc.setRequestProperty("Content-Type", "application/json;charset=UTF-8");
```

还可以为一个字段增加新值，使用addRequestProperty()方法：

```java
uc.addRequestProperty("Accept", "application/json");
```

###### 6.向服务器写入数据

我们来看一个通过URLConnection发送json数据到服务器并打印返回数据的例子：

```java
public class URLConnectionTest3 {
    public static void main(String[] args) throws Exception {
        URL u = new URL("http://114.115.168.26:8080/ncpsy/handle/login");
        URLConnection uc = u.openConnection();

        //设置doOutput为true让URLConnection使用post方法发送数据
        uc.setDoOutput(true);
        //添加Accept属性值 application/json
        uc.addRequestProperty("Accept", "application/json");
        //设置数据类型
        uc.setRequestProperty("Content-Type", "application/json;charset=UTF-8");
        //uc.setRequestProperty("Charset", "UTF-8");

        //构造json对象,用到com.alibaba.fastjson
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("zh", "qiye2");
        jsonObject.put("mm", "qiye2");

        //获取URLConnection输出流
        OutputStream raw = uc.getOutputStream();
        OutputStream buffered = new BufferedOutputStream(raw);
        OutputStreamWriter out = new OutputStreamWriter(buffered, "UTF-8");
        out.write(jsonObject.toJSONString());
        out.flush();
        out.close();
        System.out.println(uc.getContentType());
        //获取输入流，打印返回的数据
        try (InputStream in = uc.getInputStream()) {
            InputStreamReader reader = new InputStreamReader(in);
            BufferedReader bufferedReader = new BufferedReader(reader);
            int r;
            while((r = bufferedReader.read()) != -1) {
                System.out.print((char) r);
            }
            bufferedReader.close();
        }
        out.close();
    }
}
```

我们需要在连接之前设置好Accept、Content-Type信息，并构造一个json对象，通过获取输出流写入数据发送到服务器，通过获取输入流打印返回的数据。