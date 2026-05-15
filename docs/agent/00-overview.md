# AI Agent 学习路线

> 从大模型基础出发，逐步深入到 Agent 应用层。每章独立，按序学习效果最佳。

---

## 学习路线图

```
大模型基础
  │
  ├── 01 LLM 基础          ← Transformer、预训练、推理参数
  ├── 02 提示词工程         ← 如何有效与大模型沟通
  ├── 03 微调与 LoRA        ← 让大模型适配特定任务
  │
  ▼
知识增强
  │
  ├── 04 RAG 技术           ← 给大模型接入外部知识库
  │
  ▼
工具调用
  │
  ├── 05 Function Call      ← 大模型调用外部函数
  ├── 06 MCP                ← 工具调用标准化协议
  │
  ▼
Agent 应用
  │
  ├── 07 Agent 框架         ← LangChain、AutoGen 等
  ├── 08 Multi-Agent        ← 多智能体协作
  ├── 09 Context Engineering← 上下文管理与设计
  ├── 10 Agent Skill        ← Agent 能力模块化
  └── 11 Harness Engineering← Agent 评测与质量保障

扩展知识
  │
  ├── 12 Embedding 与向量化  ← RAG 底层原理，模型选型与代码
  ├── 13 结构化输出           ← JSON Mode / Pydantic，让输出可解析
  └── 14 本地模型部署         ← Ollama / llama.cpp / vLLM，数据不出本地
```

---

## 各章概览

| # | 章节 | 核心问题 | 状态 |
|---|------|---------|------|
| 01 | [LLM 基础](./01-LLM基础.md) | 大模型是什么、怎么工作的 | ✅ 完整 |
| 02 | [提示词工程](./02-提示词工程.md) | 怎么写出好的 Prompt | ✅ 完整 |
| 03 | [微调与 LoRA](./03-微调与LoRA.md) | 怎么让模型适配特定任务 | ✅ 完整 |
| 04 | [RAG 技术](./04-RAG技术.md) | 怎么给模型接外部知识 | ✅ 完整 |
| 05 | [Function Call](./05-FunctionCall.md) | 模型怎么调用外部函数 | ✅ 完整 |
| 06 | [MCP](./06-MCP.md) | 工具调用如何标准化 | ✅ 完整 |
| 07 | [Agent 框架](./07-Agent框架.md) | 有哪些主流框架、怎么选 | ✅ 完整 |
| 08 | [Multi-Agent](./08-Multi-Agent.md) | 多个 Agent 怎么协作 | ✅ 完整 |
| 09 | [Context Engineering](./09-Context-Engineering.md) | 上下文怎么管理和设计 | ✅ 完整 |
| 10 | [Agent Skill](./10-Agent-Skill.md) | Agent 能力怎么模块化 | ✅ 完整 |
| 11 | [Harness Engineering](./11-Harness-Engineering.md) | Agent 怎么评测和保障质量 | ✅ 完整 |
| 12 | [Embedding 与向量化](./12-Embedding与向量化.md) | 向量化原理、模型选型、RAG 底层 | ✅ 完整 |
| 13 | [结构化输出](./13-结构化输出.md) | JSON Mode / Pydantic，稳定输出结构化数据 | ✅ 完整 |
| 14 | [本地模型部署](./14-本地模型部署.md) | Ollama / vLLM，数据不出本地 | ✅ 完整 |

---

## 状态说明

- ✅ 完整：内容已写完，可直接阅读
- 🚧 填充中：框架在，内容需补充
- 📝 待写：只有标题，还没开始
