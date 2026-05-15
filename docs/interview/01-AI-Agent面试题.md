# AI Agent 方向面试题

> 针对 AI Agent / Python 后端方向，整理高频面试考点和参考答案。
> 答案风格：简洁、有核心结论、能展开说。

---

## 一、大模型基础

**Q1：Transformer 的核心是什么？Self-Attention 怎么计算？**

核心是 Self-Attention，让序列中每个位置都能关注所有其他位置。

计算步骤：
1. 输入向量分别乘三个矩阵得到 Q、K、V
2. 注意力分数 = softmax(QKᵀ / √d_k)
3. 输出 = 注意力分数 × V

除以 √d_k 是为了防止点积过大导致 softmax 梯度消失。Multi-Head 是多组 QKV 并行，捕捉不同维度的语义关系。

---

**Q2：Encoder-only、Decoder-only、Encoder-Decoder 分别适合什么任务？**

| 架构 | 代表 | 适合 |
|------|------|------|
| Encoder-only | BERT | 理解任务：分类、NER、相似度 |
| Decoder-only | GPT 系列 | 生成任务：对话、续写，主流 LLM |
| Encoder-Decoder | T5、BART | seq2seq：翻译、摘要 |

---

**Q3：LLM 幻觉是什么原因？怎么缓解？**

**原因**：LLM 本质是预测下一个 token，不是"查事实"，训练数据有噪声，模型对不确定的知识会"补全"。

**缓解方案（优先级排序）**：
1. RAG — 提供真实文档作为上下文，最常用
2. 降低 Temperature — 减少随机性
3. 要求引用来源 — 让模型说明依据
4. SFT / RLHF — 训练层面改善

---

**Q4：Temperature、Top-p、Top-k 的区别？**

- **Temperature**：控制概率分布的平滑程度。低 → 更确定（代码、JSON 用 0~0.3），高 → 更发散（创意写作用 0.7~1.0）
- **Top-p**（nucleus sampling）：只从累计概率 ≥ p 的候选词中采样，过滤掉长尾低概率词
- **Top-k**：只从概率最高的 k 个词中采样

> 实践：Top-p 和 Temperature 选一个调，不要都改。

---

**Q5：Scaling Law 的核心结论是什么？**

模型性能随参数量、数据量、计算量可预测地提升。
Chinchilla（2022）修正：数据量应与参数量匹配（约 20 token/参数），之前的模型普遍"训练不足"。
**关键结论：数据质量 > 数据量。**

---

## 二、RAG

**Q6：RAG 解决了什么问题？完整流程是什么？**

解决：知识截止、幻觉、私有知识无法使用三个问题。

**构建阶段**：文档 → 切片 → Embedding → 存入向量数据库  
**检索阶段**：用户问题 → Embedding → 相似度搜索 → Top-K 文档 → 拼入 Prompt → LLM 生成

---

**Q7：RAG 效果不好怎么优化？**

从三个方向排查：

**检索质量差** → 换切片策略（固定长度 → 语义切分）、换 Embedding 模型（换成 bge-m3）、加 Rerank（精排）、混合检索（向量 + BM25）

**上下文质量差** → 上下文压缩（只传相关句子）、HyDE（先生成假设答案再检索）

**生成质量差** → 优化 Prompt、降低 Temperature、要求引用来源

---

**Q8：向量数据库怎么选？**

- 个人项目/原型 → **Chroma**（轻量、本地、Python 友好）
- 生产环境 → **Qdrant**（高性能、Rust 实现、支持过滤）
- 已有 PostgreSQL → **pgvector**（不引入新组件）

---

## 三、Agent

**Q9：AI Agent 由哪几个核心组件构成？**

**LLM**（大脑）+ **Memory**（记忆）+ **Tools**（工具）+ **Planning**（规划）

- LLM：负责规划和推理
- Memory：短期（对话上下文）+ 长期（向量数据库）
- Tools：Function Call / MCP 调用外部系统
- Planning：ReAct（最主流）、Plan-and-Execute

---

**Q10：ReAct 是什么？和 CoT 的区别？**

**CoT**（Chain-of-Thought）：让模型一步步思考推理，但只在模型内部，不与外部交互。

**ReAct**：Reason + Act，思考 → 行动（调工具）→ 观察结果 → 再思考，是 Agent 的核心范式。

```
Thought: 我需要查询用户报销记录
Action: query_records(user_id=123)
Observation: 找到 3 条记录
Thought: 可以回答了
Answer: 您有 3 条报销记录
```

---

**Q11：Function Call 的工作原理？**

1. 开发者定义函数描述（JSON schema）传给模型
2. 模型判断是否需要调用工具
3. 需要 → 输出 `tool_call`（函数名 + 参数 JSON），**不直接执行**
4. 开发者代码执行函数，将结果作为 `tool` 消息返回
5. 模型根据结果生成最终回复

关键：**LLM 只输出调用意图，真正执行靠开发者代码。**

---

**Q12：MCP 是什么？解决什么问题？**

MCP（Model Context Protocol）是 Anthropic 提出的工具调用标准化协议。

**解决的问题**：以前每个 AI 应用要单独集成每个工具（N×M 问题），MCP 让工具只需实现一次，所有支持 MCP 的应用都能用（N+M）。

类比：MCP 之于 AI 工具，就像 USB 之于外设。

---

**Q13：Context Engineering 是什么？为什么重要？**

有意识地设计和管理送入 LLM 的上下文内容。

重要性：LLM 只能看到上下文窗口内的信息，上下文质量直接决定输出质量。

核心问题：上下文太长（截断/压缩）、信息不足（RAG/工具调用）、信息冗余（精简过滤）。

---

**Q14：Multi-Agent 相比单 Agent 的优势？**

- **突破上下文限制**：复杂任务拆分给多个 Agent，每个只处理子任务
- **并行执行**：多个子任务同时跑，节省时间
- **职责分离**：每个 Agent 专注一类任务，不互相干扰
- **专业化**：不同任务用不同模型（强模型做规划，弱模型做执行，节省成本）

---

## 四、微调

**Q15：LoRA 的原理是什么？为什么有效？**

冻结预训练权重，在 Transformer 层旁注入低秩矩阵 B·A（r ≪ d）。

`h = W₀x + BAx`，只训练 B 和 A，参数量减少 10000 倍。

有效的原因：权重更新具有"低内在秩"特性，不需要更新所有参数，低秩近似就能完成任务适配。推理时可直接合并 `W = W₀ + BA`，零推理延迟。

---

**Q16：SFT、RLHF、DPO 的区别？**

| 方法 | 做法 | 作用 |
|------|------|------|
| SFT（监督微调）| 用指令-回答对训练 | 让模型学会"如何回答" |
| RLHF | 人类对多个回答打分，训练奖励模型，再用 RL 优化 | 对齐人类偏好，复杂 |
| DPO | 直接用偏好对（好回答 vs 差回答）训练 | 比 RLHF 更简单稳定，效果相近 |

**典型流程**：Base Model → SFT → DPO → 最终 Chat Model

---

**Q17：什么时候选微调，什么时候选 RAG？**

| 场景 | 选择 |
|------|------|
| 需要私有/最新知识 | RAG |
| 知识频繁更新 | RAG |
| 改变模型风格/行为/角色 | 微调 |
| 需要掌握特定领域术语 | 微调 |
| 两者结合 | 微调学术语 + RAG 提供知识 |

---

## 五、Python

**Q18：Python 的 GIL 是什么？对多线程有什么影响？**

GIL（全局解释器锁）：CPython 中同一时刻只有一个线程执行 Python 字节码。

**影响**：
- CPU 密集型任务：多线程无法真正并行，用 **multiprocessing** 或 **C 扩展**
- IO 密集型任务：线程阻塞时会释放 GIL，多线程有效；但更推荐 **asyncio**

---

**Q19：`async/await` 和多线程的区别？**

| | 多线程 | asyncio |
|--|--------|---------|
| 并发方式 | 抢占式（OS 调度） | 协作式（手动 yield） |
| 切换开销 | 大（上下文切换） | 小（用户态切换） |
| 适合场景 | IO 密集 | IO 密集，尤其是大量并发请求 |
| 共享状态 | 需要锁 | 单线程，天然安全 |

Agent 开发中多用 asyncio，因为大量并发调用 LLM API 和工具。

---

**Q20：装饰器的原理？**

装饰器本质是高阶函数：接收函数作为参数，返回新函数。

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        # 前置逻辑
        result = func(*args, **kwargs)
        # 后置逻辑
        return result
    return wrapper
```

`@decorator` 等价于 `func = decorator(func)`。用 `@functools.wraps(func)` 保留原函数的元信息。

---

**Q21：Python 的内存管理机制？**

- **引用计数**：对象的引用数为 0 时立即释放
- **循环垃圾回收**：处理循环引用（引用计数无法处理 A→B→A 的情况）
- **内存池**：小对象（< 512B）复用内存块，减少 malloc 调用

与 C 的区别：Python 自动管理内存，不需要手动 malloc/free，但无法精确控制释放时机。

---

**Q22：列表和元组的区别？什么时候用元组？**

- **list**：可变，增删改，内存占用稍大
- **tuple**：不可变，哈希化（可作 dict key），速度略快

用元组的场景：函数多返回值、dict key、不需要修改的固定数据、`namedtuple` 替代简单类。

---

## 六、系统设计（Agent 方向）

**Q23：如何设计一个高可用的 RAG 系统？**

关键设计点：

```
用户请求
  ↓
查询改写（HyDE / 多查询扩展）
  ↓
混合检索（向量 + BM25）
  ↓
Rerank 精排
  ↓
上下文压缩（只传相关句）
  ↓
LLM 生成 + 引用标注
  ↓
返回结果
```

**可靠性**：检索失败降级到关键词搜索；LLM 超时有兜底回复；向量库定期重建索引。

---

**Q24：Agent 出现无限循环怎么处理？**

- 设置 `max_iterations` 硬上限（LangChain 默认 15）
- 每步工具调用结果写入状态，检测重复调用
- 工具调用失败超过 N 次，主动终止并返回错误信息
- 用超时机制（`asyncio.wait_for`）兜底

---

**Q25：如何控制 Agent 的 Token 成本？**

- 对话历史超阈值时做摘要压缩
- 检索结果 Rerank 后只传 Top-3，不传 Top-20
- 非关键子任务用小模型（gpt-4o-mini）
- 设置 max_tokens 上限
- 缓存相同问题的结果（Redis）

---

## 七、项目经验类

**Q26：介绍你的 expense-agent 项目（设计亮点、技术难点）**

参考要点：
- **架构**：FastAPI + PostgreSQL + Redis + MinIO，前端 React + Ant Design
- **AI 部分**：LLM 解析报销单、意图识别、多角色审批流
- **技术难点**：
  - 路由顺序问题（具体路径必须在通配符路径 `/{id}` 前面）
  - MinIO presigned URL 跨域（通过 `/minio` 代理重写解决）
  - Recharts 替换 Plotly（Vite CJS 兼容问题）
  - bcrypt 4.0.1 verify bug（改用 `bcrypt.checkpw()` 直接调用）

---

**Q27：为什么选择 AI Agent 方向？**

参考思路：
- C 语言系统开发背景 → 对底层原理理解扎实
- 在天融信做了 4 年网络安全，见过大量系统集成场景
- AI Agent 本质是"把 LLM 接入真实系统"，这和系统集成经验高度吻合
- 通过 expense-agent 项目验证了完整技术栈的落地能力
