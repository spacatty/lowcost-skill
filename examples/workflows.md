# Real-world workflow examples

These are proven delegation patterns from actual use cases.

## Workflow 1: Feature implementation (Express API endpoint)

**Goal**: Add password reset functionality to Express REST API

### Steps

```
1. [Advisor] Plan approach
   - Token generation in User model
   - Reset endpoint in routes
   - Email service integration
   - Tests

2. [DeepSeek low] Explore existing patterns
   Task: "Find existing User model methods and email service usage"
   Result: Located bcrypt patterns, nodemailer setup
   
3. [Advisor] Review findings, decide:
   - Use crypto.randomBytes for token
   - Expire tokens after 1 hour
   - Hash token before storage
   - Follow existing email template pattern

4. [DeepSeek low] Implement User model methods
   Task:
   ## Task
   Add password reset token methods to User model
   
   ## Context
   User model: src/models/User.ts (Mongoose schema)
   Use crypto.randomBytes(32), hash with bcrypt rounds 10
   Token expires in 1 hour (store expiry timestamp)
   
   ## Files
   - src/models/User.ts (add: generateResetToken, validateResetToken)
   - src/models/User.test.ts (add tests for token lifecycle)
   
   ## Return
   - Changes: method signatures + file:line
   - Tests: npm test User.test.ts output
   - Uncertainties: token storage field names if unclear
   
   Result: Implemented successfully, 8/8 tests pass

5. [DeepSeek low] Implement reset endpoint
   Task:
   ## Task
   Create password reset endpoint
   
   ## Context
   POST /auth/reset-password (email) generates token
   POST /auth/confirm-reset (token, newPassword) validates & resets
   Use User methods: generateResetToken, validateResetToken
   Send email via src/services/email.service.ts
   
   ## Files
   - src/routes/auth.routes.ts (add endpoints)
   - src/controllers/auth.controller.ts (add resetPassword, confirmReset)
   - src/routes/auth.routes.test.ts (integration tests)
   
   ## Return
   - Changes: file:line refs
   - Tests: npm test auth.routes.test.ts
   - Uncertainties: rate limiting on reset endpoint?
   
   Result: Implemented, 12/12 tests pass, flagged rate limiting question

6. [Advisor] Verify integration
   - Read full diff across 5 files
   - Run npm test (all suites)
   - Check email template renders correctly
   - Decision: Add rate limiting (5 requests/hour per IP)

7. [DeepSeek none] Add rate limiting
   Task:
   ## Task
   Add rate limiting to /auth/reset-password endpoint
   
   ## Context
   Use express-rate-limit (already in package.json)
   Limit: 5 requests per hour per IP
   Apply only to POST /auth/reset-password
   
   ## Files
   - src/middleware/rateLimit.ts (create resetPasswordLimiter)
   - src/routes/auth.routes.ts (apply middleware)
   
   ## Return
   - Changes: file:line
   - Tests: curl test showing 429 after 5 requests
   
   Result: Implemented, verified with curl

8. [Advisor] Final verification
   - All tests pass (28/28)
   - Manual test: full reset flow works
   - Security check: tokens hashed, single-use, expire
   - Ready to commit
```

**Cost notes:**
- 7 delegate calls (DeepSeek): ~$0.05 total
- Advisor context: ~80K tokens (integration, verification)
- Total: ~$0.08 vs ~$0.45 if all Opus

---

## Workflow 2: Bug investigation and fix

**Goal**: Fix intermittent database connection timeout in production

### Steps

```
1. [Advisor] Reproduce issue
   - Check prod logs: "MongooseError: buffering timed out after 10000ms"
   - Hypothesis: connection pool exhaustion
   - Need: find all places opening connections

2. [DeepSeek none] Search codebase
   Task:
   ## Task
   Find all database connection points
   
   ## Context
   Mongoose used for MongoDB
   Looking for: mongoose.connect, mongoose.createConnection
   Also: any place calling .lean(), .cursor() that might hold connections
   
   ## Files
   - src/**/*.ts (search)
   
   ## Return
   - List of files with connection code
   - Line numbers
   - Brief context (what the connection does)
   Keep it concise
   
   Result: Found 3 connection points, 2 cursors not properly closed

3. [Advisor] Analyze findings
   - Main connection: src/db/connection.ts (correct poolSize: 10)
   - Cursor 1: src/services/export.service.ts (not closed!)
   - Cursor 2: src/jobs/sync.job.ts (not closed!)
   - Root cause: cursors holding connections indefinitely

4. [DeepSeek low] Fix export service
   Task:
   ## Task
   Fix cursor connection leak in export service
   
   ## Context
   File: src/services/export.service.ts:45
   Cursor used for streaming large exports, never closed
   Wrap in try/finally to ensure cursor.close()
   
   ## Files
   - src/services/export.service.ts (fix exportUsers method)
   - src/services/export.service.test.ts (add test verifying close)
   
   ## Return
   - Changes: file:line
   - Tests: npm test export.service.test.ts
   
   Result: Fixed, added test, 6/6 pass

5. [DeepSeek low] Fix sync job
   Task:
   ## Task
   Fix cursor connection leak in sync job
   
   ## Context
   File: src/jobs/sync.job.ts:78
   Cursor used in cron job every hour, accumulating unclosed connections
   Wrap in try/finally to ensure cursor.close()
   
   ## Files
   - src/jobs/sync.job.ts (fix syncUsers method)
   - src/jobs/sync.job.test.ts (add test verifying close)
   
   ## Return
   - Changes: file:line
   - Tests: npm test sync.job.test.ts
   
   Result: Fixed, added test, 4/4 pass

6. [Advisor] Verify fix
   - Read diffs: both cursors now in try/finally
   - Run full test suite: 42/42 pass
   - Deploy to staging
   - Monitor: no timeouts after 6 hours (previous: timeout within 1 hour)
   - Fixed!
```

**Cost notes:**
- 3 delegate calls: ~$0.03
- Advisor: ~40K tokens
- Total: ~$0.05 vs ~$0.25 all Opus

---

## Workflow 3: Codebase exploration (new developer onboarding)

**Goal**: Understand authentication & authorization architecture

### Steps

```
1. [Advisor] Frame exploration
   - Need: auth flow, session management, role-based access
   - Focus: how requests authenticate, how permissions check

2. [DeepSeek none] Find auth entry points
   Task:
   ## Task
   Find authentication middleware and entry points
   
   ## Context
   Express API, looking for middleware that validates requests
   Likely in src/middleware/* or src/auth/*
   
   ## Files
   - src/middleware/**/*.ts (search)
   - src/auth/**/*.ts (search)
   
   ## Return
   Concise list:
   - Middleware name
   - File:line
   - What it checks (JWT? session? API key?)
   
   Result: Found 3 middleware:
   - jwtAuth: src/middleware/auth.ts:15 (validates JWT from header)
   - apiKeyAuth: src/middleware/auth.ts:45 (validates API key)
   - sessionAuth: src/middleware/auth.ts:78 (validates Express session)

3. [DeepSeek low] Trace JWT flow
   Task:
   ## Task
   Trace JWT authentication flow
   
   ## Context
   JWT middleware at src/middleware/auth.ts:15
   Need to understand:
   - Where JWTs are generated
   - What's in the payload
   - How they're validated
   - Where user object is attached to request
   
   ## Files
   - src/middleware/auth.ts (read jwtAuth)
   - Search for: jwt.sign, jwt.verify
   
   ## Return
   Flow description:
   1. JWT generated at [where]
   2. Payload contains [what]
   3. Validation checks [what]
   4. User attached at [where]
   Include file:line references
   
   Result: Complete flow documented, payload includes userId + role

4. [DeepSeek low] Find authorization checks
   Task:
   ## Task
   Find role-based access control implementation
   
   ## Context
   JWT payload includes role (admin/user/guest)
   Looking for: where roles are checked, what each role can do
   
   ## Files
   - src/middleware/**/*.ts (search for role checks)
   - src/routes/**/*.ts (search for permission checks)
   
   ## Return
   - List of protected routes
   - Role requirements per route
   - Authorization middleware used
   Keep it concise: table format if possible
   
   Result: Table of 15 routes with role requirements

5. [Advisor] Synthesize understanding
   - Auth flow: login generates JWT → jwtAuth validates → user attached
   - Roles: admin (full), user (own resources), guest (read-only)
   - Protected routes: 15 endpoints, 8 admin-only, 5 user+, 2 public
   - Session fallback: web UI uses sessions, API uses JWT
   - Complete mental model established
```

**Cost notes:**
- 3 delegate calls: ~$0.04
- Advisor: ~30K tokens (synthesis)
- Total: ~$0.06 vs ~$0.20 all Opus

---

## Workflow 4: Parallel implementation (independent features)

**Goal**: Add two independent features: CSV export and audit logging

### Steps

```
1. [Advisor] Plan
   - Feature 1: CSV export for user data
   - Feature 2: Audit log for admin actions
   - Independent: no shared files, can parallelize

2. [Parallel] Launch both delegates

   [DeepSeek low - Worker 1] CSV export
   Task:
   ## Task
   Implement CSV export for user data
   
   ## Context
   Endpoint: GET /users/export (admin only)
   Stream large datasets, don't load all in memory
   Columns: id, email, name, role, createdAt
   
   ## Files
   - src/routes/users.routes.ts (add endpoint)
   - src/services/export.service.ts (create)
   - src/services/export.service.test.ts (create tests)
   
   ## Return
   - Changes: file:line
   - Tests: npm test export.service.test.ts
   
   Constraints:
   - Use fast-csv library (already in package.json)
   - Require admin role
   - Stream, don't buffer
   
   Isolation: worktree (separate checkout)
   
   
   [DeepSeek low - Worker 2] Audit logging
   Task:
   ## Task
   Implement audit logging for admin actions
   
   ## Context
   Log all admin actions: who, what, when, details
   Store in audit_logs collection (Mongoose)
   Actions: user create/update/delete, role changes
   
   ## Files
   - src/models/AuditLog.ts (create schema)
   - src/middleware/audit.middleware.ts (create)
   - src/routes/admin.routes.ts (apply middleware)
   - src/models/AuditLog.test.ts (tests)
   
   ## Return
   - Changes: file:line
   - Tests: npm test AuditLog.test.ts
   
   Constraints:
   - Store: userId, action, targetId, details, timestamp
   - Middleware: auditLog(action) → apply to routes
   
   Isolation: worktree (separate checkout)

3. [Both complete] Worker 1: 10/10 tests pass, Worker 2: 8/8 tests pass

4. [Advisor] Integrate both
   - Review CSV export diff: looks good, streaming properly
   - Review audit log diff: looks good, all actions covered
   - Merge both worktrees to main checkout
   - Run full integration tests: 60/60 pass
   - No conflicts (independent files)
   - Ready to commit
```

**Cost notes:**
- 2 parallel delegates: ~$0.04 total (wall time: 3 min)
- Sequential would take: 6 min (same cost)
- Advisor: ~50K tokens
- Total: ~$0.07 vs ~$0.35 all Opus

**Parallelization win**: Saved 3 minutes wall time, no extra cost

---

## Workflow 5: Escalation example (worker → advisor)

**Goal**: Implement GraphQL schema for existing REST API

### Steps

```
1. [Advisor] Initial plan
   - Convert REST endpoints to GraphQL
   - Use existing models & controllers
   - Start with simple CRUD, then relationships

2. [DeepSeek low] First attempt
   Task:
   ## Task
   Create GraphQL schema for User type
   
   ## Context
   User model: src/models/User.ts
   Fields: id, email, name, role, posts (relation to Post)
   Set up basic type definition
   
   ## Files
   - src/graphql/schema.graphql (create)
   - src/graphql/resolvers/user.resolver.ts (create)
   
   ## Return
   - Schema definition
   - Basic resolvers (Query.user, Query.users)
   - File:line references
   
   Result: Created schema, but uncertain about relationship resolution strategy

3. [DeepSeek high] Second attempt (escalated reasoning)
   Task: [Same as above, with added context]
   
   ## Context (added)
   - User hasMany Posts (userId foreign key in Post)
   - Need to decide: DataLoader pattern or N+1 queries?
   - Existing REST uses eager loading via Mongoose populate
   
   Result: Implemented with DataLoader, but flagged architectural question:
   "Should GraphQL use same database layer as REST, or separate?"

4. [Advisor] Architectural decision (escalated to advisor)
   - Question: shared data layer vs separate?
   - Decision: Separate GraphQL data layer
   - Rationale: REST uses Mongoose methods, GraphQL needs flexible field resolution
   - Create src/graphql/dataSources/* for GraphQL-specific data fetching
   - Reuse models, but independent query logic

5. [DeepSeek high] Implement data layer
   Task:
   ## Task
   Create GraphQL data source layer
   
   ## Context
   Decision: GraphQL-specific data layer in src/graphql/dataSources/
   Use DataLoader for relationship batching
   Reuse Mongoose models, but write custom queries
   
   ## Files
   - src/graphql/dataSources/UserDataSource.ts (create)
   - src/graphql/dataSources/PostDataSource.ts (create)
   - src/graphql/resolvers/user.resolver.ts (update to use data sources)
   
   ## Return
   - Implementation
   - Tests: npm test graphql/**/*.test.ts
   
   Result: Implemented successfully, 15/15 tests pass

6. [Advisor] Final verification
   - Architecture: clean separation between REST & GraphQL
   - DataLoader: batching works, no N+1 queries
   - Tests: all pass
   - Manual test: GraphQL Playground queries work
   - Success!
```

**Cost notes:**
- 3 delegate calls (2 low, 1 high): ~$0.06
- Advisor: ~60K tokens (architectural decision)
- Total: ~$0.10 vs ~$0.50 all Opus

**Escalation trigger**: Worker flagged architectural uncertainty, correctly escalated to advisor

---

## Workflow 6: Windows-specific (PowerShell commands)

**Goal**: Add build scripts for Windows development environment

### Steps

```
1. [Advisor] Plan
   - Need: build, dev, test, clean scripts
   - Windows-first: PowerShell syntax, path handling
   - Package.json scripts should work cross-platform where possible

2. [DeepSeek low] Implement build scripts
   Task:
   ## Task
   Create build scripts in package.json
   
   ## Context
   TypeScript project, compile src/ to dist/
   Use tsc for build
   Add: build, dev (watch mode), clean
   
   ## Files
   - package.json (add scripts section)
   - .gitignore (ensure dist/ ignored)
   
   ## Return
   - Script definitions
   - Test each: npm run build, npm run dev (Ctrl+C after 5s), npm run clean
   
   ## Environment
   - Platform: Windows (PowerShell)
   - Prefer cross-platform commands where possible
   - If PowerShell-specific needed:
     - Use PowerShell syntax: Remove-Item, not rm
     - Quote paths with spaces: "C:\Program Files\..."
   
   Result: Implemented, but used rm -rf in clean script (Unix command!)

3. [Advisor] Catch Windows incompatibility
   - Issue: "rm -rf dist" fails on Windows in PowerShell
   - Need: cross-platform solution or PowerShell alternative

4. [DeepSeek low] Fix clean script
   Task:
   ## Task
   Fix clean script for Windows compatibility
   
   ## Context
   Current: "clean": "rm -rf dist" (fails on Windows)
   Options:
   1. Use rimraf package (cross-platform rm -rf)
   2. Use PowerShell: "Remove-Item -Recurse -Force dist"
   
   Decision: Use rimraf (already in devDependencies)
   
   ## Files
   - package.json (update clean script)
   
   ## Return
   - Updated script
   - Test: npm run clean (verify dist removed)
   
   Result: Changed to "rimraf dist", tested successfully

5. [Advisor] Verify all scripts
   - npm run build: compiles to dist/ ✓
   - npm run dev: watches for changes ✓
   - npm run clean: removes dist/ on Windows ✓
   - All cross-platform now
```

**Cost notes:**
- 2 delegate calls: ~$0.03
- Advisor: ~25K tokens (Windows compat check)
- Total: ~$0.04

**Windows lesson**: Brief should include environment context for shell commands

---

## Common brief patterns

### Exploration brief
```markdown
## Task
[Find/search/locate X]

## Context
[Where to look, what X looks like]

## Files
[Directories to search]

## Return
Concise list:
- File:line
- Brief description
Keep it short
```

### Implementation brief
```markdown
## Task
[Implement/add/create X]

## Context
[Architecture, patterns, constraints]

## Files
[Specific files to create/modify]

## Return
- Changes: file:line refs
- Tests: command + results
- Uncertainties: [what to escalate]

## Constraints
[Boundaries, patterns to follow]
```

### Review brief
```markdown
## Task
Review [PR/diff/changes] for [focus area]

## Context
[What changed, why, scope]

## Files
[Paths to review]

## Return
- Issues found: severity + file:line + description
- Suggestions: optional improvements
- Verdict: approve / request changes

## Focus
[Correctness / security / performance / style]
```

## Lessons learned

1. **Start cheap, escalate as needed**: DeepSeek low → high → Claude Haiku is usually sufficient
2. **Briefs matter more than model choice**: Clear brief + weaker model > vague brief + strong model
3. **Windows compat is easy to miss**: Include environment context when delegates might use shell
4. **Architectural decisions stay with advisor**: Don't delegate design choices
5. **Parallel delegates save wall time**: Same cost, faster completion for independent work
6. **Escalation is normal**: Workers should flag uncertainties, not guess
7. **Verification at checkpoints**: After coherent changes, not every edit
8. **Context is expensive**: Preserve advisor context for integration, delegate bulk work
