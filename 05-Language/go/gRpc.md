#### gRPC 与 rest api 的区别

底层都是采用http进行通信的。准确的说gRPC采用的是http2.0协议。

gRPC采用protobuf定义接口，有更严格的接口规范，而且protobuf是二进制传输更有效。

#### 安装

git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc

git clone https://github.com/golang/net.git $GOPATH/src/golang.org/x/net

git clone https://github.com/golang/text.git $GOPATH/src/golang.org/x/text

go get -u github.com/golang/protobuf/{proto,protoc-gen-go}

git clone https://github.com/google/go-genproto.git $GOPATH/src/google.golang.org/genproto

cd $GOPATH/src/

go install google.golang.org/grpc

#### 示例：

###### 步骤如下：

1. lister, err : = net.Lister(tpye, endpoint ) // type =  tcp / unix

2. opts := []]grpc.ServerOption{}

3. server := grpc.NewServer(opts...)

4. proto_struct.RegisterServer(rpc.server,  server)

5. server.Serve(lister)

6. handler 返回 pro.db.response 对象

```protobuf
syntax = "proto3";

package test;

service Wait {
    rpc DoMD5(Req) returns (Rep);
}

message Req{
    string fromjson = 1;
}

message Rep{
    string md5str = 1;
}

protoc --go_out=plugins=grpc:./ ./test.proto
//可以用不同的插件
```

```go
// server.go

package main

import (
    "context"
    "crypto/md5"
    "fmt"
    test "go-test/grpc/proto"
    "google.golang.org/grpc/reflection"
    "log"
    "net"

    "google.golang.org/grpc"
)

// 业务实现逻辑的结构体
type server struct{}

// 为server定义 DoMD5 方法 内部处理请求并返回结果
// 参数 (context.Context[固定], *test.Req[相应接口定义的请求参数])
// 返回 (*test.Res[相应接口定义的返回参数，必须用指针], error)
func (s *server) DoMD5(ctx context.Context, in *test.Req) (*test.Rep, error) {
    fmt.Println("测试请求： " + in.Fromjson)
    return &test.Rep{Md5Str: "MD5 :" + fmt.Sprintf("%x", md5.Sum([]byte(in.Fromjson)))}, nil
}

func main() {
    lis, err := net.Listen("tcp", ":8028") //监听所有网卡8028端口的TCP连接
    if err != nil {
        log.Fatalf("监听失败: %v", err)
    }
    s := grpc.NewServer() //创建gRPC服务

    /**注册接口服务
     * 以定义proto时的service为单位注册，服务中可以有多个方法
     * (proto编译时会为每个service生成Register***Server方法)
     * 包.注册服务方法(gRpc服务实例，包含接口方法的结构体[指针])
     */
    test.RegisterWaitServer(s, &server{})
    /**如果有可以注册多个接口服务,结构体要实现对应的接口方法
     * user.RegisterLoginServer(s, &server{})
     * minMovie.RegisterFbiServer(s, &server{})
     */
    // 在gRPC服务器上注册反射服务
    reflection.Register(s)
    // 将监听交给gRPC服务处理
    err = s.Serve(lis)
    if err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

```go
//client.go
package main

import (
    "context"
    test "go-test/grpc/proto"
    "google.golang.org/grpc"
    "log"
    "os"
)

func main() {
    // 建立连接到gRPC服务
    conn, err := grpc.Dial("127.0.0.1:8028", grpc.WithInsecure())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    // 函数结束时关闭连接
    defer conn.Close()

    // 创建Waiter服务的客户端
    t := test.NewWaitClient(conn)

    // 模拟请求数据
    res := "test123"
    // os.Args[1] 为用户执行输入的参数 如：go run ***.go 123
    if len(os.Args) > 1 {
        res = os.Args[1]
    }

    // 调用gRPC接口
    tr, err := t.DoMD5(context.Background(), &test.Req{Fromjson: res})
    if err != nil {
        log.Fatalf("could not greet: %v", err)
    }
    log.Printf("服务端响应: %s", tr.Md5Str)
}
```
