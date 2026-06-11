# Core Concepts

When you start using Agentgrader, there are a few important concepts that form the backbone of how everything works together. Let's break them down clearly.

## 1. The Test Case (`agr.yaml`)

Think of a test case as the specific challenge you are giving to your agent. It lives inside a folder containing an `agr.yaml` file and a `fixture/` directory. The fixture folder holds the starting codebase that your agent will try to fix or modify.

```yaml
# agr.yaml
name: add-error-handling
description: fetchWithRetry() crashes on network timeout. Please make it resilient.
fixture: ./fixture          # The path to the starting codebase, relative to this file
prompt: |
  The function fetchWithRetry() in src/client.ts throws an unhandled error when
  the network times out. Add proper error handling so it retries up to 3 times
  (total of 4 attempts: 1 initial and 3 retries), then throws the error if all fail.
  Please do not change the signature of fetchWithRetry.
success:
  - run: npm install && npm test   # The command used to verify the solution
    expect: { exit_code: 0 }       # What a successful result looks like
  - assert: steps <= 10            # The agent must finish in 10 tool calls or less
  - assert: cost_usd <= 0.05       # The entire run must cost less than 5 cents
timeout_seconds: 300
```

### Types of Success Criteria:
*   `run` combined with `expect.exit_code`: This runs a shell command right in the sandbox and checks the final exit code.
*   `assert: steps <= N`: This checks how many tool calls the agent used to find a solution.
*   `assert: cost_usd <= N`: This checks the total monetary cost of the model during the run.

## 2. Agent Config (`baseline.yaml`)

This configuration file tells Agentgrader exactly which model to use and how it should behave.

```yaml
id: baseline
name: Baseline Agent
model: gpt-4o-mini          # You can use any OpenRouter model string here
max_steps: 15               # This is a hard cap on ReAct loop iterations
temperature: 0.2
system_prompt: |
  You are a professional software developer. Solve the coding task in the sandbox.
  Use executeCommand to run tests. Use readFile and writeFile to edit code.
  Call submit when all tests pass.
```

The `model` field is incredibly flexible. By default Agentgrader routes through OpenRouter, so you can use any model available there, such as:
*   `openai/gpt-4o`
*   `anthropic/claude-opus-4`
*   `google/gemini-2.5-pro`
*   `meta-llama/llama-3.1-70b-instruct`

If you'd rather call a provider directly, set `provider: openai` (with an `OPENAI_API_KEY`) or `provider: anthropic` (with an `ANTHROPIC_API_KEY`) and use that provider's native model name, e.g. `provider: anthropic` with `model: claude-opus-4`.

## 3. The Sandbox

For every single run, Agentgrader provisions a fresh and completely isolated Docker container. The starting code from the fixture folder is copied directly into `/app` inside the container. 

The agent interacts with this environment using four core tools:

| Tool | What it does |
|---|---|
| `executeCommand(command)` | Runs a bash command inside the `/app` folder of the container. |
| `readFile(path)` | Reads the contents of a specific file. |
| `writeFile(path, content)` | Writes your new content into a specific file. |
| `submit({ summary })` | Lets the framework know that the agent believes the task is complete. |

Once the agent calls `submit()` or the run hits the timeout limit, Agentgrader steps in and runs the scorers to verify the final result.

## 4. Scoring

Agentgrader handles scoring automatically at the end of each evaluation. The core blocking scorers determine `passed`/`failed`:

*   **CommandScorer**: This runs every `run:` criterion as a shell command within the sandbox and validates the exit code.
*   **AssertionScorer**: This evaluates mathematical `assert:` expressions. You have access to variables like `steps`, `cost_usd`, `tokens_in`, and `tokens_out` to create very flexible success criteria.
*   **RegressionScorer**: For test cases with `test_command`/`fail_to_pass`/`pass_to_pass`, this re-runs the test suite and checks that the previously-failing `fail_to_pass` tests now pass and the `pass_to_pass` tests didn't regress. It also fails the run if the agent edited any `forbid_modified` paths (a tamper guard).
*   **DiffScorer** and **LocalizationScorer**: When `solution`/`expected_files` are configured, these compare the agent's diff against the gold patch and report precision/recall/F1 on which files were touched.

## 5. Quality Scorers & the Optimizer

On top of the pass/fail scorers above, `agr bench` also runs **additive, non-blocking** quality scorers. These never affect `passed` - they just attach extra data to `metrics` for reporting and optimization:

*   **StaticQualityScorer** (`@agentgrader/scorer-static`, always on): deterministic signals computed from the diff - lines changed, files touched, `TODO`/`FIXME` markers introduced, and Biome lint violations. Recorded under `metrics["static-quality"].quality`.
*   **LlmJudgeScorer** (`@agentgrader/scorer-llm-judge`, opt-in via `extraScorers`): asks an LLM to rate the diff from 0-1 for correctness and code quality. Recorded under `metrics["llm-judge"].quality`.

Inspect these for a single run with [`agr trace <runId> --quality`](/reference/cli#agr-trace).

### Optimizer matrices

To sweep many agent configurations at once - different models, temperatures, system prompts, or toolkits - define a **matrix** YAML and pass it to `agr bench --matrix`:

```yaml
# matrix.yaml
name: model-comparison
base:
  max_steps: 15
  temperature: 0.2
dimensions:
  model:
    - anthropic/claude-3.5-sonnet
    - anthropic/claude-3.5-haiku
    - openai/gpt-4o-mini
  temperature:
    - 0.2
    - 0.7
```

`agr bench --matrix matrix.yaml --suite ./examples/suites/typescript-bugs` expands every combination of `dimensions` (here, 3 models x 2 temperatures = 6 agent configs) on top of `base`, runs the full suite against each, and tags every run with a shared `matrixId`. Afterwards it prints a "MATRIX SUMMARY" table with each config's solve rate, average cost, and (if `StaticQualityScorer` ran) average lint violations, marking the Pareto-optimal configs with `*`.
