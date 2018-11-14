使用 Spring Boot 进行 Web 开发十分简便，只需创建 Spring Boot 然后选择所需的模块，再根据需要进行少量的自定义配置，编写业务代码后就可以运行 Web 应用。

# Spring Boot 映射静态资源

## 一、webjars 映射

`WebMvcAutoConfiguration`类是对 MVC 的自动配置，有一个添加资源映射的方法：

```java
 public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

            }
}
```

`/webjars/**`会被映射到`classpath:/META-INF/resources/webjars/`，`webjars `代表所有的静态资源以 jar 包的形式引入。[WebJars](https://www.webjars.org/) 提供了常用静态资源的 jar 包，引入静态资源的 pom 为：

```xml
        <!-- 引入 jquery -->
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.3.1</version>
</dependency>
```

引入后，静态资源内的目录结构为：

![静态资源](G:\notebook\Java\Spring Boot\img\静态资源.png)

访问这个资源的 URL 为 `http://localhost:8080/webjars/jquery/3.3.1/jquery.js`

可以用 `ResourceProperties` 类来设置与静态资源相关的参数，比如缓存时间等等。

```java
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
```

## 二、/** 映射

以下文件夹都是保存静态资源的文件夹，当一个 URL 没有找到对应的映射时，就会被映射到这些文件夹：

- `classpath:/META-INF/resources/`
- `classpath:/resources/`
- `classpath:/static/`
- `classpath:/public/`
- `/`（当前项目的根路径）

在 IDEA 中的目录结构则为：

![静态资源文件夹](G:\notebook\Java\Spring Boot\img\静态资源文件夹.png)

比如在`classpath:/static/`目录下有一个 min.js ，那么就可以直接访问`localhost:8080/min.js`。

## 三、首页映射

静态资源文件夹下的所有 index.html 的映射为 `/`，也就是说访问 `localhost:8080/`就会得到首页。

## 四、图标映射

把网页的图标放在静态资源文件夹下，把文件名改为 `favicon.ico`，那么浏览器中该网页的标签的图标就会变成指定的图标。

![图标](G:\notebook\Java\Spring Boot\img\图标.png)

## 五、配置自定义的静态资源文件夹

在 application 文件中添加配置：

```properties
spring.resources.static-location=classpath:/hello/,classpath:/whu/
```

这个配置就覆盖了默认的静态资源文件夹。

# 模板引擎

## 一、模板引擎的作用

![模板引擎](G:\notebook\Java\Spring Boot\img\模板引擎.png)

## 二、引入模板引擎  Thymeleaf   

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

Thymeleaf 已经为静态资源配置了前缀 `classpath:/templates/`和后缀`.html`，只要把 HTML 页面放在相应路径下，Thymeleaf 就会自动渲染。

## 三、在开发期间时使模板引擎实时生效

首先在 application 文件中添加配置：

```properties
# 禁用缓存
spring.thymeleaf.cache=false
```

然后每当页面修改完成以后按 `Ctrl + F9`，重新编译。

# Spring Boot 对 Spring MVC 的自动配置

[Spring Boot 开发 web 应用](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-developing-web-applications)

## 一、Spring Boot 的自动配置项

1. Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
   - 自动配置了视图解析器，其中`ContentNegotiatingViewResolver` 组合所有的视图解析器。
   - 可以向容器中添加一个定制的视图解析器（就是加上 `@Bean`），Spring Boot 会自动配置。
2. Support for serving static resources, including support for WebJars 
   - 静态资源文件夹路径，包括 webjars
3. Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
   - 类型转换器、格式化器
   - 需要在配置文件中配置日期格式化的规则 
   - 自定义的转换器也需要加入容器中
4. Support for `HttpMessageConverters`.
   - `HttpMessageConverters`用来转换 HTTP 请求和响应，比如把方法返回的对象转换成 JSON
   - 可以定制
5. Automatic registration of `MessageCodesResolver` .
   - 定义错误代码生成规则
6. Static `index.html` support.
   - 静态首页
7. Custom `Favicon` support 
   - 静态图标
8. Automatic use of a `ConfigurableWebBindingInitializer` bean 
   - 初始化 `WebDataBinder`，把请求数据绑定到 Javabean
   - 可以定制

## 二、修改 Spring Boot  默认配置的思路

首先了解 Spring Boot 进行自动配置的模式：

> 先检查容器中是否有用户自己配置的组件（这些组件加上了 @Bean或者 @Component），如果有就用用户自己配置的，如果没有就用默认配置。如果有些组件可以用多个配置（比如视图解析器），那么就将用户配置的和默认配置的组合起来。

如果想要修改默认配置，那么就可以从两方面入手：

- 扩展原有配置
- 全面接管，覆盖原有配置

## 三、扩展 Spring MVC

如果想要在保持 Spring Boot 默认配置的基础上添加自己的配置，那么可以写一个 `WebMvcConfigurer`接口的实现类，并且加上 `@Configuration`，注意==不能加上 `@EnabledWebMvc`==。

下面的例子说明了如何在 Spring Boot 中自定义 URL 的映射：

```java
@Configuration
public class MyConfiguration implements WebMvcConfigurer{

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // 浏览器请求 /successful 就得到 success.html 页面
        registry.addViewController("/successful").setViewName("success");
    }
    
  // 这是第二种方法，但显然没必要这么做  
  /* @Bean // 返回的 WebMvcConfigurer 要注册到容器中
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("login");
                registry.addViewController("/index.html").setViewName("login");
            }
        };
    }*/
}
```

新版本的 Spring Boot 中`WebMvcConfigurer`接口中的方法都是默认方法，所以也就无需再显式地 `implements WebMvcConfigurer`。

## 四、全面接管 Spring MVC

全面接管意味着完全覆盖 Spring Boot 的默认配置项，只需在自己实现的配置类上加上 `@Configuration` 和 ==`@EnabledWebMvc`==。

# Web 开发环境准备

## 一、引入资源

注意静态资源的引入，以及在前端页面中如何引用

## 二、导入 POJO 

## 三、设置整个应用的根 URL

在 application 文件中添加配置：

```properties
server.servlet.context-path=/crud
```

 # 国际化

## 一、编写国际化配置文件

抽取页面中需要国际化的内容，在配置文件中进行配置。在 IDEA 中进行国际化的配置非常简便：

![国际化配置](G:\notebook\Java\Spring Boot\img\国际化配置.png)

## 二、Spring Boot 自动配置国际化管理组件

```java
@Configuration
@ConditionalOnMissingBean(value = MessageSource.class, search = SearchStrategy.CURRENT)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Conditional(ResourceBundleCondition.class)
@EnableConfigurationProperties
public class MessageSourceAutoConfiguration {

	private static final Resource[] NO_RESOURCES = {};

	@Bean
	@ConfigurationProperties(prefix = "spring.messages")
	//...

	@Bean
	public MessageSource messageSource() {
    	//...
    }
    //...
    	protected static class ResourceBundleCondition extends SpringBootCondition {

		@Override
		public ConditionOutcome getMatchOutcome(ConditionContext context,
				AnnotatedTypeMetadata metadata) {
			String basename = context.getEnvironment()
					.getProperty("spring.messages.basename", "messages");
			ConditionOutcome outcome = cache.get(basename);
			if (outcome == null) {
				outcome = getMatchOutcomeForBasename(context, basename);
				cache.put(basename, outcome);
			}
			return outcome;
		}
        //...
        }
}
```

## 三、指定国际化配置文件的基础名

在 application 文件中添加配置：

```properties
spring.messages.basename=i18n.login
```

避免出现乱码（最好在 Default Settings 中进行设置）：

![配置文件乱码](G:\notebook\Java\Spring Boot\img\配置文件乱码.png)

## 四、自定义 Locale 组件

默认情况下，Spring Boot 会根据请求首部中的语言信息进行国际化（原理： Locale 组件），如果想要由用户自己决定国际化，就需要编写自定义的 Locale 组件。

在新版的 Spring Boot（我这里用的是 2.0.3），自定义的 Locale 组件会覆盖默认的，也就是说无法通过获取请求首部中的语言信息进行国际化，因此这里要自己进行默认设置：

```java
// 定制国际化组件
public class MyLocaleResolver implements LocaleResolver {
    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {

        String l = httpServletRequest.getParameter("l"); // 获取请求首部中的国际化参数 l=en_US
        Locale locale = null;
        if (!StringUtils.isEmpty(l)) {
            String[] split = l.split("_"); // 分割为 en US
            locale = new Locale(split[0], split[1]);
        } else {
            locale =  new Locale("en", "US"); // 默认用英语
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}

```

然后在自定义配置类 `MyMvcConfig` 中添加这个组件：

```java
    @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
```

# 登录

登录成功后为防止登录表单重复提交，进行重定向：

```java
@Controller
public class loginController {

    @PostMapping(value="/user/login") // 新版 Spring Boot 提供的 REST API
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Map<String,Object> map,
                        HttpSession session){
        if(!StringUtils.isEmpty(username) && password.equals("1234")){ // 这是模板引擎 Thymeleaf 提供的工具类
            session.setAttribute("username", username); // 用 session 来保存登录成功的用户
            return "redirect:/main.html"; // 登录成功后为防止表单重复提交，于是进行重定向
        } else {
            map.put("msg","用户名密码错误"); // 这个参数由模板引擎获取
            return "login";
        }

    }
}
```

在配置类中添加视图解析器：

```java 
registry.addViewController("main.html").setViewName("dashboard");
```

为了防止不登录就直接访问主页，增加拦截器：

```java 
package springbootwebcrud.component;


import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

// 没有登录的页面不能访问主页
public class LoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String username = (String) request.getSession().getAttribute("username");
        if(username != null){
            return true; // 登录成功，放行
        } else { // 未登录，转发到登录页面
            request.setAttribute("msg", "请先登录！");
            request.getRequestDispatcher("/index.html").forward(request, response);
            return false;
        }

    }
}

```

记得在 MVC 配置类中注册拦截器：

```java
    // 注册拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginHandlerInterceptor())
                .addPathPatterns("/main.html")  // 设置需要拦截的请求
                .excludePathPatterns("/","index.html","/user/login"); // 把对登录页面和登录处理的请求排除
                                                                      // Spring Boot 已经设置好了静态资源的映射
    }
```

注意，Spring Boot 2.0.4增加了对静态资源的拦截，需要排除。

# RESTful CRUD

```java
@Controller
public class EmployeeController {

    @Autowired
    private EmployeeDao employeeDao;

    @Autowired
    private DepartmentDao departmentDao;

    // 查询所有的员工
    @GetMapping("/emps")
    public String queryEmployeeList(Model model) {

        Collection<Employee> employees = employeeDao.getAll();
        model.addAttribute("emps", employees); // 添加到视图中
        return "emp/list"; // Thymeleaf 会自动对应到 classpath:templates/
    }

    // 访问员工添加页面
    @GetMapping("/emp")
    public String toAddPage(Model model) {
        Collection<Department> departments = departmentDao.getDepartments(); // 查询所有的部门
        model.addAttribute("depts", departments); // 在页面显示所有的部门
        return "emp/add";
    }

    // 员工添加
    @PostMapping("/emp")
    public String addEmployee(Employee employee) { // Spring MVC 会自动绑定参数
        // 注意日期时间格式
        // 可以自行配置默认的日期时间格式 spring.mvc.date-format=yyyy-MM-dd
        employeeDao.save(employee);
        return "redirect:/emps";
    }

    // 访问修改页面，请求的 URL 中带有要查询的员工 id
    @GetMapping("/emp/{id}")
    public String toEditPage(@PathVariable("id") Integer id, Model model) {
        Employee employee = employeeDao.get(id);
        model.addAttribute("emp", employee); // 查询指定员工信息并回显到页面

        Collection<Department> departments = departmentDao.getDepartments();
        model.addAttribute("depts", departments); // 页面也要显示所有的部门
        return "emp/add";
    }

    // 员工修改，需要员工的 id
    @PutMapping("/emp")
    public String updateEmployee(Employee employee) {
        employeeDao.save(employee);
        return "redirect:/emps";
    }

    // 员工删除
    @DeleteMapping("/emp/{id}")
    public String deleteEmployee(@PathVariable("id") Integer id) {
        employeeDao.delete(id);
        return "redirect:/emps";
    }
}
```

# 错误处理

## 一、Spring Boot 默认的错误处理机制

如果是客户端是浏览器，那么会返回一个默认的错误页面。

如果是其他客户端，默认响应 JSON 。

```json
{
    "timestamp": "2018-07-06T07:29:24.303+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/crud/aaa"
}
```

### 错误处理机制原理

Spring Boot 向容器中添加了以下相关组件：

- `DefaultErrorAttributes`：定义所有错误页面都可以使用的信息，比如时间戳、状态码、错误提示等等
- `BasicErrorController`：处理错误请求，返回错误页面或者 JSON
- `ErrorPageCustomizer`：定制错误处理规则
- `DefaultErrorViewResolver`：错误视图解析器，得到某个错误页面

如果出现 `4xx`或者`5xx`状态，那么`ErrorPageCustomizer`定制的错误处理规则就会起作用，然后被`BasicErrorController`处理，它会决定返回错误页面还是 JSON；如果返回错误页面，那么就由`DefaultErrorViewResolver`找到某个错误页面。

## 二、定制默认错误处理机制

### 1.定制错误页面

- 如果有模板引擎，那么在模板文件夹`templates`下新建一个`error`文件夹，这个文件夹用来放错误页面。错误页面的名称为`状态码.html`，比如`404.html`就是404错误的错误页面。`4xx.html`和`5xx.html`分别用来匹配4xx和5xx的所有错误，但是 Spring Boot 会优先查找精确的错误页面。
- 如果没有模板引擎，那么可以把`error`文件夹放在`static`文件夹下。
- 如果`templates`和`static`下都没有`error`文件夹，那么 Spring Boot 就会使用默认的错误页面。

### 2.定制错误 JSON 

可以自定义一个异常类：

```java
public class UserNotExistException extends RuntimeException{

    public UserNotExistException() {
        super("用户不存在！");
    }
}
```

- 自定义异常处理器，返回定制 JSON

  ```java
  @ControllerAdvice // 异常处理器类必须加上这个注解
  public class MyExceptionHandler {
  
      @ResponseBody
      @ExceptionHandler(UserNotExistException.class) // 指定处理哪种异常
      public Map<String, Object> handleException(Exception e){
          Map<String, Object> map = new HashMap<>();
          map.put("code", "user.notexist");
          map.put("message", e.getMessage());
          return map;
      }
  }
  ```

  缺点：不管是什么客户端都会得到 JSON，没有自适应效果。

  实现自适应效果的关键是传入状态码，让 Spring Boot 查找相应的错误页面：

  ```java
  @ControllerAdvice // 异常处理器类必须加上这个注解
  public class MyExceptionHandler {
  
      @ExceptionHandler(UserNotExistException.class) // 指定处理哪种异常
      public String handleException(Exception e, HttpServletRequest request){
          Map<String, Object> map = new HashMap<>();
          request.setAttribute("javax.servlet.error.status_code", 404); // 注意要传入状态码
          map.put("code", "user.notexist");
          map.put("message", e.getMessage());
          request.setAttribute("ext", map); // 传入定制的错误信息
          return "forward:/error";
      }
  }
  ```

  此时虽然实现了页面/JSON  的自适应，但是无法响应定制的错误信息，解决办法有：

  1. 写一个`ErrorController`的实现类或者`AbstractErrorController`子类，加到容器中。

  2. 因为容器中的`DefaultErrorAttributes`默认进行错误信息的处理，所以可以继承`DefaultErrorAttributes`，从而定制错误信息：

     ```java
     @Component
     public class MyErrorAttributes extends DefaultErrorAttributes {
         @Override
         public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
             Map<String, Object> map = super.getErrorAttributes(webRequest, includeStackTrace);
             // 从 request 中获取定制的错误信息
             Map<String, Object> ext = (Map<String, Object>) webRequest.getAttribute("ext", 0);
             map.put("ext", ext);
             return map;
         }
     }
     ```








