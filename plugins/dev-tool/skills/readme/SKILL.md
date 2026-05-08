---
name: readme
description: Write or update a project README.md. Triggers: user says "write README", "generate README", "create README", "help me with README", "update README", "写README", "帮我写README", "生成README", or any request to document a project for others to understand, install, or use.
---

# README Writer

Generate or update a professional README.md for a project. Probe the codebase first — every section must be grounded in what actually exists, not guesses.

---

## Input Detection

Determine the starting point (in priority order):

1. **Existing README** — if `README.md` (or `readme.md`) exists in the project root, read it and ask: _"Do you want me to rewrite it completely or just improve what's there?"_ Default to improve unless the user says otherwise.
2. **User-provided description** — user pastes a blurb or explains the project → use it as the authoritative project description, then fill in the rest from the codebase.
3. **Codebase only** — no description given → infer purpose from file structure, entry points, and configuration files (see Tech Stack Probing below).

If none of the above yield enough context to write even a one-sentence description, ask: _"Can you describe what this project does in one sentence?"_ Do not fabricate a description.

---

## Tech Stack Probing

Before writing, gather facts by reading the following (read only what exists — do not error on missing files):

| Priority | File(s) to read | What to extract |
|----------|-----------------|-----------------|
| High | `package.json` | name, description, scripts (`start`, `dev`, `build`, `test`), main dependencies |
| High | `pyproject.toml` / `setup.py` / `setup.cfg` | package name, description, dependencies, entry points |
| High | `pom.xml` / `build.gradle` | artifactId, description, Java version, key dependencies |
| High | `go.mod` | module name, Go version |
| High | `Cargo.toml` | package name, description, edition |
| Medium | `Dockerfile` / `docker-compose.yml` | exposed ports, service names, environment variables |
| Medium | `.env.example` / `.env.template` | required environment variables and their purpose |
| Medium | `Makefile` | available targets (install, test, run, deploy) |
| Medium | `requirements.txt` / `poetry.lock` | Python dependencies for stack identification |
| Low | Top-level `*.ts` / `*.js` / `*.py` / `*.java` entry files | confirm tech stack, identify framework (Express, FastAPI, Spring, etc.) |
| Low | `LICENSE` | license type |
| Low | `CONTRIBUTING.md` | if exists, reference rather than duplicate |

Use Glob to list the root directory before reading individual files. Stop probing once you have enough to populate all applicable sections.

---

## README Sections

Determine which sections to include based on what you found. Mark optional sections only if the evidence supports them.

### Always include

1. **Project name + one-line description** — from package metadata or inferred; must be factual
2. **Features** — 3–6 bullet points describing what the project does; derive from code/config, not imagination
3. **Prerequisites** — runtime versions (Node ≥ X, Python ≥ X, Java X, Go X) and any required external services
4. **Installation** — exact commands to clone and install dependencies
5. **Usage** — minimal working example: how to start or invoke the project after installing

### Include when evidence exists

6. **Configuration** — environment variables table (Name | Default | Description) derived from `.env.example`, `Dockerfile`, or config files; omit if no env vars found
7. **Available Scripts / Commands** — table of `npm run X`, `make X`, or `python -m X` commands derived from `package.json` scripts, `Makefile` targets, etc.
8. **Project Structure** — short annotated tree (≤ 15 lines) when the layout is non-obvious; omit for simple single-file projects
9. **API Reference** — only when the project exposes a public HTTP API or SDK; show the most important 3–5 endpoints/functions
10. **Testing** — exact command to run tests; add only if a test command is discoverable
11. **Deployment** — Docker or cloud instructions; add only if `Dockerfile` or deployment config exists
12. **Contributing** — link to `CONTRIBUTING.md` if it exists; otherwise a 3-line standard paragraph
13. **License** — one line derived from `LICENSE` file; if no LICENSE exists, omit this section

### Never add without evidence

- Badges (build, coverage, npm version) — only include if CI config files exist
- Screenshots or demo GIFs — only if the user provides them or asks for placeholders
- Roadmap or changelog — only if user asks

---

## Output Format

Write the README in GitHub-Flavored Markdown. Use this skeleton and fill in real content:

```markdown
# <Project Name>

> <One-line tagline — factual, not marketing>

<Optional: 1–2 sentence expansion if the tagline alone is insufficient>

## Features

- <feature 1>
- <feature 2>
- ...

## Prerequisites

- <runtime> ≥ <version>
- <external dependency, e.g. PostgreSQL 14+>

## Installation

\`\`\`bash
git clone <repo-url>
cd <project-dir>
<install command>
\`\`\`

## Usage

\`\`\`bash
<minimal start command>
\`\`\`

<Brief description of what happens after running>

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `VAR_NAME` | `value` | What it controls |

## Available Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server |

## Project Structure

\`\`\`
<annotated tree>
\`\`\`

## Testing

\`\`\`bash
<test command>
\`\`\`

## Deployment

<Docker or cloud instructions>

## Contributing

<Contributing guidance or link>

## License

<License name> — see [LICENSE](LICENSE)
```

Omit any section for which you found no evidence. Do not include empty sections or placeholder text like "coming soon."

---

## Writing Rules

- **Factual over fluent.** A command that works beats a polished sentence that doesn't.
- **Exact versions.** Write `Node ≥ 18` not `Node (latest)` unless you cannot determine the version.
- **Working commands.** Every code block must be a command someone could actually run — no `<placeholder>` left in shell blocks.
- **No emojis** in headings or body text unless the project's existing README already uses them.
- **Language.** Before writing (or when the user first invokes this skill), ask: _"Which language should the README be in — English, Chinese (中文), or other?"_ Use the answer for all prose. Section headings may stay in English regardless of the chosen language for internationalization compatibility. Skip the question only if the user already specified a language in their request.
- **Length calibration.** A CLI utility with 3 files needs a 30-line README. A microservice platform with Docker and env vars needs 100+ lines. Match depth to complexity.

---

## Delivery

1. If a filesystem Write tool is available, write to `README.md` at the project root. Otherwise, output the full Markdown content in the conversation.
2. After writing, report: _"README written — N sections, M lines."_ List the sections included and note any sections omitted due to missing evidence.
3. If you omitted a section the user might expect (e.g. no `.env.example` found so Configuration was skipped), call it out explicitly: _"No `.env.example` found — skipped Configuration. Add one if the project uses env vars."_

---

## Edge Cases

- **Monorepo**: write one root README that explains the repo layout and links to per-package READMEs; do not attempt to document every package in one file.
- **Private / internal tool**: skip Contributing section; replace License section with "Internal use only" if no LICENSE file exists.
- **Near-empty project** (< 5 files, no config): write a minimal README (Name + Description + Usage only) and note that more sections can be added as the project grows.
- **Existing README — quality evaluation**: before rewriting, run through this checklist against the current README and the probed codebase facts. Each item is pass (✓) or fail (✗):

  | # | Check | Pass condition |
  |---|-------|----------------|
  | 1 | Project name + description present | H1 heading exists and description is one factual sentence, not marketing copy |
  | 2 | Installation is runnable | Code block with exact clone + install commands; no `<placeholder>` tokens |
  | 3 | Usage is runnable | At least one code block showing how to start or invoke; no `<placeholder>` tokens |
  | 4 | Versions are pinned | Runtimes specified as `≥ X.Y`, not "latest" or omitted |
  | 5 | Env vars covered | All vars in `.env.example` / `Dockerfile` appear in a Configuration section (or there are none) |
  | 6 | Scripts covered | All significant `package.json` / `Makefile` targets documented (or there are none) |
  | 7 | No stale content | Commands, file paths, and version numbers match what the codebase actually has today |
  | 8 | Depth matches complexity | Simple project (< 5 files) → ≤ 40 lines; complex project (Docker + env + API) → ≥ 60 lines |

  **Decision rule:**
  - All 8 pass → report _"README passes all 8 checks — no rewrite needed."_ Stop. Do not write the file.
  - 1–2 fail → report the failing items as targeted improvements; ask _"Want me to fix just these items?"_ Do not rewrite the whole file unless the user says so.
  - 3+ fail → propose a full rewrite; show the checklist results as the rationale.
- **Non-standard project root**: if `README.md` should go in a subdirectory (user specified), write it there.
