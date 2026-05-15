# LLM（Large Language Model）

## 什么是语言模型？
- 语言是一套复杂的符号系统，语言模型（Language Model）旨在**准确预测语言符号的概率**。

### 基于统计方法的语言模型
语言模型通过对语料库（Corpus）中的语料进行统计或学习来获得预测语言符号概率的能力。通常，基于统计的语言模型通过直接统计语言符号在语料库中出现的频率来预测语言符号的概率。其中，n-grams 是最具代表性的统计语言模型。n-grams 语言模型基于马尔可夫假设和离散变量的极大似然估计给出语言符号的概率。本节首先给出 n-grams 语言模型的计算方法，然后讨论 n-grams 语言模型如何在马尔可夫假设的基础上应用离散变量极大似然估计给出语言符号出现的概率。

### 基于RNN的语言模型
### 基于Transformer的语言模型（大模型）

### 语言模型的评测


## 大模型是什么？


## 大模型是如何训练的？

## 大模型的核心能力是什么？

## Transformer架构是什么？

3.1 简单介绍Transformer架构
3.2 编码器和解码器的区别
3.3 Transformer vs RNN
3.4 分词器是什么？
3.5 Embedding是什么？
3.6 注意力机制 (Attention)


## 大模型参数如何调整？

- Temperature
- Top-p
- Top-k

## 参考文献

- [《Attention is All You Need》](https://arxiv.org/abs/1706.03762)
- [《大模型基础》](https://github.com/ZJU-LLMs/Foundations-of-LLMs)