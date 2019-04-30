Docker 部署
=========

**目录**

  - [预编译镜像](#%E9%A2%84%E7%BC%96%E8%AF%91%E9%95%9C%E5%83%8F)
    - [安装micro](#%E5%AE%89%E8%A3%85micro)
  - [Compose](#compose)
  - [基于scratch构建](#%E5%9F%BA%E4%BA%8Escratch%E6%9E%84%E5%BB%BA)
    - [编译二进制文件](#%E7%BC%96%E8%AF%91%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6)
    - [编译镜像](#%E7%BC%96%E8%AF%91%E9%95%9C%E5%83%8F)

docker部署
---------

Micro 能够容易的运行在docker容器

### 预编译镜像

预编译镜像可以在[Docker Hub](https://hub.docker.com/r/microhq/) 上获取

#### 安装micro

``` shell
docker pull microhq/micro
```

### Compose

用过 docker compose 本地部署

``` yaml
consul:
  command: -server -bootstrap -rejoin 
  image: progrium/consul:latest
  hostname: "registry"
  ports:
  - "8300:8300"
  - "8400:8400"
  - "8500:8500"
  - "8600:53/udp"
api:
  command: --registry_address=registry:8500 --register_interval=5 --register_ttl=10 api
  build: .
  links:
  - consul
  ports:
  - "8080:8080"
proxy:
  command: --registry_address=registry:8500 --register_interval=5 --register_ttl=10 proxy
  build: .
  links:
  - consul
  ports:
  - "8081:8081"
web:
  command: --registry_address=registry:8500 --register_interval=5 --register_ttl=10 web
  build: .
  links:
  - consul
  ports:
  - "8082:8082"
```

### 基于scratch构建

一个Dockerfile文件包含于出库内

```dockerfile
FROM alpine:3.2
RUN apk add --update ca-certificates && \
    rm -rf /var/cache/apk/* /tmp/*
ADD micro /micro
WORKDIR /
ENTRYPOINT [ "/micro" ]
```

#### 编译二进制文件

```shell
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-w' -i -o micro ./main.go 
```

#### 编译镜像

```shell
Build Image
```