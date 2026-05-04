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
