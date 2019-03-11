# Bot

**目录**

- [支持输入](#支持输入)
- [开始](#开始)
  - [安装micro](#安装micro)
  - [以Slack方式运行](#以slack方式运行)
  - [以HipChat方式运行](#以hipchat方式运行)
  - [帮助](#帮助)
- [添加新的命令](#添加新的命令)
  - [编写一个命令](#编写一个命令)
  - [注册一个命令](#注册一个命令)
  - [重新编译Micro](#重新编译micro)
- [作为服务命令](#作为服务命令)
  - [如何工作？](#如何工作)
  - [示例](#示例)

## micro机器人(micro bot)

微机器人是一个位于微服务环境中的机器人，您可以通过Slack、HipChat、XMPP等与之交互。它通过消息传递模仿CLI的功能。

<img src="https://micro.mu/docs/images/bot.png"/>

### 支持输入

- Slack
- HipChat

### 开始

#### 安装micro

``` shell
go get github.com/micro/micro
```

#### 以Slack方式运行

``` shell
micro bot --inputs=slack --slack_token=SLACK_TOKEN
```

<img src="https://micro.mu/docs/images/slack.png">

#### 以HipChat方式运行

``` shell
micro bot --inputs=hipchat --hipchat_username=XMPP_USER --hipchat_password=XMPP_PASSWORD
```

<img src="https://micro.mu/docs/images/hipchat.png">

通过指定逗号分隔的列表来使用多个输入

``` shell
micro bot --inputs=hipchat,slack --slack_token=SLACK_TOKEN --hipchat_username=XMPP_USER --hipchat_password=XMPP_PASSWORD
```

#### 帮助

使用 slack

``` shell
micro help

deregister service [definition] - Deregisters a service
echo [text] - Returns the [text]
get service [name] - Returns a registered service
health [service] - Returns health of a service
hello - Returns a greeting
list services - Returns a list of registered services
ping - Returns pong
query [service] [method] [request] - Returns the response for a service query
register service [definition] - Registers a service
the three laws - Returns the three laws of robotics
time - Returns the server time
```

### 添加新的命令

命令是基于文本匹配模式被机器人执行的函数。

#### 编写一个命令

``` go
import "github.com/micro/go-bot/command"

func Ping() command.Command {
    usage := "ping"
    description := "Returns pong"

    return command.NewCommand("ping", usage, desc, func(args ...string) ([]byte, error) {
        return []byte("pong"), nil
    })
}
```

#### 注册一个命令

将一个命令以键值对的形式添加到命令映射表，他可以被golang/regexp.Match匹配到

``` go
import "github.com/micro/go-bot/command"

func init() {
    command.Commands["^ping$"] = Ping()
}
```

#### 重新编译Micro

编译二进制

``` go
cd github.com/micro/micro

// For local use
go build -i -o micro ./main.go

// For docker image
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-w' -i -o micro ./main.go
```

### 作为服务命令

micro bot人具备将命令创建为微服务的能力。

#### 如何工作？

机器人通过服务的命名空间监听服务的注册。默认命名空间是go.micro.bot。此命名空间下的任何服务都将被自动添加到可用命令列表。当一个命令被执行，机器人将会通过Command.Exec调用服务。同时通过Command.Help提供使用帮助

服务接口如下，可以在[go-bot/proto](https://github.com/micro/go-bot/blob/master/proto/bot.proto)找到

``` protobuf
syntax = "proto3";

package go.micro.bot;

service Command {
    rpc Help(HelpRequest) returns (HelpResponse) {};
    rpc Exec(ExecRequest) returns (ExecResponse) {};
}

message HelpRequest {
}

message HelpResponse {
    string usage = 1;
    string description = 2;
}

message ExecRequest {
    repeated string args = 1;
}

message ExecResponse {
    bytes result = 1;
    string error = 2;
}
```

#### 示例

这里是echo 命令作为微服务的示例

``` go
package main

import (
    "fmt"
    "strings"

    "github.com/micro/go-micro"
    "golang.org/x/net/context"

    proto "github.com/micro/go-bot/proto"
)

type Command struct{}

// Help returns the command usage
func (c *Command) Help(ctx context.Context, req *proto.HelpRequest, rsp *proto.HelpResponse) error {
    // Usage should include the name of the command
    rsp.Usage = "echo"
    rsp.Description = "This is an example bot command as a micro service which echos the message"
    return nil
}

// Exec executes the command
func (c *Command) Exec(ctx context.Context, req *proto.ExecRequest, rsp *proto.ExecResponse) error {
    rsp.Result = []byte(strings.Join(req.Args, " "))
    // rsp.Error could be set to return an error instead
    // the function error would only be used for service level issues
    return nil
}

func main() {
    service := micro.NewService(
        micro.Name("go.micro.bot.echo"),
    )

    service.Init()

    proto.RegisterCommandHandler(service.Server(), new(Command))

    if err := service.Run(); err != nil {
        fmt.Println(err)
    }
}
```