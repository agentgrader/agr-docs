# Programmatic API

If you want to embed Agentgrader right into your existing CI pipelines or tools, you can easily do that using the TypeScript Programmatic API exposed by `@agentgrader/core`.

## `runSingle()`

This function lets you run a single test case programmatically.

```typescript
import { runSingle } from "@agentgrader/core";
import { DockerSandboxProvider } from "@agentgrader/sandbox-docker";
import { OpenRouterAgentAdapter } from "@agentgrader/agent-openrouter";
import { StaticQualityScorer } from "@agentgrader/scorer-static";
import { initDb } from "@agentgrader/store";
import { randomUUID } from "crypto";

const result = await runSingle({
  testCase: {
    id: "my-task",
    name: "my-task",
    fixture: "./my-fixture",
    prompt: "Fix the bug in src/index.ts",
    success: [{ run: "npm test", expect: { exit_code: 0 } }],
    timeout_seconds: 300,
  },
  agentConfig: {
    id: "baseline",
    name: "baseline",
    model: "gpt-4o-mini",
    max_steps: 20,
  },
  adapter: new OpenRouterAgentAdapter(),
  sandboxProvider: new DockerSandboxProvider(),
  db: initDb(),          // This is optional. Just omit it if you want to skip saving to the database.
  runId: randomUUID(),
  extraScorers: [new StaticQualityScorer()], // Optional, additive, non-blocking quality scorers
  matrixId: undefined,   // Optional. Tags this run as belonging to an optimizer matrix sweep.
});

console.log(result.passed);    // This returns a boolean
console.log(result.costUsd);   // The total cost in USD, for example 0.012
console.log(result.stepsCount); // How many tool iterations the agent took
console.log(result.finalDiff); // The git diff showing exactly what changed
console.log(result.metrics);   // e.g. { "static-quality": { quality: { diffLines, filesModified, ... } } }
```

## `runBenchmark()`

This function is for when you want to orchestrate multiple test cases against multiple agent configurations all at the same time.

```typescript
import { runBenchmark } from "@agentgrader/core";

const result = await runBenchmark({
  testCases: [...],      // An array of TestCase objects
  agentConfigs: [...],   // An array of AgentConfig objects
  adapter: new OpenRouterAgentAdapter(),
  sandboxProvider: new DockerSandboxProvider(),
  concurrency: 3,        // How many should run in parallel
  onRunUpdate: (run) => {
    // This gives you streaming updates as the runs progress
    console.log(`${run.testCaseId}: ${run.status}`);
  },
  extraScorers: [new StaticQualityScorer()], // Optional, applied to every test case x config combination
  matrixId: undefined,   // Optional. Tags every resulting run with a shared matrix ID.
});
```

## Optimizer Matrix Sweeps

`@agentgrader/optimizer` provides the helpers behind `agr bench --matrix`. You can use them directly to expand a matrix into agent configs, then aggregate and rank the resulting runs:

```typescript
import { expandMatrix, aggregateResults, paretoFront } from "@agentgrader/optimizer";
import { runBenchmark } from "@agentgrader/core";
import { getRunsByMatrixId } from "@agentgrader/store";
import { randomUUID } from "crypto";

// 1. Expand a matrix definition into the cartesian product of agent configs
const agentConfigs = expandMatrix({
  name: "model-comparison",
  base: { max_steps: 15, temperature: 0.2 },
  dimensions: {
    model: ["anthropic/claude-3.5-sonnet", "anthropic/claude-3.5-haiku", "openai/gpt-4o-mini"],
    temperature: [0.2, 0.7],
  },
});

// 2. Run the benchmark, tagging every run with a shared matrixId
const matrixId = randomUUID();
await runBenchmark({ testCases, agentConfigs, adapter, sandboxProvider, db, matrixId });

// 3. Aggregate solve rate / cost / quality per agent config, then find the Pareto front
const runs = await getRunsByMatrixId(db, matrixId);
const aggregates = aggregateResults(runs, agentConfigs);
const front = paretoFront(aggregates); // optimal across solveRate, avgCostUsd, and (if present) avgQuality.linterViolations
```
