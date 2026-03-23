# Lore - 团队外部知识库

**Team External Brain for Claude Code**

> 让 Claude 像熟悉项目的资深工程师一样工作

---

## 核心概念

Lore 是一个团队外部知识库系统，通过以下循环持续提升团队效率：

```
加载标准 + 经验 → 执行 + 自检 → 记录偏差 → 周回顾 → 下次更少犯错
```

**核心约束：**
- 所有路径从 `~/.claude/CLAUDE.md` 读取，禁止硬编码
- 执行大任务前必须锁定 PRD
- 禁止跳过健康检查

---

## 核心流程

### 健康检查（每个任务前执行）

```
1. PROJECT.md 存在？
   不存在 → 告知用户，停止

2. logs/ 有 > 10 个未审查文件？
   是 → "N 个未审查日志，请先执行 'weekly review'"，停止

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

### 阶段划分

| 阶段 | 说明 |
|------|------|
| Phase 0 | PRD 锁定（大任务） |
| Phase 1 | 任务开始：加载 PROJECT.md、LORE.md |
| Phase 2 | 执行 + 自检 |
| Phase 3 | 任务完成 |
| Phase 4 | LORE 检索 |
| Phase 5 | 代码考古 |
| Phase 6 | 周回顾 |
| Phase 7 | 需求关闭 |
| Phase 8 | 项目关闭 |
| Phase 9 | 规则诊断 |

---

## 触发条件

| 触发词 | 说明 |
|--------|------|
| `PROJECT.md` 存在 | 执行任务前进行健康检查并加载上下文 |
| 大任务完成出现纠错/追问/方向调整 | 生成经验日志 |
| `weekly review` | 周回顾，审查未审核的日志 |
| `lore: {keyword}` | 检索 LORE 经验库 |
| `code archaeology {path}` | 代码考古，理解不确定的代码 |
| `req close {name}` | 需求关闭 |
| `project close` | 项目关闭 |
| `check this rule: {rule}` | 规则诊断 |

---

## 目录结构

```
lore/
├── README.md                    # 本文件
├── SKILL.md                     # Skill 定义文件
├── .claude/                     # Claude 配置
└── references/                  # 模板文件
    ├── LORE-template.md         # LORE 条目模板
    ├── PROJECT-template.md       # 项目模板
    ├── REQ-template.md          # 需求模板
    ├── LOG-template.md          # 日志模板
    ├── UNKNOWNS-template.md     # 未知项模板
    ├── team-LORE-template.md     # 团队 LORE 模板
    ├── global-CLAUDE-template.md # 全局 CLAUDE 模板
    └── pre-commit               # pre-commit 脚本
```

---

## 经验库结构

```
~/.claude/projects/<项目名>/memory/
├── INDEX.md                     # 放入 System Prompt
├── general/
│   └── general.md               # 跨项目通用能力（≥2项目才晋升）
└── projects/
    ├── _template/
    │   └── PROJECT.md           # 新项目复制此模板
    ├── project-a/
    │   └── PROJECT.md           # 进行中项目
    └── archive/                 # 已结案项目
```

---

## 错误归因框架

每条错题写入前必须过此判断链：

```
Claude 执行时信息是否充足？
│
├─ 不充足 → Claude 有没有追问？
│   ├─ 没追问就执行 → Claude 的错（应问未问）→ 注入 Skill
│   └─ 追问了，补充仍不清晰 → 你的错（描述问题）→ 记你的错题本
│
└─ 充足 → Claude 判断对不对？
    ├─ 判断错 → Claude 的错 → 注入 Skill
    └─ 判断对但你改了预期 → 目标未对齐 → 检查 PRD
```

---

## 硬规则

- ❌ 跳过健康检查
- ❌ 执行大任务时未锁定 PRD
- ❌ 写入 PROJECT.md、LORE.md、UNKNOWNS.md、team LORE
- ❌ 读取完整归档 REQ 文件（只允许 bash grep）
- ❌ LORE 匹配时使用语义推断（只允许文本匹配）
- ❌ 周回顾时替用户决定保留什么
- ❌ 从单个项目晋升到 team LORE
- ❌ 硬编码任何路径
- ❌ 小任务生成日志
- ❌ 日志 ID 与文件名不匹配

---

## 使用示例

```bash
# 启动任务
# → 自动进行健康检查
# → 自动加载 PROJECT.md 和相关 LORE 条目

# 检索经验
lore: 支付超时处理

# 代码考古
code archaeology src/main/java/com/example/PaymentService.java

# 周回顾
weekly review

# 项目结案
project close
```
