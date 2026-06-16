# Recipes

Copy-paste patterns for common Agentgrader workflows.

## SWE-bench-style regression case

1. Scaffold or author `agr.yaml` with `test_command`, fixture, and prompt.
2. Run the test suite on the **broken** fixture; copy failing test names into `fail_to_pass`.
3. Copy tests that must stay green into `pass_to_pass`.
4. Validate:

```bash
agr validate my-case --strict
```

Example fields:

```yaml
test_command: "pytest --tap-stream"
fail_to_pass:
  - "test_subtract handles negative numbers"
pass_to_pass:
  - "test_add sums integers"
forbid_modified:
  - "tests/**"
```

Schema: [Test Case YAML](/reference/test-case-yaml).

## Hyperparameter matrix sweep

```yaml
# matrix.yaml
name: model-sweep
base:
  max_steps: 20
  temperature: 0.2
dimensions:
  model:
    - anthropic/claude-sonnet-4
    - openai/gpt-4o-mini
  temperature:
    - 0.2
    - 0.7
```

```bash
agr bench --matrix matrix.yaml --suite tasks/ --fail-on-failure
agr export runs --last-matrix --format jsonl --output sweep.jsonl
```

See [Optimizer Matrix YAML](/reference/matrix-yaml) and [Optimizer matrices](/guide/optimizer-matrices).

## PR baseline gate

On `main` (or nightly):

```bash
agr bench --suite tasks/ --config agent.yaml \
  --save-baseline baselines/main.json \
  --report json --output reports/main.json
```

On each PR:

```bash
agr bench --suite tasks/ --config agent.yaml --fail-on-failure
agr compare-baseline --current baselines/main.json \
  --format md \
  --output comment.md \
  --fail-on-regression
```

Upload `reports/` and post `comment.md` as a PR comment. Details: [CI workflows](/guide/ci-workflows).

## ACP agent with sandboxed MCP

Agent config (AI SDK adapter path uses the same `sandboxed: true` flag):

```yaml
mcp_servers:
  jetbrains-tools:
    command: bun
    args: [/app/toolkits/jetbrains-tools/mcp-server.ts]
    sandboxed: true
toolkits:
  - ./toolkits/jetbrains-tools
```

```bash
agr run my-case --config agent-acp.yaml --adapter acp
```

ACP rewrites sandboxed stdio servers to `agr-mcp-proxy` when the sandbox exposes `sandboxBridgeId`. See [ACP Agent Adapter](/advanced/acp-agent#sandboxed-true-for-acp-agents-via-agr-mcp-proxy).

## LLM judge with custom rubrics

Test case:

```yaml
rubrics:
  - id: correctness
    prompt: Does the patch fix the bug without regressions?
    scale: "0-1"
    weight: 2
  - id: style
    prompt: Is the code readable and idiomatic?
    scale: "1-5"
```

Bench with gate:

```bash
export ANTHROPIC_API_KEY=...
agr bench --suite tasks/ --config agent.yaml \
  --llm-judge \
  --judge-gate \
  --judge-min-score 0.75 \
  --report md --output reports/judge.md
```

Inspect per-run scores: `agr trace --last --quality` (or `agr trace <runId> --quality` for a specific run).

## Toolkit matrix A/B

```yaml
name: toolkits
base:
  model: claude-haiku-4-5-20251001
  provider: anthropic
  track_tools: ["run-tests", "find-usages"]
dimensions:
  toolkits:
    - []
    - ["./toolkits/jetbrains-tools"]
```

```bash
agr bench --matrix matrix.yaml --suite tasks/
```

Compare **TOOL USAGE BY CONFIG** and `agr trace --tools` across matrix arms. See [Toolkits guide](/guide/toolkits).

## Toolkit security audit

Before referencing a toolkit in production agents, audit it and then lock the bench to enforce it:

```bash
# Audit the toolkit for security issues (executable scripts, path traversal, etc.)
agr validate-toolkit ./toolkits/my-toolkit

# List all tools with their descriptions
agr toolkit-list ./toolkits/my-toolkit

# Check that every bin/ tool is tracked in your agent config
agr toolkit-list ./toolkits/my-toolkit --check-config agent.yaml

# Enforce clean audit in CI (exits 1 on any error finding)
agr bench --suite tasks/ --strict-toolkits --fail-on-failure
```

## Tag-based suite subsets

Add `tags:` to test cases by language, difficulty, or subsystem:

```yaml
# tasks/two-sum/agr.yaml
name: two-sum
tags:
  - python
  - easy
```

Then scope any command to that slice:

```bash
# See which test cases match
agr list-tests --tags python

# Validate only the python subset before spending bench budget
agr validate --suite tasks/ --tags python --strict

# Run only easy tasks in a matrix sweep
agr bench --suite tasks/ --matrix matrix.yaml --tags easy --dry-run
agr bench --suite tasks/ --matrix matrix.yaml --tags easy

# With a bench manifest, --tags works the same way (suite comes from the manifest)
agr bench --manifest bench.yaml --tags python
```

Tag breakdowns (pass rate per tag) print automatically at the end of every `agr bench` run.

## Quick post-run inspection

After any `agr run` or `agr bench`, use the `--last` shortcuts to inspect without looking up run IDs:

```bash
# Inspect the most recent run
agr trace --last
agr trace --last --quality
agr trace --last --tools

# Compare the two most recent runs (e.g. after an A/B bench)
agr compare --last-two --only-diff

# Export traces of the most recent run (for external analysis or OTel ingestion)
agr export traces --last --format otlp --output last-trace.json
```

## Import a case from GitHub

```bash
export GITHUB_TOKEN=...
agr import-pr owner/repo 123 --clone-fixture --out tasks/repo-123
# fill in fail_to_pass / pass_to_pass from TAP output
agr validate repo-123 --strict
```

## E2B cloud sandbox

```bash
export E2B_API_KEY=...
agr bench --suite tasks/ --config agent.yaml --sandbox e2b --concurrency 2
```

See [Choosing a sandbox](/guide/sandboxes).
