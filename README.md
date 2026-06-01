# Lite-Learning

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

> 命令行论文阅读助手 — 加载 PDF 后像请教助教一样提问，支持跨论文语义检索。

基于 ReAct 架构，LLM 可自主调用 12 种工具完成复杂研究任务。纯 Python 实现，终端原生体验。

## 目录

- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [使用指南](#使用指南)
- [命令参考](#命令参考)
- [ReAct 工具](#react-工具)
- [架构设计](#架构设计)
- [项目结构](#项目结构)
- [技术栈](#技术栈)

## 功能特性

**论文阅读**
- PDF 文本提取，支持中英文论文
- 多轮对话式问答，上下文自动关联
- 严格模式（仅限原文，杜绝幻觉）与拓展模式（可引用外部知识并标注来源）
- 一键生成结构化摘要（背景 / 问题 / 方法 / 结论 / 贡献）
- 自动生成阅读笔记（academic-paper-workflow 三部分格式）

**ReAct 智能代理**
- LLM 自主决策调用工具，多步骤完成任务
- 自然语言输入即可触发工具链
- 流式输出，实时显示 LLM 思考与决策过程
- Debug 模式可观察每轮 ReAct 推理细节

**本地知识库（轻量 RAG）**
- 论文自动分块索引，支持跨论文语义检索
- 当前基于字符 bigram 的文本匹配实现，零外部依赖
- 后续计划升级为向量嵌入模型（sentence-transformers / ChromaDB），
  届时将获得更强的语义理解能力
- 索引数据存储在本地 `knowledge_db/` 目录

**Shell 命令执行**
- ReAct 可自主调用终端命令
- 三级安全分级：safe（直接执行）/ confirm（需确认）/ forbidden（禁止）
- 覆盖 `find -exec`、输出重定向、命令注入等攻击面

**终端体验**
- Tab 自动补全（命令 + 文件名）
- 跨会话命令历史（存储在 `~/.lite-learning/.history`）
- 底部状态栏（当前论文 / 模式 / 对话轮数）

## 快速开始

### 环境要求

- Python 3.10+
- DeepSeek API Key

### 安装

```bash
git clone https://github.com/AsahiE0221/Lite-Learning.git
cd Lite-Learning
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 配置

创建 `~/.lite-learning/.env`：

```env
DEEPSEEK_API_KEY=sk-your-key-here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-v4-flash
```

### 准备论文

将 PDF 论文放入项目 `papers/` 目录。

### 启动

```bash
PYTHONPATH=src python3 src/cli.py
```

## 使用指南

### 典型工作流

```
📄 /papers                         # 扫描可用论文
📄 /load #1                        # 加载论文（按编号）
📄 这篇论文的核心论点是什么？         # 自然语言问答
📄 /summary                        # 生成结构化摘要
📄 /note                           # 保存阅读笔记
📄 /index                          # 索引到本地知识库
📄 /search 政教分离                  # 跨论文语义检索
📄 /unload                         # 卸载论文
```

### 加载论文的三种方式

```bash
/load 论文.pdf       # 按文件名
/load #3             # 按编号（先 /papers 查看）
/load @关键词         # 模糊搜索（LLM 自动匹配最相关论文）
```

### 问答模式

| 模式 | 命令 | 行为 |
|------|------|------|
| **严格模式**（默认） | `/strict` | 仅限论文原文，不编造不脑补，未涉及内容如实告知 |
| **拓展模式** | `/expand` | 可结合外部知识补充，明确区分「论文观点」与「外部延伸」 |

### 跨论文检索

```bash
📄 /load 论文A.pdf && /index       # 索引第一篇
📄 /load 论文B.pdf && /index       # 索引第二篇
📄 这两篇在方法论上有什么异同？
🤖 [ReAct 自动调 search_knowledge → 综合回答]
```

## 命令参考

### 论文操作

| 命令 | 说明 |
|------|------|
| `/papers` | 扫描 `papers/` 目录，列出所有 PDF |
| `/load #N\|文件名\|@关键词` | 加载论文 |
| `/unload` | 卸载当前论文 |

### 论文问答

| 命令 | 说明 |
|------|------|
| `/summary [要求]` | 生成结构化摘要 |
| `/note` | 生成阅读笔记，保存至 `notes/` |
| `/retry` | 重新生成上一轮回答 |

### 知识库

| 命令 | 说明 |
|------|------|
| `/index` | 将当前论文索引到本地知识库 |
| `/search <关键词>` | 语义搜索已索引论文 |
| `/kb` | 查看知识库状态（已索引论文数、分块数） |

### 对话管理

| 命令 | 说明 |
|------|------|
| `/compact` | 压缩对话历史（LLM 摘要替代旧消息） |
| `/clear` | 清除对话历史 |

### 模式与调试

| 命令 | 说明 |
|------|------|
| `/strict` | 切换严格模式 |
| `/expand` | 切换拓展模式 |
| `/status` | 查看会话状态 |
| `/debug` | 切换调试模式（流式输出 ReAct 决策过程） |

### 系统

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助 |
| `/quit` | 退出程序 |

## ReAct 工具

LLM 可自主调用的 12 个工具：

| 工具 | 分类 | 说明 |
|------|------|------|
| `scan_papers` | paper | 扫描论文目录 |
| `search_paper` | paper | 模糊搜索论文 |
| `load_paper` | paper | 加载论文 |
| `ask_paper` | paper | 基于论文内容问答 |
| `summary` | paper | 生成结构化摘要 |
| `write_note` | note | 生成阅读笔记 |
| `compact` | system | 压缩对话历史 |
| `status` | system | 查看会话状态 |
| `run_command` | system | 执行 Shell 命令（含安全分级） |
| `search_knowledge` | knowledge | 知识库语义搜索 |
| `index_paper` | knowledge | 索引论文到知识库 |
| `list_indexed` | knowledge | 列出已索引论文 |

## 架构设计

```
用户输入
  ├─ /斜杠指令 ──→ commands.py ──→ papers / conversation / context / knowledge
  └─ 自然语言 ──→ react.py (ReAct 决策循环)
                    ├─ LLM 决策 (chat_stream)
                    ├─ tools/handlers.py (工具分发)
                    └─ 反馈 → 再决策 → answer → 流式输出
```

**数据流**：所有模块通过 `session` (纯数据 dataclass) 共享状态，不依赖全局变量。

**Prompt 管理**：全部静态提示词集中在 `prompts.py`，System Prompt 完全静态以最大化 LLM 缓存命中率。模式切换标记放在 User Message 中。

**知识库**：当前基于字符 bigram + 余弦相似度的轻量实现，索引数据存储在 `knowledge_db/` 目录。分块大小 500 字符，相邻块重叠 100 字符以保证上下文连贯。后续计划替换为向量嵌入方案。

## 项目结构

```
src/
├── cli.py              # 入口：TUI 主循环
├── commands.py          # 17 个斜杠指令
├── config.py            # 路径与配置管理
├── session.py           # 会话状态 (dataclass)
├── prompts.py           # 全部静态提示词
├── papers.py            # 论文扫描、搜索、加载
├── conversation.py      # 问答、摘要、笔记 (含流式)
├── context.py           # 对话压缩、清除、重试
├── react.py             # ReAct 决策循环
├── knowledge.py         # 本地知识库 (索引 + bigram 语义搜索)
├── llm/
│   └── client.py        # DeepSeek API 封装 (chat / chat_stream)
├── tools/
│   ├── registry.py      # 工具注册表
│   ├── handlers.py      # 工具分发器
│   └── shell_security.py # Shell 安全分级
└── skills/
    └── pdf_reader.py    # PyMuPDF 文本提取
```

## 技术栈

| 组件 | 选型 | 说明 |
|------|------|------|
| LLM | DeepSeek v4 Flash | 1M token 上下文窗口 |
| PDF 解析 | PyMuPDF | 纯 Python，文字型 PDF 效果好 |
| 终端交互 | prompt_toolkit 3 | 补全、历史、状态栏 |
| 知识库 | 字符 bigram（当前） | 零依赖，后续升级为向量嵌入 |
| 语言 | Python 3.10+ | |

## 路线图

- [x] PDF 加载与文本提取
- [x] 多轮对话问答（严格 / 拓展模式）
- [x] ReAct 架构 + 12 工具注册系统
- [x] Shell 命令执行 + 安全分级
- [x] 本地知识库（bigram 语义搜索）
- [x] 流式输出 + Debug 模式
- [ ] 向量嵌入升级（sentence-transformers + ChromaDB）
- [ ] 文献综述自动生成
- [ ] 多 LLM 提供商支持

## License

MIT
# Lite-Learning
