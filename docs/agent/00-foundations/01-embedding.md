# Embedding 与向量化

> 目标：理解 Embedding 是什么、怎么用、如何选模型，这是 RAG 和语义搜索的基础。

---

## 一、什么是 Embedding？

**Embedding（嵌入）** 是把文本（或图片、音频）转换成一个固定维度的数字向量的过程。

**为什么需要它？**  
计算机不理解"苹果"和"水果"的关系，但可以计算向量之间的距离。

```
"苹果"  → [0.23, -0.51, 0.88, ...]   (1536维)
"水果"  → [0.21, -0.49, 0.85, ...]   ← 距离近，语义相近
"汽车"  → [-0.67, 0.32, -0.41, ...]  ← 距离远，语义无关
```

**核心性质**：语义相近的文本，向量距离近。

---

## 二、相似度计算

### 余弦相似度（最常用）

```python
import numpy as np

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# 结果范围 [-1, 1]，越接近 1 越相似
score = cosine_similarity(vec_a, vec_b)
```

### 点积（Dot Product）

向量已归一化时，点积等于余弦相似度，计算更快。向量数据库一般默认用这个。

### 欧氏距离

距离越小越相似，直觉上更自然，但高维空间效果不如余弦。

---

## 三、Embedding 模型选型

### 调 API（不需要本地 GPU）

| 模型 | 提供商 | 维度 | 特点 |
|------|--------|------|------|
| `text-embedding-3-small` | OpenAI | 1536 | 便宜，够用 |
| `text-embedding-3-large` | OpenAI | 3072 | 效果更好，贵 |
| `embedding-v3` | 智谱 AI | 2048 | 中文效果好 |

### 本地模型（免费，需要 GPU 或 CPU 推理）

| 模型 | 维度 | 特点 |
|------|------|------|
| `bge-m3` | 1024 | 中英文最强，支持多粒度检索，首选 |
| `bge-large-zh-v1.5` | 1024 | 中文效果好，轻量 |
| `text2vec-base-chinese` | 768 | 轻量，适合低资源环境 |

**选型建议**：
- 快速验证 → `text-embedding-3-small`（OpenAI API）
- 生产中文场景 → 本地 `bge-m3`（免费且效果好）

---

## 四、代码示例

### OpenAI API

```python
from openai import OpenAI

client = OpenAI()

def get_embedding(text: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

vec = get_embedding("报销审批流程是什么？")
print(len(vec))  # 1536
```

### 本地模型（sentence-transformers）

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("BAAI/bge-m3")

texts = ["报销审批流程", "差旅费用标准", "汽车发动机原理"]
vectors = model.encode(texts, normalize_embeddings=True)

# 计算相似度
from sentence_transformers import util
scores = util.cos_sim(vectors[0], vectors[1:])
print(scores)  # tensor([[0.89, 0.12]])
```

### 批量向量化（RAG 建库）

```python
def build_vector_store(documents: list[str], chroma_path: str):
    import chromadb
    from chromadb.utils import embedding_functions

    client = chromadb.PersistentClient(path=chroma_path)
    ef = embedding_functions.OpenAIEmbeddingFunction(
        model_name="text-embedding-3-small"
    )
    collection = client.get_or_create_collection("knowledge", embedding_function=ef)

    # 批量添加（自动向量化）
    collection.add(
        documents=documents,
        ids=[f"doc_{i}" for i in range(len(documents))]
    )
    return collection
```

---

## 五、Embedding 在 RAG 中的位置

```
构建阶段：文档 → 切片 → Embedding 模型 → 向量 → 存入向量库
检索阶段：问题 → Embedding 模型 → 查询向量 → 相似度搜索 → Top-K 文档
```

**关键要求**：构建和检索必须用**同一个 Embedding 模型**，否则向量空间不对齐，检索结果会乱。

---

## 六、常见问题

**Q：文本太长怎么办？**  
Embedding 模型有输入长度限制（一般 512~8192 token）。超长文本需要先切片，再分别向量化。

**Q：Embedding 向量能不能直接比较两篇文章的相似度？**  
可以，但效果取决于模型。长文档建议先摘要再向量化，或用 Late Interaction 模型（如 ColBERT）。

**Q：为什么 Rerank 比纯向量检索更准？**  
向量检索是"近似匹配"，Rerank 用 Cross-Encoder 对每对（问题, 文档）做精确比较，更慢但更准。实践中"向量粗召回 + Rerank 精排"是最优组合。

---

## 参考资料

- [OpenAI Embeddings 文档](https://platform.openai.com/docs/guides/embeddings)
- [BAAI/bge-m3](https://huggingface.co/BAAI/bge-m3)
- [sentence-transformers 文档](https://www.sbert.net/)
