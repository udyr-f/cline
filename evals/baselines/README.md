# Evaluation Baselines

This directory contains baseline performance metrics for Cline's three-layer evaluation system.

## Baseline Files

- **tool-precision-replace-in-file.json** - Baseline for replace_in_file tool precision tests
- **coding-exercises-50.json** - Baseline for coding exercise benchmarks
- **real-world-sonnet-4-5.json** - Baseline for real-world tasks using Claude Sonnet 4.5

## Baseline Update Policy

### When to Update

**Tool Precision Baselines:**
- Update after intentional behavior changes to the tool
- Update after fixing known bugs that affect test outcomes
- Do NOT update for transient failures or flakiness

**Coding Exercise Baselines:**
- Update quarterly or after major releases
- Update when system prompt changes significantly affect exercise performance
- Do NOT update for minor fluctuations or individual task changes

**Real-World Task Baselines:**
- Update per model version (e.g., sonnet-4-5 → sonnet-4-6)
- Maintain separate baselines for each model
- Update when cline-bench task set changes significantly
- Do NOT update for individual task additions/removals

### How to Update

**Tool Precision:**
```bash
cd evals/benchmarks/tool-precision/replace-in-file
npm test -- --output-json > ../../baselines/tool-precision-replace-in-file.json
```

**Coding Exercises:**
```bash
cd evals/benchmarks/coding-exercises
npm test -- --count 50 --output-json > ../baselines/coding-exercises-50.json
```

**Real-World Tasks:**
```bash
cd evals/benchmarks/real-world/cline-bench
export API_KEY=sk-ant-your-key
harbor run -p tasks -a cline-cli -m anthropic:claude-sonnet-4-5:1m --env docker -k 3
cd ../../../analysis
npm start -- analyze ../benchmarks/real-world/cline-bench/jobs/LATEST/ --format json > ../baselines/real-world-sonnet-4-5.json
```

### Regression Thresholds

CI will flag regressions based on these thresholds:

- **Tool precision**: Fail if pass rate drops >5% (high precision expected)
- **Coding exercises**: Fail if pass@1 drops >10% (some variance tolerated)
- **Real-world tasks**: Fail if pass@3 drops >10% (nondeterminism expected)

### Version Control

Baseline updates should be committed with:
- Clear commit message explaining why the baseline changed
- Link to relevant PRs or issues that motivated the update
- Approval from team lead before merging baseline updates

Example commit message:
```
chore(evals): update real-world baseline for Sonnet 4.5

- New baseline reflects improvements from PR #1234
- Pass@3 increased from 75% to 83% due to better context handling
- Reviewed by @team-lead
```

## Current Status

All baseline files are currently placeholders. Run the benchmark suites to generate actual baselines before enabling CI regression detection.
