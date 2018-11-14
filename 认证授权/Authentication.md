# Basic Authentication

Basic Authentication 是 HTTP 1.0 中引入的方案，十分古老，实现简单，但存在很大的缺陷。

## 一、流程

1.用户访问受限资源

```http
GET /protected_docs HTTP/1.1
Host: 127.0.0.1:3000
```

2.服务器要求身份认证

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm=protected_docs
```

响应首部中，通过`WWW-Authenticate`告知客户端，认证的方案是`basic`。同时以`realm`告知认证的范围。

3.用户发送认证请求

```http
GET /protected_docs HTTP/1.1
Authorization: Basic Y2h5aW5ncDoxMjM0NTY=
```

`Authorization`首部的格式为`Basic base64(userid:password)`，就是以用户名追加一个冒号然后拼接上密码，并将得出的结果字符串再用 Base64 算法编码。

```java
// 使用 Java 8 提供的 Base64 工具类
byte[] encoded = Base64.getEncoder().Encode("username:password".getBytes());
String encodedInfo = new String(encoded);
```

4.服务器验证请求

服务端收到用户的认证请求后，对请求进行验证。验证包含如下步骤：

- 根据用户请求资源的地址，确定资源对应的realm。
- 解析 Authorization 请求首部，获得用户名、密码。

```java
byte[] decoded = Base64.getDecoder().decode(encodedInfo.getBytes());
String decodedInfo = new String(decoded);
String username = decodedInfo.substring(0,decodedInfo.lastIndexOf(":"));
String password = decodedInfo.substring(decodedInfo.latIndexOf(":") + 1);
```

- 判断用户是否有访问该realm的权限。

- 验证用户名、密码是否匹配。

一旦上述验证通过，则返回请求资源。如果验证失败，则返回401要求重新认证，或者返回403（Forbidden）。

## 二、缺陷

1. 在传输层未加密的情况下，用户明文密码可被中间人截获。
2. 明文密码一旦泄露，如果用户其他站点也用了同样的明文密码（大概率），那么用户其他站点的安全防线也告破。

## 三、改进

1. 传输层未加密的情况下，不要使用 Basic 认证。
2. 如果使用 Basic 认证，登录密码由服务端生成。
3. 如果可能，不要使用 Basic 认证。

# Token Based Authentication

## 一、基本概念

用户输入用户名和密码，认证服务器验证通过之后就返回一个 token，只要 ==token 不过期==用户就可以用这个 token 在==同一 session== 中访问资源。

## 二、优点

1. Token 完全由应用管理，所以它可以避开同源策略
2. Token 可以避免 [CSRF 攻击](http://www.cnblogs.com/shanyou/p/5038794.html)
3. Token 可以是无状态的，可以在多个服务间共享

## 三、JWT（JSON Web Token）

JSON Web Token（缩写 JWT）是目前最流行的跨域认证解决方案，它已经是一种标准。

JWT 适用于前后端分离的场景。

JWT 本身是开源的，主流语言都有其对应的实现。

### 1.原理

服务器认证以后，生成一个 JSON 对象，发回给用户。用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。这样一来服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

### 2.数据结构

JWT 实际上就是个字符串，用`.`分割成了三个部分：

```javascript
Header.Payload.Signature
```

（1）Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Header 部分是一个 JSON 对象，描述 JWT 的元数据，`alg`属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；`typ`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。 

这个部分会用 Base64 URL 算法编码成字符串。JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略，`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。

（2）Payload

这也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段供选用。除了官方字段，你还可以在这个部分定义私有字段.

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。

这个部分也用 Base64 URL 编码成字符串。

（3）Signature

Signature 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

### 3.常见使用方式

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息`Authorization`字段里面。

```http
Authorization: Bearer <token>
```

另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面。

## 五、JWT 的几个特点

（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。

（2）JWT 不加密的情况下，不能将秘密数据写入 JWT。

（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

### Spring Boot 和 JWT 简单整合

使用 [jjwt ](https://github.com/jwtk/jjwt)这个开源的 Java JWT 实现。

#### 1.登录认证

```java
@RestController
@Component
@PropertySource({"classpath:config/config.properties"})
public class LoginController {

    @Autowired
    private StringRedisTemplate redisTemplate;

    // 这个值在实际开发中是预先设置好的，一般会从配置中读取
    @Value("${jwt.secret}")
    private String secret;

    @PostMapping("/login")
    @ResponseBody
    public RestResponse login(@RequestParam String username,
                              @RequestParam String password,
                              HttpServletResponse response){

        if (username.equals("user") && password.equals("password")){
            // 生成 JWT 发送给客户端
            // uid 是用户 id，过期时间设置为5天
            String uid = "0";
            String jwt = createJWT(uid, username,secret);

            // 将 JWT 存入缓存
            redisTemplate.opsForValue().set("jwt" + uid, jwt);
            redisTemplate.expire("jwt" + uid, 24 * 5, TimeUnit.HOURS);  // 有效时间是5天

            // 设置 Cookie
            Cookie token = new Cookie("jwt" + uid, jwt);
            token.setMaxAge(5 * 24 * 60 * 60);  // 设置过期时间
            response.addCookie(token);

            return new RestResponse(0, "login successfully", null);
        } else {
            return new RestResponse(0, "login failed", null);
        }
    }

    // 生成 JWT
    private String createJWT(String uid,String subject,String secret){
        // 用字符串生成 key
        byte[] hsKey = secret.getBytes(StandardCharsets.UTF_8);
        Key key = new SecretKeySpec(hsKey, SignatureAlgorithm.HS256.getJcaName());
        
        JwtBuilder builder = Jwts.builder()
                .setId(uid)
                .setSubject(subject)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .signWith(key);
        return builder.compact();
    }

}
```

如果用户登录成功，那么就在响应中加上一个 cookie，用来保存 JWT。注意一下如何从配置文件中读取 secret（Sting 类型）然后生成 key。

#### 2.访问受限资源前先认证

因为采用的是前后端分离，视图完全由前端框架控制，所以要求前端在跳转到受限资源页面之前先向后端发一个认证请求：

```js
  beforeCreate: function () {
      // 发送 cookie，请求认证
      axios.defaults.withCredentials = true;
      axios.get('http://localhost:8088/authentication')
      .then((response) => {
          if (response.data['code'] === 1){
              // 没有认证，跳转到登录页面
              this.$router.push('/');
          } else {
              this.show = true;
          }
      })
      .catch((error) => {
          alert(error);
      });
  }
```

后端相应的 controller 就来处理这个请求，认证通过就允许前端显示受限资源，否则前端就会跳转到登陆页面：

```java
/**
 * 这个 controller 实际上起着拦截器的作用
 * 访问受限资源之前先经过它来验证 JWT
 */
@RestController
@Component
@PropertySource({"classpath:config/config.properties"})
public class JWTController {
    @Value("${jwt.secret}")
    private String secret;

    @Autowired
    private StringRedisTemplate redisTemplate;

    @GetMapping("/authentication")
    @ResponseBody
    public RestResponse verifyJWT(HttpServletRequest request) {
        Cookie[] cookies = request.getCookies();
        for (Cookie c : cookies) {
            // 检查有没有相应的 cookie
            if (c.getName().contains("jwt")) {
                String jwtId = c.getName();
                String jwt = c.getValue();
                if (verify(jwtId, jwt)) {
                    return new RestResponse(0, "authenticated successfully", null);
                } else {
                    break;
                }
            }
        }
        return new RestResponse(1, "authenticated unsuccessfully", null);
    }

    private boolean verify(String clientJWTId, String clientJWT) {
        // 从缓存中获取 JWT 进行验证
        Object o = redisTemplate.opsForValue().get(clientJWTId);
        if (o != null) {
            String cacheJWT = (String) o;
            // 转换
            byte[] hsKey = secret.getBytes(StandardCharsets.UTF_8);
            Key key = new SecretKeySpec(hsKey, SignatureAlgorithm.HS256.getJcaName());
            Claims cacheClaims = Jwts.parser().setSigningKey(key).parseClaimsJws(cacheJWT).getBody();
            Claims clientClaims = Jwts.parser().setSigningKey(key).parseClaimsJws(clientJWT).getBody();
            // 验证
            if (cacheClaims.getId().equals(clientClaims.getId())
                    && cacheClaims.getSubject().equals(clientClaims.getSubject())
                    && cacheClaims.getIssuedAt().equals(clientClaims.getIssuedAt())) {
                return true;
            }
        }
        return false;
    }
}
```

# OpenID

