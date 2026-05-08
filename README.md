# claude-code-skills

My personal Claude Code plugin marketplace. Add it to get my skills and plugins.

## Install

```shell
/plugin marketplace add thezmmm/claude-code-skills
```

## Plugins

### `git-tool`

Git workflow tools with two skills: generate commit messages and execute full git commits.

```shell
/plugin install git-tool@thezmmm-skills
```

#### Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| `message` | `/git-tool:message` | Generate a commit message from a diff or natural language description. Follows [Conventional Commits](https://www.conventionalcommits.org/) spec. Triggers when you paste a `git diff` or describe what you changed. |
| `commit` | `/git-tool:commit` | Execute a full commit workflow. Always calls the `message` skill to generate the message first. When ≥ 5 files are changed, groups them into logical batches and proposes a multi-commit plan before staging anything. |

### `dev-tool`

Developer productivity skills: code review and README writer.

```shell
/plugin install dev-tool@thezmmm-skills
```

#### Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| `review` | `/dev-tool:review` | Review code for correctness, security, performance, and maintainability. Auto-detects input: pasted diff, pasted code, file paths, staged changes, or branch diff. Findings are grouped by severity (CRITICAL / MAJOR / MINOR / NIT) with file and line references. Verdict is one of APPROVE / APPROVE WITH COMMENTS / REQUEST CHANGES / ESCALATE. |
| `readme` | `/dev-tool:readme` | Generate or update a project `README.md` by probing the codebase (package metadata, config files, env vars, scripts). Produces factual, section-calibrated output — only includes sections backed by evidence. Handles new projects, existing READMEs (rewrite or improve), and monorepos. |

## How Skills Are Invoked

Plugins installed via `/plugin install` must be invoked **manually** with the slash command (e.g. `/git-tool:message`, `/git-tool:commit`). They are not injected into Claude's context automatically.

To have a skill **auto-injected** into Claude's context (so Claude can trigger it without you typing a command), copy the `SKILL.md` to:

```
~/.claude/skills/<skill-name>/SKILL.md
```

## Adding a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`
2. Create `plugins/<name>/skills/<skill-name>/SKILL.md` (one subdirectory per skill)
3. Add an entry to `.claude-plugin/marketplace.json`
4. Push — users refresh with `/plugin marketplace update thezmmm-skills`
