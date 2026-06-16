# Toolkits

Toolkits ship custom CLI tools and Agent Skills into the sandbox. Reference toolkit directories from `toolkits:` in [agent.yaml](/reference/agent-config-yaml) or [agr.yaml](/reference/test-case-yaml).

Layout:

```
toolkits/my-tools/
  bin/                    # executables on PATH inside the sandbox
  .claude/skills/*/SKILL.md
  setup.sh                # optional: run once at sandbox init
```

See [ACP Agent Adapter](/advanced/acp-agent) for how AI SDK and ACP adapters surface skills to agents.

## Scaffolding and auditing

```bash
agr toolkit-add find-usages --dir ./toolkits/jetbrains-tools
agr toolkit-list ./toolkits/jetbrains-tools
agr toolkit-list ./toolkits/jetbrains-tools --check-config agent.yaml
agr validate-toolkit ./toolkits/jetbrains-tools --strict
```

`validate-toolkit` and `agr bench --strict-toolkits` run the same security audit as `agr validate --audit-toolkits`.

## Measuring adoption

`agr trace <runId> --tools` and the `agr bench` **TOOL USAGE BY CONFIG** footer break down calls by the first word of the command (`executeCommand:pytest`, `terminal/create:find-usages`, etc.).

With `provider: anthropic`/`openai`, each skill is also registered as `tool_<name_with_underscores>`. `require_tools_before_submit` and `track_tools` credit both direct calls and wrapped usage.

Set [`require_tools_before_submit`](/reference/agent-config-yaml#require-tools-before-submit) to annotate `metrics["tool-adoption"]`. Set [`track_tools`](/reference/agent-config-yaml#track-tools) for optional tools without affecting pass/fail.

`agr trace --quality` shows per-tool `usedVia: direct | wrapped` when adoption was recorded.

## Design guidelines

**Slot new tools into existing workflow steps**: group optional tools with steps the agent already follows rather than adding standalone steps.

**Do not conclude from a single run**: adoption on trivial tasks varies run-to-run. Average over several runs or use tasks where verification is non-obvious.

**Redundant tools stay unused**: if a new tool overlaps an adopted one, fold its output into the adopted tool's response instead of adding a competing command.

**Exploration tools depend on the prompt**: if the prompt already states what the tool would discover, zero adoption is expected.

**Surgical-edit tools compete with readFile/writeFile**: file size alone rarely drives adoption; fold checks into adopted tools or use `require_tools_before_submit`.

## A/B testing a `toolkits` dimension with `--matrix`

When a matrix varies `dimensions.toolkits` but shares `base.system_prompt`, the no-toolkit arm may still be told about tools it lacks. Mitigate wasted `command not found` steps:

```
Note: in some sandboxes the tools above are not installed. If running one
of them fails with "command not found", don't retry it - immediately fall
back to `executeCommand`/`readFile`/`writeFile` to do that step manually
and continue with the rest of the workflow below.
```

See [Optimizer matrices](/guide/optimizer-matrices#toolkit-a-b-with-matrix).

## Toolkit setup hooks (`setup.sh`)

`setup.sh` runs once when the toolkit is injected (before the agent's first turn). Use it for dependencies missing from the base image (e.g. `pip install pytest`).

Agentgrader enforces a timeout (default 120s) on `setup.sh`. Failed setup aborts the run with a clear error.

Individual `bin/` scripts can also self-install dependencies as defense in depth.

## `cd`-prefix stripping in tool usage

`executeCommand`/`terminal/create` buckets strip a leading `cd <dir> &&` prefix so `cd /app && pytest` counts as `pytest`, not `cd`.

Wrapped tools count as used when they print a self-identifying `<name>: ...` marker line in output.

## Reference examples

Two complete, runnable toolkit examples ship under `examples/toolkits/`:

| Toolkit | Tools | Description |
|---|---|---|
| `examples/toolkits/code-search` | `find-todos`, `find-usages` | Grep-based symbol and annotation search |
| `examples/toolkits/git-context` | `recent-changes`, `file-log` | Git history tools for codebase orientation |

Both follow the standard layout: `bin/` scripts, `.claude/skills/*/SKILL.md` progressive-disclosure docs, and no external dependencies beyond git. Use them as starting points for your own toolkits.

To use `git-context` with an ACP agent:

```yaml
# agent.yaml
acp_command: claude
acp_args: [--acp]
toolkits:
  - examples/toolkits/git-context
```

## Next steps

- [Recipes: toolkit matrix A/B](/guide/recipes#toolkit-matrix-ab)
- [Debugging: trace --tools](/guide/debugging#toolkit-adoption)
- [ACP Agent Adapter](/advanced/acp-agent)
