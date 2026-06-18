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
| report by-config avg steps | `agr bench --report` HTML and Markdown by-config tables now include `Avg steps` alongside avg cost and avg duration |
| code-search toolkit skills | `examples/toolkits/code-search` now includes `SKILL.md` files for `find-todos` and `find-usages`, plus a `find-usages` bin script as a complete multi-tool toolkit reference |
| aggregate steps tests | `aggregateResults` tests now cover `avgStepsCount` in the main averages test and the missing-fields zero-fallback test |
| baseline avg duration and steps | `agr bench --save-baseline` now records `avgDurationMs` and `avgStepsCount`; `agr compare-baseline` output shows `Avg duration` and `Avg steps` rows with percentage deltas |
| export runs steps and tokens | `agr export runs` now includes `stepsCount`, `tokensIn`, and `tokensOut` in each exported record alongside cost, duration, and metrics |
| init gitignore | `agr init` now creates a `.gitignore` (skipped if one exists) ignoring `.agr/` and `.env`, preventing accidental commits of the run database |
| trace/compare token totals | `agr trace` and `agr compare` run headers now show `tokens: N in / M out` when token data is available |
| report by-config avg tokens | `agr bench --report` HTML and Markdown by-config tables now include `Avg tokens in` and `Avg tokens out` columns |
| git-context toolkit | `examples/toolkits/git-context` adds `recent-changes` (recently modified files) and `file-log` (per-file commit history) as a second complete multi-tool toolkit reference |
| status command | `agr status` prints a quick DB summary (total runs, pass/fail counts, unique test cases and configs, total cost, last run time) without launching the TUI |
| bench avg tokens per run | `agr bench` result summary now shows `avg: Nin/Mout tok/run` when token data is available; multi-config per-config breakdown also shows per-config avg token counts for model cost and token-efficiency comparisons |
| baseline avg tokens | `agr bench --save-baseline` now records `avgTokensIn` and `avgTokensOut`; `agr compare-baseline` shows `Avg tokens in` and `Avg tokens out` rows with percentage deltas when token data is present |
| provider override | `agr run --provider <name>` and `agr bench --provider <name>` override the LLM provider for all agent configs without editing YAML; combine with `--model` for quick cross-provider comparisons; also fixes `agr bench --report-dir` not being passed through |
| cleanup --json | `agr cleanup --json` outputs `{found, removed, containers[]}` as a JSON object; without `--yes` lists containers; with `--yes` reports removal results per container |
| validate-toolkit --json | `agr validate-toolkit --json` outputs `{dir, passed, findings[]}` as a JSON object with `{file, severity, rule, message}` per finding; exits with code 1 on failure; combinable with `--strict` |
| compare --json | `agr compare --json` outputs `{runA, runB, divergentCount, totalSteps, firstDivergence, steps[]}` as a single JSON object; each step has `{index, divergent, a, b}`; combinable with `--last-two`, `--test-case`, and `--config` |
| dry-run --json | `agr bench --dry-run --json` emits `{testCases, agentConfigs, totalRuns, concurrency}` as a single JSON object; useful for CI pipelines that need to inspect the planned matrix before committing to a full bench |
| trace --json | `agr trace --json` outputs the run trace as a JSON object; default mode emits `{run, steps[]}`, `--quality` emits `{run, metrics}`, `--tools` emits `{run, toolUsage}`; combinable with `--last`, `--test-case`, and `--config` |
| matrix summary tokens | `agr bench --matrix` summary now shows `tok:Nin/Mout` per config when token data is available, making model token-efficiency visible in matrix sweeps |
| list tokens | `agr list --plain` shows `tokens: Nin/Mout` per run when available; `agr list` TUI detail panel also shows token counts alongside cost, duration, and steps |
| fix-missing-await example | `examples/suites/typescript-bugs/` now has a second test case: `fix-missing-await`, a classic async loop bug where forgetting `await` produces NaN; a CI test validates all example test cases load without schema errors |
| acp-claude-with-toolkit both toolkits | `examples/configs/agent-acp-claude-with-toolkit.yaml` now references both `code-search` and `git-context` toolkits |
| init status hint | `agr init` and `agr init --blank` next-steps now include `agr status` as step 4, helping new users discover the DB summary command |
| status token totals | `agr status` now shows `Tokens: N in / M out` when token data exists in the database |
| status cleanup next hint | `agr cleanup --yes` now prints `Next: agr bench  \| agr list` after removing containers |
| status --json | `agr status --json` emits machine-readable JSON for use in CI scripts and shell pipelines |
| export runs --since | `agr export runs --since <duration|date>` filters to runs created after a given point; accepts relative durations (1h, 24h, 7d) or ISO timestamps |
| bench --limit | `agr bench --suite tasks/ --limit N` runs only the first N test cases after filtering; useful for quick smoke tests on large suites |
| status --since | `agr status --since <duration|date>` restricts DB summary stats to runs after a given point; `--json` output includes `since` field |
| list --since | `agr list --since <duration|date>` filters the run list to runs after a given point; applied before `--limit` |
| export runs --test-case/--config | `agr export runs --test-case <id>` and `--config <id>` filter exported runs by test case or agent config (substring match); combinable with `--since` and `--limit` |
| export runs --passed/--failed | `agr export runs --passed` or `--failed` filters to only passing or only failing runs; useful for building failure analysis pipelines |
| compare --last-two --test-case | `agr compare --last-two --test-case <name>` compares the two most recent runs of a specific test case without needing run IDs |
| trace --last --test-case | `agr trace --last --test-case <name>` traces the most recent run for a specific test case; pairs with compare --last-two --test-case for per-task debug workflows |
| status --test-case | `agr status --test-case <name>` shows solve rate, avg cost, avg duration, and run count for one test case; `--json` adds `solveRate`, `avgCostUsd`, `avgDurationMs` fields |
| list --test-case | `agr list --plain --test-case <name>` filters the run list to a specific test case; consistent with status/trace/compare `--test-case` pattern |
| list/status --config | `agr list --config <name>` and `agr status --config <name>` filter by agent config; useful for comparing performance between two configs via `agr status --config agent-a` vs `--config agent-b` |
| export runs --format csv | `agr export runs --format csv --output runs.csv` writes a CSV file with one row per run; all fields including `metrics` (JSON-serialized) are included as columns; default filename is `export-runs.csv` |
| bench --only-failed | `agr bench --suite tasks/ --only-failed` runs only the test cases that failed on their most recent DB run; enables tight fix-and-retry loops; exits cleanly when all previously-failed cases have since passed |
| trace/compare --config | `agr trace --last --config <name>` and `agr compare --last-two --config <name>` scope to the most recent run(s) for a specific agent config (substring match); completes the `--test-case`/`--config` filter symmetry across all per-run debug commands |
| list/status --passed/--failed | `agr list --plain --failed` and `agr status --failed` filter to only failing runs; `--passed` for passing runs only; mutually exclusive; mirrors `agr export runs --passed/--failed` to complete the outcome-filter pattern across all analytics commands |
| run --repeat | `agr run <name> --repeat 5` runs the same test case N times and prints a solve-rate summary (X/N PASS, avg cost, avg duration); useful for flakiness testing and verifying statistical consistency of a fix before scaling up with `agr bench` |
| status --by-config | `agr status --by-config` shows a per-config breakdown: solve rate, avg cost, avg duration, avg tokens per agent config, sorted by solve rate; combinable with `--since` and `--test-case`; `--json` emits a `byConfig` array |
| status --by-test-case | `agr status --by-test-case` shows a per-test-case breakdown: solve rate, avg cost, avg duration, sorted by solve rate ascending (hardest first); combinable with `--since` and `--config`; `--json` emits a `byTestCase` array |
| bench --shuffle | `agr bench --suite tasks/ --shuffle` randomizes test case order before running; reduces order-dependent bias and helps surface order-sensitive flaky tests |
| status --top | `agr status --by-test-case --top 5` or `--by-config --top 3` caps the breakdown output to N entries |
| export traces --test-case | `agr export traces --test-case hello-world` exports traces for all matching runs without a run ID; `--config`, `--since`, `--passed`, and `--limit` also filter multi-run trace exports; `agr export traces --last --test-case <name>` now correctly scopes to the most recent run for that test case (was ignoring `--test-case` before) |
| init --ci | `agr init --ci` (and `agr init --blank --ci`) writes `.github/workflows/agr.yml` - a GitHub Actions workflow that installs agentgrader and runs `agr bench --suite tasks/ --fail-on-failure` on push and pull_request, wiring the CI gate in one command |
| run/bench --model | `agr run <name> --model claude-opus-4-8` and `agr bench --suite tasks/ --model claude-opus-4-8` override the model for the run without editing the agent YAML; useful for quick model comparisons |
| run/bench --max-steps | `agr run <name> --max-steps 5` and `agr bench --suite tasks/ --max-steps 5` override `max_steps` without editing YAML; combine with `--limit` for cheap smoke tests |
| bench --skip-tags | `agr bench --suite tasks/ --skip-tags slow` excludes test cases with any of the specified tags; applied after `--tags` so you can include a broad set and then exclude a subset |
| doctor command | `agr doctor` runs a pre-flight check: Docker daemon, API keys, database, agent config, and test case discovery; exits 1 when any required check fails |
| doctor --json | `agr doctor --json` outputs `passed`, `failureCount`, `warningCount`, and `checks[]` as JSON; useful in setup scripts and CI init steps |
| init --example python | `agr init --example python` scaffolds a Python hello-world test case with `math.py` and `test_math.py` verified with `pytest -x` instead of the default JS/Node example |
| list-tests --count | `agr list-tests --count` prints only the number of matching test cases as a bare integer; useful in shell scripts and CI conditions |
| run --json | `agr run <name> --json` outputs the run result as a single JSON object and suppresses the live Ink UI; `passed` is `true`/`false`/`null`; useful for scripting (`result=$(agr run hello-world --json); echo $result \| jq .passed`) and CI pipelines that need structured output |
| bench --json | `agr bench --suite tasks/ --json` outputs a single JSON object with `passed`, `passedRuns`, `totalRuns`, `solveRate`, `totalCostUsd`, `elapsedMs`, `byConfig`, and `runs` arrays, suppressing the live dashboard; `gateReasons` is non-empty when a CI gate (`--fail-on-failure`, `--min-solve-rate`) triggers |
| bench --name | `agr bench --suite tasks/ --name "python"` filters test cases by name substring (case-insensitive), applied after `--tags` and `--skip-tags`; useful for quickly running a subset of a large suite without listing every case explicitly |
| list-tests --name | `agr list-tests --name "fix"` filters the discovery output by name substring (case-insensitive); combinable with `--tags`, `--count`, and `--json` to build targeted scripting pipelines |
| validate --json | `agr validate fix-greeting --json` outputs a JSON object with `passed`, `passedCount`, `totalCount`, and `results[]` (per-test-case `ok`, `name`, `path`, `checks[]`); suppresses per-check console output; works with `--suite` and `--strict` |
| run --repeat --json | `agr run hello-world --repeat 5 --json` outputs a summary JSON object with `passedRuns`, `totalRuns`, `solveRate`, `totalCostUsd`, `avgCostUsd`, `avgDurationMs`, and per-run `runs[]`; previously `--json` was silently ignored when combined with `--repeat` |
| run/bench --step-timeout | `agr run <name> --step-timeout 30000` and `agr bench --suite tasks/ --step-timeout 30000` override `step_timeout_ms` from the agent config for this run without editing YAML; useful in CI to cap per-LLM-call latency and abort stuck provider requests faster than the default 120s |
| validate --name | `agr validate --suite tasks/ --name "python"` filters test cases by name substring (case-insensitive) before validating; mirrors `agr bench --name` and `agr list-tests --name`; applied after `--tags` |
| run --save-baseline | `agr run hello-world --save-baseline baselines/main.json` captures a single run's result as a baseline JSON snapshot (same format as `agr bench --save-baseline`) for later comparison with `agr compare-baseline --current`; also works with `--repeat N` to save all N runs as a multi-run snapshot |
| run/bench --report-dir | `agr run <name> --report html --report-dir reports/` and `agr bench ... --report html --report-dir reports/` auto-generate a timestamped filename (`run-<timestamp>.html` / `bench-<timestamp>.html`) under the given directory when `--output` is not specified; useful in CI artifact archives where you always want a report but do not want to hardcode the filename |
| trace --json | `agr trace --last --json` outputs the step trace as a JSON object; default mode emits `{run, steps[]}`, `--quality` emits `{run, metrics}`, `--tools` emits `{run, toolUsage}`; combinable with `--last`, `--test-case`, `--config` |
| list --json | `agr list --json` outputs the run list as a JSON array (fields: `id`, `testCaseId`, `testCaseName`, `agentConfigId`, `agentConfigName`, `agentModel`, `passed`, `costUsd`, `durationMs`, `stepsCount`, `tokensIn`, `tokensOut`, `error`, `matrixId`, `createdAt`, `completedAt`); suppressess plain/TUI output; combinable with `--since`, `--test-case`, `--config`, `--passed`, `--failed`, `--limit`; completes the `--json` pattern across all agr commands |

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

**1.6.0: No breaking changes.** All new flags are opt-in. Name-based `agr run <name>` is additive: path forms still work.

**CI pipelines**: Add `--fail-on-failure` if your workflow assumed non-zero exit codes on agent failure. Previously `agr run`/`agr bench` exited `0` after a completed run regardless of pass/fail.

**ACP sandboxed MCP**: Stdio servers with `sandboxed: true` are rewritten to `agr-mcp-proxy` when the sandbox exposes `sandboxBridgeId`. See [ACP Agent Adapter](/advanced/acp-agent#sandboxed-true-for-acp-agents-via-agr-mcp-proxy).

**Docs submodule**: This site lives in [agentgrader/docs](https://github.com/agentgrader/docs). Report doc issues there or via edit links on each page.
