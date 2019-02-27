# cli

**目录**<!-- TOC -->

- [开始](#开始)
- [安装](#安装)
- [交互模式](#交互模式)
- [示例使用](#示例使用)
  - [服务列表](#服务列表)
  - [获取服务](#获取服务)
  - [调用服务](#调用服务)
  - [服务健康](#服务健康)
  - [注册/注销](#注册注销)
- [代理远程环境](#代理远程环境)
- [用例](#用例)

## micro cli

micro cli是micro工具箱的命令行接口[micro](https://github.com/micro/micro)。

### 开始

- [开始](#开始)
- [安装](#安装)
- [交互模式](#交互模式)
- [服务列表](#服务列表)
- [获取服务](#获取服务)
- [调用服务](#调用服务)
- [服务健康](#服务健康)
- [注册/注销](#注册注销)
- [代理远程环境](#代理远程环境)

### 安装

``` shell
go get github.com/micro/micro
```

### 交互模式

将cli用作交互式提示

``` shell
micro cli
```

在交互模式下，从下面的命令中调动micro

###　示例使用

#### 服务列表

``` shell
micro list services
```

#### 获取服务

``` shell
micro get service go.micro.srv.example
```

输出

``` shell

go.micro.srv.example

go.micro.srv.example-fccbb6fb-0301-11e5-9f1f-68a86d0d36b6  [::]  62421

```

#### 调用服务

``` shell
micro call go.micro.srv.example Example.Call '{"name": "John"}'
```

输出

```shell
{
    "msg": "go.micro.srv.example-fccbb6fb-0301-11e5-9f1f-68a86d0d36b6: Hello John"
}
```

#### 服务健康

``` shell
micro health go.micro.srv.example
```

输出

``` shell
node    address:port    status
go.micro.srv.example-fccbb6fb-0301-11e5-9f1f-68a86d0d36b6    [::]:62421    ok
```

#### 注册/注销

``` shell
micro register service '{"name": "foo", "version": "bar", "nodes": [{"id": "foo-1", "address": "127.0.0.1", "port": 8080}]}'
```

``` shell
micro deregister service '{"name": "foo", "version": "bar", "nodes": [{"id": "foo-1", "address": "127.0.0.1", "port": 8080}]}'
```

### 代理远程环境

使用micro proxy的代理远程环境

在针对远程环境进行开发时，您可能无法直接访问服务发现，这使得CLI难以使用。micro proxy为这样的场景提供了一个http代理。

在远程环境中运行代理(proxy)

``` shell
micro proxy
```

设置环境变量 MICRO_PROXY_ADDRESS，以便cli知道使用代理

``` shell
MICRO_PROXY_ADDRESS=staging.micro.mu:8081 micro list services
```

### 用例

``` shell
NAME:
   micro - A cloud-native toolkit

USAGE:
   micro [global options] command [command options] [arguments...]

VERSION:
   0.8.0

COMMANDS:
    api     Run the micro API
    bot     Run the micro bot
    registry    Query registry
    call    Call a service or function
    query   Deprecated: Use call instead
    stream  Create a service or function stream
    health  Query the health of a service
    stats   Query the stats of a service
    list    List items in registry
    register    Register an item in the registry
    deregister  Deregister an item in the registry
    get         Get item from registry
    proxy   Run the micro proxy
    new     Create a new micro service by specifying a directory path relative to your $GOPATH
    web     Run the micro web app
```