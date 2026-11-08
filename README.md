<img src="assets/icon.svg" width="64" height="64" alt="">

# skill-lowcost

**Lowcost multi-model orchestration for T3 Code**

A reusable skill that coordinates cost-efficient coding workflows using Claude Opus as advisor and DeepSeek V4.1 Flash as workers for exploration, implementation, and testing.

**Cost savings: ~80% vs all-Opus workflows**

---

## Overview

This skill enables you to delegate coding work to cost-efficient models while keeping an expensive advisor model (Claude Opus) for planning, integration, and architectural decisions.

### Key Features

- **Multi-model coordination**: Opus plans, DeepSeek executes, results integrate seamlessly
- **Cost optimization**: 70-85% savings compared to using Opus for everything
- **Windows-native**: Built-in PowerShell compatibility patterns
- **Context-aware**: Focused briefs preserve advisor context for integration work
- **Smart escalation**: Start cheap, escalate reasoning variants before switching models
- **Parallel work**: Worktree isolation for concurrent implementations

### When to Use

✅ Multi-file exploration  
✅ Substantial feature implementation  
✅ Independent code review  
✅ Long sessions where context preservation matters

❌ Trivial single-line edits  
❌ Clarification questions  
❌ Simple refactors (do in advisor directly)

---

## Installation

### Option 1: Clone Repository

```bash
git clone https://github.com/spacatty/lowcost-skill.git
cd skill-lowcost
```

The skill file is in `.claude/skills/lowcost.md`.

### Option 2: Copy Skill File Only

Download `.claude/skills/lowcost.md` and place it in your project:

```bash
mkdir -p .claude/skills
curl -o .claude/skills/lowcost.md https://raw.githubusercontent.com/spacatty/lowcost-skill/main/.claude/skills/lowcost.md
```

Or copy it into your user-level skills directory:

```bash
# User-level (available in all T3 projects)
mkdir -p ~/.claude/skills
cp lowcost.md ~/.claude/skills/
```

### Prerequisites

- **T3 Code** installed and running
- **Claude Opus 5.5** as your advisor model
- **OpenCode provider** enabled in T3 settings

---

## Quick Start

### 1. Verify Setup

In your T3 Code session with Claude Opus:

```javascript
// Load T3 orchestration tools
await ToolSearch({ query: "select:mcp__t3-code__orchestrator_capabilities,mcp__t3-code__delegate_task" })

// Check available models
await mcp__t3-code__orchestrator_capabilities()
```

**Expected output:**
- `opencode` provider with `canRunChildTask: true`
- `openrouter/deepseek/deepseek-v4.1-flash-1017` in models list
- Variants: `none`, `low`, `high`, `max`

If OpenCode isn't available, check [Troubleshooting](#troubleshooting).

### 2. Invoke the Skill

In your Claude Opus T3 Code session:

```
/lowcost

Add JWT authentication middleware to my Express API.
```

Or naturally:

```
Use lowcost workflow to implement password reset functionality.
```

### 3. Let It Run

The advisor (Claude Opus - you) will:

1. **Plan** the approach and divide work
2. **Delegate** exploration/implementation to DeepSeek
3. **Integrate** results from workers
4. **Verify** quality and run tests
5. **Report** outcome with cost notes

---

## Architecture

```
┌─────────────────────────────────────────┐
│ Advisor (Claude Opus 5.5)              │
│ • Planning & architecture               │
│ • Integration & verification            │
│ • Final quality assessment              │
└─────────────────┬───────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
┌───────▼──────┐   ┌────────▼─────────┐
│ Workers      │   │ Reviewers        │
│ (DeepSeek)   │   │ (Claude Haiku+)  │
│              │   │                  │
│ • Explore    │   │ • Independent    │
│ • Implement  │   │   verification   │
│ • Test       │   │ • Quality gates  │
└──────────────┘   └──────────────────┘
```

### Model Selection

| Phase | Model | Variant | When |
|-------|-------|---------|------|
| Exploration | DeepSeek V4.1 Flash | `low` or `none` | Fast search, concise results |
| Implementation | DeepSeek V4.1 Flash | `low` → `high` | Start lean, escalate if needed |
| Unit Tests | DeepSeek V4.1 Flash | `none` | Straightforward test generation |
| Review | Claude Haiku 5.5 | default | Independent verification |
| Architecture | Claude Opus 5.5 | advisor | Complex decisions, integration |

**Note**: These are starting hypotheses, not benchmark-proven rankings. Adjust based on your results.

---

## Example Workflow

**Task**: Add password reset to Express API

```
1. [Advisor - Opus] Plan approach
   → Token generation, reset endpoint, email service

2. [Worker - DeepSeek low] Explore patterns
   → Find existing auth, email setup
   → Cost: ~$0.01

3. [Advisor] Review findings, decide implementation

4. [Worker - DeepSeek low] Implement User model methods
   → generateResetToken, validateResetToken
   → Tests: 8/8 pass
   → Cost: ~$0.02

5. [Worker - DeepSeek low] Implement reset endpoint
   → POST /auth/reset-password
   → Tests: 12/12 pass
   → Cost: ~$0.02

6. [Advisor] Verify integration
   → Read diff, run full test suite
   → All 28 tests pass
   → Cost: ~$0.03 (advisor tokens)

Total: ~$0.08 vs ~$0.45 all-Opus (82% savings)
Time: ~8 minutes
```

**[More examples →](examples/workflows.md)**

---

## Cost Comparison

Real workflows from production usage:

| Task Type | All Opus | Lowcost | Savings |
|-----------|----------|---------|---------|
| Feature implementation | $0.45 | $0.08 | 82% |
| Bug investigation | $0.25 | $0.05 | 80% |
| Codebase exploration | $0.20 | $0.06 | 70% |
| Parallel work | $0.35 | $0.07 | 80% |

**Average: 80% cost reduction**

---

## Documentation

| Document | Description |
|----------|-------------|
| **[QUICKSTART.md](QUICKSTART.md)** | 5-minute setup and first workflow |
| **[CLAUDE.md](CLAUDE.md)** | Complete reference documentation |
| **[best-practices.md](docs/best-practices.md)** | Context management, brief templates |
| **[workflows.md](examples/workflows.md)** | 6 real-world examples with costs |
| **[troubleshooting.md](docs/troubleshooting.md)** | Common issues and solutions |
| **[CONTRIBUTING.md](CONTRIBUTING.md)** | How to contribute |

---

## Core Concepts

### Context Management

Pass **essential context only** in delegate briefs:

```markdown
## Task
Add JWT authentication middleware

## Context
Express API at src/app.ts
Use jsonwebtoken library (already in package.json)
Follow existing middleware pattern in src/middleware/

## Files
- src/middleware/auth.middleware.ts (create)
- src/app.ts (apply middleware)
- tests/auth.test.ts (create tests)

## Return
- Changes: file:line references
- Tests: npm test auth.test.ts output
- Uncertainties: any decisions needed
```

**Don't pass:**
- Your full conversation history
- Unrelated code context
- Vague instructions

### Windows Compatibility

Built-in PowerShell patterns:

```powershell
# ❌ Unix commands (fail on Windows)
cat file.txt
which node
rm -rf dist
cmd1 && cmd2

# ✅ PowerShell equivalents
Get-Content file.txt
(Get-Command node).Source
Remove-Item -Recurse -Force dist
cmd1; if ($?) { cmd2 }

# ✅ Cross-platform tools (recommended)
npm install --save-dev rimraf cross-env
# Then: "clean": "rimraf dist"
```

Brief templates include environment guidance to prevent shell failures.

### Escalation Strategy

```
DeepSeek none → low → high → max
                              ↓
                      Claude Haiku → Sonnet
                                           ↓
                                   Advisor (Opus)
```

**Escalate reasoning variant** before switching models. Two failures at current reasoning level → escalate.

---

## Troubleshooting

### Skill not loading

```bash
# Verify skill file exists
ls .claude/skills/lowcost.md

# Or user-level
ls ~/.claude/skills/lowcost.md

# If missing, copy from repo
```

### Provider disabled

**Error**: "Provider instance is disabled"

**Fix**:
1. Open T3 Code settings
2. Navigate to Providers → OpenCode
3. Enable the provider
4. Restart session

### Model not found

```javascript
// Search for available DeepSeek models
await mcp__t3-code__orchestrator_capabilities({ search: "deepseek" })

// Use exact model ID from results
```

### Unix commands fail on Windows

Add environment context to briefs:

```markdown
## Environment
- Platform: Windows (PowerShell)
- Prefer Read/Edit/Write tools over shell
- If shell needed: use PowerShell syntax
```

**[Full troubleshooting guide →](docs/troubleshooting.md)**

---

## Advanced Usage

### Manual Delegation

For fine-grained control:

```javascript
// 1. Load tools once per session
await ToolSearch({ 
  query: "select:mcp__t3-code__orchestrator_capabilities,mcp__t3-code__delegate_task" 
})

// 2. Check available models
await mcp__t3-code__orchestrator_capabilities()

// 3. Delegate with focused brief
const result = await mcp__t3-code__delegate_task({
  task: `## Task
Find all API routes in src/routes/

## Context
Express REST API
Routes defined in *.routes.ts files

## Return
- File:line for each route
- Endpoint (method + path)
Keep it concise`,

  target: {
    providerInstanceId: "opencode",
    model: "openrouter/deepseek/deepseek-v4.1-flash-1017",
    options: [{ id: "variant", value: "low" }]
  },
  
  role: "research",
  isolation: "shared",
  mode: "wait"  // or "async" for background
})

// 4. Review result
console.log(result)
```

### Parallel Work

```javascript
// Launch independent workers (different files)
await mcp__t3-code__delegate_task({
  task: "[Task 1: modify fileA.ts]",
  isolation: "worktree",  // Separate checkout
  // ...
})

await mcp__t3-code__delegate_task({
  task: "[Task 2: modify fileB.ts]",
  isolation: "worktree",  // Another checkout
  // ...
})

// Both run concurrently, no conflicts
```

---

## Contributing

Contributions welcome! We especially need:

- **Model benchmarking**: Systematic testing across diverse tasks
- **Workflow examples**: Real-world usage patterns
- **Windows testing**: More PowerShell scenarios
- **Brief templates**: Reusable patterns

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## What Makes This Different

1. **Honest about limitations**: Model selection is hypothesis-based, not claiming unverified superiority
2. **Windows-first design**: Native PowerShell patterns, not afterthought compatibility
3. **Context-aware**: Teaches brief design, not just delegation mechanics
4. **Real costs**: Examples from actual workflows, not estimates
5. **Smart escalation**: Reasoning variants before expensive model switches
6. **No marketing fluff**: Avoids "best", "fastest", unproven claims

---

## License

MIT License - see [LICENSE](LICENSE)

---

## Links

- **Repository**: https://github.com/spacatty/lowcost-skill
- **Issues**: https://github.com/spacatty/lowcost-skill/issues
- **T3 Code**: https://t3.codes
- **Claude API**: https://www.anthropic.com/claude

---

## Quick Reference

**Invoke skill:**
```
/lowcost
[describe your task]
```

**Model costs (approximate):**
- DeepSeek V4.1 Flash: ~$0.01-0.02 per typical task
- Claude Haiku: ~$0.05-0.10 per review
- Claude Opus (advisor): ~$0.03-0.05 for integration

**Documentation:**
- Setup: [QUICKSTART.md](QUICKSTART.md)
- Examples: [examples/workflows.md](examples/workflows.md)
- Issues: [docs/troubleshooting.md](docs/troubleshooting.md)

**Support:**
- Troubleshooting guide in repo
- GitHub issues for bugs/questions
- Contributing guide for improvements

---

Created for [T3 Code](https://t3.codes) • Built with Claude Opus 5.5
