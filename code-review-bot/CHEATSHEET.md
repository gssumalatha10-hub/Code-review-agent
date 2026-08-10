# Code Review Bot — Quick Reference

## Installation

```bash
./setup.sh . /path/to/iceoryx/fork
# Add GITHUB_TOKEN secret to fork settings
# Open a PR to test
```

## Local Run

```bash
python3 src/review_orchestrator.py \
  --repo /path/to/iceoryx \
  --config review-config.yaml \
  --pr-base main \
  --pr-head HEAD \
  --output results.json

python3 src/github_reporter.py \
  --results results.json \
  --generate-report
```

## Configuration Changes

| Goal | Edit | Example |
|------|------|---------|
| Stricter gate | `review_policy.merge_gate_threshold` | `"critical"` |
| Disable checker | `rule_mappings` | Remove entry |
| Exclude files | `review_policy.exclude_paths` | `"test/**"` |
| Add rule | `rule_mappings` | Add new entry |
| Suppress false positive | `suppressions` | Add pattern |
| Change severity | `rule_mappings[x].severity` | `"minor"` |

## Severity & Actions

| Level | Merge Gate | Comments |
|-------|-----------|----------|
| Critical | ❌ Fail | ✅ Inline |
| Major | ⚠️ Warn | ✅ Inline |
| Minor | ℹ️ Info | Summary |
| Info | ℹ️ Info | Summary |

## File Structure

```
code-review-bot/
├── src/
│   ├── review_orchestrator.py      # Add analyzers here
│   ├── github_reporter.py          # GitHub posting logic
│   └── baseline_manager.py         # Dedup/baseline
├── .github/workflows/
│   └── code-review.yml             # GitHub Actions
├── review-config.yaml              # Rules & policies
├── setup.sh                        # Installation
├── QUICKSTART.md                   # 10-min guide ⭐
├── README.md                       # Full docs
├── IMPLEMENTATION.md               # Technical
└── INDEX.md                        # Overview
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot doesn't run | Check Actions enabled + GITHUB_TOKEN set |
| No comments | Verify token has `repo`, `pull-requests`, `checks` scopes |
| Build fails | Run `cmake -B build . && cmake --build build` locally |
| Too slow | Edit `review-config.yaml`, disable slow checks |
| Too noisy | Add rule to `suppressions` section |

## GitHub API Calls

| Operation | Code | Notes |
|-----------|------|-------|
| Post check | `repo.create_check_run()` | With annotations |
| Post comment | `pull.create_review_comment()` | On specific line |
| Get PR files | `pull.get_files()` | To validate paths |
| Upload SARIF | GitHub Code Scanning | Via Actions |

## Environment Variables

```bash
GITHUB_TOKEN          # GitHub API authentication
GITHUB_REPOSITORY     # "owner/repo" format
GITHUB_PR_NUMBER      # Pull request number
GITHUB_RUN_ID         # Workflow run ID
GITHUB_ACTIONS        # "true" when in Actions
```

## Analyzers

| Tool | Command | Output | Notes |
|------|---------|--------|-------|
| Compiler | `clang++ ... 2>&1` | Warnings | Builtin |
| clang-tidy | `clang-tidy -p build` | JSON | v18+ required |
| Clang Analyzer | `scan-build` | Plist/JSON | Deep analysis |
| Coverity | API/JSON import | Custom | (Future feature) |

## SARIF Structure

```json
{
  "version": "2.1.0",
  "$schema": "...",
  "runs": [
    {
      "tool": {"driver": {"name": "iceoryx-code-review-agent"}},
      "results": [
        {
          "ruleId": "RULE-001",
          "message": {"text": "Issue description"},
          "level": "warning",
          "locations": [{"physicalLocation": {"artifactLocation": {"uri": "file.cpp"}}}]
        }
      ]
    }
  ]
}
```

## Adding a New Analyzer

1. **Implement** in `review_orchestrator.py`:
   ```python
   def _run_my_analyzer(self):
       # Run tool, parse output
       # Create Finding objects
       self.findings.extend(...)
   ```

2. **Enable** in `_run_analyzers()`:
   ```python
   if self.config['analyzers']['my_tool']['enabled']:
       self._run_my_analyzer()
   ```

3. **Configure** in `review-config.yaml`:
   ```yaml
   analyzers:
     my_tool:
       enabled: true
       version: "1.0"
   ```

## Adding a Custom Rule

```yaml
rule_mappings:
  - id: "CUSTOM-001"
    tool: "clang-tidy"
    check: "check-name"
    severity: "major"
    standards: ["project-policy"]
    description: "What this checks"
    remediation: "How to fix it"
```

## Baseline Operations

```bash
# View baseline
cat .review-baseline.json

# Reset baseline (start fresh)
rm .review-baseline.json

# Check stats
python3 src/baseline_manager.py --results results.json --baseline .review-baseline.json --output filtered.json
```

## Common Config Edits

**Allow all issues (for learning):**
```yaml
review_policy:
  merge_gate_threshold: "info"  # Don't fail on anything
```

**Fail on any new issue:**
```yaml
review_policy:
  merge_gate_threshold: "minor"  # Even minor issues fail
```

**Focus on safety only:**
```yaml
rule_mappings:
  # Keep only AUTOSAR/MISRA mapped rules
  # Remove performance/style checks
```

**Skip examples:**
```yaml
review_policy:
  exclude_paths:
    - "iceoryx_examples/**"
    - "doc/**"
```

## Performance Tips

- **Slow?** Disable clang-analyzer: `enabled: false`
- **Too many files?** Limit PR size or use parallelization
- **OOM?** Reduce build parallelism: `-j2` instead of `-j$(nproc)`
- **Timeout?** Increase timeout in `.github/workflows/code-review.yml`

## Security

- **Token:** Never commit `GITHUB_TOKEN`; use repository secrets
- **Runs as:** bot user (separate identity)
- **Permissions:** Limited to repo, PRs, checks only
- **Secrets:** Not exposed in workflow logs

---

**Print this page for your desk!** 📋
