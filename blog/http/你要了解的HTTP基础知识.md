# 你要了解的HTTP基础知识

# 1. URL

统一资源定位符（Uniform Resource Locator）是网络资源的位置和访问方法的简洁表示。

常见的url包括4个部分，结构如下图：

![URL结构](http://upload-images.jianshu.io/upload_images/2438937-326b9cbe070b5994.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

| 组件    | 含义                            |
| ----- | ----------------------------- |
| 方案    | 使用的协议，如http或https             |
| 主机    | 服务器的域名或IP地址                   |
| 路径    | 服务器上资源的本地名，用（/）与前面的scheme分隔开来 |
| 查询字符串 | 通过查询字符串来减小请求资源类型的范围，如参数       |

url的详细介绍可参考：[统一资源定位符](https://en.wikipedia.org/wiki/Uniform_Resource_Locator)

# 2.报文

HTTP报文就是网络传输的信息，包括3个部分：**起始行、首部（Header）、主体**。
报文分2类：发送请求的报文称为请求报文，响应请求的报文称为响应报文

## 2.1 请求报文

![请求报文](http://upload-images.jianshu.io/upload_images/2438937-68732302a6b1ed44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.1.1 请求行

请求报文的起始行称为请求行，包含 **请求方法、地址、HTTP版本** 3个部分，格式为：

<method>[空格]<request url>[空格]<http version>

请求行以CRLF字符结束，请求方法、地址、HTTP版本之间以空格隔开。
如上面的请求报文中，请求方法为GET，地址为/static/search/baiduapp_icon.png，HTTP版本为1.1。

http最常用请求方法为GET和POST。

| 方法   | 作用                  |
| ---- | ------------------- |
| GET  | 从服务器获取资源            |
| POST | 向服务器发送需要处理的数据，如提交表单 |

### 2.1.2 请求首部（Header）

起始行后面有零个或多个首部字段，每个首部字段都是一个键值对，首部以一个空行结束。
Header可以向服务器提供一些额外的信息，比如客户端希望接收什么类型的数据。常见的请求Header如下：

| 首部              | 描述                 |
| --------------- | ------------------ |
| Host            | 请求的服务器主机名          |
| User-Agent      | 应用程序标识，如系统版本和浏览器版本 |
| Accept          | 客户端能够接受的数据类型       |
| Accept-Encoding | 客户端能够接受的编码方式       |
| Accept-Language | 客户端能够接受的语言         |
| Cookie          | 向服务器传递的cookie      |

### 2.1.3 请求主体

请求主体是可选的，接在首部的空行之后，包含了要发送给服务器的数据，主体中可以包含任意的文本或二进制数据，比如图片、视频、音轨、软件程序。

## 2.2 响应报文


![响应报文](http://upload-images.jianshu.io/upload_images/2438937-5ae2e57e9a7f472d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.2.1 响应行

响应报文的起始行称为响应行，包含 **HTTP版本、状态码、原因短语**（解析状态码的文本，可选） 3部分，格式为：

<version>[空格]<status>[空格]<reason-phrase>

响应行同样以CRLF字符结束，HTTP版本、状态码、原因短语之间以空格隔开。
如上面的响应报文中，HTTP版本为1.1，状态码为200，原因短语为OK。

**方法是客户端告诉服务器要什么事情，状态码则是服务端用来告诉客户端事情的处理结果。**
状态码是3位的整数值，上面的例子状态码为200，表示处理成功，状态码分类如下：

| 整体范围    | 分类         |
| ------- | ---------- |
| 100～199 | 信息提示       |
| 200～299 | 成功，如200    |
| 300～399 | 重定向        |
| 400～499 | 客户端错误，如404 |
| 500～599 | 服务器错误，如500 |

详细状态码分类可参考：[HTTP状态码详解](http://tool.oschina.net/commons?type=5)

### 2.2.2 响应首部（Header）

和请求首部一样，响应首部也是由键值对构成，以空行结束，提供更多有关响应的信息，常见的响应首部如下：

| 首部             | 描述       |
| -------------- | -------- |
| Content-Length | 主体的长度或尺寸 |
| Content-Type   | 主体的对象类型  |
| Date           | 服务器时间    |
| Server         | 服务器类型    |
| Set-Cookie     | 设置Cookie |

### 2.2.3 响应主体

与请求主体一样，响应主体也是可选的，接在首部的空行之后，包含了要发送给客户端的数据，可以包含任意的文本或二进制数据。

<br>
下一篇：[关于HTTPS，你需要知道的全部](https://github.com/rushgit/zhongwenjun.github.com/blob/master/blog/http/%E5%85%B3%E4%BA%8EHTTPS%EF%BC%8C%E4%BD%A0%E9%9C%80%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E5%85%A8%E9%83%A8.md)