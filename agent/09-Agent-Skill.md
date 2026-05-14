# Agent Skills

## 什么是 Agent Skills？
Agent Skills 是 AI Agent 具备的特定能力或功能模块，使其能够执行特定类型的任务。这些技能可以是预定义的函数、工具调用、数据处理能力等，旨在扩展 AI Agent 的应用范围和实用性。

## Agent Skills 的类型
1. **工具调用技能**：允许 AI Agent 调用外部工具或 API 来完成特定任务，如搜索引擎、数据库查询、计算等。
2. **数据处理技能**：使 AI Agent 能够处理和分析数据，如文本处理、图像识别、数据可视化等。
3. **交互技能**：增强 AI Agent 与用户的交互能力，如自然语言理解、情感分析、多轮对话管理等。


## 与MCP和Function Call的关系
Agent Skills 可以通过 MCP（Model Context Protocol）和 Function Call 技术来实现和调用。MCP 提供了一个标准化的协议，使得 AI Agent 可以高效地调用各种工具和技能，而 Function Call 则允许 AI Agent 在与用户交互时调用预定义的函数来执行特定任务。

## 什么情况下用用 Agent Skills？什么情况下用 MCP 或 Function Call？
- **Agent Skills** 适用于需要特定能力或功能模块的场景，如需要进行复杂数据处理、特定工具调用等任务。
- **MCP** 适用于需要规范化工具调用的场景，特别是当 AI Agent 需要调用多个不同类型的工具时，MCP 可以提供一个统一的接口和数据格式。
- **Function Call** 适用于需要在与用户交互时动态调用函数的场景，如智能助手需要根据用户输入调用不同的函数来执行任务。

> 后续补充 Agent 技能相关内容，包含原理、实现细节、应用场景等。