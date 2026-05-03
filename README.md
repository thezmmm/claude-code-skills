# claude-code-skills

My personal Claude Code plugin marketplace. Add it to get my skills and plugins.

## Install

```shell
/plugin marketplace add thezmmm/claude-code-skills
```

## Plugins

### `git-commit`

Generates Git commit messages following the [Conventional Commits](https://www.conventionalcommits.org/) spec.

Triggers when you paste a `git diff`, ask to write a commit message, or describe what you changed.

```shell
/plugin install git-commit@thezmmm-skills
```

## Adding a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`
2. Create `plugins/<name>/skills/<name>/SKILL.md`
3. Add an entry to `.claude-plugin/marketplace.json`
4. Push — users refresh with `/plugin marketplace update thezmmm-skills`
