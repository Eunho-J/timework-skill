# Forced Feedback Loop

**[English](README.md)** | **[한국어](README.ko.md)** | **[日本語](README.ja.md)** | **[中文](README.zh.md)**

一个强制AI代理在规定预算（时间或循环次数）内持续运行自我反馈循环、不断提升工作质量的技能。代理在预算耗尽前绝不会主动停止。

## 解决的问题

- AI粗略扫一遍就宣布"完成了"
- "下次我可以帮你做X"之类只说不做的空头承诺
- 工作记录只存在于临时上下文中，会话结束后消失
- 确认偏误——代理不经真正测试就认定自己的结论正确
- 只重组已有信息导致搜索空间逐渐缩小
- 随着想法累积，知识文件变得庞大且无法搜索

## 核心原则

1. **预算耗尽前绝不停止** — `min_required_minutes` 或 `min_required_loops` 期间只进行主动工作
2. **每轮循环生成新提案** — 收敛（重组已知）与发散（探索未知）双向并行
3. **所有内容写入文件** — 在 `work-log.md` 中以倒序报告堆叠，保证跨会话连续性
4. **以独立笔记+搜索管理知识** — Obsidian风格知识库中每个想法独立存储为文件，由专用搜索子代理按需检索

## 技能文件结构

```
forced-feedback-loop/
├── SKILL.md                        # 核心执行策略（激活时加载）
└── references/
    ├── DOMAIN-TABLES.md            # 按领域的详细表格（延迟加载）
    ├── PROHIBITIONS.md             # 27条禁止行为（延迟加载）
    └── KNOWLEDGE-BASE.md           # KB系统：笔记格式、标签、搜索协议
```

## 运行时生成的产物

代理在执行过程中于工作目录生成以下文件：

```
<工作目录>/
├── work-log.md                     # 完整工作历史（倒序报告栈）
└── kb/                             # Knowledge Base（Obsidian风格）
    ├── _index.md                   # 标签→文件自动索引
    ├── raw/                        # 未验证的想法、草稿
    │   ├── 20260303-143000-xxx.md
    │   └── ...
    ├── archive/                    # 被否决的项目（含原因）
    │   ├── 20260303-150000-yyy.md
    │   └── ...
    └── curated/                    # 已验证项目
        ├── 20260303-160000-zzz.md
        ├── UNCERTAINTY-REGISTER.md # 不确定性登记表
        └── GAP-REGISTER.md         # 数据缺口登记表
```

### 为什么使用独立笔记？

单体式3文件方案随着想法累积会变得臃肿：

- 将全部内容加载到上下文中浪费Token
- 无法只检索相关条目
- 会话越长搜索效率越差

KB系统将每个想法/决定/证据存储为独立的Markdown文件，通过YAML前置数据（标签、状态、置信度等）附加元数据，使搜索子代理能精确获取所需内容。

### 搜索子代理

代理从KB检索信息时，不会读取全部内容，而是委托给搜索子代理：

1. 通过 `_index.md` 进行基于标签的查找
2. 读取匹配的笔记文件（最多10个）
3. 返回前5个结果，格式为 {id, 标题, 状态, 置信度, 摘要}
4. 同时返回相关证据、矛盾点和缺口

如果KB中的笔记少于10个，代理直接读取文件而不使用子代理。

## 支持的领域

| task_type | 用途 |
|---|---|
| `research` | 调研 |
| `code` | 开发、工程 |
| `design` | UI/UX设计 |
| `document` | 文档撰写 |
| `analysis` | 数据分析 |
| `project` | 项目管理 |
| `other` | 其他 |

证据类型、来源层级、标签体系和测试方法会根据领域自动适配。详细映射见 `references/DOMAIN-TABLES.md`。

## 使用方法

### 安装

解压下载的zip文件，放置到代理的技能目录中。

| 代理 | 路径 |
|---|---|
| Perplexity | 通过User Settings上传 |
| OpenAI Codex | `~/.codex/skills/forced-feedback-loop/` |
| Claude Code | `~/.claude/skills/forced-feedback-loop/` |
| Augment | `~/.augment/skills/forced-feedback-loop/` |
| VS Code (Copilot) | `.agents/skills/forced-feedback-loop/`（项目根目录） |

### 使用示例

**调研：**
```
花30分钟调研"韩国电动汽车电池市场趋势"。
```

**代码开发：**
```
花20分钟改善这个API服务器的响应速度。当前p99延迟为500ms。
```

**设计：**
```
花15分钟改善这个仪表盘布局的可用性。
```

**文档：**
```
花25分钟加强这份技术提案的论证结构。
```

**次数基准（无时间限制）：**
```
运行10次反馈循环来改善这个函数的错误处理。
```

**两者兼备（时间+次数）：**
```
至少20分钟以上且至少5次循环来优化这个查询。
```

可以指定时间、循环次数或两者兼备。代理会自动激活技能并持续运行反馈循环直到终止条件满足。

### 终止模式

| `min_required_minutes` | `min_required_loops` | 行为 |
|---|---|---|
| 设定 | — | **时间基准**: 运行到时间耗尽 |
| — | 设定 | **次数基准**: 精确运行N次循环 |
| 设定 | 设定 | **两者兼备**: 运行到两个条件都满足 |

两者均未设定时，默认为5分钟。

### 收敛 × 发散 双重提案

每轮循环的提案步骤交替使用两种模式：

| 模式 | 方向 | 说明 |
|---|---|---|
| **收敛 (Convergent)** | 已知 → 新假设 | 将KB中2个以上的发现组合成新提案 |
| **发散 (Divergent)** | 未知 → 新假设 | 识别KB尚未回答的相邻问题，主动搜索新信息进行验证 |

每连续3轮循环中至少包含1次发散提案。纯收敛会导致搜索空间缩小。
发散提案必须通过与 `task_goal` 的相关性检查，因此不会偏离主题。

### 读取 work-log.md

报告从上到下按最新排列。每个报告首行声明该报告的行数，支持增量读取：

```
=== Report #3 | lines: 6 | elapsed: 14:30 | type: feedback ===   ← 第1行
DIAGNOSE: Weakest decision is D2 (confidence 0.4).
PROPOSE: Inversion — what if we use push instead of pull?
TEST: grep codebase for event-driven patterns → 3 hits.
UPDATE: D2 confidence raised to 0.65.
META: Progressing. Next loop targets D4.                         ← 第6行
---
=== Report #2 | lines: 4 | elapsed: 10:00 | type: feedback ===
...
```

**`lines:` 规则：** 从标题行（含）到最后一行内容（含）。`---` 分隔符和报告间的空行不计入。
代理在每次写入报告后验证此值，在脚本环境中运行lint检查。

跨报告引用使用相对坐标（`K-reports-below`, `line N below`），因此即使日志整理或压缩后引用也不会失效。

## 多会话续接

如果上一次会话的 `work-log.md` 和 `kb/` 目录还在，代理可以在下一次会话中继续。它会读取 `kb/_index.md`，通过搜索子代理查看最近的笔记，恢复之前的记录并继续反馈循环。

```
继续昨天的电池市场调研，再花20分钟。
```

## 兼容性

遵循 [Agent Skills 开放规范](https://agentskills.io/specification)，兼容所有支持该规范的代理。
