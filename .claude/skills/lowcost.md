---
name: lowcost
description: Coordinate multi-model coding workflow using capable advisor and cost-efficient workers for exploration, implementation, and verification
---

# Lowcost multi-model orchestration

You are the **advisor and coordinator**. Your responsibilities:
- Understand the goal and decide how to divide work
- Choose appropriate models for each phase
- Integrate results from delegates
- Own final verification and quality

## Model selection strategy

**Available models:**
- **OpenCode DeepSeek V4.1 Flash 1017** (`openrouter/deepseek/deepseek-v4.1-flash-1017`)
  - Variants: `none`, `low`, `high`, `max`
  - Use for: exploration, bounded implementation, tests
- **Claude Haiku 5.5** (`claude-haiku-5-5`)
  - Fallback for tasks requiring better instruction-following
- **Claude Opus 5.5** (advisor - you)
  - Planning, integration, architecture, final verification

**Selection heuristic (starting hypothesis, not benchmark result):**
1. **Exploration and search**: DeepSeek `variant: "low"` or `"none"`
2. **Bounded implementation**: DeepSeek `variant: "low"` initially; escalate to `"high"` or Claude Haiku if blocked
3. **Tests and verification**: DeepSeek `variant: "none"` for straightforward tests
4. **Independent review**: Claude Haiku or Sonnet (different model from implementer)
5. **Architecture/unresolved failures**: Handle in advisor session or escalate to stronger model

## Delegation workflow

### When to delegate
- **DO delegate**: Multi-file exploration, substantial feature implementation, independent review
- **DON'T delegate**: Trivial edits, single-file fixes, clarifications, integration (your job)

### How to delegate

```javascript
// Call orchestrator_capabilities once per session
await mcp__t3-code__orchestrator_capabilities()

// Then delegate with focused briefs
await mcp__t3-code__delegate_task({
  task: `[Focused objective]
  
Context: [Essential background]
Files: [Relevant paths]
Return: [Expected format: findings/changes/tests/uncertainties]
Constraints: [Boundaries]`,
  
  target: {
    providerInstanceId: "opencode",
    model: "openrouter/deepseek/deepseek-v4.1-flash-1017",
    options: [{ id: "variant", value: "low" }]
  },
  
  role: "implementation", // or: research, review, test
  isolation: "shared",    // or: "worktree" for parallel edits
  mode: "async"           // or: "wait" if you need result now
})
```

### Context management best practices

**Brief structure** (pass essential context only):
```
## Task
[One clear objective]

## Context
[Domain/architecture summary - 2-3 sentences]
[Relevant constraints]

## Files
[Specific paths to read/modify]

## Return format
- Changes: [file:line references]
- Tests: [command + result]
- Uncertainties: [what needs advisor decision]
```

**Avoid:**
- Forwarding your entire conversation history
- Assuming delegate inherits your loaded skills
- Vague instructions like "fix the codebase"
- Multiple writers on same files without isolation

## Windows compatibility patterns

### PowerShell failures - common issues

**File operations:**
```powershell
# ❌ AVOID: Fails on paths with spaces, special chars
cat C:\Program Files\app\config.json

# ✅ USE: Proper quoting
Get-Content "C:\Program Files\app\config.json"

# ❌ AVOID: Unix commands not in PowerShell
which node
touch file.txt

# ✅ USE: PowerShell equivalents
(Get-Command node).Source
New-Item -ItemType File file.txt -Force:$false
```

**Path handling:**
```powershell
# ❌ AVOID: Forward slashes in native commands
node ./scripts/build.js

# ✅ USE: Backslashes or let PowerShell normalize
node .\scripts\build.js
node (Join-Path "scripts" "build.js")
```

**Shell operations:**
```powershell
# ❌ AVOID: Bash operators
command1 && command2
VAR=value command

# ✅ USE: PowerShell equivalents
command1; if ($?) { command2 }
$env:VAR = 'value'; command
```

### Delegate brief - Windows patterns

When delegating to models that may use shell commands, include this guidance:

```markdown
## Environment
- Platform: Windows (PowerShell)
- Use PowerShell syntax: `Get-Content`, not `cat`
- Quote paths with spaces: `"C:\Program Files\..."`
- Check command exists before use: `Get-Command <name> -ErrorAction SilentlyContinue`
- Prefer Read/Edit/Write tools over shell for file operations
```

## Verification workflow

After coherent changes:
1. **Delegate runs targeted checks** during work (linting, relevant unit tests)
2. **Advisor verifies integration**: read diff, run broader test suite, check architecture alignment
3. **Report failures honestly**: if tests fail, escalate; don't hide issues

**Not required:**
- Full test suite after every single-line edit
- Independent review for trivial changes
- Parallel review when implementation succeeded

## Escalation triggers

**When to escalate from DeepSeek to Claude:**
- Task attempted twice without success
- Architecture decision needed
- Integration across multiple delegates
- Implementation deviates from requirements

**When NOT to escalate:**
- First attempt failed (try variant adjustment first)
- Syntax errors (worker should fix)
- Test failures (worker should debug)

## Example workflows

### Workflow 1: Feature implementation
```
1. [Advisor] Plan approach, identify files
2. [DeepSeek low] Explore existing patterns: `role: "research"`
3. [Advisor] Review findings, decide implementation
4. [DeepSeek low→high] Implement feature: `role: "implementation"`
5. [Advisor] Review diff, run integration tests
6. [Optional] [Claude Haiku] Independent review if high-impact: `role: "review"`
```

### Workflow 2: Bug investigation
```
1. [Advisor] Reproduce issue, form hypothesis
2. [DeepSeek none] Search for relevant code: `role: "research"`
3. [Advisor] Identify root cause
4. [DeepSeek low] Fix + add test: `role: "implementation"`
5. [Advisor] Verify fix resolves original issue
```

### Workflow 3: Codebase exploration
```
1. [DeepSeek none] Broad search with concise return format
2. [Advisor] Synthesize findings, identify gaps
3. [DeepSeek low] Targeted follow-up on specific areas
4. [Advisor] Integrate into coherent understanding
```

## Cost awareness principles

- **Optimize for task completion**, not tokens-per-call (a failed cheap attempt costs more than a working expensive one)
- **Advisor token budget**: Your context is expensive; delegate to preserve it for integration work
- **Worker retries**: Escalate reasoning variant before switching models
- **Parallel work**: Use `isolation: "worktree"` when truly independent; don't duplicate overhead
- **Verification**: Targeted checks during work, broader verification after coherent change

## Limitations and honest defaults

- **Model selection is hypothesis**: No proven benchmark establishes DeepSeek > Haiku or vice versa for all tasks
- **Provider availability**: If OpenCode unavailable, report limitation; don't silently switch
- **Windows shell**: Delegates may attempt Unix commands; brief with Windows patterns for complex tasks
- **Context inheritance**: Delegates start fresh; pass essential context in brief
- **Permissions**: Skill loads for Claude advisor; cross-provider skill loading not guaranteed

## Usage

Invoke this skill when:
- User requests "multi-model workflow", "cost-aware coding", or "delegated implementation"
- Task is substantial (multi-file, exploration + implementation)
- You want to preserve your context for integration work

After invocation:
1. Confirm you understand the goal
2. Explain division of work and model choices
3. Execute with focused delegates
4. Integrate and verify results
5. Report outcome with cost-effectiveness notes
