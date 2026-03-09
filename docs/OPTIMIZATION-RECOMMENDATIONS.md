# Cross-Model Agents — Optimization Recommendations

> Generated: 2026-03-09 | Based on audit of SigmaVue monorepo integration

## Current State

- **21 Codex agents** (1,274 lines across TOML files)
- **10 Claude Code agents** (1,126 lines across MD files)
- **8 pipeline scripts** (enforcement system)
- **3 skills** (/codex-review, /council, /delegate)

## Agent Consolidation Opportunities

### Safe to Merge (No Pipeline Impact)

#### 1. `default.toml` + `executor.toml` + `explorer.toml` → `default.toml`
- **Rationale**: All three are general-purpose implementation agents with overlapping scope
- `executor` adds "implement approved plans with minimal diffs" — this is a mode, not a separate agent
- `explorer` is 5 lines using `gpt-5.3-codex-spark` for rapid discovery — add as a profile flag
- **Savings**: 2 fewer agent files, ~44 lines consolidated
- **Risk**: Low — no pipeline gates depend on these agents being separate

#### 2. `claude-reviewer.toml` + `claude-devils-advocate.toml` → `claude-review.toml`
- **Rationale**: Both perform adversarial code analysis from different angles
- Merge into a two-phase review: (1) findings by severity, (2) assumption challenges
- **Savings**: 1 fewer agent file, ~40 lines consolidated
- **Risk**: Low — pipeline gates reference gate names, not agent names

#### 3. Merge 3 stubs into their full counterparts
- `reviewer.toml` (4 lines) → merge into `claude-reviewer.toml` (or the new `claude-review.toml`)
- `security.toml` (4 lines) → merge into `claude-security.toml`
- `tester.toml` (4 lines) → merge into `claude-qa.toml`
- **Savings**: 3 fewer files, 12 lines removed
- **Risk**: Zero — stubs are non-functional as-is

### Do NOT Merge (Pipeline Critical)

| Agent | Reason |
|-------|--------|
| `anti-slop.toml` | Mandatory gate with specific scoring formula |
| `ui-validator.toml` | Mandatory gate with browser testing phases |
| `claude-security.toml` | Domain-specific OWASP methodology |
| `claude-qa.toml` | Domain-specific test strategy |
| `claude-gap-analyst.toml` | 10-dimension gap analysis framework |
| `claude-architect.toml` | Council escalation protocol |
| `planner.toml` | Context gathering + cross-review workflow |
| `council.toml` | Structured debate protocol |
| `claude-frontend-design.toml` | Opus domain strength exploitation |
| `claude-marketing.toml` | Opus domain strength exploitation |

### Net Result
- **Before**: 21 Codex agents + 10 Claude Code agents = 31 total
- **After**: 15 Codex agents + 10 Claude Code agents = 25 total
- **Reduction**: 6 agents removed (19% reduction)

## Model Updates

### Codex Agents
- `explorer.toml` currently uses `gpt-5.3-codex-spark` — update to `gpt-5.4` when merged into `default`
- All other Codex agents should use `gpt-5.4` (latest)

### Claude Code Agents
All Claude Code agents in this project should use the latest models:
- Implementation agents: `claude-opus-4-6`
- Review/gate agents: `claude-sonnet-4-6`
- Agents that delegate to Opus explicitly (frontend-design, marketing): model doesn't matter in the agent file since they invoke `claude -p --model opus`

## Pipeline Script Improvements

### Current Issues
1. Pipeline state file at `/tmp/` is volatile — lost on reboot
2. No way to view pipeline history across sessions
3. Gate results are pass/fail with no trend tracking

### Recommendations
1. **Move state to project dir**: `{repo}/.pipeline-state.json` (gitignored) instead of `/tmp/`
2. **Add gate history**: Append gate results to `{repo}/.pipeline-history.jsonl` for trend analysis
3. **Parallel gate execution**: Currently gates run sequentially; the pipeline-check script could spawn all gate agents simultaneously

## Skill Consolidation

### Current Skills (3)
- `/codex-review` — Cross-model review invocation
- `/council` — Multi-model deliberation
- `/delegate` — Task delegation protocol

### Recommended Addition
- `/pipeline-status` — Quick view of gate status (wraps `pipeline-check.sh`)
- `/pipeline-reset` — Reset pipeline state (wraps `pipeline-reset.sh`)

## Installation/Sync Improvements

The `install.sh` script creates symlinks from `~/.claude/agents/` and `~/.codex/agents/` to this project. Consider:
1. Adding a version check that warns if symlinks point to stale agent files
2. Adding `install.sh` to a post-pull git hook for auto-sync
3. Adding a `verify-install.sh` run as part of session startup

## Integration with SigmaVue Monorepo

The SigmaVue monorepo at `~/SSS Projects/Sigmavue-Mono/` references these agents via symlinks. When updating agents here, changes automatically propagate. Key integration points:

- `~/.claude/agents/codex-*.md` → symlinks to this project's `claude-code/agents/`
- `~/.claude/settings.json` → PostToolUse/PreToolUse hooks reference pipeline scripts
- `~/.githooks/pre-commit` → blocks commits without gate passage

## Action Items

- [ ] Merge `default` + `executor` + `explorer` → `default.toml`
- [ ] Merge `claude-reviewer` + `claude-devils-advocate` → `claude-review.toml`
- [ ] Merge 3 stubs into full counterparts
- [ ] Update agent model versions
- [ ] Move pipeline state from /tmp/ to project dir
- [ ] Add `/pipeline-status` skill
- [ ] Add symlink version check to install.sh
