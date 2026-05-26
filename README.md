# Claude Code Tips & Best Practices

A curated, tested guide to Claude Code skills, sub-agents, and workflows, based on concepts from **Boris Cherny** (Claude Code creator) and validated with real-world examples.

> **This guide:** Distills foundational concepts + provides 20+ examples, all tested and documented.  
> **For the complete  Boris Cherny 89-tip guide:** See [howborisusesclaudecode.com](https://howborisusesclaudecode.com)

---

## Quick Navigation

- **New to Claude Code?** → Read the [Core Concepts](#table-of-contents) section below
- **Want working examples?** → Browse [`examples/`](./examples/)
- **Need validation details?** → See [`examples/VALIDATION.md`](./examples/VALIDATION.md)
- **Credits & sources?** → See [`CREDITS.md`](./CREDITS.md) and [`REFERENCES.md`](./REFERENCES.md)

Ready to copy-paste? Browse the **[`examples/`](./examples/)** directory for real-world skills, agents, commands, hooks, and a sample `CLAUDE.md`.

---

## Table of Contents

1. [Skills](#1-skills)
2. [Sub-Agents](#2-sub-agents)
3. [Skills vs Sub-Agents: When to Use Which](#3-skills-vs-sub-agents-when-to-use-which)
4. [Boris Cherny's Workflow (Creator of Claude Code)](#4-boris-chernys-workflow-creator-of-claude-code)
5. [Hooks](#5-hooks)
6. [CLAUDE.md](#6-claudemd)
7. [Permissions](#7-permissions)
8. [MCP Integrations](#8-mcp-integrations)
9. [Bundled Skills Worth Knowing](#9-bundled-skills-worth-knowing)

---

## 1. Skills

Skills are reusable instruction sets stored as Markdown files. Unlike content in `CLAUDE.md`, a skill's body only loads into context when it's actually invoked — so large reference material costs nothing until needed.

### Creating a Skill

Every skill lives in a directory with `SKILL.md` as its entry point:

```
~/.claude/skills/my-skill/       # Personal (all projects)
.claude/skills/my-skill/         # Project-level (this repo only)
```

**Minimal `SKILL.md`:**

```markdown
---
description: Summarizes uncommitted changes and flags anything risky. Use when
  the user asks what changed, wants a commit message, or asks to review their diff.
---

## Current changes

!`git diff HEAD`

## Instructions

Summarize the changes in 2-3 bullet points, then list any risks such as missing
error handling, hardcoded values, or tests that need updating.
```

### How Claude Finds Skills

| Trigger           | How it works                                                 |
| ----------------- | ------------------------------------------------------------ |
| Auto-invocation   | Claude matches your message against each skill's `description` field |
| Direct invocation | You type `/skill-name`                                       |
| Disabled auto     | Add `disable-model-invocation: true` to frontmatter; only slash command works |

**Write descriptions as routing rules:**

```yaml
description: Use when the user asks to deploy, push to production, or release a build.
  Returns a checklist of pre-flight checks before deploying.
```

### Dynamic Context Injection

Prefix a line with `!` to run a shell command and inline its output before Claude sees the skill:

```markdown
## Current git status
!`git status`

## Open issues
!`gh issue list --limit 10`
```

This is better than asking Claude to run the commands itself as the data arrives pre-loaded.

### Supporting Files

A skill directory can hold templates, examples, and scripts:

```
.claude/skills/code-review/
├── SKILL.md              # Main instructions (required)
├── checklist.md          # Review checklist Claude fills in
├── examples/
│   └── sample-review.md  # Shows expected output format
└── scripts/
    └── run-linter.sh     # Script Claude can execute
```

Reference these from `SKILL.md` so Claude knows they exist.

### Scope & Precedence

| Location                     | Applies to              | Precedence |
| ---------------------------- | ----------------------- | ---------- |
| Enterprise                   | All org users           | Highest    |
| Personal `~/.claude/skills/` | All your projects       | ↑          |
| Project `.claude/skills/`    | This repo only          | ↓          |
| Plugin                       | Where plugin is enabled | Lowest     |

**Monorepo tip:** Skills are discovered from `.claude/skills/` in your working directory and all parent directories up to the repo root. If you're editing `packages/frontend/`, Claude also looks in `packages/frontend/.claude/skills/`.

**Live reloading:** Editing a skill takes effect immediately in the running session — no restart needed. Exception: creating a brand new skills directory requires a restart.

---

## 2. Sub-Agents

Sub-agents are isolated Claude instances with their own context window, tool permissions, and optionally a different model. Use them for safety isolation, heavy file-reading tasks, or parallel work.

### Creating a Sub-Agent

```
~/.claude/agents/my-agent.md       # Personal
.claude/agents/my-agent.md         # Project-level
```

**Example - read-only simple code reviewer:**

```markdown
---
name: code-reviewer
description: Reviews code for security issues and style. Use this agent when
  the user asks to review a PR, audit a file, or check for vulnerabilities.
tools: [Read, Grep]
---

You are a code review specialist. Focus on security vulnerabilities,
error handling gaps, and style consistency. Return a concise bullet-point
summary of findings, ordered by severity.
```

### Key Sub-Agent Behaviors

- **Parallel execution:** The main session can spawn multiple sub-agents concurrently. Each runs in its own context; the main session resumes when all finish.
- **Background a sub-agent:** Press `Ctrl+B` to send a running sub-agent to the background while you keep working.
- **No nesting:** Sub-agents cannot spawn their own sub-agents. Don't include `Agent` in a sub-agent's tools list.
- **Tool restriction is a real safety control**, not just a preference — a `Read`-only agent cannot accidentally write or delete files.

### Triggering Sub-Agents

```
# Explicit
"Refactor the auth module, use subagents"

# Appending "use subagents" tells Claude to throw more compute at the problem

# Or define named workflows:
.claude/agents/code-simplifier.md  → runs after every big change
.claude/agents/verify-app.md       → end-to-end test runner
```

### Combining Skills and Sub-Agents

You can attach skills to a sub-agent via frontmatter:

```yaml
---
name: fullstack-dev
skills:
  - frontend-design-system
  - testing-patterns
---
```

The skills load automatically into that sub-agent's context when invoked.

---

## 3. Skills vs Sub-Agents: When to Use Which

|                      | Skill                                            | Sub-Agent                                         |
| -------------------- | ------------------------------------------------ | ------------------------------------------------- |
| **Context**          | Injected into main session                       | Fully isolated                                    |
| **Tool permissions** | Inherits parent's tools                          | Can be restricted (e.g. read-only)                |
| **Model**            | Same as parent                                   | Can use a cheaper/different model                 |
| **Parallelism**      | No                                               | Yes                                               |
| **Best for**         | Reusable expertise, checklists, short procedures | File-heavy tasks, safety isolation, parallel work |

**Rule of thumb:**

- If multiple agents or sessions need the same knowledge → **Skill**
- If you need isolation, restricted tools, or parallel execution → **Sub-Agent**
- You can combine them: spawn a sub-agent that follows a skill

---

## 4. Boris Cherny's Workflow (Creator of Claude Code)

Boris describes his setup as "surprisingly vanilla",  but it's significantly more productive than how most people use Claude Code. These tips come from his January–March 2026 X threads.

### Parallel Sessions

Run 5 Claude Code instances simultaneously in your terminal, each in its own git checkout, numbered 1–5. Use iTerm2 (or equivalent) system notifications so you know when any session needs input.

On top of that, run 5–10 more sessions on `claude.ai/code` in the browser. Use the **`&` command** to background a local session and hand it to the web, or **`--teleport`** to pull a web session back to your terminal.

Start sessions from the iOS/Android app in the morning and check in on them later throughout the day.

### Git Worktrees Instead of Separate Clones

Rather than cloning the repo multiple times, use git worktrees to have multiple branches checked out simultaneously under the same repository:

```bash
git worktree add ../repo-feature-a feature-a
git worktree add ../repo-bugfix-b bugfix-b
```

Cleaner than separate checkouts, and all worktrees share the same `.git` history.

### Plan Mode First

Start most sessions with **Plan mode** (press `Shift+Tab` twice). Iterate on the plan until it's solid, then switch to auto-accept mode and let Claude one-shot the implementation.

> "A good plan is really important to avoid issues down the line."

### Prefer Opus with Thinking

Boris uses Opus 4.x with thinking mode for everything. His reasoning:

> "It's the best coding model I've ever used. Even though it's bigger and slower than Sonnet, since you have to steer it less and it's better at tool use, it is almost always faster in the end."

Less steering + better tool use = faster overall, even with a heavier model.

### Slash Commands for Inner Loops

Any workflow you run multiple times a day should be a slash command. Check them into `.claude/commands/` (or `.claude/skills/`) so the whole team shares them:

```markdown
# .claude/commands/commit-push-pr.md
Commit all staged changes, push to the current branch, and open a PR.
Use the conventional commit format. Include a short summary of what changed.
```

Slash commands can include inline `!`-prefixed bash to pre-compute context before Claude sees the prompt.

### Sub-Agents for Recurring PR Workflows

Boris keeps a small set of named sub-agents in `.claude/agents/`:

- **`code-simplifier`** — cleans up and simplifies code after Claude finishes a big change
- **`verify-app`** — detailed end-to-end testing instructions
- **`oncall-guide`** — runbook for on-call workflows

Appending **"use subagents"** to any request tells Claude to parallelize with more compute.

One team member routes all permission approval requests to an Opus-powered sub-agent via a hook — the sub-agent scans for potential attacks and auto-approves safe ones.

### Verification is the #1 Tip

> "Probably the most important thing to get great results out of Claude Code - give Claude a way to verify its work. If Claude has that feedback loop, it will 2–3x the quality of the final result."

Verification varies by domain: test suites, bash commands, simulators, browser testing (via the Claude Chrome extension). The key is closing the feedback loop so Claude can iterate without you.

### CLAUDE.md as Compounding Memory

The Anthropic team shares a single `CLAUDE.md` checked into git. The whole team adds to it multiple times a week. Any time Claude does something incorrectly, add it so Claude won't repeat the mistake.

During code review, tag `@.claude` on PRs to add learnings directly to `CLAUDE.md` as part of the PR - what Boris calls **"Compounding Engineering."**

---

## 5. Hooks

Hooks are deterministic scripts that fire at lifecycle events. Unlike prompts, they cannot hallucinate — they enforce rules at the system level.

### PostToolUse - Auto-Format After Every Edit

```json
{
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        { "type": "command", "command": "bun run format || true" }
      ]
    }
  ]
}
```

Claude generates well-formatted code ~90% of the time; the hook handles the rest and prevents CI failures.

### PreToolUse - Security Checkpoint

Fires before any tool executes. Use to block dangerous commands, inject project context, or route permission requests to a secondary model for review.

### Other Useful Lifecycle Points

- **`UserPromptSubmit`** - Intercept and modify prompts before Claude sees them
- **`Stop`** - Run checks when a long-running task finishes (useful for async workflows)

Place hook config in `.claude/settings.json` and check it into git so the whole team benefits.

---

## 6. CLAUDE.md

`CLAUDE.md` loads at the start of every session. Think of it as persistent memory for your project.

### What to Put in It

- Build/test/lint commands (so Claude never guesses)
- Coding conventions and style decisions
- PR templates and commit message format
- Past mistakes Claude has made (so it doesn't repeat them)
- Architecture notes and design guidelines

### Example Structure

```markdown
# Project: my-app

## Commands
- **Build:** `bun run build`
- **Test:** `bun run test -- -t "test name"`
- **Lint:** `bun run lint:file -- "file.ts"`
- **Typecheck:** `bun run typecheck`

## Conventions
- Always use `bun`, never `npm`
- Prefer string literal unions over TypeScript enums
- Use `type` over `interface` for object types

## Common Mistakes (Do Not Repeat)
- Don't use `any` — use `unknown` and narrow it
- Don't hardcode API URLs — use the config object

## PR Template
Title: `[scope]: short description`
Body: What changed, why, and how to test it.
```

### CLAUDE.md Scoping

Like skills, `CLAUDE.md` files are loaded hierarchically. A root-level file applies to the whole repo; a `packages/frontend/CLAUDE.md` applies only when working in that subdirectory.

---

## 7. Permissions

Avoid `--dangerously-skip-permissions` for day-to-day work. Instead, pre-approve the commands you know are safe:

```bash
/permissions
```

Check the resulting config into `.claude/settings.json` so the whole team shares the same approved list:

```json
{
  "permissions": {
    "allow": [
      "Bash(bun run build:*)",
      "Bash(bun run test:*)",
      "Bash(bun run lint:file:*)",
      "Bash(bun run typecheck:*)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ]
  }
}
```

Use `--dangerously-skip-permissions` only for sandboxed/containerized environments where isolation is already handled externally.

---

## 8. MCP Integrations

Connect Claude Code to your internal tools via MCP servers. Boris's team uses:

- **Slack** — Claude searches and posts to Slack autonomously
- **BigQuery** — runs analytics queries via the `bq` CLI
- **Sentry** — grabs error logs for debugging

```json
// .mcp.json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://slack.mcp.anthropic.com/mcp"
    }
  }
}
```

Check `.mcp.json` into git so the whole team has the same integrations available.

---

## 9. Bundled Skills Worth Knowing

These are built-in and available in every session — no setup required:

| Skill                  | What it does                                                 |
| ---------------------- | ------------------------------------------------------------ |
| `/simplify`            | Cleans up and simplifies code                                |
| `/debug`               | Systematic debugging workflow                                |
| `/batch`               | Runs multiple tasks in one shot                              |
| `/loop`                | Repeats a task until a condition is met                      |
| `/run`                 | Launches your app to verify a change is working live         |
| `/verify`              | Confirms a code change works without falling back to tests   |
| `/run-skill-generator` | Teaches `/run` and `/verify` how to build and launch your specific project |

**Run `/run-skill-generator` once per project.** It records the launch recipe (install commands, env vars, startup script) as a skill so Claude never has to rediscover it.

---

## 10. Working Examples

This repository includes **20+ tested, validated examples** ready to adapt for your projects:

### Skills (9)

- `summarize-changes` - Diff + risk analysis + commit message suggestion
- `code-review` - Severity-grouped security/correctness/test checklist
- `write-pr-description` - Auto-generate PR title + body from branch
- `db-query` - Safe read-only analytics queries (BigQuery, Snowflake, etc.)
- `write-docs` - Documentation creation/updates with style guide
- `data-cleanup` - CSV/JSON transformation with jq or Python fallback
- `filesystem-search` - Find files on NFS/SMB/S3 with filtering
- `cloud-backup` - S3/GCS/Azure sync with dry-run and cost awareness
- `update-architecture` - Update architecture docs in Carta

### Agents (6)

- `code-simplifier` - Dead code removal, deduplication, over-engineering cleanup
- `verify-app` - Autodetects package manager, runs typecheck → lint → test → build
- `security-reviewer` - OWASP-focused read-only audit (Opus model)
- `content-editor` - Edits and polishes written content
- `system-healthcheck` - Monitors disk, memory, CPU, processes
- `filesystem-audit` - Permission checks, orphaned data, compliance violations

### Commands (4)

- `commit-push-pr` - Commit, push, open PR (requires gh CLI)
- `standup` - Generates yesterday/today/blockers from git log
- `summarize-meeting` - Extracts decisions + action items from notes
- `cleanup-large-files` - Find large/stale files, reclaim space

### All Examples Include:

✅ Tested on Linux (validation report included)  
✅ Prerequisites documented (what CLI tools needed)  
✅ Error handling and fallbacks  
✅ Platform notes (Linux/macOS/BSD compatibility)  
✅ Copy-paste ready; customize for your stack  

**Browse examples:** [`examples/`](./examples/)  
**Validation details:** [`examples/VALIDATION.md`](./examples/VALIDATION.md)

---

## Quick Reference

```
# Create a personal skill
mkdir -p ~/.claude/skills/my-skill && touch ~/.claude/skills/my-skill/SKILL.md

# Create a project sub-agent
mkdir -p .claude/agents && touch .claude/agents/my-agent.md

# Open the agents/skills UI
/agents

# Run a skill directly
/my-skill

# Background a sub-agent
Ctrl+B

# Toggle plan mode
Shift+Tab (twice)

# Pre-approve safe permissions
/permissions

# Teleport a session from web to local
claude --teleport

# Add a git worktree for parallel work
git worktree add ../repo-branch-name branch-name
```

---

## Attribution & Further Reading

**Concept & Workflow:** Boris Cherny (Claude Code creator)  
→ Full 89-tip guide: https://howborisusesclaudecode.com

**Examples & Validation:** This repository (tested May 2026)  
→ Full report: [`FULL-VALIDATION-REPORT.md`](./FULL-VALIDATION-REPORT.md)

**References & Sources:** See [`REFERENCES.md`](./REFERENCES.md) and [`CREDITS.md`](./CREDITS.md)

**Official Documentation:** https://code.claude.com/docs

---

*Contributions welcome. See examples/ for guidelines on adding new skills/agents.*
