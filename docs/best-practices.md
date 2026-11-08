# Context management best practices

## Core principle

**Pass the minimum context needed for the delegate to succeed.**

Each delegate starts with a fresh context. Your conversation history, loaded skills, and accumulated understanding are not inherited.

## Brief template

```markdown
## Task
[One clear, actionable objective]

## Context
[2-3 sentences: domain, architecture, constraints]

## Files
[Specific paths to read or modify]
- path/to/file.ts (read: understand existing pattern)
- path/to/new.ts (create: implementation)
- path/to/test.ts (modify: add tests)

## Return format
- Changes: file:line references for review
- Tests: command used + pass/fail results
- Uncertainties: anything needing advisor decision

## Constraints
[Boundaries: don't modify X, use pattern Y, avoid Z]
```

### What to include

✅ **Essential context:**
- Direct task objective
- Relevant architecture/patterns
- File paths to read/modify
- Expected deliverable format
- Hard constraints

❌ **Unnecessary context:**
- Full conversation history
- Tangential implementation details
- Alternative approaches you rejected
- Your reasoning process
- Unrelated code areas

## Example briefs

### Good brief - exploration

```markdown
## Task
Find all API endpoints that interact with the User model

## Context
Express REST API in src/routes/*.routes.ts
User model defined in src/models/User.ts
Looking for routes that create, read, update, or delete users

## Files
- src/routes/**/*.routes.ts (search)
- src/models/User.ts (reference)

## Return format
List of routes with:
- File path
- Endpoint (method + path)
- User model interaction (create/read/update/delete)

Keep it concise: file:line references, not full code dumps
```

### Good brief - implementation

```markdown
## Task
Implement password reset token generation in User model

## Context
User model at src/models/User.ts (Mongoose schema)
Use crypto.randomBytes for token, hash with bcrypt
Token expires in 1 hour
Follow existing method patterns in User model

## Files
- src/models/User.ts (add methods: generateResetToken, validateResetToken)
- src/models/User.test.ts (add tests for new methods)

## Return format
- Changes: method signatures + file:line refs
- Tests: npm test src/models/User.test.ts output
- Uncertainties: token storage approach if unclear

## Constraints
- Don't modify existing User methods
- Use bcrypt rounds: 10 (constant in config)
```

### Bad brief - too vague

```markdown
❌ Fix the authentication system

[No context about what's broken, what "fix" means, or where to look]
```

### Bad brief - too much context

```markdown
❌ Implement password reset

[Followed by 500 lines of conversation history, your entire reasoning process, three alternative approaches you considered, full code of unrelated modules, and a philosophical discussion of security]
```

## Windows-specific context

When delegating tasks that involve shell commands, include:

```markdown
## Environment
- Platform: Windows (PowerShell)
- Prefer Read/Edit/Write tools over shell for file operations
- If shell needed:
  - Use PowerShell syntax: Get-Content, not cat
  - Quote paths with spaces: "C:\Program Files\..."
  - Check commands exist: Get-Command <name> -ErrorAction SilentlyContinue
```

## Multi-delegate coordination

When multiple delegates work on related tasks:

### Parallel delegates (independent work)

```javascript
// Use isolation: "worktree" for parallel file edits
await mcp__t3-code__delegate_task({
  task: "Implement auth middleware in src/middleware/auth.ts",
  isolation: "worktree",  // Separate checkout
  // ...
})

await mcp__t3-code__delegate_task({
  task: "Implement rate limiting in src/middleware/rateLimit.ts",
  isolation: "worktree",  // Another separate checkout
  // ...
})
```

### Sequential delegates (dependent work)

```javascript
// Exploration first
const exploration = await mcp__t3-code__delegate_task({
  task: "Find existing auth patterns",
  mode: "wait",  // Need result before proceeding
  // ...
})

// Then implementation using exploration results
await mcp__t3-code__delegate_task({
  task: `Implement auth using pattern found at ${exploration.pattern_location}`,
  // ...
})
```

## Context preservation strategies

### For the advisor (you)

Your context is the most expensive. Preserve it by:

1. **Delegate bulk work**: Exploration, implementation, test generation
2. **Keep integration**: Reading delegate results, making architectural decisions
3. **Synthesize findings**: Combine multiple delegate outputs into coherent understanding
4. **Verify at checkpoints**: After coherent changes, not every line edit

### For delegates

Delegate context is disposable. Use it for:

1. **Focused execution**: Complete one objective fully
2. **Return concise results**: File refs, not full code dumps
3. **Flag uncertainties**: Escalate decisions to advisor
4. **Self-contained**: Assume no prior conversation

## Anti-patterns

### ❌ Forwarding conversation history

```javascript
// Don't do this
await mcp__t3-code__delegate_task({
  task: `[Paste entire conversation history from the last hour]
  
  Now implement feature X`,
  // ...
})
```

**Why it fails**: Noise obscures signal, wastes tokens, confuses delegate

**Do instead**: Extract essential facts from conversation, pass as structured context

### ❌ Assuming skill inheritance

```javascript
// Don't assume
await mcp__t3-code__delegate_task({
  task: "Use the patterns from /cost-aware skill to implement this",
  // ...
})
```

**Why it fails**: Delegate doesn't have your loaded skills

**Do instead**: Include essential pattern info in brief

### ❌ Vague objectives

```javascript
await mcp__t3-code__delegate_task({
  task: "Improve the codebase",
  // ...
})
```

**Why it fails**: Delegate can't determine done-ness or scope

**Do instead**: Specific, measurable objectives

### ❌ Multiple writers, no isolation

```javascript
// Don't do this
await mcp__t3-code__delegate_task({
  task: "Modify src/app.ts",
  isolation: "shared",  // Same checkout
})

await mcp__t3-code__delegate_task({
  task: "Also modify src/app.ts",
  isolation: "shared",  // Same checkout - CONFLICT!
})
```

**Why it fails**: Concurrent writes to same file without isolation

**Do instead**: Use `isolation: "worktree"` or sequential delegation

## Measuring brief quality

A good brief should enable someone with no prior context to:

1. ✅ Understand what to do
2. ✅ Find relevant files
3. ✅ Know when they're done
4. ✅ Recognize what to escalate

If you can't check all four boxes, refine the brief.

## Iterating on briefs

When a delegate fails or returns unexpected results:

1. **Read the delegate's output** (don't assume)
2. **Identify the gap**: Missing context? Unclear objective? Wrong files?
3. **Refine the brief**: Add the missing piece, clarify the objective
4. **Re-delegate**: With improved brief, possibly higher reasoning variant

**Don't**: Assume delegate is incapable; often the brief was insufficient.

## Resources

- [T3 Code delegation docs](https://t3.codes/docs) (check for updates)
- Skill file: `.claude/skills/cost-aware.md` (delegation templates)
- Examples: `examples/workflows.md` (real briefs that worked)
