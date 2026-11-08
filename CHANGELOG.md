# Changelog

All notable changes to the lowcost orchestration skill.

## [1.0.0] - 2026-11-08

### Added
- Initial release of lowcost multi-model orchestration skill
- Core skill definition in `.claude/skills/lowcost.md`
- Comprehensive documentation:
  - `CLAUDE.md` - Full project documentation
  - `QUICKSTART.md` - 5-minute getting started guide
  - `docs/best-practices.md` - Context management patterns
  - `docs/troubleshooting.md` - Common issues and solutions
  - `examples/workflows.md` - Real-world workflow examples
- Windows PowerShell compatibility patterns
- Model selection strategy (DeepSeek V4.1 Flash + Claude)
- Context management best practices
- Delegation workflow patterns:
  - Exploration → Implementation
  - Parallel independent work
  - Escalation on failure
- Cost comparison examples

### Model Support
- **Primary worker**: OpenCode DeepSeek V4.1 Flash 1017
  - Variants: `none`, `low`, `high`, `max`
- **Fallback worker**: Claude Haiku 5.5
- **Review model**: Claude Haiku 5.5, Sonnet 5.5
- **Advisor**: Claude Opus 5.5

### Features
- Focused delegate briefs with essential context only
- Windows-specific shell compatibility guidance
- Escalation strategy (reasoning variants → model switch)
- Parallel work with worktree isolation
- Integration verification patterns
- Cost tracking examples (~80% savings vs all-Opus)

### Documentation
- 6 real-world workflow examples with costs
- Common brief patterns (exploration, implementation, review)
- Troubleshooting for 15+ common issues
- Model selection decision tree
- Context management anti-patterns
- Windows compatibility patterns

### Known Limitations
- Requires OpenCode provider enabled
- Cross-provider skill loading not guaranteed
- Delegates start with fresh context (no inheritance)
- Model selection based on hypothesis, not benchmarks
- Windows-optimized (Unix patterns may need adjustment)

---

## Future Improvements (Planned)

### High Priority
- [ ] Add automated skill validation tests
- [ ] Create template briefs for common tasks
- [ ] Add cost tracking utilities
- [ ] Expand model support (other providers)

### Medium Priority
- [ ] Add Unix/macOS-specific guidance
- [ ] Create interactive brief builder
- [ ] Add metrics dashboard (costs, success rates)
- [ ] Expand workflow examples (10+ scenarios)

### Low Priority
- [ ] GUI tool for model selection
- [ ] Automated escalation logic
- [ ] Brief quality analyzer
- [ ] Cost prediction before delegation

---

## Versioning

This project follows [Semantic Versioning](https://semver.org/):
- **MAJOR**: Incompatible skill API changes
- **MINOR**: New functionality (backwards-compatible)
- **PATCH**: Bug fixes (backwards-compatible)

---

## Feedback

Found an issue? Have a suggestion? [Open an issue](https://github.com/spacatty/lowcost-skill/issues) or contribute a pull request.

---

**Current Version**: 1.0.0
**Last Updated**: 2026-11-08
