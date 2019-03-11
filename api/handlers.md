# Go API

**目录**

Handlers用于处理http请求。为了方便起见，它使用http.Handler模式。

- [api](#api-handler) - 处理任意http请求。通过RPC完全控制http请求/响应。
- [broker](#broker) - 实现 go-micro broker 接口的http处理程序。
- [cloudevents](#cloudevents) -处理CloudEvents并将其发布到消息总线。
- [event](#event) - 处理任意HTTP请求并发布到消息总线。
- [http](#http) - 处理任意HTTP请求并作为反向代理转发。
- [registry](registry) - 实现go-micro registry接口的http处理程序
- [rpc](#rpc) - 处理json和protobuf POST请求。通过RPC转发。
- [web](#web) - 包含支持web socket的HTTP处理程序

## API Handler

API Handler是默认的处理程序。它为任意HTTP请求服务，并以指定格式作为RPC请求转发。

- Content-Type: Any
- Body: Any
- 转发格式: [api.Request](https://github.com/micro/go-api/blob/master/proto/api.proto#L11)/[api.Response](https://github.com/micro/go-api/blob/master/proto/api.proto#L21)
- 路径: /[service]/[method]
- 解析:路径用于解析服务和方法

## Broker Handler

Broker Handler是实现go-micro broker接口的http处理程序。

- Content-Type: Any
- Body: Any
- 转发格式: HTTP
- 路径: /
- 解析:主题以查询参数的方式被指定

Post方式发送请求，它将被发布

##　CloudEvents Handler

CloudEvents　Handler为HTTP请求提供服务，并使用go-micro/client.Publish方法在消息总线上以CloudEvents message的形式转发请求。

- Content-Type: Any
- Body: Any
- 转发格式: 请求格式为[CloudEvents](https://github.com/cloudevents/spec)消息
- 路径: /[topic]
- 解析:路径用于解析主题

## Event Handler

event handler为HTTP请求提供服务，并使用go-micro/client.Publish方法在消息总线将请求作为消息转发。

- Content-Type: Any
- Body: Any
- 转发格式: 请求格式为[go-api/proto.Event](https://github.com/micro/go-api/blob/master/proto/api.proto#L28L39)消息
- 路径: /[topic]/[event]
- 解析:路径用于解析主题和事件名

## HTTP Handler

http handler是一个内置服务发现的http反向代理。

- Content-Type: Any
- Body: Any
- 转发格式: HTTP 反向代理
- 路径: /[service]
- 解析:路径用于解析服务名

##　Registry Handler

registry handler序是一个http处理程序，它为go-micro注册表接口提供服务

- Content-Type: Any
- Body: JSON
- 转发格式: HTTP
- 路径: /
- 解析:使用GET、POST、DELETE获取服务、注册或注销

## RPC Handler

 RPC handler为json或protobuf HTTP POST请求提供服务，并作为RPC请求转发。

- Content-Type: application/json or application/protobuf
- Body: JSON or Protobuf
- 转发格式: json-rpc or proto-rpc based on content
- 路径: /[service]/[method]
- 解析:路径用于解析服务和方法

## Web Handler

web handler是一个http反向代理，内置服务发现并支持web　socket。

- Content-Type: Any
- Body: Any
- 转发格式: HTTP反向代理，包括web sockets
- 路径: /[service]
- 解析:路径用于解析服务名称