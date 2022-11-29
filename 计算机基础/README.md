# 计算机基础

## 1. http 状态码 204 301 302 304 400 401 403 404 含义

- http 状态码 204 （无内容） 服务器成功处理了请求，但没有返回任何内容
- http 状态码 301 （永久移动） 请求的网页已永久移动到新位置。 服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。
- http 状态码 302 （临时移动） 服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
- http 状态码 304 （未修改） 自从上次请求后，请求的网页未修改过。 服务器返回此响应时，不会返回网页内容。
- http 状态码 400 （错误请求） 服务器不理解请求的语法（一般为参数错误）。
- http 状态码 401 （未授权） 请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。
- http 状态码 403 （禁止） 服务器拒绝请求。（一般为客户端的用户权限不够）
- http 状态码 404 （未找到） 服务器找不到请求的网页。

## 2. http2.0 做了哪些改进 3.0 呢

http2.0 特性

- 二进制分帧传输
- 多路复用
- 头部压缩
- 服务器推送

`http` 协议是应用层协议，都是建立在传输层之上的。我们也都知道传输层上面不只有 `TCP` 协议，还有另外一个强大的协议 `UDP` 协议，`2.0` 和 `1.0` 都是基于 `TCP` 的，因此都会有 `TCP` 带来的硬伤以及局限性。而 `Http3.0` 则是建立在 `UDP` 的基础上。所以其与 `Http2.0` 之间有质的不同。

http3.0 特性

- 连接迁移
- 无队头阻塞
- 自定义的拥塞控制
- 前向安全和前向纠错


[Http2.0的一些思考以及Http3.0的优势](https://blog.csdn.net/m0_60360320/article/details/119812431)

## 3. https 加密过程是怎样的

使用了对称加密和非对称加密的混合方式。

[HTTPS](https://juejin.cn/post/6844904150115827725)

## 4. 304 是什么意思 一般什么场景出现 ，命中强缓存返回什么状态码

协商缓存命中返回 304

这种方式使用到了 headers 请求头里的两个字段，Last-Modified & If-Modified-Since 。服务器通过响应头 Last-Modified 告知浏览器，资源最后被修改的时间：
Last-Modified: Thu, 20 Jun 2019 15:58:05 GMT

当再次请求该资源时，浏览器需要再次向服务器确认，资源是否过期，其中的凭证就是请求头 If-Modified-Since 字段，值为上次请求中响应头 Last-Modified 字段的值：
If-Modified-Since: Thu, 20 Jun 2019 15:58:05 GMT

浏览器在发送请求的时候服务器会检查请求头 request header 里面的 If-modified-Since，如果最后修改时间相同则返回 304，否则给返回头(response header)添加 last-Modified 并且返回数据(response body)。

另外，浏览器在发送请求的时候服务器会检查请求头(request header)里面的 if-none-match 的值与当前文件的内容通过 hash 算法（例如 nodejs: cryto.createHash('sha1')）生成的内容摘要字符对比，相同则直接返回 304，否则给返回头(response header)添加 etag 属性为当前的内容摘要字符，并且返回内容。

综上总结为：
1. 请求头last-modified的日期与响应头的last-modified一致
2. 请求头if-none-match的hash与响应头的etag一致
3. 这两种情况会返回Status Code: 304


强缓存命中返回 200
200（from cache）


## 5. 前端性能优化之网络

- 减少 HTTP 请求数量
- 利用浏览器缓存，公用依赖包（如vue、Jquery、ui组件等）单独打包/单文件在一起，避免重复请求
- 减小cookie大小，尽量用localStorage代替
- CDN托管静态文件
- 开启 Gzip 压缩
- dns预解析
  ```html
  <meta http-equiv="x-dns-prefetch-control" content="on" />
  <link rel="dns-prefetch" href="http://xxx.com/" />
  ```

## 6. 网络七层协议（OSI模型）

OSI是一个定义良好的协议规范集，并有许多可选部分完成类似的任务。它定义了开放系统的层次结构、层次之间的相互关系以及各层所包括的可能的任务，作为一个框架来协调和组织各层所提供的服务。
OSI参考模型并没有提供一个可以实现的方法，而是描述了一些概念，用来协调进程间通信标准的制定。即OSI参考模型并不是一个标准，而是一个在制定标准时所使用的概念性框架。

第7层 应用层

应用层（Application Layer）提供为应用软件而设计的接口，以设置与另一应用软件之间的通信。例如：HTTP、HTTPS、FTP、Telnet、SSH、SMTP、POP3等。

第6层 表示层

表示层（Presentation Layer）把数据转换为能与接收者的系统格式兼容并适合传输的格式。

第5层 会话层

会话层（Session Layer）负责在数据传输中设置和维护计算机网络中两台计算机之间的通信连接。

第4层 传输层

传输层（Transport Layer）把传输表头（TH）加至数据以形成数据包。传输表头包含了所使用的协议等发送信息。例如:传输控制协议（TCP）等。

第3层 网络层

网络层（Network Layer）决定数据的路径选择和转寄，将网络表头（NH）加至数据包，以形成分组。网络表头包含了网络资料。例如:互联网协议（IP）等。

第2层 数据链路层

数据链路层（Data Link Layer）负责网络寻址、错误侦测和改错。当表头和表尾被加至数据包时，会形成信息框（Data Frame）。数据链表头（DLH）是包含了物理地址和错误侦测及改错的方法。数据链表尾（DLT）是一串指示数据包末端的字符串。例如以太网、无线局域网（Wi-Fi）和通用分组无线服务（GPRS）等。
分为两个子层：逻辑链路控制（logical link control，LLC）子层和介质访问控制（Media access control，MAC）子层。

第1层 物理层

物理层（Physical Layer）在局部局域网上发送数据帧（Data Frame），它负责管理电脑通信设备和网络媒体之间的互通。包括了针脚、电压、线缆规范、集线器、中继器、网卡、主机接口卡等。

## 7. TCP 协议三次握手


- 客户端通过 SYN 报文段发送连接请求，确定服务端是否开启端口准备连接。状态设置为 SYN_SEND;
- 服务器如果有开着的端口并且决定接受连接，就会返回一个 SYN+ACK 报文段给客户端，状态设置为 SYN_RECV；
- 客户端收到服务器的 SYN+ACK 报文段，向服务器发送 ACK 报文段表示确认。此时客户端和服务器都设置为 ESTABLISHED 状态。连接建立，可以开始数据传输了。


## 8.  TCP 协议四次挥手

- Client端发起挥手请求，向Server端发送标志位是FIN报文段，设置序列号seq，此时，Client端进入FIN_WAIT_1状态，这表示Client端没有数据要发送给Server端了。
- Server端收到了Client端发送的FIN报文段，向Client端返回一个标志位是ACK的报文段，ack设为seq加1，Client端进入FIN_WAIT_2状态，Server端告诉Client端，我确认并同意你的关闭请求。
- Server端向Client端发送标志位是FIN的报文段，请求关闭连接，同时server端进入LAST_ACK状态。
-  Client端收到Server端发送的FIN报文段，向Server端发送标志位是ACK的报文段，然后Client端进入TIME_WAIT状态。Server端收到Client端的ACK报文段以后，就关闭连接。此时，Client端等待2MSL的时间后依然没有收到回复，则证明Server端已正常关闭，那好，Client端也可以关闭连接了。

为什么不是两次？
两次情况客户端说完结束就立马断开不再接收，无法确认服务端是否接收到断开消息，并且服务端可能还有消息未发送完。
为什么不是三次？
3次情况服务端接收到断开消息，向客户端发送确认接受消息，客户端未给最后确认断开的回复。

## 9. DNS解析是去哪找的缓存？

1. 查找浏览器缓存
2. 查找系统缓存
3. 查找路由器的缓存
4. 查找isp DNS缓存
5. 递归搜索

## 10. 怎么找到DNS服务器？

递归迭代查询

## 11. DNS怎么解析出IP的？

1. 本地DNS服务器
2. 根名称服务器
3. 顶级名称服务器
4. 二级名称服务器
5. 权威名称服务器


DNS解析查找

> (1)本地 DNS服务器即将该请求转发到互联网上的根域（即一个完整域名最后面的那个点，通常省略不写）。  
>(2)根域将所要查询域名中的顶级域（假设要查询ke.qq.com，该域名的顶级域就是com）的服务器IP地址返回到本地DNS。   
> (3) 本地DNS根据返回的IP地址，再向顶级域（就是com域）发送请求。   
> (4) com域服务器再将域名中的二级域（即ke.qq.com中的qq）的IP地址返回给本地DNS。   
> (5) 本地DNS再向二级域发送请求进行查询。   
> (6) 之后不断重复这样的过程，直到本地DNS服务器得到最终的查询结果，并返回到主机。这时候主机才能通过域名访问该网站。

[用户在浏览器的地址栏中敲入了网站的网址 ，会发生哪些事情呢？](https://imweb.io/topic/55e3ba46771670e207a16bc8)

## 12. 解析出ip地址后怎么找到对方？
使用百度api接口


## 13. 握手为什么要三次？万一第三次没有发出去呢？

确保通讯双方的发送接收能力。

第三次发送失败：

**Server 端**

第三次的ACK在网络中丢失，那么Server 端该TCP连接的状态为SYN_RECV,并且会根据 TCP的**超时重传机制**，会等待3秒、6秒、12秒后重新发送SYN+ACK包，以便Client重新发送ACK包。

而Server重发SYN+ACK包的次数，可以通过设置/proc/sys/net/ipv4/tcp_synack_retries修改，默认值为5.

如果重发指定次数之后，仍然未收到 client 的ACK应答，那么一段时间后，Server自动关闭这个连接。


**Client 端**

在linux c 中，client 一般是通过 connect() 函数来连接服务器的，而connect()是在 TCP的三次握手的第二次握手完成后就成功返回值。也就是说 client 在接收到 SYN+ACK包，它的TCP连接状态就为 established （已连接），表示该连接已经建立。那么如果 第三次握手中的ACK包丢失的情况下，Client认为这个连接已经建立， 向 server端发送数据，Server端将以 `RST`包响应，而是直接发送RST报文段，进入CLOSED状态。这样做的目的是为了防止SYN洪泛攻击。

## 14. 说说https和http的区别

http具有以下风险
- 窃听风险（eavesdropping）：通信使用明文(不加密)，内容可能被窃听。

- 篡改风险（tampering）：无法证明报文的完整性，所以可能遭篡改

- 冒充风险（pretending）：不验证通信方的身份，因此有可能遭遇伪装

- HTTP 是明文传输协议，HTTPS 协议是由 SSL+HTTP 协议构建的可进行加密传输、身份认证的网络协议，比 HTTP 协议安全。
- HTTPS比HTTP更加安全，对搜索引擎更友好，利于SEO,谷歌、百度优先索引HTTPS网页;
- HTTPS需要用到SSL证书，而HTTP不用;
- HTTPS标准端口443，HTTP标准端口80;
- HTTPS在浏览器显示绿色安全锁，HTTP没有显示;
- http是无状态的

## 15. 证书颁发的流程

## 16. 常用请求头、响应头

**请求头：**

- Accept：浏览器可接受的MIME类型；

- Accept-Charset：浏览器可接受的字符集；

- Accept-Encoding：浏览器能够进行解码的数据编码方式，比如gzip。Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。许多情形下这可以减少5到10倍的下载时间；

- Accept-Language：浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到；

- Authorization：授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中；

- Connection：表示是否需要持久连接。如果Servlet看到这里的值为“Keep-Alive”，或者看到请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接），它就可以利用持久连接的优点，当页面包含多个元素时（例如Applet，图片），显著地减少下载所需要的时间。要实现这一点，Servlet需要在应答中发送一个Content-Length头，最简单的实现方法是：先把内容写入ByteArrayOutputStream，然后在正式写出内容之前计算它的大小；

- Content-Length：表示请求消息正文的长度；

- Cookie：这是最重要的请求头信息之一；

- From：请求发送者的email地址，由一些特殊的Web客户程序使用，浏览器不会用到它；

- Host：初始URL中的主机和端口；

- If-Modified-Since：只有当所请求的内容在指定的日期之后又经过修改才返回它，否则返回304“Not Modified”应答；

- Pragma：指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝；

- Referer：包含一个URL，用户从该URL代表的页面出发访问当前请求的页面。

- User-Agent：浏览器类型，如果Servlet返回的内容与浏览器类型有关则该值非常有用；

- UA-Pixels，UA-Color，UA-OS，UA-CPU：由某些版本的IE浏览器所发送的非标准的请求头，表示屏幕大小、颜色深度、操作系统和CPU类型。

**响应头：**

- Allow：服务器支持哪些请求方法（如GET、POST等）；

- Content-Encoding：文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader("Accept-Encoding")）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面；

- Content-Length：表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入ByteArrayOutputStram，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo(response.getOutputStream()发送内容；

- Content-Type： 表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentTyep。 可在web.xml文件中配置扩展名和MIME类型的对应关系；

- Date：当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦；

- Expires：指明应该在什么时候认为文档已经过期，从而不再缓存它。

- Last-Modified：文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置；

- Location：表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302；

- Refresh：表示浏览器应该在多少时间之后刷新文档，以秒计。

- set-cookie: 设置cookie

- etag: 文件hash值


## 17. 线程和进程

进程：指在系统中正在运行的一个应用程序；程序一旦运行就是进程；进程——资源分配的最小单位。

线程：系统分配处理器时间资源的基本单元，或者说进程之内独立执行的一个单元执行流。线程——程序执行的最小单位。

## http 预检
非简单请求 会在正式通信之前，增加一次HTTP请求，称之为预检请求。浏览器会先发起OPTIONS方法到服务器，以获知服务器是否允许该实际请求。

大致说明一下,有三种方式会导致这种现象:
1. 请求的方法不是GET/HEAD/POST，还有PUT,DELETE

2. POST请求的Content-Type并非application/x-www-form-urlencoded, multipart/form-data, 或text/plain

3. 请求设置了自定义的header字段，我们的Content-Type绝大多数是application/json

## HTTP报文
用于HTTP协议交互的信息被称为HTTP报文。客户端的HTTP报文叫请求报文，服务端的HTTP报文叫响应报文。

请求报文 是由请求行（请求方法、协议版本）、请求首部（请求URI、客户端信息等）和内容实体（用户信息和资源信息等，可为空）构成。

响应报文 是由状态行（协议版本、状态码）、响应首部（服务器名称、资源标识等）和内容实体（服务端返回的资源信息）构成。