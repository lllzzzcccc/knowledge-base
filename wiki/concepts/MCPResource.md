---
title: "MCP 资源 (Resources)"
type: concept
tags: [MCP, AI, 上下文]
sources: [mcp-explained]
last_updated: 2026-09-26
---

# MCP 资源 (Resources)

Resources 是 [[ModelContextProtocol]] 中向模型提供上下文数据的组件,本质是可访问的静态/动态数据:本地文件、代码仓库、文档知识库、数据库表、配置文件、API 返回数据等。核心特点是**"只提供数据,不执行操作"**——不直接改变系统状态,仅为模型补充上下文(区别于可执行的 [[MCPTool]])。

## 资源标识(URI)

每个资源有唯一 URI 定位:

```
resource://<provider>/<path>

# 示例
resource://filesystem/src/main.py
resource://docs/project_guide
resource://database/orders
```

- `provider`:资源提供方或来源类型(filesystem、docs、database)
- `path`:资源在该提供方下的具体路径或标识

## 元数据(Metadata)

描述资源属性与用途,服务于权限控制、审计与访问策略优化:名称(name)、大小(size)、类型(text/code/数据库表/日志/API)、访问权限(permissions)、创建/修改时间(created_at/updated_at)、描述信息(description)。

## 资源内容(Content)

Resource 的核心部分。Client 请求资源时,Server 返回内容并加入模型上下文,模型基于数据推理(如读取 `resource://filesystem/src/main.py` 后给出代码解释或修改建议)。

## 访问控制(Access Control)

- permissions:read-only、restricted 等
- visibility:public 或 private
- allowed_roles:允许访问的用户角色

## 来源

- [[mcp-explained]]
