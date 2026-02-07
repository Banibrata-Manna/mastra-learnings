# Manual Mastra Project Structure

This guide details how to manually set up a Mastra project structure without using CLI generators.

## Root Directory

Your project root should contain configuration files and the source directory.

```
my-mastra-project/
├── .env                  # Environment variables (API keys, DB URLs)
├── .gitignore            # Git ignore file
├── package.json          # Dependencies and scripts
├── tsconfig.json         # TypeScript configuration
└── src/                  # Source code
```

### 1. package.json

Minimal `package.json` setup:

```json
{
  "name": "my-mastra-project",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "mastra dev",
    "build": "mastra build",
    "start": "mastra start"
  },
  "dependencies": {
    "@mastra/core": "^0.1.33",
    "@mastra/cli": "^0.1.33",
    "zod": "^3.22.4"
  },
  "devDependencies": {
    "typescript": "^5.3.3",
    "@types/node": "^20.11.0"
  }
}
```

### 2. tsconfig.json

Recommended TypeScript configuration:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "noEmit": true,
    "outDir": "dist"
  },
  "include": ["src/**/*"]
}
```

## Source Directory Structure (`src/`)

The core of your application lives in `src/mastra`.

```
src/
└── mastra/
    ├── index.ts          # Main entry point (exports `mastra` instance)
    ├── agents/           # Agent definitions
    │   └── index.ts      # (Optional) Export all agents
    ├── tools/            # Custom tools
    │   └── index.ts      # (Optional) Export all tools
    ├── workflows/        # workflow definitions
    │   └── index.ts      # (Optional) Export all workflows
    └── lib/              # Shared utilities (types, constants)
```

### 1. src/mastra/index.ts

This is the most critical file. It must export your `Mastra` instance.

```typescript
import { Mastra } from '@mastra/core';

// Import your components
// import { myAgent } from './agents/myAgent';

export const mastra = new Mastra({
  agents: {
    // myAgent
  },
  logger: {
    name: 'Mastra',
    level: 'info'
  }
});
```

### 2. src/mastra/agents/

Organize agents into individual files:

```typescript
// src/mastra/agents/myAgent.ts
import { Agent } from '@mastra/core';

export const myAgent = new Agent({
  name: 'My Agent',
  instructions: 'You are a helpful assistant.',
  model: {
    provider: 'openai',
    name: 'gpt-4'
  }
});
```

## Environment Variables (.env)

Create a `.env` file in the root:

```env
# API Keys for LLM Providers
OPENAI_API_KEY=sk-...

# Database Configuration (if using Postgres)
DATABASE_URL=postgresql://user:password@localhost:5432/mastra
```

## Summary Checklist

1.  [ ] Create root directory.
2.  [ ] Initialize `package.json` and `tsconfig.json`.
3.  [ ] Create `src/mastra/` directory structure.
4.  [ ] Create `src/mastra/index.ts` and export `mastra` instance.
5.  [ ] Define at least one agent or workflow.
6.  [ ] Set up `.env` with necessary keys.
