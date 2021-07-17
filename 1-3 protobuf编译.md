# protobuf基本编译

protobuf编译是通过编译器protoc进行编译，编译器安装在其他的md中已经说明，通过这个编译器，我们可以把.proto文件生成**go**,Java,Python,C++, Ruby, JavaNano, Objective-C,或者C# 代码，生成命令如下：

```shell
 protoc --proto_path=IMPORT_PATH --go_out=DST_DIR  path/to/file.proto
```

> - --proto_path=IMPORT_PATH，IMPORT_PATH是 .proto 文件所在的路径，如果忽略则默认当前目录。如果有多个目录则可以多次调用--proto_path，它们将会顺序的被访问并执行导入。
>
> - --go_out=DST_DIR， 指定了生成的go语言代码文件放入的文件夹
>
> - 允许使用 `protoc --go_out=./   *.proto` 的方式一次性编译多个 .proto 文件
> - go语言编译时，protobuf 编译器会把 .proto 文件编译成 .pd.go 文件

简单指令：

```shell
protoc --go_out=./ *.proto
```

> 编译当前文件夹下的所有.proto文件，并把生成的go文件放置在当前文件夹下。
> 编译如上命令报错：

```shell

protoc-gen-go: unable to determine Go import path for "example.proto"

Please specify either:
• a "go_package" option in the .proto source file, or
• a "M" argument on the command line.

See https://developers.google.com/protocol-buffers/docs/reference/go-generated#package for more information.

--go_out: protoc-gen-go: Plugin failed with status code 1.\
```

> 解决方案：option go_package = "/";加入到example.proto中即可，再次运行命令在pb文件夹下生成了example.pb.go文件
> **编译生成的go文件不要编辑**
> **此编译方式只生成消息相关的结构体变量及一些函数，没有生成rpc服务，如果要编译生成相关的服务，需要用gRPC指令**

```shell
protoc --go_out=plugins=grpc:. *.proto
--go_out: protoc-gen-go: plugins are not supported; use 'protoc --go-grpc_out=...' to generate gRPC

See https://grpc.io/docs/languages/go/quickstart/#regenerate-grpc-code for more information.
```

> - 报错后按照说明，进行调整，编译时指令改为：

```shell
protoc --go-grpc_out=./ *.proto
2021/06/28 19:09:11 WARNING: Malformed 'go_package' option in "example.proto", please specify:
	option go_package = "/;_";
A future release of protoc-gen-go will reject this.
See https://developers.google.com/protocol-buffers/docs/reference/go-generated#package for more information.

2021/06/28 19:09:11 WARNING: Malformed 'go_package' option in "example.proto", please specify:
	option go_package = "/;_";
A future release of protoc-gen-go will reject this.
See https://developers.google.com/protocol-buffers/docs/reference/go-generated#package for more information.

2021/06/28 19:09:11 WARNING: Malformed 'go_package' option in "example.proto", please specify:
	option go_package = "/;_";
A future release of protoc-gen-go will reject this.
See https://developers.google.com/protocol-buffers/docs/reference/go-generated#package for more information.
```

> - 此时将proto文件中option go_package改为：option go_package = "./;pb"; 报错消失
> - 参阅相关文档:<https://grpc.io/docs/languages/go/quickstart/> 如下命令为生成go文件的相关指令

```shell
 protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

> - This will regenerate the helloworld/helloworld.pb.go and helloworld/helloworld_grpc.pb.go files, which contain:
> - Code for populating, serializing, and retrieving HelloRequest and HelloReply message types.
Generated client and server code.

- 执行如下命令后，pb文件夹同事生成了example_grpc.pb.go和example.pb.go分别是生成客户端、服务端代码和序列化的message type.

```shell
protoc --go_out=. --go-grpc_out=. *.proto
```
