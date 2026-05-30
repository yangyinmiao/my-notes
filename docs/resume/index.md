---
title: 杨银淼 · 个人简历
description: AI Agent 开发工程师 / Python 后端工程师
---

# 杨银淼

<table style="border-collapse: collapse;">
	<tr>
		<td style="width: 160px; vertical-align: top; padding-right: 16px;">
			<img src="../resume/证件照.jpg" alt="个人照片" style="width: 140px; height: auto; border-radius: 6px;" />
		</td>
		<td style="vertical-align: top;">
			<table style="border-collapse: collapse; width: 100%;">
				<tr>
					<th style="text-align: left; padding-right: 12px; border: none;">基本信息</th>
					<th style="text-align: left; border: none;">求职信息</th>
				</tr>
				<tr>
					<td style="border: none;">性别：女</td>
					<td style="border: none;">求职意向：Agent 应用开发</td>
				</tr>
				<tr>
					<td style="border: none;">电话：15527139220</td>
					<td style="border: none;">期望薪资：16k-20k</td>
				</tr>
				<tr>
					<td style="border: none;">邮箱：yang_yinmiao@outlook.com</td>
					<td style="border: none;">到岗时间：一周内</td>
				</tr>
				<tr>
					<td style="border: none;">所在地：武汉</td>
					<td style="border: none;">期望城市：武汉 / 深圳 / 上海</td>
				</tr>
			</table>
		</td>
	</tr>
</table>

---

## 自我评价

- **5 年+开发经验**，兼具 Linux C 底层开发与 Python AI 应用双重能力，2025 年完成从网络协议开发到 AI Agent 的技术转型
- **聚焦 AI Agent**：深入理解 LLM + Tools + Memory + Workflow 架构，具备 RAG 系统 0→1 设计与落地经验
- **工程习惯扎实**：重视代码质量与文档沉淀，交付准时，具备跨团队协作与项目推进能力

---

## 核心技能

| 方向 | 技能详情 |
|------|---------|
| **LLM 应用** | Prompt Engineering（Few-shot / CoT / ReAct）、RAG 全流程（文档解析 → Chunking → Embedding → 混合检索 → Rerank）、Agent 架构设计（Tools / Memory / Workflow） |
| **框架与工具** | LangChain · LangGraph · Chroma · N8N· Coze · Dify · OpenAI / 通义千问 API |
| **AI 工程化** | 模型效果评估（准确率 / 召回率 / 端到端测试）、LangFuse 可观测性、向量数据库选型 |
| **编程语言** | Python（主力）· C/C++（4 年+）· Shell · Perl |
| **云与运维** | Docker（熟练）· Kubernetes（了解）· Linux 生产环境运维 |
| **网络与安全** | TCP/IP · HTTP(S) · PKI · 加密/摘要算法 · 网络安全 |
| **工程方法** | UML 建模 · Axure 原型设计 · 数据结构与算法 |

---

## 作品集

- **Tech Reader** — Firefox 浏览器插件，帮助开发者摆脱翻译依赖、真正读懂英文技术文档，核心理念「越用越不需要它」· [GitHub](https://github.com/yangyinmiao/tech-reader)
- **AI 技术雷达** — 自动聚合 GitHub Trending 与 arXiv 论文，LLM 驱动中文分析与标签分类，Next.js + FastAPI 全栈 · [GitHub](https://github.com/yangyinmiao/ai-engineering-radar)
- **AI 结构化决策助手** — 五步决策流程，心理学框架隐式编入 LLM Prompt，Next.js + FastAPI · [GitHub](https://github.com/yangyinmiao/decision-assistant)
- **技术文档** — RAG 系统搭建指南、Prompt Engineering 最佳实践等 · [在线访问](https://yangyinmiao.github.io/my-notes/)

---

## 工作经历

### 深圳智联云科（武汉） <span class="date-tag">2025.10 - 至今</span>

**Python 开发工程师 | 互联网**

- 从零搭建公司内部 RAG 知识库系统，覆盖文档解析、混合检索、Rerank 全流程，支撑 1000+ 份内部文档的智能检索与问答
- 探索并落地 AI Agent 原型，基于 LLM + RAG + Tools 架构实现多步推理与工具调用，覆盖 5+ 业务场景，相较原人工流程减少操作 60%+
- 基于 n8n + Coze 搭建多平台内容自动分发系统，集成 AI 配图生成，相较人工发布单次耗时从 30-40 分钟压缩至 5 分钟内

---

### 天融信（武汉） <span class="date-tag">2021.06 - 2025.09</span>

**Linux C 开发工程师 | 网络安全**

- 主导数据通道核心框架设计与开发，通过 epoll 事件驱动 + 连接池复用 + 零拷贝传输，实现并发 30w+、新建连接 1.7w/s
- 独立负责 6+ Nginx 安全过滤模块（防 SQL 注入、XSS 防护、URL 过滤等），有效拦截 95%+ 恶意请求，线上稳定运行 3 年+
- 负责 PKI 证书管理模块设计与开发，提前交付并被多部门复用，输出《PKI 二次开发手册》
- 参与数据交换基础平台与安全数据交换产品升级，覆盖 HTTP/HTTPS/FTP/邮件协议代理策略

---

## 项目经历

### 模拟面试 Agent 平台

> 个人项目 · 2026.01 · 独立开发

**项目描述：** 从零构建多租户 AI 模拟面试平台，基于 LangGraph 多 Agent 架构驱动完整面试流程，支持简历/JD 解析、题库向量检索、4 种面试模式与自动评估报告生成。

**核心工作：**

- **LangGraph 多 Agent 编排**：6 个独立 Agent（简历分析 / JD 解析 / 题库检索 / 面试官 / 评估 / 监督路由），通过 StateGraph 有向图编排面试全流程，支持追问链、条件路由与状态持久化
- **异步文档处理管线**：上传即返回 + Celery 异步 Embedding，PDF/Word/Markdown 多格式解析，嵌入 Qdrant 向量数据库
- **多模式面试引擎**：实现基础 / 深度追问 / 追问链 / 压力面 4 种模式，LLM 根据 JD 匹配度与回答质量动态调整问题难度
- **LLM 可观测性**：集成 LangFuse 自部署方案，通过 LangChain CallbackHandler 实现 LLM 调用全链路追踪，按面试 session 分组，支持 token 消耗与成本分析
- **全栈工程实践**：FastAPI + PostgreSQL + SQLAlchemy 2.0 async 后端，Next.js 14 前端，Docker Compose 一键部署，JWT 多租户认证隔离
- **技术栈**：FastAPI · LangGraph · LangChain · PostgreSQL · Qdrant · MinIO · Celery · Next.js · Docker

---

### RAG 知识库系统

> 深圳智联云科 · 2025.10 - 2026.02 · 项目负责人

**项目描述：** 为公司内部从零搭建基于大语言模型的 RAG 知识库系统，实现多格式文档解析、混合检索与智能问答。在此基础上探索 AI Agent 原型，推动业务流程自动化。

**核心工作：**

- 完成文档解析模块：支持 PDF/Word/Excel 等格式，智能分块与 Embedding 向量化，入库 1000+ 份文档
- 优化检索策略：语义检索 + BM25 关键词检索 + Rerank 重排序，相较原关键词检索问答准确率提升 40%+，召回率 85%+
- 基于 LangChain 落地 AI Agent 原型，实现多步推理、工具调用、记忆管理
- **技术栈**：Python · OpenAI · Chroma · LangChain · FastAPI · Docker

---

### 数据通道（数据交换基础平台核心组件）

> 天融信 · 2023.06 - 2025.09 · 项目负责人

**项目描述：** 作为数据交换基础平台核心组件，以标准化方式为上层代理及同步业务提供数据摆渡能力，支持数据库同步、文件同步、协议代理、音视频代理等多种上层应用。

**核心工作：**

- 担任项目负责人，负责技术架构设计、技术选型及核心框架开发
- 完成系统性能调优：epoll 事件驱动 + 连接池复用 + 零拷贝传输，实现并发 30w+、新建连接 1.7w/s
- 支持双向、双单向、单向多种数据摆渡场景，提供自动化集成测试脚本，覆盖全场景回归测试

---

## 教育经历

### 湖北工业大学 <span class="date-tag">2017.09 - 2021.06</span>

**信息安全 | 本科**

- 全国大学生信息安全作品赛三等奖（2019）
- 校级奖学金（2018-2020）| NISP 一级 | CET-4 | CET-6

---

<div class="resume-footer">
<small>本简历最后更新于 2026 年 5 月</small>
</div>