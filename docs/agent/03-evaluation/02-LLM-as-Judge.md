# LLM-as-Judge

> 目标：理解用大模型评估大模型的原理、局限性，以及工程中如何落地。

---

## 推荐学习资料

1. **《Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena》（2023）**
   https://arxiv.org/abs/2306.05685
   开创性论文，必读。重点看 positional bias 和 verbosity bias 两个问题。

2. **RAGAS 文档 - Evaluation 章节**
   https://docs.ragas.io/en/stable/concepts/metrics/
   RAG 场景下用 LLM 评估的具体指标，faithfulness / answer relevancy 等。

3. **DeepEval 文档**
   https://docs.confident-ai.com/
   工程化程度最高的评估框架，看 Metrics 部分的实现思路。

4. **Lilian Weng 博客 - Evaluating LLMs**
   https://lilianweng.github.io/posts/2023-06-23-agent/
   全面的评估方法论，包含 LLM-as-Judge 的利弊分析。

---

## 一、核心思路

（待整理）

---

## 二、主要偏差与缺陷

| 偏差类型 | 说明 |
|---------|------|
| Positional Bias | 更倾向于判定第一个答案更好 |
| Verbosity Bias | 更倾向于更长的答案 |
| Self-enhancement Bias | 偏向与自己风格相似的输出 |

（待补充）

---

## 三、常用评估指标

RAG 场景：
- Faithfulness（忠实度）
- Answer Relevancy（答案相关性）
- Context Precision（上下文精准度）

Agent 场景：
- Task Completion Rate
- Tool Call Accuracy

（待整理）

---

## 四、工程落地方案

（待整理）
