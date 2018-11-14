# OAuth

## 一、使用场景

假设 LinkedIn （3rd-party application）现在需要读取用户（resource owner）的 Gmail（resource provider） 中的联系人列表，以添加好友。要实现这一点就要给予 LinkedIn 访问 Gmail 联系人列表的权限，但如果直接把 Gmail 的用户名和密码提供给 LinkedIn 就显然会存在较大的安全隐患。

OAuth（Open Authorization） 就是为了解决这个问题出现的，只要 Gmail 支持 OAuth 协议，那么 LinkedIn 就可向用户请求授权访问 Gmail 。

OAuth允许只读/读写权限，支持划分不同的访问粒度，并且允许用户管理自己的授权（比如收回授权）。

注意，OAuth 是一个开放协议，并不是标准。厂商可以根据自己的情况进行实现。

OAuth 2.0 是 OAuth 协议的下一版本，但不向下兼容 OAuth 1.0。OAuth 2.0关注客户端开发者的简易性。

## 二、OAuth 2.0 工作流程

![OAuth 2.0](img\OAuth 2.0.png)

（A）用户打开客户端（即第三方应用）以后，客户端要求用户给予授权。

（B）用户同意给予客户端授权。

（C）客户端使用上一步获得的授权，向认证服务器申请令牌。

（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。

（E）客户端使用令牌，向资源服务器申请获取资源。

（F）资源服务器确认令牌无误，同意向客户端开放资源。

从上可知，用户怎样给予客户端授权是问题的关键。

如果用户访问的时候，客户端的"访问令牌"已经过期，则需要使用"更新令牌"申请一个新的访问令牌。

## 三、客户端授权的四种模式

### 1.授权码（authorization code）模式

通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

![authorization code](img\authorization code.png)

（A）用户（就是 resource owner）访问客户端，客户端将前者导向认证服务器。

（B）用户选择是否给予客户端授权。

（C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。

（D）客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。

（E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

### 2.简化模式（implicit）

不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。

![implicit](img\implicit.png)

（A）客户端将用户导向认证服务器。

（B）用户决定是否给于客户端授权。

（C）假设用户给予授权，认证服务器将用户导向客户端指定的"重定向URI"，并在 URI 的 Hash 部分包含了访问令牌。

（D）浏览器向资源服务器发出请求，其中不包括上一步收到的 Hash 值。

（E）资源服务器返回一个网页，其中包含的代码可以获取 Hash 值中的令牌。

（F）浏览器执行上一步获得的脚本，提取出令牌。

（G）浏览器将令牌发给客户端。

### 3.密码模式

用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。

在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分，或者由一个著名公司出品。而认证服务器只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。

![password mode](img\password mode.png)

（A）用户向客户端提供用户名和密码。

（B）客户端将用户名和密码发给认证服务器，向后者请求令牌。

（C）认证服务器确认无误后，向客户端提供访问令牌。

### 4.客户端模式

客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于 OAuth 框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。

![client credentials grant](img\client credentials grant.png)

（A）客户端向认证服务器进行身份认证，并要求一个访问令牌。

（B）认证服务器确认无误后，向客户端提供访问令牌。

## 四、Spring Boot 简单整合 OAuth

[scribejava](https://github.com/scribejava/scribejava) 是用起来相对简单的一个库，参照官方文档，第三方应用授权只需经过以下几个步骤：

- 获取认证 URL
- 获取 Access Token
- 获取受限资源

下面的例子中以 GitHub 作为 resource provider。

### 1.参照 GitHub 开发者文档，注册 OAuth 应用

参照 [Web application flow](https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/#web-application-flow) 这一节的内容可知，首先需要按照要求[注册 web 应用](https://github.com/settings/applications/new)，注册成功之后注意记录下 Client ID，Client Secret 和 callback URL 这三项。

### 2.配置 OAuthService 类

`OAuth20Service`是用来实现 OAuth 2.0 的核心类。

```java
/**
 * 配置认证的相关参数
 */
@Configuration
public class OAtuth20ServiceConfig {

    @Value("${oauth.clientId}")
    private String clientId;

    @Value("${oauth.clientSecret}")
    private String clientSecret;

    @Value("${oauth.secretState}")
    private String secretState;  // 这个是用来检验认证状态的，如果 state 不一致，认证过程就不能继续进行

    @Bean(name = "OAuth20Service")
    public OAuth20Service OAuth20Service(){
        return new ServiceBuilder(clientId)
                .apiSecret(clientSecret)
                .state(secretState)
                .callback("http://localhost:8081/authorization")  // 重定向到前端页面
                .build(GitHubApi.instance());
    }
}
```

### 3.编写相关 controller

授权的第一步是请求用于认证的 URL：

```java
    // 获取认证 URL
    @GetMapping("/authorization")
    public String getAuthorizationUrl() {
        // Obtain the Authorization URL
        final String authorizationUrl = service.getAuthorizationUrl();
        return authorizationUrl;
    }
```

把这个 URL 发给前端让用户点击，用户点击之后 GitHub 会首先检查 Client ID 和 Client Secret 是否和注册应用时生成的一致。如果一致，GitHub 就会重定向到 callback URL，并且附上两个 URL 查询参数，code 和 state，此时前端要把这个 state 发给后端进行验证：

```java
    // 验证 state 是否一致
    @PostMapping("/verification")
    public String verify(@RequestParam(name = "state") String state) {
        if (!state.equals(secretState)) {
            return "Ooops, state value does not match!";
        } else {
            return "successful";
        }
    }
```

如果 state 没有发生变化，那就可以用先前得到的 code 向 GitHub 申请 Access Token；得到 token 之后就可以请求数据：

```java
    // 获取资源
    @GetMapping("/resource/{code}")
    public String getResource(@PathVariable(name = "code") String code) throws InterruptedException, ExecutionException, IOException {
        // Trade the Request Token and Verfier for the Access Token
        final OAuth2AccessToken accessToken = service.getAccessToken(code);

        // Now let's go and ask for a protected resource!
        final OAuthRequest request = new OAuthRequest(Verb.GET, "https://api.github.com/user");
        service.signRequest(accessToken, request);
        final Response response = service.execute(request);

        if (response.getCode() == 200) {
            return response.getBody();
        } else {
            return "null";
        }
    }
```

最后得到的数据长这样：

```json
{ "login": "maxwellhertz", "id": 34884395, "node_id": "MDQ6VXNlcjM0ODg0Mzk1", "avatar_url": "https://avatars1.githubusercontent.com/u/34884395?v=4", "gravatar_id": "", "url": "https://api.github.com/users/maxwellhertz", "html_url": "https://github.com/maxwellhertz", "followers_url": "https://api.github.com/users/maxwellhertz/followers", "following_url": "https://api.github.com/users/maxwellhertz/following{/other_user}", "gists_url": "https://api.github.com/users/maxwellhertz/gists{/gist_id}", "starred_url": "https://api.github.com/users/maxwellhertz/starred{/owner}{/repo}", "subscriptions_url": "https://api.github.com/users/maxwellhertz/subscriptions", "organizations_url": "https://api.github.com/users/maxwellhertz/orgs", "repos_url": "https://api.github.com/users/maxwellhertz/repos", "events_url": "https://api.github.com/users/maxwellhertz/events{/privacy}", "received_events_url": "https://api.github.com/users/maxwellhertz/received_events", "type": "User", "site_admin": false, "name": null, "company": null, "blog": "", "location": null, "email": null, "hireable": null, "bio": "Because he knows who I am.", "public_repos": 37, "public_gists": 0, "followers": 0, "following": 0, "created_at": "2017-12-27T09:56:04Z", "updated_at": "2018-10-12T10:43:54Z" }
```

实际上就是一个 JSON 数据，里面包含了用户名、repo 数量等信息，比如`repos_url`这个 URL 指向的就是用户的 repo 。

`scribejava`这个库还集成了 Facebook、Twitter 等应用的认证，不管是使用哪个应用的认证，在正式开发之前都应该先阅读相应的开发者文档。

从上面的例子可以看出，提供 OAuth 的厂商大多采用的是授权码模式。











