# skill-lowcost

Cost-aware multi-model orchestration framework for T3 Code.

## Purpose

This repository provides a reusable skill that enables efficient coding workflows by:
- Using Claude Opus 5.5 as advisor/coordinator
- Delegating exploration, implementation, and tests to DeepSeek V4.1 Flash 1017
- Applying independent review selectively with different models
- Managing context efficiently across delegates
- Handling Windows-specific shell compatibility

## Architecture

```
┌─────────────────────────────────────────┐
│ Advisor (Claude Opus 5.5)              │
│ - Planning & architecture               │
│ - Integration & verification            │
│ - Final quality assessment              │
└─────────────────┬───────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
┌───────▼──────┐   ┌────────▼─────────┐
│ Workers      │   │ Reviewers        │
│ (DeepSeek)   │   │ (Claude Haiku+)  │
│              │   │                  │
│ - Exploration│   │ - Independent    │
│ - Implement  │   │   verification   │
│ - Unit tests │   │ - Quality gates  │
└──────────────┘   └──────────────────┘
```

## Model selection strategy

| Phase | Model | Variant | Rationale |
|-------|-------|---------|-----------|
| Exploration | DeepSeek V4.1 Flash | `low` or `none` | Fast search, concise results |
| Implementation | DeepSeek V4.1 Flash | `low` → `high` | Start lean, escalate if needed |
| Unit tests | DeepSeek V4.1 Flash | `none` | Straightforward test generation |
| Review | Claude Haiku 5.5 | default | Different model, instruction-following |
| Architecture | Claude Opus 5.5 | advisor | Complex decisions, integration |

**Important**: These are starting hypotheses, not benchmark-proven rankings.

## Windows compatibility

The skill includes Windows-specific guidance for delegates:
- PowerShell syntax patterns (avoid Unix commands)
- Path handling with spaces and special characters
- Environment variable setting
- Shell operator equivalents

Common issues addressed:
- `cat` → `Get-Content`
- `which` → `(Get-Command ...).Source`
- `touch` → `New-Item -ItemType File`
- `&&` → `; if ($?) { }`
- Path quoting in native commands

## Usage

### Invoking the skill

In a Claude Opus 5.5 T3 Code session:

```
/cost-aware

[Then describe your coding task]
```

Or naturally:

```
I want to implement [feature] using a cost-aware multi-model workflow.
```

### What happens

1. **Advisor** (Claude Opus - you) understands the goal
2. **Calls** `orchestrator_capabilities` to verify available models
3. **Delegates** exploration/implementation with focused briefs:
   - Clear objective
   - Essential context only
   - Specific file paths
   - Expected return format
4. **Integrates** results and verifies quality
5. **Reports** outcome with cost notes

### Example: Adding authentication

```
User: "Add JWT authentication to our Express API using cost-aware workflow"

Advisor:
1. Plans: middleware structure, token validation, route protection
2. Delegates exploration (DeepSeek low):
   - "Find existing auth patterns in codebase"
   - "List API routes that need protection"
3. Reviews findings, decides approach
4. Delegates implementation (DeepSeek low):
   - "Implement JWT middleware in src/auth/jwt.middleware.ts"
   - "Add auth to protected routes"
   - "Write unit tests for middleware"
5. Verifies: reads diff, runs integration tests
6. Optional: Delegates review (Claude Haiku) if high-impact
```

## Repository structure

```
skill-lowcost/
├── .claude/
│   └── skills/
│       └── cost-aware.md          # Main skill definition
├── docs/
│   ├── best-practices.md          # Context mgmt, Windows compat
│   └── troubleshooting.md         # Common issues
├── examples/
│   └── workflows.md               # Real-world examples
└── CLAUDE.md                      # This file
```

## Best practices

### Context management

**Pass essential context only:**
```markdown
## Task
Add user authentication endpoint

## Context
Express REST API, existing user model in src/models/User.ts
Uses bcrypt for passwords, JWT for sessions

## Files
- src/models/User.ts (read existing schema)
- src/routes/auth.routes.ts (create)
- src/middleware/auth.middleware.ts (create)

## Return
- Changes: file:line references
- Tests: npm test command + results
- Uncertainties: any architectural decisions
```

**Don't pass:**
- Your full conversation history
- Unrelated code context
- Vague instructions

### When to delegate

✅ **DO delegate:**
- Multi-file exploration
- Bounded feature implementation
- Independent code review
- Test generation

❌ **DON'T delegate:**
- Trivial single-line edits
- Clarification questions
- Integration (advisor's job)
- Simple refactors

### Escalation

**Escalate from worker to advisor when:**
- Two attempts fail
- Architecture decision needed
- Requirements ambiguity
- Cross-delegate integration

**Escalate reasoning variant before changing models:**
- DeepSeek `none` → `low` → `high` → `max`
- Then consider Claude Haiku/Sonnet

## Limitations

1. **Cross-provider skill loading**: Skill loads for Claude advisor; not guaranteed for OpenCode delegates
2. **Model availability**: Requires OpenCode provider enabled; reports limitation if unavailable
3. **Windows-first**: Optimized for Windows/PowerShell; Unix patterns in delegates may need adjustment
4. **Context reset**: Delegates start fresh; pass essential context in each brief
5. **Hypothesis-based selection**: Model choices are starting points, not benchmark results

## Extending the skill

### Adding new models

Edit `.claude/skills/cost-aware.md` and update the model selection section:

```markdown
## Model selection strategy

**Available models:**
- **[New Model]** (`provider/model-id`)
  - Variants: ...
  - Use for: ...
```

### Custom workflows

Add workflow patterns to `examples/workflows.md`:

```markdown
### Workflow: [Name]
\```
1. [Role] Task description
2. [Role] Task description
...
\```
```

### Platform-specific guidance

Add platform sections to the skill for macOS/Linux if needed:

```markdown
## Environment - macOS/Linux
- Platform: Unix (Bash)
- Standard Unix commands available
- Prefer Read/Edit/Write tools over shell
```

## Validation

The skill has been validated for:
- ✅ T3 Code skill syntax and structure
- ✅ T3 delegation API (`delegate_task`, `orchestrator_capabilities`)
- ✅ Windows PowerShell compatibility patterns
- ✅ Context management best practices
- ✅ Model availability verification
- ✅ Focused brief templates

## Contributing

This is a working repository. To iterate:

1. **Test the skill** in real coding scenarios
2. **Document failures** in `docs/troubleshooting.md`
3. **Refine model selection** based on actual results (not assumptions)
4. **Add workflow examples** from successful runs
5. **Improve Windows compat** as issues surface

## License

Public domain / MIT (choose based on your preference).

## Links

- [T3 Code](https://t3.codes)
- [Claude API](https://www.anthropic.com/claude)
- [DeepSeek](https://www.deepseek.com)
