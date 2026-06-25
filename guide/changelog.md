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
| toolkit-list --json | `agr toolkit-list --json` outputs `{toolkitDir, tools[], auditFindings[], ok}` for CI gating; with `--check-config --json` adds `untracked[]`, `trackedButMissing[]`; pipe to `jq .ok` for pass/fail |
| status --trend | `agr status --since <window> --trend` compares current window vs previous equal-length window: shows solve-rate delta (pp), run count delta, avg cost delta with `up/down` arrows; `--json` emits `{current, previous, delta}`; combinable with `--test-case`, `--config`, `--sandbox` |
| status --by-day | `agr status --since 7d --by-day` shows a per-calendar-day breakdown (runs, solve rate, total cost) sorted oldest-first; useful for pinpointing when a regression started; combinable with `--since`, `--top`, and all filter flags; `--json` emits `{byDay: [{day, total, passed, failed, solveRate, totalCostUsd}]}` |
| status --sort-by | `agr status --by-test-case --sort-by cost` (also `--by-config`, `--by-model`) sorts breakdowns by avg cost/run descending (most expensive first); `--sort-by runs` sorts by run count; default `solve-rate` unchanged; useful for budget optimization |
| trace --steps | `agr trace --last --steps 40-60` (or `--steps 42`) shows only the specified stepIndex range; header shows `N step(s) [40-60] of 127 total`; combinable with `--json`; useful for navigating long 100+ step traces without scrolling |
| trace --grep | `agr trace --last --grep error` shows only steps whose label or content contains the pattern (case-insensitive); header shows `N matching step(s) for "error" of 127 total`; combinable with `--steps` (range applied first); useful for finding where a tool call or error first appears |
| trace --full | `agr trace --last --full` prints complete step content without the default 200-character truncation; combinable with `--steps`, `--grep`, and all run-selection flags; useful when a tool result or LLM response is cut off mid-sentence |
| status --errors | `agr status --errors` shows a deduplicated list of error messages from errored/failed runs sorted by frequency; each entry shows count, affected test cases, and an `agr trace <runId>` shortcut; combinable with `--since`, `--test-case`, `--config`, and all filter flags; `--json` emits `{errors: [{message, count, exampleRunId, testCaseIds}]}` |
| bench --sample | `agr bench --suite tasks/ --config agent.yaml --sample 5` randomly selects 5 test cases without replacement and runs only those; prints selected names so the draw is visible; combinable with `--suite`, `--tags`, `--config`, and all other bench flags; useful for quick sanity checks on large task sets |
| agr prune command | new `agr prune --before 30d --yes` deletes runs and their traces older than the cutoff; `--dry-run` previews without deleting; `--json` emits `{deleted, cutoff, dbPath}`; useful for keeping the DB lean after long eval campaigns |
| run --show-trace | `agr run hello-world --show-trace` prints all trace steps with content previews after the run completes; useful in CI where the interactive TUI is not available; output matches `agr trace --last` but inline in the run command |
| status --github-step-summary | `agr status --github-step-summary` appends a Markdown status summary to `$GITHUB_STEP_SUMMARY`; `agr init --ci` now uses this in the always-running summary step so CI dashboards show eval state even when the bench step fails |
| watch --exit-on-pass/--exit-on-fail | `agr watch --exit-on-fail` exits with code 1 as soon as any failing run appears; `--exit-on-pass` exits with code 0 when any passing run appears; both print the triggering run ID; useful for fail-fast CI or waiting for a fix to land |
| compare --passing-vs-failing | `agr compare --passing-vs-failing --test-case hello-world` compares the most recent failing run (A) vs most recent passing run (B); reveals what the agent does differently when it passes vs fails; supports `--only-diff`, `--full`, `--json`, `--test-case`, `--config` |
| bench --seed | `agr bench --sample 5 --seed 42` makes `--sample` and `--shuffle` reproducible; the same seed always produces the same ordering or selection; enables controlled experiments comparing two bench runs on identical subsets |
| agr watch command | new `agr watch` command polls `.agr/db.sqlite` every N seconds (default 3) and prints new runs as they appear; skips pre-existing runs; supports `--test-case`, `--config`, `--interval` filters; `--json` emits NDJSON for piping to jq; useful for monitoring a bench from a second terminal |
| bench --dry-run cost estimate | `agr bench --dry-run` now shows estimated total cost based on historical avg cost for matching test cases; format: `Estimated cost: ~$X.XXXX (avg $Y.YYYY/run based on N of M test case(s) with history)`; skipped gracefully when no history exists |
| trace --reverse | `agr trace --last --reverse` prints steps in reverse order (latest first); combinable with `--steps`, `--grep`, `--kind`, `--full`; useful for quickly seeing how a long trace ended without scrolling |
| export runs --deduplicate | `agr export runs --deduplicate --format csv --output snapshot.csv` keeps only the most recent run per (test case, agent config) pair before exporting; mirrors `agr list --latest`; useful for current-state snapshots without manual deduplication |
| trace --kind-summary | `agr trace --last --kind-summary` shows a compact table counting steps by kind (llm_response, tool_call, tool_result, etc.) with a proportional bar chart; `--json` emits `{run, total, kinds: [{kind, count}]}`; gives structural overview without reading the full trace |
| count --by-* solveRate | `agr count --by-test-case --json` and `--by-config --json` now include `solveRate` (0-100) in each entry; plain text also shows solve rate percentage; enables `jq '.byTestCase[] | select(.solveRate < 50)'` without arithmetic |
| init --ci workflow update | `agr init --ci` now generates a workflow using `--github-step-summary` (bench results in Actions summary), `--show-failures` (compact failure list), and an always-running `agr status --summary` step; better CI starting point |
| status --best-config/--best-model | `agr status --best-config` prints the config ID with the highest solve rate as a plain string; `--best-model` does the same for models; `--json` emits `{configId/model, solveRate, total, passed, avgCostUsd}`; useful for CI scripts that auto-promote the best-performing config |
| bench --output-run-ids | `agr bench ... --output-run-ids run-ids.txt` writes all completed run IDs to a file (one per line); useful in CI where stdout capture is awkward; complements `--print-ids` which writes to stdout |
| bench --github-step-summary | `agr bench ... --github-step-summary` appends bench results as Markdown to `$GITHUB_STEP_SUMMARY`; includes status emoji, solve rate, total cost, and per-config table; no-ops with a warning when the env var is not set; gives free bench visibility in GitHub Actions without extra tooling |
| status --summary | `agr status --summary` prints a compact one-liner: `127 runs: 89 PASS (70%) | $1.23 total avg: $0.0097/run | last: 2m ago`; `--json` emits `{totalRuns, passedRuns, failedRuns, solveRate, totalCostUsd, avgCostUsd, lastRunAt, dbPath}`; combinable with all filter flags |
| trace --min-cost/--max-cost | `agr trace --last --min-cost 0.001` shows only steps costing at least $0.001; `--max-cost 0.0001` shows cheap/free steps only; both filter before `--top-cost`; combinable with all other trace filter and view flags |
| bench --skip-passing-since | `agr bench --suite tasks/ --config agent.yaml --skip-passing-since 24h` skips test cases with a passing run in the last 24h; prints how many were skipped; exits cleanly when all cases are covered; enables incremental bench runs for large suites |
| count --errored | `agr count --errored` counts runs that crashed before scoring (distinct from `--failed` which counts scored failures); combinable with all count filters; useful for CI monitoring of infrastructure errors vs evaluation failures |
| agr --version | `agr --version` (also `agr -v`) now prints the installed CLI version; useful for CI debugging and bug reports |
| status --sort-by duration | `agr status --by-test-case --sort-by duration` (also `--by-config`, `--by-model`) sorts by avg duration descending (slowest first); completes the sort-by field set alongside `solve-rate`, `cost`, and `runs`; useful for finding slow test cases or configs |
| status --by-week | `agr status --by-week` shows per-calendar-week breakdown (runs, solve rate, total cost), labeled `YYYY-Www`; higher-level view than `--by-day` for long-running eval suites; combinable with `--since`, `--top`, `--test-case`, `--config`, and all filter flags; `--json` emits `{byWeek: [...]}` |
| trace --step-count | `agr trace --last --step-count` prints total step count as a plain number; `--json` emits `{stepCount, filteredCount, runId}`; combinable with all run-selection flags; useful in CI for asserting agent step budgets |
| bench --only-unrun | `agr bench --suite tasks/ --config agent.yaml --only-unrun` runs only test cases with no recorded runs in the DB; exits cleanly when all cases have runs; natural companion to `agr list-tests --unrun`; useful for building initial coverage of a large suite without re-running already-executed cases |
| export traces --all | `agr export traces --all --format jsonl --output all-traces.jsonl` exports traces for all runs without requiring any filter; combine with `--limit N` to cap total; previously required `--test-case`, `--config`, `--since`, or `--run-id` |
| list-tests --run-counts | `agr list-tests --run-counts` shows run counts (total, passed, failed) alongside each test case sorted fewest-first; `0 runs [unrun]` marks never-executed cases; `--json` adds `runs`, `passed`, `failed` fields; useful for identifying under-covered test cases |
| list-tests --unrun | `agr list-tests --unrun` shows only test cases with no recorded runs in `.agr/db.sqlite`; combine with `--count` for a bare integer or `--json` for JSON output; gracefully handles missing DB; useful for finding tasks in a suite that have never been executed |
| status --above | `agr status --by-test-case --above 80` shows only entries with solve rate strictly above 80%; complement to `--below`; works with `--by-config` and `--by-model`; `--above 0` excludes never-passing cases; combinable with `--below` for a solve-rate range |
| run --min-pass-rate | `agr run hello-world --repeat 5 --min-pass-rate 0.8` exits with code 1 if the solve rate across repeated runs falls below 80%; more flexible than `--fail-on-failure` (which fails on any single failure); useful for CI gates on tests where occasional failures are acceptable but consistent failures are not |
| status --solve-rate | `agr status --solve-rate` prints solve rate as a plain number (e.g. `83.3`); `--json` emits `{solveRate, passedRuns, failedRuns, totalRuns, dbPath}`; combinable with all filter flags; useful for CI shell conditions (`if [ $(agr status --solve-rate --since 24h) -lt 80 ]`); complement to `agr cost` and `agr count` |
| export --columns | `agr export runs --columns id,testCaseId,passed,costUsd --format csv` selects which columns to include; omitting `metrics` avoids large JSON blobs in CSV; applies to CSV, JSON, and JSONL formats; unknown column names print a warning and are ignored |
| bench --config-grid | `agr bench --suite tasks/ --configs a.yaml,b.yaml --config-grid` prints a PASS/FAIL grid (test cases x configs) after the bench; only shown with 2+ configs; gives at-a-glance view of which tasks passed for which configs without needing `agr status --grid` separately |
| bench --show-failures | `agr bench ... --show-failures` prints a compact list of failing test cases after the bench, including run IDs as `agr trace <id>` shortcuts and up to 80 chars of error message; avoids having to separately run `agr status --by-test-case --below 100` to identify which tasks to investigate |
| status --show-ids | `agr status --by-test-case --show-ids` appends `last run: agr trace <id>` to each breakdown row; works with `--by-config` and `--by-model` too; `lastRunId` also in `--json` output; useful for quickly tracing the last run of a failing test case without looking up the ID separately |
| list duration filters | `agr list --min-duration 60000` finds runs taking over a minute; `--max-duration 5000` finds early-terminating runs; completes the range-filter set alongside `--min-cost/--max-cost` and `--min-steps/--max-steps`; combinable with all existing list filters and `--plain`/`--json` |
| cost --by-test-case/config | `agr cost --by-test-case` prints cost breakdown per test case sorted most expensive first; plain: tab-separated `$total\ttestCaseId\t(N runs, avg $X/run)`; JSON: `{total, totalCostUsd, byTestCase: [{testCaseId, total, totalCostUsd, avgCostUsd}]}`; `--by-config` does the same per agent config; combinable with all existing cost filters |
| status --rolling | `agr status --by-test-case --rolling 5` computes solve rate using only the most recent 5 runs per entry; works with `--by-config` and `--by-model` too; useful for evaluating current agent quality without early-development failures dragging down the score; combinable with `--min-runs`, `--below`, `--top`, `--sort-by`, `--since` |
| bench --print-ids | `agr bench ... --print-ids` prints all completed run IDs to stdout after the bench (one per line under `Run IDs:`); enables shell pipelines like `agr bench ... --print-ids | tail -1 | xargs agr trace` or `while read id; do agr trace "$id" --quality; done`; combinable with all other bench flags |
| status --min-runs | `agr status --by-test-case --min-runs 5` filters breakdowns to only entries with at least N total runs; works with `--by-config` and `--by-model` too; combinable with `--below`, `--top`, `--sort-by`; useful for excluding test cases that haven't been run enough to produce statistically meaningful solve rates |
| agr cost command | new `agr cost` command prints total cost for matching runs as a plain dollar amount (`$1.2345`); supports all the same filters as `agr count`; `--json` emits `{totalCostUsd, avgCostUsd, total, dbPath}`; useful for budget checks in CI (`agr cost --last-matrix` after a bench) or shell scripting (`agr cost --since 24h --json | jq .avgCostUsd`) |
| trace --kind | `agr trace --last --kind llm_response` filters steps to those with an exact kind match (e.g. `llm_response`, `tool_call`, `tool_result`); header shows `N step(s) of kind "X" of M total`; combinable with `--steps`, `--grep`, `--full`, `--top-cost`; cleaner than `--grep` when you know the exact step type and want no false positives from content |
| trace --top-cost | `agr trace --last --top-cost 5` shows the 5 most expensive steps sorted by cost descending; header shows `top N most expensive step(s) of M total`; combinable with `--full`, `--grep`, `--steps`, and all run-selection flags; useful for finding where a run's token budget was spent |
| count --by-test-case/config | `agr count --by-test-case` prints run counts per test case sorted by total; plain: tab-separated `total\ttestCaseId\t(N passed, M failed)`; JSON: `{total, byTestCase: [{testCaseId, total, passed, failed}]}`; `--by-config` does the same per agent config; useful for finding test cases with insufficient run coverage (`--by-test-case --json | jq '.byTestCase[] | select(.total < 3)'`) |
| compare --first-and-last | `agr compare --first-and-last --test-case hello-world` compares the oldest and most recent run for a test case; useful for tracking how agent behavior changed since the first run; supports all the same options as `--last-two` including `--only-diff`, `--full`, and `--json` |
| status --grid | `agr status --grid` shows a cross-tab matrix with test cases as rows and agent configs as columns; each cell shows latest PASS, FAIL, or `--` (no run) for that pair; combinable with `--since`, `--test-case`, `--config`, and all filter flags; `--json` emits `{testCaseIds, configIds, grid[]}`; useful for seeing coverage and regressions across configs at a glance |
| status --below | `agr status --by-test-case --below 100` filters breakdown output to entries with solve rate strictly below n%; `--below 100` shows anything with at least one failure, `--below 50` shows entries failing more than half the time; works with `--by-config` and `--by-model` too; combinable with `--top`, `--sort-by`, `--since`, and all filter flags |
| list --latest | `agr list --plain --latest` deduplicates the run list to one entry per (test case, agent config) pair (the most recent run); combinable with `--passed`, `--failed`, `--json`, and all other filters; useful for a "current state" snapshot without needing `agr status --by-test-case` |
| run --until-pass | `agr run hello-world --until-pass` runs the test case repeatedly until it passes, stopping immediately on the first success; `--max-attempts N` caps the total (default 5); prints per-attempt PASS/FAIL/ERROR and a summary with which attempt passed and total cost; `--json` emits `{passed, attempts, maxAttempts, totalCostUsd, runs[]}`; useful for verifying a flaky fix without manually re-running |
| list cost/steps filters | `agr list --min-cost 0.05` shows only runs costing at least $0.05; `--max-cost`, `--min-steps`, `--max-steps` similarly filter by cost ceiling or step count range; combinable with all existing list filters and `--plain`/`--json`; useful for finding expensive outliers, early-terminating runs, or runaway agents |
| status --percentiles | `agr status --percentiles` adds p50 and p95 cost and duration stats to the base status output alongside the existing average; `--json` includes `p50CostUsd`, `p95CostUsd`, `p50DurationMs`, `p95DurationMs`; useful for spotting expensive outlier runs that skew the mean cost upward |
| status --flaky | `agr status --flaky` shows test cases with both passes and failures in their run history, sorted closest-to-50/50 first; each entry shows total, pass/fail counts, solve rate, and avg cost; combinable with `--since`, `--config`, `--model`, `--top`; `--json` emits `{flaky: [{testCaseId, total, passed, failed, solveRate, avgCostUsd, variance}]}`; useful for finding eval cases that need more runs to be statistically reliable |
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
| list --model | `agr list --model <substring>` filters runs by agent model (case-insensitive substring match on `agentModel`); combinable with `--test-case`, `--config`, `--since`, `--failed`, and `--json` |
| bench --repeat | `agr bench --repeat <n>` runs each test case N times per config; useful for pass@k metrics and detecting flaky tests; result summary shows solve rate across all trials |
| init --model/--provider | `agr init --model <model>` and `agr init --provider <provider>` customize the scaffolded `agent.yaml` for non-default providers (e.g. `agr init --model gpt-4o --provider openai`) |
| temperature override | `agr run --temperature <n>` and `agr bench --temperature <n>` override the temperature for all agent configs without editing YAML; use `--temperature 0` for deterministic runs when debugging flakiness |
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
| list --sort | `agr list --sort <field>` sorts runs by `cost`, `duration`, or `steps` (all descending); default `date` preserves the previous newest-first order; applied after all filters and `--limit`; combinable with `--plain`, `--json`, `--model`, and any other filter |
| status --model | `agr status --model <substring>` restricts the DB summary to runs where the agent model contains the given substring (case-insensitive); completes the `--model` filter symmetry across `agr list` and `agr status`; combinable with `--by-config`, `--by-test-case`, `--since`, and `--json` |
| run --dry-run | `agr run <name> --dry-run` prints the resolved test case, agent config, model, provider, sandbox, and any override flags without executing the run; combine with `--json` to emit a machine-readable object; mirrors `agr bench --dry-run` for single-run inspection and config validation |
| status --by-model | `agr status --by-model` shows a per-model breakdown (solve rate, avg cost, avg duration, avg tokens) sorted by solve rate; ideal for comparing haiku vs opus vs sonnet across all runs; combinable with `--since`, `--test-case`, `--config`, `--top`, and `--json` |
| status --by-sandbox | `agr status --by-sandbox` shows a per-sandbox breakdown (solve rate, avg cost, avg duration) sorted by solve rate; useful for validating that docker vs e2b results are consistent; combinable with `--since`, `--test-case`, `--config`, and `--json` |
| list --all | `agr list --all` loads every run from the database ignoring the default 100-run cap; combinable with `--plain`, `--json`, `--sort`, `--model`, and all other filters; useful for full-history exports (`agr list --all --json | jq length`) |
| bench --config-filter | `agr bench --suite tasks/ --configs-dir ./agents --config-filter fast` filters loaded configs by name substring after loading from `--configs-dir` or `--manifest`; useful for running a subset without editing files or listing paths; prints "N of M matched" when filter reduces the set |
| export --model --sort | `agr export runs --model haiku --sort cost` filters exported runs by model substring and sorts rows by cost, duration, or steps (descending) before applying `--limit`; mirrors the `--model` / `--sort` flags on `agr list` |
| list --matrix-id --last-matrix | `agr list --last-matrix --plain` scopes the run browser to a single bench matrix sweep; `--matrix-id <id>` pins to a specific sweep; mirrors the same flags on `agr export runs` so you can inspect a sweep interactively without exporting first |
| status --matrix-id --last-matrix | `agr status --last-matrix --by-model` restricts all stats to the most recent bench sweep; `--matrix-id <id>` pins to a specific sweep; combinable with all breakdown flags (`--by-config`, `--by-model`, `--by-sandbox`, etc.) |
| list --sandbox | `agr list --sandbox e2b` filters the run list to runs with a matching sandbox provider (substring match); mirrors `agr status --by-sandbox` for filter symmetry; combinable with all other `agr list` flags |
| list --error | `agr list --error timeout` filters runs whose error message contains the given substring; useful for grouping and counting runs by failure type without manual grep |
| status --sandbox | `agr status --sandbox e2b --by-model` restricts all stats to a specific sandbox provider; complements `--by-sandbox` breakdown and is combinable with `--model`, `--matrix-id`, and other filters |
| agr count | `agr count --failed --since 24h` prints a plain integer count of matching runs; supports the same filters as `agr list`; `--json` emits `{total, passed, failed, dbPath}`; useful for CI gates and shell conditions |
| export --sandbox --error | `agr export runs --sandbox e2b --error timeout` completes filter parity with `agr list`; both flags accept case-insensitive substrings and print a matching-count line before writing the file |
| trace --model --passed --failed | `agr trace --last --model haiku --failed` scopes `--last` to the most recent run matching model + outcome; all three new flags combine with existing `--test-case` and `--config` |
| status --by-matrix | `agr status --by-matrix` shows a per-sweep breakdown sorted newest-first (total, solve rate, avg cost); useful for tracking how solve rate evolves across successive bench runs; combinable with `--top` and all filter flags |

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
