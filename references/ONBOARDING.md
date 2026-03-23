# Lore — Onboarding

> Do this once. 10 minutes.

---

## Step 1: Clone team-lore

```bash
git clone {team-lore-repo-url} ~/team-lore
```

---

## Step 2: Set your identity

```bash
mkdir -p ~/.claude
cp ~/team-lore/global-CLAUDE.md ~/.claude/CLAUDE.md
# Edit: set member: [your-name]
```

---

## Step 3: Initialize a project

```bash
cd ~/code/{project-name}

cp ~/team-lore/PROJECT.md .
cp ~/team-lore/LORE.md .
cp ~/team-lore/UNKNOWNS.md .
mkdir -p requirements/active requirements/done logs

cp ~/team-lore/hooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit

echo "logs/" >> .gitignore
git add PROJECT.md LORE.md UNKNOWNS.md .gitignore
git commit -m "[{project}] init lore"
```

---

## Step 4: Fill PROJECT.md

Three sections. Each has a strict definition in the file.
Ask a senior teammate if unsure what belongs in each section.

```bash
git add PROJECT.md
git commit -m "[{project}] fill engineering standards"
```

---

## Step 5: Old project? Run code archaeology

```
Say: "code archaeology src/{most-critical-module}"
Claude outputs Certain / Uncertain lists
You answer what you know
Claude generates LORE drafts → you paste and commit
```

Start with the module most likely to cause bugs. One module per session.

---

## Daily workflow

```
Start task → Claude reads PROJECT.md + greps LORE automatically
             Surfaces relevant past mistakes
             Drafts PRD if needed → you lock it
             Executes with engineering standards self-check

End task   → Log auto-generated in logs/ if corrections happened

Friday     → "weekly review" — 30 min
             Claude lists logs + generates LORE drafts
             You decide what to keep → paste → commit → delete logs
```

---

## Quick commands

| Say | Claude does |
|-----|-------------|
| `lore: {keyword}` | Search past learnings |
| `weekly review` | Process logs, generate LORE drafts |
| `code archaeology {path}` | Bootstrap LORE from code |
| `req close {name}` | Archive completed requirement |
| `project close` | Close project |
| `check this rule: {rule}` | Diagnose a rule |
