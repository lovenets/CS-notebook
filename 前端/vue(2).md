# 组件详解

## Vue CLI

vue CLI 是快速构建单页应用的脚手架，配合 vuejs-template 就可以快速创建一个 vue 应用。

### 1.安装 Vue CLI

安装 Vue CLI 要用到 npm（安装方法：[npm](https://www.npmjs.com/get-npm)) 或 yarn 这些包管理工具：

```shell
npm install -g @vue/cli
```

### 2.使用 vuejs-template

依次执行以下命令：

```shell
npm install -g vue-cli
vue init webpack my-project
cd my-project
npm install
npm run dev
```

执行 `vue init webpack my-project`这一步时可能会卡住，如果出现这种情况的话就使用全局 VPN。

执行完以上步骤之后可以看到生成了一个工程：

```
VUE-WEBPACK
	|-node_modules
	|-src  
		|-assets
			logo.png
		App.vue
		main.js
	.babelrc
	.editorconfig
	.gitgnore
	index.html
	package-lock.json
	package.json
	README.md
	webpack.config.js
```

## .vue 文件

如果只是定义全局组件，那么存在以下缺陷：

- **全局定义 (Global definitions)** 强制要求每个 component 中的命名不得重复
- **字符串模板 (String templates)** 缺乏语法高亮，在 HTML 有多行的时候，需要用到丑陋的 `\`
- **不支持 CSS (No CSS support)** 意味着当 HTML 和 JavaScript 组件化时，CSS 明显被遗漏
- **没有构建步骤 (No build step)** 限制只能使用 HTML 和 ES5 JavaScript, 而不能使用预处理器，如 Pug (formerly Jade) 和 Babel

文件扩展名为 `.vue` 的 **single-file components(单文件组件)** 为以上所有问题提供了解决方法，并且还可以使用 webpack 或 Browserify 等构建工具。

一个`.vue`文件的内容看上去就是一个`.html`文件，只能有三个根元素：`<template>`（必需）,`<script>`（可选），`<style>`（可选）。

```html
<template>
    <div>
        <input type="text" v-model="message">
        <p>{{ message }}</p>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                message: ""
            };
        }
    }
</script>

<style>

</style>
```

这就定义了一个组件，注意 JavaScript 代码的格式与普通组件的区别。

### 引用其他 .vue 文件中定义的组件

如果 .vue 中存在代码：

```html
<script>
    export default {
		//...
    }
</script>
```

就说明这个组件是可导出，也就可以在别的文件中引用。假设`Input.vue`中定义了一个输入框组件，现在在`Message.vue`中要引用这个组件：

```html
<template>
    <div>
        <h1>This is a Message.</h1>
        <app-input></app-input>
    </div>
</template>

<script>
    import Input from "./Input.vue"  // 导入 
    export default {
        components: {
            "app-input": Input // 注册组件
        }
    }
</script>
```

如果使用了 vuejs-template 来构建工程，那么还可以在`main.js`文件中注册全局组件：

```js
import Vue from 'vue'
import App from './App.vue'
import Message from './Message.vue'

Vue.component("message",Message) // 注册全局组件

new Vue({
  el: '#app',
  render: h => h(App)
})
```

然后就可以在`App.vue`中使用组件`message`：

```html
<template>
  <div id="app">
      <message></message>
  </div>
</template>
```

## 通过 prop 向子组件传递数据

一个组件的 template 中可以使用另一个组件，被嵌套的组件就称为子组件：

```html
<template>
    <div>
        <h1>{{ message }}</h1>
        <!-- 使用子组件 -->
        <app-input></app-input>
    </div>
</template>

<script>
    import input from "./input.vue" // 导入外部组件
    export default {
        data() {
            return {
                message: "This is a Message."
            };
        },
        components: { // 注册组件
            "app-input": input
        }
    }
</script>
```

如果想通过父组件把数据传递给子组件，最常用的一个方法就是自定义子组件标签的属性，然后将数据作为子组件的属性传递给子组件：

```html
<!-- 子组件定义 -->
<template>
    <div>
        <!-- 输出自定义属性的值 -->
        <p>{{ msg }}</p>
    </div>
</template>

<script>
    export default {
        props: ["msg"] // 自定义一个名为 msg 的属性
    }
</script>
```

`props:["msg"]`就是自定义属性，数量没有限制，只需在数组中添加表示属性名的字符串。

此时就可以在父组件中向子组件传递数据：

```html
<template>
    <div>
        <h1>{{ message }}</h1>
        <!-- 使用子组件 -->
        <!-- 使用 v-bind 指令传递数据 -->
        <app-input :msg="message"></app-input>
    </div>
</template>

<script>
    import input from "./input.vue" // 导入外部组件
    export default {
        data() {
            return {
                message: "This is a Message."
            };
        },
        components: { // 注册组件
            "app-input": input
        }
    }
</script>
```

## 通过自定义事件向父组件传递数据

子组件向父组件传递数据，通常是用自定义事件实现的。

首先在子组件中设置传递数据的机制：

```html
<template>
    <div>
        <!-- 向文本框输入值之后，就通过 changeMessage 方法向父组件传递数据 -->
        <input type="text" @input="changeMessage">
    </div>
</template>

<script>
    export default {
        methods: {
            changeMessage: function(event) {
                // 调用 vue 的内置函数 $emit
                // 第一个参数是自定义的事件名，在父组件中需要监听这个事件
                // 第二个参数是回送给父组件监听器的值
                this.$emit("messageChanged",this.message);
            }
        }
    }
</script>
```

自定义事件的时候，需要经常用到内建函数 [$emit](https://cn.vuejs.org/v2/api/#vm-emit)。

然后在父组件中使用自定义事件：

```html
<template>
    <div>
        <h1>{{ message }}</h1>
        <!-- 使用 v-for 指令监听自定义的事件 messageChanged -->
        <!-- $event 就是获取 $emit 函数的回送值，也就是获取了子组件传递过来的数据 -->
        <app-input @messageChanged="message = $event"></app-input>
    </div>
</template>

<script>
    import input from "./input.vue"
    export default {
        data() {
            return {
                message: "This is a Message."
            };
        },
        components: {
            "app-input": input
        }
    }
</script>
```

# Vue Router

Vue Router 的作用就是把 URL 映射为组件，这么做的好处是可以构建出只有一个页面的“多页应用”。

以下的例子基于 vuejs-template 构建的工程，一般的做法参考[官方文档](https://router.vuejs.org/zh/installation.html)。

安装 Vue Router：`npm install --save vue-router`

## 基本使用 `<router-view>`

在`App.vue`中添加`<router-view>`标签：

```html
<template>
  <div id="app">
    <h1>Let's go somewhere!</h1>
    <hr>
    
    <!-- 路由匹配到的组件将会在这里进行渲染 -->
    <router-view></router-view>
  </div>
</template>
```

然后在`main.js`中进行设置：

```javascript
import Vue from 'vue'
import App from './App.vue'
import VueRouter from 'vue-router'
import User from './User.vue'
import Home from './Home.vue'

Vue.use(VueRouter); // 在模块化机制编程的情况下使用 vue-router

// 配置 URL 映射，每个路由映射为一个组件
const routes = [ 
  {path: "/user",component: User},
  {path: "/",component: Home}
];

// 创建 router 实例并配置
const router = new VueRouter({
  routes, // 缩写，相当于 routes: routes
  mode: "history" 
});

new Vue({
  el: '#app',
  router, // 注册 router
  render: h => h(App)
})

```

启动应用后访问`localhost:8080/`页面就会渲染 Home 组件，访问`localhost:8080/user`则渲染 User 组件。

### HTML5 History 模式

在创建 router 实例的时候有一个属性`mode: "history" `，这是使用传统的 URL 模式。如果不加这一个，那么访问`localhost:8080/#`页面才会渲染 Home 组件，访问`localhost:8080/#/user`则渲染 User 组件。

这两种模式的区别在于，使用 # 的话当 URL 发生变化时前端并不会向服务器发送一个请求，只是由 Vue 在前端选择匹配的组件进行渲染，如果找不到匹配的组件，则会渲染原本的`index.html`页面。

如果使用`history`模式，那么每次 URL 发生变化前端就会向服务器请求一个对应的页面，如果找不到，那就是404错误。

## 基本使用 `<router-link>`

`<router-link>`会被渲染为`<a>`标签：

```html
<template>
  <div id="app">
    <h1>Let's go somewhere!</h1>
    <hr>

    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/user">Go to User1</router-link>

    <!-- 路由匹配到的组件将会在这里进行渲染 -->
    <router-view></router-view>
  </div>
</template>
```

## URL 传递参数

比如现在有一个带参数的 URL ：

```html
<template>
  <div id="app">
    <h1>Let's go somewhere!</h1>
    <hr>

    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/user/1">Go to User1</router-link>
    <router-link to="/user/2">Go to User2</router-link>

    <!-- 路由匹配到的组件将会在这里进行渲染 -->
    <router-view></router-view>
  </div>
</template>
```

那么在 URL 匹配的组件内可以获取这个参数：

```html
<template>
  <div>
      <h1>User</h1>
      <!-- 获取参数并输出 -->
      <p>ID is {{ $route.params.id }}</p>
  </div>
</template>
```

在 JavaScript 代码中则需加上`this`：`this.$route.params.id`

## 编程式导航

前面都是通过点击链接实现组件的导航，假设现在需要点击一个按钮然后导航：

```html
<template>
  <div>
      <h1>User</h1>
      <!-- 获取参数并输出 -->
      <p>ID is {{ $route.params.id }}</p>
      <button @click="goHome">Go Home</button>
  </div>
</template>
```

对应的`goHome`方法为：

```html
<script>
export default {
    methods: {
        goHome: function() {
            this.$router.push("/");
        }
    }
}
</script>
```

`route.push`函数可以访问路由，编程式导航的更多实现可以参考[官方文档](https://router.vuejs.org/zh/guide/essentials/navigation.html)。

# Vue 和 Spring Boot 整合

用 vue-cli 搭好前端应用的脚手架、编写完所有前端代码并进行相关配置之后，执行 

```powershell
npm run build
```

会生成一个`/dist`目录，目录下就是打包好的前端应用，把里面的文件拷贝到 Spring Boot 项目的 `/resources/static`下，就完成了部署。

## 需要注意的问题

通过 axios 与 Spring Boot 通信会遇到两个问题。

### 1.跨域问题 

开发过程中，前后端通常都是启用单独的端口，因此会有跨域的问题，解决方案是写一个配置类：

```java
@Configuration
public class CorsConfig {
    /**
     允许任何域名使用
     允许任何头
     允许任何方法（post、get等）
     */
    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        // // addAllowedOrigin 不能设置为* 因为与 allowCredential 冲突,需要设置为具体前端开发地址
        corsConfiguration.addAllowedOrigin("http://localhost:9528"); // 允许哪个域名向服务器发送请求
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        // allowCredential 需设置为true
        corsConfiguration.setAllowCredentials(true);
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig());
        return new CorsFilter(source);
    }
}
```

### 2. controller 无法获取请求参数

前端发送的数据需要重新编码，修改格式：

```javascript
// 设置content-type
// 这里处理的是 针对SpringMVC Controller 无法正确获得请求参数的问题
axios.interceptors.request.use(
  config => {
    let data = config.data
    let key = Object.keys(data)
    // 重写data，由{"name":"name","password":"password"} 改为 name=name&password=password
    config.data = encodeURI(key.map(name => `${name}=${data[name]}`).join('&'))
    // 设置Content-Type
    config.headers = {
      'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8'
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)
```

这里使用了 axios 的拦截器，可以方便地进行全局配置。

### 3.将前端应用打包前的配置

config/index.js 文件就是用来配置的，有一些比较关键的配置：

```javascript
module.exports = {
  dev: {
  	//...
    // 开发环境启用的主机
    host: 'localhost', // can be overwritten by process.env.HOST
    // 开发环境启用的端口
    port: 8080, // can be overwritten by process.env.PORT, if port is in use, a free one will be determined
		//...
  },

  build: {
    // 将index.htlm 指定生成在dist下
    index: path.resolve(__dirname, '../dist/index.html'),

    assetsRoot: path.resolve(__dirname, '../dist'),
    // 指定静态文件 js/css等与index平级
    assetsSubDirectory: './',
    // 指定引用地址为相对地址，这样生成的文件就可以直接打开了
    // 若指定为 '/' 代表是根目录地址，属于可以发布在单独服务器上的
    assetsPublicPath: './', 

    //...
  }
}
```

