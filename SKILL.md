---
name: lore
description: 'Team external brain for Claude Code. Loads engineering standards and project knowledge before every task so Claude works like a senior engineer who knows this codebase. Trigger when: (1) project has PROJECT.md — run health check and load context before any task; (2) large task completes with correction, follow-up, or direction change — generate log; (3) user says "weekly review", "lore: {keyword}", "code archaeology {path}", "req close {name}", "project close", or "check this rule". Always load PROJECT.md and grep LORE.md silently at task start — even without explicit mention.'
---

# Lore

External brain for Claude Code teams.
Makes Claude work like a senior engineer who knows this codebase.

**Core loop:** load standards + lessons → execute with self-check → record deviations → weekly review → fewer mistakes next time.

All paths from `~/.claude/CLAUDE.md`. Never hardcode.

---

## Health check (before every task)

```
1. PROJECT.md exists?
   No → tell user, stop

2. logs/ has > 10 unreviewed files?
   count=$([ -d "logs" ] && grep -rl "reviewed: false" logs/ 2>/dev/null | wc -l || echo 0)
   count > 10 → "N unreviewed logs. Run 'weekly review' first.", stop

3. Task estimate?
   User declares: small or large
   Not declared → Claude defaults to large
   Small → ask user for domain (payment / auth / infra / ...),
           then go to Phase 1 directly
   Large → continue to step 4

4. Matched REQ has locked PRD?
   No REQ → identify or create
   Draft  → complete PRD, wait for lock
   Locked → Phase 1
```

---

## REQ identification

```
ls requirements/active/
  ↓
Match task against REQ filenames + domain fields
  ↓
One match  → "This looks like REQ-{name}. Correct?"
Many       → ask user to pick
None       → "Create new REQ or assign to existing?"
```

---

## Phase 0 — PRD lock (large tasks)

Read matched REQ → check `status`:
```
locked  → Phase 1
draft   → complete PRD, wait for "locked"
missing → create from template, draft PRD, wait for lock
```

**Drafting:**
1. Goal, acceptance criteria, out of scope
2. Risk flags — drift point, assumptions, applicable engineering standards
3. Clarifications → mark `[open]`, deprioritize P2/P3
4. Wait for confirmation → `status: locked`

Exception: "skip PRD" → append to REQ Decisions:
```
### PRD waiver — [date]
Reason: [stated reason]
```

---

## Phase 1 — Task start

```
Read PROJECT.md in full
  ↓
Get domain:
  Large task → from matched REQ frontmatter `domain:` field
  Small task → from user's answer at health check step 3
  ↓
grep LORE.md by domain:
  grep -B 5 -A 15 "\[domain:: {domain}\]" LORE.md
grep ~/team-lore/LORE.md by domain:
  grep -B 5 -A 15 "\[domain:: {domain}\]" ~/team-lore/LORE.md
  ↓
Three-layer match on grep results:
  Layer 1: keyword in task description → candidate
  Layer 2: scene-signal in task description → confirm
  Layer 3: exclude-signal in task description → reject
  All pass → "Note: [[entry-name]] — [insight]"
  ↓
Large task only → read matched REQ file
  ↓
TodoWrite — >60s must split, P1 first
```

Text match only. No semantic inference. Silence > noise.

---

## Phase 2 — Execution with self-check

**Step 1: Before writing any code**, infer operation types from task description and REQ:

```
Read task description + REQ acceptance criteria
  ↓
Infer which operation types this task involves:
  - Write operation? (create/update/delete data)
  - External call? (HTTP, queue, event, third-party API)
  - DB operation? (any SQL or ORM interaction)
List the inferred types explicitly.
```

**Step 2: Self-check per inferred type** (skip types not involved):

```
Write operation:
  → Concurrent access handled? Lock strategy?
  → Idempotent? What's the key?
  → Applicable engineering standards from PROJECT.md?

External call:
  → Fallback defined if call fails?
  → Timeout set?
  → Allowed per System boundaries?
    NO → blocker, stop, ask user

DB operation:
  → Known debt on this table? (check PROJECT.md Known debt)
  → Missing fields that affect this operation?
```

**Step 3: Check results → then write code**
- Known debt found → note it, handle accordingly, continue
- Boundary violated → stop, ask user
- Standard applies → implement it, continue

Small tasks: same three steps apply. No exceptions.

**Decision point (large tasks only):**
```
### [title] — [YYYY-MM-DD]
Context: [≤20 words]
Decision: [≤15 words]
Rationale: [≤20 words]
```
Append to REQ Decisions. Committed with code.

**Historical context:**
```bash
grep -A 10 "[section header]" requirements/done/REQ-{name}.md
```
Never read full archived files.

**Questions:** external fact gaps only. Ask everything at once.

---

## Phase 3 — Task complete

```
Large task + corrections / follow-ups / direction changes?
  Yes → output + generate log
  No  → output only

Small task → output only
```

**Log:** `logs/YYYYMMDD-HHMMSS-{member}-{task-slug}-{4-char-random}.md`

Attribution (pick one):
- `Claude's error (should have asked)`
- `Claude's error (wrong judgment)`
- `User's error (unclear description)`
- `PRD misalignment`

id must match filename. Mismatch → regenerate.

---

## Phase 4 — Lore search

**Trigger:** `lore: {keyword}`

```bash
grep -i -B 2 -A 10 "{keyword}" LORE.md ~/team-lore/LORE.md
```

```
Return matches as: [entry-name] | [domain] | [insight]
Max 5 results. More → return top 5, ask to narrow.
None → say so, suggest adjacent domain or broader keyword.
```

---

## Phase 5 — Code archaeology

**Trigger:** `code archaeology {path}`

```
Read module
  ↓
Certain: [what code demonstrably does]
Uncertain: [why — 2-3 possibilities per item, do not pick]
  ↓
Uncertain items > 5?
  → "Too many unknowns to process at once.
     Focus on the most critical question first: [top 1 uncertain item]
     Narrow scope or answer this one before continuing."
  ↓
User responds to uncertain items:
  User knows → output LORE entry draft
  User doesn't know → output UNKNOWNS.md draft entry
```

**LORE draft:**
```markdown
### [entry-name]
[keywords:: w1, w2]
[scene-signals:: s1, s2]
[exclude-signals:: e1]
[domain:: your-domain]
[type:: pitfall]
[date:: YYYY-MM-DD]
[promote:: false]
[insight:: one sentence]

- **Boundary:** [when not applicable]
- **Source:** archaeology — [file path]
```

**UNKNOWNS draft:**
```markdown
### [topic] — [YYYY-MM-DD]
Source: [file]
Question: [what we don't know]
Possibilities: [2-3 guesses]
Status: open
```

Claude outputs drafts. Human pastes and commits.
Commit: `[project] bootstrap lore: [entry-name]`

---

## Phase 6 — Weekly review

**Trigger:** `weekly review`

```
Check logs/ for unreviewed files:
  If logs/ doesn't exist or is empty → "No unreviewed logs. Done."
  If unreviewed count = 0 → "No unreviewed logs. Done."
  ↓
For each unreviewed log file:
  Read the file
  Output: [{id}] {deviation signal one line}
  Output: LORE entry draft
  ↓
User accepts or rejects each draft
  ↓
Accepted → user pastes into LORE.md, commits
Rejected → user deletes log file
Check LORE.md for promote = true entries → remind to PR to team-lore
Commit: "[project] weekly review — N entries"
```

Claude outputs a draft for every log. User decides what to keep — not Claude.

---

## Phase 7 — Requirement close

**Trigger:** `req close {name}`

```
Unreviewed logs? → "Run weekly review first"
promote = true entries? → "Remember to PR to team-lore"
  ↓
Output commands (user runs):
  git mv requirements/active/REQ-{name}.md requirements/done/
  Remove this line from PROJECT.md active requirements table:
    | {req-name} | [[requirements/active/REQ-{name}]] | active |
  git commit: "[project] req close: {name}"
```

---

## Phase 8 — Project close

**Trigger:** `project close`

```
Active requirements? → "Close all first: req close {name}"
Unreviewed logs? → "Run weekly review first"
  ↓
List promote = true entries
  → "Needs ≥1 other project before team promotion"
  ↓
Output summary + archive instructions
Commit: "[project] project close"
```

---

## Phase 9 — Rule diagnosis

**Trigger:** `check this rule: {rule}`

```
Read relevant LORE entries + recent logs
  ↓
Rule: [entry-name]
Evidence: [log refs]
Issue: rule itself | execution misread | missing context
Suggestion: [change] or "keep as is"
Confidence: high | medium | low
```

Claude outputs only. Never modifies files.

---

## Hard rules

- ❌ Skip health check
- ❌ Skip engineering standards self-check during execution
- ❌ Execute large task without locked PRD
- ❌ Write to PROJECT.md, LORE.md, UNKNOWNS.md, team LORE
- ❌ Read full archived REQ files — bash grep only
- ❌ Semantic inference in LORE matching — text only
- ❌ Decide what to keep in weekly review — user decides
- ❌ Assume REQ silently
- ❌ Promote to team LORE from one project only
- ❌ Hardcode any path
- ❌ Log id mismatching filename
- ❌ Generate log for small tasks
