# CI Integration

Agentgrader runs in standard CI environments (GitHub Actions, GitLab CI, TeamCity, and others) as long as Docker and an LLM API key are available.

Use `assert:` criteria in test cases (`assert: steps <= 10`, `assert: cost_usd <= 0.05`) to fail builds when evaluation limits are exceeded. Use `agr validate --strict` to reject incomplete test case definitions before running expensive agent evaluations.

## Pipeline exit codes

Use `--fail-on-failure` on `agr run` and `agr bench` to exit with code 1 when a run does not pass. By default both commands exit 0 after a completed run (even when the agent failed scoring), which is convenient for local debugging but unsuitable as a CI gate.

```bash
agr run hello-world --config agent.yaml --fail-on-failure
agr bench --suite tasks/ --config agent.yaml --fail-on-failure
```

On `agr bench`, you can also set a minimum solve rate:

```bash
agr bench --suite tasks/ --config agent.yaml --min-solve-rate 0.8
agr bench --suite tasks/ --configs a.yaml,b.yaml --min-solve-rate 0.9 --min-solve-rate-scope per-config
```

Combine both flags when you want any individual failure or an overall solve-rate drop to fail the job.

## Report artifacts for CI

Generate machine-readable or human-readable reports and upload them as workflow artifacts:

```bash
agr bench --suite tasks/ --config agent.yaml \
  --fail-on-failure \
  --report json \
  --output reports/bench.json
```

Supported formats: `json`, `jsonl`, `html`, `md`. Add `--report-include-traces` to embed step traces (large files).

GitHub Actions example with artifact upload:

```yaml
      - name: Run benchmark
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          agr bench --suite tasks/ --config agent.yaml \
            --fail-on-failure --min-solve-rate 0.8 \
            --report json --output reports/bench.json

      - name: Upload report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: agentgrader-report
          path: reports/
```

## Baseline comparison in pull requests

Save a baseline snapshot on `main`, then compare PR bench results against it:

```bash
agr bench --suite tasks/ --config agent.yaml --save-baseline baselines/main.json
agr compare-baseline --current baselines/main.json --format md --output comment.md
```

Post the markdown file as a PR comment with `actions/github-script` or `marocchino/sticky-pull-request-comment`. Use `--fail-on-regression` to fail the job when solve rate drops or any previously passing case fails.

## Validate test cases in CI

Before running agents, verify test case definitions are complete. `agr validate --suite` validates every test case under a directory in one command:

::: code-group

```bash [bun]
bun add -g agentgrader
agr validate --suite tasks/ --strict
```

```bash [npm]
npm install -g agentgrader
agr validate --suite tasks/ --strict
```

:::

`--strict` exits with code 1 if `test_command`, `fail_to_pass`, or `pass_to_pass` are missing. Without it, `agr validate` may pass with only static YAML checks (execution checks skipped when `test_command` is absent).

To also run toolkit security audits:

```bash
agr validate --suite tasks/ --strict --audit-toolkits
```

See [Best Practices: Validate before you benchmark](/guide/best-practices#validate-before-you-benchmark).

## GitHub Actions example

Benchmark on every pull request, with validate gate, dry-run preview, and report artifact:

```yaml
name: Agentgrader Benchmarks

on: [push, pull_request]

jobs:
  bench:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: oven-sh/setup-bun@v2

      - name: Install Agentgrader
        run: bun add -g agentgrader

      - name: Validate test cases
        run: agr validate --suite tasks/ --strict

      - name: Preview bench matrix (dry run)
        run: agr bench --suite tasks/ --config agent.yaml --dry-run

      - name: Run benchmark
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          agr bench --suite tasks/ --config agent.yaml \
            --concurrency 2 \
            --fail-on-failure \
            --min-solve-rate 0.8 \
            --report json --output reports/bench.json

      - name: Upload report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: agentgrader-report
          path: reports/
```

## Things to keep in mind

- **Docker** must be available on the runner. `ubuntu-latest` on GitHub Actions includes Docker.
- **API keys** must come from encrypted secrets: never hardcode them. The CLI also reads a `.env` file if present, but CI should use `env:` / secrets instead.
- **Concurrency**: use `--concurrency` to parallelize sandbox runs. Balance against runner CPU and Docker limits.
- **Cost**: agent benchmarks consume LLM tokens. Consider running on a schedule or only on labeled PRs for large suites.

For embedding evaluations in custom CI logic, see the [Programmatic API](/advanced/programmatic-api).
