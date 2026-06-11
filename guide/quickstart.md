# Quickstart

It is super easy to get started with Agentgrader. You can have your first benchmark running in just a few minutes.

Before we jump in, make sure you have a few things ready:
*   Node.js (v18+) or [Bun](https://bun.sh/) installed on your machine.
*   [Docker](https://www.docker.com/) installed and currently running.
*   An API key from OpenRouter, OpenAI, or Anthropic.

## 1. Install Agentgrader

Install the CLI globally with npm or Bun. This puts the `agr` command directly on your `PATH` - no cloning or building required.

```bash
npm install -g agentgrader
# or
bun add -g agentgrader
```

```bash
agr --help
```

## 2. Set Up Your Environment

By default, Agentgrader uses OpenRouter as the gateway for Large Language Models. You just need to set your API key in your environment variables. If you do not have an OpenRouter key, it will smoothly fall back to direct OpenAI as long as you provide an `OPENAI_API_KEY`.

```bash
export OPENROUTER_API_KEY=sk-or-...
```

Prefer to talk to Claude directly instead of going through OpenRouter? Set `provider: anthropic` in your agent config (see the [Agent Config reference](/reference/agent-config-yaml)) and export an `ANTHROPIC_API_KEY` - Agentgrader will call the Anthropic API directly using that key.

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

## 3. Get the Example Suite

The `agr` CLI itself doesn't bundle any test cases - those live in the GitHub repo. Grab a copy to try Agentgrader against the included example suite and agent configs:

```bash
git clone --depth 1 https://github.com/agentgrader/agr agentgrader-examples
cd agentgrader-examples
```

## 4. Run Your First Benchmark

Now for the fun part! Run an example benchmark that tests a baseline agent against a collection of TypeScript bugs:

```bash
agr bench \
  --suite examples/suites/typescript-bugs/ \
  --configs examples/configs/baseline.yaml
```

Sometimes you just want to focus on a single test case instead of running a full benchmark. In that case, you can simply use:
```bash
agr run examples/suites/typescript-bugs/add-error-handling/agr.yaml \
  --config examples/configs/baseline.yaml
```

## 5. Programmatic API

If you are a developer looking to integrate Agentgrader directly into your own CI/CD pipelines, tools, or custom evaluation scripts, you don't have to use the CLI. Agentgrader has a powerful programmatic API!

Check out the [Programmatic API](/advanced/programmatic-api) guide to learn how to import `@agentgrader/core` and use the `runSingle` and `runBenchmark` functions directly in your code.
