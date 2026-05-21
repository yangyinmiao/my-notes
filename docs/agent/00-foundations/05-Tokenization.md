# Tokenization 与上下文窗口

> 目标：理解 Token 的工作原理，掌握上下文窗口管理的核心策略。

---

## 推荐学习资料

1. **Tiktokenizer 交互式工具**
   https://tiktokenizer.vercel.app/
   直接上手感受 Token 是怎么切的，先玩一遍再看文档。

2. **OpenAI Tokenizer 文档**
   https://platform.openai.com/tokenizer
   官方说明，重点看中文 Token 消耗比英文高的原因。

3. **《Efficient LLM Context Management》（DeepLearning.AI 短课）**
   https://www.deeplearning.ai/short-courses/
   搜 "long context" 相关课程，30分钟左右，讲窗口管理策略。

4. **LlamaIndex 文档 - Node Parser 章节**
   https://docs.llamaindex.ai/en/stable/module_guides/loading/node_parsers/
   文档分块（chunking）策略的工程实现，和 Token 管理直接相关。

---

## 一、什么是 Token？

（待整理）

---

## 二、Token 计算规律

- 英文：约 1 Token = 0.75 个单词
- 中文：约 1 Token = 0.5~1 个汉字（视分词而定）
- 代码：比自然语言消耗更多 Token

（待补充实际测量数据）

---

## 三、上下文窗口的限制

（待整理）

---

## 四、窗口管理策略

| 策略 | 思路 | 适用场景 |
|------|------|---------|
| 截断（Truncation） | 直接切掉超出部分 | 简单场景 |
| 滑动窗口 | 保留最近 N 轮 | 对话系统 |
| 摘要压缩 | 旧对话先摘要再保留 | 长对话 |
| RAG 召回 | 只放相关内容进窗口 | 知识密集型 |

（待整理）
