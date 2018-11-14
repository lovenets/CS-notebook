# 一、拒绝将接口做为参数

使用接口作为参数可以带来更好的扩展性，比如将`[]byte`转换为`io.Reader`。

# 二、不使用`io.Reader`和`io.Writer`

`io.Reader`和`io.Writer`是两个简单、灵活的可提供许多 IO 操作的接口，使用这两个接口可以使程序的可扩展性更好。

# 三、接受宽接口作为参数

宽接口是指具有很多不必要方法的接口，与之对应的则是窄接口。

# 四、方法 vs 函数

1. 方法会改变结构体的状态（即属性），函数则不依赖于结构体的状态。
2. 方法和结构体是绑定的，函数可以接受接口作为参数

# 五、值 vs 指针

1. 值和指针实际上在性能上没有多大差别，问题只在于是否要共享结构体的状态。如果需要和方法或函数共享结构体的状态，则用指针。
2. 指针对于并发来说是不安全的。
3. 如果结构体没有状态，只定义了方法，那么就用值。

# 六、将 error 看作字符串

很多人会这么实现自定义的错误类型：

```go
func (e MyError) Error() string {
	return fmt.Sprintf("%v: %v", e.When, e.What)
}
```

乍一看似乎可以把 error 看作字符串，然后当需要比较错误类型时，实际上就变成了比较字符串。这种做法并不是很好，比较好的做法是使用`errors.New`方法声明错误类型：

```go
var ErrorNoNama = errors.New("0 length of name")

func SetName(name string) error {
    if len(name) == 0 {
        return ErrorNoName
    }
}

func foo(name string) error {
    err := SetName("bar")
    if err == ErrorNoName {
        SetName("default")
    } else {
        log.Fatal(err)
    }
}
```

大部分情况下`errors.New`已经够用了，在实现自定义的错误类型前，考虑几点因素：

- 需要提供一些 context 以保证一致性
- 需要提供和错误值不同类型的变量
- 需要提供动态的值

# 七、并发安全





