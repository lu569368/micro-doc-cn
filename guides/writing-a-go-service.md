# 编写一个Go服务

**目录**

- [编写一个Go服务](#编写一个go服务-1)
  - [编写一个服务](#编写一个服务)
    - [1.初始化](#1初始化)
    - [2. 定义接口](#2-定义接口)
    - [3.生成API接口](#3生成api接口)
    - [4.实现handler](#4实现handler)
    - [5.运行服务](#5运行服务)
    - [6.完整的服务](#6完整的服务)
  - [编写客户端](#编写客户端)

## 编写一个Go服务

这是go-micro入门指南。

如果您更喜欢首先对工具箱进行更高级的概述，请查看介绍的博客文章[https://micro.mu/blog/2016/03/20/micro.html](https://micro.mu/blog/2016/03/20/micro.html)

### 编写一个服务

顶层[服务](https://godoc.org/github.com/micro/go-micro#Service)接口是构建服务的主要组件。它将Go Micro的所有底层包封装到一个适当的接口中。

~~~ go
type Service interface {
    Init(...Option)
    Options() Options
    Client() client.Client
    Server() server.Server
    Run() error
    String() string
}
~~~

#### 1.初始化

使用 micro.NewService 创建的一个这样的服务。

~~~go
import "github.com/micro/go-micro"

service := micro.NewService()
~~~

可以在创建期间传入选项。

~~~go

service := micro.NewService(
        micro.Name("greeter"),
        micro.Version("latest"),
)

~~~

所有可用的选项都可以在[这里](https://godoc.org/github.com/micro/go-micro#Option)找到。

Go Micro 还提供了一种使用 Micro.flags 设置命令行标志的方法。

~~~go

import (
        "github.com/micro/cli"
        "github.com/micro/go-micro"
)

service := micro.NewService(
        micro.Flags(
                cli.StringFlag{
                        Name:  "environment",
                        Usage: "The environment",
                },
        )
)

~~~

要解析标志，请使用service.Init。此外，访问标志使用 micro.Action 选项。

~~~go
service.Init(
        micro.Action(func(c *cli.Context) {
                env := c.StringFlag("environment")
                if len(env) > 0 {
                        fmt.Println("Environment set to", env)
                }
        }),
)
~~~

Go Micro提供了预先定义的标志，可以在service.Init调用后设置和解析。在[这里](https://godoc.org/github.com/micro/go-micro/cmd#pkg-variables)看到所有的标志。

#### 2. 定义接口

我们使用protobuf文件来定义服务API接口。这是一种非常方便的方法，可以严格定义API并为服务器和客户机提供具体类型。

下面是一个示例定义。

greeter.proto

~~~ protobuf

syntax = "proto3";

service Greeter {
    rpc Hello(HelloRequest) returns (HelloResponse) {}
}

message HelloRequest {
    string name = 1;
}

message HelloResponse {
    string greeting = 2;
}

~~~

在这里，我们使用Hello方法定义一个名为Greeter的服务处理程序，该方法接受参数HelloRequest类型并返回HelloResponse。

#### 3.生成API接口

您将需要以下工具来生成protobuf代码

- [protoc](https://github.com/google/protobuf)
- [protoc-gen-go](https://github.com/golang/protobuf)
- [protoc-gen-micro](https://github.com/micro/protoc-gen-micro)

我们使用protoc、protoc-gen-go和protoc-gen-micro来为这个定义生成具体的go实现。

~~~ shell

go get github.com/golang/protobuf/{proto,protoc-gen-go}
~~~

~~~ shell

go get github.com/micro/protoc-gen-micro
~~~

~~~ shell

protoc --proto_path=$GOPATH/src:. --micro_out=. --go_out=.greeter.proto
~~~

生成的类型现在可以导入，并在发出请求时在服务器或客户端的handler中使用。

下面是生成的代码的一部分。

~~~go

type HelloRequest struct {
    Name string `protobuf:"bytes,1,opt,name=name" json:"name,omitempty"`
}

type HelloResponse struct {
    Greeting string `protobuf:"bytes,2,opt,name=greeting" json:"greeting,omitempty"`
}

// Client API for Greeter service

type GreeterClient interface {
    Hello(ctx context.Context, in *HelloRequest, opts ...client.CallOption) (*HelloResponse, error)
}

type greeterClient struct {
    c           client.Client
    serviceName string
}

func NewGreeterClient(serviceName string, c client.Client) GreeterClient {
    if c == nil {
        c = client.NewClient()
    }
    if len(serviceName) == 0 {
        serviceName = "greeter"
    }
    return &greeterClient{
        c:           c,
        serviceName: serviceName,
    }
}

func (c *greeterClient) Hello(ctx context.Context, in *HelloRequest, opts ...client.CallOption) (*HelloResponse, error) {
    req := c.c.NewRequest(c.serviceName, "Greeter.Hello", in)
    out := new(HelloResponse)
    err := c.c.Call(ctx, req, out, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

// Server API for Greeter service

type GreeterHandler interface {
    Hello(context.Context, *HelloRequest, *HelloResponse) error
}

func RegisterGreeterHandler(s server.Server, hdlr GreeterHandler) {
    s.Handle(s.NewHandler(&Greeter{hdlr}))
}

~~~

#### 4.实现handler

服务器需要注册handlers来服务请求。handlers 是具有符合 func(ctx context.Context, req interface{}, rsp interface{}) error 公共方法的公共类型。

正如您在上面看到的，Greeter接口的handler签名看起来是这样的。

~~~go
type GreeterHandler interface {
    Hello(context.Context, *HelloRequest, *HelloResponse) error
}
~~~

该处理程序在您的服务中注册，非常类似于http.Handler。

~~~go
service := micro.NewService(
    micro.Name("greeter"),
)

proto.RegisterGreeterHandler(service.Server(), new(Greeter))
~~~

您还可以创建一个双向流处理程序，但我们将把这个问题留到以后讨论。

#### 5.运行服务

可以通过调用server.Run来运行服务。这将导致服务绑定到配置中的地址(默认为找到的第一个RFC1918接口和一个随机端口)，并监听请求。

这将在启动时向注册中心注册服务，并在发出终止信号时取消注册。

~~~go
if err := service.Run(); err != nil {
    log.Fatal(err)
}
~~~

#### 6.完整的服务

greeter.go

~~~go

package main

import (
        "log"

        "github.com/micro/go-micro"
        proto "github.com/micro/examples/service/proto"

        "golang.org/x/net/context"
)

type Greeter struct{}

func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error {
        rsp.Greeting = "Hello " + req.Name
        return nil
}

func main() {
        service := micro.NewService(
                micro.Name("greeter"),
                micro.Version("latest"),
        )

        service.Init()

        proto.RegisterGreeterHandler(service.Server(), new(Greeter))

        if err := service.Run(); err != nil {
                log.Fatal(err)
        }
}

~~~

请注意。服务发现机制需要运行，这样服务才能被注册，客户端和其他服务才能发现他。[这里](https://github.com/micro/go-micro#getting-started)是一个快速入门。

### 编写客户端

[客户端](https://godoc.org/github.com/micro/go-micro/client)包用于查询服务。在你创建服务时，这个客户端包含在服务初始化使用过，匹配的包中。

查询上述服务非常简单，如下所示。

~~~go
// create the greeter client using the service name and client
greeter := proto.NewGreeterClient("greeter", service.Client())

// request the Hello method on the Greeter handler
rsp, err := greeter.Hello(context.TODO(), &proto.HelloRequest{
    Name: "John",
})
if err != nil {
    fmt.Println(err)
    return
}

fmt.Println(rsp.Greeter)
~~~

proto.NewGreeterClient 使用的服务名称和客户端(client)用于发出请求。

完整的例子可以在[go-micro/examples/service](https://github.com/micro/examples/tree/master/service)上找到。