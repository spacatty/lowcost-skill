<img src="assets/icon.svg" width="64" height="64" alt="">

# skill-lowcost

**Cost-aware multi-model orchestration for T3 Code**

Efficiently coordinate coding workflows using Claude Opus as advisor and DeepSeek V4.1 Flash as cost-effective workers for exploration, implementation, and testing.

---

## 🎯 What is this?

A reusable T3 Code skill that enables you to:

- **Save costs**: Use expensive models (Claude Opus) for planning and integration, cheap models (DeepSeek) for execution
- **Stay in control**: Advisor model (you) makes all architectural decisions
- **Scale efficiently**: Delegate exploration, implementation, and tests to workers
- **Work on Windows**: Built-in PowerShell compatibility patterns
- **Manage context**: Focused briefs preserve advisor context for integration work

**Cost comparison** (typical feature implementation):
- All Opus: ~$0.45
- Cost-aware workflow: ~$0.08
- **Savings: ~82%**

---

## 🚀 Quick Start

### Prerequisites
- T3 Code installed
- Claude Opus 5.5 session (advisor)
- OpenCode provider enabled

### Installation

This repository is already set up as a T3 Code project. The skill is in `.claude/skills/cost-aware.md`.

### Usage

In your Claude Opus T3 Code session:

```
/cost-aware

Add authentication middleware to my Express API
```

**That's it.** The skill guides you through delegation, integration, and verification.

**[Full Quick Start Guide →](QUICKSTART.md)**

---

## 📖 Documentation

| Document | Purpose |
|----------|---------|
| **[QUICKSTART.md](QUICKSTART.md)** | 5-minute setup and first workflow |
| **[CLAUDE.md](CLAUDE.md)** | Complete project documentation |
| **[best-practices.md](docs/best-practices.md)** | Context management, Windows patterns |
| **[workflows.md](examples/workflows.md)** | Real-world examples with briefs |
| **[troubleshooting.md](docs/troubleshooting.md)** | Common issues and fixes |

---

## 🏗️ Architecture

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

**Model selection:**
- **Exploration**: DeepSeek low/none (fast, cheap)
- **Implementation**: DeepSeek low → high (escalate as needed)
- **Review**: Claude Haiku (different model, better instruction-following)
- **Architecture**: Claude Opus (advisor - you)

---

## 📂 Repository Structure

```
skill-lowcost/
├── .claude/
│   └── skills/
│       └── cost-aware.md          # Main skill definition
├── docs/
│   ├── best-practices.md          # Context mgmt, Windows compat
│   └── troubleshooting.md         # Common issues & solutions
├── examples/
│   └── workflows.md               # Real workflow examples
├── CLAUDE.md                      # Full documentation
├── QUICKSTART.md                  # 5-minute getting started
└── README.md                      # This file
```

---

## 🎓 Example Workflow

**Task**: Add password reset to Express API

```
1. [Advisor] Plan approach
   → Token generation, reset endpoint, email service

2. [DeepSeek low] Explore patterns
   → Find existing auth, email setup

3. [Advisor] Review findings, decide implementation

4. [DeepSeek low] Implement User model methods
   → generateResetToken, validateResetToken
   → Tests: 8/8 pass

5. [DeepSeek low] Implement reset endpoint
   → POST /auth/reset-password
   → Tests: 12/12 pass

6. [Advisor] Verify integration
   → Read diff, run full test suite
   → All 28 tests pass

Cost: ~$0.08 (vs ~$0.45 all Opus)
Time: ~8 minutes
```

**[More examples →](examples/workflows.md)**

---

## 💡 Key Concepts

### Context Management
Pass **essential context only** in delegate briefs:
- Clear objective
- Relevant architecture (2-3 sentences)
- Specific file paths
- Expected return format

**Don't pass:**
- Full conversation history
- Unrelated code
- Vague instructions

### Windows Compatibility
Built-in guidance for PowerShell:
- Use `Get-Content`, not `cat`
- Quote paths: `"C:\Program Files\..."`
- Chain commands: `cmd1; if ($?) { cmd2 }`
- Prefer cross-platform tools: `rimraf`, `cross-env`

### Escalation Strategy
```
DeepSeek none → low → high → max
                              ↓
                      Claude Haiku → Sonnet
                                           ↓
                                   Advisor (Opus)
```

Escalate reasoning variant before changing models.

---

## ✅ When to Use

**DO use cost-aware workflow when:**
- ✅ Multi-file exploration needed
- ✅ Substantial feature implementation
- ✅ Independent code review required
- ✅ Context preservation matters (long session)

**DON'T use when:**
- ❌ Trivial single-line edits
- ❌ Clarification questions
- ❌ Simple refactors (do in advisor)

---

## 🔧 Customization

### Add new models

Edit `.claude/skills/cost-aware.md`:

```markdown
## Model selection strategy

**Available models:**
- **[Your New Model]** (`provider/model-id`)
  - Use for: [when to use]
```

### Custom workflows

Add to `examples/workflows.md`:

```markdown
### Workflow: [Name]
[Your proven workflow pattern]
```

---

## 🐛 Troubleshooting

Common issues:

| Issue | Fix |
|-------|-----|
| "Provider instance is disabled" | Enable OpenCode in T3 settings |
| "Model not found" | Call `orchestrator_capabilities({ search: "deepseek" })` |
| Unix commands fail on Windows | Add environment context to brief |
| Delegate returns too much context | Specify concise return format |
| Multiple delegates conflict | Use `isolation: "worktree"` |

**[Full troubleshooting guide →](docs/troubleshooting.md)**

---

## 📊 Cost Analysis

**Typical task costs** (based on real workflows):

| Task Type | All Opus | Cost-Aware | Savings |
|-----------|----------|------------|---------|
| Feature implementation | $0.45 | $0.08 | 82% |
| Bug investigation | $0.25 | $0.05 | 80% |
| Codebase exploration | $0.20 | $0.06 | 70% |
| Parallel implementation | $0.35 | $0.07 | 80% |

**Note**: Costs are estimates. Actual savings depend on:
- Task complexity
- Brief quality (clear brief = fewer retries)
- Escalation frequency
- Verification depth

---

## 🤝 Contributing

This is a working repository. To improve:

1. **Test** the skill in real scenarios
2. **Document** failures in `docs/troubleshooting.md`
3. **Refine** model selection based on results
4. **Add** successful workflows to `examples/workflows.md`
5. **Improve** Windows compatibility patterns

**Pull requests welcome!**

---

## 📜 License

MIT

---

## 🔗 Links

- **[T3 Code](https://t3.codes)** - Multi-agent coding platform
- **[Claude API](https://www.anthropic.com/claude)** - Anthropic's Claude models
- **[DeepSeek](https://www.deepseek.com)** - Cost-efficient coding models
- **[GitHub Repository](https://github.com/spacatty/lowcost-skill)** - This repo

---

## 🎯 Next Steps

1. **Read** [QUICKSTART.md](QUICKSTART.md) for setup
2. **Try** a simple exploration task
3. **Review** [examples/workflows.md](examples/workflows.md)
4. **Iterate** on brief quality
5. **Share** your successful patterns!

---

**Questions?** Check [troubleshooting](docs/troubleshooting.md) or [open an issue](https://github.com/spacatty/lowcost-skill/issues).

Created in [T3 Code](https://t3.codes).
