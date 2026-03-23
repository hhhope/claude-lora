# ~/.claude/CLAUDE.md
# Global config. Each member maintains their own. Never commit.

---

## Identity

member: [your-name]

---

## Paths

team-lore: ~/team-lore/LORE.md

---

## Project name

From `project:` field in PROJECT.md frontmatter.
Fallback: current repo directory name.

---

## Health check (run before every task, stop at first failure)

1. PROJECT.md exists?
   No → "No PROJECT.md. Initialize project first."
   Stop.

2. Local logs/ has > 10 unreviewed files?
   count=$([ -d "logs" ] && grep -rl "reviewed: false" logs/ 2>/dev/null | wc -l || echo 0)
   count > 10 → "N unreviewed logs. Run 'weekly review' first."
   Stop.

3. Task estimate?
   User declared small → ask domain, go to task start
   User declared large → step 4
   Not declared → default large, step 4

4. Matching REQ has locked PRD?
   No REQ → identify or create (see below)
   Draft → complete PRD, wait for lock
   Locked → continue

---

## REQ identification

```
ls requirements/active/
  ↓
Match task description against REQ filenames + domain fields
  ↓
Single match → confirm with user: "This looks like REQ-{name}?"
Multiple     → ask user to pick
None         → "Create new REQ or assign to existing?"
```

Never assume silently.

---

## Task start (after health check)

1. Read PROJECT.md in full
2. Get domain:
   Large task → from matched REQ frontmatter `domain:` field
   Small task → from user's answer at health check step 3
3. grep LORE.md by domain:
   `grep -B 5 -A 15 "\[domain:: {domain}\]" LORE.md`
4. grep ~/team-lore/LORE.md by domain:
   `grep -B 5 -A 15 "\[domain:: {domain}\]" ~/team-lore/LORE.md`
5. Apply three-layer match to grep results:
   - Layer 1: keyword in task description → candidate
   - Layer 2: scene-signal in task description → confirm
   - Layer 3: exclude-signal in task description → reject
   All three pass → surface: "Note: [[entry-name]] — [insight]"
6. Large task only → read matched REQ file
7. TodoWrite — >60s must split, P1 first

Text match only. No semantic inference. Silence > noise.

---

## During execution

**Step 1: Infer operation types** from task description + REQ before writing code:
  - Write operation? (create/update/delete)
  - External call? (HTTP, queue, event, third-party)
  - DB operation? (any SQL or ORM)
List inferred types explicitly.

**Step 2: Self-check per type** (skip types not involved):

Write operation?
  → Concurrent access? Lock strategy? Idempotent?

External call?
  → Fallback? Timeout? Allowed per System boundaries?
  → NO → blocker, stop, ask user

DB operation?
  → Known debt on this table? Missing fields?

**Step 3: Results → write code**
  Known debt → note and handle, continue
  Boundary violated → stop, ask user
  Standard applies → implement, continue

Small tasks: same three steps. No exceptions.

**Decision point (large tasks):**
Append to REQ Decisions:
  ### [title] — [YYYY-MM-DD]
  Context: [≤20 words]
  Decision: [≤15 words]
  Rationale: [≤20 words]

**Historical context:**
bash grep only: `grep -A 10 "[section]" requirements/done/REQ-{name}.md`

**Questions:** external fact gaps only. Once.

---

## Task complete

Large task with corrections / follow-ups / direction changes?
  Yes → deliver output + generate log in logs/
  No  → deliver output only

Small task → deliver output only, no log.

Log filename: YYYYMMDD-HHMMSS-{member}-{task-slug}-{4-char-random}.md
Template: logs/LOG-template.md
logs/ is gitignored — local only.
id in frontmatter must match filename exactly.

---

## Claude never

- Writes to PROJECT.md, LORE.md, UNKNOWNS.md, team LORE
- Outputs drafts only — human pastes and commits
- Reads full requirements/done/ files — bash grep only
- Assumes REQ silently — always confirms
- Makes semantic inference in LORE matching — text only
- Proposes rule changes without explicit invitation
- Promotes to team LORE from one project only
- Skips health check
- Skips engineering standards self-check during execution
