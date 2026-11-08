# Contributing to skill-lowcost

Thanks for your interest in improving lowcost orchestration!

## How to Contribute

### 1. Report Issues

Found a bug or limitation? [Open an issue](https://github.com/spacatty/lowcost-skill/issues) with:

- **Clear title**: "Delegate fails on Windows with path spaces"
- **Environment**: T3 Code version, OS, provider
- **Steps to reproduce**: Brief used, model selected
- **Expected vs actual**: What should happen, what actually happened
- **Logs/errors**: Relevant error messages (sanitize sensitive data)

### 2. Share Successful Workflows

Got a workflow that works well? Add it to `examples/workflows.md`:

```markdown
### Workflow X: [Name]

**Goal**: [What you built]

### Steps

\```
1. [Advisor] ...
2. [Worker] ...
...
\```

**Cost notes:**
- X delegate calls: $Y
- Advisor: Z tokens
- Total: $T vs $R all Opus
```

### 3. Improve Documentation

Documentation improvements are always welcome:

- **Clarify confusing sections**: Submit PR with improved wording
- **Add missing examples**: Real-world briefs that worked
- **Fix errors**: Typos, outdated info, broken links
- **Expand troubleshooting**: New issues + solutions you discovered

### 4. Add New Model Support

To add support for a new model:

1. **Verify availability**: Call `orchestrator_capabilities({ search: "model-name" })`
2. **Test with real tasks**: Try exploration, implementation, review
3. **Document results**: Success rate, cost, failure modes
4. **Update skill**: Edit `.claude/skills/lowcost.md`:
   ```markdown
   ## Model selection strategy
   
   **Available models:**
   - **[New Model Name]** (`provider/model-id`)
     - Variants: [list]
     - Use for: [when to use]
     - Cost: [rough cost estimate]
   ```
5. **Add examples**: Workflows using the new model

### 5. Refine Model Selection

Model selection is **hypothesis-based**, not benchmark-proven. Help refine it:

**What to track:**
- Task type (exploration, implementation, review)
- Model used (DeepSeek low/high, Claude Haiku, etc.)
- Success/failure
- Cost
- Escalation needed?

**How to contribute:**
1. Test different models on same task
2. Document results in `examples/workflows.md`
3. Propose updates to selection strategy in `.claude/skills/lowcost.md`

**Note**: Avoid claims like "Model X is always better than Y" without evidence from multiple real tasks.

### 6. Windows Compatibility

Found a Windows-specific issue? Document the fix:

**In `docs/best-practices.md` or `docs/troubleshooting.md`:**

```markdown
### Issue: [PowerShell command fails]

**Symptom:**
\```powershell
Error: ...
\```

**Cause:** [Why it fails]

**Fix:**
\```powershell
# ❌ WRONG
[failing command]

# ✅ RIGHT
[correct command]
\```
```

### 7. Improve Context Management

Better context = better results. Share patterns:

**Add to `docs/best-practices.md`:**

```markdown
### Pattern: [Name]

**When to use:** [Scenario]

**Brief template:**
\```markdown
## Task
[Template]

## Context
[What context to include]
...
\```

**Why it works:** [Explanation]
```

---

## Development Workflow

### Local Testing

1. **Clone repo**:
   ```bash
   git clone https://github.com/spacatty/lowcost-skill.git
   cd skill-lowcost
   ```

2. **Test skill**:
   - Open T3 Code in this directory
   - Invoke `/lowcost` in a Claude Opus session
   - Test with real coding tasks

3. **Iterate**:
   - Edit `.claude/skills/lowcost.md`
   - Test changes
   - Document results

### Documentation Changes

- Edit markdown files directly
- Preview locally (VS Code, GitHub preview, etc.)
- Check links work
- Verify code examples are correct

### Submitting Changes

1. **Fork** the repository
2. **Create branch**: `git checkout -b improve-windows-compat`
3. **Make changes**: Edit relevant files
4. **Test**: Verify skill still works
5. **Commit**: 
   ```bash
   git commit -m "Add PowerShell path quoting examples"
   ```
6. **Push**: `git push origin improve-windows-compat`
7. **Pull request**: Open PR with description of changes

---

## Code of Conduct

### Be Respectful

- **Constructive feedback**: Focus on ideas, not people
- **Assume good intent**: Contributors are trying to help
- **Inclusive language**: Avoid jargon, explain acronyms
- **Patient**: Not everyone has same experience level

### Focus on Quality

- **Test before submitting**: Verify changes work
- **Document thoroughly**: Explain what and why
- **Keep it real**: Don't claim unverified benefits
- **Cite sources**: Link to docs, benchmarks, examples

### Model Selection Ethics

**DO:**
- ✅ Share real results: "Tried DeepSeek low on X task, succeeded"
- ✅ Document failures: "DeepSeek high failed on Y, escalated to Haiku"
- ✅ Compare honestly: "Task A: DeepSeek succeeded, Task B: needed Haiku"

**DON'T:**
- ❌ Claim universal superiority: "DeepSeek is always better than Haiku"
- ❌ Ignore costs: "Just use Opus for everything"
- ❌ Cherry-pick successes: Only report wins, hide failures

**Why**: Model performance varies by task. Honest reporting helps everyone.

---

## Areas Needing Help

### High Priority

1. **Model benchmarking**: Systematic comparison on diverse tasks
2. **Windows testing**: More real-world PowerShell scenarios
3. **Cost tracking**: Utilities to measure actual savings
4. **Brief templates**: Reusable patterns for common tasks

### Medium Priority

5. **Unix compatibility**: Bash-specific guidance
6. **Error handling**: Better escalation triggers
7. **Integration tests**: Automated skill validation
8. **Video tutorials**: Walkthrough of workflows

### Low Priority

9. **GUI tools**: Interactive brief builder
10. **Metrics dashboard**: Cost/success visualizations
11. **Alternative providers**: Support for more model providers

---

## Recognition

Contributors will be:
- Listed in CHANGELOG.md
- Credited in relevant documentation sections
- Thanked in release notes

Significant contributions may earn:
- Co-maintainer status
- Dedicated recognition in README

---

## Questions?

- **General questions**: [Open a discussion](https://github.com/spacatty/lowcost-skill/discussions)
- **Bug reports**: [Open an issue](https://github.com/spacatty/lowcost-skill/issues)
- **Feature requests**: [Open an issue](https://github.com/spacatty/lowcost-skill/issues) with "Feature:" prefix
- **Pull requests**: [Submit a PR](https://github.com/spacatty/lowcost-skill/pulls)

---

## License

By contributing, you agree your contributions will be licensed under the MIT License.

---

**Thank you for helping improve lowcost orchestration!**
