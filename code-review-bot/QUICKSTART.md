# Quick Start Guide: Eclipse iceoryx Code Review Bot

Get the bot running in **10 minutes**.

## Prerequisites

- A forked copy of https://github.com/eclipse-iceoryx/iceoryx
- GitHub account with token access
- Linux environment (or macOS with Homebrew)

## Installation (5 minutes)

### 1. Clone the review bot

```bash
cd ~/workspace  # or your preferred location
git clone <THIS-REPO> iceoryx-review-bot
cd iceoryx-review-bot
```

### 2. Run setup

```bash
./setup.sh . ~/your-iceoryx-fork
```

Replace `~/your-iceoryx-fork` with the actual path to your forked iceoryx repository.

**What it does:**
- Copies workflow file to `.github/workflows/`
- Installs bot source and config to `.github/review-bot/`
- Creates setup documentation

### 3. Configure GitHub

**Get a token:**

1. Go to GitHub → Settings → Personal access tokens → Tokens (classic)
2. "Generate new token (classic)"
3. Scopes: `repo`, `pull-requests`, `checks`
4. Copy the token

**Add to your fork:**

1. Go to your forked iceoryx repo
2. Settings → Secrets and variables → Actions
3. "New repository secret"
4. Name: `GITHUB_TOKEN`
5. Paste token

**Enable Actions:**

If Actions are disabled:
1. Actions tab → "I understand my workflows, go ahead and enable them"

## Test (2 minutes)

### Option A: Local Test

```bash
cd ~/your-iceoryx-fork

# Install dependencies
pip3 install pyyaml

# Run bot on current branch
.github/review-bot/test-local.sh .

# View results
cat .review-results-local.json | jq '.summary'
```

### Option B: Test on PR

1. Create a test branch with a small C++ change
2. Push and open a PR
3. Bot runs automatically (check Actions tab)
4. Results appear as:
   - Check run summary
   - Inline comments (for major/critical issues)
   - Artifacts with detailed report

## Understanding Results

### Check Run (Summary)

Shows total count by severity:

```
❌ Code Review: 5 findings
- Critical: 0
- Major: 2
- Minor: 3
```

Click to see details.

### Inline Comments

High-severity issues get comments directly on changed lines:

```
**[MAJOR]** AUTOSAR-CXX14-PTR-001
Standard: AUTOSAR
Confidence: HIGH

Potential null pointer dereference after cast

—
Detected by iceoryx-code-review-agent (Run #42)
```

### Artifacts

Download detailed reports:

- `review-report.md` — Markdown summary
- `review-results.json` — Full findings with metadata
- `.review-results/review-results.sarif` — Code Scanning format

## Customization (5 minutes)

Edit `.github/review-bot/review-config.yaml` to:

**Change merge gate threshold:**

```yaml
review_policy:
  merge_gate_threshold: "critical"  # Fail only on critical
```

**Add custom rule:**

```yaml
rule_mappings:
  - id: "MY-RULE-001"
    tool: "clang-tidy"
    check: "bugprone-branch-clone"
    severity: "major"
    standards: ["project-policy"]
```

**Exclude files:**

```yaml
review_policy:
  exclude_paths:
    - "examples/**"
    - "*_test.cpp"
```

**Disable a checker:**

```yaml
analyzers:
  clang-tidy:
    enabled: false  # Turn off if clang-tidy is too slow
```

After editing, commit and push:

```bash
cd ~/your-iceoryx-fork
git add .github/review-bot/review-config.yaml
git commit -m "chore: customize code review config"
git push
```

Next PR will use new settings.

## What Gets Checked

The bot reviews C++ code for:

✅ **Compiler warnings** — -Wall, -Wextra, -Wconversion, -Wshadow, etc.
✅ **clang-tidy checks** — Modern C++, code style, performance, and safety
✅ **Clang Static Analyzer** — Deep static analysis for logic errors
✅ **MISRA C++ violations** — Via internal approved mappings
✅ **AUTOSAR C++14 compliance** — Safety-critical automotive standards
✅ **Project policies** — iceoryx-specific rules (no malloc, no exceptions, etc.)

## Troubleshooting

### "Bot didn't run on my PR"

Check Actions tab:
1. Is the workflow listed? (Should be `code-review.yml`)
2. Did it run? (Check recent runs)
3. Any error messages in logs?

```bash
# Verify workflow file exists
less ~/your-iceoryx-fork/.github/workflows/code-review.yml
```

### "Token permission denied"

Regenerate token with these scopes:
- `repo` ✓
- `pull-requests` ✓
- `checks` ✓

Update the repository secret with the new token.

### "Build failed"

Check the workflow log for:
- Missing dependencies (installed automatically in workflow)
- iceoryx build issues (run locally to test)

```bash
cd ~/your-iceoryx-fork
cmake -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
cmake --build build
```

### "No comments on PR"

Possible causes:
1. Token permissions insufficient → Regenerate with correct scopes
2. No high-severity findings → Check artifacts for details
3. File not in diff → Only changed files get comments

## Examples

### Example PR Review Flow

```
Developer creates PR with these changes:
  - Adds unsafe cast
  - Ignores return value
  - Uninitialized variable

↓

Bot runs:
  1. Builds iceoryx
  2. Analyzes 3 changed files
  3. Finds: 1 critical (uninit var), 2 major (cast + return value)
  4. Creates issue fingerprints (to avoid duplicates)

↓

Posts to PR:
  Check run: "Code Review: 3 findings (1 critical, 2 major)"
  Inline comments: On each changed line with high issues
  Artifacts: Detailed report as JSON + SARIF

↓

Developer:
  - Reads inline comments
  - Fixes issues
  - Force-pushes
  - Bot re-runs automatically
  - All issues resolved → Check passes ✓
```

### Example Inline Comment

```cpp
// Line 42 in power_controller.cpp
void set_speed(uint32_t rpm) {
  speed = static_cast<uint8_t>(rpm);  // ← Comment here
}
```

**Comment:**

> **[MAJOR]** AUTOSAR-CXX14-CAST-001  
> Standard: AUTOSAR  
> Confidence: HIGH  
>  
> Implicit narrowing conversion: cast from uint32_t to uint8_t loses data.  
> If rpm > 255, value wraps silently. Add range validation before cast.

## Next Steps

1. **Test on real PRs** — Open a feature PR and see results
2. **Collect feedback** — Adjust rules based on team needs
3. **Add to CI gate** — Require bot check to pass before merge
4. **Monitor metrics** — Track precision and false positives
5. **Extend bot** — Add LLM-based explanations, Coverity import, etc.

## Support

**For bot issues:**
- Check workflow logs: Your fork → Actions tab
- Review setup docs: `.github/review-bot/SETUP.md`
- See full docs: `IMPLEMENTATION.md` in this repo

**For iceoryx issues:**
- Visit: https://github.com/eclipse-iceoryx/iceoryx

---

**Now:** Open a PR and run the bot! 🚀
