# Multi-Agent

> 目标：理解多 Agent 系统的必要性、拓扑结构、协作模式和工程挑战。

---

## 一、为什么需要多 Agent？

单个 Agent 的局限：
- 上下文窗口有限，任务太长装不下
- 一个 LLM 同时扮演多个角色，容易混乱
- 无法并行处理多个子任务
- 不同任务需要不同模型或工具

**多 Agent 的思路**：拆分职责，每个 Agent 只做一件事，通过协作完成复杂任务。

---

## 二、常见拓扑结构

### 主从模式（Orchestrator-Worker）

```
Orchestrator（规划 + 分配任务）
  ├── Worker A（数据查询）
  ├── Worker B（文档生成）
  └── Worker C（审批判断）
```

**特点**：最常用，Orchestrator 负责全局规划，Worker 专注具体执行。

---

### 流水线模式（Pipeline）

```
Agent A（提取信息）→ Agent B（分析）→ Agent C（生成报告）
```

**特点**：每个 Agent 的输出是下一个的输入，适合有明确步骤的流程。

---

### 对等协作模式（Peer-to-Peer）

```
Agent A ←→ Agent B
    ↕           ↕
Agent C ←→ Agent D
```

**特点**：Agent 之间可以互相发消息、互相调用，适合需要互相验证的场景（如代码审查）。

---

## 三、典型协作模式

### 角色分工

```
用户请求："帮我分析这份报销数据并生成月报"

Planner Agent：拆解任务
  → Data Agent：查询数据库，返回原始数据
  → Analyst Agent：分析数据，找出异常
  → Writer Agent：生成报告文本
  → Reviewer Agent：审查报告质量
```

### 辩论模式（用于提升准确性）

```
同一问题 → Agent A 给出答案 → Agent B 质疑和反驳 → Agent A 修正
```

用于需要高准确性的判断任务（医疗、法律、财务合规）。

---

## 四、通信方式

| 方式 | 实现 | 适用场景 |
|------|------|---------|
| **直接函数调用** | A 调用 B 的接口 | 同进程、简单场景 |
| **消息队列** | Redis / RabbitMQ | 异步、解耦、高并发 |
| **共享状态** | 数据库 / 内存 | 需要多 Agent 读写同一状态 |
| **对话消息** | AutoGen 的 ChatMessage | 模拟人类对话协作 |

---

## 五、主流框架支持

| 框架 | Multi-Agent 支持 |
|------|----------------|
| **LangGraph** | 用图节点表示 Agent，边表示消息流，支持循环和条件路由 |
| **AutoGen** | 原生支持多 Agent 对话，ConversableAgent 相互发消息 |
| **CrewAI** | 定义角色（Role）+ 任务（Task），自动分配和协作 |

---

## 六、工程挑战

| 挑战 | 说明 | 应对方式 |
|------|------|---------|
| **状态管理** | 多个 Agent 共享状态容易冲突 | 用统一状态存储，加锁或事件溯源 |
| **错误传播** | 一个 Agent 出错可能导致整个流程失败 | 每个节点加错误处理和重试 |
| **调试困难** | 多 Agent 的调用链很难追踪 | 接入 LangSmith / 自建日志追踪 |
| **成本失控** | 多 Agent 并行调用 LLM，费用叠加 | 设置 token 预算，非关键任务用小模型 |
| **一致性** | 多 Agent 并发修改同一资源 | 任务级别加锁，或设计为无副作用 |

---

## 参考资料

- [LangGraph Multi-Agent](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/multi-agent-collaboration/)
- [AutoGen 文档](https://microsoft.github.io/autogen/stable/)
- [CrewAI 文档](https://docs.crewai.com/)
