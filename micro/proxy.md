# 服务代理

>**摘要:** micro proxy是一个服务到服务的代理。

**目录**

- [概述](#概述)
- [安装](#安装)
- [依赖](#依赖)
- [运行代理](#运行代理)
- [代理服务](#代理服务)
- [单个后端](#单个后端)


服务代理是充当一个服务到另一个服务请求的中介的服务。

<img src="https://micro.mu/docs/images/proxy.svg" />

## 概述

micro proxy提供go-micro框架的代理实现。这将go-micro特性整合到一起，允许将服务发现、负载平衡、容错、插件、包装器等卸载(offloading)到代理本身。比起更新到每一个Go Micro应用，更容易把他们放在代理。它还允许集成任何语言实现的轻量级客户端，而不必实现所有特性。

## 安装

~~~ shell
go get -u github.com/micro/micro
~~~

## 依赖

代理使用Go Micro，这意味着它依赖于服务发现。默认使用MDNS，意味着零配置。

如果需要更具伸缩性，可以安装consul，并使用——registry=consul 参数指定。

``` shell
brew install consul
consul agent -dev
```

## 运行代理

启动代理

``` shell
micro proxy
```

服务器地址是动态的，但可以按照以下方式配置。

MICRO_SERVER_ADDRESS=localhost:9090 micro proxy

## 代理服务

现在代理正在运行，您可以非常简单地通过它代理请求。

像这样启动你的go micro应用

``` shell
MICRO_PROXY_ADDRESS=localhost:9090 go run main.go
```

确保代理在指定的地址上运行。

```shell
MICRO_SERVER_ADDRESS=localhost:9090 micro proxy
```

## 单个后端

将代理作为单个后端的前端代理

``` shell
MICRO_SERVER_NAME=helloworld \
MICRO_PROXY_BACKEND=localhost:10001 \
micro proxy
```

所有对helloworld的请求都将发送到后端localhost:10001