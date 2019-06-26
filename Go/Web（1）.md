# 工作原理

## 一、基本概念

- request：用户请求的信息，用来解析用户的请求信息，包括 cookie、URL 等信息
- response：服务器需要反馈给客户端的信息
- conn：用户的每次请求链接
- handler：处理请求和生成返回信息的处理逻辑

## 二、Go HTTP 工作模式

![HTTP 工作模式](img\HTTP 工作模式.png)

```go
func handleRequest(writer http.ResponseWriter,request *http.Request){
	request.ParseForm()
	fmt.Println(request.Method)
	fmt.Println(request.URL)
	writer.Write([]byte("hello go"))
}

func main() {
	http.HandleFunc("/",handleRequest)
	err := http.ListenAndServe(":8080",nil)
	if err != nil {
		log.Fatal("ListenAndServe:",err)
	}
}
```

Go HTTP 工作的详细流程如下图：

![HTTP 详细流程](img\HTTP 详细流程.png)

1. 调用`ListenAndServe`函数，在底层初始化一个 server 对象，然后调用`net.Listen("tcp",addr)`，监听指定端口
2. 通过 listener 接收请求，然后创建一个 conn，最后单独开了一个`goroutine`，用来处理请求，这就意味着每一个请求都用一个`goroutine`来处理，体现高并发。
3. conn 首先会解析 request,然后获取相应的 handler，也就是在调用函数`ListenAndServe`时候的第二个参数 。这个参数通常是`nil`，默认获取`handler = DefaultServeMux`，这实际上是一个路由，用来匹配 URL，并交由设置好的 handler 去处理。URL 所对应的 handler 就是在调用函数`HandleFunc`时设置好的。

## 三、ServeMux 

`http`包中定义了一个接口：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)  
}
```

所有的 handler 都必须实现这个接口，才能对请求进行响应。但是在`http`又定义了一个类型：

```go
type HandlerFunc func(ResponseWriter, *Request)
```

这就意味着所有签名为`func(ResponseWriter, *Request)`的函数都会被转成`HandlerFunc`类型，然后再调用`ServeHTTP`方法：

```go
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

默认的`ServeMux`会找到已经配置好的 handler 来执行，也可以自定义`ServeMux`：

```go
type MyMux struct {
}

func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) { // 让 MyMux 实现 Handler 接口
    if r.URL.Path == "/" {
        sayhelloName(w, r)
        return
    }
    http.NotFound(w, r)
    return
}

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello myroute!")
}

func main() {
    mux := &MyMux{}
    http.ListenAndServe(":9090", mux)
}
```

# 表单处理

## 一、获取表单数据 

```go
func handleLogin(writer http.ResponseWriter,request *http.Request) {
	if request.Method == "GET" { // 如果是 GET 方法就显示登录界面
		t, _ := template.ParseFiles("login.html")
		t.Execute(writer, nil)
	}

	if request.Method == "POST" { // 如果是 POST 方法就处理登录请求
		// ...
		request.ParseForm() // 解析表单数据后才能在控制台打印输出
		fmt.Println("username",request.Form["username"])
		fmt.Println("password",request.Form["password"])
	}
}
```

`request.Form`实际上存储了所有的请求参数。注意，`request.Form["username"]`这种代码的返回结果实际上是一个字符串数组，因为请求中可能包含有同名参数。

## 二、验证数据

Web 开发的一大原则：==不能相信用户输入的任何数据==。

可以用正则表达式进行验证，但是也不要滥用，毕竟正则表达式还是会影响效率。

### 1.验证是否为空

`request.Form`对不同表单数据的空值有不同处理，对于文本框、文本区、文件等，如果用户没有提交数据，那`Form`中就为空；对于复选框、单选按钮，如果用户没有选中，那么`Form`压根就没有相应的字段。

如果已知某个字段**只能有一个值**，那么最好用`request.Form.Get()`获取，如果为空，那么返回的就是空串：

```go
if request.Form.Get("username") != "" {
	//...
}
```

如果某个字段可能对应有多个值，那么用`request.Form[key]`的方法来获取：

```go
if request.Form["username"][0] != "" { // 第一个用户名
	//...
}
```

### 2.验证数字

用户提交的数据==都是字符串==，所以要先转换才能进行验证：

```go
num,err := strconv.ParseInt(request.Form.Get("age"),10,64) // 第二个参数是进制，第三个参数是位数
if err != nil { // 如果出错说明可能用户提交的不是数字
	//...	
} else { // 进行进一步验证，比如验证范围
	if num > 100 {
		// ....
	}
}
```

### 3.验证中英文

直接用正则表达式比较方便，验证中文：

```go
if m, _ := regexp.MatchString("^\\p{Han}+$", r.Form.Get("realname")); !m {
    return false
}
```

验证英文：

```go
if m, _ := regexp.MatchString("^[a-zA-Z]+$", r.Form.Get("engname")); !m {
    return false
}
```

### 4.验证下拉菜单

有这样一个下拉菜单：

```go
<select name="fruit">
<option value="apple">apple</option>
<option value="pear">pear</option>
<option value="banane">banane</option>
</select>
```

为了避免有人伪造数据，需要验证用户提交的值是否为预设的下拉菜单值：

```go
slice:=[]string{"apple","pear","banane"}

v := r.Form.Get("fruit")
for _, item := range slice {
    if item == v {
        return true
    }
}

return false
```

对于单选框也是如此。复选框有多个选项，当用户选中多个选项时处理起来比较麻烦，但是大同小异。

### 5.验证日期时间

可以将用户提交的年、月、日转换成相应的日期，然后再进行进一步验证：

```go
t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
```

### 6.验证电子邮箱、手机号码、身份证号

这些用正则表达式去验证。

## 三、预防跨站脚本

一是验证所有输入数据，有效检测攻击;另一个是对所有输出数据进行适当的处理，以防止任何已成功注入的脚本在浏览器端运行。 

预防跨站脚本的一个方法是过滤 HTML。Go 提供了几个函数，将一些内容进行转义后再输出：

- `func HTMLEscape(w io.Writer, b []byte)` //把参数 b 进行转义之后写到 w
- `func HTMLEscapeString(s string) string` //转义参数 s 之后返回结果字符串
- `func HTMLEscaper(args ...interface{}) string` //支持多个参数一起转义，返回结果字符串

比如：

```go
template.HTMLEscape(w, []byte("<script>alert()</script>")) //输出到客户端
```

那么客户端看到的是：

```html
&lt;script&gt;alert()&llt;/script&gt;
```

## 四、防止重复提交

解决方案是在表单中添加一个带有唯一值的隐藏字段。在验证表单时，先检查带有该唯一值的表单是否已经递交过了。如果是，拒绝再次递交；如果不是，则处理表单进行逻辑处理。另外，如果是采用了 Ajax 模式递交表单的话，当表单递交后，通过 JavaScript 来禁用表单的递交按钮。 

一个基本的解决方案：

```go
用户名:<input type="text" name="username">
密码:<input type="password" name="password">
<input type="hidden" name="token" value="{{.}}">
<input type="submit" value="登陆">
```

模版里面增加了一个隐藏字段`token`，通过 MD5(时间戳)来获取唯一值，然后把这个值存储到服务器端 session 中，以方便表单提交时比对判定。 

```go
func handleLogin(writer http.ResponseWriter,request *http.Request) {
	if request.Method == "GET" { // 如果是 GET 方法就显示登录界面
		curtime := time.Now().Unix()
		h := md5.New()
		io.WriteString(h,strconv.FormatInt(curtime,10))
		token := fmt.Sprintf("%x",h.Sum(nil)) // 生成 token
		t, _ := template.ParseFiles("login.html")
		t.Execute(writer, token) // 将 token 写入页面
	}
	if request.Method == "POST" { // 如果是 POST 方法就处理登录请求
		request.ParseForm()
		token := request.Form.Get("token")
		if token != "" {
			// 验证，即查看在 session 中是否已经有这个 token
		} else { // token 不存在则报错
			log.Fatal("Token not exists!")
		}
		// 验证数据，进行处理...
	}
}
```

## 五、上传文件

服务器端处理客户端通过表单上传的文件：

```go
func handleUploadFile(writer http.ResponseWriter,request *http.Request){
	if request.Method == "GET" {
		curtime := time.Now().Unix()
		h := md5.New()
		io.WriteString(h,strconv.FormatInt(curtime,10))
		token := fmt.Sprintf("%x",h.Sum(nil)) // 生成 token
		t, _ := template.ParseFiles("upload.html")
		t.Execute(writer, token) // 将 token 写入页面
	}
	if request.Method == "POST" {
		request.ParseMultipartForm(32 << 20) // 设置最大内存，如果上传的文件大小超过了这个值，那么剩下的部分存在系统临时文件中
		 									             // 注意调用这个函数之后不需要调用 ParseForm，因为在需要的时候Go自动会去调用；而且这个函数只需调用一次
		file,handler,err := request.FormFile("uploadfile") // handler 包含有文件名、MIME 等信息
		if err != nil {
			fmt.Println(err)
			return
		}
		defer file.Close() // 记得关闭文件
		fmt.Fprintf(writer,"%v",handler.Header)
		f,err := os.OpenFile("G:/test/" + handler.Filename,os.O_WRONLY|os.O_CREATE, 0666)
		if err != nil {
			fmt.Println(err)
			return
		}
		defer f.Close()
		io.Copy(f,file) // 保存文件
	}
}
```

关键步骤：

1. 调用`ParseMultipartForm`设置最大内存
2. 调用`FormFile`获取文件、handler
3. 进行处理

# 数据库

Go 官方不提供某个具体数据库的实现，只提供了用来开发数据库驱动的接口。

## 一、基本接口

`database/sql`包中提供了一组接口，如果是面向这些接口进行开发数据库驱动，那么在更换数据库时就无需做大的改动。

### 1.sql.Register

这个函数用来注册一个数据库驱动，可以同时注册多个数据库驱动，只要不重复：

```go
Register(name string, driver driver.Driver)
```

在已经实现好的数据库驱动中，通常是在`init`函数中调用这个函数。因此经常会看到这样的代码：

```go
   import (
       "database/sql"
        _ "github.com/mattn/go-sqlite3"
   )
```

导入了包但不直接使用，因为已经通过`init`函数注册了驱动。

### 2.driver.Driver

```go
type Driver interface {
    Open(name string) (Conn, error)
}
```

`Open`函数返回一个数据库连接，注意一个连接只能用于一个`goroutine`。

### 3.driver.Conn

```go
type Conn interface {
    Prepare(query string) (Stmt, error)
    Close() error
    Begin() (Tx, error)
}
```

- `Prepare`函数返回与当前连接相关的执行 SQL 语句的准备状态，用返回的`Stmt`对象来对数据库进行操作。
- `Close`函数关闭当前的连接，执行释放连接拥有的资源等清理工作。
- `Begin`函数返回`Tx`代表事务处理。

 ### 4.driver.Stmt

`Stmt`是一种准备好的状态，和`Conn`相关联，而且只能应用于一个`goroutine`中，不能应用于多个`goroutine`。

```go
type Stmt interface {
    Close() error
    NumInput() int
    Exec(args []Value) (Result, error)
    Query(args []Value) (Rows, error)
}
```

- `Close`函数关闭当前的链接状态，但是如果当前正在执行查询，那么也还是会返回结果集。
- `NumInput`函数返回当前预留参数的个数，当返回值大于等于0时数据库驱动就会智能检查调用者的参数。当数据库驱动包不知道预留参数的时候，返回-1。
- `Exec`函数执行`Prepare`函数准备好的 SQL，传入参数执行 update、insert 等操作，返回`Result`数据
- `Query`函数执行`Prepare`函数准备好的 SQL，传入需要的参数执行 select 操作，返回`Rows`结果集。

### 5.driver.Tx

用来进行事务的提交、回滚。

```go
type Tx interface {
    Commit() error
    Rollback() error
}
```

### 6.driver.Result

```go
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```

这个接口用来代表结果。

- `LastInsertId`函数返回由数据库执行插入操作得到的自增ID号。
- `RowsAffected`函数返回操作影响的数据条目数。

### 7.driver.Rows

这个接口用来代表结果集。

```go
type Rows interface {
    Columns() []string
    Close() error
    Next(dest []Value) error
}
```

- `Columns`函数返回查询数据库表的字段信息，注意返回的是 SQL 语句中的字段，而不是整张表的字段。
- `Close`函数用来关闭迭代器。
- `Next`函数用来返回下一条数据。`dest`里面的元素必须是`driver.Value`的值。除了`string`，返回的数据里面所有的`string`都必须要转换成`[]byte`。如果最后没数据了，`Next`函数最后返回`io.EOF`。

### 8.driver.Value

```go
type Value interface{}
```

这个空接口用来容纳数据库驱动所能操作的数据类型。

 ### 9.driver.ValueConverter

```go
type ValueConverter interface {
    ConvertValue(v interface{}) (Value, error)
}
```

程序和数据库之间通过驱动进行交互时通常要进行数据类型的转换，这个接口就用来定义转换的规则。

### 10.driver.Valuer

```go
type Valuer interface {
    Value() (Value, error)
}
```

很多类型都实现了这个接口，用来转换成`driver.Value`。

## 二、MySQL 

目前使用得较多的一个 MySQL 驱动是 [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)。使用时注意导入包：

```go
import _ "go-sql-driver/mysql"
```

### 1.建立连接

```go
db,err := sql.Open("mysql","root:loveisKING@/studytest")
if err != nil {
	panic(err)
}
```

### 2.查询

不带参数的查询：

```go
rows,err := db.Query("SELECT id,name,salary FROM user") // 执行
if err != nil {
	panic(err)
}
defer rows.Close()

var id int
var name string
var salary float64
// 遍历结果集，否则 rows 永远不会释放
for rows.Next() { 
	err = rows.Scan(&id,&name,&salary) // 将结果存入变量
	if err != nil {
		panic(err)
	}
	fmt.Print("id:",id," ")
	fmt.Print("name:",name," ")
	fmt.Println("salary:",salary)
	fmt.Println("---------------------------")
}
```

有参数的查询：

```go
stmt,err := db.Prepare("SELECT * FROM user WHERE id=?")
if err != nil {
    panic(err)
}
rows,err := stmt.Query("1")
if err != nil {
    panic(err)
}
```

### 3.插入

```go
func insert(db *sql.DB,sql string,params map[string]string) int64{
	stmt,err := db.Prepare(sql)   // 准备 SQL 语句
	if err != nil {
		panic(err)
	}

	res,err := stmt.Exec(params["name"],params["salary"]) // 根据传入的参数执行，注意 Exec 的参数全是 string
	if err != nil {
		panic(err)
	}
	rows,err :=  res.RowsAffected()
	if err != nil {
		panic(err)
	}
	return rows
}
```

调用：

```go
params := map[string]interface{} {
	"name":"haah",
	"salary":"1000.0",
}
insert(db,"INSERT INTO user(name,salary) VALUES(?,?)",params)
```

### 4.更新、删除

与插入大体相同，只是 SQL 语句不用。

### 5.关闭连接

```go
db.Close()
```

# Cookie 和 Session

## 一、Cookie

Go 提供了用来表示 Cookie 的 struct：

```go
type Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string

// MaxAge=0 means no 'Max-Age' attribute specified.
// MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
// MaxAge>0 means Max-Age attribute present and given in seconds
    MaxAge   int
    Secure   bool
    HttpOnly bool
    Raw      string
    Unparsed []string // Raw text of unparsed attribute-value pairs
}
```

通过`http.SetCookie(w ResponseWriter, cookie *Cookie)`函数来设置 Cookie：

```go
expiration := time.Now()
expiration = expiration.AddDate(1, 0, 0)
cookie := http.Cookie{Name: "username", Value: "astaxie", Expires: expiration}
http.SetCookie(w, &cookie)
```

通过 request 来读取 Cookie：

```go
// 指定读取某一个 Cookie
cookie, _ := request.Cookie("username")

// 读取多个 Cookie
for _, cookie := range request.Cookies() {
    //...
}
```

## 二、Session

###  1.基本原理

- 生成全局唯一标识符`sessionid`
- 开辟数据存储空间。一般会在内存中创建相应的数据结构，但这种情况下，系统一旦掉电，所有的会话数据就会丢失，如果是电子商务类网站，这将造成严重的后果。所以为了解决这类问题，你可以将会话数据写到文件里或存储在数据库中，当然这样会增加`I/O`开销，但是它可以实现某种程度的`session`持久化，也更有利于`session`的共享；
- 将`session`的全局唯一标示符发送给客户端。一般是通过 Cookie 发送给客户端，如果浏览器禁用 Cookie，那么就将`sessionid`写在 URL 之后。

### 2.基本操作

Go 没有提供官方的 Session API，可以使用第三方实现 [github.com/gorilla/sessions ](https://github.com/gorilla/sessions)，[文档](https://godoc.org/github.com/gorilla/sessions)。

```go
var store = sessions.NewCookieStore([]byte("something-very-secret")) // 用来存储所有的 session，传入的参数是一个用来认证的密钥

func sessionHandler(w http.ResponseWriter, r *http.Request){
	// Get 函数用来获取一个 session。如果不存在那么就创建一个
	session, err := store.Get(r, "session-name")
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	// 设置 session 的值
    // Values 这个字段的类型是 map[interface{}]interface{}
	session.Values["foo"] = "bar"
	session.Values[42] = 43
	// 在将 session 发送给客户端之前必须先调用 Save
	session.Save(r, w)
}
```

==细节==：

（1）如果使用了多个 Session，那么可以直接调用`sessions.Save`，而无需对每个 Session 调用`Save`。

（2）可以更改 Session 的配置，比如有效时间等等。

```go
session.Options = &sessions.Options{
	Path:     "/",
	MaxAge:   86400 * 7,
	HttpOnly: true,
}
```

这种设置方法对单个 Session 有效，如果想进行全局性的配置，那么就通过 store 进行配置：

```go
store.Options = &sessions.Options{
	Path:     "/",
	MaxAge:   86400 * 7,
	HttpOnly: true,	
}
```

（3）如果要在 Session 中存储结构体之类的复杂类型，首先需要在`gob`中进行注册，以便序列化/反序列化：

```go
type Person struct {
	FirstName	string
	LastName 	string
	Email		string
	Age			int
}

func init() {
	gob.Register(&Person{}) // 这里传递了指针类型
}
```

注意从 Session 中获取值的时候需要进行类型断言：

```go
val := session.Values["person"]
var person = &Person{}
if person, ok := val.(*Person); !ok {
	// 如果不是 Person 类型
}
```

### 3.预防 Session 劫持

可以通过伪造一个一模一样的 Session 来欺骗服务器，这叫做 Session 劫持。防止 Session 劫持的基本方法有两种：

（1）设置 Cookie 的 httponly 属性为 true，防止这个 Cookie 被 XSS 读取；然后在每个请求中都带上一个隐藏的 token，用来验证请求的唯一性。

（2）一旦超过一个指定的时间就销毁 Session，重新创建一个。

# 数据处理

## 一、XML 

### 1.解析 XML

XML 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<books> 
  <book id="001"> 
     <title>Harry Potter</title> 
     <author>J K. Rowling</author> 
  </book> 
  <book id="002"> 
     <title>Learning XML</title> 
     <author>Erik T. Ray</author> 
  </book> 
</books>
```

一般是通过 `func Unmarshal(data []byte, v interface{}) error`函数来解析 XML，`data`是输入的 XML，`v`是输出的类型。

通常情况下是将 XML 输出为结构体，此时要求结构体的每个字段都是可导出的。解析 XML 时最重要的就是结构体的设计：

```go
type Recurlybooks struct { // 根元素
	XMLName xml.Name `xml:"books"`   // 代表根元素 <books>
	Books []Book  `xml:"book"` // 代表子元素 <book>
	Description string `xml:",innerxml"` // 代表 XML 文档根标签下的所有子标签
}

type Book struct { // <book> 元素
	XMLName xml.Name `xml:"book"` // 代表元素 <book>
	Id string `xml:"id,attr"`     // 代表属性 id
	Title string  `xml:"title"` // 代表子元素 <title>
	Author string `xml:"author"` // 代表子元素 <author>.
}
```

每个字段后面多了一些用``包围的内容，叫做 struct tag，用来辅助反射，简单的说就是指定 XML 文档的哪些内容绑定到哪些字段上。

解析文件：

```go
func readXML(fileName string){
	file,err := os.Open(fileName)
	if err != nil {
		fmt.Println("error:",err)
		return
	}

	defer file.Close()

	data,err := ioutil.ReadAll(file)
	if err != nil {
		fmt.Println("error:",err)
		return
	}
	v := Recurlybooks{}
	err = xml.Unmarshal(data,&v)  // 将读取到的内容保存到 v 中
	if err != nil {
		fmt.Println("error:",err)
		return
	}

	fmt.Println(v)
}
```

### 2.详解 struct tag

- 如果`struct`中有一个字段`XMLName`，且类型为`xml.Name`，那么在解析的时候就会保存这个元素的名字到该字段。
- 如果某个`struct`字段的 tag 定义中含有 XML 结构中元素的名称，那么解析的时候就会把相应的元素值赋值给该字段，如上面的``xml:"title"``。
- 如果某个`struct`字段的 tag 定义了中含有`,attr`，那么解析的时候就会将该结构所对应的元素的响应属性的值赋值给该字段。

- 如果某个`struct`字段的 tag 定义为`"a>b>c"`，那么会将 a 下面的 b 下面的 c 元素的值赋值给该字段。
- 如果某个`struct`字段的 tag 定义为`"-"`,那么不会为该字段解析匹配数据。
- 如果`struct`字段后面的 tag 定义为`",any"`，他的子元素在不满足其他的规则的时候就会匹配到这个字段。
- 如果某个 XML 元素包含一条或者多条注释，那么这些注释将被累加到第一个 tag 含有`",comments"`的字段上，这个字段的类型可能是`[]byte`或`string`,如果没有这样的字段存在，那么注释将会被丢弃。

## 二、JSON

### 1.解析 JSON

解析 JSON 用到的是 `encoding/json`包中的`func Unmarshal(data []byte, v interface{}) error`函数。

JSON 数据：

```json
{
  "books":[{
    "title":"'Harry Potter",
    "author":"J K. Rowling"
  },
    {
      "title":"The Heaven Sword and Dragon Saber",
      "author":"Jin Yong"
    }]
}
```

（1）将数据读取到结构体中

设计与 JSON 相匹配的结构体：

```go
type BookJson struct {
	Title string
	Author string
}

type BooksJson struct {
	Books []BookJson
}
```

结构体的设计原则与 XML 大体相似，可以添加相应的`struct tag`。如果不加`struct tag`，那么只有可导出的字段可以和 JSON 中的属性进行匹配。

（2）将数据读取到接口中

如果事先不知道 JSON 的结构，那么无法将数据读取到结构体中。这时就用空接口来存储 JSON 数据：

```go
var f interface{}
err := json.Unmarshal(data, &f)
```

此时`f`存储了一个`map`：

```go
f = map[string]interface{}{
    "books": []interface{} {
        map[string]interface{}{
            "title":"Harry Potter",
            "author":"J K. Rowling",
        },
        map[string]interface{}{
            "title":"The Heaven Sword and Dragon Saber",
            "author":"Jin Yong",           
        },
    },
}
```

如果想要访问其中的数据，就需要使用类型断言。Go 类型和 JSON 类型的对应关系为：

- `bool` 代表 JSON `booleans`
- `float64` 代表 JSON `numbers`
- `string` 代表 JSON `strings`
- `nil` 代表 JSON `null`

```go
for k, v := range m {
    switch vv := v.(type) {
    case string:
        fmt.Println(k, "is string", vv)
    case int:
        fmt.Println(k, "is int", vv)
    case float64:
        fmt.Println(k,"is float64",vv)
    case []interface{}:
        fmt.Println(k, "is an array:")
        for i, u := range vv {
            fmt.Println(i, u)
        }
    default:
        fmt.Println(k, "is of a type I don't know how to handle")
    }
}
```

使用类型断言比较麻烦，可以使用第三方包 [simplejson](https://github.com/bitly/go-simplejson)。

```go
js, err := NewJson([]byte(`{
    "test": {
        "array": [1, "2", 3],
        "int": 10,
        "float": 5.150,
        "bignum": 9223372036854775807,
        "string": "simplejson",
        "bool": true
    }
}`))

arr, _ := js.Get("test").Get("array").Array()
i, _ := js.Get("test").Get("int").Int()
ms := js.Get("test").Get("string").MustString()
```

### 2.生成 JSON

通过函数`func Marshal(v interface{}) ([]byte, error)`来生成 JSON。

```go
func  CreateJson(){
	var books BooksJson
	book1 := BookJson{Title:"Harry Potter",Author:"J K. Rowling"}
	book2 := BookJson{Title:"The Heaven Sword and Dragon Saber",Author:"Jin Yong"}
	books.Books = append(books.Books,book1,book2)

	jsonData,err := json.Marshal(books)
	if err != nil {
		fmt.Println(err)
		return
	}

	var jsonResult BooksJson
	err = json.Unmarshal(jsonData,&jsonResult)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(string(jsonData))
}
```

可以在输出结果中看到所有的属性名都是大写，也就是和结构体中定义的一样。如果想变成小写，或是有更多的需求，就通过`struct tag`来实现。

用`Marshal`生成 JSON 时，需要注意：

- 结构体中首字母小写的字段不会导出，也就是说不会出现在 JSON 中

- JSON 对象只支持`string`作为属性名，所以要编码一个`map`，那么必须是`map[string]T`这种类型(`T`是 Go 语言中任意的类型)
- **`channel`, `complex`和函数不能被编码成 JSON **
- 嵌套的数据是不能编码的，不然会让 JSON 编码进入死循环
- **指针在编码的时候会输出指针指向的内容，**而空指针会输出`null`

### 3.详解 struct tag

- 字段的 tag 是`"-"`，那么这个字段不会输出到 JSON
- tag 中带有自定义名称，那么这个自定义名称会作为 JSON 的属性名
- tag 中如果带有`"omitempty"`，那么如果该字段值为空，就不会输出到JSON串中
- 如果字段类型是`bool`, `string`, `int`, `int64`等，而 tag 中带有`",string"`选项，那么这个字段在输出到 JSON 的时候会把该字段对应的值转换成 JSON 字符串
- 如果 tag 中有多个 key，那么用*空格*分开

```go
type User struct {
    Username string 'json:"username" gorm:"column:username"'
    Password string 'json:"-"'
    Email string 'json:"email,omitemppty"'
    Secretary bool 'json:"string"'
}
```

# Socket

Unix 认为“一切皆文件”，用“打开 open –> 读写 write/read –> 关闭 close”模式来完成一切操作。 Socket 就是该模式的一个实现，网络的 Socket 数据传输是一种特殊的 I/O，Socket 也是一种文件描述符。 

常用的Socket类型有两种：

- 流式 Socket（SOCK_STREAM）：流式是一种面向连接的 Socket，针对于面向连接的 TCP 服务应用。
- 数据报式 Socket（SOCK_DGRAM）：数据报式 Socket 是一种无连接的 Socket，对应于无连接的 UDP 服务应用。

 使用 TCP/IP 协议的应用程序通常采用应用编程接口—— UNIX BSD 的套接字（socket）来实现网络进程之间的通信。 

## 一、IP

Go 提供的 IP 类型定义为：

```go
type IP []byte
```

`net`包提供了函数用于将一个字符串转为 IP：

```go
func ParseIP(s string) IP
```

## 二、TCP Socket

在 Go 语言的`net`包中有一个类型`TCPConn`，这个类型可以用来作为客户端和服务器端交互的通道，它有两个主要的函数：

```go
func (c *TCPConn) Write(b []byte) (n int, err os.Error)
func (c *TCPConn) Read(b []byte) (n int, err os.Error)
```

`TCPConn`可以用在客户端和服务器端来读写数据。

### 1.客户端

客户端通过函数`DialTCP`建立与服务器的 TCP 连接：

```go
func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err os.Error)
```

参数含义：

- `net`参数是`"tcp4"`、`"tcp6"`、`"tcp"`中的任意一个，分别表示 TCP(IPv4-only)、TCP(IPv6-only) 或者 TCP(IPv4,IPv6的 任意一个)
- `laddr`表示本机地址，一般设置为`nil
- `raddr`表示远程的服务地址

客户端示例代码：

```go
	tcpAddr,err := net.ResolveTCPAddr("tcp4","localhost")
	if err != nil {
		fmt.Println(err)
		return
	}
	conn,err := net.DialTCP("tcp",nil,tcpAddr) // 获取 IP 地址
	if err != nil {
		fmt.Println(err)
		return
	}
	_,err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n")) // 发送请求
	if err != nil {
		fmt.Println(err)
		return
	}
	result,err := ioutil.ReadAll(conn) // 读取请求
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(string(result))
```

### 2.服务器

服务器端需要监听指定端口，当有客户端请求到达时就需要进行响应。服务器端主要用到两个函数：

```go
func ListenTCP(net string, laddr *TCPAddr) (l *TCPListener, err os.Error)  // 监听端口是否有客户端请求送达
func (l *TCPListener) Accept() (c Conn, err os.Error) // 接受请求
```

服务器端代码示例：

```go
func server() {
	service := ":8000"
	tcpAddr, err := net.ResolveTCPAddr("tcp4", service) // 指定端口
	if err != nil {
		fmt.Println(err)
		return
	}
	listener, err := net.ListenTCP("tcp", tcpAddr) // 监听端口
	if err != nil {
		fmt.Println(err)
		return
	}
	for {
		conn, err := listener.Accept() // 接受请求
		if err != nil {
			fmt.Println(err)
			continue // 这里不是 return，因为在实际开发中最好不要影响到整个服务器端的运行
		}
		go handleClient(conn) // 提高并发性
	}
}
func handleClient(conn net.Conn) {
	defer conn.Close()                                    // 记得关闭连接
	conn.SetReadDeadline(time.Now().Add(2 * time.Minute)) // 设置读取的最长持续时间，超过这个时间连接自动断开。如果是长连接就要特别注意这个设置
	request := make([]byte, 128) // 指定一个最大长度以防止 flood attack
	for {
		read_len, err := conn.Read(request)
		if err != nil {
			fmt.Println(err)
			break
		}

		if read_len == 0 {
			break   // 客户端已经关闭连接
		} else if strings.TrimSpace(string(request[:read_len])) == "timestamp"{
			daytime := strconv.FormatInt(time.Now().Unix(),10)
			conn.Write([]byte(daytime))
		} else {
			daytime := time.Now().String()
			conn.Write([]byte(daytime))
		}
		request = make([]byte,128) // 清空内容
	}

}
```

### 3.控制 TCP 连接

（1）设置建立连接的超时时间（客户端和服务器端都适用），当超过设置时间时，连接自动关闭。 

```go
func DialTimeout(net, addr string, timeout time.Duration) (Conn, error)
```

（2）设置写入/读取一个连接的超时时间。当超过设置时间时，连接自动关闭。 

```go
func (c *TCPConn) SetReadDeadline(t time.Time) error
func (c *TCPConn) SetWriteDeadline(t time.Time) error
```

（3）设置`keepAlive`属性，是操作系统层在 TCP 上没有数据和 ACK 的时候，会间隔性的发送`keepalive`包，操作系统可以通过该包来判断一个 TCP 连接是否已经断开。

```go
func (c *TCPConn) SetKeepAlive(keepalive bool) os.Error
```

## 三、UDP Socket

UDP 缺少了对客户端连接请求的 Accept 函数。其他与 TCP 基本几乎一模一样。

```go
func ResolveUDPAddr(net, addr string) (*UDPAddr, os.Error)
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err os.Error)
func ListenUDP(net string, laddr *UDPAddr) (c *UDPConn, err os.Error)
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err os.Error)
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (n int, err os.Error)
```

## 四、WebSocket

WebSocket 是 HTML5 的重要特性，它实现了基于浏览器的远程 socket，它使浏览器和服务器可以进行全双工通信，许多浏览器（Firefox、Google Chrome 和 Safari）都已对此做了支持。WebSocket 采用了一些特殊的报头，使得浏览器和服务器只需要做一个握手的动作，就可以在浏览器和服务器之间建立一条连接通道。它解决了 Web 实时化的问题，相比传统 HTTP 有如下好处：

- 一个 Web 客户端只建立一个 TCP 连接
- Websocket 服务端可以主动推送(push)数据到 web 客户端.
- 有更加轻量级的头，减少数据传送量

### 1.原理

WebSocket URL的起始输入是`ws://`或是`wss://`（在 SSL 上）。WebSocket 的协议颇为简单，在第一次握手通过以后，连接便建立成功，其后的通讯数据都是以`”\x00″`开头，以”`\xFF”`结尾。在客户端，这个是透明的，WebSocket 组件会自动将原始数据“掐头去尾”。

### 2.服务器端示例代码

Go 本身没有提供 WebSocket 的相关 API，但是在由官方维护的go.net子包中有对这个的支持。

```go
func Echo(ws *websocket.Conn) {
    var err error

    for {
        var reply string

        if err = websocket.Message.Receive(ws, &reply); err != nil {
            fmt.Println("Can't receive")  // 接受客户端请求
            break
        }

        fmt.Println("Received back from client: " + reply)

        msg := "Received:  " + reply
        fmt.Println("Sending to client: " + msg)

        if err = websocket.Message.Send(ws, msg); err != nil { // 响应
            fmt.Println("Can't send")
            break
        }
    }
}

func main() {
    http.Handle("/", websocket.Handler(Echo))

    if err := http.ListenAndServe(":1234", nil); err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}
```

# REST

实现 RESTful API 的方法其实有多种，有一个第三方库 [httprouter](https://github.com/julienschmidt/httprouter) 提供了 RESTful API。

```go
func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func getuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    uid := ps.ByName("uid")
    fmt.Fprintf(w, "you are get user %s", uid)
}

func modifyuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    uid := ps.ByName("uid")
    fmt.Fprintf(w, "you are modify user %s", uid)
}

func deleteuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    uid := ps.ByName("uid")
    fmt.Fprintf(w, "you are delete user %s", uid)
}

func adduser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    // uid := r.FormValue("uid")
    uid := ps.ByName("uid")
    fmt.Fprintf(w, "you are add user %s", uid)
}

func main() {
    router := httprouter.New()
    router.GET("/", Index)
    router.GET("/hello/:name", Hello)

    router.GET("/user/:uid", getuser)
    router.POST("/adduser/:uid", adduser)
    router.DELETE("/deluser/:uid", deleteuser)
    router.PUT("/moduser/:uid", modifyuser)

    log.Fatal(http.ListenAndServe(":8080", router))
}
```

## httprouter 详解

### 一、路由冲突

当两个路由的 HTTP 方法一致、路径前缀一致、且在某个位置 A 路由是参数而 B 路由是普通字符串时，就会产生路由冲突，进而在==初始化阶段==就`panic`。比如在下面的例子中，如果第二个路由的 id 参数为 info，httprouter 就无法处理。

```go
GET /user/info/:name
GET /user/:id
```

## 二、原理

httprouter 在内部使用压缩字典树这种数据结构来存储众多的路由的：

![压缩字典树](img\压缩字典树.png)

压缩字典树是字典树的变种，差别在于字典树的结点存储的是单个字符，而压缩字典树的结点存储的是字符串。

httprouter 为每种 HTTP 方法都创建一棵树，也就是说 GET 和 POST 方法的路由之间是不共享数据的。

# RPC

RPC（Remote Procedure Call Protocol）——远程过程调用协议，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。它假定某些传输协议的存在，如 TCP 或 UDP，以便为通信程序之间携带信息数据。通过它可以使函数调用模式网络化。在 OSI 网络通信模型中，RPC 跨越了传输层和应用层。RPC 使得开发包括网络分布式多程序在内的应用程序更加容易。 

RPC 实现函数调用模式的网络化。客户端就像调用本地函数一样，把参数打包之后通过网络传递到服务端，服务端解包到处理过程中执行，然后执行的结果反馈给客户端。 

![rpc](img\rpc.png)

Go 标准包支持三个级别的 RPC：TCP、HTTP、JSONRPC。但 Go RPC 和传统的 RPC 系统不同，它只支持 Go 开发的服务器与客户端之间的交互，因为在内部它们采用了 Gob 来编码。

Go RPC 的函数只有符合下面的条件才能被远程访问，不然会被忽略，详细的要求如下：

- 函数必须是导出的(首字母大写)
- 必须有两个导出类型的参数，
- 第一个参数是接收的参数，第二个参数是返回给客户端的参数，第二个参数必须是指针类型的
- 函数还要有一个返回值 error

## 一、HTTP RPC

服务器端示例代码：

```go
type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
    *reply = args.A * args.B
    return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
    if args.B == 0 {
        return errors.New("divide by zero")
    }
    quo.Quo = args.A / args.B
    quo.Rem = args.A % args.B
    return nil
}

func main() {

    arith := new(Arith)
    rpc.Register(arith)  // 注册 RPC 服务
    rpc.HandleHTTP()     // 使用 HTTP 

    err := http.ListenAndServe(":1234", nil)
    if err != nil {
        fmt.Println(err.Error())
    }
}
```

客户端示例代码：

```go
type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

func main() {
    if len(os.Args) != 2 {
        fmt.Println("Usage: ", os.Args[0], "server")
        os.Exit(1)
    }
    serverAddress := os.Args[1]

    client, err := rpc.DialHTTP("tcp", serverAddress+":1234")
    if err != nil {
        log.Fatal("dialing:", err)
    }
    // 异步调用
    args := Args{17, 8}
    var reply int
    err = client.Call("Arith.Multiply", args, &reply)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

    var quot Quotient
    err = client.Call("Arith.Divide", args, &quot)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

}
```

客户端使用`Call`函数来实现远程调用，第1个参数是要调用的函数的名字，第2个是要传递的参数，第3个是要返回的参数(注意是指针类型)。

## 二、TCP RPC

与 HTTP RPC 大同小异，只不过是使用的协议不同。

## 三、JSON RPC

JSON RPC 是数据编码采用了 JSON，而不是 gob 编码， 除此之外没有什么不同。服务器端使用`ServeConn`函数来处理请求。