# Real-World Benchmark Tasks

This directory contains real-world coding tasks derived from actual Cline user sessions via the [cline-bench](https://github.com/cline/cline-bench) project.

## cline-bench Submodule

The `cline-bench/` subdirectory is a git submodule pointing to the external [cline/cline-bench](https://github.com/cline/cline-bench) repository.

**Why a submodule?**
- cline-bench is agent-agnostic and maintained independently
- Clear boundary: we consume the benchmark, but don't modify it
- Updates are explicit and tracked via git commit hash
- Other projects can use the same benchmark

## Initial Setup

If you cloned the Cline repo without `--recursive`:

```bash
git submodule update --init --recursive
```

## Updating cline-bench

To pull the latest version of cline-bench:

```bash
cd cline-bench
git fetch origin
git checkout main
git pull
cd ../../..
git add benchmarks/real-world/cline-bench
git commit -m "chore(evals): update cline-bench submodule to latest"
```

## Running Benchmarks

See the [cline-bench README](cline-bench/README.md) for detailed instructions on running benchmarks with Harbor.

**Quick example:**

```bash
cd cline-bench

# Set environment variables
export DAYTONA_API_KEY=dtn_your-key
export API_KEY=sk-ant-your-key

# Run a single task
harbor run -p tasks/01k7a12sd1nk15j08e6x0x7v9e-discord-trivia-approval-keyerror \
           -a cline-cli \
           -m anthropic:claude-sonnet-4-5:1m \
           --env docker \
           -k 3

# Analyze results with the Cline analysis framework
cd ../../analysis
npm start -- analyze ../benchmarks/real-world/cline-bench/jobs/LATEST/
```

## CI Integration

When using cline-bench in GitHub Actions, ensure submodules are checked out:

```yaml
- uses: actions/checkout@v4
  with:
    submodules: true  # Critical: fetch submodules
```

## More Information

- [cline-bench repository](https://github.com/cline/cline-bench)
- [cline-bench initiative blog post](https://cline.bot/blog/cline-bench-initiative)
- [Harbor framework documentation](https://harborframework.com)
