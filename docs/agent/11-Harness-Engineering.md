# Harness Engineering

> 目标：理解 Agent 测试评估的核心方法，能够搭建系统化的评测体系。

---

## 一、为什么 Agent 测试难？

普通软件测试：输入确定 → 输出确定 → 对比断言。

Agent 测试的挑战：
- LLM 输出具有**随机性**，同样输入不一定同样输出
- 工具调用**链路长**，中间任何一步出错都影响结果
- **正确性难定义**：回答"合理"和"正确"之间的边界模糊
- **幻觉难检测**：模型可能给出听起来对但实际错的答案

---

## 二、测试层次

```
                    ┌──────────────────┐
                    │   端到端测试      │  ← 完整用户场景，最接近真实
                    ├──────────────────┤
                    │   工具调用测试    │  ← 工具选择和参数是否正确
                    ├──────────────────┤
                    │   单元测试        │  ← 每个工具函数的逻辑
                    └──────────────────┘
```

---

## 三、评测维度

| 维度 | 说明 | 评测方式 |
|------|------|---------|
| **准确性** | 回答是否正确 | 对比标准答案 / LLM-as-Judge |
| **工具调用正确性** | 是否调用了正确的工具和参数 | 断言检查 tool_calls |
| **完整性** | 是否覆盖了用户问题的所有要点 | LLM-as-Judge |
| **幻觉率** | 是否生成了不在上下文中的信息 | 对照检查 |
| **延迟** | 端到端响应时间 | 计时 |
| **成本** | Token 消耗 | 统计 API 用量 |
| **鲁棒性** | 面对模糊/错误输入的表现 | 边界用例测试 |

---

## 四、LLM-as-Judge

用另一个 LLM 对 Agent 输出打分，是目前最实用的自动化评测方式。

```python
JUDGE_PROMPT = """
你是一个客观的评判者。请根据以下标准对 AI 回答打分（1-5分）：

问题：{question}
参考答案：{reference}
AI 回答：{response}

评分标准：
- 5分：完全正确，信息准确，表达清晰
- 3分：部分正确，有小错误或遗漏
- 1分：答非所问或存在明显错误

请直接返回 JSON：{{"score": <分数>, "reason": "<原因>"}}
"""

def llm_judge(question, reference, response) -> dict:
    result = llm.invoke(JUDGE_PROMPT.format(...))
    return json.loads(result.content)
```

**注意**：Judge LLM 要用比被测 Agent 更强的模型，避免"用弱判强"。

---

## 五、测试数据集构建

### 黄金数据集（Golden Dataset）

手工构建，每条包含：
```json
{
  "input": "帮我查一下李明上个月的报销总额",
  "expected_tools": ["query_expense_records"],
  "expected_tool_args": {"user_name": "李明", "month": "2024-04"},
  "expected_output_contains": ["李明", "报销", "金额"],
  "reference_answer": "李明上个月共报销 3 笔，合计 2,580 元"
}
```

### 自动生成测试用例

```python
# 用 LLM 从真实对话中生成测试用例
generator_prompt = """
基于以下真实用户对话，生成 5 个类似的测试用例，每个包含：
- 用户问题变体
- 期望调用的工具
- 期望的回答要点

对话：{real_conversation}
"""
```

---

## 六、回归测试流程

```
代码变更（Prompt / 工具 / 模型）
  ↓
运行黄金数据集（自动化）
  ↓
计算指标：准确率 / 工具调用正确率 / 平均分
  ↓
与 baseline 对比
  ├── 指标下降 > 阈值 → 阻断上线，人工 review
  └── 指标持平或提升 → 允许上线
```

---

## 七、常用工具

| 工具 | 用途 |
|------|------|
| **LangSmith** | LangChain 官方，追踪 Agent 调用链、管理测试数据集 |
| **Braintrust** | 独立评测平台，支持 LLM-as-Judge、版本对比 |
| **Promptfoo** | 开源 Prompt 测试工具，命令行友好 |
| **Pytest** | 工具函数单元测试，最基础的保障 |
| **自建日志** | 记录每次 Agent 运行的完整 trace，用于分析失败案例 |

---

## 八、最小可行评测体系

如果资源有限，至少要做这三件事：

1. **工具调用断言**：验证关键场景下模型调用了正确的工具和参数（用 pytest）
2. **30 条黄金数据集**：覆盖主流程的核心场景，每次上线前跑一遍
3. **线上日志采样**：每天抽 10% 的真实请求做人工抽查

---

## 参考资料

- [LangSmith 文档](https://docs.smith.langchain.com/)
- [Promptfoo](https://github.com/promptfoo/promptfoo)
- [RAGAS（RAG 专项评测）](https://github.com/explodinggradients/ragas)
- [Braintrust](https://www.braintrustdata.com/)
