# AI Agent 学习路线

> 从大模型基础出发，逐步深入到 Agent 应用层。每章独立，按序学习效果最佳。

---

## 学习路线图

```
大模型基础
  │
  ├── LLM 基础          ← Transformer、预训练、推理参数
  ├── 提示词工程         ← 如何有效与大模型沟通
  ├── 微调与 LoRA        ← 让大模型适配特定任务
  ├── Embedding 与向量化  ← RAG 底层原理，模型选型
  └── 结构化输出          ← JSON Mode / Pydantic，让输出可解析
  │
  ▼
Agent 核心技术
  │
  ├── RAG 技术           ← 给大模型接入外部知识库
  ├── Function Call      ← 大模型调用外部函数
  ├── MCP                ← 工具调用标准化协议
  ├── Context Engineering← 上下文管理与设计
  ├── Agent 框架         ← LangChain、AutoGen 等
  ├── Multi-Agent        ← 多智能体协作
  └── Agent Skill        ← Agent 能力模块化
  │
  ▼
工程实践
  │
  ├── 本地模型部署        ← Ollama / llama.cpp / vLLM
  ├── n8n / Coze / Dify  ← 三大 AI 应用平台选型对比
  └── Vibe Coding 技巧   ← AI 辅助开发实践
  │
  ▼
评估体系
  │
  ├── Harness Engineering← Agent 评测与质量保障
  └── Agent 性能评估     ← Benchmark 与评估维度
```

---

## 各章概览

### 大模型基础

| 章节 | 核心问题 | 状态 |
|------|---------|------|
| [LLM 基础](./foundations/llm基础.md) | 大模型是什么、怎么工作的 | ✅ 完整 |
| [Embedding 与向量化](./foundations/embedding.md) | 向量化原理、模型选型、RAG 底层 | ✅ 完整 |
| [提示词工程](./foundations/提示词工程.md) | 怎么写出好的 Prompt | ✅ 完整 |
| [结构化输出](./foundations/结构化输出.md) | JSON Mode / Pydantic，稳定输出结构化数据 | ✅ 完整 |
| [微调与 LoRA](./foundations/微调与LoRA.md) | 怎么让模型适配特定任务 | ✅ 完整 |

### Agent 核心技术

| 章节 | 核心问题 | 状态 |
|------|---------|------|
| [Function Call](./core/function-call.md) | 模型怎么调用外部函数 | ✅ 完整 |
| [MCP](./core/mcp.md) | 工具调用如何标准化 | ✅ 完整 |
| [RAG 技术](./core/rag.md) | 怎么给模型接外部知识 | ✅ 完整 |
| [Context Engineering](./core/context-engineering.md) | 上下文怎么管理和设计 | ✅ 完整 |
| [Agent 框架](./core/agent框架.md) | 有哪些主流框架、怎么选 | ✅ 完整 |
| [Multi-Agent](./core/multi-agent.md) | 多个 Agent 怎么协作 | ✅ 完整 |
| [Agent Skill](./core/agent-skill.md) | Agent 能力怎么模块化 | ✅ 完整 |

### 工程实践

| 章节 | 核心问题 | 状态 |
|------|---------|------|
| [本地模型部署](./engineering/本地模型部署.md) | Ollama / vLLM，数据不出本地 | ✅ 完整 |
| [n8n / Coze / Dify 对比](./engineering/n8n-coze-dify.md) | 三大 AI 应用平台选型指南 | ✅ 完整 |
| [Vibe Coding 技巧](./engineering/vibe-coding.md) | AI 辅助开发实践 | ✅ 完整 |

### 评估体系

| 章节 | 核心问题 | 状态 |
|------|---------|------|
| [Harness Engineering](./evaluation/harness-engineering.md) | Agent 怎么评测和保障质量 | ✅ 完整 |
| [Agent 性能评估](./evaluation/agent性能评估.md) | 从哪些维度评估 Agent | ✅ 完整 |

---

## 状态说明

- ✅ 完整：内容已写完，可直接阅读
- 🚧 填充中：框架在，内容需补充
- 📝 待写：只有标题，还没开始
