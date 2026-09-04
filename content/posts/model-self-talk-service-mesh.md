---
title: 三个 Pod，不会让一段推理变成架构
description: 一段思考被拆进消息队列后，最先被丢掉的是 Context，而不是系统复杂度。
publishDate: 2026-08-30
tags:
  - AI暴论
  - AI Engineering
  - Agent
  - Microservices
  - 分布式系统
  - 软件架构
  - Kernel Panic
---

模型写出计划。

系统把计划塞进 Kafka。

下一个模型收到一段 JSON，重新猜一次前一个模型刚刚理解过的东西。

分布式系统从这里开始。

智能没有。

Planner、Executor、Reviewer。

三个名字。  
三次调用。  
三套 Deployment。  
一条看起来很专业的链路。

模型没有得到新的事实。  
没有拿到新的权限。  
没有接住新的后果。

它只是被要求把自己的思考，转述给下一个自己。

Context 在第一跳开始缩水。

原本一个模型看过的代码、错误、工具返回和业务条件，被压成一个 task。  
后面的模型读到 task，以为自己拿到了上下文。

它拿到的是上下文的尸体。

微服务不是把东西拆开。

微服务是把责任钉死。

谁拥有订单状态。  
谁可以付款。  
谁必须幂等。  
谁的故障不能拖垮其他系统。

这些东西拆开以后，现实会变。

Planner 被删掉，现实不会变。

Reviewer 被删掉，权限不会变。

Executor 换一个 Pod，订单也不会自动多一层保障。

没有独立状态、独立权限和独立失败后果的 Agent，不是服务。

是一个穿着 Helm Chart 的 prompt。
