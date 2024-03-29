# 基本知识

## 无状态

最初为了 HTTP 能快速处理大量事务，确保协议的可伸缩性，特意把 HTTP 设计成无状态的，这意味着 HTTP 协议不会对任何请求或相应报文做持久化处理。

## 使用 URI 定位资源

1. 客户端请求特定资源时需要把 URI 包含在请求报文中
2. 客户端如果是对服务器本身发起请求，可以用 * 代替 URI，比如：

```http
OPTIONS * HTTP/1.1
```

## HTTP 方法

以下介绍的都是 HTTP/1.1 中可用的方法。

### 1.GET

请求已被 URI 标识的资源，指定的资源经服务器解析后返回相应内容。

### 2.POST

传输请求报文实体的主体。

### 3.PUT

传输文件，要求在请求报文主体中包含文件内容。

鉴于 HTTP/1.1 的 PUT 方法不带验证机制，所以出于安全原因一般网站不使用这个方法，除非配置了验证机制或者网站采用的是 REST 架构。

### 4.HEAD

获得响应报文首部，而不返回响应报文主体。用于确认 URI 的有效性及资源更新的日期等。

### 5.DELETE

删除文件，和 PUT 方法对应。

### 6.OPTIONS

查询针对请求 URI 指定的资源支持的方法。

### 7.TRACE

发送请求时会在`Max-Forwards`首部字段中填入数值，每经过一个服务器端就会减1，当数值为0时停止传输，最后收到请求的服务器端会返回 200 OK 的状态码。

客户端可以通过 TRACE 方法查询发送出去的请求是怎样被加工/篡改的，因为在传输的过程中可能会通过代理中转。

但是 TRACE 方法容易引发 XST（Cross-Site Tracing） 攻击，通常不会使用。

![TRACE](img\TRACE.jpg)

### 8.CONNECT

要求在与代理服务器通信时建立隧道，用隧道协议进行 TCP 通信。主要使用 SSL（Secure Sockets Layer） 和 TLS（Transport Layer Security）协议把通信内容加密后经网络隧道传输。

## 持久连接

在早期的 HTTP 协议中，每进行一次 HTTP 通信就要断开一次 TCP 连接。当数据量增加时，每次的请求都会造成无谓的 TCP 连接建立和断开，增加开销。

### 1. keep-alive

持久连接的特点是，只要任意一段没有明确提出要断开连接，那么就保持 TCP 连接。HTTP/1.1 中所有的连接默认都是持久连接。

![持久连接](img\持久连接.jpg)

### 2.管线化

持久连接使得管线化发送成为可能，无需等待响应就可发送下一个请求。

## 使用 Cookie 进行状态管理

1. 响应报文的`Set-Cookie`首部字段通知客户端保存 Cookie
2. 请求报文中的`Cookie`首部字段将 Cookie 值发送给服务器

# 报文

报文整体上可划分为报文首部和报文主体两部分，很多情况下并没有报文主体。

![报文结构](img\报文结构.jpg)

- 请求行：包含请求方法、请求 URI 和 HTTP 版本
- 状态行：包含状态码、原因短语和 HTTP 版本
- 首部字段：表示请求和响应的各种条件和属性，分为通用首部、请求首部、响应首部和实体首部
- 其他：可能包含 RFC 里未定义的首部

## 报文主体 & 实体主体

- 报文：HTTP 通信中的基本单位
- 实体：作为请求或响应的有效载荷数据被传输，其内容由实体首部和实体主体组成

==HTTP 报文主体用于传输请求或响应的实体主体==

## 报文首部

HTTP 报文首部字段传递了报文主体大小、语言、认证信息等最重要的信息，因此首部字段是报文中最重要的部分。

> 当 HTTP 报文首部出现了多个相同字段名的字段时，因为 RFC 并没有明确规范，所以不同浏览器的处理逻辑会不同。有的浏览器会优先处理第一次出现的首部字段，而另一些则会优先处理最后出现的首部字段。

### 通用首部字段

请求和响应报文都会使用的首部。

#### 1. Cache-Control

用来操作缓存的工作机制。可以有多个指令，多个指令之间用“,“分割。

```http
Cache-Control: private, max-age=0, no-cache
```

缓存请求指令：

![缓存请求指令1](img\缓存请求指令1.jpg)

![缓存请求指令2](img\缓存请求指令2.jpg)

缓存响应指令：

![缓存响应指令](img\缓存响应指令.jpg)

#### 2. Connection

（1）指定不再转发给代理的首部字段

比如`Connection: Upgrade`指示代理服务器删除报文中的`Upgrade`首部字段之后才能进行转发。

（2）管理持久连接

HTTP/1.1 默认都是持久连接，如果要关闭，则这个字段的值为`close`。HTTP/1.1 之前的版本如果想要使用持久连接，则设置这个字段的值为`Keep-Alive`。

#### 3. Date

这个字段表明报文的创建日期和时间。HTTP/1.1 使用在 RFC1123 中规定的格式：

```http
Date: Tue, 03 Jul 2012 04:39:30 GMT
```

#### 4. Pragma

这是一个 HTTP/1.1 之前版本的遗留字段，其形式唯一，而且只用在请求报文中，用于要求所有的中间服务器不要返回缓存的资源。

```http
Pragma: no-cache
```

在不能保证所有中间服务器都使用了 HTTP/1.1 的情况下，请求报文中会同时存在`Pragma`和`Cache-Control`两个首部字段。

#### 5. Trailer

比如`Trailer: Expires`表明在报文主体之后还有一个`Expires`首部字段。常用在 HTTP/1.1 分块传输编码时。

### 请求首部字段

#### 1. Accept

用来通知服务器用户代理能够处理的煤体类型以及这些类型的相对优先级。

![媒体类型](img\媒体类型.jpg)

如果想要指定优先级，则在媒体类型值之后加上分号，然后使用`q=`来额外表示权重。权重值的范围是0~1（可精确到小数点后3位），1 为最大值。默认权重为1.0。当服务器提供多种内容时，优先返回权重高的媒体类型。

```http
Accept: text/plain; q=0.3, text/html; q=0.8
```

#### 2. Accept-Charset

用来通知服务器用户代理支持的字符集以及相对优先顺序。

```http
Accept-Charset: iso8859-1, unicode-1-1; q=0.8
```

#### 3. Accept-Encoding

用来通知服务器用户代理支持的内容编码以及内容编码的优先级。

#### 4.Accept-Language

用来通知服务器用户代理支持的自然语言集和相对优先级。

#### 5. Authorization

将用户代理的认证信息通知给服务器。

#### 6. Host

当多个虚拟主机使用同一 IP 时，需要使用这个首部字段加以区分。

#### 7. If-Match

形如 If-xxx 的首部字段统称为条件请求。只有判断条件为真时服务器才会执行请求。

`If-Match`会告知服务器匹配资源所用的实体标记值（ETag）。如果使用*号指定这个字段的值，那么只要资源存在服务器就会进行响应。

![If-Match](img\If-Match.jpg)

#### 8. If-Modified-Since

用于确认代理或客户端拥有的资源的有效性。

![If-Modified-Since](img\If-Modified-Since.jpg)

#### 9. If-Range

和`Range`字段配合使用。当这个字段的值（ETag 或者时间）和资源的相应值一致时，服务器会将这个请求视为范围请求进行处理；否则返回全体资源。

如果不使用这个字段，当客户端范围请求的资源失效时，服务器就会返回状态码412，让客户端再次发送请求。这种情况下就要进行两次通信。

![不使用 If-Range](img\不使用 If-Range.jpg)

#### 10. Referer

告知服务器请求的原始资源的 URI。

![Referer](img\Referer.jpg)

#### 11. User-Agent

这个字段将会告知服务器创建请求的浏览器和用户代理名称等信息。

### 响应首部字段

#### 1.Accept-Ranges

告知客户端能否处理范围请求，`Accept-Ranges: none`表明不能处理，`Accept-Ranges: bytes`表明能够处理。

#### 2. Age

告知客户端源服务器在多久前创建了响应，单位是秒。若创建这个响应的服务器是缓存服务器，那么`Age`值指的是缓存后的响应再次发起认证到认证完成的时间值。

#### 3. Location

这个字段指出重定向后应该请求的 URI。

### 实体首部字段

#### 1. Allow

指出服务器能够支持哪些 HTTP 方法。如果请求报文使用了服务器不支持的方法，那么服务器就会返回`405 Method Not Allowed`，并且将所有能够支持的 HTTP 方法写入`Allow`字段。

#### 2. Content-Encoding

指出服务器对实体的主体部分使用的编码方式。

#### 3.Content-Language

将实体主体使用的自然语言告知客户端。

#### 4. Content-Length

指出实体主体部分的大小，单位是字节。

#### 5. Content-Type

指出实体主体内对象的媒体类型。

#### 6. Expires

将资源失效的日期时间告知客户端。

如果源服务器不希望缓存服务器进行缓存，最好在`Expires`字段内写入与`Date`字段相同的值。

当`Cache-Control`指定了`max-age`时，会优先处理`max-age`指令。

#### 7. Last-Modified

指明资源最终被修改的时间。

### 为 Cookie 服务的首部字段

虽然 Cookie 没有编入 HTTP/1.1 的 RFC2616 中，但在 Web 中得到了广泛的应用。

#### 1. Set-Cookie

这是一个响应首部字段，用于开始状态管理所使用的 Cookie 信息。

![Set-Cookie 字段的属性1](img\Set-Cookie 字段的属性1.jpg)

![Set-Cookie 字段的属性2](img\Set-Cookie 字段的属性2.jpg)

#### 2. Cookie

这是一个请求首部字段，客户端用这个字段发送保存着的 Cookie。

## 压缩传输的内容编码

HTTP 为了压缩数据，使用了内容编码，内容编码指明应用在实体内容上的编码格式，并保持实体信息鸳鸯压缩。内容编码后的实体由客户端接收并负责解码。

## 分块传输编码

请求的实体经过编码之后如果尚未全部传输完成，则浏览器无法显示请求到的资源。在传输大容量数据时，通过分割数据让浏览器逐步显示也页面。

分块传输编码（Chunked Transfer Coding）会将实体主体分成多个块，每一个用十六进制数来标记块的大小，最后一块会用“0（CR + LF）”标记。

## MIME

MIME（Multipurpose Internet Mail Extensions）机制使电子邮件可以发送多种数据类型的附件，而 MIME 扩展中使用多部分对象集合（Multipart）来容纳多份不同类型的数据。

相应地 HTTP 协议也采纳了多部分对象集合，一份报文主体内可以包含多种类型的实体。通常用于传输图片或文本文件等。

### 1.multipart/form-data

用于表单。

![表单](img\表单.jpg)

### 2.multipart/byteranges

用于状态码为206且包含多个范围的内容的响应报文。

![byeranges1](img\byeranges1.jpg)

![byteranges2](img\byteranges2.jpg)

Tip:

- 多部分对象集合可以嵌套
- 多部分对象集合的每个部分类型中都可以含有首部字段
- 使用`boundary`字符字符串来划分多部分对象集合指明的各类实体，实体起始行之前的`boundary`字符串之前要加上“--”，多部分对象集合的最后一个字符串的之后的`boundary`字符串之后要加上"--"

## 获取部分内容的范围请求

通过指定请求的实体范围可以实现断点续传，这种指定范围发送的请求叫做范围请求（Range Request）。范围请求的报文首部要用到`Range`字段。

示例：

（1）请求5001—10000字节

```http
Range: bytes=5001-10000
```

（2）请求 5001 字节之后所有内容

```http
Range: bytes=5001-
```

（3）请求开头到3000字节和5000—7000字节的内容

```http
Range: bytes=-3000, 5000-7000
```

针对范围请求，响应会返回状态码为206的响应报文。对于多重范围请求，如示例（3），响应报文会设置首部字段`Content-Type=multipart/byteranges`。

如果服务器无法响应范围请求，那么就返回200 OK 和完整的实体内容。

## 内容协商

内容协商（Content Negotiation）就是让服务器返回最合适的内容，最常见的场景就是国际化。内容协商会以响应资源的语言、字符集、编码方式等作为判断的标准。请求报文中通常会使用到以下首部字段：`Accept`,`Accept-Charset`,`Accept-Encoding`,`Accept-Lanuage`,`Content-Language`。

# 状态码

状态码由3位数字和原因短语组成，第1位数字指定响应类别，后两位无分类。

![状态码类别](img\状态码类别.jpg)

RFC 中定义的状态码大概有60多种，最常见的大概就14种。

## 2XX

### 1. 200 OK

表示请求已被服务器正常处理。

### 2. 204 No Content

请求已被成功处理但是返回的响应报文中不含实体的主体部分，同时也不允许返回任何实体的主体。

一般在服务器不需要向客户端发送新的数据时使用。这种情况下浏览器页面不会刷新。

### 3. 206 Partial Content

客户端进行了范围请求，服务器成功执行了这部分的 GET 请求。

## 3XX

### 1. 301 Moved Permanently

表示请求的资源已经被分配了新的 URI，以后应该使用新的 URI。这叫做永久重定向。

如果浏览器把资源对应的 URI 保存为书签，那么应该按照 `Location`首部字段提示的 URI 重新保存。

### 2. 302 Found

和 301 类似，但是这种重定向是临时的，只是在本次请求中使用新的 URI。

### 3. 303 See Other

和302类似，也是一种临时重定向，但是303要求客户端使用 GET 方法获取资源，而无视原先请求的方法。

注意：当出现301、302、303状态码时，几乎所有的浏览器都会将 POST 方法改为 GET 并删除请求报文内的主体。（尽管301、302标准禁止浏览器更改请求方法）

### 4. 304 Not Modified

注意这个状态码和重定向无关，它表示客户端发送带条件的请求时（比如`If-Match`,`If-Modified-Since`）服务器虽然允许访问资源，但未满足条件。

### 5.307 Temporary Redirect

这个状态码与302的区别在于它遵照浏览器标准不会将 POST 改为 GET。但是，对于处理响应时的行为，每种浏览器有可能出现不同的情况。

## 4XX

### 1. 400 Bad Request

请求报文中存在语法错误。浏览器会像对待 200 一样对待 400，但是需要修改请求的内容之后再次发送请求。

### 2. 401 Unauthorized

表示发送的请求需要有通过 HTTP 认证（BASIC 认证、DIGEST 认证）的认证信息。如果之前已经经过1次请求，则表示用户认证失败。

这种响应必须包含一个适用于被请求资源的`WWW-Authenticate`首部用于质询用户信息。

### 3. 403 Forbidden

服务器拒绝访问某资源。这种情况下服务器没有必要给出拒绝的详细理由。

### 4. 404 Not Found

服务器找不到资源。也可以在服务器拒绝请求且不想说明理由时使用。

### 5.405 Method Not Allowed

服务器不允许使用该 HTTP 方法。

## 5XX

### 1. 500 Internal Server Error

 服务器在执行请求时发生错误。

### 2. 503 Service Unavailable

服务器暂时处于超负荷或者正在停机维护。如果事先得知这种状态的解除时间，最好写入响应报文的`Retry-After`首部字段。

# 服务器

## 单台虚拟主机实现多个域名

HTTP/1.1 允许通过虚拟主机技术在同一计算机上搭建多个网站。

一个计算机的 IP 是唯一的，所以当一台计算机上搭载了多个网站时，就要在 HTTP 请求的`Host`首部字段中完整指定主机名或者域名的 URI。

## 通信数据转发流程

### 1.代理

代理是一种具有转发功能的**应用程序**，接收请求并转发给服务器，同时也接收响应并转发给客户端。

可以级联多台代理服务器，转发时需要附加`Via`首部字段以标记出经过的主机信息。

![代理](img\代理.jpg)

（1）使用场景

- 利用缓存减少网络带宽的流量
- 组织内部针对特定网站的访问控制
- 获取访问日志

（2）分类

- 缓存代理：转发响应时会对资源进行缓存
- 透明代理：转发时不会对报文做任何处理

### 2.网关

转发其他服务器通信数据的**服务器**。网关能使通信线路上的服务器提供非 HTTP 协议的服务。

利用网关可以提高安全性，因为可以在客户端与网关之间的通信线路上加密。

代理服务器也是网关的一种。

### 3.隧道

隧道是在客户端和服务器两者之间进行中转并保持双方通信连接的**应用程序**。为了保证安全，隧道建立一条与其他服务器通信的安全线路，使用 SSL 等手段进行加密。

隧道本身不会去解析 HTTP 请求，这意味着对客户端而言隧道是透明的。隧道会在通信双方断开连接时结束。













