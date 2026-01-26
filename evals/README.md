# Cline Evaluation Framework

A comprehensive three-layer testing system for measuring Cline's performance across different granularities and time scales.

## Architecture Overview

```
evals/
├── benchmarks/
│   ├── tool-precision/         # Individual tool testing (seconds)
│   ├── coding-exercises/       # Small programming tasks (minutes)
│   └── real-world/             # Production bugs via cline-bench (20-30 min/task)
│
├── analysis/                   # Unified TypeScript analysis framework
│   ├── src/
│   │   ├── parsers/            # Harbor, tool, exercise result parsers
│   │   ├── reporters/          # Markdown, JSON output generators
│   │   ├── classifier.ts       # Failure pattern matching
│   │   ├── metrics.ts          # pass@k, pass^k, flakiness calculations
│   │   └── cli.ts              # Main CLI entry point
│   └── patterns/
│       └── cline-failures.yaml # Known failure patterns (Gemini #7974, Claude #7998)
│
└── baselines/                  # Performance baselines for regression detection
```

## Three Complementary Test Layers

### 1. Tool Precision Testing (Fast - Seconds)

Tests individual Cline tools in isolation without full agent complexity.

**Location:** `benchmarks/tool-precision/`

**Current Coverage:**
- `replace-in-file/` - 100+ test cases for diff-based file editing

**Purpose:**
- High-precision validation of core tool behavior
- Quick feedback during development
- Regression detection for tool changes

**Example:**
```bash
cd benchmarks/tool-precision/replace-in-file
npm test
```

### 2. Coding Exercises (Medium - Minutes)

Small, focused programming tasks for fast regression testing.

**Location:** `benchmarks/coding-exercises/`

**Coverage:** Polyglot exercises (Python, JavaScript, Rust, Go, etc.)

**Purpose:**
- Smoke tests before deployment
- Quick model capability checks
- Language-specific regression testing

**Example:**
```bash
cd benchmarks/coding-exercises
npm test -- --count 10
```

### 3. Real-World Tasks (Slow - 20-30 min/task)

Full agent complexity on production bugs from real Cline user sessions.

**Location:** `benchmarks/real-world/cline-bench/` (git submodule)

**Source:** [cline/cline-bench](https://github.com/cline/cline-bench) - 12 curated real-world tasks

**Purpose:**
- Measure full agent performance on production scenarios
- Capture nondeterministic behavior and flakiness
- Compare model capabilities on realistic workloads

**Example:**
```bash
cd benchmarks/real-world/cline-bench
export API_KEY=sk-ant-your-key
harbor run -p tasks/01k7a12sd1nk15j08e6x0x7v9e-discord-trivia-approval-keyerror \
           -a cline-cli \
           -m anthropic:claude-sonnet-4-5:1m \
           --env docker \
           -k 3
```

## Unified Analysis Framework

All three test layers feed into a common TypeScript analysis framework.

### Key Features

**Failure Classification:**
- Pattern-based categorization (provider bugs, transient failures, task failures)
- Links failures to known GitHub issues (#7974, #7998)
- Distinguishes retriable from non-retriable failures

**Advanced Metrics:**
- **pass@k:** P(at least 1 of k trials passes) - measures solution-finding capability
- **pass^k:** P(all k trials pass) - measures reliability/consistency
- **Flakiness Score:** Entropy-based variance measure

**Rich Reporting:**
- Markdown reports with color coding and visual indicators
- Structured JSON output with schema versioning
- Minimal JSON for CI/CD integration

### CLI Usage

```bash
cd analysis

# Analyze a completed Harbor job
npm start -- analyze ../benchmarks/real-world/cline-bench/jobs/2025-01-25__10-30-00/

# Compare baseline vs current for regression detection
npm start -- compare ../baselines/real-world-sonnet-4-5.json pr-results.json --threshold 10

# Generate different output formats
npm start -- analyze <job-dir> --format markdown|json|minimal --output report.md
```

### Example Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Cline Bench Analysis Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Job: jobs/2025-01-25__10-30-00/
Model: anthropic:claude-sonnet-4-5:1m
Tasks: 12 | Trials per task: 3

Overall Metrics:
  pass@1: 75.0%  (9/12 tasks passed on first try)
  pass@3: 83.3%  (10/12 tasks passed within 3 tries)
  pass^3: 66.7%  (8/12 tasks passed ALL 3 trials - reliability)

Known Issues Detected:
  • gemini_signature (1 occurrence) - Issue #7974
  • rate_limit (2 occurrences)

Total Cost: $12.34 | Avg per task: $1.03
```

## CI Integration

### Regression Detection

GitHub Actions runs tool precision tests on every PR:

```yaml
# Triggered on PRs touching src/core/, src/api/, or evals/
- Run tool precision tests
- Compare against baseline
- Fail if pass rate drops >5%
- Post comment on PR if regression detected
```

### Baseline Management

Baselines stored in `baselines/` directory:
- **tool-precision-replace-in-file.json** - Update after tool behavior changes
- **coding-exercises-50.json** - Update quarterly
- **real-world-sonnet-4-5.json** - Update per model version

See `baselines/README.md` for update policy and procedures.

## Getting Started

### Prerequisites

- Node.js 20+
- Docker (for real-world tasks)
- API keys (Anthropic, OpenAI, etc.)

### Setup

```bash
# Clone with submodules
git clone --recursive https://github.com/cline/cline.git

# Or if already cloned
git submodule update --init --recursive

# Install analysis framework
cd evals/analysis
npm install
```

### Running Your First Evaluation

**Quick Start (Tool Precision):**
```bash
cd evals/benchmarks/tool-precision/replace-in-file
npm install
npm test
```

**Real-World Tasks:**
```bash
cd evals/benchmarks/real-world/cline-bench
export API_KEY=sk-ant-your-key
harbor run -p tasks/01k7a12sd1nk15j08e6x0x7v9e-discord-trivia-approval-keyerror \
           -a cline-cli \
           -m anthropic:claude-sonnet-4-5:1m \
           --env docker \
           -k 3

# Analyze results
cd ../../analysis
npm start -- analyze ../benchmarks/real-world/cline-bench/jobs/LATEST/
```

## Architecture Decisions

### Why Three Layers?

Different test types serve different purposes:

- **Tool Precision:** Fast feedback, high signal-to-noise, isolated testing
- **Coding Exercises:** Balanced speed/realism, good for smoke tests
- **Real-World Tasks:** Full complexity, captures edge cases, measures production readiness

### Why TypeScript?

- Consistent with main Cline codebase
- Strong typing for schema validation
- Rich Node.js ecosystem for parsing/reporting
- Fast enough for analysis workloads

### Why cline-bench as a Submodule?

- **Agent-agnostic:** Can be used by other AI coding agents
- **Clear boundary:** We consume benchmarks, don't modify them
- **Explicit updates:** Tracked via git commit hash
- **Community benefit:** Other projects can use the same benchmark

### Why pass@k and pass^k?

In nondeterministic AI systems:
- **pass@k** measures solution-finding capability (at least one success in k tries)
- **pass^k** measures reliability (all k tries succeed)

As trials increase, these metrics diverge for flaky tasks:
- pass@k → 100% (eventually finds a solution)
- pass^k → 0% (can't reliably reproduce)

Both metrics matter for production AI systems.

## Contributing

### Adding Test Cases

**Tool Precision:**
See `benchmarks/tool-precision/replace-in-file/README.md`

**Coding Exercises:**
See `benchmarks/coding-exercises/README.md`

**Real-World Tasks:**
Contribute to [cline/cline-bench](https://github.com/cline/cline-bench)

### Test Data Provenance

All test cases must include metadata documenting their origin:
- Source (real session, synthetic, user-contributed, regression bug)
- Date collected
- License (MIT or Apache 2.0)
- Contributor
- Description

See individual benchmark READMEs for specific provenance requirements.

## Legacy Code

The previous evaluation system (`evals/cli/`) has been moved to `evals/legacy/cli/` for reference. The new system provides:
- Better separation of concerns (execution vs analysis)
- Unified metrics across all test types
- Improved failure classification
- Schema-versioned output for stability

## Resources

- [cline-bench repository](https://github.com/cline/cline-bench)
- [cline-bench initiative blog post](https://cline.bot/blog/cline-bench-initiative)
- [Harbor framework documentation](https://harborframework.com)
- [HumanEval paper](https://arxiv.org/abs/2107.03374) (pass@k methodology)

## License

Test data and benchmarks:
- Tool precision tests: MIT
- Coding exercises: MIT (derived from Aider's polyglot-benchmark)
- Real-world tasks: Apache 2.0 (via cline-bench)

Analysis framework: Apache 2.0 (same as Cline)
