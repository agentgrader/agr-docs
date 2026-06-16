# Changelog & migration

Release notes and breaking changes are published on GitHub:

- [agentgrader/agr releases](https://github.com/agentgrader/agr/releases)
- [npm `agentgrader` versions](https://www.npmjs.com/package/agentgrader?activeTab=versions)

## Recent CLI additions (1.6.0)

If you are upgrading from 1.5.x, these flags and commands are new in 1.6.0:

| Feature | CLI surface |
|---|---|
| Project scaffold | `agr init` / `agr init --blank` |
| Test case discovery | `agr list-tests [dir]`, `agr list-tests --json`, `agr list-tests --tags` |
| Name-based resolution | `agr run <name>`, `agr validate <name>`, `agr bench <name>` (no path needed) |
| Dry run preview | `agr bench --dry-run` |
| Tag-based filtering | `--tags python,fast` on `bench`, `validate --suite`, `list-tests` |
| Multi-validate | `agr validate task-a task-b --strict` |
| Suite validate | `agr validate --suite tasks/ --strict` |
| Last-run shortcut | `agr trace --last`, `agr trace --last --quality` |
| Toolkit commands | `agr toolkit-add <name>`, `agr toolkit-list <dir>` |
| Positional bench args | `agr bench task-a task-b --matrix matrix.yaml` |
| Next-step hints | `agr run`, `agr bench`, `agr validate` print a contextual `Next:` line on success |
| Tags inline | `agr list-tests` and `agr bench --dry-run` show `[tag1, tag2]` per row when any test case has tags |
| Bench result summary | Non-matrix bench prints `Result: N/M PASS (X%) cost: $Y`; multi-config adds per-config breakdown |
| Bugfix: trace --last steps | `agr trace --last` was showing 0 steps (bug: passed `undefined` to `getTraces`); now fixed |
| --tags without --suite warning | `agr bench --tags` and `agr validate --tags` without `--suite` now print a warning instead of silently doing nothing |
| More Next: hints | `agr compare` now prints `Next: agr trace <idA>  \| agr trace <idB>` after the step diff summary; `agr compare-baseline --output` suggests posting the file as a PR comment |

**Earlier additions** (1.5.x):

| Feature | CLI surface |
|---|---|
| CI exit gates | `--fail-on-failure`, `--min-solve-rate`, `--min-solve-rate-scope` |
| Bench reports | `--report`, `--output`, `--report-include-traces` |
| Baselines | `agr bench --save-baseline`, `agr compare-baseline` |
| Cloud sandboxes | `--sandbox e2b` (requires `E2B_API_KEY`) |
| Toolkit audit | `agr validate-toolkit`, `--audit-toolkits`, `--strict-toolkits` |
| Export | `agr export runs\|traces`, `AGR_EXPORT_ON_BENCH` |
| LLM judge | `--llm-judge`, `--judge-gate`, `--judge-min-score`, `rubrics:` in agr.yaml |

Full reference: [CLI](/reference/cli).

## Migration notes

**1.6.0 — No breaking changes.** All new flags are opt-in. Name-based `agr run <name>` is additive: path forms still work.

**CI pipelines**: Add `--fail-on-failure` if your workflow assumed non-zero exit codes on agent failure. Previously `agr run`/`agr bench` exited `0` after a completed run regardless of pass/fail.

**ACP sandboxed MCP**: Stdio servers with `sandboxed: true` are rewritten to `agr-mcp-proxy` when the sandbox exposes `sandboxBridgeId`. See [ACP Agent Adapter](/advanced/acp-agent#sandboxed-true-for-acp-agents-via-agr-mcp-proxy).

**Docs submodule**: This site lives in [agentgrader/agr-docs](https://github.com/agentgrader/agr-docs). Report doc issues there or via edit links on each page.
