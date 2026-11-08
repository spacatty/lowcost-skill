# Quick Start Guide

Get started with lowcost orchestration in 5 minutes.

## Prerequisites

1. **T3 Code** installed and running
2. **Claude Opus 5.5** as your advisor model (current session)
3. **OpenCode provider** enabled in T3 settings

## Step 1: Verify setup

In your T3 Code session:

```javascript
// Load T3 orchestration tools
await ToolSearch({ query: "select:mcp__t3-code__orchestrator_capabilities,mcp__t3-code__delegate_task" })

// Check available models
await mcp__t3-code__orchestrator_capabilities()
```

**Expected**: You should see:
- `opencode` provider with `canRunChildTask: true`
- `openrouter/deepseek/deepseek-v4.1-flash-1017` in models list
- Variants: `none`, `low`, `high`, `max`

**If not**: See [Troubleshooting - Provider disabled](docs/troubleshooting.md#issue-delegate-task-fails-with-provider-instance-is-disabled)

## Step 2: Invoke the skill

Type in your T3 Code session:

```
/lowcost

I want to add user authentication to my Express API.
```

**Or naturally:**

```
Use lowcost workflow to implement password reset functionality.
```

## Step 3: Let the advisor coordinate

The skill guides you (Claude Opus advisor) through:

1. **Understanding the goal** - clarify requirements
2. **Planning the approach** - divide work into phases
3. **Delegating to workers** - exploration, implementation, tests
4. **Integrating results** - verify, merge, final checks
5. **Reporting outcome** - what was done, cost notes

You don't need to manually call `delegate_task` unless you want fine control.

## Example run: Add API endpoint

**Your request:**
```
/lowcost

Add a GET /users/:id endpoint that returns user details.
Require authentication.
```

**What happens:**

```
[Advisor - you]
I'll implement this using lowcost workflow:

1. Exploration: Find existing user routes & auth patterns
2. Implementation: Add endpoint with auth middleware
3. Tests: Unit + integration tests
4. Verification: Full diff review + test run

Starting...

[Delegate 1 - DeepSeek low]
Task: Find existing user routes & auth middleware
Result: Found routes in src/routes/users.routes.ts, auth at src/middleware/auth.ts

[Advisor]
Reviewed findings. Auth pattern uses JWT via jwtAuth middleware.

[Delegate 2 - DeepSeek low]
Task: Implement GET /users/:id endpoint with jwtAuth
Files: src/routes/users.routes.ts, src/controllers/users.controller.ts
Result: Implemented getUserById endpoint, added tests
Tests: 8/8 pass

[Advisor]
Verified changes:
- src/routes/users.routes.ts:45 - added route with jwtAuth
- src/controllers/users.controller.ts:78 - getUserById handler
- tests/users.test.ts:120 - integration tests

Ran full test suite: 42/42 pass

✅ Complete. Cost: ~$0.05 (vs ~$0.30 all Opus)
```

## Manual delegation (advanced)

If you want direct control:

```javascript
// 1. Load tools (once per session)
await ToolSearch({ query: "select:mcp__t3-code__orchestrator_capabilities,mcp__t3-code__delegate_task" })

// 2. Check capabilities (once per session)
await mcp__t3-code__orchestrator_capabilities()

// 3. Delegate with focused brief
const result = await mcp__t3-code__delegate_task({
  task: `## Task
Find all API routes in the codebase

## Context
Express REST API
Routes defined in src/routes/*.routes.ts

## Files
- src/routes/**/*.routes.ts (search)

## Return
Concise list:
- File:line
- Endpoint (method + path)
Keep it short`,

  target: {
    providerInstanceId: "opencode",
    model: "openrouter/deepseek/deepseek-v4.1-flash-1017",
    options: [{ id: "variant", value: "low" }]
  },
  
  role: "research",
  isolation: "shared",
  mode: "wait"  // Wait for result
})

// 4. Review result
console.log(result)
```

## Common patterns

### Pattern 1: Exploration → Implementation

```javascript
// Phase 1: Explore (fast, cheap)
const exploration = await mcp__t3-code__delegate_task({
  task: "[Search/find task]",
  target: { 
    providerInstanceId: "opencode",
    model: "openrouter/deepseek/deepseek-v4.1-flash-1017",
    options: [{ id: "variant", value: "none" }]  // Fastest
  },
  role: "research",
  mode: "wait"
})

// Phase 2: Implement (based on findings)
await mcp__t3-code__delegate_task({
  task: `[Implementation task using findings]`,
  target: { 
    providerInstanceId: "opencode",
    model: "openrouter/deepseek/deepseek-v4.1-flash-1017",
    options: [{ id: "variant", value: "low" }]
  },
  role: "implementation",
  mode: "async"  // Run in background
})
```

### Pattern 2: Parallel independent work

```javascript
// Launch both at once (different files)
await mcp__t3-code__delegate_task({
  task: "[Task 1: modify file A]",
  isolation: "worktree",  // Separate checkout
  // ...
})

await mcp__t3-code__delegate_task({
  task: "[Task 2: modify file B]",
  isolation: "worktree",  // Another checkout
  // ...
})

// Both run in parallel, no conflicts
```

### Pattern 3: Escalate on failure

```javascript
// Attempt 1: DeepSeek low
let result = await mcp__t3-code__delegate_task({
  task: "[Complex task]",
  target: { 
    providerInstanceId: "opencode",
    model: "openrouter/deepseek/deepseek-v4.1-flash-1017",
    options: [{ id: "variant", value: "low" }]
  },
  mode: "wait"
})

if (result.failed) {
  // Attempt 2: DeepSeek high
  result = await mcp__t3-code__delegate_task({
    task: "[Same task, refined brief]",
    target: { 
      providerInstanceId: "opencode",
      model: "openrouter/deepseek/deepseek-v4.1-flash-1017",
      options: [{ id: "variant", value: "high" }]  // More reasoning
    },
    mode: "wait"
  })
}

if (result.failed) {
  // Attempt 3: Claude Haiku
  result = await mcp__t3-code__delegate_task({
    task: "[Same task]",
    target: { 
      providerInstanceId: "claudeAgent",
      model: "claude-haiku-5-5"
    },
    mode: "wait"
  })
}

// If still failing: handle in advisor session (you)
```

## Tips for success

### 1. Start simple
Begin with one-phase tasks (just exploration OR just implementation) before complex multi-phase workflows.

### 2. Brief quality matters
Spend time crafting clear briefs. A good brief with DeepSeek low beats a vague brief with Opus.

**Good brief checklist:**
- [ ] One clear objective
- [ ] Essential context only (2-3 sentences)
- [ ] Specific file paths
- [ ] Expected return format
- [ ] Boundaries/constraints

### 3. Verify integration points
After delegates complete, verify:
- Read the diff
- Run tests
- Check files actually changed as expected
- Integration between delegate outputs

### 4. Don't over-delegate
Simple tasks (typo fix, one-line change) are faster in advisor session.

### 5. Windows compat
Include environment context in briefs if delegate might use shell:

```markdown
## Environment
- Platform: Windows (PowerShell)
- Prefer Read/Edit/Write tools over shell
- Use PowerShell syntax if shell needed
```

## Learning resources

- **Skill documentation**: `.claude/skills/lowcost.md` - full reference
- **Best practices**: `docs/best-practices.md` - context management, Windows patterns
- **Real examples**: `examples/workflows.md` - proven workflows with briefs
- **Troubleshooting**: `docs/troubleshooting.md` - common issues & fixes

## Next steps

1. ✅ Try a simple exploration task first
2. ✅ Then a bounded implementation
3. ✅ Review the workflows in `examples/workflows.md`
4. ✅ Iterate: refine briefs based on results
5. ✅ Document your own patterns

## Getting help

- **Check troubleshooting**: `docs/troubleshooting.md`
- **Read examples**: `examples/workflows.md`
- **Test tools**: Call `orchestrator_capabilities` to diagnose
- **Report issues**: [github.com/spacatty/lowcost-skill](https://github.com/spacatty/lowcost-skill/issues)

---

**Ready?** Invoke `/lowcost` and describe your first task!
