# Tool Precision Benchmarks

This directory contains focused tests for individual Cline tools, testing them in isolation without full agent complexity.

## Purpose

Tool precision tests provide:
- **Fast feedback** - Run in seconds, ideal for development
- **High signal-to-noise** - Isolated testing reduces confounding variables
- **Regression detection** - Catch tool behavior changes immediately
- **Fine-grained debugging** - Pinpoint exact tool issues

## Current Test Suites

### replace-in-file/

Tests Cline's `replace_in_file` tool for diff-based file editing.

**Coverage:** 100+ test cases from real Cline sessions and synthetic scenarios

**Test Dimensions:**
- Single vs multi-line replacements
- Whitespace handling (tabs, spaces, mixed)
- Edge cases (empty lines, special characters, regex patterns)
- Large file handling
- Error recovery

**Run:**
```bash
cd replace-in-file
npm install
npm test
```

**Test Case Structure:**
```json
{
  "test_id": "example-001",
  "provenance": {
    "source": "real-cline-session",
    "session_id": "01k6kr5hbv8za80v8vnze3at8h",
    "date": "2024-11-15",
    "description": "Edge case from production session",
    "contributor": "@username",
    "license": "MIT"
  },
  "messages": [...],
  "file_contents": "...",
  "file_path": "src/example.js",
  "expected_output": "..."
}
```

**Metrics:**
- Pass rate (% of cases where tool successfully edits file)
- Average latency (ms per edit)
- Token efficiency (tokens used per successful edit)
- Known failure patterns

**Baseline:** `evals/baselines/tool-precision-replace-in-file.json`

## Adding New Tool Tests

### 1. Choose a Tool to Test

Candidates for tool precision testing:
- `write_to_file` - File creation and full-file writing
- `execute_command` - Shell command execution
- `ask_followup_question` - User interaction
- `web_search` - Search query formulation

### 2. Create Test Suite Directory

```bash
mkdir -p tool-precision/my-tool-name
cd tool-precision/my-tool-name
npm init -y
```

### 3. Define Test Case Schema

Create `cases/schema.json` documenting your test case structure:
```json
{
  "test_id": "string - unique identifier",
  "provenance": "object - test case origin metadata",
  "input": "object - tool inputs",
  "expected_output": "object - expected tool behavior",
  "context": "object - environment/state"
}
```

### 4. Collect Test Cases

Sources:
- **Real sessions:** Extract from production Cline usage (anonymized)
- **Synthetic:** Generate programmatically for coverage
- **Regression bugs:** Add cases when bugs are fixed
- **User-contributed:** Community submissions

**IMPORTANT:** All test cases must include full provenance metadata (see example above).

### 5. Implement Test Runner

```typescript
// test-runner.ts
import { ClineWrapper } from './wrapper'

interface TestCase {
  test_id: string
  provenance: ProvenanceMetadata
  // ... tool-specific fields
}

async function runTest(testCase: TestCase): Promise<TestResult> {
  const wrapper = new ClineWrapper(/* config */)

  // 1. Setup: Configure tool state
  // 2. Execute: Call tool with test inputs
  // 3. Verify: Compare output to expected
  // 4. Teardown: Clean up resources

  return {
    test_id: testCase.test_id,
    passed: /* boolean */,
    latency_ms: /* number */,
    token_usage: /* object */,
    error: /* string | null */
  }
}
```

### 6. Create npm Scripts

```json
{
  "scripts": {
    "test": "tsx test-runner.ts",
    "test:watch": "tsx --watch test-runner.ts",
    "test:verbose": "tsx test-runner.ts --verbose"
  }
}
```

### 7. Generate Baseline

```bash
npm test -- --output-json > ../../baselines/tool-precision-my-tool-name.json
```

### 8. Update CI Configuration

Add to `.github/workflows/cline-evals-regression.yml`:
```yaml
tool-precision-my-tool:
  runs-on: ubuntu-latest
  steps:
    - run: cd evals/benchmarks/tool-precision/my-tool-name && npm test
    - run: cd evals/analysis && npm start -- compare ...
```

## Test Data Provenance

All test cases MUST include provenance metadata:

```typescript
interface ProvenanceMetadata {
  source: 'real-cline-session' | 'synthetic' | 'user-contributed' | 'regression-bug'
  session_id?: string          // If from real session
  github_issue?: string        // If from bug report
  date: string                 // ISO 8601 date
  description: string
  contributor: string
  license: 'MIT' | 'Apache-2.0'
}
```

**Privacy:** When extracting test cases from real Cline sessions:
- Obtain user consent
- Remove personally identifiable information
- Strip sensitive file paths, API keys, credentials
- Generalize company-specific code to generic examples

**Attribution:** Credit original contributors and maintain license compliance.

## Best Practices

### Test Case Design

- **Atomic:** Each test case should test one specific behavior
- **Independent:** Tests shouldn't depend on each other
- **Deterministic:** Same input should always produce same output (no randomness)
- **Fast:** Keep individual tests under 1 second when possible
- **Representative:** Cover real-world usage patterns, not just edge cases

### Naming Conventions

- Test IDs: `{tool-name}-{category}-{number}` (e.g., `replace-multiline-003`)
- Categories: `basic`, `edge`, `error`, `performance`, `regression`

### Error Handling

Tests should distinguish:
- **Tool errors:** Bug in the tool implementation
- **Test errors:** Bug in the test harness
- **Expected failures:** Tool correctly rejecting invalid input

### Performance Benchmarking

Track latency over time to catch performance regressions:
```json
{
  "test_id": "replace-large-file-001",
  "baseline_latency_ms": 450,
  "threshold_multiplier": 2.0  // Fail if >2x slower
}
```

## License

All test cases in this directory are licensed under MIT unless otherwise specified in the test case metadata.
