# Agent 安全与 Guardrails

> 目标：理解 Agent 在生产环境中面临的安全风险，掌握输入/输出防护的工程方案。

---

## 推荐学习资料

1. **OWASP Top 10 for LLM Applications**
   https://owasp.org/www-project-top-10-for-large-language-model-applications/
   LLM 安全领域的权威清单，重点看 Prompt Injection 和 Insecure Output Handling。

2. **Guardrails AI 文档**
   https://www.guardrailsai.com/docs
   最成熟的开源 Guardrails 框架，看核心概念和 Validator 设计。

3. **NeMo Guardrails 文档（NVIDIA）**
   https://github.com/NVIDIA/NeMo-Guardrails
   企业级方案，重点看 Colang 语言定义对话边界的思路。

4. **《Prompt Injection Attacks and Defenses》**
   https://arxiv.org/abs/2306.05499
   系统梳理了 Prompt 注入的攻击面，帮助理解为什么 Guardrails 难做。

---

## 一、主要风险类型

| 风险 | 说明 |
|------|------|
| Prompt Injection | 用户输入覆盖系统 Prompt 的指令 |
| Jailbreak | 绕过模型的安全限制 |
| 幻觉输出 | 模型编造不存在的信息 |
| 敏感信息泄露 | 输出训练数据或系统 Prompt |
| 有害内容 | 输出违规、有害的文本 |

（待补充）

---

## 二、防护层设计

```
用户输入
  ↓
[输入过滤层] ← 检测注入、敏感词、格式校验
  ↓
LLM 调用
  ↓
[输出校验层] ← 幻觉检测、格式验证、内容过滤
  ↓
返回用户
```

（待整理）

---

## 三、工程实现方案

（待整理）
