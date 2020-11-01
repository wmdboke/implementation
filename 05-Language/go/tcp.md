##### 服务器端示例：

```go
//go-tcpsock/server.go
func HandleConn(conn net.Conn) {
    defer conn.Close()

    for {
        // read from the connection
        // ... ...
        // write to the connection
        //... ...
    }
}

func main() {
    listen, err := net.Listen("tcp", ":8888")
    if err != nil {
        fmt.Println("listen error: ", err)
        return
    }

    for {
        conn, err := listen.Accept()
        if err != nil {
            fmt.Println("accept error: ", err)
            break
        }

        // start a new goroutine to handle the new connection
        go HandleConn(conn)
    }
}
```

##### 客户端示例：

```go
//阻塞
conn, err := net.Dial("tcp", "www.baidu.com:80")
if err != nil {
    //handle error
}
//超时
conn, err := net.DialTimeout("tcp", "www.baidu.com:80", 2*time.Second)
if err != nil {
    //handle error
}
```

<font color=#0099ff>服务器backlog满时，会阻塞客户端Dail，直到服务器一次accept 。</font>

##### socket的属性

SetKeepAlive   
SetKeepAlivePeriod   
SetLinger   
SetNoDelay （默认no delay）   
SetWriteBuffer   
SetReadBuffer
