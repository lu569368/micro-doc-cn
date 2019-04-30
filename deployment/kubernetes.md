# Kubernetes部署

Micro在Kubernetes是一个Kubernetes本地微服务。
Micro是一个微服务工具集。Kubernetes是容器编排器。
它们共同为微服务平台提供了基础。

  - [特征](#%E7%89%B9%E5%BE%81)
  - [入门指南](#%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97)
  - [安装Micro](#%E5%AE%89%E8%A3%85micro)
  - [编写一个服务](#%E7%BC%96%E5%86%99%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1)
  - [部署服务](#%E9%83%A8%E7%BD%B2%E6%9C%8D%E5%8A%A1)
  - [healthchecking sidecar](#healthchecking-sidecar)
      - [运行healtchecker](#%E8%BF%90%E8%A1%8Chealtchecker)
      - [K8s 部署](#k8s-%E9%83%A8%E7%BD%B2)
  - [K8s 负载均衡](#k8s-%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)
      - [用法](#%E7%94%A8%E6%B3%95)
      - [K8s 部署](#k8s-%E9%83%A8%E7%BD%B2-1)
      - [k8s 服务](#k8s-%E6%9C%8D%E5%8A%A1)

### 特征

- 无额外的依赖
- 客户端服务发现缓存
- 可选的k8s服务负载平衡
- gRPC传输协议
- 预初始化工具集

### 入门指南

  - [特征](#%E7%89%B9%E5%BE%81)
  - [入门指南](#%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97)
  - [安装Micro](#%E5%AE%89%E8%A3%85micro)
  - [编写一个服务](#%E7%BC%96%E5%86%99%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1)
  - [部署服务](#%E9%83%A8%E7%BD%B2%E6%9C%8D%E5%8A%A1)
  - [healthchecking sidecar](#healthchecking-sidecar)
      - [运行healtchecker](#%E8%BF%90%E8%A1%8Chealtchecker)
      - [K8s 部署](#k8s-%E9%83%A8%E7%BD%B2)
  - [K8s 负载均衡](#k8s-%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)
      - [用法](#%E7%94%A8%E6%B3%95)
      - [K8s 部署](#k8s-%E9%83%A8%E7%BD%B2-1)
      - [k8s 服务](#k8s-%E6%9C%8D%E5%8A%A1)


### 安装Micro

```shell
go get github.com/micro/kubernetes/cmd/micro
```

或

```shell
docker pull microhq/micro:kubernetes
```

使用go-micro

```go
import "github.com/micro/kubernetes/go/micro"
```

### 编写一个服务

像编写其他任何go-micro服务一样编写一个服务

```shell
import (
    "github.com/micro/go-micro"
    k8s "github.com/micro/kubernetes/go/micro"
)

func main() {
    service := k8s.NewService(
        micro.Name("greeter")
    )
    service.Init()
    service.Run()
}
```

### 部署服务

这里是k8s部署micro service的例子

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  namespace: default
  name: greeter
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: greeter-srv
    spec:
      containers:
        - name: greeter
            command: [
                "/greeter-srv",
                "--server_address=0.0.0.0:8080",
                "--broker_address=0.0.0.0:10001"
            ]
          image: microhq/greeter-srv:kubernetes
          imagePullPolicy: Always
          ports:
          - containerPort: 8080
            name: greeter-port
```

使用kubectl部署

```shell
kubectl create -f greeter.yaml
```

### healthchecking sidecar

healthchecking sidecar以http endpoint的方式暴露/health，并通过rpc访问服务上的Debug.Health endpoint。每一个 go-micro 服务都内置了一个Debug.Health endpoint。

安装 healthchecker

```shell
go get github.com/micro/kubernetes/cmd/health
```

或

```shell
docker pull microhq/health:kubernetes
```

#### 运行healtchecker

```shell
health --server_name=greeter --server_address=localhost:9091
```

通过localhost:8080访问healtchecker

```shell
curl http://localhost:8080/health
```

#### K8s 部署

添加到kubernetes部署

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  namespace: default
  name: greeter
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: greeter-srv
    spec:
      containers:
        - name: greeter
        command: [
            "/greeter-srv",
            "--server_address=0.0.0.0:8080",
            "--broker_address=0.0.0.0:10001"
        ]
          image: microhq/greeter-srv:kubernetes
          imagePullPolicy: Always
          ports:
          - containerPort: 8080
            name: greeter-port
        - name: health
            command: [
                "/health",
                    "--health_address=0.0.0.0:8081",
                "--server_name=greeter",
                "--server_address=0.0.0.0:8080"
            ]
          image: microhq/health:kubernetes
          livenessProbe:
            httpGet:
              path: /health
              port: 8081
            initialDelaySeconds: 3
            periodSeconds: 3
```

### K8s 负载均衡

虽然Micro默认包含了客户端负载均衡，但是kubernetes也提供了负载均衡策略。我们可以通过[静态选择器](https://github.com/micro/go-plugins/tree/master/selector/static)和k8s服务使用k8s的负载均衡。

不是解析服务地址,而是通过[静态选择器](https://github.com/micro/go-plugins/tree/master/selector/static)返回服务名称加一个固定端口而，例如greeter返回

#### 用法

当运行你的服务时，通过指定命令行参数或设置环境变量的方式来使用静态选择器

```shell
MICRO_SELECTOR=static ./service
```

或

```shell
./service --selector=static
```

#### K8s 部署

一个部署示例

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  namespace: default
  name: greeter
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: greeter-srv
    spec:
      containers:
        - name: greeter
          command: [
            "/greeter-srv",
            "--selector=static",
            "--server_address=0.0.0.0:8080",
            "--broker_address=0.0.0.0:10001"
         ]
          image: microhq/greeter-srv:kubernetes
          imagePullPolicy: Always
          ports:
          - containerPort: 8080
            name: greeter-port
```

#### k8s 服务

因为k8s服务要使用静态选择器，所以需要确保为每一个micro service创建一个k8s服务。

这里是一个简单服务

```yaml
apiVersion: v1
kind: Service
metadata:
  name: greeter
  labels:
    app: greeter
spec:
  ports:
  - port: 8080
    protocol: TCP
  selector:
    app: greeter
```

通过kubectl部署

```shell
kubectl create -f service.yaml
```

从你的服务访问微服务"greeter"会被路由到k8s 服务greeter:8080。