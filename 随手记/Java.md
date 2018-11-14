## 一、List 转成数组的方法：String[] arr = list.toArray(new String[0]);

 ## 二、新版本的 Cookie（参见RFC 2109）目前还不被 Javax.servlet.http.Cookie 包所支持。在 Cookie Version 0 中，某些特殊的字符，例如：空格，方括号，圆括号，等于号（=），逗号，双引号，斜杠，问号，@符号，冒号，分号都不能作为 Cookie 的内容。

 

## 四、Junit

1. 在需要测试的方法之前加上 @Test，如果想同时测试多个方法，就在测试类上右键 Run as -> Junit。
2. @Before ：在测试方法运行之前执行，比如初始化资源。
3. @After：在测试方法之后执行，比如释放资源。
4. @BeforeClass：在类加载的时候执行。
5. @AfterClass：在类销毁时执行。
6. 使用断言

- Assert.assertEquals(expected,actual)：如果实际值会期望值相等，则测试通过。
- Assert.NotNull(object)：如果对象非 null，则测试通过。
- Assert.assertTrue(value)：如果值为 true，则测试通过。

## 五、判断对象是否为空

用 == 不用 equals！太谜了！！！！

## 六、Spring 常用注解

1. `@Component`：用在类上，比如说自己实现的拦截器类。如果类的属性要从配置文件中读取值，那么这个类也要加上这个注解。
2. `@Bean`：一般用在方法上，方法返回的对象会被注册为 bean
3. `@Autowired`：用在类的属性上
4. `@Service`：service 接口的实现类
5. `@Entity`：使用 Spring Data JPA 的时候这个注解用在实体类上
6. `@Configuration`：用在自定义配置类上
7. `@Value`：常用于从配置文件中读取值注入属性

# 七、IDEA 中的 classpath

IDEA 的 classpath 实际上是 target/classes：

![IDEA classpath](img\IDEA classpath.png)

所以如果要通过 classpath: 这种相对路径来引用工程中的一个文件的话，就要注意对应：

- src/main/java/com/example/springbootjwt/controller.java -> classpath:com/example/springbootjwt/controller.java
- src/main/resources/config/config.properties -> classpath:config/config.properties

![IDEA project structure](img\IDEA project structure.png)

# 八、Spring Boot 工程中读取配置文件属性

application.properties 文件会被 Spring Boot 项目默认加载，但是其他自定义的配置文件需要手动引用：

- 首先在需要引用配置文件属性值的类上加上注解`@PropertySource`，值是一个字符串数组，用来指定配置文件路径，通常使用的是 classpath 相对路径
- 然后在需要注入值的属性上加上注解`@Value`，值是字符串，比如`${book.name}`就代表引用配置文件中的 book.name 这个属性的值

# 九、IDEA Error:(1, 1) java: 非法字符: '\ufeff'

用 Windows 记事本打开并修改 .java 文件保存后重新编译运行项目出现“Error:(1, 1) java: 非法字符: '\ufeff'”错误，如下图所示：



![img](https:////upload-images.jianshu.io/upload_images/1925324-ae0a7195f636c1a7.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/277/format/webp)



原来这是因为 Windows 记事本在修改 UTF-8 文件时自作聪明地在文件开头添加BOM导致的，所以才会导致 IDEA 不能正确读取 .java 文件从而程序出错。

在编辑器 IDEA 中将文件编码更改为 UTF-16，再改回 UTF-8 即可，其实就相当于刷新了一下文件编码。

