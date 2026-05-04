---
name: message
description: Generate professional Git commit messages following Conventional Commits specification. Triggers: user pastes a git diff, asks to "write a commit message", "help me commit this", or describes code changes wanting a commit. Also triggers for casual phrasing like "what should I write for my commit?"
---

# Git Commit Message Generator

Generate concise, accurate Git commit messages following the **Conventional Commits** specification, in **English**.

---

## Two Trigger Modes

### Mode A — Git Diff Input

User pastes a `git diff` or `git diff --staged` output.

**Steps:**

1. Analyze the diff: identify what changed, why it likely changed, and what the impact is
2. Infer the commit type from the nature of the change (see Types below)
3. Determine the scope from the file paths or module names affected (see Scope Rules below)
4. Output the commit message (see Format below) — nothing else

### Mode B — Natural Language Description

User describes what they changed in plain language.

**Steps:**

1. Ask for clarification **only** in these specific cases:
   - Cannot determine if the change is a new feature (`feat`) or a bug fix (`fix`)
   - Cannot identify which module or service was affected
   - The description mentions "refactor" but also hints at behavior changes
   - Otherwise, make a reasonable inference and proceed
2. Map their description to a commit type and scope
3. Output the commit message — nothing else

---

## Conventional Commits Format

```
<type>(<scope>): <short description>

[optional footer]
```

### Rules

- **Subject line**: ≤72 characters, imperative mood, no period at end
- **Type**: lowercase, from the list below
- **Scope**: optional, lowercase, noun describing the section of codebase (e.g. `auth`, `user-service`, `db`)
- **Body**: never include — always omit
- **Footer**: only for breaking changes (`BREAKING CHANGE: ...`) or issue references (`Closes #123`)

### Commit Types

| Type       | When to use                              |
| ---------- | ---------------------------------------- |
| `feat`     | New feature or capability                |
| `fix`      | Bug fix                                  |
| `refactor` | Code restructure, no behavior change     |
| `perf`     | Performance improvement                  |
| `test`     | Add or update tests                      |
| `docs`     | Documentation only                       |
| `chore`    | Build, deps, tooling, config             |
| `style`    | Formatting, whitespace (no logic change) |
| `ci`       | CI/CD pipeline changes                   |
| `revert`   | Revert a previous commit                 |

---

## Scope Rules

**Include a scope** when the change is clearly isolated to one module, layer, or service.

**Omit the scope** when:
- The change spans multiple unrelated modules with no dominant one
- It's a project-wide change (e.g. global config, root-level tooling)
- The type already implies the location (e.g. `ci:` for pipeline files)

---

## Output Format

### Output structure

Output **only** the commit message in a code block — no preamble, no explanation, no reasoning before or after.



---

## Java / Agent-Specific Guidance

Since this user works on **Java backend and AI agent development**, apply these heuristics:

- Files in `service/`, `impl/` → likely `feat` or `refactor`
- Files in `mapper/`, `repository/`, `dao/` → scope: `db` or `persistence`
- Files in `controller/`, `api/` → scope: `api`
- Files in `agent/`, `tool/`, `chain/` → scope: `agent`
- Files in `config/`, `application.yml` → `chore(config)` or `feat(config)`
- Adding `@Test` classes → `test`
- Updating `pom.xml` deps only → `chore(deps)`

---

## Edge Cases

- **Multiple unrelated changes in one diff**: point it out and suggest splitting into separate commits; if the user insists on one, use the dominant change type
- **Trivial change** (e.g. fix typo): use `fix` or `style`, one-liner only, no body
- **Breaking change**: append `!` after type/scope (`feat(api)!:`) and add `BREAKING CHANGE:` footer
- **WIP / draft**: prefix subject with `WIP:` and note it's not ready for review
