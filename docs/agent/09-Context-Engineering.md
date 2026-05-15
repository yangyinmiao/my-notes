# Context Engineering

> 目标：理解上下文管理的核心问题、设计原则，以及在实际 Agent 开发中的工程实践。

---

## 一、什么是 Context Engineering？

LLM 只能看到输入给它的内容——上下文窗口里有什么，它才能用什么。

**Context Engineering** 是指：**有意识地设计、构建、管理送入 LLM 的上下文内容**，让模型在有限的窗口内获得最有用的信息。

> "垃圾进，垃圾出。"上下文的质量直接决定输出质量。

---

## 二、上下文的组成

```
┌─────────────────────────────────────────┐
│              Context Window              │
│                                          │
│  [System Prompt]     ← 角色、规则、背景  │
│  [Memory]            ← 长期记忆摘要      │
│  [Retrieved Docs]    ← RAG 检索结果      │
│  [Tool Results]      ← 工具调用历史      │
│  [Conversation]      ← 对话历史          │
│  [User Input]        ← 当前用户输入      │
│                                          │
└─────────────────────────────────────────┘
                    ↓
              有限的 Token 预算
```

---

## 三、核心问题

### 3.1 上下文太长

超出窗口限制，或成本、延迟增加。

**应对策略：**

| 策略 | 做法 |
|------|------|
| **截断** | 保留最近 N 轮对话，丢弃早期历史 |
| **摘要压缩** | 用 LLM 把早期对话压缩成摘要 |
| **滑动窗口** | 始终保留最近 K 个 token |
| **选择性保留** | 只保留包含关键信息的轮次 |

### 3.2 信息不足

模型没有回答问题所需的背景知识。

**应对策略：**
- RAG 检索相关文档
- 工具调用获取实时数据
- 在 System Prompt 中预埋常用背景

### 3.3 信息冗余/干扰

上下文中充斥无关内容，稀释有效信息，导致模型"注意力分散"。

**应对策略：**
- 上下文压缩：只传关键句而不是整段
- Rerank 后只传 Top-K 最相关的文档块
- 工具结果精简：只传模型需要的字段

---

## 四、记忆系统设计

| 类型 | 内容 | 存储 | 生命周期 |
|------|------|------|---------|
| **工作记忆** | 当前对话上下文 | 内存 / 变量 | 本次会话 |
| **情节记忆** | 历史对话摘要 | 数据库 | 跨会话 |
| **语义记忆** | 用户偏好、知识点 | 向量数据库 | 长期 |
| **程序记忆** | 如何完成某类任务 | Prompt / 代码 | 长期 |

**实践**：工作记忆超出阈值时，触发摘要压缩写入情节记忆；检索时从情节记忆和语义记忆联合召回。

---

## 五、System Prompt 设计原则

1. **角色先行**：第一句定义模型是谁、能做什么
2. **规则明确**：不要用"尽量"，用"必须"/"不得"
3. **背景精简**：只放模型每次都需要的信息，动态信息用 RAG
4. **输出格式固定**：明确指定 JSON schema 或输出结构
5. **Few-shot 放这里**：固定的示例放 System Prompt，而不是每次 User 消息里带

```
# 示例 System Prompt 结构
你是[角色]，负责[职责]。

## 规则
- 必须...
- 不得...

## 输出格式
返回 JSON，字段如下：
{"status": "approve|reject", "reason": "..."}

## 背景信息
当前日期：{date}
用户部门：{department}
```

---

## 六、Token 预算管理

```python
MAX_CONTEXT = 8000  # 留出输出空间

def build_context(system, history, retrieved_docs, user_input):
    budget = MAX_CONTEXT
    budget -= count_tokens(system)
    budget -= count_tokens(user_input)
    budget -= 500  # 预留给工具调用结果

    # 优先级：retrieved_docs > 近期历史 > 早期历史
    docs_tokens = count_tokens(retrieved_docs)
    if docs_tokens < budget * 0.5:
        context_docs = retrieved_docs
    else:
        context_docs = truncate_to_budget(retrieved_docs, budget * 0.5)

    remaining = budget - count_tokens(context_docs)
    context_history = trim_history(history, remaining)

    return system + context_docs + context_history + user_input
```

---

## 参考资料

- [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
- [LangChain Memory 文档](https://python.langchain.com/docs/how_to/#memory)
