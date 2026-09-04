---
title: 把模型的自言自语接进 Service Mesh，不叫架构
description: 没有独立状态、权限与失败后果的 Agent 服务，只是被部署出去的思考步骤。
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

很多 Agent 系统，做的不是协作。

是给模型的自言自语分配 Pod。

一个 Agent 写计划。  
一个 Agent 调工具。  
一个 Agent 审查结果。

三个 prompt。  
三个容器。  
三套日志。  
一条消息队列。

模型没有多知道一条事实。  
系统没有多出一项权限。  
业务没有少一个失败点。

只多了一张架构图。

Kubernetes 不会让 prompt 更聪明。  
Kafka 也不会让模型多理解一段 Context。

它们只会让模型把刚刚想过的东西，压成 JSON，再发给下一个自己。

Context 开始丢失。  
state 开始复制。  
重试开始改变顺序。  
一个回答错了，终于可以在三个服务里一起错。

微服务拆的是责任。

Agent 微服务拆的，常常只是句子。

支付失败，谁承担后果。  
客户数据泄露，谁越过了权限。  
订单被重复执行，哪个系统必须回滚。

这些地方需要边界。

因为这里有真实状态。  
真实权限。  
真实失败域。

Planner 没有。

Reviewer 也没有。

如果删掉一个 Agent 服务，业务状态、权限关系和失败后果都没有变化。

它不是服务。

它只是一个被部署出去的思考步骤。

没有独立后果的 Agent 边界，叫角色扮演。
