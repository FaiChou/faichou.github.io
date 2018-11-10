---
layout: post
title: "从HTTP请求到Charles抓取HTTPS包"
date: 2017-04-08
---

------


#### 1. HTTP

HTTP是一个非常庞大的知识体，网上资料也铺天盖地，维基百科上解释***“HTTP(超文本传输协议)是互联网上应用最为广泛的一种网络协议，设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。通过HTTP或者HTTPS协议请求的资源由统一资源标识符（Uniform Resource Identifiers，URI）来标识。HTTP是一个客户端终端（用户）和服务器端（网站）请求和应答的标准”***。就像2000年**Roy Fielding**博士在他的论文中提出REST（Representational State Transfer）风格的软件架构模式成为了Web API的标准，HTTP的发展是由**Tim Berners-Lee**在CERN发起由W3C和IETF制定的一套规范。**蒂姆·伯纳斯-李**作为万维网的发明者，在去年(2016年)获得图灵奖。

本篇提供常用开发知识点，如果深入挖掘，TCP/IP, UDP,  OSI模型, 三次握手等知识点是挖掘不完的，就像一只爬虫，爬到多少是个头呢，所以本章简要总结**请求格式和响应格式**。

- 请求格式

  发出的请求信息（message request）包括请求行，请求头，空行，消息体。

  - 请求行

    请求行是请求消息的第一行，由三部分组成：分别是请求方法（GET/POST/DELETE/PUT/HEAD）、请求资源的URI路径、HTTP的版本号。

    `GET / HTTP/1.1`

  - 请求头

    请求头中的信息有和缓存相关的头（Cache-Control，If-Modified-Since）、客户端身份信息（User-Agent）等等。

    ```
    Host: faichou.space
    Connection: keep-alive
    Cache-Control: max-age=0
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/527.36 (KHTML, like Gecko) Chrome/57.0.2557.133 Safari/557.37
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Encoding: gzip, deflate, sdch
    Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4
    Cookie: _ga=GA1.2.1965313251.1464240344; _gat=1
    If-Modified-Since: Sun, 26 Mar 2017 14:29:48 GMT
    ```

  - 空行

    空的行。

  - 消息体

    请求体是客户端发给服务端的请求数据，这部分数据并不是每个请求必须的。



- 响应格式

  服务器接收处理完请求后返回一个HTTP响应消息给客户端。HTTP响应消息的格式包括：状态行、响应头、空行、消息体。每部分内容占一行。

  - 状态行

    状态行位于相应消息的第一行，有HTTP协议版本号，状态码和状态说明三部分构成。

    `HTTP/1.1 304 Not Modified`

  - 响应头

    响应头是服务器传递给客户端用于说明服务器的一些信息，以及将来继续访问该资源时的策略

    ```
    Server: GitHub.com
    Date: Sat, 08 Apr 2017 07:15:29 GMT
    Last-Modified: Sun, 26 Mar 2017 14:29:48 GMT
    Access-Control-Allow-Origin: *
    Expires: Sat, 08 Apr 2017 07:25:29 GMT
    Cache-Control: max-age=600
    X-GitHub-Request-Id: F804:6693:14F6B41:1C1E639:58E88E11
    ```

  - 空行

    空的行。

  - 消息体

    响应体是服务端返回给客户端的HTML文本内容，或者其他格式的数据，比如：视频流、图片或者音频数据。



#### 2. Cookie

> 指某些网站为了辨别用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密）。— wikipedia

Cookie解决了HTTP连接无状态的问题，是客户端和服务器共同保持维护一些状态数据。Cookie会被附加到http请求中，大多数浏览器都支持，无需用户做额外操作，当然用户可以手动删除cookie。cookie的最大长度是4KB左右。使用cookie可以维持用户的登录态，只需要在cookie中添加一个token就可以。

客户端请求信息，服务器响应返回，那么cookie被放在哪里呢？

通常是在响应头中，被放置在`Set-Cookie`字段中。这个字段浏览器会自动识别，并保存下来，下次请求时候会将cookie附加到请求头中去。

举个栗子



浏览器第一次给`www.jianshu.com`发送请求

```
GET / HTTP/1.1
Host: www.jianshu.com
...
```



服务器返回带有两个`Set-Cookie`字段的头

```
HTTP/1.1 200 OK
Server: Tengine
Date: Sat, 08 Apr 2017 07:38:22 GMT

Set-Cookie: theme=light
Set-Cookie: sessionToken=abc123; Expires=Wed, 09 Jun 2021 10:18:14 GMT
...
```

服务器返回的信息除了包含首页外，还有两个重要的头部信息`Set-Cookie`，它指示浏览器保存下来。

Cookie的分类

> Cookie总是保存在客户端中，按在客户端中的存储位置，可分为内存Cookie和硬盘Cookie。
> 内存Cookie由浏览器维护，保存在内存中，浏览器关闭后就消失了，其存在时间是短暂的。硬盘Cookie保存在硬盘里，有一个过期时间，除非用户手工清理或到了过期时间，硬盘Cookie不会被删除，其存在时间是长期的。所以，按存在时间，可分为非持久Cookie和持久Cookie。

按照以上分类，服务器返回的两个Cookie一个是内存Cookie(session cookie)另一个是硬盘Cookie(persistent cookie)。`theme`指定了网页的主题是亮色，它不包含`Expires`或`Max-Age`字段，故生存期短，浏览器关闭即死亡。`sessionToken`是客户端和服务器共同保持一个长登陆态的标志，它生存到`Wed, 09 Jun 2021 10:18:14 GMT`，所以说它是一个persistent cookie。



接下来，浏览器继续请求钱包页面，其中请求头中包含`Cookie`字段，字段中有上次服务器发来的`theme`和`sessionToken`

```
GET /wallet HTTP/1.1
Host: www.jianshu.com
Cookie: theme=light; sessionToken=abc123
...
```

这样服务器就知道这个用户的这次请求是和前一次请求是相关的。服务器再收到请求后，返回钱包页面，并且返回新的`Set-Cookie`给用户。这样就可以指示用户添加新cookie，更新cookie，或者删除cookie了。





#### 3. HTTPS

HTTPS，常称为HTTP over TLS，HTTP over SSL或HTTP Secure，它是一个网络安全传输协议。HTTPS经由HTTP通信，利用TSL/SSL来加密数据包。

在计算机网络知识中，TSL/SSL介于应用层和TCP层之间。应用层数据不再直接传递给传输层，而是传递给TSL/SSL层，TSL/SSL层对从应用层收到的数据进行加密，并增加自己的TSL/SSL头。

与HTTP的URL由`http://`起始且默认使用端口80不同，HTTPS的URL由`https://`起始且默认使用端口443。

HTTPS比HTTP多的就是安全，HTTP的数据是明文传输，在OSI模型中，任意一层捕获到数据，既可以一览无余，导致信息泄露，甚至可能被篡改，运营商流量劫持就是一个很好的例子。

在原理上是如何做到安全的呢？我们知道建立TCP连接需要三次握手，而TLS握手需要九次，而这个过程主要就是用来做非对称加密。这里引申出来两个重要概念，非对称加密和对称加密，理解它们有助于理解Charles抓HTTPS包。

- 对称加密

  对称加密就是加解密使用同一个密钥。客户端和服务器通信一般会动态生成一个密钥，所以密钥如何安全传递就成了问题。非对称加密正是为了解决这个问题，也就是**密钥交换（也叫密钥协商）**。

- 非对称加密

  浏览器和服务器每次新建会话时都使用非对称密钥交换算法协商出对称密钥，使用这些对称密钥完成应用数据的加解密和验证，整个会话过程中的密钥只在内存中生成和保存，而且每个会话的对称密钥都不相同（除非会话复用），中间者无法窃取。

  服务器有一对公钥和私钥，公钥用于加密，私钥用于解密。密钥交换过程：服务器的公钥是公开的，私钥是不公开的，只有服务器持有。浏览器先向服务器取得公钥，然后用公钥加密自己的私钥连同自己私钥加密的请求一并发送给服务器。服务器使用自己私钥解密得到浏览器的私钥，使用浏览器的私钥解密请求。然后再用浏览器的私钥加密response发送回服务器。

![非对称加密](https://raw.githubusercontent.com/FaiChou/faichou.github.io/master/img/qiniu/asymmetric%20cryptography.png)



#### 4. Charles

> Charles 是在 Mac 下常用的网络封包截取工具，在做移动开发时，我们为了调试与服务器端的网络通讯协议，常常需要截取网络封包来分析。
> Charles 通过将自己设置成系统的网络访问代理服务器，使得所有的网络访问请求都通过它来完成，从而实现了网络封包的截取和分析。
> 除了在做移动开发中调试端口外，Charles 也可以用于分析第三方应用的通讯协议。配合 Charles 的 SSL 功能，Charles 还可以分析 Https 协议。

传统的中间人攻击是无法攻击HTTPS的，那么Charles是如何抓取的呢？

之前一直好奇，总认为Charles不能获取服务器的秘钥，是怎么抓取解密到包呢？简直是魔法呀！

其实Charles也是个中间人，只不过它伪装成了服务器，也打扮成了客户端，是一个双重身份。Charles使用时我们需要手动添加证书信任。这样浏览器眼中的服务器就是Charles，而服务器眼中的客户端也是Charles，从而实现了双向数据解密。是不是很简单呢。

![Charles双向数据解密](https://raw.githubusercontent.com/FaiChou/faichou.github.io/master/img/qiniu/Charles%20crack%20https.png)





#### 5. Reference

1. [Charles 从入门到精通](http://blog.devtang.com/2015/11/14/charles-introduction/)
2. [charles是如何抓取https数据包的](http://legendtkl.com/2015/11/30/charles-https/)
3. [iOS http & https & 网络请求过程](http://www.jianshu.com/p/db9c716c3558)
4. [网络请求过程扫盲](http://www.jianshu.com/p/8a40f99da882)
5. [Hypertext Transfer Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
6. [HTTP cookie](https://en.wikipedia.org/wiki/HTTP_cookie)

