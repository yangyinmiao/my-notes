# Agent 框架

> 目标：理解 Agent 的核心架构、主流框架对比，以及什么场景用什么框架。

---

## 一、什么是 AI Agent？

**Agent = LLM + 记忆 + 工具 + 规划**

LLM 只会"说话"，Agent 能"做事"：感知环境 → 规划目标 → 调用工具 → 执行行动 → 观察结果 → 循环。

```
用户输入
  ↓
LLM 规划（思考：我需要做什么？）
  ↓
需要工具？
  ├── 是 → 调用工具 → 获取结果 → 回到 LLM 规划
  └── 否 → 生成最终回复
```

---

## 二、核心组件

| 组件 | 作用 | 实现方式 |
|------|------|---------|
| **LLM** | 大脑，负责规划和推理 | GPT-4o / Claude / Llama3 |
| **Memory** | 记住上下文和历史 | 短期：对话历史；长期：向量数据库 |
| **Tools** | 执行具体动作 | Function Call / MCP |
| **Planning** | 决定行动步骤 | ReAct / CoT / Plan-and-Execute |

---

## 三、规划模式

### ReAct（最主流）

```
Thought: 我需要查询用户的报销记录
Action: query_expense_records(user_id=123)
Observation: 找到 3 条记录，最近一条状态为"待审批"
Thought: 已有结果，可以回答用户
Answer: 您有 3 条报销记录，最新一条正在审批中
```

**特点**：思考和行动交替，每步都有观察，适合需要多步工具调用的任务。

### Plan-and-Execute

```
1. 先规划完整步骤（生成任务列表）
2. 按步骤依次执行
```

**特点**：适合长流程、步骤明确的任务，比 ReAct 更可控。

---

## 四、主流框架对比

| 框架 | 定位 | 优点 | 缺点 | 适用场景 |
|------|------|------|------|---------|
| **LangChain** | 全能工具箱 | 生态最大、组件最多 | 抽象层多、调试复杂 | 快速原型 |
| **LangGraph** | 有状态的 Agent 图 | 流程可控、支持循环 | 学习曲线 | 复杂多步 Agent |
| **AutoGen** | 多 Agent 对话 | 多 Agent 协作简单 | 对话管理复杂 | 多角色协作任务 |
| **CrewAI** | 角色扮演多 Agent | 上手快、角色清晰 | 定制性弱 | 快速搭多 Agent |
| **Dify** | 无代码/低代码平台 | 可视化编排、开箱即用 | 灵活性受限 | 业务人员、快速交付 |
| **原生实现** | 自己写循环 | 完全可控 | 工作量大 | 生产级、定制需求 |

**选型建议：**
- 学习阶段 → LangChain（资料最多）
- 生产级复杂 Agent → LangGraph 或原生实现
- 多 Agent 协作 → AutoGen 或 CrewAI
- 给业务部门用 → Dify

---

## 五、LangChain 核心概念速查

```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model="gpt-4o")

# 定义工具
@tool
def search_expense(expense_id: str) -> str:
    """根据 ID 查询报销单"""
    return db.get_expense(expense_id)

# 创建 Agent
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是报销助手"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

agent = create_tool_calling_agent(llm, [search_expense], prompt)
executor = AgentExecutor(agent=agent, tools=[search_expense], verbose=True)

result = executor.invoke({"input": "查一下 EXP-001 的状态"})
```

---

## 六、Agent 的常见问题

| 问题 | 原因 | 解决方式 |
|------|------|---------|
| **无限循环** | 工具调用失败但模型不知道如何终止 | 设置 max_iterations 上限 |
| **幻觉工具调用** | 模型编造不存在的工具或参数 | 工具描述要精确，添加参数校验 |
| **上下文溢出** | 多轮工具调用累积太多 token | 截断历史或压缩中间步骤 |
| **效果不稳定** | 规划受随机性影响 | 降低 temperature，用结构化输出 |

---

## 参考资料

- [LangChain 文档](https://python.langchain.com/docs/)
- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [AutoGen 文档](https://microsoft.github.io/autogen/)
- [Dify 文档](https://docs.dify.ai/zh-hans)
