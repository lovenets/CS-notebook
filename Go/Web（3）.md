# middleware 中间件

不要把业务逻辑和非业务逻辑代码混在一起，对于大多数场景来说，非业务的需求都是在 HTTP 请求处理之前之后做一些事情（打印日志等），在实际开发中可以用中间件把这些非业务代码剥离出去。

## 原理

```go
func Hello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "hello world")
}

func main() {
	http.HandleFunc("/", Hello)
	http.ListenAndServe(":8080", nil)
}
```

现在要求在执行`Hello`之前要先打印日志，执行完毕之后输出执行时间，这些都是与核心业务逻辑无关的需求，可以交给中间件完成。

### 1.使用中间件

```go
type Middleware func(http.HandlerFunc) http.HandlerFunc

// Logging logs all requests with its path and the time it took to process
func Logging() Middleware {

	// Create a new Middleware
	return func(f http.HandlerFunc) http.HandlerFunc {

		// Define the http.HandlerFunc
		return func(w http.ResponseWriter, r *http.Request) {

			// Do middleware things
			start := time.Now()
			defer func() { log.Println(r.URL.Path, time.Since(start)) }()

			// Call the next middleware/handler in chain
			f(w, r)
		}
	}
}

// Chain applies middlewares to a http.HandlerFunc
func Chain(f http.HandlerFunc, middlewares ...Middleware) http.HandlerFunc {
	for _, m := range middlewares {
		f = m(f)
	}
	return f
}

func Hello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "hello world")
	print("hello")
}

func main() {
	http.HandleFunc("/", Chain(Hello,Logging()))
	http.ListenAndServe(":8080", nil)
}
```

当执行`http.HandleFunc("/", Chain(Hello,Logging()))`这行代码时，程序首先调用`Logging`创建一个中间件，然后进入到`Chain`的调用过程中。调用`Chain`时候，通过 debug 跟踪程序可知，一开始 f 引用的是`Hello`，执行完循环体之后 f 就变成了引用`Logging`，然后`Chain`返回，包装的过程结束，开始正式响应请求。

 仔细看`Logging`的代码可以发现，`f(w, r)`这句实际上就是执行`Hello`，因为在执行`Chain`中的`f = m(f)`这句代码时就是把`Hello`作为参数传给了中间件——中间件的定义本来就是一个函数。因此这就实现了把中间件的功能和`Hello`的功能进行整合。

### 2.详解

在上面的代码中，中间件结构体的定义用到了`http.handlerFunc`：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

// HandlerFunc 是接口 Handler 的实现
type HandlerFunc func(ResponseWriter, *Request)
// ServeHTTP 函数才是真正地响应请求
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
     f(w, r)
}
```

再看`http.handleFunc`的内部执行过程：

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}

// 调用

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	mux.Handle(pattern, HandlerFunc(handler))
}
```

也就是说，我们把业务逻辑实现在一个 handler 中，这个 handler 的函数签名是`func(ResponseWriter, *Request)`，将其传给`HnaldeFunc`；`HandleFunc`在内部将我们自定义的 handler 转换成 `HandlerFunc`类型，因为只有这个类型才能调用`ServeHTTP`来响应请求。

因此，中间件的实现概括起来也就是把 handler （执行核心业务逻辑）和其他非核心业务逻辑代码包装成一个`HandlerFunc`。

现在在实际开发中可以不用自己实现中间件，因为一些流行的框架已经有成熟的中间件方案，比如 gin。

# validator 参数校验

把所有的请求参数封装为`struct`，使用 [validator](https://github.com/go-playground/validator) 这个库，通过 json tag 定义校验条件。

```go
type User struct {
	FirstName      string     `validate:"required"`
	LastName       string     `validate:"required"`
	Age            uint8      `validate:"gte=0,lte=130"`
	Email          string     `validate:"required,email"`
	FavouriteColor string     `validate:"iscolor"`                // alias for 'hexcolor|rgb|rgba|hsl|hsla'
	Addresses      []*Address `validate:"required,dive,required"` // a person can have a home and cottage...
}
```

这个库的实现大量使用了`reflect`，而 Go 的反射机制的性能并不是很出众，当然在实际开发中这也不一定就会造成程序的性能瓶颈。

# DB

目前流行的 Go 数据库框架：https://my.oschina.net/u/168737/blog/1531834

## 一、ORM

ORM 的最大好处就是省去了手写 SQL 和将查询结果封装为对象的工作，但 ORM 往往也会隐藏过多的执行细节，这对大型系统来说可能是很危险的。比如有些 ORM 会偷偷执行 join 操作。



## 二、SQL builder

SQL builder 可以看作 SQL 在代码中的一种 dialect，可以让程序员不用手写所有的 SQL，用起来大概像这样：

```go
where := map[string]interface{} {
    "order_id > ?" : 0,
    "customer_id != ?" : 0,
}
limit := []int{0,100}
orderBy := []string{"id asc", "create_time desc"}

orders := orderModel.GetList(where, limit, orderBy)
```

相比 ORM，SQL builder 在 SQL 和项目可维护性之间取得了较好的平衡，因为它并没有封装过多的细节，而且经过简单的封装之后用起来也会更为方便。

## 三、总结

不管是 ORM 还是 SQL builder 都没有办法在系统上线之前进行 SQL 审核。比如下面的代码：

```go
where := map[string]interface{} {
    "product_id = ?" : 10,
    "user_id = ?" : 1232 ,
}

if order_id != 0 {
    where["order_id = ?"] = order_id
}

res, err := historyModel.GetList(where, limit, orderBy)
```

如果包含了很多的判断，那么在测试的时候可能就没法完全覆盖，这就给系统埋下隐患。

现在业界的一种做法是把核心业务代码中的 SQL 放在显眼的位置，交给 DBA 进行审查。

# ratelimit 限流

不管服务瓶颈在哪（I/O、CPU等），最终都要通过限流来提高性能。

## 一、限流算法

流量限制的手段有很多，最常见的：漏桶、令牌桶两种：

1. 漏桶是指我们有一个一直装满了水的桶，每过固定的一段时间即向外漏一滴水。如果你接到了这滴水，那么你就可以继续服务请求，如果没有接到，那么就需要等待下一滴水。
2. 令牌桶则是指匀速向桶中添加令牌，服务请求时需要从桶中获取令牌，令牌的数目可以按照需要消耗的资源进行相应的调整。如果没有令牌，可以选择等待，或者放弃。

这两种方法看起来很像，不过还是有区别的。漏桶流出的速率固定，而令牌桶只要在桶中有令牌，那就可以拿。也就是说令牌桶是允许一定程度的并发的，比如同一个时刻，有 100 个用户请求，只要令牌桶中有 100 个令牌，那么这 100 个请求全都会放过去。令牌桶在桶中没有令牌的情况下也会退化为漏桶模型。

![漏桶算法](img\漏桶算法.jpg)

实际应用中令牌桶应用较为广泛，开源界流行的限流器大多数都是基于令牌桶思想的。并且在此基础上进行了一定程度的扩充，比如 [github.com/juju/ratelimi](https://github.com/juju/ratelimit)。

## 二、原理

### 1.生产令牌

```go
	var fillInterval = time.Millisecond * 10
	var capacity = 100
	var tokenBucket = make(chan struct{}, capacity)

	fillToken := func() {
		ticker := time.NewTicker(fillInterval)
		for {
			select {
			case <-ticker.C:
				select {
				case tokenBucket <- struct{}{}:
				default:
				}
				fmt.Println("current token cnt:", len(tokenBucket), time.Now())
			}
		}
	}

	go fillToken()
```

可以看到代码中使用了`select`来实现并发安全。

### 2.消费令牌

先考虑一种直接的解法，假设一次取一个令牌：

```go
func TakeAvailable(block bool) bool{
    var takenResult bool
    if block {
        select {
        case <-tokenBucket:
            takenResult = true
        }
    } else {
        select {
        case <-tokenBucket:
            takenResult = true
        default:
            takenResult = false
        }
    }

    return takenResult
}
```

我们来思考一下，令牌桶每隔一段固定的时间向桶中放令牌，如果我们记下上一次放令牌的时间为 t1，和当时的令牌数 k1，放令牌的时间间隔为 ti，每次向令牌桶中放 x 个令牌，令牌桶容量为 cap。现在如果有人来调用 `TakeAvailable` 来取 n 个令牌，我们将这个时刻记为 t2。在 t2 时刻，令牌桶中理论上应该有多少令牌呢？伪代码如下：

```go
cur = k1 + ((t2 - t1)/ti) * x
cur = cur > cap ? cap : cur
```

我们用两个时间点的时间差，再结合其它的参数，理论上在取令牌之前就完全可以知道桶里有多少令牌了。那劳心费力地像本小节前面向 channel 里填充 token 的操作，理论上是没有必要的。只要在每次 `Take` 的时候，再对令牌桶中的 token 数进行简单计算，就可以得到正确的令牌数。这就类似于惰性求值。

在得到正确的令牌数之后，再进行实际的 Take 操作就好，这个 Take 操作只需要对令牌数进行简单的减法即可，记得加锁以保证并发安全。`github.com/juju/ratelimit` 这个库就是这样做的。

# 封装和抽象

## 一、使用函数封装业务逻辑

比如现在有一个需求，要创建一笔订单，其中包含如下业务逻辑：

```go
func CreateOrder() {
    // 判断是否是地区限定商品
    // 检查是否是只提供给 vip 的商品
    // 从用户系统获取更详细的用户信息
    // 从商品系统中获取商品在该时间点的详细信息
    // 扣减库存
    // 创建订单快照
}
```

如果把这些逻辑都实现在一个函数中，显然这个函数就会变得很膨胀，可读性和可维护性都会变差。因此把每一步流程都用一个函数进行封装：

```go
func CreateOrder() {
    ValidateDistrict() // 判断是否是地区限定商品
    ValidateVIPProduct() // 检查是否是只提供给 vip 的商品
    GetUserInfo() // 从用户系统获取更详细的用户信息
    GetProductDesc() // 从商品系统中获取商品在该时间点的详细信息
    DecrementStorage() // 扣减库存
    CreateOrderSnapshot() // 创建订单快照
    return CreateSuccess
}
```

## 二、使用 interface 进行抽象

比如现在要做一个平台系统，可以用来处理不同类型的业务，但是业务流程和数据规范都是一样的，这个时候就可以用接口进行抽象：
![用接口进行抽象](img\用接口进行抽象.png)

 在业务进行迭代时，平台的代码是不用修改的，这样我们便把这些接入业务当成了平台代码的插件(plugin)引入进来了。

```go
type BusinessInstance interface {
    ValidateLogin()
    ValidateParams()
    AntispamCheck()
    GetPrice()
    CreateOrder()
    UpdateUserStatus()
    NotifyDownstreamSystems()
}

// 入口函数，判断业务类型
func entry() {
    var bi BusinessInstance
    switch businessType {
        case TravelBusiness:
            bi = travelorder.New()
        case MarketBusiness:
            bi = marketorder.New()
        default:
            return errors.New("not supported business")
    }
}

func BusinessProcess(bi BusinessInstance) {
    bi.ValidateLogin()
    bi.ValidateParams()
    bi.AntispamCheck()
    bi.GetPrice()
    bi.CreateOrder()
    bi.UpdateUserStatus()
    bi.NotifyDownstreamSystems()
}
```

不管日后新增多少类型的业务，只需实现平台提供的接口就行，而这些增长对平台而言都是透明的。

### Go interface 的优缺点

#### 1.优点：正交性

比如下面这个函数使用`io.Writer`作为参数： 

```go
func SetOutput(w io.Writer) {
    output = w
}
```

`io.Writer`是一个接口，既然是接口那么就可以自己实现接口，然后再调用函数`SetOutput`，这样也就不用`import`其他模块了。在 Go 的正交 interface 的设计场景下甚至可以去除依赖。

而且，编译器可以帮助我们在编译期就能检查到类似“未完全实现接口”这样的错误。

#### 2.缺点

但这种“正交”性也会给我们带来一些麻烦。当我们接手了一个几十万行的系统时，如果看到定义了很多 interface，例如订单流程的 interface，我们希望能直接找到这些 interface 都被哪些对象实现了。但直到现在，这个简单的需求也就只有 GoLand 实现了，并且体验尚可。Visual Studio Code 则需要对项目进行全局扫描，来看到底有哪些 struct 实现了该 interface 的全部函数。那些显式实现 interface 的语言，对于 IDE 的 interface 查找来说就友好多了。另一方面，我们看到一个 struct，也希望能够立刻知道这个 struct 实现了哪些 interface，但也有着和前面提到的相同的问题。

## 三、table-driven

在函数中如果有 if 和 switch 的话，会使函数的圈复杂度上升。所以可以用 table-driven 的方式对上面提到的入口函数进行修改：

```go
func entry() {
    var bi BusinessInstance
    switch businessType {
        case TravelBusiness:
            bi = travelorder.New()
        case MarketBusiness:
            bi = marketorder.New()
        default:
            return errors.New("not supported business")
    }
}
```

可以修改为：

```go
var businessInstanceMap = map[int]BusinessInstance {
    TravelBusiness : travelorder.New(),
    MarketBusiness : marketorder.New(),
}

func entry() {
    bi := businessInstanceMap[businessType]
}
```

在日常的开发工作中可以多多思考，哪些不必要的 switch case 可以用一个字典和一行代码就可以轻松搞定。





