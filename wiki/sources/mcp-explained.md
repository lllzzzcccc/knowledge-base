---
title: "MCP 到底是什么？一文带你彻底搞懂(附:从零构建MCP实例)"
type: source
tags: [MCP, AI, 协议, Agent, 工具调用]
date: 2026-09-26
source_file: raw/articles/mcp-explained.md
---

## Summary

腾讯云开发者社区文章(原始发表于微信公众号),系统讲解 MCP(Model Context Protocol)的概念、架构、核心组件与通信机制,并附 FastMCP 从零构建实例。全文覆盖:MCP 的定位("AI 应用的 USB-C 接口")、客户端-服务器架构(MCP Client / MCP Server / Host)、三大核心原语([[MCPTool]] / [[MCPResource]] / [[MCPPrompt]])的字段规范、七步工作流程,以及三种传输方式(STDIO / SSE / Streamable HTTP)的对比与演进——SSE 已被官方弃用,由 Streamable HTTP 替代。

## Key Claims

- MCP 是由 Claude 母公司 [[Anthropic]] 推出的开放协议,用于连接 LLM 与外部工具、数据源和应用系统,标准化模型获取上下文与执行操作的方式。
- MCP 采用客户端-服务器架构:MCP Client 是"调度与通信层"(连接、发现能力、协议转换、结果回灌上下文),MCP Server 是"能力网关"(暴露 Tools/Resources 并执行具体操作);MCP Host 指启动连接的应用程序(Cursor、Claude Desktop、Cline)。
- 三大核心原语分工:[[MCPTool]] 可执行、会改变系统状态(类似函数调用);[[MCPResource]] 只读数据、只提供上下文不执行操作;[[MCPPrompt]] 标准化任务模板,让模型"不仅知道做什么,还知道如何做"。
- 工作流程七步:建立连接 → 协商沟通规则 → 客户端发现能力清单 → LLM 思考选工具并传参 → 客户端转成标准协议消息 → 服务器调外部系统执行 → 结果逐级回传、上下文更新。
- 通信协议统一为 JSON-RPC,传输方式三种:STDIO(默认,父子进程 stdin/stdout,无网络低延迟强隔离,要求纯净 JSON 输出,本质是"本地进程级 RPC 通道")、SSE(HTTP 单向流式,官方已弃用)、Streamable HTTP(推荐替代,兼顾请求-响应与流式输出,易集成认证/网关/负载均衡)。
- 工程价值不在于替代现有架构,而在于解耦模型与工具之间的交互复杂度,让开发者专注业务逻辑。

## Key Quotes

> "我们可以形象地将 MCP 看做是 AI 应用程序的 USB-C 接口" —— 对 MCP 定位的核心类比

> "STDIO 本质上是一种'本地进程级 RPC 通道'" —— STDIO 通信方式的本质概括

> "MCP 的价值不在于'替代现有架构',而在于解耦模型与工具之间的交互复杂度" —— 文末小结的工程判断

## Connections

- [[ModelContextProtocol]] — 全文主题:MCP 的定义、架构、工作流程与通信机制
- [[MCPTool]] / [[MCPResource]] / [[MCPPrompt]] — 文章核心章节展开的三大原语
- [[Anthropic]] — MCP 协议的推出方
- [[Pipe]] — STDIO 父子进程经 stdin/stdout 通信,本质上复用了操作系统匿名管道的进程间通信机制
- [[Socket]] — SSE/Streamable HTTP 基于 HTTP 长连接与网络通信,与 Socket 跨网络通信能力属于同一层的网络 IPC 范畴

## Contradictions

- 无。与现有 OS 主题互补而非冲突:本文的 STDIO 机制([[Pipe]] 的进程间通信)与网络传输([[Socket]])恰好是 [[six-ipc-mechanisms]] 中两类 IPC 在 AI 协议层的应用实例。
