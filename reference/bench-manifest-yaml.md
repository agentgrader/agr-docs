# Bench Manifest (`bench.yaml`)

A bench manifest describes **which test suite** and **which agent configs** to use in a single `agr bench --manifest` invocation. Paths in the manifest are resolved relative to the manifest file location.

## Minimal example

```yaml
name: typescript-bugs-bench
suite: ./suites/typescript-bugs
agents:
  glob: "./configs/*.yaml"
concurrency: 2
```

Run it:

::: code-group

```bash [npm]
npm install -g agentgrader
agr bench --manifest bench.yaml
```

```bash [bun]
bunx agentgrader bench --manifest bench.yaml
```

:::

## Schema reference

### `name`

**Type:** `string` (optional)

Human-readable label for the benchmark. Shown in logs and useful in CI job names.

### `suite`

**Type:** `string` (required)

Path to the directory containing test case folders (each with an `agr.yaml`). Resolved relative to the manifest file.

Example: if `bench.yaml` lives in `my-benchmark/` and `suite: ./tasks`, Agentgrader loads every `agr.yaml` under `my-benchmark/tasks/`.

### `agents`

**Type:** `object` (required)

Describes which agent config YAML files to load. You must set at least one of `paths` or `glob`.

#### `agents.paths`

**Type:** `string[]` (optional)

Explicit list of agent config files, relative to the manifest directory.

```yaml
agents:
  paths:
    - ./agents-configs/claude.yaml
    - ./agents-configs/gpt-mini.yaml
```

#### `agents.glob`

**Type:** `string` or `string[]` (optional)

Glob pattern(s) matched from the manifest directory. Useful when you have many configs in one folder.

```yaml
agents:
  glob:
    - "./agents-configs/*.yaml"
    - "./experiments/*.yaml"
```

### `concurrency`

**Type:** `number` (optional)

Maximum parallel sandbox runs. Defaults to the CLI default (`2`) when omitted. Increase on powerful runners; decrease when Docker or API rate limits bite.

## Full example

```
my-benchmark/
  bench.yaml
  agents-configs/
    claude-sonnet.yaml
    gpt-4o-mini.yaml
  tasks/
    fix-greeting/
      agr.yaml
      fixture/
    fix-parser/
      agr.yaml
      fixture/
```

**`bench.yaml`:**

```yaml
name: nightly-regression
suite: ./tasks
agents:
  glob: "./agents-configs/*.yaml"
concurrency: 3
```

Equivalent without a manifest:

```bash
agr bench \
  --suite tasks/ \
  --configs-dir agents-configs/ \
  --concurrency 3
```

## Manifest vs. matrix vs. CLI flags

| Approach | When to use |
|---|---|
| `bench.yaml` | Stable team config: suite + agent globs in one checked-in file |
| `--configs-dir` / `--configs` | Ad-hoc runs, scripting, one-off comparisons |
| `--matrix matrix.yaml` | Cartesian sweep over hyperparameters (model, temperature, prompts) |

A manifest does **not** expand matrices. For sweeps, use [Core Concepts: Optimizer matrices](/guide/concepts#optimizer-matrices).

## CI usage

```yaml
- name: Run benchmark
  env:
    OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
  run: agr bench --manifest bench.yaml
```

Validate test cases first with `agr validate --strict` (see [Best Practices](/guide/best-practices#validate-before-you-benchmark)).

## Related

- [CLI Reference: `agr bench`](/reference/cli)
- [Agent Config YAML](/reference/agent-config-yaml)
- [CI Integration](/advanced/ci-integration)
