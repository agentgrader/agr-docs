# Custom Sandbox Provider

The default CLI sandbox is local Docker (`--sandbox docker`). Agentgrader also ships a built-in [E2B](https://e2b.dev/) provider:

```bash
export E2B_API_KEY=...
agr run tasks/hello-world/agr.yaml --config agent.yaml --sandbox e2b
agr bench --suite test-cases/ --config agent.yaml --sandbox e2b
```

If you prefer another cloud provider (Daytona, Firecracker, etc.), implement the `SandboxProvider` interface below and pass your provider into the programmatic API, or wire it into a custom CLI wrapper.

You just need to fulfill the `SandboxProvider` interface. This interface handles provisioning the environment and returns a `SandboxHandle`, which details exactly how to execute commands or interact with files in that specific environment.

## Implementation Example

```typescript
import type { SandboxProvider, SandboxHandle } from "@agentgrader/core";

export class MyCloudProvider implements SandboxProvider {
  readonly name = "my-cloud";

  async create(opts: { image?: string; gitSnapshot?: string }): Promise<SandboxHandle> {
    // 1. Spin up a cloud environment using the specified image
    // 2. Clone the snapshot if one is provided

    return {
      exec: async (cmd) => { 
        // Put your logic here to run bash commands
      },
      readFile: async (path) => { 
        // Logic to read files
      },
      writeFile: async (path, content) => { 
        // Logic to write content
      },
      gitDiff: async () => { 
        // Returns the diff of the entire workspace
      },
      destroy: async () => { 
        // Teardown and clean up the environment
      },
    };
  }
}
```

Once your custom provider is ready, you can pass it right into the Programmatic API using either `runSingle` or `runBenchmark`.
