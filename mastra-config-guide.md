# Mastra Configuration and Instance Creation

This guide explains how to configure and instantiate a Mastra application, based on the setup in `hc-agents/src/mastra/index.ts`.

## Overview

A Mastra application is initialized by collecting various components (Agents, Workflows, Tools) and infrastructure configurations (Logging, Storage, Server) into a single configuration object, which is then passed to the `Mastra` class constructor.

## File Structure

The configuration typically lives in `src/mastra/index.ts`. This file serves as the entry point for your Mastra application.

## Configuration Breakdown

### 1. Imports

First, import the necessary classes from Mastra core and other packages.

```typescript
import { Mastra } from '@mastra/core/mastra';
import { Observability, DefaultExporter, CloudExporter, SensitiveDataFilter } from '@mastra/observability';
import { PinoLogger } from '@mastra/loggers';
// Import your agents, tools, etc.
import { orderRoutingAgent } from './agents/orderRoutingAgent';
// Import storage and auth implementations
import { PostgresStore } from '@mastra/pg';
import { PostgresJwtAuth } from './auth/postgres-jwt-auth';
```

### 2. Storage Setup

Initialize your storage layer. This is used by Mastra for persisting workflows, memory, and other state.

```typescript
const storage = new PostgresStore({
  id: 'hc-agent-store', // Unique identifier for this store
  connectionString: process.env.DATABASE_URL || '',
  schemaName: process.env.DATABASE_SCHEMA || ''
});
```

### 3. The Configuration Object

Construct the `config` object. This object tells Mastra about all the capabilities of your application.

```typescript
const config = {
  // --- Components ---
  agents: {
    orderRoutingAgent // Register your agents here
  },
  // workflows: { ... }, // Register workflows here if any

  // --- Infrastructure ---
  logger: new PinoLogger({ name: 'HC-Agents', level: 'debug' }),
  storage, // The storage instance created above

  // --- Server Configuration ---
  server: {
    port: process.env.MASTRA_PORT || 4111,
    host: process.env.MASTRA_HOST || 'localhost',
    build: {
      swaggerUI: true, // Enable Swagger UI for API documentation
    },
    // Authentication configuration (Optional but recommended for production)
    auth: new PostgresJwtAuth({
      connectionString: process.env.DATABASE_URL || '',
      jwtSecret: process.env.JWT_SECRET || '',
      store: storage,
      // Define public routes that don't satisfy authentication
      public: [
        '/api/studio/*',
        '/_next/*',
        '/assets/*',
        '/settings',
        '/swagger-ui',
        '/api/openapi.json'
      ]
    }),
    // Custom API Routes
    apiRoutes: [
      {
        path: '/',
        method: 'GET',
        handler: (c) => c.redirect('/settings'),
        requiresAuth: false
      }
    ]
  },

  // --- Observability (Tracing & Metrics) ---
  observability: new Observability({
    configs: {
      default: {
        serviceName: 'mastra',
        exporters: [
          new DefaultExporter(), // Store traces locally for Mastra Studio
          // new CloudExporter(), // Uncomment to send to Mastra Cloud
        ],
        spanOutputProcessors: [
          new SensitiveDataFilter(), // Filter sensitive data from logs/traces
        ],
      },
    },
  }),
};
```

### 4. Creating the Instance

Finally, create and export the `Mastra` instance.

```typescript
export const mastra = new Mastra({
  ...config,
});
```

## detailed Explanation of Sections

*   **`agents`**: A map of agent instances. These will be exposed via the API and accessible by the system.
*   **`storage`**: Persistence layer. `PostgresStore` is robust, but Mastra also supports `LibSQL` and others.
*   **`server`**: Configures the HTTP server that Mastra spins up.
    *   **`auth`**: Handles authentication for your API routes.
    *   **`apiRoutes`**: Allows you to add custom endpoints to the Mastra server.
*   **`observability`**: Configures OpenTelemetry tracing. Crucial for debugging Agent interactions and Workflow executions.

## Usage

Once exported, this `mastra` instance is used by the CLI to run your server (`mastra dev` or `mastra start`).
