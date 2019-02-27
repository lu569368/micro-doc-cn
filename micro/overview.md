# Micro 工具箱oolkit

> **摘要**：Micro是一个用于微服务开发的工具箱

**目录**


- [概述](#概述)
  - [特征](#特征)
  - [安装Micro](#安装micro)
  - [依赖](#依赖)
  - [服务发现](#服务发现)
    - [Consul](#consul)
  - [Protobuf](#protobuf)
  - [编写一个服务](#编写一个服务)
    - [生成模板](#生成模板)
  - [示例](#示例)
    - [服务列表](#服务列表)
    - [获取服务](#获取服务)
    - [调用服务](#调用服务)
    - [运行API](#运行api)
    - [调用API](#调用api)
  - [插件](#插件)
    - [可插入的功能](#可插入的功能)
    - [使用插件](#使用插件)
    - [编译二进制](#编译二进制)
    - [启用插件](#启用插件)

## 概述

Micro解决了构建可伸缩系统的关键需求。它采用微服务架构模式并将其转换为一套构建积木(blocks)平台的工具。Micro能够处理复杂的分布式系统并为开发者提供能够已经了解的简单抽象。


技术在不断演进，基础技术栈总是在变化，Micro是能够解决这些问题的可插拔工具箱。插入任何技术栈或潜在技术，用micro构建不会过时的系统。

### 特征

工具箱由一下功能组成：

- **API Gateway:** 一个由通过服务发现实现动态请求路由的单点入口。API网关允许你在后端构建可扩展的微服务架构并将前端服务整合到一个公共API。micro api通过服务发现和可插拔的handlers为http, grpc, websockets, publish events等提拱了强大的路由。

- **Interactive CLI:** cil通过终端(terminal)来描述，查询，并你的平台和服务直接交互。cli提供你希望了解的所有微服务命令。它还包含一个交互模式。

- **Service Proxy:** 一个基于[Go Micro](https://github.com/micro/go-micro) 和 [MUCP](https://github.com/micro/protocol) 构建的透明代理，将服务发现、负载平衡、消息编码、中间件、传输和代理插件卸载(Offload)到单个位置。将它独立运行或与你的服务一起运行

- **Service Templates:** 快速生成新的服务模板。Micro为微服务提供预定义模板。始终以相同的方式开始，高效的构建相同服务。

- **SlackOps Bot:** 一个运行在你的平台上的机器人，让你从Slack管理你的应用程序。微机器人支持ChatOps，让你能够通过消息与你的团队一起做任何事情。它还包括把创建slack命令作为可被动态发现的能力。
  
- **Web Dashboard:** Web仪表盘让你能够浏览你的服务、描述他们的端点(endpoints)、请求和响应格式、甚至直接查询。仪表盘还内建了一个cli，开发人员可以直接进入终端。

### 安装Micro

~~~ shell
go get -u github.com/micro/micro
~~~

或

~~~ shell
docker pull microhq/micro
~~~

### 依赖

Micro工具箱有两个依赖项:

- [服务发现](https://micro.mu/docs/toolkit.html#service-discovery)-用于名称解析
- [Protobuf](https://micro.mu/docs/toolkit.html#protobuf)-用于代码生成

### 服务发现

服务发现用于名称解析、路由和元数据集中。

Micro通过[go-micro](https://github.com/micro/go-micro)注册进行服务发现。MDNS是默认设置。这将无需额外配置。如果你希望更具伸缩性，请使用consul。

#### Consul

如果您想安装和运行Consul

```shell
# install
brew install consul
# run
consul agent -dev
```

通过--registry=consul 或给任何命令设置环境变量MICRO_REGISTRY=consul

```shell
# Use flag
micro --registry=consul list services

# Use env var
MICRO_REGISTRY=consul micro list services
```

有关更多服务发现插件，请参见[go-plugins](https://github.com/micro/go-plugins)。

### Protobuf

Protobuf用于代码生成。它减少了需要编写的样板代码的数量。

```shell
# install protobuf
brew install protobuf

# install protoc-gen-go
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}

# install protoc-gen-micro
go get -u github.com/micro/protoc-gen-micro
```

有关更多细节，请参见[protoc-gen-micro](https://github.com/micro/protoc-gen-micro)。

### 编写一个服务

Micro包含了新模板用来生成器用来加速应用程序的编写

有关编写服务的详细信息，请参见[go-micro](https://github.com/micro/go-micro)。

#### 生成模板

在这里，我们将使用micro new快速生成一个示例模板

指定一个相对于$GOPATH的路径

``` shell
micro new github.com/micro/example
```

命令将输出

```shell
example/
    Dockerfile	# A template docker file
    README.md	# A readme with command used
    handler/	# Example rpc handler
    main.go		# The main Go program
    proto/		# Protobuf directory
    subscriber/	# Example pubsub Subscriber
```

使用protoc编译protobuf代码

```shell
protoc --proto_path=. --micro_out=. --go_out=. proto/example/example.proto
```

现在像其他go应用程序一样运行它

```shell
go run main.go
```

### 示例

现在我们有一个使用模板生成的应用程序正在运行，让我们测试一下。

- [服务列表](#服务列表)
- [获取服务](#获取服务)
- [调用服务](#调用服务)
- [运行API](#运行api)
- [调用API](#调用api)

#### 服务列表

每个服务都注册到服务发现，所以我们应该能够找到它。

```shell
micro list services
```

输出

```shell
consul
go.micro.srv.example
topic:topic.go.micro.srv.example
```

示例应用程序已经注册了完全合格的域名go.micro.srv.example

#### 获取服务

每个注册的服务有一个唯一的id、地址和元数据。

```shell
micro get service go.micro.srv.example
```

输出

```shell
service  go.micro.srv.example

version latest

ID  Address  Port  Metadata
go.micro.srv.example-437d1277-303b-11e8-9be9-f40f242f6897	192.168.1.65	53545	transport=http,broker=http,server=rpc,registry=consul

Endpoint: Example.Call
Metadata: stream=false

Request: {
    name string
}

Response: {
    msg string
}


Endpoint: Example.PingPong
Metadata: stream=true

Request: {}

Response: {}


Endpoint: Example.Stream
Metadata: stream=true

Request: {}

Response: {}


Endpoint: Func
Metadata: subscriber=true,topic=topic.go.micro.srv.example

Request: {
    say string
}

Response: {}


Endpoint: Example.Handle
Metadata: subscriber=true,topic=topic.go.micro.srv.example

Request: {
    say string
}

Response: {}
```

#### 调用服务

通过CLI进行RPC调用。以json发送查询请求。

```shell
micro call go.micro.srv.example Example.Call '{"name": "John"}'
```

输出

```shell
{
    "msg": "Hello John"
}
```

查看[cli](https://micro.mu/docs/cli.html)文档以获取更多信息。

现在让我们测试通过HTTP调用服务

#### 运行API

micro api是一个动态路由到后端服务的http网关

让我们运行它，以便查询示例服务。

``` shell
MICRO_API_HANDLER=rpc \
MICRO_API_NAMESPACE=go.micro.srv \
micro api
```

一些信息:

- MICRO_API_HANDLER设置http handler
- MICRO_API_NAMESPACE设置服务名称空间

#### 调用API

使用json向api发出POST请求

```shell
curl -XPOST -H 'Content-Type: application/json' -d '{"name": "John"}' http://localhost:8080/example/call
```

输出

```shell
{"msg":"Hello John"}
```

有关更多信息，请参见[api文档](https://micro.mu/docs/api.html)。

### 插件

Micro是在[go-micro](https://github.com/micro/go-micro)的基础上构建的，它是一个可插拔的工具箱。

Go-micro为分布式系统提供了可被替换的基础设施抽象。

#### 可插入的功能

可插拔的micro特性:

- 经济人(broker) - 发布订阅消息的经纪人
- 注册(registry) - 服务发现
- 选择器(selector) - 客户端负载平衡
- 传输(transport) - 请求-响应或双向流
- 客户端(client) - 管理上述功能的客户端
- 服务(server) - 管理上述特性的服务

在[go-plugins](https://github.com/micro/go-plugins)中找到插件

#### 使用插件

集成go-micro插件，只需将它们链接到一个单独的文件中

创建一个plugins.go文件

```go
import (
    // etcd v3 registry
    _ "github.com/micro/go-plugins/registry/etcdv3"
    // nats transport
    _ "github.com/micro/go-plugins/transport/nats"
    // kafka broker
    _ "github.com/micro/go-plugins/broker/kafka"
)
```

#### 编译二进制

使用Go工具链重新构建微二进制文件

```go
# For local use
go build -i -o micro ./main.go ./plugins.go

# For docker image
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-w' -i -o micro ./main.go ./plugins.go
```

#### 启用插件

使用命令行参数或环境变量启用插件

``` shell
# flags
micro --registry=etcdv3 --transport=nats --broker=kafka [command]

# env vars
MICRO_REGISTRY=etcdv3 MICRO_TRANSPORT=nats MICRO_BROKER=kafka micro [command]
```