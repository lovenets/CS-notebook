# 安装

推荐链接到一个可以手动更新的指定版本号：

```html
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.17/dist/vue.js"></script>
```

在用 Vue 构建大型应用时推荐使用 npm 安装：

```shell
# 最新稳定版
$ npm install vue
```

安装之后就可以通过编写 JavaScript 脚本来使用 Vue。

入门示例：

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width,initial-scale=1">
        
        <title>vue.js</title>

    </head>

    <body>
        <div id="app">
            <h1>{{msg}}</h1>
        </div>
    </body>

    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.17/dist/vue.js"></script>

    <script>
        var app = new Vue({
            el: '#app', // 操作的元素
            data: {     // 数据用 json
                msg:'Hello Vue.js'
            }
        })
    </script>
</html>
```



# Vue 实例

每个 Vue 应用都是通过用 `Vue` 函数创建一个新的 **Vue 实例**开始的：

```javascript
var vm = new Vue({
  // ....
})
```

# 指令

指令用来指示浏览器要如何渲染元素。

## v-text  & v-html

### 1.v-text

这个指令将标签内的文本内容替换成指定的值。

```html
<div id="app">
     <h1 v-text="msg"></h1>
</div>

<script>
    var app = new Vue({
        el: '#app',
        data: {
            msg:'Hello Vue.js'
        }
    })
</script>
```

### 2.v-html

这个指令与`v-text`类似，但不同之处在于如果字符串包含有 HTML 标签，那么这个指令会进行渲染，而`v-tetx`指令只会将其作为普通的文本。

```html
    <body>
        <div id="app">
            <h1 v-html="msg"></h1>
        </div>
    </body>

    <script>
        var app = new Vue({
            el: '#app',
            data: {
                msg:'<strong>Hello Vue.js</strong>'
            }
        })
    </script>
```

## v-show & v-if

###  1.v-show

这个指令的参数是布尔值，如果为 true，那么标签的内容就会显示。

```html
<h1 v-show="viewed">You'll see something.</h1>
```

### 2.v-if

这个指令和`v-show`的效果相同，但是原理不同：`v-show`只是控制是否显示这个标签，而`v-if`会控制是否将这个标签从 DOM 中移除。

```html
<!-- 如果 viewed 的值为 false，那么这个标签就会被移除 -->
<h1 v-if="viewed">You'll see something.</h1>
```

`v-if`的值还可以是==表达式==：

```html
<h1 v-if="viewed1 !== viewed2">You'll see something.</h1>
   
<script>
     var app = new Vue({
         el: '#app',
         data: {
             msg:'<strong>Hello Vue.js</strong>',
             viewed1: true,
             viewed2: false
         }
     })
</script>
```

还可以和`v-else`配合使用，相当于`id-else`：

```html
<!-- viewed1 为 true 则为 v-if，否则就是 v-else -->
<h1 v-if="viewed1">You'll see something.</h1>
<h1 v-else>You'll see something else.</h1>
```

## v-pre

这个指令将会禁止渲染：

```html
<!-- 将会在页面上显示 {{ msg }} -->
<h1 v-pre>{{ msg }}</h1>
```

## v-once

这个指令将会只渲染一次，当数据发生改变时将不会再次渲染：

```html
<!-- 如果在首次加载页面后 msg 的值发生改变，那么这个标签将不会收到影响 -->
<h1 v-once>{{ msg }}</h1>
```

## v-cloak

只有当整个视图都加载完成后，这个指令才会进行渲染：

```html
<h1 v-cloak>{{ msg }}</h1>
```

## v-bind

这个指令的重要用途就是把数据绑定到标签的属性上，而不仅仅是把数据作为向用户展示的输出内容。

```html
<h1 v-bind:title="msg">Hello World</h1>

<script>
    var app = new Vue({
        el: '#app',
        data: {
            msg:'You loaded this page on ' + new Date(), // 可以直接使用 JavaScript 代码
        }
    })
</script>
```

因为这个指令很常用，所以 Vue 提供了缩写：

```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

## v-for

这个指令用来实现循环。

```html
<ul>
    <!-- 用 todo 来迭代每一项，然后用 {{}} 取值 -->
	<li v-for="todo in todos">{{ todo.text }}</li>
</ul>

<script>
     var app = new Vue({
         el: '#app',
         data: {
             todos: [
                 {text: 'Learn Vue'},
                 {text: 'Like the video'}
             ]
        }
     })
</script>
```

`v-fof`还有一种用法：

```html
<!-- 将会生成10个列表项 -->
<li v-for="count in count">{{ count }}</li>

<script>
     var app = new Vue({
         el: '#app',
         data: {
             count: 10
        }
     })
</script>
```

# 双向绑定

Vue 的设计受到了 MVVM 模式的影响，Vue 的双向绑定指的就是 view- model 数据的双向绑定。

```html
<input type="text" v-model="msg">

<script>
    var app = new Vue({
         el: '#app',
         data: {
             msg: 'Hello Vue.js',
         }
   })
</script>
```

改变`msg`的值，输入框内的值会发生改变；改变输入框的值，那么`msg`的值也会改变。

用 `v-model` 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素， `v-model`本质上不过是语法糖。

# 事件处理

`v-on`指令用来进行事件处理：

```html
<body>
    <div id="app">
         <h1>You have clicked {{ count }} times.</h1>
         <button v-on:click="countUp">Count up!</button>
         <button v-on:click="countDown">Count down!</button>
    </div>
</body>

<script>
     var app = new Vue({
         el: '#app',
         data: {
            count: 0
         },
         methods: {
             countUp: function() {
                 this.count++;
             },
             countDown: function() {
                 this.count--;
             }
        }
    })
</script>
```

`v-on:click`表明这是处理点击事件，后面指定响应该事件的函数，也可以直接写上一个表达式。`methods`中可以有多个函数，也是用键值对的方式来声明。

因为`v-on`也是一个常用的指令，所以 Vue 也提供了缩写：

```html
<input v-on:click="submit">

<!-- 缩写语法 -->
<input @click="submit">
```

# 计算属性

计算属性就是通过已有的数据属性生成新的数据。

```html
<body>
    <div id="app">
         <h1>Hello {{fullname}}</h1>
         <ul>
            <li>First Name: {{first}}</li>
            <li>Last Name: {{last}}</li>
         </ul>
         <br>
         First Name: <input type="text" v-model="first">
         Last Name: <input type="text" v-model="last">
     </div>
</body>    

<script>
        var app = new Vue({
            el: '#app',
            data: {
                first: '',
                last: ''
            },
            computed: {
                fullname: function() {
                    return this.first + " " + this.last;
                }
            }
        })
</script>
```

计算属性的 key 为 computed，每一个计算属性的值可以是函数。在上面的例子中，当 first 和 last 属性发生变化时 fullname 也会随之改变。

## computed 与 methods 的区别

在上面的例子中，也可以把计算属性改为方法：

```html
<body>
    <div id="app">
         <h1>Hello {{fullname()}}</h1>
         <ul>
            <li>First Name: {{first}}</li>
            <li>Last Name: {{last}}</li>
         </ul>
         <br>
         First Name: <input type="text" v-model="first">
         Last Name: <input type="text" v-model="last">
     </div>
</body>    

<script>
        var app = new Vue({
            el: '#app',
            data: {
                first: '',
                last: ''
            },
            methods: {
                fullname: function() {
                    return this.first + " " + this.last;
                }
            }
        })
</script>
```

在用户看来这两种实现方法最终实现的效果是一样的，但是在这两种情况下视图的行为会有所不同。

- 计算属性：视图会缓存先前计算得到的属性，除非计算属性所依赖的其他属性发生改变，否则 Vue.js 将不会再次计算该属性
- 方法：视图不会缓存方法调用的结果，每次用到这个方法就会产生一次新的调用

## getter & setter

实际上赋给计算属性的函数在默认情况下是一个 getter，这也就意味着在下面的例子中无法通过修改`Full Name`文本框的值来改变 fullname 属性：

```html
    <body>
        <div id="app">
            <h1>Hello {{fullname}}</h1>
            <ul>
                <li>First Name: {{first}}</li>
                <li>Last Name: {{last}}</li>
            </ul>
            <br>
            <!-- 即使用了 v-model 指令也无法实现双向绑定 -->
            Full Name: <input type="text" v-model="fullname">
            First Name: <input type="text" v-model="first">
            Last Name: <input type="text" v-model="last">
        </div>
    </body>

    <script>
        var app = new Vue({
            el: '#app',
            data: {
                first: '',
                last: ''
            },
            computed: {
                fullname: function() {
                    return this.first + " " + this.last;
                }
            }
        })
    </script>
```

如果想要实现和普通属性一样的双向绑定，就要给计算属性添加 setter：

```html
    <script>
        var app = new Vue({
            el: '#app',
            data: {
                first: '',
                last: ''
            },
            computed: {
                fullname: {
                    //  getter
                    get: function() {
                        return this.first + " " + this.last;
                    },
                    // 添加 setter
                    set: function(value) {
                        var name = value.split(' ');
                        this.first = name[0];
                        this.last = name[name.length - 1];
                    }
                }
            }
        })
    </script>
```

# axios

当需要调用外部 API 时，可以使用 axios 这个库：

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

axios 可以发送 HTTP 请求。

示例：输入编码，异步查询编码对应的城市。（下面的例子还使用了 lodash 的`_.debounce`函数）

```html
<body>
    <div id="app">
         <input type="text" placeholder="Starting Zip" v-model="startingZip">
         <span>{{ startingCity }}</span>
    </div>
</body>

<script>
        var app = new Vue({
            el: '#app',
            data: {
                startingZip: '',
                startingCity: ''
            },
            // 监听器
            watch: {
                startingZip: function() {
                    this.startingCity = " ";
                    if(this.startingZip.length === 5){
                        this.lookupStartingCity();
                    }
                }
            },
            methods: {
                lookupStartingCity: _.debounce(function() {
                    var app = this
                    app.startingCity = "Searching...";
                    axios.get("http://ziptasticapi.com/" + app.startingZip) // 调用外部 API
                    .then(function(response) { // 服务器响应
                        app.startingCity = response.data.city + "," + response.data.state
                    })
                    .catch(function(error) { // 出现错误
                        app.startingCity = "Invalid Zipcode";
                    });
                },500)
            }
        })
</script>
```

说明：

（1）`watch`就是监听器，当监听的属性发生变化时执行指定的操作

（2）axios 会使用 Ajax 来调用外部 API，基本格式是：

```javascript
axios.get(" ") // 发送 HTTP 请求
    .then(function(response) { // 处理响应
    //....
})
    .catch(function(error) { // 处理错误
    //...
})
```

# 组件

组件就是可重用的 Vue 实例。

```javascript
    Vue.component("app-user", {
        data: function () {
            return {
                users: [{
                        username: "Max"
                    },
                    {
                        usernmae: "Chris"
                    },
                    {
                        username: "Anna"
                    }
                ]
            };
        },
        template: "<div><div v-for='user in users'><p>Username: {{user.username}}</p></div></div>"
    })
```

这就注册了一个叫做`app-user`的组件，在页面中使用时看上去就像是定义了一个新的标签：

```html
<app-user></app-user>
```

这个标签就等同于组件定义中`template`的值：

```html
<div>
    <div v-for="user in users">
        <p>Username: {{user.username}}</p>
    </div>
</div>
```

说明：

（1）组件的`data`必须是一个函数，返回 JSON 数据。这样设计的目的是每个实例可以维护一份被返回对象的独立的拷贝。

（2）组件分为全局组件和局部组件。

```javascript
// 全局组件
Vue.component("globalComponent",{
	//...
})

// 局部组件
var componentA = {
    // 组件定义
} 

new Vue({
    el: "#app",
    components: {
        // 注册了一个叫做 localComponent 的局部组件
        "localComponent": ComponentA
    }
})
```

全局组件可以在任何 Vue 实例中使用，但是局部组件只能在指定的 Vue 实例中使用。

（3）组件必须有一个根元素：

```html
<!-- 正确 -->
<div>
    <div v-for="user in users">
        <p>Username: {{user.username}}</p>
    </div>
</div>

<!-- 错误 -->
<div v-for="user in users">
    <p>Username: {{user.username}}</p>
</div>
```







