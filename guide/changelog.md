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
| Debug hints on empty suite | `agr bench --suite` and `agr validate --suite` print `agr list-tests <dir>` when no test cases are found or no cases match `--tags` |
| --report without --output warning | `agr run --report <fmt>` and `agr bench --report <fmt>` without `--output` now warn instead of silently writing no report |
| toolkit-list untracked hint | `agr toolkit-list --check-config` prints the exact `track_tools:` YAML snippet to copy when untracked tools are found |
| Bench startup log | `agr bench` now prints `Starting N run(s), concurrency: M` before the live dashboard, making total jobs and concurrency visible without `--dry-run` |
| Bench elapsed time | `agr bench` result summary now shows `elapsed: Xs` wall-clock time |
| Export record count | `agr export runs` and `agr export traces` now print the record count in the output line, e.g. `Export written to export-runs.json (42 records)` |
| export traces --last | `agr export traces --last` exports the most recent run without specifying a run ID (mirrors `agr trace --last`) |
| compare --last-two | `agr compare --last-two` compares the two most recent runs; multi-config bench `Next:` hint now suggests it |
| list-tests next hint | `agr list-tests` now shows a context-aware next hint: suggests `agr bench --suite --tags` when `--tags` are active, or `agr bench --suite` for multiple results |
| run pass/fail hint | `agr run` shows `Next: agr bench <name>` on pass (scale up) and `Inspect: agr trace --last` on fail (debug) |
| trace next hints | `agr trace` now prints a `Next:` hint at the end of each view mode, cross-linking `--quality`, `--tools`, and `agr compare --last-two` |
| bench failed cases | `agr bench` (single-config) now prints `Failed: case-a, case-b` after the result summary when up to 10 test cases fail, showing which tests need attention inline |
| bench error distinction | `agr bench` result now shows separate `Failed:` (test didn't pass) and `Errored:` (sandbox crash) lines, with the error snippet for crashes |
| bench startup breakdown | `agr bench` startup log now shows `N test case(s) x M config(s) = K run(s)` breakdown for multi-config runs |
| bench tag breakdown notes | Tag breakdown rows now show `all passed` or `none passed` annotation for 100%/0% solve rate tags |
| bench multi-config failed cases | `agr bench` multi-config result now shows `Failed:` and `Errored:` sub-lines per agent config, so you can see which test cases each config is stuck on at a glance |
| bench gate inspect hint | `agr bench` prints `Inspect:` instead of `Next:` when a CI gate fires (`--fail-on-failure`, `--min-solve-rate`), signaling debug mode vs. normal progression |
| list --plain next hints | `agr list --plain` footer now shows `agr trace --last`, `agr compare --last-two`, and `agr list` (TUI) as actionable next steps instead of the circular "Open interactively" message |
| validate-toolkit next hints | `agr validate-toolkit` prints `Next:` on success and a fix-and-rerun instruction on failure, completing the toolkit audit workflow |
| bench zero-solve inspect hint | `agr bench` shows `Inspect:` instead of `Next:` when solve rate is 0% (even without gates), since a 0% result always means the user needs to debug |
| bench large failure count | `agr bench` now shows `Failed: N cases (see \`agr list\`)` when more than 10 cases fail, instead of silently omitting the failure detail |
| trace human duration | `agr trace` now shows run duration in human-readable format (`45.0s`, `2m 30s`) instead of raw milliseconds (`45000ms`) |
| export runs --last-matrix | `agr export runs --last-matrix` exports the most recent matrix sweep without requiring the matrix ID; `agr bench --matrix` Next: hint now uses it |
| compare human duration | `agr compare` now shows run duration in human-readable format (`45.0s`, `2m 30s`) consistent with `agr trace`; `formatDuration` is now a shared utility |
| list --plain duration | `agr list --plain` now shows run duration alongside cost and step count for quick performance comparison |
| validate failure hint | `agr validate` now prints a fix-and-rerun instruction when validation fails, mirroring `agr validate-toolkit`; multi-case runs list only the failing names |
| run human duration | `agr run` RUN SUMMARY now shows duration in human-readable format (`1m 30s`) consistent with `agr trace`, `agr compare`, and `agr list --plain` |
| bench/list TUI human duration | `agr bench` live dashboard and `agr list` TUI detail panel now show duration in human-readable format, completing the `formatDuration` rollout across all views |
| report human duration | `agr bench --report` HTML and Markdown reports now show duration in human-readable format instead of raw milliseconds |
| run trace-id hint | `agr run` next/inspect hint now uses the actual run ID (`agr trace <runId>`) instead of `--last`, so the reference stays stable after subsequent runs |
| toolkit-list next hints | `agr toolkit-list` now prints a `Next:` hint in the base case and a fix-and-rerun or proceed hint after `--check-config`, completing the toolkit audit workflow |
| bench avg cost per run | `agr bench` result summary now shows `avg: $X.XXXX/run` when running multiple test cases; multi-config breakdown also shows per-config avg |
| bench avg duration per run | `agr bench` result now shows `avg: Xs/run` alongside avg cost; multi-config breakdown shows per-config avg duration for speed vs cost comparisons |
| report steps column | `agr bench --report` HTML and Markdown reports now include a `Steps` column alongside cost and duration |
| formatDuration tests | `formatDuration` utility now has unit tests covering sub-second, sub-minute, and multi-minute cases plus zero-ms edge case |
| report by-config avg duration | `agr bench --report` HTML and Markdown by-config tables now include `Avg duration` alongside avg cost for speed vs cost comparisons |
| trace steps header | `agr trace` now shows `steps: N` in the run header, consistent with `agr compare` |
| bench avg steps per run | `agr bench` result now shows `avg: N steps/run` alongside avg cost and avg duration; per-config breakdown also shows avg step count; duration avg note now uses consistent `avg:` prefix |

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
