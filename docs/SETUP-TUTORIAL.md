# Claude Code Setup Tutorial

A step-by-step guide to setting up Claude Code for maximum productivity. Follow each step in order - by the end, you'll have a fully configured environment with skills, hooks, rules, agents, and MCPs.

---

## Table of Contents

1. [Install Claude Code](#step-1-install-claude-code)
2. [Create Your CLAUDE.md](#step-2-create-your-claudemd)
3. [Set Up Rules](#step-3-set-up-rules)
4. [Install Everything Claude Code (ECC)](#step-4-install-everything-claude-code)
5. [Configure MCP Servers](#step-5-configure-mcp-servers)
6. [Set Up Hooks](#step-6-set-up-hooks)
7. [Add Subagents](#step-7-add-subagents)
8. [Learn the Key Commands](#step-8-learn-the-key-commands)
9. [Manage Your Context Window](#step-9-manage-your-context-window)
10. [Editor Integration](#step-10-editor-integration)

---

## Step 1: Install Claude Code

Claude Code runs in your terminal, as a desktop app, a web app, or inside your IDE.

**Terminal (CLI):**

```bash
npm install -g @anthropic-ai/claude-code
```

**Desktop App:** Download from [claude.ai/code](https://claude.ai/code) (Mac and Windows).

**IDE Extensions:** Available for VS Code and JetBrains - search "Claude Code" in the extension marketplace.

**Verify it works:**

```bash
claude --version
```

Launch it:

```bash
cd your-project
claude
```

You should see the Claude Code prompt. Type a message and hit Enter to confirm it's working, then move on.

---

## Step 2: Create Your CLAUDE.md

`CLAUDE.md` is how you teach Claude about your project and preferences. Claude reads it automatically at the start of every session.

There are two levels:

| Level | Location | Scope |
|-------|----------|-------|
| **User** | `~/.claude/CLAUDE.md` | All projects (personal preferences) |
| **Project** | `./CLAUDE.md` in repo root | This project only |

### Create a user-level CLAUDE.md

```bash
mkdir -p ~/.claude
touch ~/.claude/CLAUDE.md
```

Add your personal defaults:

```markdown
# My Claude Code Preferences

## Code Style
- Prefer immutability - never mutate objects or arrays
- Many small files over few large files (200-400 lines typical, 800 max)
- No emojis in code or comments

## Git
- Conventional commits: feat:, fix:, refactor:, docs:, test:
- Small, focused commits
- Always test before committing

## Testing
- TDD: write tests first
- 80% minimum coverage

## Privacy
- Never paste secrets, API keys, tokens, or passwords
- Review output before sharing
```

### Create a project-level CLAUDE.md

In your repo root:

```markdown
# Project Name

## Overview
[What this project does, tech stack, key dependencies]

## Running Tests
[Exact commands to run tests]

## Architecture
[Key directories and what lives where]

## Available Commands
- /tdd - Test-driven development
- /plan - Implementation planning
- /code-review - Quality review
```

> **Tip:** Keep it concise. Claude reads the entire file every session - don't bloat it with info Claude can find by reading the code.

---

## Step 3: Set Up Rules

Rules are modular `.md` files that Claude always follows. They're more organized than stuffing everything into CLAUDE.md.

```bash
mkdir -p ~/.claude/rules
```

Create focused rule files:

**`~/.claude/rules/security.md`**
```markdown
# Security Rules
- No hardcoded secrets - use environment variables
- Validate all user inputs
- Parameterized queries only - no string concatenation for SQL
- Enable CSRF protection
```

**`~/.claude/rules/coding-style.md`**
```markdown
# Coding Style
- Prefer const over let; never var
- Keep functions under 50 lines
- One export per file for components
- Use early returns to reduce nesting
```

**`~/.claude/rules/testing.md`**
```markdown
# Testing Rules
- Write tests before implementation (TDD)
- 80% minimum coverage target
- Unit tests for utilities, integration tests for APIs
- E2E tests for critical user flows
```

**`~/.claude/rules/git-workflow.md`**
```markdown
# Git Workflow
- Conventional commits: feat:, fix:, refactor:, docs:, test:
- Never commit to main directly
- PRs require review
- All tests must pass before merge
```

Claude automatically picks up every `.md` file in `~/.claude/rules/` and your project's `.claude/rules/` directory.

---

## Step 4: Install Everything Claude Code

ECC is a plugin that bundles production-ready skills, hooks, agents, commands, and MCP configs. It's the fastest way to go from zero to a fully configured environment.

**Option A: Plugin Marketplace (recommended)**

```bash
# From inside Claude Code
/plugin marketplace add affaan-m/everything-claude-code
```

**Option B: Manual install from source**

```bash
# Clone and install
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code
./install.sh --profile full
```

Install profiles:

| Profile | What you get |
|---------|-------------|
| `minimal` | Core rules and essential hooks |
| `standard` | Rules + hooks + commands + key skills |
| `full` | Everything: all agents, skills, commands, hooks, MCP configs |

You can also install language-specific components:

```bash
./install.sh typescript python golang
```

---

## Step 5: Configure MCP Servers

MCP (Model Context Protocol) servers connect Claude to external services - GitHub, databases, deployment platforms, documentation, and more.

### Essential MCPs to start with

Add these to your `~/.claude.json` under `mcpServers`:

**GitHub** (interact with PRs, issues, repos):
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_your_token_here"
      }
    }
  }
}
```

**Memory** (persistent knowledge across sessions):
```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**Context7** (live documentation lookup):
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

**Playwright** (browser automation and testing):
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp", "--browser", "chrome"]
    }
  }
}
```

### Nice-to-have MCPs

| MCP | What it does |
|-----|-------------|
| `sequential-thinking` | Chain-of-thought reasoning for complex problems |
| `vercel` | Deploy and manage Vercel projects |
| `railway` | Railway deployments |
| `supabase` | Database operations |
| `firecrawl` | Web scraping and crawling |
| `exa-web-search` | AI-powered web search |

> **Warning:** Each enabled MCP consumes context window. Keep **under 10 MCPs enabled** at a time. Configure many, enable few. Disable unused ones per-project using `disabledMcpServers` in your project settings.

See `mcp-configs/mcp-servers.json` in this repo for a full catalog of pre-configured servers you can copy from.

---

## Step 6: Set Up Hooks

Hooks are automations that fire on specific events. They catch mistakes, enforce quality, and save you time.

### Hook types

| Hook | When it fires | Use for |
|------|--------------|---------|
| `PreToolUse` | Before a tool runs | Validation, reminders, blocking bad commands |
| `PostToolUse` | After a tool finishes | Auto-formatting, type checking, linting |
| `Stop` | When Claude finishes responding | Final quality checks |
| `SessionStart` | When a session begins | Loading context, detecting environment |
| `PreCompact` | Before context compaction | Saving state |

### Starter hooks

Add to your `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -q 'npm run dev\\|yarn dev\\|pnpm dev' && [ -z \"$TMUX\" ]; then echo 'Consider running dev servers inside tmux for session persistence' >&2; fi"
          }
        ],
        "description": "Remind to use tmux for dev servers"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.file_path // empty'); if [ -n \"$FILE\" ] && echo \"$FILE\" | grep -qE '\\.(js|ts|jsx|tsx)$'; then npx prettier --write \"$FILE\" 2>/dev/null; fi"
          }
        ],
        "description": "Auto-format JS/TS files after edits"
      }
    ]
  }
}
```

> **Tip:** ECC ships with a full hook suite. If you installed ECC in Step 4, these are already configured. Use `ECC_HOOK_PROFILE=minimal` to start light, then switch to `standard` or `strict` as you get comfortable.

---

## Step 7: Add Subagents

Subagents are specialized Claude instances that handle focused tasks. The main Claude delegates to them, keeping its own context clean.

### Create your first agent

```bash
mkdir -p ~/.claude/agents
```

**`~/.claude/agents/code-reviewer.md`**
```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
tools:
  - Read
  - Glob
  - Grep
model: sonnet
---

You are a code reviewer. Analyze the provided code for:

1. **Security** - injection vulnerabilities, hardcoded secrets, unsafe operations
2. **Quality** - naming, structure, duplication, complexity
3. **Testing** - adequate coverage, edge cases, missing tests
4. **Performance** - N+1 queries, unnecessary re-renders, memory leaks

Output a structured review with severity levels: critical, warning, info.
```

**`~/.claude/agents/planner.md`**
```markdown
---
name: planner
description: Creates implementation plans for features
tools:
  - Read
  - Glob
  - Grep
model: sonnet
---

You are a planning agent. When given a feature request:

1. Analyze the existing codebase to understand patterns and conventions
2. Break the feature into discrete, testable steps
3. Identify files to create or modify
4. Flag risks or dependencies
5. Output a numbered plan ready for execution
```

> **Tip:** Use `model: sonnet` for most agents to save cost. Reserve `model: opus` for complex tasks like architecture decisions.

ECC ships with 30 pre-built agents covering code review, TDD, security, build errors, E2E testing, and more.

---

## Step 8: Learn the Key Commands

These are the commands you'll use daily:

### Planning and development

| Command | What it does |
|---------|-------------|
| `/plan` | Create an implementation plan before coding |
| `/tdd` | Test-driven development workflow |
| `/build-fix` | Diagnose and fix build errors |
| `/refactor-clean` | Remove dead code and clean up |

### Quality and review

| Command | What it does |
|---------|-------------|
| `/code-review` | Review code for quality and security |
| `/test-coverage` | Analyze test coverage gaps |
| `/e2e` | Generate and run end-to-end tests |
| `/verify` | Run the verification loop |

### Session management

| Command | What it does |
|---------|-------------|
| `/compact` | Manually compress context when running low |
| `/save-session` | Save session state for later |
| `/resume-session` | Resume a saved session |

### Learning and docs

| Command | What it does |
|---------|-------------|
| `/learn` | Extract reusable patterns from the current session |
| `/docs` | Look up live documentation for a library |
| `/skill-create` | Generate a new skill from git history |

### Keyboard shortcuts

| Shortcut | Action |
|----------|--------|
| `Shift+Enter` | Multi-line input |
| `Tab` | Toggle extended thinking display |
| `Esc Esc` | Interrupt Claude / restore code |
| `Shift+Tab` | Cycle between modes (code/plan/fast) |
| `@` | Search for files to reference |
| `/` | Open slash command menu |

---

## Step 9: Manage Your Context Window

The context window is your most precious resource. A 200k window can shrink to 70k if you have too many MCPs and tools enabled. Here's how to stay efficient:

### Do

- **Disable unused MCPs** - configure many, enable few per project
- **Use subagents** - delegate research and review to keep the main context clean
- **Compact proactively** - run `/compact` when context gets heavy
- **Use skills** - they load on demand instead of sitting in context permanently
- **Fork conversations** - use `/fork` for parallel tasks instead of overloading one session

### Don't

- Enable all MCPs at once (each one registers tools that consume context)
- Paste large files into chat (use `@filename` to reference them)
- Run multiple unrelated tasks in one session
- Ignore the context percentage in your status line

### Check your context health

```
/mcp          # See which MCPs are enabled
/statusline   # Monitor context % in real-time
```

> **Rule of thumb:** Keep under 10 MCPs enabled and under 80 tools active.

---

## Step 10: Editor Integration

Claude Code works from any terminal, but pairing it with an editor gives you real-time file tracking and quick navigation.

### VS Code / Cursor

Install the Claude Code extension from the marketplace. It provides:
- Integrated chat panel in the sidebar
- Automatic file sync between editor and Claude
- LSP functionality for type checking and go-to-definition

### Zed

A fast, Rust-based editor with native Claude integration:
- Agent Panel for tracking file changes in real-time
- `Cmd+Shift+R` command palette for slash commands
- Minimal resource usage (important when running Opus)

### Terminal-only setup

Split your terminal:
- Left pane: Claude Code
- Right pane: your editor (vim, neovim, etc.)

Use tmux for session persistence:

```bash
tmux new -s dev
# Left pane: claude
# Right pane: vim
# Detach with Ctrl+B, D
# Reattach with: tmux attach -t dev
```

### Git worktrees for parallel work

Run multiple Claude instances on the same repo without conflicts:

```bash
git worktree add ../feature-a feature-a
git worktree add ../feature-b feature-b
# Run separate Claude instances in each directory
```

---

## Quick-Start Checklist

Use this to track your progress:

- [ ] Claude Code installed and running
- [ ] User-level `~/.claude/CLAUDE.md` created
- [ ] Project-level `CLAUDE.md` added to your repo
- [ ] Rules created in `~/.claude/rules/`
- [ ] ECC plugin installed
- [ ] At least 2-3 MCP servers configured (GitHub + Memory recommended)
- [ ] Hooks set up (or using ECC defaults)
- [ ] At least one subagent created (or using ECC agents)
- [ ] Key commands practiced: `/plan`, `/tdd`, `/code-review`
- [ ] Context window management understood

---

## What's Next

Once you're comfortable with the basics:

1. **Read the [Shorthand Guide](../the-shortform-guide.md)** for tips from 10+ months of daily use
2. **Read the [Longform Guide](../the-longform-guide.md)** for advanced patterns: token optimization, continuous learning, verification loops, and parallelization
3. **Read the [Security Guide](../the-security-guide.md)** for attack vectors, sandboxing, and defense patterns
4. **Explore `skills/`** in ECC for 136 workflow skills across 12 language ecosystems
5. **Explore `examples/`** for real-world project configurations (Next.js, Django, Go, Laravel, Rust)

---

*Part of [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) - the production-ready Claude Code plugin.*
