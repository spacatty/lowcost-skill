# Troubleshooting

Common issues when using the cost-aware orchestration skill.

## Delegation issues

### Issue: Delegate task fails with "Provider instance is disabled"

**Symptom:**
```
Error: Provider instance is disabled
Provider: opencode
```

**Cause:** OpenCode provider not enabled in T3 settings

**Fix:**
```powershell
# Check provider status
# In T3 Code UI: Settings > Providers > OpenCode > Enable
```

Or call `orchestrator_capabilities` to verify:
```javascript
await mcp__t3-code__orchestrator_capabilities()
// Check "constraints" field for each provider
```

**Workaround:** Use Claude Haiku as worker until OpenCode enabled

---

### Issue: "Model not found" error

**Symptom:**
```
Error: Model not found
Model: openrouter/deepseek/deepseek-v4.1-flash-1017
```

**Cause:** Model ID changed or removed from provider

**Fix:**
```javascript
// Search for available DeepSeek models
await mcp__t3-code__orchestrator_capabilities({ search: "deepseek" })
// Use the exact model ID from results
```

**Workaround:** Update skill with correct model ID, or use Claude Haiku

---

### Issue: Delegate times out or hangs

**Symptom:**
- Delegate task shows "running" for >10 minutes
- No output or progress

**Cause:** 
- Brief too vague (delegate doesn't know when done)
- Task too large (should be broken down)
- Actual model hang (rare)

**Fix:**
1. Check task is specific and bounded
2. Break large tasks into smaller steps
3. Cancel hung task:
   ```javascript
   await task_cancel({ taskId: "<id>" })
   ```

**Prevention:** Use focused briefs with clear done-ness criteria

---

### Issue: Delegate returns "I don't have access to that file"

**Symptom:**
Delegate says file doesn't exist, but it does in your checkout

**Cause:** 
- File path incorrect (typo, wrong directory)
- Delegate in different worktree (with `isolation: "worktree"`)
- File created after delegate started

**Fix:**
1. Verify file path is correct (absolute or relative to project root)
2. If using worktree isolation, file must exist in that worktree
3. For new files, create in main checkout first, then delegate

**Prevention:** Pass paths relative to project root, not current directory

---

## Windows compatibility issues

### Issue: Delegate uses Unix commands that fail

**Symptom:**
```
Error: 'rm' is not recognized as an internal or external command
Error: 'cat' is not recognized as an internal or external command
```

**Cause:** Delegate attempted Unix commands on Windows

**Fix in brief:**
```markdown
## Environment
- Platform: Windows (PowerShell)
- Prefer Read/Edit/Write tools over shell for file operations
- If shell needed:
  - Use PowerShell syntax: Get-Content, not cat
  - Use cross-platform tools: rimraf, not rm -rf
```

**Quick fixes:**
```powershell
# Add cross-platform tools to project
npm install --save-dev rimraf cross-env npm-run-all

# Then use in package.json scripts:
"clean": "rimraf dist"
"build": "cross-env NODE_ENV=production tsc"
```

---

### Issue: Path with spaces fails in PowerShell

**Symptom:**
```
Error: 'C:\Program' is not recognized as an internal or external command
```

**Cause:** Path not quoted in PowerShell command

**Fix:**
```powershell
# ❌ WRONG
node C:\Program Files\app\script.js

# ✅ RIGHT
node "C:\Program Files\app\script.js"
```

**In briefs:**
```markdown
## Constraints
- Quote all file paths: "C:\path\to\file"
- Use backslashes or let PowerShell normalize
```

---

### Issue: `&&` operator fails in PowerShell

**Symptom:**
```
Error: The token '&&' is not a valid statement separator in this version.
```

**Cause:** PowerShell 5.1 doesn't support `&&` operator

**Fix:**
```powershell
# ❌ WRONG
npm run build && npm run test

# ✅ RIGHT
npm run build; if ($?) { npm run test }

# Or use cross-platform tool
npm install --save-dev npm-run-all
# Then in package.json:
"build-and-test": "run-s build test"
```

---

## Context management issues

### Issue: Delegate "forgets" context from earlier in conversation

**Symptom:**
Delegate asks for information you already provided, or ignores earlier decisions

**Cause:** Delegates start with fresh context; they don't inherit your conversation

**Fix:** Pass essential context in every delegate brief

**Example:**
```markdown
## Context
(Always include, even if you mentioned it before)

Database: MongoDB via Mongoose
User model: src/models/User.ts
Authentication: JWT tokens
Existing pattern: bcrypt for passwords, rounds: 10
```

---

### Issue: Delegate's response is too verbose (wastes tokens)

**Symptom:**
Delegate returns full file contents instead of concise references

**Cause:** Brief didn't specify return format

**Fix in brief:**
```markdown
## Return format
- File:line references only (not full code)
- Concise summary (2-3 sentences)
- List format for multiple items

Example:
- src/routes/user.routes.ts:45 - added auth middleware
- src/controllers/user.controller.ts:78 - updated validation
```

---

### Issue: Multiple delegates conflict (edit same file)

**Symptom:**
```
Error: Git conflict in src/app.ts
```

**Cause:** Two delegates with `isolation: "shared"` edited same file concurrently

**Fix:** Use `isolation: "worktree"` for parallel edits
```javascript
await mcp__t3-code__delegate_task({
  task: "...",
  isolation: "worktree",  // Separate checkout
  // ...
})
```

Or: Run delegates sequentially (mode: "wait" on first, then launch second)

---

## Cost management issues

### Issue: Costs higher than expected

**Symptom:**
- Many delegate calls with minimal work
- Large context passed to each delegate
- Frequent escalations to expensive models

**Diagnosis:**
1. **Briefs too large**: Trim context to essentials
2. **Over-delegation**: Do simple edits in advisor session
3. **Premature escalation**: Try reasoning variants before stronger models

**Fix:**
```markdown
# ❌ TOO LARGE BRIEF (500 lines of context)
## Context
[Entire conversation history, full file dumps, ...]

# ✅ FOCUSED BRIEF (50 lines)
## Context
User model: src/models/User.ts
Task: Add resetToken field (string, optional)
Pattern: Follow existing emailVerificationToken field
```

**Cost optimization checklist:**
- [ ] Brief under 100 lines
- [ ] Only essential file paths
- [ ] Concise return format specified
- [ ] Simple tasks done in advisor
- [ ] Reasoning escalation before model escalation

---

### Issue: Delegate fails twice, advisor does third attempt

**Symptom:**
- DeepSeek low: failed
- DeepSeek high: failed
- Advisor does same task: succeeds

**Diagnosis:** Task needs stronger model OR briefs were insufficient

**Fix:**
1. Read delegate outputs to diagnose failure cause
2. If brief was vague: refine brief, try again
3. If task genuinely hard: escalate to Claude Haiku/Sonnet, don't retry with advisor

**Cost impact:**
- 2 failed + 1 advisor success = 3× cost
- Better: 1 refined brief + DeepSeek high = 0.2× cost
- Best: 1 Claude Haiku on hard task = 0.1× cost (vs failed retries)

---

## Model selection issues

### Issue: Uncertain which model to use for a task

**Symptoms:**
- Task seems borderline between worker/advisor
- Not sure if task is "hard enough" for escalation

**Decision tree:**

```
Is task well-defined and bounded?
├─ YES: Start with DeepSeek low
│   ├─ First attempt fails → try DeepSeek high
│   └─ Second attempt fails → Claude Haiku
└─ NO: Is it an architectural decision?
    ├─ YES: Advisor (don't delegate)
    └─ NO: Clarify scope, then start with DeepSeek low
```

**Examples:**

| Task | Model | Reasoning |
|------|-------|-----------|
| Add CRUD endpoint | DeepSeek low | Bounded, clear pattern |
| Search for all API calls | DeepSeek none | Fast search, low reasoning |
| Fix failing test | DeepSeek low | Debugging, needs reasoning |
| Design auth architecture | Advisor | Architectural decision |
| Review security-critical code | Claude Haiku | Independent review, high stakes |
| Refactor large module | DeepSeek high→Haiku | Escalate if complex |

---

### Issue: Model unavailable (rate limit or downtime)

**Symptom:**
```
Error: Rate limit exceeded
Provider: opencode
```

**Immediate fix:**
```javascript
// Fall back to Claude Haiku
await mcp__t3-code__delegate_task({
  target: {
    providerInstanceId: "claudeAgent",
    model: "claude-haiku-5-5"
  },
  // ... rest of task
})
```

**Prevention:**
- Don't launch many parallel delegates at once (rate limits)
- Stagger delegate launches (wait 1-2s between calls)
- Use `mode: "wait"` for sequential tasks

---

## Verification issues

### Issue: Tests pass in delegate, fail in advisor verification

**Symptom:**
- Delegate reports "10/10 tests pass"
- Advisor runs tests: "2/10 fail"

**Cause:**
- Delegate ran subset of tests
- Test environment difference (worktree vs main)
- Race condition in tests

**Fix:**
1. Verify delegate ran correct test command
2. If worktree isolation: ensure test setup identical
3. Run tests in advisor after merge/integration
4. For race conditions: add proper async handling

**Prevention in briefs:**
```markdown
## Return
- Tests: EXACT command used
- Result: Full output (including pass/fail counts)
- Environment: Any setup required (DB seed, env vars)
```

---

### Issue: Advisor skips verification, ships broken code

**Symptom:**
Changes merged, then prod breaks

**Cause:** Advisor trusted delegate without verification

**Fix:** Always verify at checkpoints:
```markdown
After coherent change (feature/bugfix):
1. Read full diff (advisor)
2. Run relevant test suite (advisor or delegate)
3. Manual spot-check if high-impact
4. Then commit

Not needed:
- After every line edit during development
- For trivial changes (typo fix, comment)
```

**Checklist:**
- [ ] Read diff across all changed files
- [ ] Run tests (at least subset)
- [ ] Check build succeeds
- [ ] Manual test for UI/integration changes
- [ ] Security check for auth/data changes

---

## Skill invocation issues

### Issue: Skill doesn't load when invoked

**Symptom:**
```
/cost-aware
Error: Skill not found
```

**Cause:** Skill file not in correct location

**Fix:**
```powershell
# Verify skill file exists
ls .claude/skills/cost-aware.md

# If missing, create it
mkdir -p .claude/skills
# Then copy skill file to .claude/skills/cost-aware.md
```

**Alternative:** Skill might be in user-level directory
```powershell
ls ~/.claude/skills/cost-aware.md
# Or: C:\Users\<user>\.claude\skills\cost-aware.md on Windows
```

---

### Issue: Skill loads but orchestrator tools not available

**Symptom:**
```
Skill loaded successfully
Error: Tool not found: orchestrator_capabilities
```

**Cause:** T3 Code MCP tools not loaded

**Fix:**
```javascript
// Tools are deferred; load them first
await ToolSearch({ query: "select:mcp__t3-code__orchestrator_capabilities,mcp__t3-code__delegate_task" })

// Then use them
await mcp__t3-code__orchestrator_capabilities()
```

**Or:** T3 Code version too old (update to latest)

---

## Getting help

If issue persists after trying fixes above:

1. **Check T3 Code docs**: [t3.codes/docs](https://t3.codes/docs)
2. **Verify model availability**: Call `orchestrator_capabilities` with no search
3. **Test simple delegation**: Try minimal brief with echo task
4. **Report to skill repo**: [github.com/spacatty/lowcost-skill/issues](https://github.com/spacatty/lowcost-skill/issues)

When reporting:
- Include error message
- Provider & model used
- Brief (or sanitized version)
- Expected vs actual behavior
- T3 Code version
