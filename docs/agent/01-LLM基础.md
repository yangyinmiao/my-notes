# LLM 基础知识地图

> 目标：搞清楚大模型是什么、怎么工作、怎么控制输出。不求全，只记核心结论。

---

## 一、语言模型的演进

```
n-gram（统计）→ RNN/LSTM（序列）→ Transformer（注意力）→ LLM（大规模预训练）
```

| 阶段 | 代表 | 核心问题 |
|------|------|---------|
| 统计语言模型 | n-gram | 只看前 n 个词，长距离依赖处理不了 |
| 循环神经网络 | LSTM、GRU | 可以处理序列，但梯度消失、无法并行 |
| Transformer | BERT、GPT | 全局注意力、可并行训练，成为主流 |
| 大语言模型 | GPT-4、Claude、Llama | 规模足够大后涌现出推理、In-context Learning 等能力 |

---

## 二、Transformer 架构

### 2.1 整体结构

```
输入文本
  ↓ Tokenizer（分词）
Token IDs
  ↓ Embedding（词向量）
向量序列
  ↓ × N 层 Transformer Block
    ├── Multi-Head Self-Attention
    ├── Add & LayerNorm
    ├── Feed-Forward Network (FFN)
    └── Add & LayerNorm
  ↓
输出向量 → 预测下一个 Token 的概率分布
```

**Encoder-only**（BERT）：适合理解任务，如分类、NER  
**Decoder-only**（GPT 系列）：适合生成任务，主流 LLM 都用这个  
**Encoder-Decoder**（T5、BART）：适合翻译、摘要等 seq2seq 任务

---

### 2.2 Self-Attention（自注意力）

核心：每个词都能"看到"序列中所有其他词，计算相关性。

```
Q = X · Wq   （Query：我要找什么）
K = X · Wk   （Key：我有什么）
V = X · Wv   （Value：我的内容是什么）

Attention(Q, K, V) = softmax(QKᵀ / √d_k) · V
```

- 除以 `√d_k` 是为了防止点积过大导致 softmax 梯度消失
- **Multi-Head**：多组 QKV 并行，捕捉不同维度的语义关系，最后拼接

---

### 2.3 位置编码

Transformer 本身不感知顺序，需要显式注入位置信息。

| 方式 | 代表模型 | 特点 |
|------|---------|------|
| 绝对位置编码（正弦） | 原始 Transformer | 固定，不可学习 |
| 可学习位置编码 | BERT、GPT-2 | 简单，但外推能力差 |
| RoPE（旋转位置编码） | LLaMA、Qwen、ChatGLM | 支持长上下文外推，目前主流 |
| ALiBi | MPT | 推理时可超出训练长度 |

---

### 2.4 分词器（Tokenizer）

文本 → Token IDs 的映射。Token 不等于一个字/词。

- **BPE**（Byte Pair Encoding）：GPT 系列用，按频率合并字节对
- **WordPiece**：BERT 用
- 中文"你好"可能被拆成 1 个 Token，也可能是 2 个，取决于词表
- Token 数量影响计费和上下文长度

---

## 三、预训练

### 3.1 训练目标

| 目标 | 代表 | 做法 |
|------|------|------|
| CLM（因果语言模型） | GPT 系列 | 预测下一个词，单向注意力 |
| MLM（掩码语言模型） | BERT | 随机遮住 15% 的词，预测被遮住的词 |

> 现在主流 LLM 基本都是 CLM（Decoder-only）。

---

### 3.2 Scaling Law（规模定律）

核心结论（Kaplan et al., 2020）：

> **模型性能随参数量、数据量、计算量的增加而可预测地提升**

- 三者都很重要，但算力预算固定时，**同时扩大模型和数据**比只扩一个更优
- Chinchilla（2022）修正：之前的模型普遍"训练不足"，**数据量应与参数量匹配**（约 20 token/参数）
- 实践结论：**数据质量 > 数据量**，低质量数据多了会损害性能

---

### 3.3 涌现能力（Emergent Abilities）

规模到达某个阈值后，模型突然具备之前没有的能力：
- In-context Learning（上下文学习）
- Chain-of-Thought 推理
- 指令跟随

这些能力**无法通过小模型实验预测**，是 LLM 最重要的特性之一。

---

## 四、推理参数控制

调用 LLM API 时最常用的几个参数：

| 参数 | 作用 | 典型值 |
|------|------|-------|
| **Temperature** | 控制随机性。越低越确定，越高越发散 | 0（确定性）~ 1（均衡）~ 2（随机）|
| **Top-p**（nucleus sampling） | 只从累计概率达到 p 的候选词中采样 | 0.9 常用 |
| **Top-k** | 只从概率最高的 k 个词中采样 | 40~100 |
| **Max tokens** | 最大输出长度 | 按需设置 |
| **Stop sequences** | 遇到指定字符串就停止生成 | `\n`、`###` 等 |

> 实践：要稳定输出（代码、JSON）用低 temperature（0~0.3）；要创意内容用高 temperature（0.7~1.0）。Top-p 和 Temperature 通常选一个用，不要都调。

---

## 五、关键概念速查

| 概念 | 一句话 |
|------|-------|
| **上下文窗口（Context Window）** | 模型一次能"看到"的最大 Token 数，超出会截断 |
| **KV Cache** | 推理时缓存已计算的 K、V，避免重复计算，加速生成 |
| **幻觉（Hallucination）** | 模型生成看似合理但实际错误的内容，根源是自回归生成的本质 |
| **RLHF** | 用人类反馈强化学习，让模型输出更符合人类偏好（ChatGPT 的关键） |
| **SFT** | 监督微调，用高质量对话数据让预训练模型学会"如何回答" |
| **量化** | 用 INT8/INT4（GGUF Q4_K_M）等低精度存储模型权重，减少显存占用；FP16→INT8 省 50%，Q4 省约 87%，效果损失极小 |

---

## 参考资料

- [《Attention is All You Need》](https://arxiv.org/abs/1706.03762) — Transformer 原论文
- [《Scaling Laws for Neural Language Models》](https://arxiv.org/abs/2001.08361) — Scaling Law
- [《Training Compute-Optimal LLMs》](https://arxiv.org/abs/2203.15556) — Chinchilla
- [《大模型基础》ZJU](https://github.com/ZJU-LLMs/Foundations-of-LLMs) — 中文教材
