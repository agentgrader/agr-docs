# CI workflows

Patterns for running Agentgrader in pull-request and nightly pipelines.

## Validate before you benchmark

Agent runs are slow and cost money. Gate incomplete definitions first:

```bash
# Validate a single test case
agr validate fix-greeting --strict

# Validate every test case in a directory at once
agr validate --suite tasks/ --strict

# Also audit toolkit security (all referenced toolkits directories)
agr validate --suite tasks/ --strict --audit-toolkits
```

`--strict` exits 1 when SWE-bench fields (`test_command`, `fail_to_pass`, `pass_to_pass`) are missing. This makes it suitable as a pre-bench CI gate.

## Preview before running

Before a large bench (many tasks or a matrix sweep), use `--dry-run` to confirm the test case and config matrix without spending any API budget or starting Docker containers:

```bash
agr bench --suite tasks/ --matrix matrix.yaml --dry-run
```

The output shows every test case and config that would be included, with total job count.

## Exit codes for CI gates

By default `agr run` and `agr bench` exit `0` even when the agent fails scoring. Use explicit gates:

```bash
# Single run gate
agr run hello-world --config agent.yaml --fail-on-failure

# Suite gate with solve-rate floor
agr bench --suite tasks/ --config agent.yaml \
  --fail-on-failure \
  --min-solve-rate 0.8 \
  --strict-toolkits
```

| Flag | Effect |
|---|---|
| `--fail-on-failure` | Exit 1 if any run does not pass |
| `--min-solve-rate` | Exit 1 if solve rate is below threshold |
| `--min-solve-rate-scope per-config` | Apply solve-rate gate per agent config |
| `--strict-toolkits` | Exit 1 if a referenced toolkit fails security audit |

## Report artifacts

```bash
agr bench --suite tasks/ --config agent.yaml \
  --fail-on-failure \
  --report json \
  --output reports/bench.json
```

Formats: `json`, `jsonl`, `html`, `md`. Add `--report-include-traces` for full step traces (large files).

## Baseline comparison on pull requests

On `main`:

```bash
agr bench --suite tasks/ --config agent.yaml --save-baseline baselines/main.json
```

On a PR branch (after bench):

```bash
agr compare-baseline --current baselines/main.json \
  --format md \
  --output comment.md \
  --fail-on-regression
```

Post `comment.md` as a PR comment. See [Run history & export](/guide/persistence#baseline-snapshots).

## Enumerate test cases from scripts

Use `agr list-tests --json` to get a machine-readable list of test case names and paths, useful for build scripts or dynamic CI matrix generation:

```bash
# Get all test case names as a JSON array
agr list-tests --json

# Pipe to jq to extract names
agr list-tests --json | jq -r '.[].name'
```

## Runner checklist

- Install with `bun add -g agentgrader` or `npm install -g agentgrader`
- Store API keys in encrypted CI secrets, not in the repo
- Cap parallelism with `--concurrency` to match runner CPU and Docker limits
- Gate expensive suites behind labels or scheduled workflows
- Pull Docker base images before large benches

Full GitHub Actions example: [CI Integration](/advanced/ci-integration).
