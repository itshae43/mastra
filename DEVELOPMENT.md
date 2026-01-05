# Mastra Development Guide

This guide provides instructions for developers who want to contribute to or work with the Mastra codebase.

## Prerequisites

- **Node.js** (v22.13.0 or later)
- **pnpm** (v10.18.0 or later) - Mastra uses pnpm for package management
- **Docker** (for local development services) - Only needed for a subset of tests, not required for general development

## Repository Structure

Mastra is organized as a monorepo with the following key directories:

- **packages/** - Core packages that make up the Mastra framework
  - **core/** - The foundation of the Mastra framework that provides essential components including agent system, LLM abstractions, workflow orchestration, vector storage, memory management, and tools infrastructure
  - **cli/** - Command-line interface for creating, running, and managing Mastra projects, including the interactive playground UI for testing agents and workflows
  - **deployer/** - Server infrastructure and build tools for deploying Mastra applications to various environments, with API endpoints for agents, workflows, and memory management
  - **rag/** - Retrieval-augmented generation tools for document processing, chunking, embedding, and semantic search with support for various reranking strategies
  - **memory/** - Memory systems for storing and retrieving conversation history, vector data, and application state across sessions
  - **evals/** - Evaluation frameworks for measuring LLM performance with metrics for accuracy, relevance, toxicity, and other quality dimensions
  - **mcp/** - Model Context Protocol implementation for standardized communication with AI models, enabling tool usage and structured responses across different providers

- **deployers/** - Platform-specific deployment adapters for services like Vercel, Netlify, and Cloudflare, handling environment configuration and serverless function deployment
- **stores/** - Storage adapters for various vector and key-value databases, providing consistent APIs for data persistence across different storage backends

- **voice/** - Speech-to-text and voice processing capabilities for real-time transcription and voice-based interactions
- **client-sdks/** - Client libraries for different platforms and frameworks that provide type-safe interfaces to interact with Mastra services
- **examples/** - Example applications demonstrating various Mastra features including agents, workflows, memory systems, and integrations with different frameworks

## Getting Started

### Setting Up Your Development Environment

1. **Clone the repository**:

   ```bash
   git clone https://github.com/mastra-ai/mastra.git
   cd mastra
   ```

2. **Enable corepack** (ensures correct pnpm version):

   ```bash
   corepack enable
   ```

3. **Install dependencies and build initial packages**:

   ```bash
   pnpm run setup
   ```

   This command installs all dependencies and builds the CLI package, which is required for other packages.

### Building Packages

If you run into the following error during a build:

```text
Error [ERR_WORKER_OUT_OF_MEMORY]: Worker terminated due to reaching memory limit: JS heap out of memory
```

you can increase Node’s heap size by prepending your build command with:

```bash
NODE_OPTIONS="--max-old-space-size=4096" pnpm build
```

- **Build all packages**:

  ```bash
  pnpm build
  ```

- **Build specific package groups**:

  ```bash
  pnpm build:packages         # All core packages
  pnpm build:deployers        # All deployment adapters
  pnpm build:combined-stores  # All vector and data stores
  pnpm build:speech           # All speech processing packages
  pnpm build:clients          # All client SDKs
  ```

- **Build individual packages**:
  ```bash
  pnpm build:core             # Core framework package
  pnpm build:cli              # CLI and playground package
  pnpm build:deployer         # Deployer package
  pnpm build:rag              # RAG package
  pnpm build:memory           # Memory package
  pnpm build:evals            # Evaluation framework package
  pnpm build:docs-mcp         # MCP documentation server
  ```

## Testing

Mastra uses Vitest for testing. You can run all tests or only specific packages.

- All tests:
  ```bash
  pnpm test
  ```
- Specific package tests:
  ```bash
  pnpm test:core             # Core package tests
  pnpm test:cli              # CLI tests
  pnpm test:rag              # RAG tests
  pnpm test:memory           # Memory tests
  pnpm test:evals            # Evals tests
  pnpm test:clients          # Client SDK tests
  pnpm test:combined-stores  # Combined stores tests
  ```
- Watch mode (for development):
  ```bash
  pnpm test:watch
  ```

Some tests require environment variables to be set. If you're unsure about the required variables, ask for help in the pull request or wait for CI to run the tests.

Create a `.env` file in the root directory with the following content:

```text
OPENAI_API_KEY=
COHERE_API_KEY=
PINECONE_API_KEY=
CLOUDFLARE_ACCOUNT_ID=
CLOUDFLARE_API_TOKEN=
DB_URL=postgresql://postgres:postgres@localhost:5432/mastra
```

Afterwards, start the development services:

```bash
pnpm run dev:services:up
```

## Testing Local Changes

When you make changes to Mastra packages and want to test them with agents or workflows, use the `examples/agent` directory.

### Using examples/agent for Local Testing

The `examples/agent` directory is configured to use your local Mastra packages through pnpm overrides. This means any changes you make to Mastra will be reflected immediately when you run the example.

1. **Build the packages you modified**:

   ```bash
   # From the monorepo root
   pnpm build:core        # If you modified the core package
   pnpm build:memory      # If you modified the memory package
   # Or build all packages at once
   pnpm build
   ```

2. **Add your agent or workflow to `examples/agent`**:

   Navigate to `examples/agent/src/mastra/` and add your code:
   - **For agents**: Create or edit files in `src/mastra/agents/`
   - **For workflows**: Create or edit files in `src/mastra/workflows/`
   - **For tools**: Create or edit files in `src/mastra/tools/`

3. **Register your agent/workflow in the Mastra config**:

   Edit `examples/agent/src/mastra/index.ts` to register your agent or workflow:

   ```typescript
   // Import your agent or workflow
   import { myTestAgent } from './agents/my-test-agent';
   import { myTestWorkflow } from './workflows/my-test-workflow';

   const config = {
     agents: {
       // ... existing agents
       myTestAgent, // Add your agent here
     },
     workflows: {
       // ... existing workflows
       myTestWorkflow, // Add your workflow here
     },
     // ... rest of config
   };
   ```

4. **Start Mastra Studio**:

   ```bash
   cd examples/agent
   pnpm mastra:dev
   ```

   This will start the Mastra Studio at `http://localhost:4111`, where you can:
   - Test your agents interactively
   - Run and debug workflows
   - View execution traces
   - Test tools

5. **Iterate on your changes**:
   - Make changes to the Mastra packages
   - Rebuild the modified packages (`pnpm build:core`, etc.)
   - The changes will be reflected in the running example
   - Refresh or restart the studio to see your changes

### Example Workflow

Here's a typical workflow for fixing a bug in Mastra:

1. Find a bug in your own project that uses Mastra
2. Clone the Mastra repository locally
3. Make your fix in the relevant package (e.g., `packages/core`)
4. Build the package: `pnpm build:core`
5. Add a test agent/workflow to `examples/agent/src/mastra/`
6. Register it in `examples/agent/src/mastra/index.ts`
7. Run `cd examples/agent && pnpm mastra:dev`
8. Test your fix in the Mastra Studio at `http://localhost:4111`
9. Once verified, write tests for your fix and submit a PR

### Other Example Directories

While `examples/agent` is the primary directory for testing, you can also use other example directories. Each example that has a `mastra:dev` script can be used similarly:

- `examples/a2a` - Agent-to-agent communication
- `examples/agui` - Agent UI examples
- `examples/ai-sdk-v5` - AI SDK v5 integration

All example directories use pnpm overrides to link to local packages, so they will all use your local changes.

## Contributing

1. **Create a branch for your changes**:

   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes and ensure tests pass**:

   ```bash
   pnpm test
   ```

3. **Create a changeset** (for version management):

   ```bash
   pnpm changeset
   ```

   Follow the prompts to describe your changes.

4. **Open a pull request** with your changes.

## Documentation

The documentation site is built from the `/docs` directory. Follow its [documentation guide](./docs/CONTRIBUTING.md) for instructions on contributing to the docs.

## Need Help?

Join the [Mastra Discord community](https://discord.gg/BTYqqHKUrf) for support and discussions.
