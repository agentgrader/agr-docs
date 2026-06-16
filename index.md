---
layout: home

hero:
  name: Agentgrader
  text: Benchmark AI coding agents
  tagline: Run agents against real programming tasks in Docker sandboxes. Score with objective test suites, track cost and tokens, and compare architectures in CI.
  image:
    src: /LGO.svg
    alt: Agentgrader Logo
  actions:
    - theme: brand
      text: Quickstart
      link: /guide/quickstart
    - theme: alt
      text: Best Practices
      link: /guide/best-practices
    - theme: alt
      text: GitHub
      link: https://github.com/agentgrader/agr

features:
  - icon: 🐳
    title: Real sandboxes
    details: Every run gets a fresh Docker container. Agents execute real commands and edit real files. No mocks.
  - icon: 📊
    title: Objective scoring
    details: Pass and fail come from test suites (npm test, pytest, go test) plus optional SWE-bench regression checks.
  - icon: 💰
    title: Cost tracking
    details: Token usage and USD cost per model are recorded for every run in a local SQLite database.
  - icon: 🔌
    title: Pluggable adapters
    details: Swap the AI SDK loop, OpenRouter, Anthropic, OpenAI, or external ACP agents (Claude Code, Cursor Agent) without changing core logic.
  - icon: ⚡
    title: Node and Bun
    details: Install globally or embed programmatically. Works on Node.js 18+ and Bun with the same packages.
  - icon: 🧪
    title: CI ready
    details: Validate YAML, gate on assert limits, run matrix sweeps, and integrate with GitHub Actions in minutes.
---

## Install in one line

::: code-group

```bash [npm]
npm install -g agentgrader
agr init my-benchmark && cd my-benchmark
```

```bash [bun]
bun add -g agentgrader
agr init my-benchmark && cd my-benchmark
```

:::

Set an API key in `.env`, then run your first evaluation:

```bash
agr run hello-world --verbose
```
