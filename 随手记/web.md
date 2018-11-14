## 一.<form> 的 enctype 属性的区别

1. multipart/form-data

将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当上传的字段是文件时，会有 Content-Type 来表名文件类型；content-disposition，用来说明字段的一些信息；由于有 boundary 隔离，所以 multipart/form-dat a既可以上传文件，也可以上传键值对，它采用了键值对的方式，所以可以上传多个文件。

2. x-www-form-urlencoded： 

就是 application/x-www-from-urlencoded,会将表单内的数据转换为键值对，比如,name=java&age = 23

## 二、RESTful

### 1.REST，即 Representational State Transfer 的缩写，可以理解为"表现层状态转化"。如果一个架构符合 REST 原则，就称它为 RESTful 架构。

- 表现层（Representation）：

  "资源"是一种信息实体，它可以有多种外在表现形式。把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。在 REST 原则中，一个 URI 就指向一个资源，而且也__仅仅是指向资源__。

- 状态转化 （State Transfer ）：互联网通信协议 HTTP 协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。客户端的手段就是 HTTP 协议中的四个动词，**GET 用来获取资源，POST 用来新建资源（也可以用于更新资源），PUT 用来更新资源，DELETE 用来删除资源。** 

总而言之，RESTful 架构的三个要点：

- 每一个 URI 代表一种资源
- 客户端和服务器之间，传递这种资源的某种表现层
- 客户端通过四个 HTTP 动词，对服务器端资源进行操作，实现"表现层状态转化"

### 2. RESTful 架构设计的常见误区

- URL 包含动词

前面已经说过，URI 的唯一功能就是指向一个资源，客户端想要对资源所进行的操作用 HTTP 动词来表达。

错误的 URI ：

```HTTP
POST /accounts/1/transfer/500/to/2
```

改成 RESTful URI：

```http
　　POST /transaction HTTP/1.1
　　Host: 127.0.0.1
　　
　　from=1&to=2&amount=500.00
```

- 在 URI 中包含版本号

不管是什么版本，本质上都是同一种资源，版本只是不同的表现形式。

错误的 URI：

```http
　　http://www.example.com/app/1.0/foo
```

改成 RESTful URI，去掉版本号，把版本号写在请求头中：

```http
　　Accept: vnd.example-com.foo+json; version=1.0
```

### 3.**Richardson Maturity Model** 

![Richardson Maturity Model](img\Richardson Maturity Model.jpg)

- level 0：POX 指的 Plain Old XML，也就是通过网络传输的是 XML 数据。这种架构叫做 Simple Object Access Protocol，简单对象访问协议，关注的是行为和处理。
- level 1：开始面向资源，但仍然只是把 HTTP 当做一个传输的通道，没有把 HTTP 当做一种传输协议。 
- level 2：真正将 HTTP 作为了一种传输协议，最直观的一点就是使用了 HTTP 动词。__客户端通过动词表达想要进行的操作，服务器端的响应报文中的动词也能让客户端明白能进行什么操作__。
- level 3：这一层有一个相关的名词叫 HATEOAS（Hypertext As The Engine Of Application State），中文翻译为“将超媒体格式作为应用状态的引擎”，核心思想就是每个资源都有它的状态，不同状态下，可对它进行的操作不一样。满足这一层的 Restful API 给使用者带来了很大的便利，__使用者只需要知道如何获取资源的入口，之后的每个 URI 都可以通过请求获得，无法获得就说明无法执行那个请求。__

现在绝大多数的 RESTful API 都做到了 Level 2 的层次，做到 Level3 的比较少。当然，这个模型并不是一种规范，只是用来理解Restful的工具。同样地，RESTful 本身也不是一种规范，只是一种架构风格。

### 4.数据格式

Restful 对数据格式没有限制，可以用 XML，JSON 或者其他格式，只要符合上面提到的几个特征，就算 Restful。

### 5.RESTful API 设计指南

[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

# 三、MyBatis



1. java.lang.IllegalArgumentException: Mapped Statements collection does not contain value for

这个异常产生的原因是映射文件中没有某条指定的 SQL 语句，比如说在调用 session.update("test.updateUser", user); 时抛出这个异常，就去检查映射文件中是否真的配置了 test.updateUser 这条 SQL 语句。

1. invalid bound statement (not found)

一般是因为映射文件和接口的类名不一致，或者二者不在同一目录下。

# 四、SSM 整合

1. 异常：Name [spring.profiles.active] is not bound in this Context. Unable to find [spring.profiles.active].

解决方案是在 web.xml 中添加以下配置：

```xml
<context-param>  

    <param-name>spring.profiles.active</param-name>  

    <param-value>dev</param-value>  

</context-param>  

<context-param>  

    <param-name>spring.profiles.default</param-name>  

    <param-value>dev</param-value>  

</context-param>

<context-param>  

    <param-name>spring.liveBeansView.mbeanDomain</param-name>  

    <param-value>dev</param-value>  

</context-param>  

```

2. 异常：javax.el.PropertyNotFoundException: Property [xxx] not found on type com.example.Bean

前端的锅！！！！！！前端页面引用了一个并不存在的属性！！！！

3. 异常： `Nested in org.springframework.web.util.NestedServletException: Handler processing failed; nested exception is java.lang.StackOverflowError`

堆栈溢出可首先检出是否给 JVM 分配的内存太小（一般都不会），然后检查代码中是否出现了死递归、死循环。

4. Spring 注解`@Bean`和`@Component`的区别
   - `@Component`用在类上，`@Bean`一般用在属性或方法上。

   - Spring 扫描 classpath 中的类，发现`@Compenent`时就把这个类注入到容器中；而`@Bean`注解的对象则灵活得多，比如说如果`@Bean`加在方法上，那么方法退出时才把返回的对象加入容器中。

   - 有时候只能用`@Bean`，比如说引用了一个第三方库，但是没有这个库的源码，这个时候就不能用`@Component`进行自动装配。

# 五、 OPTIONS 请求

进行跨域请求时，如果请求方法不是`GET`、`POST`和`HEAD`中的任何一种，那么客户端就要首先发送一个方法为`OPTIONS`的请求。这种请求叫做 preflight request，目的是确保客户端将要发送的请求会被服务器信任，客户端请求中的 header，method,origin 对服务器来说都是安全的。这种情况下服务器首要要响应`OPTIONS`	请求，判断客户端是否值得信任，在响应报文中根据情况设置`Access-Control-Allow-Origin`。

# 六、 cookie 命名规范

cookie 名称中不能含有`=,; \t\r\n\013\014`这些字符中的任何一个。\013是回车，\014是移位输出。

# 七、Spring @ResponseBody 注解报错No converter found for return value of type

返回值类型没有 getter 和 setter

