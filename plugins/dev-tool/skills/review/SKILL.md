---
name: review
description: Review code for correctness, security, performance, and style. Triggers: user pastes a diff or code snippet and asks for review, says "review this", "code review", "check my code", "is this ok?", "看看这段代码", "review一下", or any request to evaluate code quality before merging or sharing.
---

# Code Review

Perform a structured code review. Be direct and specific — every finding must reference a concrete line, function, or pattern. No generic advice.

---

## Input Detection

Determine what to review from context (in priority order):

1. **Pasted diff** — user pastes `git diff` or `git diff --staged` output
2. **Pasted code** — user pastes a raw code block
3. **File path(s)** — user names specific files → read them with the Read tool
4. **Staged changes** — user says "review my changes" or "review before commit" with no paste → run `git diff --staged`; if nothing staged, run `git diff HEAD`
5. **Branch diff** — user says "review this PR" or "review vs main" → run `git diff main...HEAD`

If the input is ambiguous and none of the above apply, ask: _"What would you like me to review — staged changes, a specific file, or something you want to paste?"_

---

## Review Dimensions

Evaluate the input against all applicable dimensions. Skip dimensions that are clearly irrelevant (e.g. security for a pure documentation change).

### 1. Correctness
- Logic errors, off-by-one, null/nil dereference, unhandled edge cases
- Incorrect assumptions about data shape, ordering, or concurrency
- Missing or incorrect error handling at system boundaries (API calls, DB queries, file I/O)
- Broken invariants: preconditions not checked, postconditions not guaranteed

### 2. Security
- Injection risks: SQL, command, LDAP, XPath, template injection
- Sensitive data exposure: credentials, tokens, or PII in logs, responses, or error messages
- Authentication / authorization gaps: missing checks, privilege escalation paths
- Unsafe deserialization, path traversal, SSRF, open redirect
- Cryptographic misuse: weak algorithms, hardcoded secrets, insecure random

### 3. Performance
- N+1 query patterns, missing indexes implied by query structure
- Unnecessary allocations in hot paths, large object copies
- Blocking calls on async threads, missing pagination on unbounded result sets
- Repeated computation that could be cached or precomputed

### 4. Maintainability
- Functions / methods doing more than one thing (violates SRP)
- Magic numbers or strings that should be named constants
- Misleading names: variables, functions, or types that don't match their behavior
- Dead code, commented-out code, or stale TODOs

### 5. Test Coverage (if tests are in scope)
- Critical paths or edge cases with no test
- Tests that assert the wrong thing (pass even when the code is broken)
- Overly broad mocking that defeats the purpose of the test

### 6. API / Interface Design (if public APIs are changed)
- Breaking changes to existing callers
- Inconsistent naming conventions vs. the rest of the codebase
- Missing or incorrect documentation on public methods

---

## Severity Levels

Assign each finding one of four levels:

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Will cause bugs, data loss, or security vulnerabilities in production — must fix before merge |
| **MAJOR** | Likely to cause problems under real conditions or significantly harms maintainability — strongly recommend fixing |
| **MINOR** | Correctness risk is low but the code is harder to understand or maintain than it needs to be |
| **NIT** | Style preference, cosmetic — take it or leave it |

---

## Output Format

### Structure

```
## Code Review

### Summary
<2–3 sentences: what the change does, overall quality signal, and the single most important finding>

### Findings

**[CRITICAL]** `path/to/file.java:42` — <title>
<Specific explanation of why this is a problem. What can go wrong. Include a minimal fix or the concrete change needed.>

**[MAJOR]** `path/to/file.java:88` — <title>
<Explanation + fix suggestion>

**[MINOR]** `path/to/file.java:15` — <title>
<Explanation>

**[NIT]** `path/to/file.java:7` — <title>
<One-line note>

### Verdict

**[LABEL]**

> <one sentence rationale>

**Before merging:**
- <blocking item referencing finding title, or "(none)" for APPROVE>

**Follow-up (non-blocking):**
- <MINOR/NIT items worth tracking, or omit section entirely if none>
```

### Verdict labels

| Label | When to use |
|-------|-------------|
| **APPROVE** | Zero CRITICAL or MAJOR findings — ship when ready |
| **APPROVE WITH COMMENTS** | Zero CRITICAL, one or more MAJOR — author may merge after addressing MAJORs or documenting why they're deferred |
| **REQUEST CHANGES** | One or more CRITICAL findings — do not merge until all are resolved |
| **ESCALATE** | Finding(s) require architect/security/team input that goes beyond this review's scope; may have no CRITICAL findings but the risk or design decision cannot be resolved here alone |

**Before merging** lists only CRITICAL-derived blockers (for REQUEST CHANGES) or agreed MAJORs (for APPROVE WITH COMMENTS). It must be a concrete, checkable list — not a restatement of the verdict label.

**Follow-up** is optional. Include it when MINOR/NIT items are worth tracking but not blocking merge. Omit the section entirely if there is nothing worth noting.

### Rules for output

- **Reference file and line for every finding.** If reviewing raw code with no filename, use `line N`.
- **No findings section if there are zero issues** — just write "No issues found." after the Summary.
- Group findings by severity, descending. Within a severity group, order by file then line number.
- Do not pad the output. If there are 2 findings, show 2 findings — not a list of empty sections.
- Write in English unless the user wrote in Chinese, in which case write findings in Chinese.

---

## Tech Stack Heuristics

Apply these when reviewing Java backend or AI agent code:

- **Service/impl layer**: check for missing `@Transactional` on multi-step DB operations; check that exceptions are not swallowed silently
- **Controller/API layer**: verify input validation (`@Valid`, manual null checks), check HTTP status codes match the operation
- **Repository/DAO layer**: flag N+1 patterns, check for `findAll()` without pagination on large tables
- **Agent/tool/chain layer**: check that tool outputs are validated before passing downstream; flag unbounded loops or missing max-iteration guards
- **Config / application.yml**: flag secrets not externalized via env vars; flag missing timeouts on HTTP clients
- **`pom.xml` deps**: flag snapshot versions, known CVE ranges (if visible), or duplicate dependency declarations

---

## Edge Cases

- **Very large diff (>500 lines)**: focus on the highest-risk sections (public API surface, auth paths, DB queries) and explicitly note that the review is scoped
- **Pure rename/move**: confirm no logic changed; if logic is interleaved with the rename, separate the concerns in your findings
- **Generated code**: note it's generated, skip style/maintainability findings, focus only on CRITICAL security or correctness issues
- **No staged changes and no paste**: report `git status` output and stop — do not fabricate a review
- **Test-only change**: skip Security and Performance dimensions; focus on Correctness (are the tests actually testing the right thing?) and Maintainability
