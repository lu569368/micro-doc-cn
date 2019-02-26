# Go Micro

> 摘要：Go Micro是用来开发微服务的框架。

**目录**

- [概述](#概述)
- [特征](#特征)
- [开始](#开始)
    - [安装Protobuf](#安装protobuf)
    - [服务发现](#服务发现)
        - [consul](#consul)
    - [编写一个服务](#编写一个服务)
        - [创建proto服务](#创建proto服务)
        - [生成proto](#生成proto)
        - [编写服务](#编写服务)
        - [运行服务](#运行服务)
            - [定义一个客户端](#定义一个客户端)
        - [运行客户端](#运行客户端)
    - [编写一个函数](#编写一个函数)
        - [定义函数](#定义函数)
    - [发布订阅](#发布订阅)
        - [发布](#发布)
        - [订阅](#订阅)
    - [插件](#插件)
        - [通过插件编译](#通过插件编译)
        - [插件选项](#插件选项)
        - [编写插件](#编写插件)
    - [包装器](#包装器)
        - [客户端](#客户端)
    - [示例](#示例)
    - [其他语言](#其他语言)
    - [赞助](#赞助)

## 概述

Go Micro 满足分布式系统的核心需求，包括RPC和事件驱动的通信。**Micro** 的设计哲学是健全的可插拔结构。我们提供了默认设置但是所有设置可以轻松替换。

## 特征

Go Micro 抽象了分布式系统的细节。下面是主要特征。

- **服务发现** - 自动服务注册与名称解析（name resolution）。服务发现是微服务开发的核心。当服务A需要和服务B对话时，它需要服务B的位置。默认的发现机制是组播DNS (mdns)，一个零配置系统。你可以轻松的设置基于p2p网络使用SWIM协议的gossip或基于弹性云原生（cloud-native）的consul（You can optionally set gossip using the SWIM protocol for p2p networks or consul for a resilient cloud-native setup.）。

- **负载均衡** - 客户端负载均衡基于服务发现。一旦我们发现了一定数量的服务地址，我们需要决定路由到其中一个节点。我们通过哈希散列负载均衡算法提供平均的服务分发并且当一个节点出现故障时会重试另一个。

- **消息编码** - 基于内容息类型（content-type）的动态消息编码。客户端和服务端将通过编解码器连同消息类型无缝的为你编码和解码go的类型。各种的消息可以被编码并被不同客户端发送，客户端和服务端会自动的处理，默认支持protobuf 和json。

- **请求/响应** - RPC 基于双向流的请求/响应。我们会提供一个抽象的同步通信，向服务发送的请求将会被自动解析、负载均衡、拨号和流处理(streamed)。默认采用http/1.1或当tls可用时采用http2。

- **异步消息** - 订阅作为一等公民，是通过异步通信和事件驱动结构构建的.事件通知是微服务开发的核心模式。消息默认基于点对点http/1.1或当tls可用时基于http/2。

- **可插拔接口** - Go Micro 使用go接口作为每一个分布式系统提供的抽象。以为这些接口是可插拔并且与运行无关，所以你可以安装任何潜在的插件。插件下载地址[github.com/micro/go-plugins](ht+tps://github.com/micro/go-plugins)。

## 开始

有关go-micro的架构、安装和使用的详细信息，请参考 [原文文档](https://micro.mu/docs/go-micro.html)


- [安装Protobuf](#安装Protobuf)
- [服务发现](#服务发现)
- [编写一个服务](#编写一个服务)
- [编写一个函数](#编写一个函数)
- [发布订阅](#发布订阅)
- [插件](#插件)
- [包装器](#包装器)
- [示例](#示例)

### 安装Protobuf

代码生成需要Protobuf
你需要安装：
  - [protoc-gen-micro](https://github.com/micro/protoc-gen-micro)

### 服务发现

服务发现用于将服务名称解析为地址。
默认的发现系统是基于组播DNS(multicast DNS),无需额外配置。如过你需要更好的弹性和多主机，那么可以使用consul。
#### consul

[consul](https://www.consul.io) 被作为默认的服务发现系统。
发现系统是可插拔的。在[[micro/go-plugins](https://github.com/micro/go-plugins)]中可以找到 etcd, kubernetes, zookeeper 或其他插件

[安装指南](https://learn.hashicorp.com/consul/getting-started/install.html)

通过命令加参数 <font color="#000">-registry=consul</font> 或在环境变量中写 <font color="#000">MICRO_REGISTRY=consul</font>的方式使用

    MICRO_REGISTRY=consul go run main.go

### 编写一个服务

这是一个简单的欢迎RPC服务示例
在[examples/service](https://github.com/micro/examples/tree/master/service)可以找到这个列子

#### 创建proto服务

微服务的关键需求之一是强定义接口。Micro使用protobuf来实现这一点。

在这里，我们使用Hello方法定义了Greeter处理程序。它需要一个HelloRequest和HelloResponse，都有一个字符串参数。
在这里，我们定义了带有Hello方法的Greeter handler。它需要一个HelloRequest和HelloResponse，都有一个字符串参数。

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

#### 生成proto

当编写好proto定义后，我们必须通过protoc和micro plugin来编译它。

~~~ shell

protoc --proto_path=$GOPATH/src:. --micro_out=. --go_out=. path/to/greeter.proto

~~~

#### 编写服务

下面是欢迎服务的代码。

它执行以下操作：

  1. 实现在Greeter handler定义的接口
  2. 初始化一个微服务
  3. 注册Greeter handler
  4. 运行服务

~~~ go

package main
import (
    "context"
    "fmt"

    micro "github.com/micro/go-micro"
    proto "github.com/micro/examples/service/proto"
)
type Greeter struct{}

func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error {
    rsp.Greeting = "Hello " + req.Name
    return nil
}

func main() {
    // 创建一个新服务。这里有选择性的包含一些选项。
    service := micro.NewService(
        micro.Name("greeter"),
    )

    // Init将解析命令行参数。
    service.Init()

    // 注册 handler
    proto.RegisterGreeterHandler(service.Server(), new(Greeter))

    // 运行服务
    if err := service.Run(); err != nil {
        fmt.Println(err)
    }
}

~~~

#### 运行服务

~~~ shell
go run examples/service/main.go
~~~

输出

~~~ shell
2016/03/14 10:59:14 Listening on [::]:50137
2016/03/14 10:59:14 Broker Listening on [::]:50138
2016/03/14 10:59:14 Registering node: greeter-ca62b017-e9d3-11e5-9bbb-68a86d0d36b6
~~~

##### 定义一个客户端

下面是查询欢迎服务的客户端代码。
生成的proto 包含一个迎客户端（greeter client ）来减少样板代码编写。

~~~ go
package main

import (
    "context"
    "fmt"

    micro "github.com/micro/go-micro"
    proto "github.com/micro/examples/service/proto"
)


func main() {
    // 创建一个新服务。这里有选择性的包含一些选项。
    service := micro.NewService(micro.Name("greeter.client"))
    service.Init()

    // 创建一个欢迎客户端
    greeter := proto.NewGreeterService("greeter", service.Client())

    // 调用欢迎方法
    rsp, err := greeter.Hello(context.TODO(), &proto.HelloRequest{Name: "John"})
    if err != nil {
        fmt.Println(err)
    }

    // 打印响应结果
    fmt.Println(rsp.Greeting)
}
~~~

#### 运行客户端

~~~ shell
go run client.go
~~~

输出

~~~ shell
    Hello John
~~~

### 编写一个函数

Go Micro包括函数编程模型。

函数是在完成请求后退出的只执行一次行服务。

#### 定义函数

~~~ go

    package main

    import (
        "context"

        proto "github.com/micro/examples/function/proto"
        "github.com/micro/go-micro"
    )

    type Greeter struct{}

    func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error {
        rsp.Greeting = "Hello " + req.Name
        return nil
    }

    func main() {
        // create a new function
        fnc := micro.NewFunction(
            micro.Name("greeter"),
        )

        // init the command line
        fnc.Init()

        // register a handler
        fnc.Handle(new(Greeter))

        // run the function
        fnc.Run()
    }

~~~

就这么简单。

### 发布订阅

Go-micro为事件驱动架构提供了一个内置的message broker接口。

发布订阅的RPC操作基于相同protobuf生成的消息。它们自动编码/解码，并通过代理发送。默认情况下，go-micro包括一个点对点的http代理，但这可以通过go-plugins替换。

#### 发布

通过主题(topic)名称和service client创建一个新的publisher 

~~~ go
p := micro.NewPublisher("events", service.Client())
~~~

发布一个proto消息

~~~ go
p.Publish(context.TODO(), &proto.Event{Name: "event"})
~~~
#### 订阅

创建消息处理程序。它的签名(signature)应该是  func(context.Context, v interface{}) error。

~~~ go

    func ProcessEvent(ctx context.Context, event *proto.Event) error {
        fmt.Printf("Got event %+v\n", event)
        return nil
    }

~~~

注册message handler并带topic参数

~~~ go
    micro.RegisterSubscriber("events", ProcessEvent)
~~~

有关完整示例，请参见示例[examples/pubsub](https://github.com/micro/examples/tree/master/pubsub)

### 插件

默认情况下，go-micro 只提供少量核心接口的实现，但是完全可以通过插件是实现。在
[ github.com/micro/go-plugins](https://github.com/micro/go-plugins)已经提供了几十个可用的插件。欢迎贡献！

#### 通过插件编译

如果您想集成插件，只需将它们链接到一个单独的文件中并重新编译即可
创建一个插件 .go 文件

~~~ go

import (
        // etcd v3 registry
        _ "github.com/micro/go-plugins/registry/etcdv3"
        // nats transport
        _ "github.com/micro/go-plugins/transport/nats"
        // kafka broker
        _ "github.com/micro/go-plugins/broker/kafka"
)

~~~

编译二进制

~~~ shell

// For local use
go build -i -o service ./main.go ./plugins.go

~~~

插件参数的用例

~~~ shell

service --registry=etcdv3 --transport=nats --broker=kafka

~~~

#### 插件选项

或者，您可以将插件设置为服务的选项
~~~ go

import (
        "github.com/micro/go-micro" 
        // etcd v3 registry
        "github.com/micro/go-plugins/registry/etcdv3"
        // nats transport
        "github.com/micro/go-plugins/transport/nats"
        // kafka broker
        "github.com/micro/go-plugins/broker/kafka"
)

func main() {
    registry := etcdv3.NewRegistry()
    broker := kafka.NewBroker()
    transport := nats.NewTransport()

        service := micro.NewService(
                micro.Name("greeter"),
                micro.Registry(registry),
                micro.Broker(broker),
                micro.Transport(transport),
        )

    service.Init()
    service.Run()
}

~~~

#### 编写插件

插件是建立在Go接口基础上的概念。每个包维护一个高级接口抽象。只需实现接口并将其作为选项传递给服务。

~~~ go

type Registry interface {
    Register(*Service, ...RegisterOption) error
    Deregister(*Service) error
    GetService(string) ([]*Service, error)
    ListServices() ([]*Service, error)
    Watch() (Watcher, error)
    String() string
}

~~~

浏览[go-plugins](https://github.com/micro/go-plugins)以更好地了解实现细节。

### 包装器

Go-micro包含了中间件作为包装器的概念。客户端或者处理程序handlers 可以通过装饰器模式被包装起来。

Handler

下面是记录传入请求的服务handler包装器样例

~~~ go

    // implements the server.HandlerWrapper
    func logWrapper(fn server.HandlerFunc) server.HandlerFunc {
        return func(ctx context.Context, req server.Request, rsp interface{}) error {
            fmt.Printf("[%v] server request: %s", time.Now(), req.Endpoint())
            return fn(ctx, req, rsp)
        }
    }

~~~

它可以在创建服务时初始化

~~~ go

service := micro.NewService(
    micro.Name("greeter"),
    // wrap the handler
    micro.WrapHandler(logWrapper),
)

~~~

#### 客户端

下面是一个记录请求的客户端包装器示例

~~~ go

type logWrapper struct {
    client.Client
}

func (l *logWrapper) Call(ctx context.Context, req client.Request, rsp interface{}, opts ...client.CallOption) error {
    fmt.Printf("[wrapper] client request to service: %s endpoint: %s\n", req.Service(), req.Endpoint())
    return l.Client.Call(ctx, req, rsp)
}

// implements client.Wrapper as logWrapper
func logWrap(c client.Client) client.Client {
    return &logWrapper{c}
}

它可以在创建服务时初始化

service := micro.NewService(
    micro.Name("greeter"),
    // wrap the client
    micro.WrapClient(logWrap),
)

~~~

### 示例

示例服务可以在[examples/service](https://github.com/micro/examples/tree/master/service)服务中找到，而函数可以在[ examples/function](https://github.com/micro/examples/tree/master/function)函数中找到。

[示例](https://github.com/micro/examples)目录包含使用中间件/包装器、选择器过滤器、发布/订阅、grpc、插件等的示例。关于完整的欢迎示例，请看[examples/greeter](https://github.com/micro/examples/tree/master/greeter)欢迎。其他示例可以在GitHub仓库中找到。

观看Golang UK Conf 2016视频获得一个更深入的了解。

### 其他语言

查看[Java-micro](https://github.com/Sixt/ja-micro)以编写Java服务

### 赞助

Sixt 是Micro的赞助公司

<a href="https://micro.mu/blog/2016/04/25/announcing-sixt-sponsorship.html"><img src="https://micro.mu/sixt_logo.png" width=150px height="auto" /></a>
