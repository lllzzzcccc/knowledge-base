---
title: "MCP 提示词 (Prompts)"
type: concept
tags: [MCP, AI, 提示词]
sources: [mcp-explained]
last_updated: 2026-09-26
---

# MCP 提示词 (Prompts)

Prompts 是 [[ModelContextProtocol]] 中用于规范 AI 模型行为和任务执行流程的标准化模板。作用是为模型提供明确的指令和上下文格式,使模型按预期方式组织信息、调用[[MCPResource|资源]]和[[MCPTool|工具]]并生成结构化输出——模型不仅知道**要做什么**,还知道**如何做**。

## 字段规范

- **Prompt ID/名称**:唯一标识,便于复用和调用,例如 `prompt://code_review`、`prompt://generate_summary`
- **描述(Description)**:用途和适用场景
- **输入要求(Input Schema)**:执行任务所需的数据格式或上下文,例如需要引用哪些 Resources
- **输出格式(Output Schema)**:结果结构(JSON、Markdown、列表、文本块),便于后续工具调用或系统处理
- **默认行为或模板指令**:具体提示词文本/模板,可含占位符(如 `{resource_content}`、`{user_question}`),**运行时由 MCP Client 填充实际数据**

## 示例

```json
{
  "id": "prompt://code_review",
  "description": "对源代码文件进行审查,输出问题和改进建议",
  "input_schema": {
    "resources": ["resource://filesystem/src/main.py"],
    "user_prompt": "string"
  },
  "output_schema": { "issues": "list", "suggestions": "list" },
  "template": "请阅读以下代码文件内容:{resource_content}。基于最佳实践指出潜在问题,并提出改进建议。"
}
```

## 来源

- [[mcp-explained]]
