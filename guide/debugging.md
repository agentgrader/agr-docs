# Debugging failed runs

Every completed run gets a UUID stored in `.agr/db.sqlite`.

## Quick commands

```bash
# Most recent run (no UUID needed)
agr trace --last
agr trace --last --quality
agr trace --last --tools

# Specific run by ID
agr trace <runId>
agr trace <runId> --quality
agr trace <runId> --tools

agr compare <runIdA> <runIdB> --only-diff
agr list
```

Use `--verbose` during `agr run` to watch tool calls live. Run IDs appear in bench/run output and in `agr list`.

## What to check first

1. **Run error**: sandbox, provider, or adapter failure (exit 1 from `agr` itself)
2. **CommandScorer**: did `success` `run:` commands pass inside the sandbox?
3. **AssertionScorer**: did the run exceed `steps` or `cost_usd` limits?
4. **RegressionScorer**: TAP test names in `fail_to_pass` / `pass_to_pass` (see [Troubleshooting](/guide/troubleshooting#tap-and-fail_to_pass))
5. **Quality metrics**: non-blocking signals under `metrics["static-quality"]` and `metrics["llm-judge"]`

```bash
agr trace <runId> --quality
```

## Side-by-side agent comparison

When two configs ran the same test case, compare traces to see where they diverged:

```bash
agr compare <runIdA> <runIdB> --only-diff
agr compare <runIdA> <runIdB> --only-diff --full
```

## Toolkit adoption

```bash
agr trace <runId> --tools
```

The `agr bench` **TOOL USAGE BY CONFIG** footer aggregates tool-call counts per config. For structured adoption metrics, set [`track_tools`](/reference/agent-config-yaml#track-tools) or [`require_tools_before_submit`](/reference/agent-config-yaml#require-tools-before-submit) in agent.yaml. Deep dive: [Toolkits guide](/guide/toolkits).

## Reset history

Delete `.agr/db.sqlite` to start fresh. Test case folders on disk are never modified.
