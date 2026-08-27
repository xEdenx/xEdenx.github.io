---
title: Multi-Agent 不像组织架构，更像同一个模型的会议纪要
description: 没有真实边界的 Multi-Agent，通常只是同一模型在不同 prompt 里的会议纪要。
publishDate: 2026-07-02
tags:
  - AI暴论
  - AI Engineering
  - Multi-Agent
  - LLM
  - 软件架构
  - Kernel Panic
---

很多 Multi-Agent 系统没有增加新的能力。

它们只是把同一个模型，放进了三个不同的 prompt。

一个负责规划。  
一个负责执行。  
一个负责审查。

看起来像组织。实际上是同一套 weights，在不同窗口里重复阅读问题。

Planner 不会凭空知道更多业务。  
Reviewer 也不会突然拥有不同的判断力。  
它们共享同一代模型的盲点，只是不共享 token。

角色不是边界。  
prompt 不是权限。  
两个 Agent 互相发消息，也不等于两个系统真的独立。

没有独立的权限、独立的 Context、并发任务、专用工具或不同的失败域，Multi-Agent 通常只是在把一次推理拆成一段 workflow。

token 变多。  
latency 变长。  
state 开始同步。  
错误开始转发。  
最后多出一套需要解释的 orchestration。

真正需要 Multi-Agent 的场景并不神秘。

一个 Agent 可以支付。  
一个只能读取。  
一个处理隔离的客户数据。  
一个任务必须并发，另一个必须独立验证。

这时拆分的是现实世界的边界。

其他时候，拆分的往往只是开发者对单个模型的不放心。

一个强模型。  
正确的 Context。  
一组能执行真实工作的 Tools。  
一个能停止的 Loop。

这比三个名字不同、头像不同、上下文不同的 Agent 更像系统。

把角色名称删掉以后，业务里究竟少了什么？
