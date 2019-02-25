# Micro [![License](https://img.shields.io/:license-apache-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![GoDoc](https://godoc.org/github.com/micro/micro?status.svg)](https://godoc.org/github.com/micro/micro) [![Travis CI](https://travis-ci.org/micro/micro.svg?branch=master)](https://travis-ci.org/micro/micro) [![Go Report Card](https://goreportcard.com/badge/micro/micro)](https://goreportcard.com/report/github.com/micro/micro)

go语言微服务工具箱文档中文翻译

Micro是一套微服务开发工具箱。

## 概述

Micro解决了构建可伸缩系统的关键需求。它采用微服务架构模式并将其转换为一套构建积木(blocks)平台的工具。Micro能够处理复杂的分布式系统并为开发者提供能够已经了解的简单抽象。

<img src="https://micro.mu/micro-diag.svg" />

技术在不断演进，基础技术栈总是在变化，Micro是能够解决这些问题的可插拔工具箱。插入任何技术栈或潜在技术，用micro构建不会过时的系统。

## 特征

工具箱由一下功能组成：

- **API Gateway:** 一个由通过服务发现实现动态请求路由的单点入口。API网关允许你在后端构建可扩展的微服务架构并将前端服务整合到一个公共API。micro api通过服务发现和可插拔的handlers为http, grpc, websockets, publish events等提拱了强大的路由。

- **Interactive CLI:** cil通过终端(terminal)来描述，查询，并你的平台和服务直接交互。cli提供你希望了解的所有微服务命令。它还包含一个交互模式。

- **Service Proxy:** 一个构建在[Go Micro](https://github.com/micro/go-micro) and the [MUCP](https://github.com/micro/protocol) 协议上的透明代理，将服务发现，负载均衡，容错，消息编码，中间件，监视等飞流到单个位置。将它独立运行或在你的服务端运行。

- **Service Templates:** 快速生成新的服务模板。Micro为微服务提供预定义模板。始终以相同的方式开始，高效的构建相同服务。

- **SlackOps Bot:** 一个运行在你的平台上的机器人，让你从Slack本身管理你的应用程序。微机器人支持ChatOps，让你能够通过消息与你的团队一起做任何事情。它还包括创建slack commmand作为动态发现的服务的能力。

- **Web Dashboard:** 微博仪表盘让你能够浏览你的服务、描述他们的端点(endpoints)、请求和响应格式、甚至
直接查询。仪表盘还内建了一个cli，开发人员可以直接进入终端。

## 准备开始
有关工具箱的体系结构、安装和使用的详细信息，请参见[翻译文档](catalogue.md)或[原始文档](https://micro.mu/docs/toolkit.html)。

## 赞助

Sixt 是Micro的赞助企业

<a href="https://micro.mu/blog/2016/04/25/announcing-sixt-sponsorship.html"><img src="https://micro.mu/sixt_logo.png" width=150px height="auto" /></a>
