# MastraMemory Class Diagram

This diagram illustrates the structure of the `MastraMemory` class and its relationships with other core components like `MastraCompositeStore`, `MastraVector`, and memory processors.

```mermaid
classDiagram
    class MastraBase {
        +String component
        +String name
        +Logger logger
    }

    class MastraMemory {
        <<Abstract>>
        +String id
        +MastraCompositeStore storage
        +MastraVector vector
        +MastraEmbeddingModel embedder
        +MemoryConfig threadConfig
        +__registerMastra(mastra)
        +setStorage(storage)
        +setVector(vector)
        +setEmbedder(embedder, options)
        +getSystemMessage(input)
        +listTools(config)
        +estimateTokens(text)
        +getThreadById(threadId)*
        +listThreads(args)*
        +saveThread(args)*
        +saveMessages(args)*
        +recall(args)*
        +createThread(args)
        +deleteThread(threadId)*
        +generateId(context)
        +getWorkingMemory(args)*
        +getWorkingMemoryTemplate(args)*
        +updateWorkingMemory(args)*
        +getInputProcessors(configuredProcessors, context)
        +getOutputProcessors(configuredProcessors, context)
        +deleteMessages(messageIds)*
        +cloneThread(args)*
    }

    class MastraCompositeStore {
        +String id
        +StorageDomains stores
        +Boolean disableInit
        +getStore(storeName)
        +init()
    }

    class MastraVector {
        <<Abstract>>
        +String id
        +String indexSeparator
        +query(params)*
        +upsert(params)*
        +createIndex(params)*
        +listIndexes()*
        +describeIndex(params)*
        +deleteIndex(params)*
        +updateVector(params)*
        +deleteVector(params)*
        +deleteVectors(params)*
    }

    class MemoryProcessor {
        <<Abstract>>
        +process(messages, opts)
    }

    class MessageHistory {
        +process(messages, opts)
    }

    class WorkingMemory {
        +process(messages, opts)
    }

    class SemanticRecall {
        +process(messages, opts)
    }
    
    class MastraEmbeddingModel {
        <<Interface>>
    }

    MastraMemory --|> MastraBase : Inherits
    MastraCompositeStore --|> MastraBase : Inherits
    MastraVector --|> MastraBase : Inherits
    MemoryProcessor --|> MastraBase : Inherits

    MessageHistory --|> MemoryProcessor : Inherits
    WorkingMemory --|> MemoryProcessor : Inherits
    SemanticRecall --|> MemoryProcessor : Inherits

    MastraMemory --> MastraCompositeStore : uses (storage)
    MastraMemory --> MastraVector : uses (vector)
    MastraMemory --> MastraEmbeddingModel : uses (embedder)

    MastraMemory ..> MessageHistory : creates/uses
    MastraMemory ..> WorkingMemory : creates/uses
    MastraMemory ..> SemanticRecall : creates/uses
```

## Relationships Breakdown

1.  **Inheritance**:
    *   `MastraMemory` extends `MastraBase`.
    *   `MastraCompositeStore` extends `MastraBase`.
    *   `MastraVector` extends `MastraBase`.
    *   `MemoryProcessor` extends `MastraBase`.
    *   `MessageHistory`, `WorkingMemory`, and `SemanticRecall` extend `MemoryProcessor`.

2.  **Associations**:
    *   **Storage**: `MastraMemory` relies on `MastraCompositeStore` (`storage` property) for persisting threads and messages.
    *   **Vector**: `MastraMemory` optionally uses `MastraVector` (`vector` property) for semantic search capabilities.
    *   **Embedder**: `MastraMemory` optionally uses `MastraEmbeddingModel` (`embedder` property) to generate embeddings for semantic recall.

3.  **Dependencies (Usage)**:
    *   `MastraMemory` instantiates and returns `MessageHistory`, `WorkingMemory`, and `SemanticRecall` processors in its `getInputProcessors` and `getOutputProcessors` methods, enabling these features in the agent workflow.
