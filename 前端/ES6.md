# 箭头函数

有些浏览器可能不支持这个特性。

## 一、格式

有两种格式：

只包含一个表达式，连`{ ... }`和`return`都省略掉了：

```javascript
x => x * x 
// 相当于 
// function (x) {
//    return x * x;
// }
```

包含多条语句，这时候就不能省略`{ ... }`和`return`：

```javascript
x => {
    if (x > 0) {
        return x * x;
    }
    else {
        return - x * x;
    }
}
```

### 多参数

如果参数不是一个，就需要用括号`()`括起来：

```javascript
// 两个参数:
(x, y) => x * x + y * y

// 无参数:
() => 3.14

// 可变参数:
(x, y, ...rest) => {
    var i, sum = x + y;
    for (i=0; i<rest.length; i++) {
        sum += rest[i];
    }
    return sum;
}
```

### 返回对象

如果要返回一个对象，就要注意，如果是单表达式，这么写的话会报错：

```javascript
// SyntaxError:
x => { foo: x }
```

因为和函数体的`{ ... }`有语法冲突，所以要改为：

```javascript
// ok:
x => ({ foo: x })
```

## 二、this

箭头函数内部的`this`是词法作用域，由上下文确定。

```javascript
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = () => new Date().getFullYear() - this.birth; // this指向obj对象
        return fn();
    }
};
obj.getAge(); // 25
```

