# 编写一个Go方法

## 编写一个Go方法

这是一个入门go-micro函数的指南。函数是一次执行服务。

如果您更喜欢先了解对工具包进行更高级的概述，请查看介绍的博客文章[https://micro.mu/blog/2016/03/20/micro.html](https://micro.mu/blog/2016/03/20/micro.html)。

### 写一个函数

顶层[函数](https://github.com/micro/go-micro#getting-started)接口是go-micro函数编程模型的主要组成部分。它封装服务接口，同时提供一次性执行。

~~~go

// Function is a one time executing Service
type Function interface {
    // Inherits Service interface
    Service
    // Done signals to complete execution
    Done() error
    // Handle registers an RPC handler
    Handle(v interface{}) error
    // Subscribe registers a subscriber
    Subscribe(topic string, v interface{}) error
}

~~~

#### 1.初始化

使用micro.NewFunction创建一个这样的函数。

~~~go

import "github.com/micro/go-micro"

function := micro.NewFunction() 

~~~

可以在创建期间传入选项。

~~~go

function := micro.NewFunction(
        micro.Name("greeter"),
        micro.Version("latest"),
)

~~~

所有可用的选项都可以在[这里](https://godoc.org/github.com/micro/go-micro#Option)找到。

Go Micro还提供了一种使用Micro.flags设置命令行标志的方法。

~~~ go

import (
        "github.com/micro/cli"
        "github.com/micro/go-micro"
)

function := micro.NewFunction(
        micro.Flags(
                cli.StringFlag{
                        Name:  "environment",
                        Usage: "The environment",
                },
        )
)

~~~

要解析标志，请使用function.Init。此外，访问标志使用micro.Action选项。

~~~ go

function.Init(
        micro.Action(func(c *cli.Context) {
                env := c.StringFlag("environment")
                if len(env) > 0 {
                        fmt.Println("Environment set to", env)
                }
        }),
)

~~~

Go Micro提供了预定义的标志，当function.Init被调用后，可以解析和设置标志。在[这里](https://godoc.org/github.com/micro/go-micro/cmd#pkg-variables)看到所有的标志。

#### 2.定义API

我们使用protobuf文件来定义API接口。这是一种非常方便的方法，可以严格定义API并为服务器和客户机提供具体类型。

下面是一个示例定义。

~~~ protobuf

greeter.proto

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

在这里，我们使用Hello方法定义一个名为Greeter的函数handler，该方法接受参数HelloRequest类型并返回HelloResponse

#### 3.生成API接口

我们使用protoc和protoc-gen-go来为这个定义生成具体的go实现。

Go-micro使用代码生成提供客户端stub方法来减少样板代码，就像gRPC一样。这是通过一个protobuf插件完成的，它需要一个[golang/protobuf](https://github.com/golang/protobuf)的分支，可以在[github.com/micro/protobuf](https://micro.mu/docs/github.com/micro/protobuf)找到。

~~~ shell
go get github.com/micro/protobuf/{proto,protoc-gen-go}
protoc --go_out=plugins=micro:. greeter.proto
~~~

生成的类型现在可以导入，并在发出请求时在服务器或客户机的处理程序中使用。

下面是生成的代码的一部分。

~~~ go

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

#### 4. 实现handlergeix

服务器需要注册handler来服务请求。handlers 是具有符合 func(ctx context.Context, req interface{}, rsp interface{}) error 签名(signature)公共方法的公共类型。

正如您在上面看到的，Greeter接口的handler签名(signature )看起来是这样的。

~~~ go

type GreeterHandler interface {
    Hello(context.Context, *HelloRequest, *HelloResponse) error
}

~~~

下面是一个Greeter handler的实现。

~~~ go

import proto "github.com/micro/examples/service/proto"

type Greeter struct{}

func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error {
    rsp.Greeting = "Hello " + req.Name
    return nil
}

~~~

handler注册起来很像http.Handler。

~~~ go

function := micro.NewFunction(
    micro.Name("greeter"),
)

proto.RegisterGreeterHandler(service.Server(), new(Greeter))

~~~

另外，函数接口提供了更简单的注册模方式。

~~~ go

function := micro.NewFunction(
        micro.Name("greeter"),
)

function.Handle(new(Greeter))

~~~

您还可以使用订阅方法注册一个异步订阅器。

#### 5.运行函数

该函数可以通过调用function. run来运行。这将导致它绑定到配置中的地址(默认为找到的第一个RFC1918接口和一个随机端口)，并侦听请求。

这将在启动时向注册表注册函数，并在发出终止信号时注销注册表。

~~~ go

if err := function.Run(); err != nil {
    log.Fatal(err)
}

~~~

在处理完请求后，函数将退出。您可以使用[micro run](https://micro.mu/docs/run.html)来管理函数的生命周期。完整的示例可以在[examples/function](https://github.com/micro/examples/tree/master/function)中找到。

#### 6.完整的函数

greeter.go

~~~ go

package main

import (
    "log"

    "github.com/micro/go-micro"
    proto "github.com/micro/examples/function/proto"

    "golang.org/x/net/context"
)

type Greeter struct{}

func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error {
    rsp.Greeting = "Hello " + req.Name
    return nil
}

func main() {
    function := micro.NewFunction(
            micro.Name("greeter"),
            micro.Version("latest"),
    )

    function.Init()

    function.Handle(new(Greeter))

    if err := function.Run(); err != nil {
            log.Fatal(err)
    }
}

~~~

请注意。服务发现机制需要运行，这样函数才能被注册 ，他才能被发现查询。[这里](https://github.com/micro/go-micro#getting-started)是一个快速入门。

### 编写一个客户端

[客户端](https://godoc.org/github.com/micro/go-micro/client)包用于查询功能和服务。当您创建一个函数时，被服务使用初始化的包中，包含了一个匹配的客户端。

查询上面的函数非常简单，如下所示。

~~~ go

// create the greeter client using the service name and client
greeter := proto.NewGreeterClient("greeter", function.Client())

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

proto.NewGreeterClien 使用名称和客户端发起请求。

完整的例子可以在[go-micro/examples/function](https://github.com/micro/examples/tree/master/function)中找到。