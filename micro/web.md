# Web

> **摘要：** Micro Web提供了一个仪表盘来可视化和浏览服务

**目录**

  - [安装](#%E5%AE%89%E8%A3%85)
  - [运行](#%E8%BF%90%E8%A1%8C)
  - [使用ACME](#%E4%BD%BF%E7%94%A8acme)
  - [设置TLS证书](#%E8%AE%BE%E7%BD%AEtls%E8%AF%81%E4%B9%A6)
  - [截屏](#%E6%88%AA%E5%B1%8F)

## 安装

``` shell
go get github.com/micro/micro
```

## 运行

``` shell
micro web
```

访问localhost:8082

## 使用ACME

micro web仪表盘通过Let 's Encrypt支持ACME。它会自动为您的域名获取TLS证书。

``` shell
micro --enable_acme web
```

可选地指定主机白名单

``` shell
micro --enable_acme --acme_hosts=example.com,api.example.com web
```

## 设置TLS证书

仪表盘支持使用TLS证书地保证服务安全

``` shell
micro --enable_tls --tls_cert_file=/path/to/cert --tls_key_file=/path/to/key web
```

## 截屏

<img src="https://micro.mu/docs/images/web1.png"/>
<img src="https://micro.mu/docs/images/web2.png"/>
<img src="https://micro.mu/docs/images/web3.png"/>
<img src="https://micro.mu/docs/images/web4.png"/>
