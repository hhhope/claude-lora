---
tags: [lore/requirement]
project: project-name
req: req-name
domain: domain-name
status: draft
estimate: large
created: YYYY-MM-DD
---

# REQ — [req-name]

> estimate: small (< 5 min, no code risk, PRD optional)
>           large (≥ 5 min or has risk, PRD required before execution)

---

## Goal

[One sentence — what outcome, not what to build]

---

## Acceptance criteria

> Each must be verifiable. "Implemented X" is not acceptable.

- [ ]
- [ ]

---

## Out of scope

-

---

## Risk flags

> Claude fills this when drafting PRD.

- Most likely drift point:
- Assumptions made:
- Engineering standards that apply here:

---

## Clarifications

| Status | Item | Priority |
|--------|------|----------|
| [open] | | P2 |
| [resolved] | | resolved: ... |

---

## Decisions

> Claude appends during execution. Committed with code. Permanent record.
> When > 10 entries: move oldest to ## Archived decisions below.

<!--
### [title] — [YYYY-MM-DD]
Context: [≤20 words]
Decision: [≤15 words]
Rationale: [≤20 words]
-->

---

## Archived decisions

> Older decisions moved here when active Decisions exceeds 10 entries.
> Claude does not load these automatically.
> To query after REQ is archived:
>   grep -A 5 "[decision title]" requirements/done/REQ-{this-file-name}.md
