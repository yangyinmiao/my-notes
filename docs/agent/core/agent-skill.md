# Agent Skill

> 目标：理解 Agent Skill 的概念、分类，以及如何设计和复用技能模块。

---

## 一、什么是 Agent Skill？

**Agent Skill** 是 Agent 能力的模块化封装——把一类可复用的能力打包成独立单元，按需挂载到 Agent 上。

**类比**：如果 Agent 是一个员工，Skill 就是他的"专业技能证书"——不同岗位挂不同技能，技能可以复用。

---

## 二、Skill 的分类

### 工具类（Tool Skills）
调用外部系统获取信息或执行操作：

| Skill | 能力 |
|-------|------|
| Web Search | 搜索互联网 |
| Code Executor | 执行 Python 代码 |
| File Manager | 读写文件 |
| Database Query | 查询数据库 |
| API Caller | 调用第三方 API |

### 认知类（Cognitive Skills）
增强模型推理和处理能力：

| Skill | 能力 |
|-------|------|
| Summarizer | 长文摘要 |
| Classifier | 文本分类 |
| Extractor | 信息抽取（实体、关系） |
| Translator | 多语言翻译 |
| Planner | 任务拆解与规划 |

### 记忆类（Memory Skills）
管理上下文和长期记忆：

| Skill | 能力 |
|-------|------|
| Short-term Memory | 管理对话上下文 |
| Long-term Memory | 向量数据库存取 |
| Episodic Memory | 历史对话摘要检索 |

---

## 三、Skill 与 Function Call / MCP 的关系

```
Function Call ← 底层机制：LLM 如何调用函数
MCP           ← 标准协议：工具如何注册和发现
Agent Skill   ← 业务抽象：一组相关工具+逻辑的封装
```

**Skill 是更高层的概念**，一个 Skill 可以包含多个 Function Call 或 MCP 工具，以及调用它们的业务逻辑。

---

## 四、如何设计一个 Skill

好的 Skill 应该：
1. **单一职责**：只做一件事
2. **接口清晰**：输入输出定义明确
3. **可复用**：不依赖特定 Agent 的内部状态
4. **可测试**：独立可验证

```python
class ExpenseQuerySkill:
    """报销查询技能：根据条件查询报销单列表"""

    name = "expense_query"
    description = "查询报销单，支持按状态、时间、申请人筛选"

    def get_tools(self) -> list[Tool]:
        return [
            Tool(name="query_by_status", func=self.query_by_status, ...),
            Tool(name="query_by_user",   func=self.query_by_user,   ...),
            Tool(name="get_detail",      func=self.get_detail,      ...),
        ]

    def query_by_status(self, status: str) -> list[dict]:
        return db.query(ExpenseRecord).filter_by(status=status).all()
```

---

## 五、Skill 的注册与挂载

```python
# 定义技能库
skill_registry = {
    "expense_query":  ExpenseQuerySkill(),
    "expense_submit": ExpenseSubmitSkill(),
    "approval":       ApprovalSkill(),
}

# 根据用户角色挂载对应技能
def build_agent(user_role: str) -> AgentExecutor:
    if user_role == "employee":
        skills = ["expense_query", "expense_submit"]
    elif user_role == "manager":
        skills = ["expense_query", "approval"]

    tools = []
    for skill_name in skills:
        tools.extend(skill_registry[skill_name].get_tools())

    return create_agent(tools=tools)
```

---

## 六、常见设计模式

### 技能路由（Skill Routing）
根据用户意图自动选择调用哪个 Skill，而不是把所有工具都暴露给模型（减少干扰）。

```
用户输入 → 意图识别 → 路由到对应 Skill → Skill 内部执行
```

### 技能链（Skill Chaining）
一个 Skill 的输出作为另一个 Skill 的输入：
```
OCR Skill（图片→文本）→ Extractor Skill（文本→结构化数据）→ Submit Skill（提交报销）
```

---

## 参考资料

- [Semantic Kernel Skills 文档](https://learn.microsoft.com/en-us/semantic-kernel/agents/plugins/)
- [LangChain Tools 文档](https://python.langchain.com/docs/how_to/tools_builtin/)
