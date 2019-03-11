# Go Config

> **摘要**:Go Config是一个可插拔的动态配置库

**目录**

- [特征](#特征)
- [开始](#开始)
- [Source](#source)
  - [变更集(Changeset)](#变更集changeset)
- [Encoder](#encoder)
- [Reader](#reader)
- [Config](#config)
- [用例](#用例)
  - [样例配置](#样例配置)
  - [新建Config](#新建config)
  - [加载文件](#加载文件)
  - [读取配置](#读取配置)
  - [读取值](#读取值)
  - [监视路径](#监视路径)
  - [多源](#多源)
  - [设置源编码器](#设置源编码器)
  - [添加Reader Encoder](#添加reader-encoder)
- [常见问题](#常见问题)
  - [和Viper有啥不同？](#和viper有啥不同)
  - [编码器(Encoder)和阅读器(Reader)有什么不同？](#编码器encoder和阅读器reader有什么不同)
  - [为什么变更集数据没有表示为map[string]interface{}？](#为什么变更集数据没有表示为mapstringinterface)

## 概述

应用程序中的大多数配置都是静态配置，或者包含从多个源加载的复杂逻辑。Go-config使这变得简单、可插拔和可合并。你再也不用用同样的方法处理配置了。

### 特征

- **动态载入** - 根据需要从多个源加载配置。Go Config管理后台的配置源，并自动合并和更新内存视图。
- **可插拔的来源** - 从任意数量的源中选择要加载和合并配置。后端源被抽象成标准格式，在内部使用，并通过编码器解码。源可以是环境变量、命令行参数、文件、etcd、k8s configmap等。
- **可以合并配置** - 如果指定多个配置源，无论格式如何，它们都将被合并并显示在一个视图中。这极大地简化了加载优先级顺序和基于环境的更改。
- **观察变化** - 可以选择查看配置，查看对特定值的更改。使用Go Config的watcher热加载应用程序。您不需要处理特别的 ad-hoc hup 重载或其他任何事情，如果您需要被通知，只要继续阅读配置并观察更改。
- **安全修复** - 如果配置加载不当或由于某种未知原因被完全删除，您可以在直接访问任何配置值时指定回退值。这可以确保您在出现问题时始终能够读取一些正常的默认值。

### 开始

- [Source](#source) - 加载配置的后端(backend)
- [Encoder](#encoder) - 处理编码/解码源配置
- [Reader](#reader) - 将多个已编码的源合并为统一格式
- [Config](#config) - 管理多个源的配置管理器
- [用例](#用例) - go-config的示例用法
- [常见问题](#常见问题) - 常见问题解答
- [备忘](#备忘) - 备忘 任务/功能

### Source

源是加载配置的后端(backend)。可以同时使用多个源。

支持源如下:

- [configmap](https://github.com/micro/go-config/tree/master/source/configmap) - 从k8s configmap读取
- [consul](https://github.com/micro/go-config/tree/master/source/consul) - 从consul读取
- [etcd](https://github.com/micro/go-config/tree/master/source/etcd) - 从etcd v3读取
- [env](https://github.com/micro/go-config/tree/master/source/env) - 从环境变量读取
- [file](https://github.com/micro/go-config/tree/master/source/file) - 从文件读取
- [flag](https://github.com/micro/go-config/tree/master/source/flag) - 从命令行参数读取
- [grpc](https://github.com/micro/go-config/tree/master/source/grpc) - 从 grpc服务读取
- [内存](https://github.com/micro/go-config/tree/master/source/memory) - 从内存读取
- [microcli](https://github.com/micro/go-config/tree/master/source/microcli) - 从micro 命令行参数读取

TODO:

- vault
- git url

#### 变更集(Changeset)

源以变更集的形式返回配置。这是用于多个后端的单个内部抽象。

``` go
type ChangeSet struct {
    // 原始编码配置数据
    Data      []byte
    // MD5数据校验和
    Checksum  string
    // 编码格式 json, yaml, toml, xml等
    Format    string
    // 配置源 文件, consul, etcd等
    Source    string
    // 加载或更新的时间
    Timestamp time.Time
}
```

### Encoder

Encoder处理源配置编码/解码。后端源可能以多种不同的格式存储配置。编码器使我们能够处理任何格式。如果没有指定编码器，则默认为json。

支持以下编码格式:

- json
- yaml
- toml
- xml
- hcl

### Reader

阅读器将多个变更集(Changeset)合并为一个的可查询的值集。

``` go
type Reader interface {
    // 将多个变更集(Changeset)合并到一个格式中
    Merge(...*source.ChangeSet) (*source.ChangeSet, error)
    // Return 返回Go可维护值
    Values(*source.ChangeSet) (Values, error)
    // reader名称 例如 json reader
    String() string
}
```

reader利用编码器将变更集解码成map[string]interface{}，然后将它们合并成一个变更集(Changeset)。它查看Format字段来确定编码器。然后，变更集被表示为一组值，具有检索Go类型和回退的能力，这些类型和回退的值不能被加载。

``` go
// Values is returned by the reader
type Values interface {
    // Return raw data
        Bytes() []byte
    // Retrieve a value
        Get(path ...string) Value
    // Return values as a map
        Map() map[string]interface{}
    // Scan config into a Go type
        Scan(v interface{}) error
}
```

The Value interface allows casting/type asserting to go types with fallback defaults.
当回退默认值时，Value接口允许强制类型转换为go类型。

``` go
type Value interface {
    Bool(def bool) bool
    Int(def int) int
    String(def string) string
    Float64(def float64) float64
    Duration(def time.Duration) time.Duration
    StringSlice(def []string) []string
    StringMap(def map[string]string) map[string]string
    Scan(val interface{}) error
    Bytes() []byte
}
```

### Config

Config管理所有配置，抽象出源、编码器和读取器。

它管理来自多个后端源的读取、同步和监视，并将它们合并为一个可查询的源。

``` go
//onfig是动态配置的接口抽象
type Config interface {
    // 提供 reader.Values 接口
    reader.Values
    // 停止配置加载/监视
    Close() error
    // 加载配置源
    Load(source ...source.Source) error
    // 强制源更改集(Changeset)同步
    Sync() error
    // 监视value变化
    Watch(path ...string) (Watcher, error)
}
```

### 用例

- [样例配置]()
- [New Config]()
- [Load File]()
- [Read Config]()
- [Read Values]()
- [Watch Path]()
- [Multiple Sources]()
- [Set Source Encoder]()
- [Add Reader Encoder]()

#### 样例配置

配置文件可以是任何格式，只要我们有一个编码器来支持它。
示例json配置:

``` json
{
    "hosts": {
        "database": {
            "address": "10.0.0.1",
            "port": 3306
        },
        "cache": {
            "address": "10.0.0.2",
            "port": 6379
        }
    }
}
```

####　新建Config
创建一个新配置(或者使用默认实例)

``` go
import "github.com/micro/go-config"

conf := config.NewConfig()
```

#### 加载文件

从文件源加载配置。它使用文件扩展名来确定配置格式。

``` go
import (
    "github.com/micro/go-config"
)

// Load json config file
config.LoadFile("/tmp/config.json")
```

通过指定具有适当文件扩展名的文件来加载yaml、toml或xml文件

``` go
// Load yaml config file
config.LoadFile("/tmp/config.yaml")
```

如果扩展名不存在，请指定编码器

``` go
import (
    "github.com/micro/go-config"
    "github.com/micro/go-config/source/file"
)

enc := toml.NewEncoder()

// Load toml file with encoder
config.Load(file.NewSource(
    file.WithPath("/tmp/config"),
    source.WithEncoder(enc),
))
```

####　读取配置

将整个配置作为映射读取

``` go
// retrieve map[string]interface{}
conf := config.Map()

// map[cache:map[address:10.0.0.2 port:6379] database:map[address:10.0.0.1 port:3306]]
fmt.Println(conf["hosts"])
```

####　读取值

将配置扫描到结构中

``` go
type Host struct {
        Address string `json:"address"`
        Port int `json:"port"`
}

type Config struct{
    Hosts map[string]Host `json:"hosts"`
}

var conf Config

config.Scan(&conf)

// 10.0.0.1 3306
fmt.Println(conf.Hosts["database"].Address, conf.Hosts["database"].Port)
```

读取Values

将配置中的值扫描到结构中

``` go
type Host struct {
    Address string `json:"address"`
    Port int `json:"port"`
}

var host Host

config.Get("hosts", "database").Scan(&host)

// 10.0.0.1 3306
fmt.Println(host.Address, host.Port)
```

将单个值读取为Go类型

``` go
// Get address. Set default to localhost as fallback
address := config.Get("hosts", "database", "address").String("localhost")

// Get port. Set default to 3000 as fallback
port := config.Get("hosts", "database", "port").Int(3000)
```

#### 监视路径

监听路径的变化。当文件变化，新的值将变的可以使用

``` go
w, err := config.Watch("hosts", "database")
if err != nil {
    // do something
}

// wait for next value
v, err := w.Next()
if err != nil {
    // do something
}

var host Host

v.Scan(&host)
```

####　多源

可以加载和合并多个源。合并优先级与顺序相反。

``` go
config.Load(
    // base config from env
    env.NewSource(),
    // override env with flags
    flag.NewSource(),
    // override flags with file
    file.NewSource(
        file.WithPath("/tmp/config.json"),
    ),
)
```

#### 设置源编码器

源需要编码器对数据进行编码/解码，并指定变更集(changeset )格式。
默认编码器是json。要将编码器更改为yaml、xml, toml需要指定为一个选项。

``` go
e := yaml.NewEncoder()

s := consul.NewSource(
    source.WithEncoder(e),
)
```

#### 添加Reader Encoder

读者使用编码器来解码来自不同格式源的数据。
默认reader支持json、yaml、xml、toml和hcl。它将合并后的配置表示为json。
通过指定一个选项来添加一个新的编码器。

``` go
e := yaml.NewEncoder()

r := json.NewReader(
    reader.WithEncoder(e),
)
```

### 常见问题

#### 和Viper有啥不同？

Viper和go-config正在解决同样的问题。Go-config提供了一个不同的接口，并且是更大的micro 生态系统工具的一部分。

#### 编码器(Encoder)和阅读器(Reader)有什么不同？

后端源使用编码器(Encoder)对其数据进行编码/解码。阅读器(Reader)使用编码器解码来自多个源不同格式的数据，然后将它们合并成一种编码格式。

对于文件源，我们使用文件扩展名来确定配置格式，因此不使用编码器。

对于consul、etcd或类似的键值对数据源，我们可以从包含多个键的前缀加载，这意味着源需要理解编码，以便能够返回单个更改集。

在环境变量和命令行参数(flags)的情况下，我们还需要一种方法将值编码为字节并指定格式，以便稍后被阅读器(reader)合并。

#### 为什么变更集数据没有表示为map[string]interface{}？

在某些情况下，源数据实际上可能不是键值，因此更容易将其表示为字节，并将解码延迟到阅读器(reader)。