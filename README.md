# claude-code

> **Everything you need to run Claude Code at scale** — hooks, CLAUDE.md templates, MCP configs, and operator patterns distilled from real sessions.

<p align="center">
  <a href="https://github.com/hmzainjamil/claude-code/stargazers"><img src="https://img.shields.io/github/stars/hmzainjamil/claude-code?style=for-the-badge&labelColor=555&color=yellow" alt="Stars"></a>
  <a href="https://github.com/hmzainjamil/claude-code/forks"><img src="https://img.shields.io/github/forks/hmzainjamil/claude-code?style=for-the-badge&labelColor=555&color=blue" alt="Forks"></a>
  <a href="https://github.com/hmzainjamil/claude-code/issues"><img src="https://img.shields.io/github/issues/hmzainjamil/claude-code?style=for-the-badge&labelColor=555&color=red" alt="Issues"></a>
  <a href="https://github.com/hmzainjamil/claude-code/pulls"><img src="https://img.shields.io/github/issues-pr/hmzainjamil/claude-code?style=for-the-badge&labelColor=555&color=green" alt="PRs"></a>
  <a href="https://github.com/hmzainjamil/claude-code/commits/main"><img src="https://img.shields.io/github/last-commit/hmzainjamil/claude-code?style=for-the-badge&labelColor=555" alt="Last Commit"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-191919?style=flat&labelColor=555" alt="Claude Code">
  <img src="https://img.shields.io/badge/Anthropic-AA3E1E?style=flat&labelColor=555" alt="Anthropic">
  <img src="https://img.shields.io/badge/MCP-5E81AC?style=flat&labelColor=555" alt="MCP">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=flat&labelColor=555&logo=gnubash" alt="Bash">
  <img src="https://img.shields.io/badge/JSON-000000?style=flat&labelColor=555" alt="JSON">
</p>

---

## Why This Exists

Claude Code ships with sane defaults but production use requires careful configuration — hooks, CLAUDE.md hierarchies, MCP server wiring, permission allowlists. This repo is a battle-tested configuration kit and pattern library. No fluff. No screenshots of "just install the extension." Only configs that run.

---

## At a Glance

| Dimension | Detail |
|---|---|
| **Config file** | `~/.claude/settings.json` (global) + `.claude/settings.json` (project) |
| **Instruction file** | `CLAUDE.md` — loaded at session start, injected into context |
| **Hooks** | PreToolCall, PostToolCall, PreCompact, Stop, Notification |
| **MCP protocol** | Model Context Protocol — standardized tool/resource server spec |
| **Skills** | `~/.claude/skills/<name>/SKILL.md` — dynamically loaded knowledge |
| **Memory** | `~/.claude/projects/<hash>/memory/MEMORY.md` — persisted context |
| **Session logs** | `~/.claude/projects/<hash>/*.jsonl` — full turn-by-turn transcript |
| **Worktrees** | Git worktrees via `EnterWorktree`/`ExitWorktree` for parallel agents |
| **Model routing** | Route sub-tasks to Groq/Ollama/Gemini — preserve Claude quota |
| **Cost** | ~$0.003/1k tokens Haiku, ~$0.015/1k Sonnet, ~$0.075/1k Opus |
| **Rate limits** | Tier-dependent; increase via Anthropic Console |
| **Token budget** | `--max-tokens` flag or `budget_tokens` in extended thinking |

---

## 🧠 CONCEPTS

| Concept | Description | Why It Matters |
|---|---|---|
| **CLAUDE.md** | Markdown file loaded as context at session start | Persistent instructions without re-typing |
| **Hooks** | Shell scripts or commands triggered by lifecycle events | Automate pre/post actions per tool call |
| **MCP Server** | Local or remote JSON-RPC 2.0 server exposing tools/resources | Extend Claude with any external capability |
| **Settings.json** | JSON config controlling permissions, model, env vars | Lock down what Claude can and cannot do |
| **Skills** | Markdown documents in `~/.claude/skills/` | Load domain knowledge on demand |
| **Memory files** | Markdown in `memory/` subdir of project | Cross-session persistence without context bleed |
| **Worktrees** | Git worktrees checked out per parallel task | Run multiple agents on the same repo safely |
| **Extended thinking** | Budget tokens for chain-of-thought before answering | Better reasoning on hard architectural decisions |
| **Compact** | Compress conversation history mid-session | Stay under context limit on long sessions |
| **Sub-agents** | Spawn child `Agent()` tasks within a session | Parallelize independent subtasks |
| **Tool allowlist** | Explicit permission grants in settings.json | Avoid confirmation prompts for known-safe tools |
| **Session transcript** | JSONL file with every turn, tool call, result | Audit trail, replay, fine-tune dataset |

### 🔥 Hot

| Pattern | Detail | Why |
|---|---|---|
| Global CLAUDE.md | `~/.claude/CLAUDE.md` — applies to every session | One place to set model routing, caveman mode, etc. |
| PreToolCall hook | Intercept every tool before execution | Block dangerous operations, add logging |
| MCP allowlist | Whitelist MCP tool names in settings.json | Stop permission prompts on known-safe MCPs |
| Parallel worktrees | `EnterWorktree` per sub-agent | Independent git checkouts, no conflicts |

---

## ⚙️ HOW IT WORKS

```
User prompt
    ↓
Claude reads CLAUDE.md (global + project)
    ↓
Claude calls tools (Bash, Read, Edit, MCP, etc.)
    ↓
PreToolCall hook runs (optional intercept/block)
    ↓
Tool executes
    ↓
PostToolCall hook runs (optional logging/alert)
    ↓
Response streamed to user
    ↓
Stop hook runs (optional cleanup/memory write)
```

Config hierarchy (lowest → highest priority):
1. Global: `~/.claude/settings.json`
2. Project: `.claude/settings.json`
3. CLI flags: `--max-tokens`, `--model`, etc.

---

## 🚀 INSTALL

```bash
# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Verify
claude --version

# Set API key
export ANTHROPIC_API_KEY=sk-ant-...

# Run
claude
```

### Global config setup

```bash
mkdir -p ~/.claude/skills ~/.claude/projects

# Create global settings
cat > ~/.claude/settings.json << 'EOF'
{
  "model": "claude-sonnet-4-5",
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)",
      "Bash(python3 *)",
      "Read(*)",
      "Edit(*)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo rm *)"
    ]
  }
}
EOF
```

---

## 📟 USAGE

### Basic session

```bash
# Interactive
claude

# One-shot
claude -p "Refactor auth.py to use async/await"

# With specific model
claude --model claude-opus-4-5 -p "Design the database schema for a SaaS app"

# Non-interactive with output
claude -p "List all TODO comments in src/" --output-format json
```

### Hooks

```bash
mkdir -p ~/.claude/hooks

# PreToolCall — log every bash command
cat > ~/.claude/hooks/pre-tool.sh << 'EOF'
#!/bin/bash
# $1 = tool name, stdin = tool input JSON
TOOL=$1
INPUT=$(cat)
echo "$(date): $TOOL $INPUT" >> ~/.claude/tool-log.txt
EOF
chmod +x ~/.claude/hooks/pre-tool.sh
```

```json
// In settings.json
{
  "hooks": {
    "PreToolCall": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "~/.claude/hooks/pre-tool.sh Bash" }] }
    ]
  }
}
```

### MCP server config

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

---

## ⚙️ CONFIGURATION

| Key | Type | Default | Description |
|---|---|---|---|
| `model` | string | sonnet | Active model — haiku/sonnet/opus + version |
| `permissions.allow` | array | [] | Tool patterns allowed without confirmation |
| `permissions.deny` | array | [] | Tool patterns always blocked |
| `hooks.PreToolCall` | array | [] | Commands run before each tool call |
| `hooks.PostToolCall` | array | [] | Commands run after each tool call |
| `hooks.Stop` | array | [] | Commands run when session ends |
| `hooks.Notification` | array | [] | Commands run on notifications |
| `mcpServers` | object | {} | MCP server configs (name → command) |
| `env` | object | {} | Environment variables injected into all tools |
| `includeCoAuthoredBy` | bool | true | Add "Co-Authored-By: Claude" to commits |
| `cleanupPeriodDays` | int | 30 | Days to retain session logs |
| `autoCompact` | bool | false | Auto-compact when context hits 80% |

---

## 💡 TIPS AND TRICKS

### Context Management

| Tip | Detail | Source |
|---|---|---|
| Keep CLAUDE.md under 500 tokens | Every token in CLAUDE.md costs on every request — be ruthless | [HMZ](https://github.com/hmzainjamil) |
| Use memory files for persistence | Write learnings to `memory/MEMORY.md` in Stop hook — not CLAUDE.md | [HMZ](https://github.com/hmzainjamil) |
| Compact early and often | Run `/compact` before context hits 80% — quality drops at high fill | [HMZ](https://github.com/hmzainjamil) |

### Cost Control

| Tip | Detail | Source |
|---|---|---|
| Route sub-tasks to Groq/Ollama | Use Haiku for code gen, Sonnet for review, Opus only for arch decisions | [HMZ](https://github.com/hmzainjamil) |
| Cache tool results | Store expensive bash outputs in temp files, read instead of re-running | [HMZ](https://github.com/hmzainjamil) |
| Batch tool calls | One multi-step bash script beats 10 separate Bash tool invocations | [HMZ](https://github.com/hmzainjamil) |

### Hooks Power Moves

| Tip | Detail | Source |
|---|---|---|
| Block dangerous rm commands | PreToolCall hook matching `Bash(rm -rf *)` → exit 1 | [HMZ](https://github.com/hmzainjamil) |
| Auto-write session summary | Stop hook → summarize JSONL → append to `memory/MEMORY.md` | [HMZ](https://github.com/hmzainjamil) |
| Send Slack alerts on errors | PostToolCall hook on non-zero exit → `curl` to Slack webhook | [HMZ](https://github.com/hmzainjamil) |

### Parallel Execution

| Tip | Detail | Source |
|---|---|---|
| Use worktrees for parallel agents | Each sub-agent gets its own git worktree — no merge conflicts | [HMZ](https://github.com/hmzainjamil) |
| Spawn sub-agents for research | `Agent(model="haiku")` for parallel data gathering | [HMZ](https://github.com/hmzainjamil) |
| Use TCC blast for bulk tasks | `tcc blast "t1" "t2" "t3"` — parallel cross-LLM execution | [HMZ](https://github.com/hmzainjamil) |

---

## 🔧 TROUBLESHOOTING

| Issue | Cause | Fix |
|---|---|---|
| `ANTHROPIC_API_KEY not set` | Missing env var | `export ANTHROPIC_API_KEY=sk-ant-...` |
| Context limit hit mid-session | Long session, large files | Run `/compact` or start new session with summary |
| MCP server not connecting | Wrong command path or missing env vars | Test server manually: `npx @modelcontextprotocol/server-X` |
| Hook not running | Wrong file path or not executable | `chmod +x hook.sh`, verify path in settings.json |
| Permission prompt loops | Tool not in allowlist | Add `"Bash(your-command *)"` to `permissions.allow` |
| Model not available | Region restriction or wrong model ID | Check Anthropic Console for available models |
| Slow responses | High context fill, complex tools | Compact context, simplify prompt |
| Sub-agent token overuse | Sub-agents using Sonnet/Opus | Route with `Agent(model="claude-haiku-4-5")` |

---

## 📊 ARCHITECTURE

```
~/.claude/
├── settings.json          ← global config (model, permissions, hooks, MCP)
├── CLAUDE.md              ← global instructions (always loaded)
├── skills/
│   └── <skill-name>/
│       └── SKILL.md       ← on-demand domain knowledge
├── hooks/
│   ├── pre-tool.sh        ← PreToolCall handler
│   ├── post-tool.sh       ← PostToolCall handler
│   └── stop.sh            ← Stop handler
├── projects/
│   └── <project-hash>/
│       ├── memory/
│       │   └── MEMORY.md  ← persisted cross-session context
│       └── *.jsonl        ← session transcripts

project/
├── .claude/
│   └── settings.json      ← project-level config (overrides global)
└── CLAUDE.md              ← project-level instructions
```

---

## 🗺️ ROADMAP

| Priority | Feature | Status |
|---|---|---|
| P0 | CLAUDE.md templates | ✅ Done |
| P0 | Settings.json examples | ✅ Done |
| P0 | Hook examples | ✅ Done |
| P1 | MCP server catalog | 🔄 In Progress |
| P1 | Cost tracking scripts | 📅 Planned |
| P2 | Parallel worktree patterns | 📅 Planned |
| P2 | Auto-memory Stop hook | 📅 Planned |
| P3 | Session replay tool | 📅 Planned |

---

## ☠️ STARTUPS / BUSINESSES

| Old Way | What This Replaces | Disruption |
|---|---|---|
| GitHub Copilot | Claude Code does whole-file edits, multi-file refactors, test gen | 🔥 High |
| Cursor / Windsurf | Claude Code runs in terminal, zero IDE lock-in | 🔥 High |
| Manual code review | Hooks + sub-agents auto-review every PR diff | 💀 Total |
| Onboarding docs | CLAUDE.md = living codebase onboarding document | 🔥 High |
| Jira + Confluence | Task decomposition + memory files replace ticket tracking | 🔥 High |
| Custom automation scripts | Hooks handle pre/post logic without separate tooling | 💀 Total |

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/claude-code&type=Date)](https://star-history.com/#hmzainjamil/claude-code&Date)

---

Built by [HMZ](https://github.com/hmzainjamil)

---

## CLAUDE.md Template

```markdown
## PROJECT CONTEXT
Repo: <name> | Stack: <tech> | Owner: <team>

## ALWAYS-ON RULES
- Never use `rm -rf` without confirmation
- Run tests before committing: `npm test`
- Use conventional commits: feat/fix/docs/refactor
- Branch naming: `feature/<ticket>-<description>`

## MODEL ROUTING
- Simple tasks → `claude-haiku-4-5`
- Code review → `claude-sonnet-4-5`
- Architecture → `claude-opus-4-5`

## BANNED PATTERNS
- No `console.log` in production code
- No hardcoded credentials
- No direct DB queries outside repository layer

## DIRECTORY STRUCTURE
src/
  api/        ← Express routes
  services/   ← Business logic
  models/     ← Database models
  utils/      ← Shared helpers
tests/        ← Jest test files
```

---

## Settings.json Full Example

```json
{
  "model": "claude-sonnet-4-5",
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)",
      "Bash(node *)",
      "Bash(python3 *)",
      "Bash(pytest *)",
      "Bash(docker *)",
      "Bash(gh *)",
      "Read(*)",
      "Edit(*)",
      "Write(*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo rm *)",
      "Bash(curl * | bash)",
      "Bash(wget * | bash)"
    ]
  },
  "env": {
    "NODE_ENV": "development",
    "LOG_LEVEL": "debug"
  },
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/bin/session-summarizer.py"
          }
        ]
      }
    ]
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me"]
    }
  },
  "includeCoAuthoredBy": true,
  "cleanupPeriodDays": 30
}
```

---

## Common CLAUDE.md Patterns

### Monorepo navigation

```markdown
## MONOREPO STRUCTURE
packages/
  api/       ← NestJS backend (port 3001)
  web/       ← Next.js frontend (port 3000)
  shared/    ← Shared types + utils

## RUNNING
- All: `pnpm dev` from root
- API only: `pnpm --filter api dev`
- Tests: `pnpm test` (runs all packages)

## IMPORTANT
- All shared types go in packages/shared
- Never import api code from web or vice versa
- Use workspace protocol: `"@app/shared": "workspace:*"`
```

### Agent task decomposition

```markdown
## TASK EXECUTION
Every complex task → decompose first:
1. List sub-tasks as bullet points
2. Identify which can run in parallel
3. Route parallel tasks to sub-agents
4. Synthesize results

## MODEL BUDGET
- Research/gather: Haiku ($0.00025/1k)
- Code/edit: Sonnet ($0.003/1k)
- Arch/design: Opus ($0.015/1k)
- Never use Opus for routine tasks
```

---

## Session Management Commands

```bash
# List all sessions
ls ~/.claude/projects/

# View latest session transcript
ls -t ~/.claude/projects/-Users-me-myproject/*.jsonl | head -1 | xargs tail -50

# Search across all session transcripts
grep -r "error\|failed\|bug" ~/.claude/projects/ --include="*.jsonl" | head -20

# Check memory files
cat ~/.claude/projects/-Users-me-myproject/memory/MEMORY.md

# Clear old sessions (keep last 30 days)
find ~/.claude/projects/ -name "*.jsonl" -mtime +30 -delete
```

---

## Resources

- [Claude Code Docs](https://docs.anthropic.com/claude/docs/claude-code)
- [MCP Specification](https://modelcontextprotocol.io)
- [Anthropic API Reference](https://docs.anthropic.com/claude/reference)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
- [awesome-claude-code](https://github.com/hmzainjamil/awesome-claude-code)
- [awesome-claude-skills](https://github.com/hmzainjamil/awesome-claude-skills)
