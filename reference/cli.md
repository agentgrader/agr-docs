# CLI Reference

The `agr` command ships with the [`agentgrader`](https://www.npmjs.com/package/agentgrader) npm package.

::: code-group

```bash [npm]
npm install -g agentgrader
agr <command>
```

```bash [bun]
bun add -g agentgrader
agr <command>
```

```bash [bunx / npx]
bunx agentgrader <command>
```

:::

> **Output convention**: `agr` output never contains emoji or other pictographic symbols, anywhere. Status is conveyed with plain text labels (`PASS`/`FAIL`, `[OK]`/`[WARN]`/`[FAIL]`, `[error]`) and, where the terminal supports it, ANSI/Ink color (green for pass, red for fail, yellow for warnings/in-progress, cyan for headers, gray for secondary text).

## `agr init`

Scaffold a minimal, runnable agentgrader project in the current directory (or `[dir]`), so you can try `agr run` immediately without writing any YAML by hand.

```bash
agr init
# or into a new directory:
agr init my-project
```

This creates:

- `agent.yaml`: a baseline agent config using `claude-haiku-4-5-20251001` with `provider: anthropic` and `max_steps: 15`.
- `.gitignore`: ignores `.agr/` (run history and exports) and `.env`. Skipped if a `.gitignore` already exists.
- `tasks/hello-world/agr.yaml` and `tasks/hello-world/fixture/`: a tiny, self-contained test case. The fixture is a `math.js` with an unimplemented `add(a, b)` and a `math.test.js` using Node's built-in test runner (`node --test`), so no `npm install` or `pip install` is needed inside the sandbox.

After scaffolding, set `ANTHROPIC_API_KEY` in your environment and run:

```bash
agr run hello-world --verbose
```

Use `--blank` if you'd rather start from an empty project and write your own test cases under `tasks/`:

```bash
agr init --blank
```

This writes only `agent.yaml` and an empty `tasks/` directory, skipping the `hello-world` sample. See [Core Concepts](/guide/concepts) for the `agr.yaml` schema, then run `agr list-tests` to confirm your test cases are found.

Add `--ci` to also generate a GitHub Actions workflow:

```bash
agr init --ci
# or combined:
agr init --blank --ci
```

This writes `.github/workflows/agr.yml`, a minimal GitHub Actions workflow that installs `agentgrader` and runs `agr bench --suite tasks/ --fail-on-failure` on every push to `main` and on pull requests. Add `ANTHROPIC_API_KEY` as a repository secret to enable it. The CI workflow is a starting point; customize the `run:` step to add `--min-solve-rate`, reports, or baseline comparisons.

### Options

| Flag | Default | Description |
|---|---|---|
| `[dir]` | `.` | Directory to scaffold into. Created if it does not exist. |
| `--force` | `false` | Overwrite `agent.yaml` if it already exists. Without it, `agr init` refuses to run on a directory that already has an `agent.yaml`, similar to `git init` on an existing repo. |
| `--blank` | `false` | Only write `agent.yaml` and an empty `tasks/` directory, without the `hello-world` sample test case. |
| `--ci` | `false` | Also write `.github/workflows/agr.yml`, a GitHub Actions CI workflow that runs `agr bench --suite tasks/ --fail-on-failure` on push and pull_request. Skipped if the file already exists. |
| `--example <lang>` | `js` | Sample test case language for the scaffolded hello-world. `js` (default) uses Node's built-in test runner. `python` (or `py`) uses `pytest -x` and scaffolds `math.py` and `test_math.py` instead. Requires `pytest` in the sandbox Docker image. |
| `--model <model>` | `claude-haiku-4-5-20251001` | Model to write into the scaffolded `agent.yaml`. Use to start a project for a non-default model without editing the YAML afterward. |
| `--provider <provider>` | `anthropic` | Provider to write into the scaffolded `agent.yaml`. Combine with `--model` to scaffold for a different provider (e.g. `agr init --model gpt-4o --provider openai`). |

### Examples

```bash
# Scaffold into the current directory
agr init

# Scaffold into a new directory
agr init my-project

# Re-scaffold, overwriting agent.yaml
agr init my-project --force

# Blank project: agent.yaml + empty tasks/, no sample test case
agr init --blank

# With CI workflow
agr init --ci
agr init --blank --ci

# Python test case (pytest-based)
agr init --example python
```

## `agr list-tests`

Recursively scan a directory for test case files (`agr.yaml` and similar) and print each one's `name`, relative path, description, and tags (when any test case in the set has tags). Useful for discovering what `agr run`/`agr bench` accept as a short name instead of a full path.

```bash
agr list-tests
agr list-tests tasks/
```

### Options

| Flag | Default | Description |
|---|---|---|
| `[dir]` | `.` | Directory to scan recursively for test case YAML files. |
| `--name <substring>` | (none) | Filter by test case name substring (case-insensitive). Combinable with `--tags` and `--count`. |
| `--json` | `false` | Print results as a JSON array (`name`, `path`, `relativePath`, optional `description`, optional `tags`) instead of a human-readable table. Useful for scripting or CI step that enumerates test cases. |
| `--count` | `false` | Print only the number of matching test cases as a bare integer. Useful in shell conditions: `if [ $(agr list-tests --count) -eq 0 ]; then ...`. |
| `--tags <tags>` | (none) | Comma-separated list of tags; only show test cases whose `tags:` list contains at least one match. |

### Examples

```bash
# List test cases under the current directory
agr list-tests

# List test cases under a specific directory
agr list-tests tasks/

# Machine-readable JSON (for scripting)
agr list-tests --json | jq '.[].name'

# Count only (for shell conditions)
agr list-tests --count
agr list-tests --tags python --count

# Show only Python test cases
agr list-tests --tags python
```

## `agr run`

Run a single test case with one agent config. Useful for debugging a specific case or iterating on prompts.

```bash
agr run hello-world --config agent.yaml
```

`<testCase>` accepts any of:

- a path to an `agr.yaml` file (with or without the `.yaml`/`.yml` extension)
- a path to a directory containing an `agr.yaml`
- a test case's `name:` field, or its directory's basename, looked up by recursively searching the current directory (as `agr list-tests` would find it)

`agr run` renders a live terminal UI (built with Ink) while the agent works, then a summary panel and diff once it finishes:

- **Live steps**: each `StepEvent` the agent emits appears as soon as it happens, color-coded by kind (tool calls, tool results, messages, thinking). With `--verbose`, each step shows its tool name and a truncated content preview (up to 200 characters). Without `--verbose`, you instead get a compact running counter of step count and accumulated cost.
- **Summary panel**: a bordered `RUN SUMMARY` box showing status (`PASSED`/`FAILED`), step count, cost, duration, the prompt-cache hit rate (`prompt cache: X/Y input tokens served from cache (Z%)`), any run error, and the regression/diff/localization metric lines (skipped checks are flagged with `[skip]`).
- **Diff**: if the agent changed any files, a `Diff` panel renders the unified git diff with added lines in green, removed lines in red, and hunk headers (`@@ ...`) in cyan. Large diffs are capped at 60 lines with a "... N more line(s)" note.

Exit codes: `0` once the run completes by default, even when the agent scores `FAILED`. Use `--fail-on-failure` to exit `1` on a failed run (required for CI gates). Exit `1` also when the run itself throws (e.g. a sandbox or provider error).

### Options

| Flag | Default | Description |
|---|---|---|
| `<testCase>` | Required | Path to an `agr.yaml` file, a directory containing one, or a test case name/directory basename (see above). |
| `--config <path>` | Required if not in agr.yaml | Path to an agent config YAML. Required when the test case's `agr.yaml` does not specify `agent_config:`. |
| `--model <model>` | (none) | Override the model from the agent config for this run only (e.g. `claude-opus-4-8`). Useful for quick one-off comparisons without editing YAML. |
| `--provider <provider>` | (none) | Override the provider from the agent config for this run only (e.g. `anthropic`, `openai`, `openrouter`). Combine with `--model` to switch provider and model in one command. |
| `--temperature <n>` | (none) | Override the temperature from the agent config for this run only (0.0-1.0). Use `--temperature 0` for deterministic runs when debugging flakiness. |
| `--max-steps <n>` | (none) | Override `max_steps` from the agent config for this run only. Useful for budget-capped smoke tests (`--max-steps 5`) or extended runs (`--max-steps 50`). |
| `--adapter <name>` | `ai-sdk` | Agent adapter: `ai-sdk` (default AI SDK loop) or `acp` (external ACP agent). See [ACP Agent Adapter](/advanced/acp-agent). |
| `--verbose` | `false` | Show full per-step detail (tool name + content preview) in the live step list, instead of the compact step/cost counter. |
| `--repeat <n>` | (none) | Run the test case N times sequentially and print a solve-rate summary. Useful for flakiness testing or measuring statistical consistency of a fix before scaling up with `agr bench`. |
| `--fail-on-failure` | `false` | Exit with code 1 if the run does not pass. With `--repeat`, exits 1 if any run fails. |
| `--report <format>` | (none) | Write a report after the run (`json`, `jsonl`, `html`, `md`). |
| `--output <path>` | (none) | Output path for `--report`. |
| `--report-dir <dir>` | (none) | Directory for auto-named report files. Generates `run-<timestamp>.<ext>` when `--output` is not given. Useful in CI to always archive reports without hardcoding a filename. |
| `--report-include-traces` | `false` | Include full step traces in `--report` output. |
| `--sandbox <provider>` | `docker` | Sandbox provider: `docker` (local Docker) or `e2b` (E2B cloud). See [Custom Sandbox](/advanced/custom-sandbox). |
| `--llm-judge` | `false` | Run `LlmJudgeScorer` after the agent completes. |
| `--llm-judge-provider <name>` | `anthropic` | LLM judge provider: `anthropic` or `openai`. |
| `--llm-judge-model <model>` | (provider default) | Model slug for the LLM judge. |
| `--judge-gate` | `false` | Fail the run when the LLM judge score is below `--judge-min-score`. |
| `--judge-min-score <score>` | `0.7` | Minimum normalized judge score when `--judge-gate` is set. |
| `--step-timeout <ms>` | (none) | Override `step_timeout_ms` from the agent config for this run only. Sets the per-LLM-call abort timeout in milliseconds. Useful in CI to cap provider latency without editing YAML (default is 120000). |
| `--save-baseline <path>` | (none) | Write a baseline snapshot JSON after the run completes. Same format as `agr bench --save-baseline`; use `agr compare-baseline --current <path>` to compare. Also works with `--repeat N` to save all N runs as a multi-run snapshot. |
| `--json` | `false` | Output the run result as a single JSON object and suppress the live Ink UI. Useful for scripting and CI pipelines. |

### JSON output

With `--json`, the live dashboard is skipped and a single JSON line is printed to stdout:

```json
{
  "passed": true,
  "runId": "3fa85f64-...",
  "testCaseId": "hello-world",
  "agentConfigId": "my-agent",
  "model": "claude-haiku-4-5-20251001",
  "costUsd": 0.0012,
  "durationMs": 4200,
  "stepsCount": 3,
  "metrics": null,
  "error": null
}
```

`passed` is `true` (scored pass), `false` (scored fail), or `null` (run threw an error). On error the object contains only `passed: null`, `runId`, and `error`.

With `--repeat N --json`, the output is a summary object instead:

```json
{
  "testCaseId": "hello-world",
  "agentConfigId": "my-agent",
  "model": "claude-haiku-4-5-20251001",
  "repeat": 5,
  "passedRuns": 4,
  "totalRuns": 5,
  "solveRate": 0.8,
  "totalCostUsd": 0.006,
  "avgCostUsd": 0.0012,
  "avgDurationMs": 4200,
  "runs": [
    { "runId": "...", "passed": true, "costUsd": 0.0012, "durationMs": 4100 }
  ]
}
```

### Examples

```bash
# Name-based lookup (resolves tasks/fix-greeting/agr.yaml automatically)
agr run fix-greeting --config agent.yaml

# Watch tool calls and messages in real time
agr run fix-greeting --config agent.yaml --verbose

# Run 5 times to measure flakiness before scaling with bench
agr run fix-greeting --config agent.yaml --repeat 5

# External ACP agent (Claude Code, Cursor Agent, ...)
agr run fix-greeting --config agent-acp.yaml --adapter acp

# CI gate: fail the job when the agent does not pass
agr run fix-greeting --config agent.yaml --fail-on-failure

# LLM judge with a pass/fail gate
agr run fix-greeting --config agent.yaml --llm-judge --judge-gate --judge-min-score 0.75

# Machine-readable output for scripting (single run)
result=$(agr run hello-world --json)
echo "$result" | jq .passed

# Flakiness measurement with JSON output (5 runs)
agr run hello-world --repeat 5 --json | jq .solveRate

# Save a baseline for later PR comparison
agr run hello-world --config agent.yaml --save-baseline baselines/main.json
agr compare-baseline --current baselines/main.json --format md --output comment.md
```

## `agr bench`

Run test cases against one or more agent configs (or an optimizer matrix). Compare solve rates, cost, and quality across configs. Shows a live terminal dashboard and runs evaluations in parallel.

`<testCases>` accepts the same forms as [`agr run`](#agr-run): a path to an `agr.yaml` file, a directory containing one, or a test case name/directory basename resolved by searching the current directory. When positional names are given without `--suite`, each is resolved individually. When `--suite` is given alongside positional names, the suite is loaded and then filtered to the named subset.

```bash
agr bench hello-world --matrix matrix.yaml
agr bench task-a task-b --configs agent.yaml
agr bench --suite tasks/ --configs agent.yaml,agent-alt.yaml --concurrency 2
agr bench --suite tasks/ --configs-dir agents-configs/
agr bench --manifest bench.yaml
```

### Options

| Flag | Default | Description |
|---|---|---|
| `[...testCases]` | (all in suite) | One or more test case names, directories, or `agr.yaml` paths to run. When given without `--suite`, each is resolved via name search (see above). |
| `--manifest <path>` | (none) | Bench manifest YAML with `suite` and `agents` (paths/glob). Replaces `--suite` + `--configs` on the CLI. |
| `--suite <path>` | Required without positional args or `--manifest` | Directory containing test case folders (each with an `agr.yaml`). |
| `--configs <paths>` | One agent source required | Comma-separated paths to agent config YAML files. |
| `--config <path>` | (none) | Alias for `--configs` when you have a single agent config file. |
| `--configs-dir <dir>` | (none) | Load every `.yaml`/`.yml` file in the directory as an agent config. |
| `--matrix <path>` | (none) | Optimizer matrix YAML. Expands into the cartesian product of agent configs, tags runs with a shared `matrixId`, and prints a Pareto summary. See [Core Concepts: Optimizer matrices](/guide/concepts#optimizer-matrices). |
| `--adapters <names>` | `ai-sdk` | Comma-separated adapter names (`ai-sdk`, `acp`). Runs the full config matrix once per adapter. |
| `--concurrency <n>` | `2` | Number of parallel sandbox executions. Overrides manifest `concurrency` when set. |
| `--fail-on-failure` | `false` | Exit with code 1 if any run in the comparison sweep fails. |
| `--min-solve-rate <rate>` | (none) | Exit with code 1 if solve rate is below this threshold (0–1). |
| `--min-solve-rate-scope <scope>` | `global` | Apply `--min-solve-rate` globally or per agent config (`global`, `per-config`). |
| `--report <format>` | (none) | Write a report after the bench (`json`, `jsonl`, `html`, `md`). |
| `--output <path>` | (none) | Output path for `--report`. |
| `--report-dir <dir>` | (none) | Directory for auto-named report files. Generates `bench-<timestamp>.<ext>` when `--output` is not given. Useful in CI to always archive reports without hardcoding a filename. |
| `--report-include-traces` | `false` | Include full step traces in `--report` output. |
| `--save-baseline <path>` | (none) | Write a baseline snapshot JSON after the bench completes. |
| `--sandbox <provider>` | `docker` | Sandbox provider: `docker` or `e2b`. |
| `--strict-toolkits` | `false` | Exit with code 1 if any referenced toolkit fails the security audit. |
| `--llm-judge` | `false` | Run `LlmJudgeScorer` on each completed run. |
| `--llm-judge-provider <name>` | `anthropic` | LLM judge provider: `anthropic` or `openai`. |
| `--llm-judge-model <model>` | (provider default) | Model slug for the LLM judge. |
| `--judge-gate` | `false` | Fail runs when the LLM judge score is below `--judge-min-score`. |
| `--judge-min-score <score>` | `0.7` | Minimum normalized judge score when `--judge-gate` is set. |
| `--dry-run` | `false` | Print the resolved test case x config matrix and exit without starting any runs. Tags (if any) are shown inline per test case. Combine with `--json` to emit `{testCases, agentConfigs, totalRuns, concurrency}` for scripting. |
| `--tags <tags>` | (none) | Comma-separated list of tags; only test cases whose `tags:` list matches at least one are included. Requires `--suite`. |
| `--skip-tags <tags>` | (none) | Comma-separated list of tags; test cases with any of these tags are excluded. Applied after `--tags`. Requires `--suite`. |
| `--limit <n>` | (none) | Run only the first N test cases after filtering. Useful for quick smoke tests on large suites without running the full set. |
| `--only-failed` | `false` | Run only the test cases that failed on their most recent run in the DB. Useful for tight fix-and-retry loops: bench the full suite once, fix failing cases, then re-run only those with `--only-failed`. Exits cleanly if all previously-failed cases have since passed. |
| `--shuffle` | `false` | Randomize the order of test cases before running. Reduces order-dependent bias in large suites and helps surface flaky tests that only fail when run after certain other tests. |
| `--model <model>` | (none) | Override the model for all agent configs in this bench run (e.g. `claude-opus-4-8`). Useful for quick model comparisons without editing YAML or creating a new agent config file. |
| `--provider <provider>` | (none) | Override the provider for all agent configs in this bench run (e.g. `anthropic`, `openai`, `openrouter`). Combine with `--model` to switch provider and model without editing YAML. |
| `--temperature <n>` | (none) | Override the temperature for all agent configs in this bench run (0.0-1.0). Use `--temperature 0` for deterministic reproducibility experiments. |
| `--repeat <n>` | `1` | Run each test case N times per config. The result summary shows solve rate across all trials (e.g. `--repeat 5` on 3 test cases = 15 runs per config). Useful for pass@k metrics and detecting flaky tests. |
| `--max-steps <n>` | (none) | Override `max_steps` for all agent configs in this bench run. Combine with `--limit` for fast, cheap smoke tests: `--limit 3 --max-steps 5`. |
| `--name <substring>` | (none) | Filter test cases by name substring (case-insensitive). Applied after `--tags` and `--skip-tags`. Requires `--suite`. |
| `--step-timeout <ms>` | (none) | Override `step_timeout_ms` for all agent configs in this bench run. Sets the per-LLM-call abort timeout in milliseconds. Useful in CI to cap provider latency without editing YAML (default is 120000). |
| `--json` | `false` | Output bench results as a single JSON object and suppress the live dashboard. Useful for scripting and CI pipelines. |

Use only **one** agent source per run: `--manifest`, `--configs`/`--config`, `--configs-dir`, or `--matrix`.

Exit codes: `0` by default after a completed bench. Use `--fail-on-failure`, `--min-solve-rate`, and/or `--strict-toolkits` for CI gates. See [CI Integration](/advanced/ci-integration).

### Examples

```bash
# Run a single named test case with a matrix (no --suite needed)
agr bench hello-world --matrix matrix.yaml

# Run two named test cases against a specific agent config
agr bench task-a task-b --configs agent.yaml

# Filter a suite to one test case
agr bench hello-world --suite tasks/ --configs agent.yaml

# Bench manifest (suite + agent glob in one file)
agr bench --manifest bench.yaml

# All agent YAMLs in a folder
agr bench --suite tasks/ --configs-dir agents-configs/

# Single agent config (--config is an alias for --configs)
agr bench --suite tasks/ --config agent.yaml

# Multiple agent configs
agr bench --suite tasks/ --configs agent.yaml,agent-openrouter.yaml

# ACP agent adapter
agr bench --suite tasks/ --configs agent-acp.yaml --adapters acp

# Compare AI SDK loop vs ACP agent
agr bench --suite tasks/ --configs agent.yaml,agent-acp.yaml --adapters ai-sdk,acp

# Optimizer matrix sweep
agr bench --suite tasks/ --matrix matrix.yaml

# Higher parallelism
agr bench --manifest bench.yaml --concurrency 4

# CI gate with report artifact
agr bench --suite tasks/ --config agent.yaml \
  --fail-on-failure --min-solve-rate 0.8 \
  --report json --output reports/bench.json

# Save baseline snapshot for PR comparison
agr bench --suite tasks/ --config agent.yaml --save-baseline baselines/main.json

# Preview what would run (no sandbox, no API calls)
agr bench --suite tasks/ --matrix matrix.yaml --dry-run

# Run only test cases tagged "python" or "fast"
agr bench --suite tasks/ --config agent.yaml --tags python,fast

# Machine-readable output for scripting
result=$(agr bench --suite tasks/ --config agent.yaml --json)
echo "$result" | jq .solveRate
```

### JSON output

With `--json`, the dashboard is suppressed and a single JSON object is printed to stdout:

```json
{
  "passed": false,
  "passedRuns": 2,
  "totalRuns": 3,
  "solveRate": 0.6667,
  "totalCostUsd": 0.0045,
  "elapsedMs": 12400,
  "matrixId": null,
  "gateReasons": [],
  "byConfig": [
    { "configId": "my-agent", "passedRuns": 2, "totalRuns": 3, "solveRate": 0.6667, "totalCostUsd": 0.0045 }
  ],
  "runs": [
    { "runId": "3fa85f64-...", "testCaseId": "hello-world", "agentConfigId": "my-agent", "passed": true, "costUsd": 0.0012, "durationMs": 4200, "stepsCount": 3, "error": null }
  ]
}
```

`passed` is `true` only when all runs passed. `gateReasons` is a non-empty array when `--fail-on-failure` or `--min-solve-rate` triggers; the exit code still reflects the gate result.

See [Bench Manifest YAML](/reference/bench-manifest-yaml) for the manifest file format.

Every bench run is also scored by `StaticQualityScorer` (diff size, lint violations, etc.). See [Quality scorers](/guide/concepts#quality-scorers-and-the-optimizer).

After the dashboard finishes, `agr bench` also prints a **TOOL USAGE BY CONFIG** block: per agent config, aggregated counts of each `tool_call` name across all runs in that bench session. Use this to compare toolkit adoption between configs (for example default tools vs. a custom `toolkits` matrix dimension) without calling `agr trace <runId> --tools` on every cell. Per-run detail remains available via `agr trace --tools`.

## `agr validate`

Validate one or more test cases the way SWE-bench validates a candidate instance: static checks, then (when `test_command` is set) pre-patch and post-patch execution inside Docker.

```bash
agr validate fix-greeting
agr validate task-a task-b task-c --strict
```

Each `<testCase>` accepts the same forms as [`agr run`](#agr-run): a path to an `agr.yaml` file, a directory containing one, or a test case name/directory basename resolved by searching the current directory.

When `test_command` is missing, execution checks are skipped (shown with ⚠️). Use `--strict` in CI to require `test_command`, `fail_to_pass`, and `pass_to_pass`.

### Options

| Flag | Default | Description |
|---|---|---|
| `[...testCases]` | Required unless `--suite` | One or more paths, directories, or test case names. When multiple are given, each is validated in turn; exits 1 if any fail. |
| `--suite <dir>` | (none) | Validate every test case found recursively under this directory. Alternative to listing names as arguments. |
| `--strict` | `false` | Exit with code 1 if `test_command`, `fail_to_pass`, or `pass_to_pass` are missing. |
| `--sandbox <provider>` | `docker` | Sandbox provider used for execution checks: `docker` or `e2b`. |
| `--audit-toolkits` | `false` | Run the toolkit security audit on every `toolkits:` path referenced by the test case. |
| `--tags <tags>` | (none) | Comma-separated list of tags; only validate test cases whose `tags:` list contains at least one match. Requires `--suite`. |
| `--name <substring>` | (none) | Filter test cases by name substring (case-insensitive). Applied after `--tags`. Requires `--suite`. |
| `--json` | `false` | Output results as a single JSON object and suppress per-check console output. |

### JSON output

With `--json`, a single JSON object is printed to stdout:

```json
{
  "passed": true,
  "passedCount": 2,
  "totalCount": 2,
  "results": [
    {
      "ok": true,
      "name": "hello-world",
      "path": "/tasks/hello-world/agr.yaml",
      "checks": [
        { "name": "required-fields", "passed": true, "detail": "ok" }
      ]
    }
  ]
}
```

### Examples

```bash
# Static + execution checks (when test_command is configured)
agr validate my-case
# On success prints: Next: agr run my-case  |  agr bench my-case

# CI gate: reject incomplete definitions
agr validate my-case --strict

# Validate multiple named test cases at once
agr validate task-a task-b task-c --strict

# Validate everything in the tasks/ directory
agr validate --suite tasks/ --strict
# On success prints: Next: agr bench --suite tasks/  |  agr bench --suite tasks/ --matrix matrix.yaml

# Validate only the python subset
agr validate --suite tasks/ --tags python --strict

# Machine-readable output for scripting
agr validate fix-greeting --json | jq .passed
```

## `agr import-pr`

Fetch a GitHub pull request diff, split it into solution and test patches, and scaffold a new test case with `expected_files` and `forbid_modified` pre-filled.

```bash
agr import-pr owner/repo 123 --out tasks/new-case --clone-fixture
```

With `--clone-fixture`, the repo is checked out at the PR's base commit into `<out>/fixture`, and `success`/`test_command` are guessed from the project layout (Python, Node, Go). You still need to fill in `fail_to_pass` and `pass_to_pass` manually before `agr validate` can verify execution.

Set `GITHUB_TOKEN` to avoid GitHub's low unauthenticated rate limits.

### Options

| Flag | Default | Description |
|---|---|---|
| `<repo>` | Required | GitHub repository in `owner/repo` format. |
| `<prNumber>` | Required | Pull request number. |
| `--out <dir>` | `./imported/<repo>-pr-<n>` | Output directory for the scaffolded test case. |
| `--clone-fixture` | `false` | Clone the repo at the PR's base commit into `<out>/fixture`. |
| `--validate` | `false` | Run `agr validate` after scaffolding. Most useful after filling in test name lists. |

### Examples

```bash
# Scaffold patches and agr.yaml only
agr import-pr astropy/astropy 12907 --out tasks/astropy-12907

# Clone fixture and guess test commands
agr import-pr astropy/astropy 12907 --clone-fixture --out tasks/astropy-12907

# Scaffold and validate (after filling in fail_to_pass/pass_to_pass)
agr import-pr astropy/astropy 12907 --clone-fixture --validate
```

## `agr trace`

Print the step trace and metrics for a single run, looked up by run ID (shown in bench/run output and stored in `.agr/db.sqlite`). Use `--last` to skip the ID lookup entirely after `agr run`.

```bash
agr trace 3f1c2e2a-8b4d-4e1f-9c3a-1a2b3c4d5e6f
agr trace --last
```

### Options

| Flag | Default | Description |
|---|---|---|
| `[runId]` | Required unless `--last` | UUID of the run to inspect. |
| `--last` | `false` | Trace the most recent run in `.agr/db.sqlite`. Overrides `[runId]` if both are given. |
| `--test-case <name>` | (none) | With `--last`, trace the most recent run for this specific test case (substring match on `testCaseId`). |
| `--config <name>` | (none) | With `--last`, trace the most recent run for this specific agent config (substring match on `agentConfigId`). |
| `--quality` | `false` | Show only the quality-metrics breakdown (`static-quality`, `llm-judge`, diff, localization) instead of the full step trace. |
| `--tools` | `false` | Show only a tool-usage breakdown: how many times each tool name appears across the run's `tool_call` steps, sorted by call count. |
| `--json` | `false` | Output as a JSON object. Default mode emits `{run, steps[]}` (each step has `stepIndex`, `kind`, `tool`, `content`, `tokensIn`, `tokensOut`, `cachedTokens`, `costUsd`). With `--quality` emits `{run, metrics}`. With `--tools` emits `{run, toolUsage}`. |

### Examples

```bash
# Full step-by-step trace (most recent run)
agr trace --last

# Tool-usage breakdown for the most recent run
agr trace --last --tools

# Trace the most recent run of a specific test case
agr trace --last --test-case hello-world

# Trace the most recent run of a specific agent config
agr trace --last --config agent-a

# Full step-by-step trace by ID
agr trace 3f1c2e2a-8b4d-4e1f-9c3a-1a2b3c4d5e6f

# Quality metrics only
agr trace 3f1c2e2a-8b4d-4e1f-9c3a-1a2b3c4d5e6f --quality

# Tool-usage breakdown (which tools the agent actually called, and how often)
agr trace 3f1c2e2a-8b4d-4e1f-9c3a-1a2b3c4d5e6f --tools
```

`--tools` is especially useful for checking whether a custom toolkit, MCP server, or Agent Skill was actually used by the agent versus only made available to it. For example, if you wire in a custom toolkit but its tools show a count of `0`, the agent saw the tool but chose not to call it, which may be a sign the system prompt needs to nudge it more explicitly.

## `agr list`

Browse saved runs from `.agr/db.sqlite` in an interactive terminal UI: a scrollable run list, a detail view (run metadata, agent diff, trace preview), and a side-by-side diff compare between any two runs.

```bash
agr list
```

### Options

| Flag | Default | Description |
|---|---|---|
| `--db <path>` | `.agr/db.sqlite` | Path to the SQLite database to read. |
| `--limit <n>` | `100` | Maximum number of most-recent runs to load. |
| `--plain` | `false` | Print a plain text list instead of the interactive UI. Used automatically when stdout is not a TTY. |
| `--since <duration\|date>` | (none) | Only show runs after this point. Accepts relative durations (`1h`, `24h`, `7d`) or ISO timestamps. Applied before `--limit`. |
| `--test-case <name>` | (none) | Only show runs for this specific test case (substring match on `testCaseId`). Applied before `--limit`. |
| `--config <name>` | (none) | Only show runs for this specific agent config (substring match on `agentConfigId`). Applied before `--limit`. |
| `--passed` | `false` | Only show runs that passed. Mutually exclusive with `--failed`. |
| `--failed` | `false` | Only show runs that failed. Mutually exclusive with `--passed`. |
| `--model <substring>` | (none) | Only show runs where the agent model contains this substring (case-insensitive). Useful for comparing run histories across model versions (e.g. `--model haiku`). |
| `--sort <field>` | `date` | Sort runs by `date` (newest first, default), `cost` (highest first), `duration` (longest first), or `steps` (most steps first). Applied after all filters and `--limit`. |
| `--json` | `false` | Output runs as a JSON array and suppress plain-text and TUI output. Each element includes `id`, `testCaseId`, `testCaseName`, `agentConfigId`, `agentConfigName`, `agentModel`, `passed`, `costUsd`, `durationMs`, `stepsCount`, `tokensIn`, `tokensOut`, `error`, `matrixId`, `createdAt`, `completedAt`. Combinable with all filters. |

### Examples

```bash
# Open the interactive run browser
agr list

# Print a plain text summary of the 20 most recent runs
agr list --limit 20 --plain

# Show only runs from the last 24 hours
agr list --plain --since 24h

# Show all runs for a specific test case
agr list --plain --test-case hello-world

# Show all runs for a specific agent config
agr list --plain --config agent-fast

# Show only failing runs (for triage)
agr list --plain --failed

# Show most expensive runs first
agr list --plain --sort cost

# Find the slowest runs in the last week
agr list --plain --since 7d --sort duration
```

```bash
# Machine-readable JSON for scripting
agr list --json | jq '.[].testCaseId'
agr list --json --failed | jq 'length'
agr list --json --since 24h | jq '[.[] | {id, testCaseId, passed, costUsd}]'
agr list --json --sort cost | jq '.[0:5] | [.[] | {id, testCaseId, costUsd}]'
```

In the interactive UI, use the arrow keys (or `j`/`k`) to move through the run list, `Enter` to open a run's detail view (agent diff plus a trace preview), `c` to start a diff comparison between two runs, `b`/`Esc` to go back, and `q` to quit.

## `agr compare`

Compare the step traces of two completed runs side by side. Useful after a matrix or bench run with multiple agent configs on the same test case: see when and where the agents diverged (different tool calls, files, or reasoning).

```bash
agr compare 3f1c2e2a-8b4d-4e1f-9c3a-1a2b3c4d5e6f 7a8b9c0d-1e2f-3a4b-5c6d-7e8f9a0b1c2d
```

Run IDs come from bench/run output or the `runs` table in `.agr/db.sqlite`. Both runs should usually share the same `test_case_id` so step indices align.

### Options

| Flag | Default | Description |
|---|---|---|
| `[runIdA]` | Required unless `--last-two` | First run to compare (shown as column A). |
| `[runIdB]` | Required unless `--last-two` | Second run to compare (shown as column B). |
| `--last-two` | `false` | Compare the two most recent runs without specifying IDs. |
| `--test-case <name>` | (none) | With `--last-two`, scope to the two most recent runs for this specific test case (substring match on `testCaseId`). |
| `--config <name>` | (none) | With `--last-two`, scope to the two most recent runs for this specific agent config (substring match on `agentConfigId`). |
| `--full` | `false` | Print full step content without the 200-character truncation used by `agr trace`. |
| `--only-diff` | `false` | Show only divergent steps, plus one step of context before and after each divergence. |
| `--json` | `false` | Output the comparison as a single JSON object `{runA, runB, divergentCount, totalSteps, firstDivergence, steps[]}`. Each step has `{index, divergent, a, b}` where `a`/`b` are `{kind, tool, content}` or `null`. |

### Examples

```bash
# Full side-by-side comparison
agr compare <runIdA> <runIdB>

# Only where the runs diverged (with 1-step context)
agr compare <runIdA> <runIdB> --only-diff

# Full content for divergent steps
agr compare <runIdA> <runIdB> --only-diff --full

# Compare the two most recent runs (no IDs needed)
agr compare --last-two --only-diff

# Compare the two most recent runs of a specific config
agr compare --last-two --config agent-a

# Compare the two most recent runs of a specific test case
agr compare --last-two --test-case hello-world

# Machine-readable divergence check for CI
agr compare --last-two --json | jq .divergentCount
```

## `agr compare-baseline`

Compare two baseline snapshot JSON files, or compare the most recent runs in `.agr/db.sqlite` against a saved baseline. Useful for PR comments and regression gates. See [CI Integration: Baseline comparison](/advanced/ci-integration#baseline-comparison-in-pull-requests).

```bash
agr compare-baseline baselines/main.json baselines/pr.json
agr compare-baseline --current baselines/main.json --format md --output comment.md
```

### Options

| Flag | Default | Description |
|---|---|---|
| `[snapshotA]` | (none) | Path to the baseline snapshot (older/reference). |
| `[snapshotB]` | (none) | Path to the current snapshot (newer/candidate). |
| `--current <path>` | (none) | Compare recent runs in the SQLite DB against this baseline snapshot instead of two files. |
| `--format <format>` | `md` | Output format: `md` (PR comment markdown) or `json`. |
| `--output <path>` | (none) | Write comparison to a file instead of stdout. |
| `--db <path>` | `.agr/db.sqlite` | SQLite database for `--current`. |
| `--fail-on-regression` | `false` | Exit with code 1 if solve rate dropped or any previously passing case regressed. |

### Examples

```bash
# Side-by-side snapshot diff on stdout
agr compare-baseline baselines/main.json baselines/pr-42.json

# PR comment markdown from the latest bench run in the DB
agr compare-baseline --current baselines/main.json --format md --output comment.md

# Fail CI when regressions are detected
agr compare-baseline --current baselines/main.json --fail-on-regression
```

Pair with `agr bench --save-baseline <path>` to produce snapshot files.

## `agr validate-toolkit`

Run the toolkit security audit on a toolkit directory (`bin/` scripts and `.claude/skills/`). Reports risky patterns (network calls, shell escapes, missing skill docs).

```bash
agr validate-toolkit ./toolkits/jetbrains-tools
agr validate-toolkit ./toolkits/jetbrains-tools --strict
```

### Options

| Flag | Default | Description |
|---|---|---|
| `<dir>` | Required | Path to the toolkit directory. |
| `--strict` | `false` | Exit with code 1 on warnings as well as errors. |
| `--json` | `false` | Output the audit result as a JSON object `{dir, passed, findings[]}` with `{file, severity, rule, message}` per finding. Exits with code 1 on failure same as human mode. |

`agr bench --strict-toolkits` and `agr validate --audit-toolkits` use the same audit logic across referenced toolkits.

## `agr export`

Export run metadata or step traces from `.agr/db.sqlite` for downstream analytics or observability pipelines.

```bash
agr export runs --format jsonl --output runs.jsonl
agr export runs --format csv --output runs.csv
agr export traces --run-id <runId> --format otlp --output trace.json
agr export traces --test-case hello-world --format jsonl --output hello-traces.jsonl
agr export traces --last --test-case hello-world --format otlp --output last-hello.json
```

### Subcommands

| Subcommand | Description |
|---|---|
| `runs` | Export run rows with optional filters. |
| `traces` | Export step traces. Use `--run-id` or `--last` for a single run, or `--test-case` / `--config` to export traces for all matching runs at once. |

### Options

| Flag | Default | Description |
|---|---|---|
| `--format <format>` | `json` | Export format: `json`, `jsonl`, `csv` (runs only), or `otlp` (traces only). |
| `--output <path>` | auto-named | Output file path. |
| `--db <path>` | `.agr/db.sqlite` | SQLite database to read. |
| `--run-id <id>` | (none) | Run UUID for `export traces` (single-run). |
| `--last` | `false` | Export the most recent run's traces. Combines with `--test-case` and `--config` to scope to the most recent run matching those filters. |
| `--matrix-id <id>` | (none) | Filter `export runs` to a bench matrix id. |
| `--last-matrix` | `false` | Export runs for the most recent matrix sweep (no `--matrix-id` needed). |
| `--limit <n>` | (none) | Maximum number of runs to export. |
| `--since <duration\|date>` | (none) | Filter to runs created after this point. Accepts relative durations (`1h`, `24h`, `7d`) or ISO timestamps (`2026-06-15T00:00:00Z`). Works for both `runs` and multi-run `traces` export. |
| `--test-case <id>` | (none) | Filter by `testCaseId` (substring match). For `export runs`: filter run rows. For `export traces`: export traces for all matching runs (or scope `--last` to the most recent matching run). |
| `--config <id>` | (none) | Filter by `agentConfigId` (substring match). Works for both `runs` and `traces`. |
| `--passed` | `false` | Export only runs that passed. Mutually exclusive with `--failed`. Works for both `runs` and multi-run `traces`. |
| `--failed` | `false` | Export only runs that failed. Mutually exclusive with `--passed`. Works for both `runs` and multi-run `traces`. |

**Multi-run trace export:** when `--test-case` or `--config` is given without `--run-id` or `--last`, `export traces` fetches traces for every matching run. JSON output is an array of `{ runId, testCaseId, agentConfigId, passed, resourceSpans }` objects; JSONL output is one OTel JSON per line.

Each `export runs` record includes: `id`, `testCaseId`, `agentConfigId`, `passed`, `costUsd`, `durationMs`, `stepsCount`, `tokensIn`, `tokensOut`, `matrixId`, and `metrics`. For `--format csv`, all fields are included as columns; `metrics` is JSON-serialized in the cell.

Set `AGR_EXPORT_ON_BENCH=true` to auto-export run JSON after each `agr bench` completes (written under `.agr/exports/`).

## `agr toolkit-add`

Scaffold a new toolkit tool: creates a `bin/<name>` shell script stub and a `.claude/skills/<name>/SKILL.md` description template in the target toolkit directory.

```bash
agr toolkit-add find-usages
agr toolkit-add run-tests --dir ./toolkits/jetbrains-tools
```

### Options

| Flag | Default | Description |
|---|---|---|
| `<name>` | Required | Tool name (lowercase letters, digits, and hyphens). |
| `--dir <dir>` | `./toolkit` | Toolkit directory to scaffold into. |

After scaffolding, implement `bin/<name>` and fill in the SKILL.md description, then reference the toolkit directory from `toolkits:` in an agent config or test case. See [Toolkits](/guide/toolkits) for the full layout and conventions.

## `agr toolkit-list`

List every `bin/` tool in a toolkit directory alongside its SKILL.md description. Also runs the toolkit security audit. Useful for auditing what tools are actually present before referencing the toolkit in a bench.

```bash
agr toolkit-list ./toolkits/jetbrains-tools
agr toolkit-list ./toolkits/jetbrains-tools --check-config matrix-jetbrains-toolkits.yaml
```

### Options

| Flag | Default | Description |
|---|---|---|
| `<dir>` | Required | Path to the toolkit directory (must contain `bin/`). |
| `--check-config <file>` | (none) | Diff the toolkit's `bin/` tools against an agent config's `track_tools` and `require_tools_before_submit` lists. Reports tools that are present in `bin/` but not tracked, and tracked names that are missing from `bin/`. |

The `--check-config` flag accepts both agent config YAMLs (top-level `track_tools`) and matrix YAMLs (`base.track_tools`). This catches the common drift where a new toolkit tool is added to `bin/` but forgotten in one or more agent configs.

## `agr doctor`

Run a pre-flight check of the local environment before your first comparison sweep. Checks Docker, API keys, the run database, an `agent.yaml`, and at least one test case.

```bash
agr doctor
agr doctor --suite my-tasks/
agr doctor --json | jq .passed
```

Each check prints a status icon:

- `✓` pass
- `!` warning (non-fatal, but worth reviewing)
- `✗` fail (exits 1)
- `-` skipped (optional prerequisite not present)

Checks performed:

| Check | Failure condition |
|---|---|
| Docker daemon | `docker info` fails or Docker is not installed |
| `ANTHROPIC_API_KEY` | Env var not set (warning, not fatal) |
| `OPENAI_API_KEY` | Not set (skipped, only needed for `provider: openai` or LLM judge) |
| `E2B_API_KEY` | Not set (skipped, only needed for `--sandbox e2b`) |
| Database (`--db`) | `.agr/db.sqlite` not found (warning -- will be created on first run) |
| Agent config | `agent.yaml` not found in cwd (warning) |
| Test cases (`--suite`) | No `agr.yaml` found under the suite directory (warning) |
| Runtime | Always passes; shows Node.js version |

### Options

| Flag | Default | Description |
|---|---|---|
| `--db <path>` | `.agr/db.sqlite` | Database path to check. |
| `--suite <dir>` | `tasks` | Suite directory to scan for `agr.yaml` files. |
| `--json` | `false` | Output a single JSON object (`passed`, `failureCount`, `warningCount`, `checks[]`) and suppress human-readable output; exits 1 when any check has `fail` status. |

With `--json`, each entry in `checks` has `label`, `status` (`pass` \| `fail` \| `warn` \| `skip`), and optional `detail`.

## `agr cleanup`

Lists (or, with `--yes`, removes) leftover sandbox containers from runs whose process exited or was killed before the `cleanup` workflow step could call `destroy()` - for example a hung provider request that an external `timeout` had to kill. These show up as containers running `tail -f /dev/null`, labeled `agentgrader.sandbox=true` by `DockerSandboxProvider`.

```bash
agr cleanup
agr cleanup --yes
```

### Options

| Flag | Default | Description |
|---|---|---|
| `--yes` | `false` | Actually remove the listed containers. Without it, `agr cleanup` only lists what would be removed. |
| `--json` | `false` | Output result as a JSON object `{found, removed, containers[]}` with `{id, image, status, ageMs}` per container. Without `--yes`, `removed` is always 0; with `--yes`, each container entry has a `removed` boolean and optional `error`. |

Set `step_timeout_ms` in `agent.yaml` (see [Agent Config: `step_timeout_ms`](/reference/agent-config-yaml#step-timeout-ms)) to prevent new leftovers in the first place - `agr cleanup` is for sweeping up containers from runs that predate that fix, or from any other interrupted run.

Containers created before this label was added (older `@agentgrader/sandbox-docker` versions) won't be found by `agr cleanup`; remove those manually with `docker ps -a` / `docker rm -f`.

## `agr status`

Print a quick summary of the local run database without launching the interactive TUI. Useful as a health-check in scripts or CI logs.

```bash
agr status
agr status --json                     # machine-readable output
agr status --since 24h                # stats for just the last 24 hours
agr status --test-case hello-world    # solve rate and avg cost for one task
agr status --failed                   # count and cost of failing runs only
agr status --by-config                # per-config breakdown sorted by solve rate
agr status --by-config --test-case hello-world  # per-config for one task
agr status --by-test-case             # per-task breakdown, hardest first
```

Output includes:

- Total runs with pass/fail/error breakdown and solve rate
- Number of unique test cases and agent configs
- Total and average cost, average duration
- Total accumulated token usage (when available)
- Timestamp of the most recent run

### Options

| Flag | Default | Description |
|---|---|---|
| `--db <path>` | `.agr/db.sqlite` | SQLite database to read. |
| `--json` | off | Emit JSON to stdout instead of formatted text. |
| `--since <duration\|date>` | (none) | Restrict stats to runs after this point. Accepts relative durations (`1h`, `24h`, `7d`) or ISO timestamps. |
| `--test-case <name>` | (none) | Restrict stats to runs for this specific test case (substring match). Shows solve rate, avg cost, and avg duration for that task. |
| `--config <name>` | (none) | Restrict stats to runs for this specific agent config (substring match). Useful for comparing performance between two configs. |
| `--passed` | `false` | Restrict stats to runs that passed. Mutually exclusive with `--failed`. |
| `--failed` | `false` | Restrict stats to runs that failed. Mutually exclusive with `--passed`. |
| `--by-config` | `false` | Show a per-config breakdown: solve rate, avg cost, avg duration, and avg tokens per agent config, sorted by solve rate. Combinable with `--since` and `--test-case` to scope the data. |
| `--by-test-case` | `false` | Show a per-test-case breakdown: solve rate, avg cost, avg duration, sorted by solve rate ascending (hardest first). Combinable with `--since` and `--config` to scope the data. |
| `--top <n>` | (none) | With `--by-config` or `--by-test-case`, show only the first N entries. |

The `--json` output contains: `exists`, `dbPath`, `since`, `testCase`, `config`, `passed`, `totalRuns`, `passedRuns`, `failedRuns`, `erroredRuns`, `solveRate`, `uniqueTestCases`, `uniqueConfigs`, `matrixRuns`, `totalCostUsd`, `avgCostUsd`, `avgDurationMs`, `totalTokensIn`, `totalTokensOut`, `lastRunAt`, `lastRunTestCaseId`, `lastRunAgentConfigId`. With `--by-config`, instead emits `{ exists, dbPath, since, testCase, byConfig: [{configId, total, passed, failed, solveRate, avgCostUsd, avgDurationMs, avgTokensIn, avgTokensOut}] }`.
