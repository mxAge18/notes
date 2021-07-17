# grpc安装

## 文档

<https://grpc.io/docs/languages/go/quickstart/>

## github

<https://github.com/grpc/grpc-go>

- 之前在protobuf安装的时候测试grpc时，采用git clone的方式下载过源代码测试。
- 现使用go get方式安装包。
- tips:国内可能不可用，需翻墙？

```shell
go get -u google.golang.org/grpc
```

## demo

测试grpc使用，根据官方的测试和学习建立简单的客户端和服务器service。

### proto buffer

> 1.编写proto文件

- pb/persion.proto

```ptoro
syntax = "proto3"; //指定版本信息
package pb; //后期生成go文件的包名
option go_package = "./;pb";

message Teacher{
    //   名字
    string name = 1;
    //   年龄
    int32  age = 2 ;
}

//定义RPC服务
service TeacherService {
    rpc SayHi (Teacher)returns (Teacher);
}

```

> 2.编译生成go文件

```shell
protoc --go_out=. --go-grpc_out=. *.proto
// 正确生成两个文件即success
```

### server

> - 新建./grpc-go-teacher_service_demo作为存储简单服务器和客户端代码的包
> - 新建./grpc-go-teacher_service_demo/server/main.go

```go

type server struct {
}

func (this *server) SayHi(ctx context.Context, in *pb.Teacher) (*pb.Teacher, error) {
}

func main() {
    lis, err := net.Listen()
    s := grpc.NewServer()
    pb.RegisterTeacherServiceServer(s, &server{})
    s.Serve(lis)
}
```

> - 列出了主要的一些步骤：
> - 1.定义结构体，并实现服务的SayHi方法
> - 2.主函数利用net包的Listen函数建立一个管道监听对应端口的连接
> - 3.grpc.NewServer新建服务，并用pb.RegisterTeacherServiceServer注册服务
> - 4.new server服务在管道上监听，Serve(lis)

### client

> - 新建./grpc-go-teacher_service_demo/client/main.go

```go

    conn, err := grpc.Dial(address,grpc.WithInsecure(), grpc.WithBlock())
    defer conn.Close()
    c := pb.NewTeacherServiceClient(conn)
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    r, err := c.SayHi(ctx, &pb.Teacher{Name: name})
```

> - 列出了主要的一些步骤：
> - 1.grpc.Dial建立连接
> - 2.pb.NewTeacherServiceClient得到一个客户端连接的管道
> - 3.c.SayHi访问服务器的SayHi函数，并返回result

### 运行测试

运行代码测试demo是否成功

#### server.go

go run server/main.go

遇到了一些问题：
package go_code/grpc-go is not in GOROOT (/usr/local/go/src/go_code/grpc-go)
解决方案：<https://github.com/grpc/grpc-go>中写了一些关于安装的问题，于是启用了go mod进行包管理，再次运行无问题。

```shell
go mod edit -replace=google.golang.org/grpc=github.com/grpc/grpc-go@latest
go mod tidy
go mod vendor
go build -mod=vendor
```

#### client.go

```shell
go run client/main.go

2021/06/29 15:15:17 result: Hi world
```
