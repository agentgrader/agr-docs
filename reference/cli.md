# CLI Reference

The command line tool for this project is called `agr`. You can easily install it globally by running `npm install -g agentgrader`.

## `agr bench`

This command runs a complete benchmark. It evaluates all of your specified test cases against every agent configuration you provide. It also supports running these tests in parallel to save time. While it runs, you will see a live interactive dashboard right in your terminal to keep track of the progress.

```bash
agr bench \
  --suite examples/suites/typescript-bugs/ \
  --configs examples/configs/baseline.yaml \
  --concurrency 2
```

Every run is also scored by the additive `StaticQualityScorer`, which annotates `metrics["static-quality"]` with diff size, files touched, TODOs introduced, and lint violations - see [Quality Scorers & the Optimizer](/guide/concepts#5-quality-scorers-the-optimizer).

### Available Options

| Flag | Default | Description |
|---|---|---|
| `--suite` | Required | The path pointing to a directory filled with test case folders. |
| `--configs` | One of `--configs`/`--matrix` required | A comma separated list of paths to your agent configuration YAML files. |
| `--matrix` | One of `--configs`/`--matrix` required | Path to an optimizer matrix YAML file. Expands into the cartesian product of agent configs (alternative to `--configs`), tags every resulting run with a shared `matrixId`, and prints a Pareto-marked "MATRIX SUMMARY" table after the run. See [Quality Scorers & the Optimizer](/guide/concepts#5-quality-scorers-the-optimizer). |
| `--concurrency` | `2` | Determines how many benchmark runs should execute at the same time. |

## `agr run`

If you want to run just a single test case, this is the command to use. It is especially helpful when you are debugging specific tests or actively developing new agents.

```bash
agr run examples/suites/typescript-bugs/add-error-handling/agr.yaml \
  --config examples/configs/baseline.yaml
```

### Available Options

| Flag | Default | Description |
|---|---|---|
| (positional) | Required | The exact path to a specific `agr.yaml` test case file. |
| `--config` | Optional | The path to your agent config YAML. If you leave this out, it defaults to using `gpt-4o-mini` with a maximum of 20 steps. |

## `agr validate`

This command validates a test case the way SWE-bench validates a candidate task before it's added to a benchmark. It runs static checks, a pre-patch run (verifying FAIL_TO_PASS tests fail and PASS_TO_PASS tests pass), and a post-patch run using the gold solution.

```bash
agr validate examples/suites/typescript-bugs/add-error-handling/agr.yaml
```

### Available Options

| Flag | Default | Description |
|---|---|---|
| (positional) | Required | The exact path to a specific `agr.yaml` test case file. |

## `agr import-pr`

This command fetches a PR's diff from GitHub, splits it into a gold solution patch and an optional test patch, and automatically scaffolds a new `agr.yaml` test case complete with `expected_files` and `forbid_modified` fields.

```bash
agr import-pr agentgrader/agr 123 --out examples/suites/new-bug --clone-fixture --validate
```

Set `GITHUB_TOKEN` in your environment to avoid GitHub's low unauthenticated rate limits.

### Available Options

| Flag | Default | Description |
|---|---|---|
| (positional) | Required | The GitHub repository in `owner/repo` format. |
| (positional) | Required | The Pull Request number. |
| `--out` | `./imported/<repo>-pr-<number>` | The destination directory to scaffold the test case into. |
| `--clone-fixture` | `false` | Clones the repository and checks out the PR's `base.sha` into `<out>/fixture`, so the scaffolded test case is immediately runnable (modulo filling in `test_command`/`fail_to_pass`/`pass_to_pass`). |
| `--validate` | `false` | Runs `agr validate` against the scaffolded `agr.yaml` once it has been written. Most useful combined with `--clone-fixture` after filling in the TODO fields. |

## `agr trace`

Prints the recorded step trace and metrics for a single run, looked up by its run ID (shown in the `bench`/`run` dashboard and stored in the database).

```bash
agr trace 3f1c2e2a-...
```

With `--quality`, it instead prints just the additive quality-metrics breakdown for the run: the `StaticQualityScorer` results (`metrics["static-quality"]`), the `LlmJudgeScorer` results (`metrics["llm-judge"]`), and the `DiffScorer`/`LocalizationScorer` summaries.

```bash
agr trace 3f1c2e2a-... --quality
```

### Available Options

| Flag | Default | Description |
|---|---|---|
| (positional) | Required | The ID of the run to inspect. |
| `--quality` | `false` | Show only the quality-metrics breakdown instead of the full step trace. |
