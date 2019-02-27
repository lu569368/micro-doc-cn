# API网关

> **摘要：**micro api是一个api网关

**目录**

  - [概述](#%E6%A6%82%E8%BF%B0)
  - [安装](#%E5%AE%89%E8%A3%85)
  - [运行](#%E8%BF%90%E8%A1%8C)
  - [使用ACME](#%E4%BD%BF%E7%94%A8acme)
  - [设置TLS证书](#%E8%AE%BE%E7%BD%AEtls%E8%AF%81%E4%B9%A6)
  - [设置命名空间](#%E8%AE%BE%E7%BD%AE%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4)
  - [示例](#%E7%A4%BA%E4%BE%8B)
    - [运行示例](#%E8%BF%90%E8%A1%8C%E7%A4%BA%E4%BE%8B)
    - [查询](#%E6%9F%A5%E8%AF%A2)
  - [API](#api)
  - [Handlers](#handlers)
    - [API Handler](#api-handler)
    - [RPC Handler](#rpc-handler)
    - [Proxy Handler](#proxy-handler)
    - [Event Handler](#event-handler)
    - [Web Handler](#web-handler)
    - [调用示例](#%E8%B0%83%E7%94%A8%E7%A4%BA%E4%BE%8B)
  - [分解器（Resolver）](#%E5%88%86%E8%A7%A3%E5%99%A8resolver)
    - [RPC Resolver](#rpc-resolver)
    - [代理(proxy)解析器](#%E4%BB%A3%E7%90%86proxy%E8%A7%A3%E6%9E%90%E5%99%A8)

使用API网关[模式](https://microservices.io/patterns/apigateway.html)为您的服务提供单个公共入口点。micro api提供HTTP服务和通过服务发现实现的动态路由。

<img src="https://micro.mu/docs/images/api.png">

## 概述
micro api是一个HTTP api。对API发起的的请求采用HTTP协议，并使用发我发现实现动态路由。它以[go-micro](https://github.com/micro/go-micro)为基础，利用它进行服务发现、负载平衡、编码和基于RPC的通信。

因为micro api在内部使用go-micro，这也使得它可以插入。有关对gRPC、kubernetes、etcd、nats、rabbitmq等的支持，请参见[go-plugins](https://github.com/micro/go-plugins)。此外，它还使用[go-api](https://github.com/micro/go-api)，他使得handlers也可以配置。

## 安装

```shell
go get -u github.com/micro/micro
```

## 运行

```shell
# Default port 8080
micro api
```

## 使用ACME

默认情况下，ACME安全服务使用Let 's Encrypt

```shell
MICRO_ENABLE_ACME=true micro api
```

可选地指定主机白名单

```shell
MICRO_ENABLE_ACME=true \
MICRO_ACME_HOSTS=example.com,api.example.com \
micro api
```

## 设置TLS证书

该API支持使用TLS证书安全地提供服务

```shell
MICRO_ENABLE_TLS=true \
MICRO_TLS_CERT_FILE=/path/to/cert \
MICRO_TLS_KEY_FILE=/path/to/key \
micro api
```
## 设置命名空间

该API使用名命名间将后端服务和面向公共的服务逻辑地分离开来。命名空间和http路径用于解析服务name/methode。例如 GET /foo HTTP/1.1路由到名为go.micro.api.foo的服务。

默认命名空间是go.micro.api，可以这样更改

```shell
MICRO_NAMESPACE=com.example.api micro api
```

若要禁用命名空间，请将其设置为空白。这是一个漏洞，我们将修复他。

```shell
MICRO_NAMESPACE=' '
```

## 示例

这里我们有一个3层架构的例子

- micro api: (localhost:8080) -作为http入口端点服务
- api service: (go.micro.api.greeter) - 面向公共的api服务
- backend service: (go.micro.srv.greeter) - 内部范围的服务

完整的例子在[examples/greeter](https://github.com/micro/examples/tree/master/greeter)

### 运行示例

 先决条件: 我们默认使用consul服务发现，所以确保其安装和运行。例如  consul agent -dev

```shell

# Download example
git clone https://github.com/micro/examples

# Start the service
go run examples/greeter/srv/main.go

# Start the API
go run examples/greeter/api/api.go

# Start the micro api
micro api

```

### 查询

通过micro api进行HTTP调用

``` shell
curl "http://localhost:8080/greeter/say/hello?name=John"
```

HTTP路径 /greeter/say/hello 映射到 service go.micro.api.greeter 的 Say.Hello方法

绕过api服务，直接通过/rpc调用后端

``` shell
curl -d 'service=go.micro.srv.greeter' \
     -d 'method=Say.Hello' \
     -d 'request={"name": "John"}' \
     http://localhost:8080/rpc
```

使用JOSN 发起相同的调用

``` shell
curl -H 'Content-Type: application/json' \
    -d '{"service": "go.micro.srv.greeter", "method": "Say.Hello", "request": {"name": "John"}}' \
    http://localhost:8080/rpc
```

## API

micro api提供以下HTTP api

```sehll
- /[service]/[method]  # HTTP路径被动态映射到服务
- /rpc    # 通过名称和方法显式调用后端服务
```

见下文例子

## Handlers

Handlers是管理请求路由的HTTP处理程序

默认handler使用来自注册中心的端点元数据来确定服务路由。如果没有找到路由匹配，它将退回到“rpc” handler。您可以使用[go-api](https://github.com/micro/go-api)在注册中心配置路由。

该API具有以下可配置的请求Handlers。

[api](#api-handler) - 处理任何HTTP请求。通过RPC对http请求/响应的完全控制。
[rpc](#rpc-handler) - 处理json和protobuf POST请求，通过RPC转发。
[proxy](#proxy-handler) - 处理HTTP并作为反向代理转发。
[event](#event-handler) - 处理任何HTTP请求并发布到消息总线。
[web](#web-handler) - 包含web套接字的HTTP反向代理。

可以选择使用 [/rpc](https://micro.mu/docs/api.html#rpc-endpoint) 端点绕过处理程序

### API Handler

API handler为任何HTTP请求提供服务，并将其作为具有特定格式的RPC请求转发。

- Content-Type: Any
- Body: Any
- 格式转化(Forward Format): [api.Request](https://github.com/micro/go-api/blob/master/proto/api.proto#L11)/[api.Response](https://github.com/micro/go-api/blob/master/proto/api.proto#L21)
- 路径(Path): /[service]/[method]
- 解析器(Resolver): 路径用于解析服务和方法
- 配置(Configure): 命令行参数 --handler=api 或 环境变量 MICRO_API_HANDLER=api

### RPC Handler

RPC handler提供json或protobuf HTTP POST请求，并作为RPC请求转发。

- Content-Type: application/json or application/protobuf
- Body: JSON or Protobuf
- 格式转化(Forward Format): json-rpc or proto-rpc based on content
- 路径(Path): /[service]/[method]
- 解析器(Resolver): 路径用于解析服务和方法
- 配置(Configure): 命令行参数 --handler=rpc 或环境变量  MICRO_API_HANDLER=rpc
- 没有指定Handler时的默认Handler
  
### Proxy Handler

代理handler是一个内置服务发现的http备用代理。

- Content-Type: Any
- Body: Any
- 格式转化(Forward Format): HTTP 反向代理
- 路径(Path): /[service]
- 解析器(Resolver): 路径用于解析服务名称
- 配置(Configure): 命令行参数 --handler=proxy 或环境变量 MICRO_API_HANDLER=proxy
- REST可以作为微服务在API下实现

### Event Handler

事件处理程序服务于HTTP，并使用go-micro 经纪人(broker)上将请求作为消息转发到消息总线上。

- Content-Type: Any
- Body: Any
- 格式转化(Forward Format): 请求被格式化为[go-api/proto.Event](https://github.com/micro/go-api/blob/master/proto/api.proto#L28L39)
- 路径(Path): /[topic]/[event]
- 解析器(Resolver): 路径用于解析服务名称
- 配置(Configure): 命令行参数 --handler=event 或环境变量 MICRO_API_HANDLER=event

### Web Handler

web Handler是一个内置服务发现和支持web socket的http备用代理。

- Content-Type: Any
- Body: Any
- 格式转化(Forward Format):包括web套接字在内的HTTP反向代理
- 路径(Path): /[service]
- 解析器(Resolver): 路径用于解析服务名称
- 配置(Configure): 命令行参数 --handler=web 或环境变量 MICRO_API_HANDLER=web

###　RPC endpoint

/rpc端点让您绕过主handler直接与任何服务通信

- 请求参数
  - service - 设置服务名称
  - method - 设置服务方法
  - request - 请求主体
  - address - 可选地指定目标主机地址

### 调用示例

``` shell
curl -d 'service=go.micro.srv.greeter' \
     -d 'method=Say.Hello' \
     -d 'request={"name": "Bob"}' \
     http://localhost:8080/rpc
```

请在[github.com/micro/examples/api](https://github.com/micro/examples/tree/master/api)中查找工作示例

## 分解器（Resolver）

使用命名空间值和HTTP路径动态路由到服务。

默认名称空间是go.micro.api。通过——namespace或MICRO_NAMESPACE=设置命名空间。


下面解释使用的解析器。

### RPC Resolver

RPC服务有一个名称(go.micro.api.greeter)和一个方法(Greeter.Hello)。

URLs 解析如下:

| Path | Service | Method |
|----|----|---|
| /foo/bar | go.micro.api.foo | Foo.Bar |
|/foo/bar/baz | go.micro.api.foo | Bar.Baz
|/foo/bar/baz/cat | go.micro.api.foo.bar | Baz.Cat

版本化的API Urls可以很容易地映射到服务名称:

| Path | Service | Method |
|----|----|---|
| /foo/bar | go.micro.api.foo | Foo.Bar |
| /v1/foo/bar | go.micro.api.v1.foo | Foo.Bar |
| /v1/foo/bar/baz | go.micro.api.v1.foo | Bar.Baz |
| /v2/foo/bar | go.micro.api.v2.foo | Foo.Bar |
| /v2/foo/bar/baz | go.micro.api.v2.foo	| Bar.Baz |

### 代理(proxy)解析器

对于代理(proxy)handler，我们只需要处理解析服务名称。因此，解决方案与RPC解析器略有不同。

URLs 解析如下

| Path | Service | Service Path |
|----|----|---|
| /foo | go.micro.api.foo | /foo |
| /foo/bar | go.micro.api.foo | /foo/bar |
| /greeter | go.micro.api.greeter | /greeter |
| /greeter/:name | go.micro.api.greeter | /greeter/:name |