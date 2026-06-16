# Quickstart

You can run your first agent evaluation in a few minutes. Install the CLI from npm, create a minimal test case on disk, and run it.

## Prerequisites

- Node.js 18+ or [Bun](https://bun.sh/)
- [Docker](https://www.docker.com/) installed and running
- An API key from [OpenRouter](https://openrouter.ai/),  [OpenAI](https://platform.openai.com/login?next=/settings/organization/api-keys), or Anthropic [Anthropic API Keys](https://platform.claude.com/docs/en/api/admin/api_keys/retrieve)

## 1. Install the CLI

Install globally so `agr` is on your `PATH`, or invoke it per-project without a global install:

::: code-group

```bash [npm]
npm install -g agentgrader
agr --help
```

```bash [bun]
bun add -g agentgrader
agr --help
```

```bash [bunx / npx]
bunx agentgrader --help
# or: npx agentgrader --help
```

:::

## 2. Set your API key

The scaffold created in the next step uses `provider: anthropic` with `claude-haiku-4-5`, so `ANTHROPIC_API_KEY` is all you need. The CLI loads a `.env` file from the current working directory automatically:

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...
```

Or export it in your shell:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

To use OpenRouter or OpenAI directly, set `provider: openrouter` or `provider: openai` in your agent config (see [Agent Config](/reference/agent-config-yaml)) and provide `OPENROUTER_API_KEY` or `OPENAI_API_KEY` instead.

## 3. Scaffold a project

Create a project directory and let `agr init` generate a ready-to-run agent config and a tiny, self-contained test case:

```bash
mkdir my-benchmark && cd my-benchmark
agr init
```

This creates:

- **`agent.yaml`**: a baseline agent config (`claude-haiku-4-5`, `provider: anthropic`, `max_steps: 15`)
- **`tasks/hello-world/agr.yaml`**: a test case that asks the agent to implement `add(a, b)` in `math.js`
- **`tasks/hello-world/fixture/`**: the starter project for that test case (`math.js` + `math.test.js`, checked with Node's built-in test runner, so no `npm install` is needed inside the sandbox)

If you'd rather start from an empty project and write your own test cases by hand, use `agr init --blank` instead. It writes only `agent.yaml` and an empty `tasks/` directory; see [Core Concepts](/guide/concepts) for the `agr.yaml` schema.

## 4. Run your first evaluation

From `my-benchmark/` (where your `.env` lives):

```bash
agr run hello-world --verbose
```

`hello-world` is the test case's `name:` from `tasks/hello-world/agr.yaml`, so there's no need to type out the full path. Because `agent_config` is set in `agr.yaml`, Agentgrader loads `../../agent.yaml` automatically. You can still override with `--config` when experimenting with a different agent.

Expected flow:

1. Agentgrader copies the fixture into a fresh Docker container.
2. The agent reads `math.js`, implements `add()`, and runs `node --test math.test.js`.
3. A run summary prints pass/fail, step count, cost, and duration.

Example verbose output:

```
[step 1] tool_call: readFile({"path":"math.js"})
[step 2] tool_result: readFile -> function add(a, b) { ...
[step 3] tool_call: writeFile({"path":"math.js", ...})
[step 4] tool_call: executeCommand({"command":"node --test math.test.js"})
...
```

Once the run finishes, inspect the trace with `agr trace --last` (or add `--tools` to see a tool-call breakdown). Once you've added more test cases under `tasks/`, run `agr list-tests` to see every test case's `name`, path, and description, then refer to any of them the same way: `agr run <name>`.

## 5. Run a benchmark (optional)

Point `agr bench` at a directory of test cases and one or more agent configs:

```bash
agr bench \
  --suite tasks/ \
  --config agent.yaml
```

`--config` is a shorthand alias for `--configs` when you only have a single agent config. Use `--concurrency 2` (default) to run evaluations in parallel.

## Next steps

- [Core Concepts](/guide/concepts): test cases, agent configs, scoring, and where results are stored
- [Best Practices](/guide/best-practices): CI gates, `fail_to_pass`/`pass_to_pass`, matrix sweeps, troubleshooting
- [CLI Reference](/reference/cli): full command and flag reference
- [Programmatic API](/advanced/programmatic-api): embed evaluations in your own Node.js or Bun code
