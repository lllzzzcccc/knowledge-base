---
title: "MCP 工具 (Tools)"
type: concept
tags: [MCP, AI, 工具调用]
sources: [mcp-explained]
last_updated: 2026-09-26
---

# MCP 工具 (Tools)

Tools 是 [[ModelContextProtocol]] 中模型可调用的可执行能力模块,类似函数调用(function calling)。AI 在推理过程中通过 Tools 直接与外部系统交互、执行操作、处理数据。Tools 的核心特征是**可执行、会改变系统状态或触发操作**(区别于只读的 [[MCPResource]])。

## 字段规范

一个 Tool 通常包含:

- **工具标识(Tool ID/Name)**:唯一标识,便于模型或 MCP Client 调用,例如 `tool://search_docs`、`tool://run_shell_command`
- **描述(Description)**:说明工具功能和用途
- **输入参数(Input Schema)**:输入数据的格式与字段(参数类型、必填项、默认值)
- **输出结构(Output Schema)**:返回数据格式,便于 Client 整合进模型上下文或后续流程
- **执行权限和约束(Permissions/Security)**:谁可调用、是否需要额外认证(如限定角色或沙箱环境)
- **示例调用模板(可选)**:默认调用方式或示例,帮助模型正确构建调用请求

## 与 Resources 的分工

Tools 执行操作(有副作用),Resources 只提供数据(无副作用),二者共同构成 AI 可访问的数据与功能集合。LLM 在工作流程的第 4 步(思考选工具)决定调用哪个 Tool 并传入参数。

## 来源

- [[mcp-explained]]
