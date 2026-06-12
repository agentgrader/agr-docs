# Bench Manifest (`bench.yaml`)

A bench manifest describes a full benchmark in one file: which test case suite to run, which agent configs to use, and optional concurrency. Paths in the manifest are resolved relative to the manifest file's directory.

Use it when you keep one YAML per agent architecture and do not want long `--configs` lists on the command line.

```yaml
name: my-benchmark
suite: ./test-cases
agents:
  glob: "./agents-configs/*.yaml"
concurrency: 2
```

```bash
agr bench --manifest bench.yaml
```

## Schema reference

### `name`

**Type:** `string` (optional)

Human-readable label for the benchmark. Used in CLI log output.

### `suite`

**Type:** `string` (required)

Path to the test case suite directory (same as `agr bench --suite`). Agentgrader recursively finds `agr.yaml` files under this directory.

### `agents`

**Type:** `object` (required)

Describes which agent config files to load. At least one of `paths` or `glob` is required. Both can be combined; duplicates are removed.

#### `agents.paths`

**Type:** `string[]` (optional)

Explicit list of agent config YAML files, relative to the manifest file.

```yaml
agents:
  paths:
    - ./agents/claude-debugger.yaml
    - ./agents/gpt-fast.yaml
```

#### `agents.glob`

**Type:** `string` or `string[]` (optional)

Glob pattern(s) matched relative to the manifest file. Supports `*` and `**`.

```yaml
agents:
  glob: "./agents-configs/*.yaml"
```

```yaml
agents:
  glob:
    - "./agents/*.yaml"
    - "./experiments/extra-agent.yaml"
```

### `concurrency`

**Type:** `number` (optional)

Parallel sandbox runs. Defaults to `2` unless overridden by `agr bench --concurrency`.

## Example layout

```
my-benchmark/
  bench.yaml
  agents-configs/
    claude-debugger.yaml
    gpt-fast.yaml
  test-cases/
    fix-greeting/
      agr.yaml
      fixture/
```

**`bench.yaml`:**

```yaml
name: greeting-benchmark
suite: ./test-cases
agents:
  glob: "./agents-configs/*.yaml"
```

## Related options

| Goal | Command |
|---|---|
| Manifest (suite + agents in one file) | `agr bench --manifest bench.yaml` |
| Agent folder only (suite on CLI) | `agr bench --suite test-cases/ --configs-dir agents-configs/` |
| Explicit file list | `agr bench --suite test-cases/ --configs a.yaml,b.yaml` |
| Hyperparameter sweep (inline fields) | `agr bench --suite test-cases/ --matrix matrix.yaml` |

See [CLI Reference: agr bench](/reference/cli#agr-bench) and [Core Concepts](/guide/concepts#organizing-agent-configs).
