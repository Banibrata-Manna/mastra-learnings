# Mastra Prompt Flow

This diagram illustrates how a prompt is processed through the Mastra framework, from the initial `Agent.generate()` or `Agent.stream()` call down to the low-level LLM and Tool execution loops.

```mermaid
flowchart TD
    %% Nodes
    User([User])
    AgentGenerate["Agent.generate() / stream()"]
    AgentExecute["Agent.#execute()"]
    
    subgraph PrepareStreamWorkflow ["Prepare Stream Workflow"]
        direction TB
        PrepareTools["Prepare Tools Step"]
        PrepareMemory["Prepare Memory Step"]
        MapResults["Map Results Step"]
        StreamStep["Stream Step"]
        
        PrepareTools & PrepareMemory --> MapResults
        MapResults --> StreamStep
    end
    
    subgraph LLMStream ["MastraLLM.stream()"]
        LoopFn["loop()"]
        WorkflowLoopStream["workflowLoopStream"]
    end
    
    subgraph AgenticLoopWorkflow ["Agentic Loop Workflow"]
        direction TB
        DoWhileLoop{Do While Loop}
        
        subgraph AgenticExecutionWorkflow ["Agentic Execution Workflow"]
            direction TB
            LLMExec["LLM Execution Step"]
            AddResponse["Add Response to Memory"]
            ExtractTools["Extract Tool Calls"]
            ToolCallStep["Tool Call Step (foreach)"]
            LLMMapping["LLM Mapping Step"]
            
            LLMExec --> AddResponse
            AddResponse --> ExtractTools
            ExtractTools --> ToolCallStep
            ToolCallStep --> LLMMapping
        end
        
        DoWhileLoop -->|Start/Continue| AgenticExecutionWorkflow
        AgenticExecutionWorkflow -->|Check Stop Condition| DoWhileLoop
    end

    %% Connections
    User --> AgentGenerate
    AgentGenerate --> AgentExecute
    AgentExecute --> PrepareStreamWorkflow
    
    StreamStep -->|Calls| LLMStream
    LoopFn --> WorkflowLoopStream
    WorkflowLoopStream --> AgenticLoopWorkflow

    %% Styling
    classDef workflow fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef step fill:#fff9c4,stroke:#fbc02d,stroke-width:1px;
    classDef core fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    
    class PrepareStreamWorkflow,AgenticLoopWorkflow,AgenticExecutionWorkflow workflow;
    class PrepareTools,PrepareMemory,MapResults,StreamStep,LLMExec,AddResponse,ExtractTools,ToolCallStep,LLMMapping step;
    class AgentExecute,LoopFn,WorkflowLoopStream core;
```

## Key Components

1.  **Agent Entry Point**: The process starts with `Agent.generate()` or `Agent.stream()`, which delegates to `Agent.#execute()`.
2.  **Prepare Stream Workflow**: Configures the execution environment.
    *   **Prepare Tools**: Converts available tools (toolsets, memory, system) into `CoreTool` format.
    *   **Prepare Memory**: Retrieves thread history and context.
    *   **Stream Step**: The bridge to the LLM interaction.
3.  **LLM Stream**: The `MastraLLM.stream()` method (typically `MastraLLMVNext`) initiates the execution loop using `loop()`.
4.  **Agentic Loop Workflow**: Handles the iterative logic of "Think -> Act -> Observe". It runs the `AgenticExecutionWorkflow` in a loop until a stop condition (e.g., max steps, final answer) is met.
5.  **Agentic Execution Workflow**: A single iteration of the reasoning loop.
    *   **LLM Execution**: Calls the model API (OpenAI, Anthropic, etc.).
    *   **Add Response**: Saves the assistant's message to memory.
    *   **Tool Call Step**: Executes any requested tools (potentially in parallel).
    *   **LLM Mapping**: Formats tool results back into messages for the next LLM call.
