#### 安装

###### 安装Protobuf的编译器protoc

```go
wget https://github.com/google/protobuf/releases/download/v3.6.1/protobuf-all-3.6.1.tar.gz 
tar zxvf protobuf-all-3.6.1.tar.gz
cd protobuf-3.6.1
./configure
make
make install
protoc   -h
protoc --version
```

###### 获取goprotobuf提供的插件 protoc-gen-go(放到$GOPATH/bin下)

```cmd
go get github.com/golang/protobuf/protoc-gen-go

cd github.com/golang/protobuf/protoc-gen-go

go build

go install

vi /etc/profile 将$GOPATH/bin 加入环境变量

source profile
```

###### 获取goprotobuf提供的支持库，包含编码（marshaling），解码(unmarshaling)的功能

```go
go get github.com/golang/protobuf/proto

cd github.com/golang/protobuf/proto

go build

go install
```

###### 使用语法：

protoc -I.

-I 代表 -IPATH。在哪个路径下搜索导入目录
