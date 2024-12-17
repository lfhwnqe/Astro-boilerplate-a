---
layout: '@/templates/BasePost.astro'
title: serverless架构选型
description: serverless架构选型，无惧高并发
pubDate: 2024-12-17T00:00:00Z
imgSrc: '/assets/images/image-post5.jpeg'
imgAlt: 'Image post 2'
---
# serverless架构选型，无惧高并发

## 目录
- [serverless架构选型，无惧高并发](#serverless架构选型无惧高并发)
  - [目录](#目录)
  - [传统开发痛点](#传统开发痛点)
  - [常规Serverless平台解决方案](#常规serverless平台解决方案)
  - [常规Serverless平台的局限性](#常规serverless平台的局限性)
  - [AWS Lambda解决方案](#aws-lambda解决方案)
    - [AWS Lambda优势](#aws-lambda优势)
    - [价格示例](#价格示例)
      - [示例1: Lambda@Edge 函数计费](#示例1-lambdaedge-函数计费)
      - [示例2: 食品订购应用后端计费](#示例2-食品订购应用后端计费)
  - [架构选型决策指南](#架构选型决策指南)
  - [FAQ](#faq)
    - [1. 计算密集型应用](#1-计算密集型应用)
    - [2. 需要长连接的应用](#2-需要长连接的应用)
    - [3. 有状态且对延迟敏感的应用](#3-有状态且对延迟敏感的应用)
    - [4. 具有特殊系统要求的应用](#4-具有特殊系统要求的应用)
  - [总结](#总结)

## 传统开发痛点
基于传统服务端部署方案的ssr服务，想要构建一个可以抗住高并发的系统，常会有以下痛点：

| 类别 | 具体问题 |
|------|-----------|
| 性能瓶颈 | • 服务端渲染消耗大量cpu与内存<br>• 数据库连接数限制<br>• 静态资源和动态请求混合，导致负载不均衡 |
| 扩容挑战 | • 手动配置扩容缩容策略<br>• 多实例，额外的负载均衡<br>• 扩容过程可能影响用户体验<br>• 冷启动时间 |
| 成本问题 | • ec2实例，按时计费<br>• 应对高峰需要预留足够资源<br>• 多可用区部署成本<br>• 运维人员投入成本 |

## 常规Serverless平台解决方案
目前的常规serverless平台如（vercel、netlify），基本解决了上述问题，他们提供了：
- 自动扩缩容,不用担心资源配置问题
- Edge Network加速,全球化部署容易
- 集成了CI/CD,部署流程简单高效
- 内置CDN,静态资源自动优化
- 零运维成本,专注业务开发
- 按使用量付费,避免资源浪费

对于小型的应用，使用vercel、netlify是不错的选择。它不但提供了快速开发部署的能力，还可以减少大量的成本（按量付费，运维）。

## 常规Serverless平台的局限性
虽然这些平台已经非常优秀，但对于企业级应用，他们仍然存在一些问题：

| 局限性类型 | 具体问题 |
|------------|----------|
| 安全与依赖 | • 需要暴露源码给平台，安全性不可控<br>• 平台锁定<br>• 费用不低，高级功能按需付费，跨地区部署产生额外费用 |
| 技术限制 | • 函数执行时长限制(Vercel为10秒,Netlify为10分钟)<br>• 函数内存限制(Vercel为1GB,Netlify为1.5GB)<br>• 部署包大小限制(通常在50MB-100MB之间)<br>• Lambda层打包大小限制<br>• 冷启动延迟(可能达到几百毫秒)<br>• WebSocket支持有限<br>• 某些Node.js原生模块可能不兼容 |
| 扩展性限制 | • 跨函数通信效率较低<br>• 长连接支持不够理想<br>• 无法直接访问某些底层系统资源<br>• 数据持久化依赖外部服务<br>• 自定义运行时受限<br>• 无法运行某些特殊的系统服务 |

## AWS Lambda解决方案
怎么解决上述平台的问题，又能享受serverless带来的便利呢？直接使用aws lambda架构实现。上述的serverless平台其实它们底层都是基于aws lambda进行深度封装的，我们可以直接使用aws lambda来构建自己的serverless架构。既可以享受serverless带来的便利，对于定制化需求，也可以更好的实现。

### AWS Lambda优势

| 优势类别 | 具体特点 |
|----------|----------|
| 技术规格限制 | • 内存：128MB-10GB可选择<br>• 执行时间：最长15分钟<br>• 部署包：50MB(直接上传),250MB(包含层)<br>• 支持自定义运行时<br>• 可以使用容器镜像<br>• 完整的VPC网络配置能力 |
| 架构灵活度 | • 可与多种AWS服务集成<br>• 自定义Layer<br>• 更多触发器、编排函数<br>• 适用于复杂后端服务 |
| 安全与稳定性 | • 代码本地管理，只需要构建产物上传<br>• 完整的IAM权限控制<br>• 可使用私有API网关<br>• 支持自定义SSL证书<br>• 监控和告警更完整<br>• 问题排查能力更强 |

### 价格示例

#### 示例1: Lambda@Edge 函数计费

| 费用类型 | 计算方式 | 金额(USD) |
|----------|----------|------------|
| 计算费用 | 100000秒 × $0.00000625125/128MB-秒 | $0.63 |
| 请求费用 | 10M请求 × $0.60/M | $6.00 |
| 总费用 | 计算费用 + 请求费用 | $6.63/月 |

#### 示例2: 食品订购应用后端计费

| 费用类型 | 计算明细 | 金额(USD) |
|----------|----------|------------|
| 计算费用 | (540000 GB-s - 400000免费GB-s) × $0.0000166667 | $2.33 |
| 请求费用 | (300万请求 - 100万免费请求) × $0.20/M | $0.40 |
| 总费用 | 计算费用 + 请求费用 | $2.73/月 |

显然，如果要构建企业级应用，无论从安全性，架构灵活性，成本上看，aws lambda都是更好的选择。



## 架构选型决策指南

| 场景 | 推荐方案 | 原因 |
|------|----------|------|
| 小型应用 | Vercel/Netlify | • 开发效率高<br>• 运维成本低<br>• 快速部署 |
| 企业应用 | AWS Lambda | • 安全性高<br>• 可定制性强<br>• 成本可控 |
| 高并发场景 | AWS Lambda | • 自动扩缩容<br>• 性能稳定<br>• 成本优势 |

## FAQ

Q: Serverless适合所有类型的应用吗？
A: 不是。以下场景可能不适合：

### 1. 计算密集型应用
- 需要持续高CPU计算的应用
- 视频转码等重量级处理
- 机器学习训练任务

### 2. 需要长连接的应用
- 实时游戏服务器
- WebSocket密集型应用
- 需要保持大量TCP连接的服务

### 3. 有状态且对延迟敏感的应用
- 需要共享内存的应用
- 对延迟要求在毫秒级的金融交易系统
- 需要大量内存缓存的应用

### 4. 具有特殊系统要求的应用
- 需要特定系统库的应用
- 需要直接访问硬件的应用
- 需要特定运行时环境的遗留系统

## 总结
目前讨论的是如何基于serverless架构进行选型。你可能会有疑问，如果我的服务是一个传统架构的服务，比如nestjs服务，我怎么才能利用serverless架构的优势？需要完全重构么？

其实不需要完全重构。服务的lambda化，侵入性很低。

那么怎么搭建/改造一个 nestjs 后端服务成为一个基于aws lambda 的 serverless 架构的服务？

下一章，介绍如何搭建一个基于 aws lambda 的 nestjs 后端服务。