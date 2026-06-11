# Agent Config (baseline.yaml)

The Agent Config file is where you decide exactly which model to use. It lets you fine tune hyperparameters and provide the system prompt that the agent will use during its evaluation.

```yaml
id: baseline
name: Baseline Agent
provider: openrouter   # optional - "openrouter" (default), "openai", or "anthropic"
model: gpt-4o-mini
max_steps: 15
temperature: 0.2
system_prompt: |
  You are a professional software developer. Solve the coding task in the sandbox.
  Use executeCommand to run tests. Use readFile and writeFile to edit code.
  Call submit when all tests pass.
```

## Schema Reference

### `id`
**Type:** `string`  
A unique string identifier for this particular agent configuration.

### `name`
**Type:** `string`  
A friendly, human readable name for the configuration so you can easily identify it.

### `model`
**Type:** `string`  
The specific LLM you want to evaluate. This field is super flexible and supports any valid OpenRouter model string. For example, you can use `openai/gpt-4o`, `anthropic/claude-opus-4`, or `google/gemini-2.5-pro`.

### `max_steps`
**Type:** `number`  
This sets a hard limit on the number of ReAct loop iterations the agent is allowed to execute before it is forced to stop.

### `temperature`
**Type:** `number`  
The sampling temperature for the model, which controls how creative or deterministic the outputs will be.

### `system_prompt`
**Type:** `string`  
This is the core prompt injected right into the LLM at the very beginning of the run. It is where you define the persona and explain which tools the agent has available to use.
