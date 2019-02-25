# 架构

**目录**

- [工具箱](#工具箱)
  - [API](#api)
  - [Web](#web)
  - [代理](#代理)
  - [Bot](#bot)
  - [CLI](#cli)
- [Go Micro](#go-micro)
  - [注册(Registry)](#注册registry)
  - [选择器](#选择器)
  - [经纪人(Broker)](#经纪人broker)
  - [传输层(Transport)](#传输层transport)
  - [客户端](#客户端)
  - [服务](#服务)
- [插件](#插件)

Micro为微服务提供了基本的构建模块。他的目标是简化分布式系统的开发。因为微服务是一种架构模式，所以Micro通过工具实现逻辑上的责任分离。

查看关于架构的博文[https://micro.mu/blog/2016/04/18/micro-architecture.html](https://micro.mu/blog/2016/04/18/micro-architecture.html)以获得详细的概述。

本节将更多地解释micro是如何构建的，以及各种库之间是如何相互关联的。

## 工具箱

### API

API充当网关或代理，以支持单个入口点来访问微服务。它应该在基础设施的边缘运行。它将HTTP请求转换为RPC并转发到适当的服务。

<img src="https://micro.mu/docs/images/api.png"/>

### Web

UI是go-micro的web版本，允许在环境中进行可视交互。在未来，它也将成为聚合微web服务的一种方式。它包括一种代理到web应用程序的方法。/[name]将路由到注册表中的一个a服务。Web UI添加了前缀“go.micro.web”（可配置）到name,然后在注册表中查找它，然后反向代理它

<img src="https://micro.mu/docs/images/web.png"/>

### 代理

代理是对远程环境的一个cli代理。

<img src="https://micro.mu/docs/images/car.png">

### Bot

Bot是一种Hubot风格的Bot，它位于您的微服务平台中，可以通过Slack、HipChat、XMPP等进行交互。它通过消息传递提供CLI的特性。可以添加其他命令来自动化常见的ops任务。

<img src="https://micro.mu/docs/images/bot.png">

### CLI

The Micro CLI is a command line version of go-micro which provides a way of observing and interacting with a running environment.

<img src="https://micro.mu/docs/images/go-micro.svg">

## Go Micro

### 注册(Registry)

注册提供一个可插拔的服务发现库来查找正在运行的服务。目前实现了consul，etcd，memory 和 kubernetes。如果你的选择不同，接口很容易实现。

### 选择器

选择器通过选择提供负载平衡机制。当客户机向服务发出请求时，它将首先查询服务的注册表。这通常返回表示服务的正在运行的节点列表。选择器将选择用于查询的这些节点之一。对选择器的多次调用将允许使用平衡算法。目前的方法有循环、随机哈希和黑名单。

### 经纪人(Broker)

经纪人(Broker)是发布/订阅的可插入接口。微服务是事件驱动的体系结构，其中发布和订阅事件应该是一等公民。当前的实现包括nats、rabbitmq和http(用于开发)。

### 传输层(Transport)

传输层是一个可插拔的接口，用于点到点的消息传输。当前的实现是http、rabbitmq和nats。通过提供这种抽象，可以无缝地替换传输层。

### 客户端

客户端提供了一种进行RPC查询的方法。它结合了注册表、选择器、经济人(Broker)和传输。它还提供重试、超时、上下文使用等。

### 服务

服务是构建正在运行的微服务的接口。它提供了一种服务RPC请求的方法。

## 插件

go-micro的插件可以在[插件](https://github.com/micro/go-plugins)上找到。