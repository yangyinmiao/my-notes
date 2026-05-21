# Function Call

> 目标：理解 Function Call 的工作原理、数据格式，以及如何在项目中使用。

---

## 一、是什么？

**Function Call** 允许 LLM 在生成回复时，决定"我需要调用一个外部函数"，并输出结构化的调用参数，由开发者代码执行后将结果返回给模型。

**本质**：LLM 不能直接执行代码，它只是输出"我想调用什么函数、传什么参数"的 JSON，真正执行靠你的代码。

---

## 二、完整交互流程

```
1. 开发者定义函数列表（工具描述）传给模型
        ↓
2. 用户发送消息
        ↓
3. 模型判断：需要调用工具？
   ├── 否 → 直接生成文本回复，结束
   └── 是 → 输出 tool_call（函数名 + 参数 JSON）
        ↓
4. 开发者代码执行该函数，获取结果
        ↓
5. 将函数结果作为 tool message 追加到对话
        ↓
6. 模型根据函数结果生成最终回复
```

---

## 三、代码示例（OpenAI API）

### 定义工具

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_expense_status",
            "description": "查询报销单的审批状态",
            "parameters": {
                "type": "object",
                "properties": {
                    "expense_id": {
                        "type": "string",
                        "description": "报销单 ID"
                    }
                },
                "required": ["expense_id"]
            }
        }
    }
]
```

### 发起请求

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "帮我查一下 EXP-2024-001 的审批进度"}],
    tools=tools,
    tool_choice="auto"   # auto / none / required
)
```

### 处理工具调用

```python
message = response.choices[0].message

if message.tool_calls:
    tool_call = message.tool_calls[0]
    func_name = tool_call.function.name          # "get_expense_status"
    func_args = json.loads(tool_call.function.arguments)  # {"expense_id": "EXP-2024-001"}

    # 执行函数
    result = get_expense_status(**func_args)

    # 将结果追加回对话
    messages.append(message)  # 模型的 tool_call 消息
    messages.append({
        "role": "tool",
        "tool_call_id": tool_call.id,
        "content": json.dumps(result, ensure_ascii=False)
    })

    # 再次请求，让模型生成最终回复
    final_response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages
    )
```

---

## 四、tool_choice 参数

| 值 | 含义 |
|----|------|
| `"auto"` | 模型自己决定要不要调用工具（默认） |
| `"none"` | 禁止调用工具，强制生成文本 |
| `"required"` | 必须调用工具 |
| `{"type": "function", "function": {"name": "xxx"}}` | 强制调用指定函数 |

---

## 五、并行工具调用

模型可以在一次回复中输出多个 tool_call，需要并行执行：

```python
if message.tool_calls:
    for tool_call in message.tool_calls:
        # 依次执行每个工具调用
        result = dispatch_tool(tool_call.function.name,
                               json.loads(tool_call.function.arguments))
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": str(result)
        })
```

---

## 六、Function Call vs MCP

| | Function Call | MCP |
|--|--------------|-----|
| **定义方式** | 在 API 请求里传 JSON schema | 通过标准协议注册到服务 |
| **作用范围** | 单次 API 调用 | 跨应用、跨模型共享工具 |
| **适用场景** | 业务逻辑中调用特定函数 | 工具标准化、多 Agent 共享 |

> Function Call 是机制，MCP 是标准化协议。实际开发先用 Function Call，规模大了再考虑 MCP。

---

## 参考资料

- [OpenAI Function Calling 文档](https://platform.openai.com/docs/guides/function-calling)
- [OpenAI Cookbook - Function Calling](https://cookbook.openai.com/examples/how_to_call_functions_with_chat_models)
