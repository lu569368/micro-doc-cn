# GRPC网关

**目录**

  - [GRPC网关](#grpc%E7%BD%91%E5%85%B3-1)
    - [代码](#%E4%BB%A3%E7%A0%81)
    - [准备](#%E5%87%86%E5%A4%87)
      - [安装 protobuf](#%E5%AE%89%E8%A3%85-protobuf)
      - [安装 plugins](#%E5%AE%89%E8%A3%85-plugins)
    - [欢迎服务](#%E6%AC%A2%E8%BF%8E%E6%9C%8D%E5%8A%A1)
    - [GRPC 网关](#grpc-%E7%BD%91%E5%85%B3)
    - [运行示例](#%E8%BF%90%E8%A1%8C%E7%A4%BA%E4%BE%8B)



## GRPC网关

本指南帮助您在go-micro服务中使用grpc网关。

[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)是[protoc](http://github.com/google/protobuf)的一个插件。它读取[gRPC](https://github.com/grpc/grpc-experiments)服务定义，并生成一个反向代理服务器，该服务器将RESTful JSON API转换为gRPC。

我们使用[go-grpc](https://github.com/micro/go-grpc)编写后台服务。Go-GRPC是go-micro和用于客户机和服务器的grpc插件的简单包装器。当调用[grpc.NewService](https://github.com/grpc/grpc-experiments)。它返回一个[micro.Service](https://godoc.org/github.com/micro/go-micro#Service)

### 代码

在[examples/grpc](https://github.com/micro/examples/tree/master/grpc)中查找示例代码。

### 准备

这些是先决条件

#### 安装 protobuf

~~~ shell

mkdir tmp
cd tmp
git clone https://github.com/google/protobuf
cd protobuf
./autogen.sh
./configure
make
make check
sudo make install

~~~

#### 安装 plugins

~~~

go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
go get -u github.com/micro/protobuf/protoc-gen-go

~~~

### 欢迎服务

在这个例子中，我们使用go-grpc创建了一个欢迎微服务。服务很简单。
proto 如下:

~~~ protobuf

syntax = "proto3";

package go.micro.srv.greeter;

service Say {
    rpc Hello(Request) returns (Response) {}
}

message Request {
    string name = 1;
}

message Response {
    string msg = 1;
}

~~~

服务代码如下:

~~~ go

package main

import (
    "log"
    "time"

    hello "github.com/micro/examples/greeter/srv/proto/hello"
    "github.com/micro/go-grpc"
    "github.com/micro/go-micro"

    "golang.org/x/net/context"
)

type Say struct{}

func (s *Say) Hello(ctx context.Context, req *hello.Request, rsp *hello.Response) error {
    log.Print("Received Say.Hello request")
    rsp.Msg = "Hello " + req.Name
    return nil
}

func main() {
    service := grpc.NewService(
        micro.Name("go.micro.srv.greeter"),
        micro.RegisterTTL(time.Second*30),
        micro.RegisterInterval(time.Second*10),
    )

    // optionally setup command line usage
    service.Init()

    // Register Handlers
    hello.RegisterSayHandler(service.Server(), new(Say))

    // Run server
    if err := service.Run(); err != nil {
        log.Fatal(err)
    }
}

~~~

### GRPC 网关

grpc网关使用与服务相同的proto ，并添加了http选项

~~~ protobuf
syntax = "proto3";

package greeter;

import "google/api/annotations.proto";

service Say {
    rpc Hello(Request) returns (Response) {
        option (google.api.http) = {
            post: "/greeter/hello"
            body: "*"
        };
    }
}

message Request {
    string name = 1;
}

message Response {
    string msg = 1;
}

~~~

proto使用以下命令生成grpc stub 和反向代理

~~~ shell
protoc -I/usr/local/include -I. \
-I$GOPATH/src \
-I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
--go_out=plugins=grpc:. \
path/to/your_service.proto 

protoc -I/usr/local/include -I. \
-I$GOPATH/src \
-I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
--grpc-gateway_out=logtostderr=true:. \
path/to/your_service.proto

~~~


我们使用下面的代码为欢迎服务创建了一个简单api。编写类似的代码来注册其他端点。注意，网关需要欢迎服务的端点地址。

~~~ go
package main

import (
    "flag"
    "net/http"

    "github.com/golang/glog"
    "github.com/grpc-ecosystem/grpc-gateway/runtime"
    "golang.org/x/net/context"
    "google.golang.org/grpc"

    hello "github.com/micro/examples/grpc/gateway/proto/hello"
)

var (
    // the go.micro.srv.greeter address
    endpoint = flag.String("endpoint", "localhost:9090", "go.micro.srv.greeter address")
)

func run() error {
    ctx := context.Background()
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()

    mux := runtime.NewServeMux()
    opts := []grpc.DialOption{grpc.WithInsecure()}

    err := hello.RegisterSayHandlerFromEndpoint(ctx, mux, *endpoint, opts)
    if err != nil {
        return err
    }

    return http.ListenAndServe(":8080", mux)
}

func main() {
    flag.Parse()

    defer glog.Flush()

    if err := run(); err != nil {
        glog.Fatal(err)
    }
}

~~~~

### 运行示例

运行欢迎服务。指定mdns，因为我们不需要发现

~~~ shell
go run examples/grpc/greeter/srv/main.go --registry=mdns --server_address=localhost:9090
~~~

运行网关。对于欢迎服务，它默认为端点localhost:9090

~~~ shell
go run examples/grpc/gateway/main.go
~~~
用Curl 请求你的网关(localhost:8080)

~~~ shell
    curl -d '{"name": "john"}' http://localhost:8080/greeter/hello
~~~

局限

示例grpc网关需要提供服务地址，而我们自己的micro api使用服务发现、动态路由和负载平衡。这使得grpc网关的集成不太通用。

访问[github.com/micro/micro](https://github.com/micro/micro)了解更多信息