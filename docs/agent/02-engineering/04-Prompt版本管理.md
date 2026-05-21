# Prompt 版本管理

> 目标：理解工程化 Prompt 管理的必要性，掌握主流工具的使用方式。

---

## 推荐学习资料

1. **LangFuse Prompt Management 文档**
   https://langfuse.com/docs/prompts/get-started
   最实用的入门，直接看怎么创建版本、在代码里拉取。

2. **PromptFlow 文档（Microsoft）**
   https://microsoft.github.io/promptflow/
   企业级方案，重点看 Flow 的概念和 Prompt 变体测试。

3. **《Prompt Engineering for LLMs》第8章（O'Reilly）**
   讲 Prompt 从"写在代码里"到"独立管理"的演进，有助于建立工程化思维。

---

## 一、为什么需要 Prompt 版本管理？

Prompt 硬编码在代码里的问题：
- 修改一个词要重新发布代码
- 无法 A/B 测试不同版本
- 没有回滚机制
- 多人协作容易冲突

（待整理）

---

## 二、核心能力

- 版本控制：每次修改可追溯
- 环境隔离：dev / staging / prod 各一份
- 动态拉取：代码运行时从服务拉 Prompt，不重新部署
- 评估绑定：Prompt 版本和评估结果关联

（待整理）

---

## 三、LangFuse 实战

（待整理）
