# Code Review Agent for Eclipse iceoryx

**Autonomous C++ code review bot** for Eclipse iceoryx with MISRA, AUTOSAR, and project-specific standards compliance checking.

## 🎯 Overview

This project implements an **MVP (Minimum Viable Product)** code review bot that:

- ✅ Automatically analyzes C++ pull requests using clang-tidy, clang analyzer, and compiler warnings
- ✅ Maps findings to MISRA C++, AUTOSAR C++14, and project-specific policies
- ✅ Posts results to GitHub as check runs and inline comments
- ✅ Tracks baseline findings and reports only new issues
- ✅ Provides SARIF output for GitHub Code Scanning integration
- ✅ Fully configurable via YAML

## 📂 Project Structure

```
Code-review-agent/
├── code-review-bot/                 # Main bot implementation ⭐
│   ├── src/
│   │   ├── review_orchestrator.py  # Analysis pipeline (550L)
│   │   ├── github_reporter.py      # GitHub integration (350L)
│   │   └── baseline_manager.py     # Deduplication (168L)
│   ├── .github/workflows/
│   │   └── code-review.yml         # GitHub Actions
│   ├── review-config.yaml          # Rule mappings & config
│   ├── setup.sh                    # Installation script
│   ├── QUICKSTART.md               # 10-min setup guide ⭐⭐
│   ├── README.md                   # Full documentation
│   ├── IMPLEMENTATION.md           # Technical guide
│   └── CHEATSHEET.md               # Quick reference
│
├── DELIVERABLES.md                 # What was built (this file's parent)
├── requirement.txt                 # Original requirements
└── README.md                       # This file
```

## 🚀 Quick Start (10 minutes)

### 1. Install the bot

```bash
cd code-review-bot
./setup.sh . /path/to/your/iceoryx/fork
```

### 2. Configure GitHub

1. Go to your fork → Settings → Secrets and variables → Actions
2. Create new secret `GITHUB_TOKEN` with a personal access token (scopes: repo, pull-requests, checks)

### 3. Test

Open a PR on your fork → Bot runs automatically!

**📖 Detailed guide:** [code-review-bot/QUICKSTART.md](code-review-bot/QUICKSTART.md)

## ✨ Key Features

### Analysis

- **Clang Compiler:** -Wall, -Wextra, -Wconversion, -Wshadow, etc.
- **clang-tidy:** 100+ checks for safety, performance, style, modern C++
- **Clang Static Analyzer:** Path-sensitive bug detection
- **Build Integration:** CMake with compile_commands.json

### Standards

- **MISRA C++** ← customizable mappings
- **AUTOSAR C++14** ← safety-critical automotive
- **Project Policies** ← iceoryx-specific rules

### GitHub Integration

- ✅ Check runs with findings summary
- ✅ Inline comments on changed lines (high-severity only)
- ✅ SARIF upload to Code Scanning
- ✅ Markdown reports
- ⚙️ Configurable merge gates

### Smart Filtering

- ✅ Only reports NEW findings (baseline tracking)
- ✅ Deduplicates across multiple tools
- ✅ Marks issues as fixed/suppressed
- ✅ Stable fingerprints to avoid re-reporting

## 📊 What Gets Checked

| Category | Examples |
|----------|----------|
| **Safety** | Null pointers, use-after-free, resource leaks, uninitialized vars |
| **Conversions** | Narrowing casts, implicit conversions, signed/unsigned issues |
| **Memory** | Dangling pointers, lifetime issues, RAII violations |
| **Exceptions** | Uncaught errors, ignored return values |
| **Style** | Naming conventions, const-correctness, shadowing |
| **Modern C++** | Auto, lambdas, smart pointers, ranges |
| **Performance** | Unnecessary copies, inefficient algorithms |

## 🎯 Use Cases

✅ **New feature PRs** — Catch standards violations before review  
✅ **Refactoring** — Ensure safety during code changes  
✅ **Onboarding** — Teach team coding standards via feedback  
✅ **Compliance audit** — Track MISRA/AUTOSAR conformance  
✅ **Quality gate** — Block merge of code with critical issues  

## 📚 Documentation

| Document | Audience | Time | Content |
|----------|----------|------|---------|
| [QUICKSTART.md](code-review-bot/QUICKSTART.md) | Everyone | 10 min | Setup, first run, test |
| [README.md](code-review-bot/README.md) | Users | 30 min | Features, config, examples |
| [CHEATSHEET.md](code-review-bot/CHEATSHEET.md) | Users | 5 min | Quick reference |
| [IMPLEMENTATION.md](code-review-bot/IMPLEMENTATION.md) | Developers | 60 min | Architecture, troubleshooting |
| [DELIVERABLES.md](DELIVERABLES.md) | Leads | 15 min | What was built, status |

**👉 Start here:** [QUICKSTART.md](code-review-bot/QUICKSTART.md)

## 💻 Technical Stack

- **Language:** Python 3.8+
- **Analyzers:** Clang/LLVM 18
- **Build:** CMake
- **Git:** GitHub Actions
- **Config:** YAML
- **Output:** SARIF 2.1.0

## 📋 Implementation Details

### 1,068 Lines of Core Code

```
review_orchestrator.py   550 lines   Build, analyze, normalize findings
github_reporter.py       350 lines   Post to GitHub PR
baseline_manager.py      168 lines   Dedup, baseline filtering
Total Python:           1068 lines
```

### Plus Comprehensive Documentation

```
QUICKSTART.md            5 min guide
README.md                Full docs
IMPLEMENTATION.md        Technical deep-dive
CHEATSHEET.md            Quick reference
Total Docs:             ~2,000 lines
```

## 🔧 Configuration Example

`review-config.yaml` controls everything:

```yaml
# Fail on critical findings
review_policy:
  merge_gate_threshold: "critical"

# Map tool findings to standards
rule_mappings:
  - id: "AUTOSAR-CXX14-PTR-001"
    tool: "clang-analyzer"
    check: "core.NullDereference"
    severity: "critical"
    standards: ["AUTOSAR"]

# Exclude test files
review_policy:
  exclude_paths:
    - "**/*_test.cpp"
    - "examples/**"
```

## 🧪 Testing

### Local Test (2 minutes)

```bash
cd code-review-bot

python3 src/review_orchestrator.py \
  --repo /path/to/iceoryx \
  --config review-config.yaml \
  --pr-base main \
  --pr-head your-branch \
  --output results.json

python3 src/github_reporter.py \
  --results results.json \
  --generate-report \
  --output report.md
```

### GitHub Test (5 minutes)

1. Fork iceoryx
2. Run `setup.sh`
3. Add GitHub token
4. Open PR → Bot runs

## 🎓 How It Works

```
GitHub PR
   ↓
GitHub Actions Workflow
   ├─→ Checkout iceoryx + review-bot
   ├─→ Setup environment (clang-18, CMake, Python)
   ├─→ Build project → compile_commands.json
   ├─→ Run Analysis:
   │   ├─→ Clang warnings
   │   ├─→ clang-tidy checks
   │   └─→ Clang Static Analyzer
   ├─→ Normalize findings → SARIF
   ├─→ Filter new only + deduplicate
   ├─→ Post results to GitHub:
   │   ├─→ Check run summary
   │   ├─→ Inline comments (high severity)
   │   ├─→ SARIF to Code Scanning
   │   └─→ Artifacts (JSON + report)
   └─→ Done!

PR Timeline shows:
  ✅ Check run (summary)
  💬 Inline comments (details)
  📊 Code Scanning (SARIF results)
  📦 Artifacts (reports)
```

## 🏆 Status

✅ **READY FOR PRODUCTION**

- [x] Core pipeline implemented (1,068 lines)
- [x] GitHub integration complete
- [x] SARIF output working
- [x] Baseline & deduplication ready
- [x] Configuration system flexible
- [x] Comprehensive documentation
- [x] Setup automation
- [x] Error handling & robustness
- [x] Tested architecture

**Next:** Deploy to your iceoryx fork!

## 📦 What's Inside `code-review-bot/`

```
├── src/
│   ├── review_orchestrator.py      # Main orchestration engine
│   ├── github_reporter.py          # GitHub API integration
│   └── baseline_manager.py         # Deduplication + baseline
│
├── .github/workflows/
│   └── code-review.yml             # GitHub Actions workflow
│
├── review-config.yaml              # Configuration & rule mappings
├── setup.sh                        # Installation script
├── requirements.txt                # Python deps (PyGithub, pyyaml)
│
├── QUICKSTART.md                   # 👈 Start here!
├── README.md                       # Full documentation
├── IMPLEMENTATION.md               # Technical guide
└── CHEATSHEET.md                   # Quick reference
```

## 🚀 Getting Started

**For quick start:** See [code-review-bot/QUICKSTART.md](code-review-bot/QUICKSTART.md)

**For full docs:** See [code-review-bot/README.md](code-review-bot/README.md)

**For technical details:** See [code-review-bot/IMPLEMENTATION.md](code-review-bot/IMPLEMENTATION.md)

**For complete inventory:** See [DELIVERABLES.md](DELIVERABLES.md)

## 🤝 Contributing

To extend the bot:

1. Read [IMPLEMENTATION.md](code-review-bot/IMPLEMENTATION.md)
2. Modify `src/*.py` or `review-config.yaml`
3. Test locally
4. Submit PR

## 📄 License

Apache License 2.0 (matches Eclipse iceoryx)

## 📞 Support

- **Setup issues?** → [QUICKSTART.md](code-review-bot/QUICKSTART.md)
- **How to use?** → [README.md](code-review-bot/README.md)
- **Configuration?** → [CHEATSHEET.md](code-review-bot/CHEATSHEET.md)
- **Technical deep-dive?** → [IMPLEMENTATION.md](code-review-bot/IMPLEMENTATION.md)
- **What was built?** → [DELIVERABLES.md](DELIVERABLES.md)

---

**Status:** ✅ MVP Complete and Ready for Deployment

**Next Action:** [Read QUICKSTART.md →](code-review-bot/QUICKSTART.md)
