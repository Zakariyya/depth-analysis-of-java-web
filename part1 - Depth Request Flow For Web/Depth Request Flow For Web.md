# 深入Web请求过程

[TOC]



## B/S网络架构概述

**B/S**网络架构从前端到后端都得到了简化，都基于同一的应用层协议HTTP来交互数据，与大多数传统**C/S**互联网应用程序采用的长连接的交互模式不同，HTTP采用无状态的短链接的通讯方式，通常情况下，一次请求就完成了一次数据交互，通常也对应一个业务逻辑，然后这次通讯连接就会断开了。采用这种方式是为了能同时服务更多的用户，因为当前互联网应用每天都会处理上亿的用户请求，不可能每个用户访问一次后就一直保持这个连接。

不管网络架构如何变化，始终有一些固定不变的原则需要遵守。

- 互联网上所有资源都要用一个URL来表示。URL就是统一资源定位符，如果你要发布一个服务或者一个资源到互联网上，让别人能够访问到，那么你首先必须要有一个世界上独一无二的URL。不要小看这个URL，它几乎包含了整个互联网架构的精髓。
- 必须基于HTTP与服务端交互。不管你要访问的是国内的还是国外的数据，是文本数据还是流媒体，都必须按照套路出牌，也就是都得采用统一打招呼的方式，这样人家才会明白你要的是什么。
- 数据展示必须在浏览器中进行。当你获取到数据资源后，必须在浏览器上才能恢复它的容貌。

只要满足上面几点，一个互联网应用基本上就能正确地运转起来了，当然这里面还有好多细节，这些细节在后面将分别进行详细讲解。

## 如何发起一个请求

如何发起一个HTTP请求和如何建立一个**Socket**连接区别不大，只不过**outputStream.write** 写的二进制字节数据格式要符合HTTP。浏览器在建立Socket连接之前，必须根据地址栏里输入URL的域名DNS解析出IP地址，再根据这个IP地址和默认的80端口与远程服务器建立Socket连接，然后浏览器根据这个**URL**组装成一个get类型的HTTP请求头通过**outputStream**.**write**发送到目标服务器，服务器等待i**nputStream.read** 返回数据，最后断开这个连接。

当然，不同浏览器在如何使用这个已经建立好的连接以及根据什么规则来管理连接上，有各种不同的实现方法。一句话，发起一个**HTTP**请求的过程就是建立**Socket**通信的过程。

既然发起一个**HTTP**连接本质就是建立一个**Socket**连接，那么我们完全可以模拟浏览器来发起HTTP请求，这很好实现，也有很多方法实现，如**HttpClient**就是一个开源的通过程序实现的处理**HTTP**请求的工具包。当然如果你对HTTP的数据结构非常熟悉，你完全可以自己再实现另外一个**HttpClient**，甚至可以自己写个简单的浏览器。

下面是基本的**HttpClient**的调用示例：

~~~java
HttpClient httpClient = createHttpClient();
PostMethod postMethod;
String domainName = Switcher.domain;
postMethod = new PostMethod(domainName);
postMethod.addRequestHeader("Content-Type","application/x-www-from-urlencoded; charset=GBK");
for(FilterData filterData:filterDatas){
    postMethod.addParameter("ip",filterData.ip);
    postMethod.addParameter("count",String.valueOf(filterData.count));
}
try{
    httpClient.executeMethod(postMethod);
    postMethod.getResponseBodyAsString();
}catch(Exception e){
    logger.error(e);
}
~~~

处理Java中使用非常普遍的HttpClient还有很多类似的工具，如linux中的curl命令，通过curl+URL 就可以简单地发起HTTP请求，非常方便。

## HTTP解析

**B/S**网络结构的核心是HTTP，掌握HTTP对一个从事互联网的程序员来说是非常重要的，也许你已经非常熟悉HTTP。

要理解HTTP，最重要的就是熟悉HTTP中**HTTP Header**，HTTP Header控制这互联网上成千上万的用户的数据的传输。关键的是，它控制着用户浏览器的渲染行为和服务器的执行逻辑。例如，当服务器没有用户请求的数据时就会返回一个**404**状态码，告知浏览器没有请求的数据，通常浏览器会展示一个非常不愿意看到的“**该页面不存在**”的错误信息。

常见的HTTP请求头和响应头分别如表1、2。

常见的额HTTP状态码如表3

常见的HTTP请求头

| 请求头             | 说明                                       |
| --------------- | ---------------------------------------- |
| Accept-Charset  | 用于指定客户端接收的字符集                            |
| Accept-Encoding | 用于指定可接受的内容编码，如Accept-Encoding:gzip.deflate |
| Accept-language | 用于指定一种自然语言，如Accept-language:zh-cn        |
| Host            | 用于指定被请求资源的Internet主机和端口号，如Host:www.taobao.com |
| User-Agent      | 客户端将它的操作系统，浏览器和其他属性告诉服务器                 |
| Connextion      | 当前连接是否保持，如Connetion:Keep-Alive           |

常见的HTTP响应头

| 响应头              | 说明                                       |
| ---------------- | ---------------------------------------- |
| Server           | 使用服务器名称，如Server:Apache/1.3.6(Unix)       |
| Content-Type     | 用来指明发送给接收者的实体正文的媒体类型，如Content-Type:text/html;charset=GBK |
| Content-Encoding | 与请求包头Accept-Encoding对应，告诉浏览器服务端采用的是什么压缩编码 |
| Content-language | 描述了资源所用的自然语言。与Accept-language对应          |
| Content-length   | 指明实体正文的长度，用以字节方式储存的十进制数字来表示              |
| Keep-Alive       | 保持连接的时间，如keep-Alive:timeout=5,max=120    |

常见的HTTP状态码

| 状态码  | 说明                     |
| ---- | ---------------------- |
| 200  | 客户端请求成功                |
| 302  | 临时跳转，跳转的地址通过location指定 |
| 400  | 客户端请求有语法错误，不能被服务器识别    |
| 403  | 服务器收到请求，但是拒绝提供服务       |
| 404  | 请求的资源不存在               |
| 500  | 服务器发生不可预期的错误           |

***

### 查看HTTP信息的工具

chrome：F12

firefox/IE:懒得看了，不用这两玩意

***

### 浏览器缓存机制

浏览器缓存是一个比较复杂但又是比较重要的机制，在我们浏览一个页面时发现有一场的情况下，通常考虑的就是是不是浏览器做了缓存，所以一般的做法是按Ctrl+F5 重新请求一次这个页面，重新请求的页面肯定是最新的页面。

> 首先在浏览器，如果是按ctrl+F5刷新页面，浏览器会直接项目标URL发送请求，而不会使用浏览器缓存的数据；
>
> 其次，即使请求发生到服务端，也有可能访问的是缓存数据，比如，在我们的应用服务器的前端部署一个缓存服务器，如[Varnish](https://zh.wikipedia.org/wiki/Varnish_cache)代理，那么Varnish也有可能直接使用缓存数据

所以为了保证用户能够看到最新的数据，必须通过HTTP来控制。

当我们使用ctrl+F5刷新一个页面时，在HTTP的请求头会增加一些请求头，它告诉服务器我们要获取最新的数据而不是缓存。

> 尝试这查看下普通刷新和ctrl+F5之后返回的请求头

**最重要的是在其请求头中增加了两个请求项 Pragma:no-cache 和 Cache-Control:no-cache。**

1.  **Cache-Control/Pragma:no-cache**

这个HTTP Head 字段用于指定所有缓存机制在整个请求/相应链中必须服从的指令，如果知道该页面是否为缓存，不仅可以控制浏览器，还可以控制和HTTP相关的缓存或代理服务器。HTTP Head字段有一些可选值，这些值说明表如下

| 可选值                                  | 说明                                       |
| ------------------------------------ | ---------------------------------------- |
| Public                               | 所有内容都将被缓存，在响应头中设置                        |
| Private                              | 内容只缓存到私有缓存中，在响应头中设置                      |
| no-cache                             | 所有内容都不会被缓存，在请求头和响应头中设置                   |
| no-store                             | 所有内容都不会被缓存到缓存或Internet临时文件中，在响应头中设置      |
| must-revalidation/proxy-revalidation | 如果缓存的内容失败，请求必须发送到服务器/代理服务器以进行重新验证，在请求头中设置 |
| max-age=xxx                          | 缓存的内容将在xxx秒后失效，这个选项只在HTTP1.1中可用，和Last-Modified一起使用时优先级较高，在响应头中设置 |

Cache-Control请求字段被各个浏览器支持得较好，而且它的优先级较高，它和其他一些请求字段（eg; Expires）同时出现时，Cache-Control 会覆盖其他字段。

Pragma字段的作用和 Cache-Control有点类似，它也是在HTTP头中包含一个特殊的指令，使相关的服务器的遵守该指令，最常用的就是Pragma:no-cache，它和Cache-Control:no-cache的作用是一样的。

2. **Expires**

Expires通常的使用格式是Expires:Sat,25 Feb 2012 12:22:17 GMT  ，后面跟着一个日期和时间，超过这个时间值后，缓存的内容将会失效，也就是浏览器在发出请求之前检查这个页面的这个字段，看该页面是否已经过期，过期了就重新向服务器发起请求。

3. **Last-Modified/Etag**

Last-Modified/Etag 字段一般用于表示一个服务器上的资源的最后修改时间，资源可以是静态（静态内容加上Last-Modified 字段）或者动态的内容（如Servlet提供了一个getLast-Modified方法用于检查某个动态内容是否已经更新），通过这个最后修改时间可以判断当前请求的资源是否是最新的。

一般服务端在响应头中返回一个Last-Modified字段，告诉浏览器这个页面的最后修改时间，如Last-Modified:Sat,25 Feb 2012 12:55:04 GMT ，浏览器再次请求时在请求头中增加一个If-modified-Since:Sat,25 Feb 2012 12:55:04 GMT 字段，询问当前缓存的页面是否是最新的，如果是最新的就返回**304**状态码，告诉浏览器是最新的，服务器也不会传输新的数据。

与Last-Modified字段类似的功能还有一个**Etag**字段，这个字段的作用是让服务端给每个页面分配一个唯一的编号，然后通过这个编号来区分当前这个页面是否是最新的。这种方式比使用Last-Modified更加灵活，但是在后端的web服务器有多台时，比较难处理，因为每个web服务器都要记住网站的所有资源，否则浏览器返回的这个编号就没有意义了。

##DNS域名解析