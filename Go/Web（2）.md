# 网络安全

==永远不要相信外部的输入==

## 一、CSRF 

CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。 ![csrf](img\csrf.png)

### 1.概述

从上图可以看出，要完成一次 CSRF 攻击，受害者必须依次完成两个步骤 ：

- 1.登录受信任网站 A，并在本地生成 Cookie 。
- 2.在不退出 A 的情况下，访问危险网站 B。

对于用户来说很难避免在登陆一个网站之后不点击一些链接进行其他操作，所以随时可能成为 CSRF 的受害者。

CSRF 攻击主要是因为 Web 的隐式身份验证机制，Web 的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，但却无法保证该请求是用户批准发送的。

### 2.防御手段

现在一般的 CSRF 防御在服务端进行。 主要从以下2个方面入手：

- 正确使用 GET, POST 和 Cookie；
- 在非 GET 请求中增加伪随机数；

第一步，限制对资源的访问方法，这里可以用 RESTful API 来实现：

```go
mux.Get("/user/:uid", getuser)
mux.Post("/user/:uid", modifyuser) // 对资源的修改只能用 POST
```

第二步，在非 GET 请求中添加随机数，防止伪造：

```go
h := md5.New()
io.WriteString(h, strconv.FormatInt(crutime, 10))
io.WriteString(h, "ganraomaxxxxxxxxx")
token := fmt.Sprintf("%x", h.Sum(nil))

t, _ := template.ParseFiles("login.html")
t.Execute(w, token)
```

## 二、输入过滤

过滤用户数据其实就是要验证数据的合法性。通过对所有的输入数据进行过滤，可以避免恶意数据在程序中被误用。

输入过滤通常由三个步骤组成：

### 1.识别数据的来源

数据可能来自客户端，也可能来自数据库或者第三方提供的接口数据。

### 2.验证数据的合法性，进行过滤

首先把握一个原则，绝对不要试图修改非法的数据。比如说绝对不能出现“当银行密码后两位是0时，输入前四位就能登录系统”的情况。

过滤数据经常用到的函数：

- `strconv`包中的字符串转换函数
- `string`包中的字符串修饰函数，如`Trim`
- `regexp`包中的正则匹配

### 3.区分过滤数据

在开发 Web 应用时需要区分合法和非法的数据，这样可以保证过滤数据的完整性而不影响输入的数据。可以把所有的经过过滤的数据放入一个全局`map`中，必须保证这里面存放的都是合法数据：

- 每个请求都需要把`map`初始化为空
- 进行检查，并阻止外部数据源的变量名和这个`map`一致

## 三、SQL 注入

预防 SQL 注入，可以考虑以下措施：

- 严格限制 Web 应用操作数据库的权限，赋予用户的权限应该是能满足其需要的最低权限
- 严格检查输入的数据
- 对输入数据库的特殊字符进行转义处理或者编码处理。`text/template`包中的`HTMLEscapeString`函数可以实现转义处理
- 使用数据库提供的参数化查询接口，不要直接拼接 SQL 语句
- 在发布应用之前用专业的 SQL 注入检测工具进行检测
- 避免网页打印出 SQL 错误信息，从而将 SQL 语句暴露出来

## 四、密码存储

### 1.普通方案

使用单向哈希算法。单向哈希算法的主要特征是无法通过哈希后的摘要恢复原始数据。

Go 提供的三种单向哈希算法：

```go
//import "crypto/sha256"
h := sha256.New()
io.WriteString(h, "His money is twice tainted: 'taint yours and 'taint mine.")
fmt.Printf("% x", h.Sum(nil))

//import "crypto/sha1"
h := sha1.New()
io.WriteString(h, "His money is twice tainted: 'taint yours and 'taint mine.")
fmt.Printf("% x", h.Sum(nil))

//import "crypto/md5"
h := md5.New()
io.WriteString(h, "需要加密的密码")
fmt.Printf("%x", h.Sum(nil))
```

单向哈希有两个特点：

- 同一个密码进行单向哈希，得到的总是唯一确定的摘要
- 计算速度快

利用第一个特点，攻击者可以将所有密码常见组合进行单向哈希，得到一个摘要组合，称为`ranibow table`，然后与数据库中的摘要进行比对。一旦数据库泄露，那么用户的密码就和明文无异。

### 2.进阶方案

采用“加盐”的方式存储密码：先将密码进行一次哈希，然后将得到的值加上一些只有管理员知道的随机串，最后再进行一次哈希。

```go
//import "crypto/md5"
//假设用户名abc，密码123456
h := md5.New()
io.WriteString(h, "123456")

//pwmd5等于e10adc3949ba59abbe56e057f20f883e
pwmd5 :=fmt.Sprintf("%x", h.Sum(nil))

//指定两个 salt： salt1 = @#$%   salt2 = ^&*()
salt1 := "@#$%"
salt2 := "^&*()"

//salt1+用户名+salt2+MD5拼接
io.WriteString(h, salt1)
io.WriteString(h, "abc")
io.WriteString(h, salt2)
io.WriteString(h, pwmd5)

last :=fmt.Sprintf("%x", h.Sum(nil))
```

但随着并行计算能力的提升，这种加盐的也可能被破解。

### 3.专家方案

只要时间与资源允许，没有破译不了的密码，所以方案是:故意增加密码计算所需耗费的资源和时间。这类方案有一个特点，算法中都有个因子，用于指明计算密码摘要所需要的资源和时间，也就是计算强度。计算强度越大，攻击者建立`rainbow table`越困难，以至于不可能继续。

`scrypt`方案是由著名的 FreeBSD 黑客 Colin Percival 为他的备份服务 Tarsnap 开发的。

```go
dk := scrypt.Key([]byte("some password"), []byte(salt), 16384, 8, 1, 32)
```

通过上面的的方法可以获取唯一的相应的密码值，这是目前为止最难破解的。

 ## 五、加密解密

### 1.BASE64

这是一种比较基本的方案：

```go
func base64Encode(src []byte) []byte {
    return []byte(base64.StdEncoding.EncodeToString(src))
}

func base64Decode(src []byte) ([]byte, error) {
    return base64.StdEncoding.DecodeString(string(src))
}

func main() {
    // encode
    hello := "你好，世界！ hello world"
    debyte := base64Encode([]byte(hello))
    fmt.Println(debyte)
    // decode
    enbyte, err := base64Decode(debyte)
    if err != nil {
        fmt.Println(err.Error())
    }

    if hello != string(enbyte) {
        fmt.Println("hello is not equal to enbyte")
    }

    fmt.Println(string(enbyte))
}
```

### 3.高级方案

Go 语言的`crypto`里面支持对称加密的高级加解密包有：

- `crypto/aes`包：AES(Advanced Encryption Standard)，又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。
- `crypto/des`包：DES(Data Encryption Standard)，是一种对称加密标准，是目前使用最广泛的密钥系统，特别是在保护金融数据的安全中。曾是美国联邦政府的加密标准，但现已被 AES 所替代。

两种方案的使用方法类似。

```go
var commonIV = []byte{0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f}

func main() {
    //需要去加密的字符串
    plaintext := []byte("My name is Astaxie")
    //如果传入加密串的话，plaint就是传入的字符串
    if len(os.Args) > 1 {
        plaintext = []byte(os.Args[1])
    }

    //aes的加密字符串
    key_text := "astaxie12798akljzmknm.ahkjkljl;k"
    if len(os.Args) > 2 {
        key_text = os.Args[2]
    }

    fmt.Println(len(key_text))

    // 创建加密算法aes
    c, err := aes.NewCipher([]byte(key_text))
    if err != nil {
        fmt.Printf("Error: NewCipher(%d bytes) = %s", len(key_text), err)
        os.Exit(-1)
    }

    //加密字符串
    cfb := cipher.NewCFBEncrypter(c, commonIV)
    ciphertext := make([]byte, len(plaintext))
    cfb.XORKeyStream(ciphertext, plaintext)
    fmt.Printf("%s=>%x\n", plaintext, ciphertext)

    // 解密字符串
    cfbdec := cipher.NewCFBDecrypter(c, commonIV)
    plaintextCopy := make([]byte, len(plaintext))
    cfbdec.XORKeyStream(plaintextCopy, ciphertext)
    fmt.Printf("%x=>%s\n", ciphertext, plaintextCopy)
}
```

# 国际化和本地化

国际化与本地化（Internationalization and localization,通常用 i18n 和 L10N 表示），国际化是将针对某个地区设计的程序进行重构，以使它能够在更多地区使用；本地化是指在一个面向国际化的程序中增加对新地区的支持。 

目前，Go 语言的标准包没有提供对 i18n 的支持，但有一些比较简单的第三方实现。

## 一、设置默认地区

### 1.Locale

Locale 是一组描述世界上某一特定区域文本格式和语言习惯的设置的集合。Locale 名通常由三个部分组成：

- 一个强制性的表示语言的缩写，例如`en`表示英文，`zh`表示中文。
- 跟在一个下划线之后，是一个可选的国家说明符，用于区分讲同一种语言的不同国家，例如`en_US`表示美国英语，而`en_UK`表示英国英语。
- 跟在一个句点之后，是可选的字符集说明符，例如`zh_CN.gb2312`表示中国使用 gb2312 字符集。 

### 2.设置 Locale

（1）通过域名参数

目前最常用的设置 Locale 的方式是在 URL 里面带上参数，例如 www.asta.com/hello?locale=zh 或者 www.asta.com/zh/hello 。这样我们就可以设置地区：`i18n.SetLocale(params["locale"])`。

（2）通过客户端

在一些特殊的情况下，我们需要根据客户端的信息而不是通过 URL 来设置 Locale，这些信息可能来自于客户端设置的喜好语言(浏览器中设置)，用户的 IP 地址，用户在注册的时候填写的所在地信息等。 

客户端请求的时候在 HTTP 头信息里面有`Accept-Language`，一般的客户端都会设置该信息：

```go
AL := r.Header.Get("Accept-Language")
if AL == "en" {
    i18n.SetLocale("en")
} else if AL == "zh-CN" {
    i18n.SetLocale("zh-CN")
} else if AL == "zh-TW" {
    i18n.SetLocale("zh-TW")
}
```

 在实际开发中，可能还需要进行更加严格的判断。

## 二、本地化资源

Go 把格式信息存储在 JSON 中，然后通过合适的方式展现出来。 

### 1.本地化文本消息

一种基本的解决方案是把`map`来存储不同语言的信息：

```go
func main() {
    locales = make(map[string]map[string]string, 2)
    en := make(map[string]string, 10)
    en["pea"] = "pea"
    en["bean"] = "bean"
    locales["en"] = en
    cn := make(map[string]string, 10)
    cn["pea"] = "豌豆"
    cn["bean"] = "毛豆"
    locales["zh-CN"] = cn
    lang := "zh-CN"
    fmt.Println(msg(lang, "pea"))
    fmt.Println(msg(lang, "bean"))
}

func msg(locale, key string) string {
    if v, ok := locales[locale]; ok {
        if v2, ok := v[key]; ok {
            return v2
        }
    }
    return ""
}
```

### 2.本地化日期和时间

这里要顾及时区和格式的问题。首先使用`time.LoadLocation(name string)`获取相应于地区的 Locale，比如`Asia/Shanghai`或`America/Chicago`对应的时区信息，然后再利用此信息与调用`time.Now`获得的`Time`对象协作来获得最终的时间。 

对于格式，可以采用类似于文本格式的处理方法：

```go
en["date_format"]="%Y-%m-%d %H:%M:%S"
cn["date_format"]="%Y年%m月%d日 %H时%M分%S秒"

fmt.Println(date(msg(lang,"date_format"),t))

func date(fomate string,t time.Time) string{
    year, month, day = t.Date()
    hour, min, sec = t.Clock()
    //解析相应的%Y %m %d %H %M %S然后返回信息
    //%Y 替换成2012
    //%m 替换成10
    //%d 替换成24
}
```

### 3.本地化货币

```go
en["money"] ="USD %d"
cn["money"] ="￥%d元"

fmt.Println(date(msg(lang,"date_format"),100))

func money_format(fomate string,money int64) string{
    return fmt.Sprintf(fomate,money)
}
```

### 4.本地化视图和资源

在实际开发中，静态资源的组织结构可能是：

```go
views
|--en  //英文模板
    |--images     //存储图片信息
    |--js         //存储JS文件
    |--css        //存储css文件
    index.tpl     //用户首页
    login.tpl     //登陆首页
|--zh-CN //中文模板
    |--images
    |--js
    |--css
    index.tpl
    login.tpl
```

那么就可以在合适的地方渲染：

```go
s1, _ := template.ParseFiles("views"+lang+"index.tpl")
VV.Lang=lang
s1.Execute(os.Stdout, VV)
```

而对于里面的`index.tpl`里面的资源设置如下：

```go
// js文件
<script type="text/javascript" src="views/{{.VV.Lang}}/js/jquery/jquery-1.8.0.min.js"></script>
// css文件
<link href="views/{{.VV.Lang}}/css/bootstrap-responsive.min.css" rel="stylesheet">
// 图片文件
<img src="views/{{.VV.Lang}}/images/btn.png">
```

## 三、国际化站点

如果一个网站需要支持多种语言，那么就需要制定一个组织结构，把相关的 Locale 配置组织好，然后进行国际化。

# 日志和错误处理

Go 提供的`log`包是一个简易的日志系统。[logrus](https://github.com/sirupsen/logrus) 和 [seelog](https://github.com/cihub/seelog) 是实际开发中应用得较多的第三方日志系统。

## 一、logrus

logrus 是用 Go 语言实现的一个日志系统，与标准库 log 完全兼容并且核心 API 很稳定,是 Go 语言目前最活跃的日志库。 

基本使用：

```go
func init() {
	// 日志格式化为JSON而不是默认的ASCII
	log.SetFormatter(&log.JSONFormatter{})

	// 输出stdout而不是默认的stderr，也可以是一个文件
	log.SetOutput(os.Stdout)

	// 只记录严重或以上警告
	log.SetLevel(log.WarnLevel)
}

func main() {
	// 因为在 init 函数中已经设置日志的最低级别是 warn，所以这一条信息实际上不会输出
	log.WithFields(log.Fields{ 
		"animal": "walrus",
		"size":   10,
	}).Info("A group of walrus emerges from the ocean")

	log.WithFields(log.Fields{
		"omg":    true,
		"number": 122,
	}).Warn("The group's number increased tremendously!")

	log.WithFields(log.Fields{
		"omg":    true,
		"number": 100,
	}).Fatal("The ice breaks!")

	// 通过日志语句重用字段
	// logrus.Entry返回自WithFields()
	contextLogger := log.WithFields(log.Fields{
		"common": "this is a common field",
		"other":  "I also should be logged always",
	})

	contextLogger.Info("I'll be logged with common and other field")
	contextLogger.Info("Me too")
}
```

## 二、seelog

seelog 是用 Go 语言实现的一个日志系统，它提供了一些简单的函数来实现复杂的日志分配、过滤和格式化。主要有如下特性：

- XML 的动态配置，可以不用重新编译程序而动态的加载配置信息
- 支持热更新，能够动态改变配置而不需要重启应用
- 支持多输出流，能够同时把日志输出到多种流中、例如文件流、网络流等
- 支持不同的日志输出
  - 命令行输出
  - 文件输出
  - 缓存输出
  - 支持 log rotate
  - SMTP 邮件

使用的时候可以先自定义一个日志处理包：

```go
package logs

import (
    // "errors"
    "fmt"
    // "io"

    seelog "github.com/cihub/seelog"
)

var Logger seelog.LoggerInterface

func loadAppConfig() {
    appConfig := `
<seelog minlevel="warn">
    <outputs formatid="common">
        <rollingfile type="size" filename="/data/logs/roll.log" maxsize="100000" maxrolls="5"/>
        <filter levels="critical">
            <file path="/data/logs/critical.log" formatid="critical"/>
            <smtp formatid="criticalemail" senderaddress="astaxie@gmail.com" sendername="ShortUrl API" hostname="smtp.gmail.com" hostport="587" username="mailusername" password="mailpassword">
                <recipient address="xiemengjun@gmail.com"/>
            </smtp>
        </filter>
    </outputs>
    <formats>
        <format id="common" format="%Date/%Time [%LEV] %Msg%n" />
        <format id="critical" format="%File %FullPath %Func %Msg%n" />
        <format id="criticalemail" format="Critical error on our server!\n    %Time %Date %RelFile %Func %Msg \nSent by Seelog"/>
    </formats>
</seelog>
`
    logger, err := seelog.LoggerFromConfigAsBytes([]byte(appConfig))
    if err != nil {
        fmt.Println(err)
        return
    }
    UseLogger(logger)
}

func init() {
    DisableLog()
    loadAppConfig()
}

// DisableLog disables all library log output
func DisableLog() {
    Logger = seelog.Disabled
}

// UseLogger uses a specified seelog.LoggerInterface to output library log.
// Use this func if you are using Seelog logging system in your app.
func UseLogger(newLogger seelog.LoggerInterface) {
    Logger = newLogger
}
```

然后再使用：

```go
package main

import (
    "net/http"
    "project/logs"
    "project/configs"
    "project/routes"
)

func main() {
    addr, _ := configs.MainConfig.String("server", "addr")
    logs.Logger.Info("Start server at:%v", addr)
    err := http.ListenAndServe(addr, routes.NewMux())
    logs.Logger.Critical("Server err:%v", err)
}	
```

## 三、处理错误

示例：

````go
// 记录错误，重定向到错误页面
	func NotFound404(w http.ResponseWriter, r *http.Request) { 
        log.Error("页面找不到")   //记录错误日志
        t, _ = t.ParseFiles("tmpl/404.html", nil)  //解析模板文件
        ErrorInfo := "文件找不到" //获取当前用户信息
        t.Execute(w, ErrorInfo)  //执行模板的merger操作
    }

// 记录错误，一般还要发邮件通知管理员，然后重定向到错误页面
    func SystemError(w http.ResponseWriter, r *http.Request) {
        log.Critical("系统错误")   //系统错误触发了Critical，那么不仅会记录日志还会发送邮件
        t, _ = t.ParseFiles("tmpl/error.html", nil)  //解析模板文件
        ErrorInfo := "系统暂时不可用" //获取当前用户信息
        t.Execute(w, ErrorInfo)  //执行模板的merger操作
    }
````

## 四、处理异常

其实很多错误都是可以预期发生的，而不需要异常处理，应该当做错误来处理，这也是为什么 Go 语言采用了函数返回错误的设计。但是还有一种情况，有一些操作几乎不可能失败，而且在一些特定的情况下也没有办法返回错误，也无法继续执行，这样情况就应该`panic`。 最常见的情况就是数组越界，一般而言`panic`之后要`recover`，否则就会影响到整个系统的健壮性。

```go
func GetUser(uid int) (username string) {
    defer func() {
        if x := recover(); x != nil { // recover
            username = ""
        }
    }()

    username = User[uid] // 数组越界就会 panic
    return
}
```

在实际开发中，如果自定义的函数有可能失败，它就应该返回一个错误；而`panic`和`recover`是针对自己实现的逻辑中的一些特殊情况来设计的。 

# 部署和维护

实际上成功运行 main 函数之后整个 Web 应用就算是运行起来了。

因为 Go 程序编译之后是一个可执行文件，所以采用 daemon（守护进程，在 Windows 中就是服务）就可以完美的实现程序后台持续运行，但是目前 Go 还无法完美的实现 daemon，因此，可以利用第三方工具来部署，第三方的工具有很多，例如 Supervisord、upstart、daemontools 等。

## 一、Supervisord

### 1.Supervisord 安装

Supervisord 可以通过`sudo easy_install supervisor`安装，当然也可以通过Supervisord官网下载后解压并转到源码所在的文件夹下执行`setup.py install`来安装。

### 2.Supervisord 配置

Supervisord 默认的配置文件路径为 /etc/supervisord.conf，通过文本编辑器修改这个文件，下面是一个示例的配置文件：

```properties
;/etc/supervisord.conf

[unix_http_server]

file = /var/run/supervisord.sock

chmod = 0777

chown= root:root

 

[inet_http_server]

# Web 管理界面设定

port=9001

username = admin

password = yourpassword

 

[supervisorctl]

; 必须和'unix_http_server'里面的设定匹配

serverurl = unix:///var/run/supervisord.sock

 

[supervisord]

logfile=/var/log/supervisord/supervisord.log ; (main log file;default $CWD/supervisord.log)

logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)

logfile_backups=10          ; (num of main logfile rotation backups;default 10)

loglevel=info               ; (log level;default info; others: debug,warn,trace)

pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)

nodaemon=true              ; (start in foreground if true;default false)

minfds=1024                 ; (min. avail startup file descriptors;default 1024)

minprocs=200                ; (min. avail process descriptors;default 200)

user=root                 ; (default is current user, required if root)

childlogdir=/var/log/supervisord/            ; ('AUTO' child log dir, default $TEMP)

 

[rpcinterface:supervisor]

supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

 

; 管理的单个进程的配置，可以添加多个 program

[program:blogdemon]

command=/data/blog/blogdemon

autostart = true

startsecs = 5

user = root

redirect_stderr = true

stdout_logfile = /var/log/supervisord/blogdemon.log

```

### 3.管理

Supervisord 安装完成后有两个可用的命令行 supervisor 和 supervisorctl，命令使用解释如下：

- `supervisord`：初始启动 Supervisord，启动、管理配置中设置的进程。
- `supervisorctl stop programxxx`：停止某一个进程`programxxx`，`programxxx`为`[program:blogdemon]`里配置的值，这个示例就是`blogdemon`。
- `supervisorctl start programxxx`：启动某个进程
- `supervisorctl restart programxxx`：重启某个进程
- `supervisorctl stop all`：停止全部进程，注：start、restart、stop 都不会载入最新的配置文件。
- `supervisorctl reload`：载入最新的配置文件，并按新的配置启动、管理所有进程。

# 缓存

[go-cache](https://github.com/patrickmn/go-cache)





 



 