---
name: commit
description: Stage changes and create a git commit. Triggers: user says "commit this", "commit my changes", "git commit", "help me commit", or any request to actually execute a commit (not just generate a message). Always uses the `message` skill to generate the commit message first.
---

# Git Commit Executor

Execute a complete git commit workflow. You **must** invoke the `message` skill to generate the commit message before running any git command.

---

## Workflow

### Step 1 — Assess scope of changes

Run `git status` and `git diff` (unstaged) + `git diff --staged` (staged) to get the full picture.

**If the total number of changed files is ≥ 5**, apply the **Multi-Commit Rule** (see below) before proceeding to Step 2.

Otherwise, treat all changes as a single commit and continue from Step 2.

### Step 2 — Confirm staged changes

- If nothing is staged, ask the user which files to stage, or offer to stage based on the commit plan
- If files are staged, proceed

### Step 3 — Generate commit message via `message` skill

**Always invoke the `message` skill for each commit.** Do not write a commit message yourself.

- Pass the diff for the current batch of files to the `message` skill
- Wait for the `message` skill to output the final commit message before proceeding

### Step 4 — Execute the commit

Run `git add <files in this batch>` then `git commit -m "<message from Step 3>"` using the exact message produced by the `message` skill — do not paraphrase, truncate, or alter it.

### Step 5 — Repeat or confirm success

- **Multi-commit**: repeat Steps 3–4 for each remaining batch until all changes are committed
- **Single commit**: show `git log --oneline -1` so the user can verify the commit landed correctly
- After all batches are done, show `git log --oneline -<n>` where n = number of commits made

---

## Multi-Commit Rule

Triggered when changed files ≥ 5. Group files into logical batches **before staging anything**:

### Grouping strategy (apply in order)

1. **By commit type first**: separate `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `ci` changes — files of different types almost never belong in the same commit
2. **By module/layer within the same type**: e.g. `api` changes and `db` changes are separate even if both are `feat`
3. **By independent feature**: unrelated features go into separate commits even if they share the same type and layer

### Presentation

Before executing any commits, present the proposed plan to the user:

```
Proposed commits (3 total):
  1. feat(api): ...   → UserController.java, AuthController.java
  2. chore(deps): ... → pom.xml
  3. test(auth): ...  → AuthServiceTest.java, UserServiceTest.java
Proceed? (yes / adjust)
```

Wait for confirmation before staging or committing anything. If the user says "adjust", let them reassign files between batches or merge/split batches.

### When NOT to split

- User explicitly says "one commit" or "squash everything" → respect it, use dominant change type
- All files are genuinely part of one atomic change (e.g. a rename that touches 20 files) → keep as one commit and note why

---

## Rules

- **Never skip the `message` skill** — even if the user provides a commit message themselves, still invoke `message` to validate or refine it before committing
- **Do not amend existing commits** unless the user explicitly asks (`--amend`)
- **Do not force-push** under any circumstance
- **Do not bypass hooks** (`--no-verify`) unless the user explicitly requests it
- If `git commit` fails (e.g. pre-commit hook rejects it), report the error and fix the underlying issue before retrying

---

## Edge Cases

- **User provides their own message**: pass it to `message` skill as "natural language description" for validation; use the skill's output
- **Nothing to commit**: report `git status` output and stop
- **Merge conflict markers present**: warn the user and do not commit
- **Detached HEAD**: warn before committing; ask user to confirm
