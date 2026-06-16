# Best Practices

Practical patterns for teams running Agentgrader in day-to-day development and CI. These recommendations come from real benchmark workflows: fast feedback loops, reproducible scoring, and predictable cost.

## Start with `agr init`

Use the built-in scaffold instead of hand-writing every file:

::: code-group

```bash [npm]
npm install -g agentgrader
agr init my-benchmark
cd my-benchmark
```

```bash [bun]
bun add -g agentgrader
agr init my-benchmark
cd my-benchmark
```

:::

Set your API key, then run the bundled hello-world task:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
agr run hello-world --config agent.yaml --verbose
```

Once that passes, replace the fixture and prompt with your own task. Keep the same folder layout (`tasks/<name>/agr.yaml` + `fixture/`).

## Project layout

A layout that scales well for teams:

```
my-benchmark/
  bench.yaml                 # optional manifest for agr bench --manifest
  agent.yaml                 # default agent for quick agr run
  agents-configs/            # one YAML per architecture you want to compare
    claude-sonnet.yaml
    gpt-mini.yaml
  tasks/
    fix-greeting/
      agr.yaml
      fixture/
  .env                       # local only, never commit API keys
  .agr/db.sqlite             # run history (gitignore this)
```

Version-control test cases and agent configs. Ignore `.agr/` and `.env` in git.

## Validate before you benchmark

Agent runs are slow and cost money. Catch YAML mistakes first:

```bash
agr validate fix-greeting
agr validate fix-greeting --strict
```

`--strict` fails when SWE-bench fields (`test_command`, `fail_to_pass`, `pass_to_pass`) are missing. Use it in CI before any `agr bench` step. See [CI workflows](/guide/ci-workflows).

`agr validate` accepts multiple test case names or a `--suite` directory, making it easy to gate a whole suite before benchmarking:

```bash
# Named list
agr validate task-a task-b task-c --strict

# Everything in a directory
agr validate --suite tasks/ --strict
```

Exits 1 if any test case fails; prints a summary line at the end (`N/M validations passed`).

Before a large bench (many tasks or a matrix sweep), use `--dry-run` to confirm the test case and config list without spending any API budget or starting Docker containers:

```bash
agr bench --suite tasks/ --matrix matrix.yaml --dry-run
```

The output shows every test case name and path, every expanded agent config with its model, and the total job count. Re-run without `--dry-run` once the matrix looks right.

## Fill in regression fields deliberately

For SWE-bench style scoring, `agr validate` does **not** auto-populate `fail_to_pass` or `pass_to_pass`. You must run the test suite once, read TAP output, and copy test names into `agr.yaml`:

1. Run `test_command` inside the fixture (locally or in Docker).
2. Note which tests fail on the broken fixture (`fail_to_pass`).
3. Note which tests must stay green (`pass_to_pass`).
4. Re-run `agr validate --strict` to confirm execution checks pass.

See [Test Case YAML](/reference/test-case-yaml) for the full schema. TAP pitfalls: [Troubleshooting](/guide/troubleshooting#tap-and-fail_to_pass).

## Set budget guardrails in YAML

Add `assert:` criteria so a run fails when an agent spirals:

```yaml
success:
  - run: npm test
    expect: { exit_code: 0 }
  - assert: steps <= 15
  - assert: cost_usd <= 0.10
timeout_seconds: 300
```

Tune limits per task difficulty. Tight limits catch runaway tool loops early. More on scorers: [Scoring & quality](/guide/scoring).

## Choose the right bench mode

| Goal | Command |
|---|---|
| One task, one agent | `agr run <name> --config agent.yaml` |
| Full suite × several agents | `agr bench --suite tasks/ --configs-dir agents-configs/` |
| Hyperparameter sweep (model × temperature) | `agr bench --matrix matrix.yaml --suite tasks/` |
| Reproducible team config in one file | `agr bench --manifest bench.yaml` |
| Run only tagged subset of a suite | `agr bench --suite tasks/ --config agent.yaml --tags python,fast` |

Use [Bench Manifest YAML](/reference/bench-manifest-yaml) for suite + agent globs in one file. Use [Optimizer Matrix YAML](/reference/matrix-yaml) for cartesian products. Guide: [Optimizer matrices](/guide/optimizer-matrices).

## Compare adapters fairly

When benchmarking an external ACP agent (Claude Code, Cursor Agent) against the built-in AI SDK loop, pass both adapters explicitly:

```bash
agr bench \
  --suite tasks/ \
  --configs agent.yaml,agents-configs/agent-acp-claude.yaml \
  --adapters ai-sdk,acp
```

See [ACP Agent Adapter](/advanced/acp-agent) for config fields and tool routing.

## Debug failed runs

Every run gets a UUID stored in `.agr/db.sqlite`. Use `agr trace --last` immediately after `agr run` to inspect the trace without copying the UUID, or `agr trace <runId>` to look up an older run. Add `--quality` or `--tools` as needed.

Full workflow: [Debugging failed runs](/guide/debugging).

## Toolkits

Custom CLI tools and skills need deliberate design, adoption tracking, and optional `setup.sh` hooks.

Full guide: [Toolkits guide](/guide/toolkits).

## CI recommendations

Validate with `--strict`, gate with `--fail-on-failure`, upload `--report` artifacts, and compare PR benches to baselines.

Full checklist: [CI workflows](/guide/ci-workflows). GitHub Actions example: [CI Integration](/advanced/ci-integration).

## Docker checklist

- Docker daemon must be running locally and on CI runners (or use `--sandbox e2b`)
- Pull base images ahead of large benches to avoid cold-start timeouts
- Increase `timeout_seconds` for tasks that compile heavy dependencies on first run

See [Choosing a sandbox](/guide/sandboxes).

### Validate a SWE-bench fixture's `success:` command before trusting FAIL

A `FAIL` on a SWE-bench style task can mean the agent's patch is wrong, or it can mean the fixture's own `success:` command never passes even on the *unmodified* fixture (a broken dependency pin, an unpinned transitive build dependency, a flaky registry timeout). Before treating repeated `FAIL` results on a given fixture as an agent-quality signal, run the `success:` command once against the unmodified fixture (e.g. in a throwaway Docker container) to confirm it can pass at all. A common culprit for Python fixtures: `pip install -e ".[test]"` without `--no-build-isolation` builds in an isolated venv populated from `pyproject.toml`'s `[build-system] requires`, so a version pin applied only to the outer `pip install` (e.g. `pip install "setuptools<60"`) does not protect the isolated build env, which can resolve a newer, incompatible build tool. If the `success:` command fails this way, fix it once (e.g. add the same pin to `[build-system] requires`) and re-validate before spending bench budget on that fixture again.

## Next steps

- [Quickstart](/guide/quickstart): minimal end-to-end walkthrough
- [Recipes](/guide/recipes): copy-paste workflows
- [Troubleshooting](/guide/troubleshooting): common failure modes
- [Programmatic API](/advanced/programmatic-api): embed runs in custom tooling
