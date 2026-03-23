# Lore / 团队外部知识库

**Team External Brain for Claude Code**

> 让 Claude 像熟悉项目的资深工程师一样工作
> Makes Claude work like a senior engineer who knows this codebase.

---

## 概述 / Overview

Lore 是一个团队外部知识库系统，通过持续的经验积累让团队少犯错、不重复犯错。

Lore is a team external brain system that reduces mistakes through continuous learning.

**核心循环 / Core Loop：**
```
加载标准 + 经验 → 执行 + 自检 → 记录偏差 → 周回顾 → 下次更少犯错
Load standards + lessons → Execute with self-check → Record deviations → Weekly review → Fewer mistakes
```

---

## 核心约束 / Hard Rules

- ❌ 跳过健康检查 / Skip health check
- ❌ 执行大任务时未锁定 PRD / Execute large task without locked PRD
- ❌ 写入 PROJECT.md、LORE.md / Write to PROJECT.md or LORE.md
- ❌ 硬编码任何路径 / Hardcode any path
- ❌ 小任务生成日志 / Generate logs for small tasks

---

## 健康检查 / Health Check

每个任务前执行 / Before every task:

```
1. PROJECT.md 存在？/ Exists?
   No → 告知用户，停止 / Tell user, stop

2. logs/ 有 > 10 个未审查文件？/ > 10 unreviewed logs?
   是 → "请先执行 'weekly review'" / Run 'weekly review' first

3. 任务规模 / Task estimate?
   small → 询问领域，直接进入 Phase 1 / Ask domain, go to Phase 1
   large → 继续步骤 4 / Continue to step 4

4. 匹配的 REQ 是否已锁定 PRD？/ Matched REQ has locked PRD?
   已锁定 → Phase 1 / Locked → Phase 1
```

---

## 阶段 / Phases

| 阶段 | English | 说明 |
|------|---------|------|
| Phase 0 | PRD Lock | PRD 锁定（大任务）/ Lock PRD for large tasks |
| Phase 1 | Task Start | 加载 PROJECT.md、LORE.md |
| Phase 2 | Execution | 执行 + 自检 / Execute with self-check |
| Phase 3 | Complete | 任务完成 / Task complete |
| Phase 4 | Lore Search | LORE 检索 / Search LORE |
| Phase 5 | Code Archaeology | 代码考古 / Understand uncertain code |
| Phase 6 | Weekly Review | 周回顾 / Review logs |
| Phase 7 | REQ Close | 需求关闭 / Close requirement |
| Phase 8 | Project Close | 项目关闭 / Close project |
| Phase 9 | Rule Diagnosis | 规则诊断 / Diagnose rules |

---

## 触发条件 / Triggers

| 触发词 | English | 说明 |
|--------|---------|------|
| `PROJECT.md` | PROJECT.md exists | 执行任务前进行健康检查 |
| `weekly review` | Weekly review | 周回顾，审查未审核的日志 |
| `lore: {keyword}` | Lore search | 检索 LORE 经验库 |
| `code archaeology {path}` | Code archaeology | 代码考古，理解不确定的代码 |
| `req close {name}` | REQ close | 需求关闭 |
| `project close` | Project close | 项目关闭 |
| `check this rule: {rule}` | Check rule | 规则诊断 |

---

## 目录结构 / Directory Structure

```
lore/
├── README.md                    # 本文件 / This file
├── SKILL.md                     # Skill 定义 / Skill definition
├── .claude/                     # Claude 配置 / Claude config
└── references/                 # 模板文件 / Templates
    ├── LORE-template.md         # LORE 条目模板
    ├── PROJECT-template.md       # 项目模板
    ├── REQ-template.md          # 需求模板
    ├── LOG-template.md          # 日志模板
    ├── UNKNOWNS-template.md      # 未知项模板
    ├── team-LORE-template.md    # 团队 LORE 模板
    ├── global-CLAUDE-template.md # 全局 CLAUDE 模板
    └── pre-commit               # pre-commit 脚本
```

---

## 经验库结构 / Memory Structure

```
~/.claude/projects/<项目名>/memory/
├── INDEX.md                     # 放入 System Prompt
├── general/
│   └── general.md              # 跨项目通用能力（≥2项目才晋升）
└── projects/
    ├── _template/
    │   └── PROJECT.md          # 新项目复制此模板
    ├── project-a/
    │   └── PROJECT.md          # 进行中项目
    └── archive/                # 已结案项目
```

---

## 错误归因 / Error Attribution

每条错题写入前必须过此判断链：

```
Claude 执行时信息是否充足？/ Was info sufficient when Claude executed?
│
├─ 不充足 / Insufficient → Claude 有没有追问？/ Did Claude ask?
│   ├─ 没追问就执行 → Claude 的错（应问未问）/ Claude error (should have asked)
│   └─ 追问了，补充仍不清晰 → 你的错（描述问题）/ Your error (unclear description)
│
└─ 充足 / Sufficient → Claude 判断对不对？/ Was Claude's judgment correct?
    ├─ 判断错 → Claude 的错 / Claude error
    └─ 判断对但你改了预期 → 目标未对齐 / Goal misalignment
```

---

## 使用示例 / Usage Examples

```bash
# 启动任务（自动健康检查）
# Start task (auto health check)

# 检索经验
# Search experience
lore: 支付超时处理

# 代码考古
# Code archaeology
code archaeology src/main/java/PaymentService.java

# 周回顾
# Weekly review
weekly review

# 项目结案
# Project close
project close
```
