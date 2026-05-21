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
| [LLM 基础](./00-foundations/00-llm基础.md) | 大模型是什么、怎么工作的 | ✅ 完整 |
| [Embedding 与向量化](./00-foundations/01-embedding.md) | 向量化原理、模型选型、RAG 底层 | ✅ 完整 |
| [提示词工程](./00-foundations/02-提示词工程.md) | 怎么写出好的 Prompt | ✅ 完整 |
| [结构化输出](./00-foundations/03-结构化输出.md) | JSON Mode / Pydantic，稳定输出结构化数据 | ✅ 完整 |
| [微调与 LoRA](./00-foundations/04-微调与LoRA.md) | 怎么让模型适配特定任务 | ✅ 完整 |
| [Tokenization](./00-foundations/05-Tokenization.md) | Token 原理、计算规律、上下文窗口管理 | 📝 待写 |

### Agent 核心技术

| 章节 | 核心问题 | 状态 |
|------|---------|------|
| [Function Call](./01-core/00-function-call.md) | 模型怎么调用外部函数 | ✅ 完整 |
| [MCP](./01-core/01-mcp.md) | 工具调用如何标准化 | ✅ 完整 |
| [RAG 技术](./01-core/02-rag.md) | 怎么给模型接外部知识 | ✅ 完整 |
| [Context Engineering](./01-core/03-context-engineering.md) | 上下文怎么管理和设计 | ✅ 完整 |
| [Agent 框架](./01-core/04-agent框架.md) | 有哪些主流框架、怎么选 | ✅ 完整 |
| [Multi-Agent](./01-core/05-multi-agent.md) | 多个 Agent 怎么协作 | ✅ 完整 |
| [Agent Skill](./01-core/06-agent-skill.md) | Agent 能力怎么模块化 | ✅ 完整 |
| [记忆系统](./01-core/07-记忆系统.md) | Agent 怎么记住信息、存哪里、怎么选型 | 📝 待写 |
| [规划策略](./01-core/08-规划策略.md) | ReAct / CoT / ToT / Plan-and-Execute 对比 | 📝 待写 |
| [安全与 Guardrails](./01-core/09-安全与Guardrails.md) | Prompt 注入防护、输出过滤、安全边界 | 📝 待写 |

### 工程实践

| 章节 | 核心问题 | 状态 |
|------|---------|------|
| [本地模型部署](./02-engineering/00-本地模型部署.md) | Ollama / vLLM，数据不出本地 | ✅ 完整 |
| [n8n / Coze / Dify 对比](./02-engineering/01-n8n-coze-dify.md) | 三大 AI 应用平台选型指南 | ✅ 完整 |
| [Vibe Coding 技巧](./02-engineering/02-vibe-coding.md) | AI 辅助开发实践 | ✅ 完整 |
| [可观测性](./02-engineering/03-可观测性.md) | 生产环境 LLM 追踪、调试、监控 | 📝 待写 |
| [Prompt 版本管理](./02-engineering/04-Prompt版本管理.md) | 工程化管理 Prompt，支持版本和 A/B 测试 | 📝 待写 |

### 评估体系

| 章节 | 核心问题 | 状态 |
|------|---------|------|
| [Harness Engineering](./03-evaluation/00-harness-engineering.md) | Agent 怎么评测和保障质量 | ✅ 完整 |
| [Agent 性能评估](./03-evaluation/01-agent性能评估.md) | 从哪些维度评估 Agent | ✅ 完整 |
| [LLM-as-Judge](./03-evaluation/02-LLM-as-Judge.md) | 用大模型评估大模型的方法与局限 | 📝 待写 |

---

## 状态说明

- ✅ 完整：内容已写完，可直接阅读
- 🚧 填充中：框架在，内容需补充
- 📝 待写：只有标题，还没开始
