# 安装指南

**目录**
- [安装指南](#%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)
  - [安装指南](#%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97-1)
    - [依赖](#%E4%BE%9D%E8%B5%96)
      - [Consul](#consul)
  - [Go Micro](#go-micro)
      - [安装](#%E5%AE%89%E8%A3%85)
      - [Protobuf](#protobuf)
  - [工具箱(Toolkit)](#%E5%B7%A5%E5%85%B7%E7%AE%B1toolkit)
      - [安装](#%E5%AE%89%E8%A3%85-1)
      - [Docker](#docker)
      - [尝试命令行(CLI)](#%E5%B0%9D%E8%AF%95%E5%91%BD%E4%BB%A4%E8%A1%8Ccli)

## 安装指南

### 依赖

我们使用服务发现来定位服务。默认情况下采用 multicast DNS，所以配置为无需配置。如果您需要多主机和更有弹性，请使用consul。查看go-plugins以尝试其他东西。

#### Consul

如果使用Consul

~~~ shell

    brew install consul
    consul agent -dev
~~~

或

~~~shell

docker run consul
~~~

通过任何micro命令带参数 --registry=consul 来使用consul，例如micro --registry=consul list services

## Go Micro

Go Micro是一个用于在Go中开发微服务的RPC框架

#### 安装

~~~ shell 
go get github.com/micro/go-micro
~~~
#### Protobuf

您还需要使用prototype -gen-micro来生成代码

- [protoc-gen-micro](https://github.com/micro/protoc-gen-micro)

## 工具箱(Toolkit)

微工具包提供了各种访问微服务的方法

#### 安装

~~~ shell

go get github.com/micro/micro
~~~

#### Docker

可以使用预构建的docker映像

~~~ shell
docker pull microhq/micro
~~~
#### 尝试命令行(CLI)

运行欢迎服务

~~~ shell

go get github.com/micro/examples/greeter/srv && srv
~~~

服务列表

~~~ shell

$ micro list services
consul
go.micro.srv.greeter
~~~

获取服务

~~~ shell

$ micro get service go.micro.srv.greeter
service  go.micro.srv.greeter

version 1.0.0

Id  Address Port   Metadata
go.micro.srv.greeter-34c55534-368b-11e6-b732-68a86d0d36b6   192.168.1.66   62525   server=rpc,registry=consul,transport=http,broker=http

Endpoint: Say.Hello
Metadata: stream=false

Request: {
    name string
}

Response: {
    msg string
}

~~~

调用服务

~~~ shell

$ micro call go.micro.srv.greeter Say.Hello '{"name": "John"}'
{
    "msg": "Hello John"
}
~~~

访问[github.com/micro/micro](https://github.com/micro/micro)了解更多信