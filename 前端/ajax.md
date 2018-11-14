AJAX （Asynchronous JavaScript And XML），用异步的方式发送 HTTP 请求，请求成功发送之后通过回调函数接受响应，从而不影响其他操作。

不使用 AJAX 的弊端：如果仔细观察一个 Form 的提交，你就会发现，一旦用户点击“Submit”按钮，表单开始提交，浏览器就会刷新页面，然后在新页面里告诉你操作是成功了还是失败了。如果不幸由于网络太慢或者其他原因，就会得到一个404页面。  

# 一、基本使用

使用原生 JavaScript 发送 AJAX 请求：

```javascript
function success(text) {
    var textarea = document.getElementById('test-response-text');
    textarea.value = text;
}

function fail(code) {
    var textarea = document.getElementById('test-response-text');
    textarea.value = 'Error code: ' + code;
}

var request = new XMLHttpRequest(); // 新建XMLHttpRequest对象

request.onreadystatechange = function () { // 状态发生变化时，函数被回调
    if (request.readyState === 4) { // 成功完成
        // 判断响应结果:
        if (request.status === 200) {
            // 成功，通过responseText拿到响应的文本:
            return success(request.responseText);
        } else {
            // 失败，根据响应码判断失败原因:
            return fail(request.status);
        }
    } else {
        // HTTP请求还在继续...
    }
}

// 发送请求:
request.open('GET', '/api/categories');
request.send();

alert('请求已发送，请等待响应...');
```

1. 当创建了`XMLHttpRequest`对象后，要先设置`onreadystatechange`的回调函数。在回调函数中，通常我们只需通过`readyState === 4`判断请求是否完成，如果已完成，再根据`status === 200`判断是否是一个成功的响应。
2. `XMLHttpRequest`对象的`open()`方法有3个参数，第一个参数指定是 GET 还是 POST ，第二个参数指定 URL 地址，第三个参数指定是否使用异步，默认是`true`，所以不用写。
3. 最后调用`send()`方法才真正发送请求。GET 请求不需要参数，POST 请求需要把 body 部分以字符串或者 FormData 对象传进去。

## 低版本 IE 的写法

对于低版本的IE，需要换一个`ActiveXObject`对象：

```javascript
var request = new ActiveXObject('Microsoft.XMLHTTP'); // 新建Microsoft.XMLHTTP对象

request.onreadystatechange = function () { // 状态发生变化时，函数被回调
    if (request.readyState === 4) { // 成功完成
        // 判断响应结果:
        if (request.status === 200) {
            // 成功，通过responseText拿到响应的文本:
            return success(request.responseText);
        } else {
            // 失败，根据响应码判断失败原因:
            return fail(request.status);
        }
    } else {
        // HTTP请求还在继续...
    }
}

// 发送请求:
request.open('GET', '/api/categories');
request.send();

alert('请求已发送，请等待响应...');
```

如果想把两种写法混在一起：

```javascript
var request;
if (window.XMLHttpRequest) {
    request = new XMLHttpRequest();
} else {
    request = new ActiveXObject('Microsoft.XMLHTTP');
}
```

通过检测`window`对象是否有`XMLHttpRequest`属性来确定浏览器是否支持标准的`XMLHttpRequest`。

# 二、安全限制

默认情况下，JavaScript 在发送 AJAX 请求时，URL 的域名必须和当前页面完全一致。这就是==同源策略==：域名要相同（`www.example.com`和`example.com`不同），协议要相同（`http`和`https`不同），端口号要相同（默认是`:80`端口，它和`:8080`就不同）。有的浏览器口子松一点，允许端口不同，大多数浏览器都会严格遵守这个限制。

如果要用 AJAX 请求外域 URL，可以考虑以下两种方法。

## 1.代理服务器

同源域名下架设一个代理服务器来转发，JavaScript 负责把请求发送到代理服务器：

```javascript
'/proxy?url=http://www.sina.com.cn'
```

代理服务器再把结果返回，这样就遵守了浏览器的同源策略。这种方式麻烦之处在于需要服务器端额外做开发。

## 2.JSONP

它有个限制，只能用 GET 请求，并且要求返回 JavaScript。这种方式跨域实际上是利用了浏览器允许跨域引用 JavaScript 资源：

```html
<html>
<head>
    <script src="http://example.com/abc.js"></script>
    ...
</head>
<body>
...
</body>
</html>
```

JSONP 通常以函数调用的形式返回，例如，返回 JavaScript 内容如下：

```javascript
foo('data');
```

这样一来，我们如果在页面中先准备好`foo()`函数，然后给页面动态加一个`<script>`节点，相当于动态读取外域的 JavaScript 资源，最后就等着接收回调了。

示例：

以163的股票查询 URL 为例，对于 URL：http://api.money.126.net/data/feed/0000001,1399001?callback=refreshPrice，你将得到如下返回：

`refreshPrice({"0000001":{"code": "0000001", ... });`
因此我们需要首先在页面中准备好回调函数：

```javascript
function refreshPrice(data) {
    var p = document.getElementById('test-jsonp');
    p.innerHTML = '当前价格：' +
        data['0000001'].name +': ' + 
        data['0000001'].price + '；' +
        data['1399001'].name + ': ' +
        data['1399001'].price;
}
```

触发：

```javascript
function getPrice() {
    var js = document.createElement('script'),
    var head = document.getElementsByTagName('head')[0];
    js.src = 'http://api.money.126.net/data/feed/0000001,1399001?callback=refreshPrice';
    head.appendChild(js); // 动态添加节点
}
```


就完成了跨域加载数据。

# 三、CORS

CORS 全称 Cross-Origin Resource Sharing，是 HTML5 规范定义的如何跨域访问资源。

Origin 表示本域，也就是浏览器当前页面的域。当 JavaScript 向外域（如 sina.com）发起请求后，浏览器收到响应后，首先检查 Access-Control-Allow-Origin 是否包含本域，如果是，则此次跨域请求成功，如果不是，则请求失败，JavaScript 将无法获取到响应的任何数据。

比如，假设本域是 my.com，外域是 sina.com，只要响应头 Access-Control-Allow-Origin 为 http://my.com，或者是*，本次请求就可以成功。可见，跨域能否成功，取决于对方服务器是否愿意给你设置一个正确的Access-Control-Allow-Origin，决定权始终在对方手中。

## 1.简单请求

简单请求包括 GET、HEAD 和 POST（POST 的 Content-Type 类型
仅限`application/x-www-form-urlencoded`、`multipart/form-data`和`text/plain`），并且不能出现任何自定义头（例如，`X-Custom: 12345`），通常能满足90%的需求。

## 2.其他请求

对于 PUT、DELETE 以及其他类型如`application/json`的 POST 请求，在发送 AJAX 请求之前，浏览器会先发送一个`OPTIONS`请求（称为 preflighted 请求）到这个 URL 上，询问目标服务器是否接受：

```http
OPTIONS /path/to/resource HTTP/1.1
Host: bar.com
Origin: http://my.com
Access-Control-Request-Method: POST
```

服务器必须响应并明确指出允许的 Method：

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://my.com
Access-Control-Allow-Methods: POST, GET, PUT, OPTIONS
Access-Control-Max-Age: 86400
```

浏览器确认服务器响应的`Access-Control-Allow-Methods`头确实包含将要发送的 AJAX 请求的 Method，才会继续发送 AJAX，否则，抛出一个错误。

