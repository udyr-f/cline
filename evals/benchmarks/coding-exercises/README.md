# Coding Exercise Benchmarks

Small, focused programming tasks for fast regression testing of Cline's coding capabilities across multiple languages.

## Purpose

Coding exercises provide:
- **Fast smoke tests** - Run in minutes, ideal for pre-deployment checks
- **Language coverage** - Test Python, JavaScript, Rust, Go, Java, etc.
- **Balanced realism** - More realistic than isolated tools, faster than real-world tasks
- **Quick model comparison** - Rapidly compare different models/prompts

## Test Suite

### Coverage

**Languages:** Python, JavaScript, TypeScript, Rust, Go, Java, C++, Ruby, PHP
**Exercise Count:** ~50 exercises across languages
**Typical Duration:** 30-60 seconds per exercise
**Total Suite Runtime:** 30-50 minutes (can run subset with `--count` flag)

### Exercise Types

- **Algorithm implementation** - Classic algorithms (sorting, searching, graph traversal)
- **Data structure manipulation** - Lists, trees, hashmaps
- **String processing** - Parsing, formatting, validation
- **Math problems** - Number theory, combinatorics
- **Real-world utilities** - Date handling, file parsing, API interaction

### Example Exercises

```
python/leap-year          - Determine if a year is a leap year
javascript/hello-world    - Classic first program
rust/gigasecond          - Calculate gigasecond anniversary
go/two-fer               - String formatting exercise
```

## Running Exercises

### Basic Usage

```bash
cd evals/benchmarks/coding-exercises
npm install
npm test
```

### Options

```bash
# Run specific count
npm test -- --count 10

# Run specific language
npm test -- --language python

# Run with specific model
npm test -- --model anthropic:claude-sonnet-4-5

# Verbose output
npm test -- --verbose

# Multiple attempts per exercise (for pass@k)
npm test -- --count 10 --attempts 3
```

### Output

```
Running coding exercises...

✓ python/leap-year (1.2s, $0.02)
✓ javascript/hello-world (0.8s, $0.01)
✗ rust/gigasecond (2.1s, $0.03) - Test failure: expected 2046-10-01, got 2046-09-30

Summary:
  Total: 10 exercises
  Passed: 9 (90%)
  Failed: 1 (10%)
  pass@1: 0.90
  Total cost: $0.18
  Total duration: 12.4s
```

## Attribution

### Source

Test cases are derived from the [Aider Polyglot Benchmark](https://github.com/Aider-AI/polyglot-benchmark), which itself uses exercises from [Exercism](https://exercism.org/).

**Original Repository:** https://github.com/Aider-AI/polyglot-benchmark
**Curated By:** Aider team (Paul Gauthier)
**Original Source:** Exercism (https://exercism.org/)
**License:** MIT License
**Included in Cline:** 2025-01-25
**Modifications:** None (used as-is)

### License

```
MIT License

Copyright (c) Exercism
Copyright (c) Aider team (Paul Gauthier)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### Why These Exercises?

Aider's polyglot-benchmark was chosen because:
- **Well-tested:** Exercises have clear specifications and test suites
- **Polyglot:** Covers major programming languages
- **Realistic:** Represents common coding tasks
- **Maintained:** Active community and clear license
- **Proven:** Used by Aider for model evaluation

## Metrics

### Standard Metrics

- **Pass Rate:** % of exercises where generated code passes all tests
- **pass@k:** P(at least 1 of k attempts succeeds) - solution finding
- **pass^k:** P(all k attempts succeed) - consistency/reliability
- **Cost:** Total API cost in USD
- **Duration:** Wall-clock time per exercise

### Language-Specific Metrics

Track performance by language to catch language-specific regressions:
```json
{
  "python": {"pass_rate": 0.95, "avg_duration_sec": 8.2},
  "javascript": {"pass_rate": 0.92, "avg_duration_sec": 6.5},
  "rust": {"pass_rate": 0.78, "avg_duration_sec": 12.1}
}
```

## Baseline Management

### Current Baseline

`evals/baselines/coding-exercises-50.json`

### Update Policy

- **Quarterly:** Update baseline every 3 months with latest performance
- **Major releases:** Update when Cline version changes significantly (e.g., 3.0 → 4.0)
- **System prompt changes:** Update if system prompt affects exercise performance
- **Do NOT update:** For minor fluctuations or individual exercise changes

### Updating Baseline

```bash
cd evals/benchmarks/coding-exercises
npm test -- --count 50 --attempts 3 --output-json > ../baselines/coding-exercises-50.json
```

Commit with clear justification:
```bash
git add baselines/coding-exercises-50.json
git commit -m "chore(evals): update coding exercises baseline

- New baseline reflects improvements from PR #1234
- Pass@1 increased from 88% to 92% due to better language detection
- Reviewed by @team-lead
"
```

## CI Integration

### GitHub Actions

The `cline-evals-regression.yml` workflow can optionally run coding exercises:

```yaml
coding-exercises:
  if: contains(github.event.pull_request.labels.*.name, 'eval:exercises')
  steps:
    - run: npm test -- --count 20
    - run: compare against baseline
```

To trigger on a PR:
1. Add the `eval:exercises` label to your PR
2. Workflow will run 20 representative exercises
3. Fail if pass@1 drops >10%

## Contributing

### Adding New Exercises

**Preferred:** Contribute exercises to [Aider's polyglot-benchmark](https://github.com/Aider-AI/polyglot-benchmark), then update our submodule/integration.

**Direct contribution** (for Cline-specific exercises):

1. Create exercise in appropriate language directory
2. Follow Exercism structure:
   - `instructions.md` - Problem description
   - `test.{ext}` - Test suite
   - `example.{ext}` - Reference solution
3. Document provenance:
   ```json
   {
     "exercise_id": "cline-specific-task-001",
     "source": "cline-team",
     "author": "@username",
     "date": "2025-01-25",
     "license": "MIT"
   }
   ```

### Testing Guidelines

- **Clear specifications:** Exercise requirements must be unambiguous
- **Comprehensive tests:** Cover normal cases, edge cases, errors
- **Language-appropriate:** Use idiomatic patterns for each language
- **Reasonable difficulty:** Should be solvable by models in 1-2 attempts
- **Isolated:** No dependencies on other exercises or external services

## Comparison with Other Test Layers

| Aspect | Tool Precision | Coding Exercises | Real-World Tasks |
|--------|----------------|------------------|------------------|
| Speed | Seconds | Minutes | 20-30 min/task |
| Realism | Low | Medium | High |
| Signal/Noise | High | Medium | Low |
| Use Case | Development | Smoke tests | Model evaluation |

Coding exercises occupy the middle ground: realistic enough to catch regressions, fast enough for frequent testing.

## Resources

- [Aider Polyglot Benchmark](https://github.com/Aider-AI/polyglot-benchmark)
- [Exercism](https://exercism.org/)
- [HumanEval paper](https://arxiv.org/abs/2107.03374) (pass@k methodology)
- [Aider's benchmark analysis](https://aider.chat/docs/leaderboards/)

## License

All exercises in this directory are licensed under the MIT License (see above).
