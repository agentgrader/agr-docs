# Packages Architecture

The Agentgrader framework is built as a Turborepo monorepo. It contains eight interdependent packages that all live neatly under the `packages/` directory.

## Architecture Diagram

The dependencies flow upward from the store all the way to the CLI:

```mermaid
graph TD
    Store["@agentgrader/store"] --> Core["@agentgrader/core"]
    Core --> SandboxDocker["@agentgrader/sandbox-docker"]
    Core --> AgentOpenRouter["@agentgrader/agent-openrouter"]
    Core --> ScorerStatic["@agentgrader/scorer-static"]
    Core --> ScorerLlmJudge["@agentgrader/scorer-llm-judge"]
    Core --> Optimizer["@agentgrader/optimizer"]
    Core --> CLI[agentgrader]
    Store --> CLI
    SandboxDocker --> CLI
    AgentOpenRouter --> CLI
    ScorerStatic --> CLI
    ScorerLlmJudge --> CLI
    Optimizer --> CLI
```

## Packages Overview

### `@agentgrader/store`
This is the SQLite persistence layer. It is built using Drizzle ORM and `better-sqlite3`. This package handles all your run records, calculates costs, and tracks telemetry.

### `@agentgrader/core`
Think of this as the core engine. It defines the central interfaces, runner abstractions, schemas, and the scoring logic like CommandScorer, AssertionScorer, RegressionScorer, DiffScorer, and LocalizationScorer. Naturally, it depends on the `store` package.

### `@agentgrader/sandbox-docker`
This is our default Sandbox Provider that utilizes local Docker containers. It expertly manages the lifecycle of spinning up isolated execution environments, copying over fixture directories, and orchestrating shell execution. It depends on `core`.

### `@agentgrader/agent-openrouter`
This is the adapter for OpenRouter and OpenAI. It evaluates tasks using standard large language models by bridging the Agentgrader agent interfaces directly to the OpenRouter and OpenAI APIs. It depends on `core`.

### `@agentgrader/scorer-static`
An additive, non-blocking quality scorer. It never fails a run - it annotates `metrics["static-quality"]` with deterministic code-quality signals computed from the agent's diff: lines changed, files touched, `TODO`/`FIXME` markers introduced, and Biome lint violations on the changed files. It depends on `core` and is wired into `agr bench` by default.

### `@agentgrader/scorer-llm-judge`
Another additive, non-blocking scorer. It asks an LLM (Anthropic Claude Haiku or OpenAI GPT-4o-mini by default, via the AI SDK's `generateObject`) to rate the agent's diff from 0-1 for correctness and code quality, annotating `metrics["llm-judge"]`. It depends on `core` and degrades gracefully (no `quality` field) if the diff is empty or the judge call fails.

### `@agentgrader/optimizer`
Provides the matrix-sweep helpers behind `agr bench --matrix`: `expandMatrix()` turns a matrix YAML into the cartesian product of agent configs, `aggregateResults()` groups runs by agent config into solve rate / cost / quality averages, and `paretoFront()` filters those aggregates down to the Pareto-optimal set. It depends on `core`.

### `agentgrader` (CLI)
This is the main CLI binary known as `agr`. It uses Ink to build a fantastic terminal dashboard and handles commands like `bench`, `run`, `validate`, `import-pr`, and `trace`. Because it is the top layer, it depends on all the other packages.
