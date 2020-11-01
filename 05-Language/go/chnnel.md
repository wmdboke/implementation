### channel

channel是一种通信机制，内部绑定了数据类型。



###### 创建与关闭

ch := make(chan, int)

close(ch)



###### 读

从已经关闭的channle中读取不会panic，且能读出channel中还未读出的数据，如果均已经读出则会读出的类型为零值。 从已经关闭的channel中读取不会阻塞，会返回为false的 ok-idom, 可以判断channel是否关闭

```go
ch := make(chan int, 10)
ch <- 11
ch <- 12

close(ch)

for x := range ch {
    fmt.Println(x)
}

x, ok := <- ch
fmt.Println(x, ok)


-----
output:

11
12
0 false
```

###### 缓存

ch := make(chan, int, 10)

只要channel中有缓存读取写入就不会阻塞，无缓存的情况都会阻塞。

###### 单项channel

```go
func foo(ch chan<- int) <-chan int {...}
```

`han<- int` 表示一个只可写入的 channel，`<-chan int` 表示一个只可读取的 channel。上面这个函数约定了 foo 内只能从向 ch 中写入数据，返回只一个只能读取的 channel，虽然使用普通的 channel 也没有问题，但这样在方法声明时约定可以防止 channel 被滥用，这种预防机制发生再编译期间。




