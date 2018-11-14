# 嵌入式 servlet 容器

Spring Boot 默认使用嵌入式的 Tomcat。

## 一、修改嵌入式 Tomcat 的配置

1. 在 application 文件中进行配置：

```properties
server.port=8081
server.tomcat.uri-encoding=UTF-8
```

`server.xxx`就是通用的 servlet 容器配置，`servlet.tomcat`则是 Tomcat 的配置。

2. 在自定义配置类中实现一个嵌入式 servlet 容器的定制器：

   ```java
       @Bean
       public WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> configurableServletWebServerFactoryWebServerFactoryCustomizer(){
           return server -> {
               server.setPort(8081);
           };
       }
   ```

   上面这种方法进行的一般是 servlet 容器的通用配置，也可以针对某种具体的容器进行配置：

   ```java
   @Bean
   public ConfigurableServletWebServerFactory webServerFactory() {
   	TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
   	factory.setPort(9000);
   	factory.setSessionTimeout(10, TimeUnit.MINUTES);
   	factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
   	return factory;
   }二、servlet 容器自动配置原理
   ```


## 二、嵌入式 servlet 容器自动配置原理

1.Spring Boot 根据导入的依赖，向容器中添加相应的`ConfigurableServletWebServerFactory`，比如 Tomcat 对应的就是`TomcatServletWebServerFactory` 。

![继承关系](G:\notebook\Java\Spring Boot\img\继承关系.png)

2.当容器中的某个组件要实例化时，Spring Boot 会启动后置处理器，作用是在 bean 创建完成之后进行初始化赋值。servlet 容器的后置处理器`ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar`会执行相应的初始化工作。

3.后置处理器从容器中获取所有的定制器`WebServerFactoryCustomizer`，调用定制器的定制方法进行自定义配置。

## 三、嵌入式 servlet 容器启动原理

## 四、使用外部 servlet 容器

嵌入式容器的主要特点：简便易用，将应用打成`jar`包。缺点是定制复杂，也不支持 JSP。

使用的外部 servlet 容器的基本步骤：

1. 形成传统的 Web 应用的目录结构

![传统 web 应用目录1](G:\notebook\Java\Spring Boot\img\传统 web 应用目录1.png)

![传统 web 应用目录2](G:\notebook\Java\Spring Boot\img\传统 web 应用目录2.png)

2. 配置 Tomcat

配置 Tomcat 时注意选择把应用打成`war`包。在 POM 中将嵌入式 Tomcat 指定为 provided：

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
 </dependency>
```

3. 必须编写一个`SpringBootServletInitializer`的子类，调用`configure`方法：

```java
public class ServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        // 传入 Spring Boot 应用的主程序
        return builder.sources(SpringBootWebCrudApplication.class);
    }
}
```

## 五、外部 servlet 容器启动 Spring Boot 应用的原理

# 整合 Shiro

与 Spring Security 一样都是一个权限的安全框架，但是与Spring Security 相比，在于 Shiro 使用了比较简单易懂易于使用的授权方式。 

![Shiro 核心组件](G:\notebook\Java\Spring Boot\img\Shiro 核心组件.jpg)

- Subject： 当前用户操作 
- SecurityManager： 用于管理所有的Subject 
- Realms： 用于进行权限信息的验证

## 一、核心原理

要继承 Shiro，就要先了解大体的一些管理对象：

1. ShiroFilterFactory

   Shiro过滤器工厂类，具体的实现类是：ShiroFilterFactoryBean，此实现类依赖于 SecurityManager 安全管理器。

2. SecurityManager

   Shiro 的安全管理，主要是身份认证的管理，缓存管理， cookie 管理，所以在实际开发中我们主要是和 ==SecurityManager== 进行打交道的，而 ShiroFilterFactory 主要是配置好 Filter 。当然 SecurityManager 也进行身份认证缓存的实现，我们需要进行对应的编码然后进行注入到安全管理器中。

3. Realm

   用于身份信息权限信息的验证。

4. 其它的就是缓存管理，记住登录之类的，这些大部分都是需要自己进行简单的实现，然后注入到 SecurityManager 让 Shiro 的安全管理器进行管理就好了。

POM 依赖：

```xml
       <dependency>
           <groupId>org.apache.shiro</groupId>
           <artifactId>shiro-spring</artifactId>
           <version>1.3.2</version>
       </dependency>
```

基本配置：

Shiro 核心通过 Filter 来实现，也就是说通过 URL 规则来进行过滤和权限校验，所以我们需要定义一系列关于 URL 的规则和访问权限。

```java
@Configuration
public class ShiroConfig {

    // 注意要先配置 SecurityManager
    @Bean
    public SecurityManager securityManager(){
        DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
        return securityManager;
    }

    // 配置 URL 拦截规则
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean  = new ShiroFilterFactoryBean();

        // 必须设置 SecurityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);

        // 配置拦截器

        
                filterMap.put("/asserts/css/signin.css", "anon"); // anon：所有的 URL 可以匿名访问
        filterMap.put("/login.html", "anon"); // 访问登录页面不拦截
        filterMap.put("/", "anon");
        filterMap.put("/signin.html", "anon"); // 注册页面也不用拦截
        // 要将 /** 放在最下边！！！
        filterMap.put("/**", "authc"); // authc：访问所有的 URL 都要经过验证


        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
        return shiroFilterFactoryBean;
    }
}
```

说明：在上面的代码中有一句`filterMap.put("/**", "authc");`，这里使用了 Shiro 已经实现好的过滤器。常用的过滤器有

- anon:所有 URL 都都可以匿名访问;访问CSS 等静态资源、登录页面、注册页面都不用拦截，这里比较容易出错
- authc: 需要认证才能进行访问;
- user:配置记住我或认证通过可以访问；

## 二、用户授权

### 1.拦截未登录

要拦截没有登录的用户，首先在配置类的 shiroFilter 方法中添加以下代码：

```java
       shiroFilterFactoryBean.setLoginUrl("/login"); // 登录
                                                      // 默认是寻找根目录下 login.jsp
        shiroFilterFactoryBean.setSuccessUrl("/select.html"); // 登录成功后
        shiroFilterFactoryBean.setUnauthorizedUrl("/index.html"); // 未授权
```

如果配置成功，那么未登录的时候访问其他页面就会跳转到登录页面。

注意：上面的设置中 URL 的值必须和 Spring MVC 的相关配置一一对应。比如没有登录时跳转的 URL 是`/login.html`，那么 Spring MVC 中就要有对这个 URL 对应的视图的配置。

### 2.身份认证

在 Shiro 中，最终是通过 Realm 来获取应用程序中的用户、角色及权限信息。通常情况下 Realm 中会直接从我们的数据源中获取 Shiro 需要的验证信息。可以说，Realm 是专用于安全框架的 DAO。

要做的就是自定义一个 Realm 类，继承`AuthorizingRealm`抽象类，重载`doGetAuthenticationInfo ()`，重写获取用户信息的方法。 

（1）实体类

在一个完善的权限管理系统中，至少有下面几张表：

- 用户表：在用户表中保存了用户的基本信息，账号、密码、姓名，性别等；
- 权限表（资源+控制权限）：这个表中主要是保存了用户的 URL 地址，权限信息，简单地说就是用户可以访问哪些资源，进行哪些操作等；
- 角色表：保存了系统存在的角色；
- 关联表：用户-角色管理表（用户在系统中都有什么角色，比如 admin，vip 等），角色-权限关联表（每个角色都有什么权限可以进行操作）。

据此分析，实体类有三个。

用户信息类：

```java
/**
 * 用户信息表
 */
@Entity
public class UserInfo {
    @Id
    @GeneratedValue
    private long uid;       //用户id;

    @Column(unique=true)
    private String username;   // 账号，本案例中就是学号

    private String name; // 用户姓名

    private String password; //密码;
    private String salt;//加密密码的盐

    private byte state;//用户状态,0:创建未认证（比如没有激活，没有输入验证码等等）--等待验证的用户 , 1:正常状态,2：用户被锁定.

    // 用户信息表和用户角色表是多对多关系
    @ManyToMany(fetch= FetchType.EAGER) //立即从数据库中进行加载数据;
    @JoinTable(name = "SysUserRole", joinColumns = { @JoinColumn(name = "uid") }, inverseJoinColumns ={@JoinColumn(name = "roleId") })
    private List<SysRole> roleList;// 一个用户具有多个角色

    public List<SysRole> getRoleList() {
        return roleList;
    }

    public void setRoleList(List<SysRole> roleList) {
        this.roleList = roleList;
    }

    public long getUid() {
        return uid;
    }

    public void setUid(long uid) {
        this.uid = uid;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getSalt() {
        return salt;
    }

    public void setSalt(String salt) {
        this.salt = salt;
    }

    public byte getState() {
        return state;
    }

    public void setState(byte state) {
        this.state = state;
    }

    /**
     * 密码盐.
     * @return
     */
    public String getCredentialsSalt(){
        return this.username+this.salt;
    }
}
```

用户角色类：

```java
/**
 * 用户角色表
 */
@Entity
public class SysRole {
    @Id
    @GeneratedValue
    private Long id; // 编号

    private String role; // 角色标识程序中判断使用,如"admin",这个是唯一的:

    private String description; // 角色描述,UI界面显示使用

    private Boolean available = Boolean.FALSE; // 是否可用,如果不可用将不会添加给用户

    // 用户角色表和用户权限表是多对多关系
    @ManyToMany(fetch= FetchType.EAGER)
    @JoinTable(name="SysRolePermission",joinColumns={@JoinColumn(name="roleId")},inverseJoinColumns={@JoinColumn(name="permissionId")})
    private List<SysPermission> permissions;

    // 用户 - 角色关系定义;
    @ManyToMany
    @JoinTable(name="SysUserRole",joinColumns={@JoinColumn(name="roleId")},inverseJoinColumns={@JoinColumn(name="uid")})
    private List<UserInfo> userInfos;// 一个角色对应多个用户

    public List<UserInfo> getUserInfos() {
        return userInfos;
    }
    public void setUserInfos(List<UserInfo> userInfos) {
        this.userInfos = userInfos;
    }
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
    public String getRole() {
        return role;
    }
    public void setRole(String role) {
        this.role = role;
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    public Boolean getAvailable() {
        return available;
    }
    public void setAvailable(Boolean available) {
        this.available = available;
    }
    public List<SysPermission> getPermissions() {
        return permissions;
    }
    public void setPermissions(List<SysPermission> permissions) {
        this.permissions = permissions;
    }
}
```

用户权限类：

```java
/**
 * 用户权限表
 */
@Entity
public class SysPermission {
    @Id
    @GeneratedValue
    private long id;//主键.

    private String name;//名称.

    @Column(columnDefinition="enum('menu','button')")
    private String resourceType;//资源类型，[menu|button]

    private String url;//资源路径.

    private String permission; //权限字符串,menu例子：role:*，button例子：role:create,role:update,role:delete,role:view

    private Long parentId; //父编号

    private String parentIds; //父编号列表

    private Boolean available = Boolean.FALSE;

    @ManyToMany
    @JoinTable(name="SysRolePermission",joinColumns={@JoinColumn(name="permissionId")},inverseJoinColumns={@JoinColumn(name="roleId")})
    private List<SysRole> roles;

    public long getId() {
        return id;
    }
    public void setId(long id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getResourceType() {
        return resourceType;
    }
    public void setResourceType(String resourceType) {
        this.resourceType = resourceType;
    }
    public String getUrl() {
        return url;
    }
    public void setUrl(String url) {
        this.url = url;
    }
    public String getPermission() {
        return permission;
    }
    public void setPermission(String permission) {
        this.permission = permission;
    }
    public Long getParentId() {
        return parentId;
    }
    public void setParentId(Long parentId) {
        this.parentId = parentId;
    }
    public String getParentIds() {
        return parentIds;
    }
    public void setParentIds(String parentIds) {
        this.parentIds = parentIds;
    }
    public Boolean getAvailable() {
        return available;
    }
    public void setAvailable(Boolean available) {
        this.available = available;
    }
    public List<SysRole> getRoles() {
        return roles;
    }
    public void setRoles(List<SysRole> roles) {
        this.roles = roles;
    }
}
```

3个实体类对应的是数据库的五张表：UserInfo、SysUserRole、SysRole、SysRolePermission、SysPermission

然后是常规的 MVC 模式：

```java
/**
 * 用户信息 DAO
 */
public interface UserInfoRepository extends CrudRepository<UserInfo,Long> {

    // 通过用户名查询用户信息
    UserInfo findByUsername(String username);
}
```

```java
/**
 * 用户信息服务接口
 */
public interface UserInfoService {

    public UserInfo findByUsername(String username);
}
```

```java
@Service
public class UserInfoServiceImpl implements UserInfoService {

    @Resource
    private UserInfoRepository userInfoRepository;

    @Override
    public UserInfo findByUsername(String username) {
        return userInfoRepository.findByUsername(username);
    }

}
```

前面进行的都是准备工作，接下来才是 Shiro 身份认证功能的核心。

实现一个 Realm，此Realm继承`AuthorizingRealm`：（有一个细节需要注意，登录表单提交的时候用户名的参数名称是 username，密码的参数名称是 password）

```java
/**
 * 身份验证核心配置
 */
public class MyShiroRealm extends AuthorizingRealm {

    @Resource
    private UserInfoService userInfoService;

    // 身份验证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String username = (String) authenticationToken.getPrincipal(); // 获取用户名
        // 这一步可以做缓存，如果不做，Shiro 在两分钟内也不会重复执行
        UserInfo userInfo = userInfoService.findByUsername(username); // 查询有无此用户
        if(userInfo == null){
            return null;
        }

        // 这里还可以进行权限信息的获取

        // 验证用户名和密码
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(userInfo,
                userInfo.getPassword(),
                ByteSource.Util.bytes(userInfo.getSalt()),
                getName());
        return simpleAuthenticationInfo;
    }

    // 授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }
}
```

 因为这里的数据库是加密保存用户密码，所以还要注入加密算法类，可以直接用 Shiro 的实现：

```java
    // 注册 Realm
    @Bean
    public MyShiroRealm myShiroRealm(){
        MyShiroRealm myShiroRealm = new MyShiroRealm();
        myShiroRealm.setCredentialsMatcher(hashedCredentialsMatcher()); // 注入凭证匹配器
        return myShiroRealm;
    }

    // 凭证匹配器
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher(){
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();

        hashedCredentialsMatcher.setHashAlgorithmName("md5"); //散列算法:这里使用MD5算法;
        hashedCredentialsMatcher.setHashIterations(2); //散列的次数，比如散列两次，相当于 md5(md5(""));

        return hashedCredentialsMatcher;
    }
```

最后就是写登录控制器：

```java
    // 注意这里只处理登录失败的情况，登录成功的情况已经由 Shiro 处理了
    @PostMapping(value = "/login") // 这个 URL 和配置 Shiro 的时候设置的 LoginUrl 相同
    public String login(HttpServletRequest request, Model model){
        String exception = (String) request.getAttribute("shiroLoginFailure");
        String msg = ""; // 提示信息

        if(exception != null){
            if (UnknownAccountException.class.getName().equals(exception)) {
                // 用户名不存在
                System.out.println("UnknownAccountException -- > 账号不存在：");
                msg = "用户名不存在";
            } else if (IncorrectCredentialsException.class.getName().equals(exception)) {
                // 密码错误
                System.out.println("IncorrectCredentialsException -- > 密码不正确：");
                msg = "密码错误";
            } else {
                // 其他错误
                msg = exception;
            } // 其实这里还可以添加验证码的验证
        }

        model.addAttribute("msg", msg);
        return "login"; // 重新回到登录页面
    }
```

### 3.身份授权

授权的意义就是，从数据库中获取该用户具有哪些权限，然后在 Shiro 中进行配置。

授权要用 AOP，所以要先开启注解，在 Shiro 配置类中：

```java
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager){
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
```

然后实现 Realm 的`doGetAuthorizationInfo`方法：

```java
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("身份授权");

        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        UserInfo userInfo = (UserInfo) principalCollection.getPrimaryPrincipal();

        // 从数据库中获取该用户的角色、权限
        for (SysRole role : userInfo.getRoleList()) {
            authorizationInfo.addRole(role.getRole()); // 配置角色
            for (SysPermission p : role.getPermissions()) {
                authorizationInfo.addStringPermission(p.getPermission()); // 配置权限
            }
        }
        return authorizationInfo;
    }
```

然后写控制器：

```java
@Controller
@RequestMapping("/userInfo")
public class UserInfoController {

    // 查询用户
    @GetMapping("/userList")
    @RequiresPermissions("userInfo:view")// 记得加注解
    public String userList(){
        return "userList";
    }
    
    // 添加用户、删除用户...
}
```

### 4.开启缓存

实际开发中最好还是用缓存，可以提高程序的性能。

POM 依赖：

```xml
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>
```

在 Shiro 配置类中注入缓存：

```java
    // 缓存管理器
    @Bean
    public EhCacheManager ehCacheManager(){
        System.out.println("ShiroConfiguration.getEhCacheManager()");
        EhCacheManager cacheManager = new EhCacheManager();
        cacheManager.setCacheManagerConfigFile("classpath:config/ehcache-shiro.xml");
        return cacheManager;
    }
```

```java
    @Bean
    public SecurityManager securityManager(){
        DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
        // 设置 Realm
        securityManager.setRealm(myShiroRealm());
        // 注入缓存管理器
        securityManager.setCacheManager(ehCacheManager());
        return securityManager;
    }
```

最后添加缓存配置文件，在`src/main/resouces/config`添加`ehcache-shiro.xml`： 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache name="es">
  
    <diskStore path="java.io.tmpdir"/>
    
    <!--
       name:缓存名称。
       maxElementsInMemory:缓存最大数目
       maxElementsOnDisk：硬盘最大缓存个数。 
       eternal:对象是否永久有效，一但设置了，timeout将不起作用。 
       overflowToDisk:是否保存到磁盘，当系统当机时
       timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
       timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false. 
       diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。 
       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
       memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。 
        clearOnFlush：内存数量最大时是否清除。
         memoryStoreEvictionPolicy:
            Ehcache的三种清空策略;
            FIFO，first in first out，这个是大家最熟的，先进先出。
            LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
            LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
    -->
     <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            />
            
            
    <!-- 登录记录缓存锁定10分钟 -->
    <cache name="passwordRetryCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="3600"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true">
    </cache>
    
</ehcache>
```

### 5.记住我

登录页面有一个“记住我”的复选框`<input type="checkbox" name="rememberMe"/>`。

首先在配置类中添加 Cookie 对象和 Cookie 管理对象，并且注入到 SecurityManager 中：

```java
    // 记住我 cookie
    @Bean
    public SimpleCookie rememberMeCookie(){
        System.out.println("ShiroConfiguration.rememberMeCookie()");
        //这个参数是cookie的名称，对应前端的 checkbox 的 name = rememberMe
        SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
        // cookie 生效时间30天,单位秒
        simpleCookie.setMaxAge(259200);
        return simpleCookie;
    }

    // cookie 管理对象
    @Bean
    public CookieRememberMeManager rememberMeManager(){
        System.out.println("ShiroConfiguration.rememberMeManager()");
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(rememberMeCookie());
        return cookieRememberMeManager;
    }
```

```java
    @Bean
    public SecurityManager securityManager(){
        DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
        // 设置 Realm
        securityManager.setRealm(myShiroRealm());
        // 注入缓存管理器
        securityManager.setCacheManager(ehCacheManager());
        //注入记住我管理器;
        securityManager.setRememberMeManager(rememberMeManager());
        return securityManager;
    }
```

然后在 ShiroFilterFactoryBean 中添加记住我过滤器： 

```java
 //配置记住我或认证通过可以访问的 URL
 filterMap.put("/select.html", "user");
```

注意，UserInfo 实体类必须序列化，不然实现不了记住我这个功能。

### 6.注册

根据 MVC 架构，先写 service 接口，然后实现：

```java
public interface UserInfoService {

    // 查询用户
    public UserInfo findByUsername(String username);

    // 用户注册
    public boolean signin(String username,String password,String name);
}
```

```java
@Service
public class UserInfoServiceImpl implements UserInfoService {

    //...  

    @Override
    public boolean signin(String username, String password, String name) {
        // 首先检查数据库是否已经存在该用户名
        if(findByUsername(username) != null){
            return false;
        } else {
            // 将用户名作为盐值，然后加密密码
            String salt = ByteSource.Util.bytes(username).toHex();
            // 第一个参数是加密算法，第四个参数是加密次数，这里设置的要和 Shiro 配置类中对凭证匹配器一致
            String cryptedPassword = new SimpleHash("MD5", password, salt, 2).toHex();
            UserInfo userInfo = new UserInfo();
            userInfo.setUsername(username);
            userInfo.setPassword(cryptedPassword);
            userInfo.setSalt(salt);
            userInfo.setName(name);
            userInfoRepository.save(userInfo); // 将用户保存到数据库
            return true;
        }
    }
}
```

最后写控制器：

```java
@Controller
public class SigninController {

    @Autowired
    private UserInfoService userInfoService;

    @PostMapping("/signin")
    public String signin(@RequestParam(name = "username") String username,
                         @RequestParam(name = "password") String password,
                         @RequestParam(name = "name") String name,
                         Model model){
        if(userInfoService.signin(username, password, name)){
            // 注册成功
            model.addAttribute("msg", "注册成功");
            // 重定向到登录页面
            return "redirect:/login.html";
        } else {
            model.addAttribute("msg", "用户名已存在");
            return "signin";
        }
    }
}
```



