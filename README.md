# Lore / 团队外部知识库

> **[English](#english)** | **[中文](#中文)**

---

## English

### Overview

**Lore** — Team External Brain for Claude Code.

Makes Claude work like a senior engineer who knows this codebase.

**Core Loop:** `Load standards + lessons → Execute with self-check → Record deviations → Weekly review → Fewer mistakes next time`

### Hard Rules

- ❌ Skip health check
- ❌ Execute large task without locked PRD
- ❌ Write to PROJECT.md, LORE.md, UNKNOWNS.md, or team LORE
- ❌ Hardcode any path
- ❌ Generate logs for small tasks

### Health Check (Before Every Task)

```
1. PROJECT.md exists?
   No → Tell user, stop

2. logs/ has > 10 unreviewed files?
   Yes → "Run 'weekly review' first.", stop

3. Task estimate?
   User declares: small or large
   Not declared → Claude defaults to large
   Small → Ask domain, go to Phase 1
   Large → Continue to step 4

4. Matched REQ has locked PRD?
   No REQ → Identify or create
   Draft → Complete PRD, wait for lock
   Locked → Phase 1
```

### Phases

| Phase | Name | Description |
|-------|------|-------------|
| 0 | PRD Lock | Lock PRD for large tasks |
| 1 | Task Start | Load PROJECT.md, grep LORE.md |
| 2 | Execution | Execute with self-check |
| 3 | Complete | Task done, generate log if needed |
| 4 | Lore Search | `lore: {keyword}` |
| 5 | Code Archaeology | `code archaeology {path}` |
| 6 | Weekly Review | `weekly review` |
| 7 | REQ Close | `req close {name}` |
| 8 | Project Close | `project close` |
| 9 | Rule Diagnosis | `check this rule: {rule}` |

### Triggers

| Trigger | Description |
|---------|-------------|
| `PROJECT.md` exists | Run health check before any task |
| `weekly review` | Review unreviewed logs |
| `lore: {keyword}` | Search LORE entries |
| `code archaeology {path}` | Understand uncertain code |
| `req close {name}` | Close a requirement |
| `project close` | Close a project |
| `check this rule: {rule}` | Diagnose a rule |

### Directory Structure

```
lore/
├── README.md
├── SKILL.md
├── .claude/
└── references/
    ├── LORE-template.md
    ├── PROJECT-template.md
    ├── REQ-template.md
    ├── LOG-template.md
    ├── UNKNOWNS-template.md
    ├── team-LORE-template.md
    ├── global-CLAUDE-template.md
    └── pre-commit
```

### Memory Structure

```
~/.claude/projects/<project>/memory/
├── INDEX.md
├── general/
│   └── general.md              # Cross-project (≥2 projects to promote)
└── projects/
    ├── _template/
    │   └── PROJECT.md
    ├── project-a/
    │   └── PROJECT.md
    └── archive/
```

### Error Attribution

```
Was info sufficient when Claude executed?
│
├─ Insufficient → Did Claude ask?
│   ├─ Didn't ask → Claude error (should have asked)
│   └─ Asked but still unclear → Your error (unclear description)
│
└─ Sufficient → Was Claude's judgment correct?
    ├─ Wrong → Claude error
    └─ Right but you changed expectation → Goal misalignment
```

---

## 中文

### 概述

**Lore** — 团队外部知识库。

让 Claude 像熟悉项目的资深工程师一样工作。

**核心循环：** `加载标准 + 经验 → 执行 + 自检 → 记录偏差 → 周回顾 → 下次更少犯错`

### 硬规则

- ❌ 跳过健康检查
- ❌ 执行大任务时未锁定 PRD
- ❌ 写入 PROJECT.md、LORE.md、UNKNOWNS.md 或 team LORE
- ❌ 硬编码任何路径
- ❌ 小任务生成日志

### 健康检查（每个任务前）

```
1. PROJECT.md 存在？
   不存在 → 告知用户，停止

2. logs/ 有 > 10 个未审查文件？
   是 → "请先执行 'weekly review'"，停止

3. 任务规模？
   用户声明：small 或 large
   未声明 → 默认 large
   small → 询问领域，直接进入 Phase 1
   large → 继续步骤 4

4. 匹配的 REQ 是否已锁定 PRD？
   无 REQ → 识别或创建
   草稿 → 完成 PRD，等待锁定
   已锁定 → Phase 1
```

### 阶段

| 阶段 | 名称 | 说明 |
|------|------|------|
| 0 | PRD 锁定 | 大任务必须先锁定 PRD |
| 1 | 任务开始 | 加载 PROJECT.md、LORE.md |
| 2 | 执行 | 执行 + 自检 |
| 3 | 完成 | 任务完成，必要时生成日志 |
| 4 | LORE 检索 | `lore: {keyword}` |
| 5 | 代码考古 | `code archaeology {path}` |
| 6 | 周回顾 | `weekly review` |
| 7 | 需求关闭 | `req close {name}` |
| 8 | 项目关闭 | `project close` |
| 9 | 规则诊断 | `check this rule: {rule}` |

### 触发条件

| 触发词 | 说明 |
|--------|------|
| `PROJECT.md` 存在 | 执行任务前进行健康检查 |
| `weekly review` | 周回顾，审查未审核的日志 |
| `lore: {keyword}` | 检索 LORE 经验库 |
| `code archaeology {path}` | 代码考古，理解不确定的代码 |
| `req close {name}` | 需求关闭 |
| `project close` | 项目关闭 |
| `check this rule: {rule}` | 规则诊断 |

### 目录结构

```
lore/
├── README.md
├── SKILL.md
├── .claude/
└── references/
    ├── LORE-template.md
    ├── PROJECT-template.md
    ├── REQ-template.md
    ├── LOG-template.md
    ├── UNKNOWNS-template.md
    ├── team-LORE-template.md
    ├── global-CLAUDE-template.md
    └── pre-commit
```

### 经验库结构

```
~/.claude/projects/<项目名>/memory/
├── INDEX.md
├── general/
│   └── general.md              # 跨项目通用能力（≥2项目才晋升）
└── projects/
    ├── _template/
    │   └── PROJECT.md
    ├── project-a/
    │   └── PROJECT.md
    └── archive/
```

### 错误归因

```
Claude 执行时信息是否充足？
│
├─ 不充足 → Claude 有没有追问？
│   ├─ 没追问就执行 → Claude 的错（应问未问）
│   └─ 追问了，补充仍不清晰 → 你的错（描述问题）
│
└─ 充足 → Claude 判断对不对？
    ├─ 判断错 → Claude 的错
    └─ 判断对但你改了预期 → 目标未对齐
```
