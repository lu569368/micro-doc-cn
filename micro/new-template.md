# 新建模板

**目录**

- [用例](#用例)
  - [选项](#选项)
  - [帮助](#帮助)

## micro 新建[服务]

micro new命令是一种快速生成微服务模板的方法。

### 用例

通过指定相对于$GOPATH的目录路径来创建新服务

``` shell
micro new github.com/micro/foo
```

这就是实际情况

``` shell
micro new github.com/micro/foo

creating service go.micro.srv.foo
creating /Users/asim/checkouts/src/github.com/micro/foo
creating /Users/asim/checkouts/src/github.com/micro/foo/main.go
creating /Users/asim/checkouts/src/github.com/micro/foo/handler
creating /Users/asim/checkouts/src/github.com/micro/foo/handler/example.go
creating /Users/asim/checkouts/src/github.com/micro/foo/subscriber
creating /Users/asim/checkouts/src/github.com/micro/foo/subscriber/example.go
creating /Users/asim/checkouts/src/github.com/micro/foo/proto/example
creating /Users/asim/checkouts/src/github.com/micro/foo/proto/example/example.proto
creating /Users/asim/checkouts/src/github.com/micro/foo/Dockerfile
creating /Users/asim/checkouts/src/github.com/micro/foo/README.md

download protobuf for micro:

go get github.com/micro/protobuf/{proto,protoc-gen-go}

compile the proto file example.proto:

protoc -I/Users/asim/checkouts/src \
    --go_out=plugins=micro:/Users/asim/checkouts/src \
    /Users/asim/checkouts/src/github.com/micro/foo/proto/example/example.proto
```

#### 选项

指定更多选项，如命名空间、类型、fqdn和别名

``` shell
micro new --fqdn com.example.srv.foo github.com/micro/foo
```

#### 帮助

``` shell

NAME:
   micro new - Create a new micro service

USAGE:
   micro new [command options] [arguments...]

OPTIONS:
   --namespace "go.micro"    Namespace for the service e.g com.example
   --type "srv"              Type of service e.g api, srv, web
   --fqdn                    FQDN of service e.g com.example.srv.service (defaults to namespace.type.alias)
   --alias                   Alias is the short name used as part of combined name if specified

```