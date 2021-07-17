# Protobuf macOs 安装

```shell
brew install protobuf
```

测试是否安装成功：

```shell
protoc --version

libprotoc 3.14.0`
```

## gRPC--go语言插件安装

- Install the protocol compiler plugins for Go using the following commands:

```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
#出现以下报错：
go: modules disabled by GO111MODULE=off; see 'go help modules'
#改env后重新安装
go env -w GO111MODULE="on"
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
```

- Update your `PATH` so that the `protoc` compiler can find the plugins:

```shell
export PATH="$PATH:$(go env GOPATH)/bin"
```

## 测试gRPC是否安装成功

- git clone the example of grpc-go

  ```shell
  git clone -b v1.35.0 https://github.com/grpc/grpc-go
  
  Cloning into 'grpc-go'...
  remote: Enumerating objects: 26571, done.
  remote: Counting objects: 100% (215/215), done.
  remote: Compressing objects: 100% (150/150), done.
  remote: Total 26571 (delta 104), reused 148 (delta 61), pack-reused 26356
  Receiving objects: 100% (26571/26571), 14.36 MiB | 13.34 MiB/s, done.
  Resolving deltas: 100% (16520/16520), done.
  Note: switching to '577eb696279ea85069a02c9a4c2defafdab858c5'.
  
  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by switching back to a branch.
  
  If you want to create a new branch to retain commits you create, you may
  do so (now or later) by using -c with the switch command. Example:
  
    git switch -c <new-branch-name>
  
  Or undo this operation with:
  
    git switch -
  
  Turn off this advice by setting config variable advice.detachedHead to false
  ```

- Test  grpc demo

  ```shell
  cd grpc-go/examples/helloworld
  go run greeter_server/main.go
  # open another terminal and run the client example
  go run greeter_client/main.go
  Greeting: Hello world
  ```
