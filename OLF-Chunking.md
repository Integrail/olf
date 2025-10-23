# OLF Document Chunking Specification

## Overview

OLF Chunking enables efficient handling of large documents by splitting them into manageable JSON chunks. This approach supports streaming, partial loading, lazy evaluation, and improved performance for large-scale documents.

## Core Chunking Schema

### Chunk Metadata

```typescript
interface OlfChunkManifest {
    // Chunk metadata
    format_version: "1.0.0-chunked";
    document_id: string;
    total_chunks: number;
    chunk_strategy: ChunkStrategy;
    
    // Document metadata (from original)
    title: string;
    created: ISO8601DateTime;
    modified: ISO8601DateTime;
    
    // Chunk index
    chunks: ChunkDescriptor[];
    
    // Dependency graph
    dependencies: ChunkDependency[];
    
    // Assembly information
    assembly: AssemblyInfo;
    
    // Integrity
    checksum: string; // SHA-256 of complete assembled document
    chunk_checksums: Record<string, string>; // Per-chunk integrity
}

interface ChunkDescriptor {
    id: string; // chunk_001, chunk_002, etc.
    type: ChunkType;
    size_bytes: number;
    content_type: ContentType;
    
    // Content summary
    blocks_included: string[]; // Block IDs in this chunk
    priority: number; // 1-10, for loading order
    
    // Dependencies
    depends_on: string[]; // Other chunk IDs needed
    referenced_by: string[]; // Chunks that reference this one
    
    // Storage information
    storage_path?: string; // Relative path or URL
    compression?: CompressionType;
    
    // Validation
    checksum: string;
    schema_version: string;
}

type ChunkStrategy = 
    | "block-based"      // One or more blocks per chunk
    | "size-based"       // Fixed size chunks
    | "semantic-based"   // Semantically related content
    | "dependency-based" // Group by data dependencies
    | "hybrid";          // Combination of strategies

type ChunkType = 
    | "header"      // Document metadata and structure
    | "content"     // Document blocks
    | "relationships" // Inter-block relationships
    | "assets"      // External resources
    | "index"       // Semantic index and search data
    | "metadata";   // AI context and annotations

type ContentType = 
    | "text/blocks"     // Text blocks
    | "data/tables"     // Data blocks
    | "visual/charts"   // Visual blocks
    | "ai/reasoning"    // AI reasoning blocks
    | "mixed";          // Multiple block types
```

### Individual Chunk Schema

```typescript
interface OlfChunk {
    // Chunk identification
    chunk_id: string;
    document_id: string;
    chunk_index: number; // 0-based position
    
    // Chunk metadata
    type: ChunkType;
    content_type: ContentType;
    created: ISO8601DateTime;
    
    // Content payload
    payload: ChunkPayload;
    
    // References and dependencies
    external_references: ExternalReference[];
    internal_dependencies: string[]; // Other chunk IDs needed
    
    // Validation
    checksum: string;
    schema_version: string;
}

interface ChunkPayload {
    // Core content based on chunk type
    blocks?: Block[]; // For content chunks
    relationships?: Relationship[]; // For relationship chunks
    metadata?: any; // For metadata chunks
    assets?: AssetReference[]; // For asset chunks
    
    // Chunk-specific data
    chunk_metadata?: Record<string, any>;
    
    // Partial document structure (if needed)
    partial_layout?: PartialLayout;
    partial_index?: PartialSemanticIndex;
}

interface ExternalReference {
    target_chunk: string;
    target_type: "block" | "relationship" | "asset";
    target_id: string;
    reference_type: "dependency" | "reference" | "link";
}
```

## Chunking Strategies

### 1. Block-Based Chunking

Groups blocks together based on logical boundaries and size constraints.

```typescript
interface BlockBasedChunking {
    strategy: "block-based";
    
    // Configuration
    max_blocks_per_chunk: number; // Default: 50
    max_chunk_size_mb: number; // Default: 5MB
    preserve_block_integrity: boolean; // Default: true
    
    // Grouping rules
    grouping_rules: {
        keep_related_blocks_together: boolean; // Group dependent blocks
        respect_layout_boundaries: boolean; // Don't split visual sections
        prioritize_sequential_blocks: boolean; // Keep document order
    };
    
    // Block type preferences
    block_type_affinity: Record<string, string[]>; // Which types work well together
}

// Example block-based chunking result
const blockChunking: ChunkDescriptor[] = [
    {
        id: "chunk_001",
        type: "header",
        blocks_included: [], // Header chunk contains no blocks
        content_type: "mixed",
        priority: 10 // Highest priority - load first
    },
    {
        id: "chunk_002", 
        type: "content",
        blocks_included: ["intro_text", "overview_chart"],
        content_type: "mixed",
        priority: 9 // High priority - introductory content
    },
    {
        id: "chunk_003",
        type: "content", 
        blocks_included: ["sales_data", "revenue_chart"],
        content_type: "data/tables",
        priority: 8 // Related data blocks grouped together
    }
];
```

### 2. Size-Based Chunking

Splits documents based on size limits, useful for network transfer optimization.

```typescript
interface SizeBasedChunking {
    strategy: "size-based";
    
    // Size constraints
    target_chunk_size_mb: number; // Default: 2MB
    max_chunk_size_mb: number; // Default: 5MB  
    min_chunk_size_mb: number; // Default: 0.1MB
    
    // Content handling
    allow_block_splitting: boolean; // Can split large blocks
    compression_enabled: boolean; // Apply compression
    
    // Overflow handling
    overflow_strategy: "new_chunk" | "compress" | "split_content";
}
```

### 3. Semantic-Based Chunking

Groups semantically related content together for better AI processing.

```typescript
interface SemanticBasedChunking {
    strategy: "semantic-based";
    
    // Semantic grouping
    similarity_threshold: number; // 0-1, minimum similarity to group
    topic_coherence: boolean; // Keep same topics together
    
    // AI-driven organization
    use_ai_clustering: boolean; // Use AI to determine groups
    preserve_narrative_flow: boolean; // Maintain story progression
    
    // Semantic rules
    grouping_criteria: {
        by_topic: boolean; // Group by main topics
        by_entity: boolean; // Group by named entities
        by_relationship_strength: boolean; // Group by connection strength
        by_reasoning_chain: boolean; // Keep logical flows together
    };
}
```

### 4. Dependency-Based Chunking

Organizes chunks based on data dependencies and references.

```typescript
interface DependencyBasedChunking {
    strategy: "dependency-based";
    
    // Dependency analysis
    include_dependency_graph: boolean; // Map all dependencies
    minimize_cross_chunk_deps: boolean; // Reduce inter-chunk references
    
    // Loading optimization
    create_dependency_layers: boolean; // Chunks that can load in parallel
    prioritize_root_dependencies: boolean; // Load foundational data first
    
    // Reference handling
    resolve_circular_deps: "break" | "merge" | "reference"; // How to handle cycles
}
```

## Chunk Assembly Process

### Assembly Manager

```typescript
interface ChunkAssemblyManager {
    // Loading and validation
    loadManifest(manifestUrl: string): Promise<OlfChunkManifest>;
    validateManifest(manifest: OlfChunkManifest): ValidationResult;
    
    // Chunk loading
    loadChunk(chunkId: string, options?: LoadOptions): Promise<OlfChunk>;
    loadChunks(chunkIds: string[], parallel?: boolean): Promise<OlfChunk[]>;
    loadAllChunks(manifest: OlfChunkManifest): Promise<OlfChunk[]>;
    
    // Assembly
    assembleDocument(chunks: OlfChunk[], manifest: OlfChunkManifest): Promise<OlfDocument>;
    partialAssembly(chunks: OlfChunk[], requestedBlocks?: string[]): Promise<Partial<OlfDocument>>;
    
    // Streaming assembly
    streamAssembly(manifest: OlfChunkManifest, options?: StreamOptions): AsyncIterable<AssemblyProgress>;
    
    // Validation
    validateAssembly(document: OlfDocument, manifest: OlfChunkManifest): ValidationResult;
    verifyIntegrity(document: OlfDocument, expectedChecksum: string): boolean;
}

interface AssemblyProgress {
    stage: "loading" | "validating" | "assembling" | "complete";
    chunks_loaded: number;
    chunks_total: number;
    bytes_loaded: number;
    bytes_total: number;
    current_chunk?: string;
    partial_document?: Partial<OlfDocument>;
    errors?: AssemblyError[];
}

interface LoadOptions {
    validate_checksum?: boolean; // Default: true
    decompress?: boolean; // Default: true
    include_metadata?: boolean; // Default: true
    timeout_ms?: number; // Default: 30000
}
```

### Assembly Algorithm

```typescript
class OlfAssembler {
    async assembleDocument(chunks: OlfChunk[], manifest: OlfChunkManifest): Promise<OlfDocument> {
        // 1. Validate all chunks
        for (const chunk of chunks) {
            this.validateChunk(chunk, manifest);
        }
        
        // 2. Sort chunks by dependency order
        const sortedChunks = this.topologicalSort(chunks, manifest.dependencies);
        
        // 3. Initialize document structure
        const document: Partial<OlfDocument> = {
            format_version: manifest.format_version.replace('-chunked', ''),
            id: manifest.document_id,
            title: manifest.title,
            created: manifest.created,
            modified: manifest.modified,
            blocks: [],
            relationships: [],
            layout: { type: "linear", blocks: [] }
        };
        
        // 4. Merge chunks in order
        for (const chunk of sortedChunks) {
            await this.mergeChunk(document, chunk);
        }
        
        // 5. Resolve cross-references
        this.resolveCrossReferences(document);
        
        // 6. Rebuild semantic index
        document.semantic_index = this.rebuildSemanticIndex(document);
        
        // 7. Final validation
        this.validateFinalDocument(document as OlfDocument);
        
        return document as OlfDocument;
    }
    
    private mergeChunk(document: Partial<OlfDocument>, chunk: OlfChunk): void {
        switch (chunk.type) {
            case "content":
                if (chunk.payload.blocks) {
                    document.blocks!.push(...chunk.payload.blocks);
                }
                break;
                
            case "relationships": 
                if (chunk.payload.relationships) {
                    document.relationships!.push(...chunk.payload.relationships);
                }
                break;
                
            case "metadata":
                // Merge AI context, export metadata, etc.
                Object.assign(document, chunk.payload.metadata);
                break;
                
            case "index":
                if (chunk.payload.partial_index) {
                    this.mergeSemanticIndex(document, chunk.payload.partial_index);
                }
                break;
        }
    }
}
```

## Streaming and Progressive Loading

### Progressive Loading Strategy

```typescript
interface ProgressiveLoadingConfig {
    // Loading priorities
    priority_levels: {
        critical: string[]; // Chunk types to load first (header, key content)
        high: string[]; // Important content chunks
        medium: string[]; // Supporting content
        low: string[]; // Optional content (detailed metadata, etc.)
        lazy: string[]; // Load on demand only
    };
    
    // Performance settings
    concurrent_chunk_limit: number; // Max parallel downloads
    chunk_cache_size: number; // How many chunks to keep in memory
    prefetch_strategy: "none" | "next" | "smart"; // Predictive loading
    
    // User experience
    show_loading_progress: boolean;
    allow_partial_rendering: boolean; // Render before fully loaded
    fallback_content: Record<string, any>; // Show while loading
}

interface ProgressiveLoader {
    // Initialize progressive loading
    startLoading(manifestUrl: string, config: ProgressiveLoadingConfig): Promise<LoadingSession>;
    
    // Loading control
    pauseLoading(session: LoadingSession): void;
    resumeLoading(session: LoadingSession): void;
    cancelLoading(session: LoadingSession): void;
    
    // Priority management
    requestChunk(session: LoadingSession, chunkId: string, priority: "immediate" | "high" | "normal"): Promise<OlfChunk>;
    requestBlocks(session: LoadingSession, blockIds: string[]): Promise<Block[]>;
    
    // Progress monitoring
    onProgress(callback: (progress: LoadingProgress) => void): void;
    onChunkLoaded(callback: (chunk: OlfChunk) => void): void;
    onError(callback: (error: LoadingError) => void): void;
}

interface LoadingSession {
    id: string;
    manifest: OlfChunkManifest;
    loaded_chunks: Set<string>;
    loading_chunks: Set<string>;
    failed_chunks: Set<string>;
    partial_document: Partial<OlfDocument>;
    config: ProgressiveLoadingConfig;
}
```

### Streaming API

```typescript
// Server-side streaming endpoint
interface ChunkStreamingAPI {
    // Start streaming session
    "POST /api/olf/documents/:id/stream": {
        body: {
            chunk_strategy?: ChunkStrategy;
            priority_filter?: string[]; // Only stream certain chunk types
            compression?: boolean;
        };
        response: {
            session_id: string;
            manifest: OlfChunkManifest;
            stream_url: string; // WebSocket or SSE endpoint
        };
    };
    
    // WebSocket streaming
    "WS /api/olf/stream/:session_id": {
        // Incoming messages
        messages: {
            "request_chunk": { chunk_id: string; priority?: number };
            "request_blocks": { block_ids: string[] };
            "cancel_chunk": { chunk_id: string };
            "set_priority": { chunk_id: string; priority: number };
        };
        
        // Outgoing messages  
        events: {
            "manifest": { manifest: OlfChunkManifest };
            "chunk_data": { chunk: OlfChunk };
            "chunk_progress": { chunk_id: string; bytes_loaded: number; bytes_total: number };
            "assembly_update": { partial_document: Partial<OlfDocument> };
            "error": { error: string; chunk_id?: string };
            "complete": { final_document: OlfDocument };
        };
    };
}

// Client-side streaming consumer
class OlfStreamingClient {
    async startStream(documentId: string, options: StreamingOptions): Promise<DocumentStream> {
        const response = await fetch(`/api/olf/documents/${documentId}/stream`, {
            method: 'POST',
            body: JSON.stringify(options)
        });
        
        const { session_id, manifest, stream_url } = await response.json();
        
        const ws = new WebSocket(stream_url);
        
        return new DocumentStream(ws, manifest, options);
    }
}

class DocumentStream {
    constructor(
        private ws: WebSocket, 
        private manifest: OlfChunkManifest,
        private options: StreamingOptions
    ) {
        this.setupEventHandlers();
    }
    
    // Request specific chunks
    requestChunk(chunkId: string, priority: number = 5): void {
        this.ws.send(JSON.stringify({
            type: "request_chunk",
            chunk_id: chunkId,
            priority
        }));
    }
    
    // Request specific blocks (will load containing chunks)
    requestBlocks(blockIds: string[]): void {
        this.ws.send(JSON.stringify({
            type: "request_blocks", 
            block_ids: blockIds
        }));
    }
    
    // Event handlers
    onChunkReceived(callback: (chunk: OlfChunk) => void): void { ... }
    onAssemblyUpdate(callback: (doc: Partial<OlfDocument>) => void): void { ... }
    onComplete(callback: (doc: OlfDocument) => void): void { ... }
}
```

## Implementation Examples

### Basic Chunking Example

```typescript
// Create a document chunker
const chunker = new OlfChunker({
    strategy: "block-based",
    max_blocks_per_chunk: 10,
    max_chunk_size_mb: 2
});

// Split a large OLF document
const manifest = await chunker.chunkDocument(largeOlfDoc, {
    output_directory: "./chunks/",
    compress_chunks: true,
    generate_manifest: true
});

// Save manifest
await fs.writeFile("./chunks/manifest.json", JSON.stringify(manifest, null, 2));

console.log(`Document split into ${manifest.total_chunks} chunks`);
```

### Progressive Loading Example

```typescript
// Load document progressively
const loader = new ProgressiveLoader();

const session = await loader.startLoading("./chunks/manifest.json", {
    priority_levels: {
        critical: ["header"],
        high: ["content"], 
        medium: ["relationships"],
        low: ["metadata", "index"]
    },
    concurrent_chunk_limit: 3,
    allow_partial_rendering: true
});

// Show loading progress
loader.onProgress((progress) => {
    console.log(`Loading: ${progress.chunks_loaded}/${progress.chunks_total} chunks`);
    
    // Render partial document as it loads
    if (progress.partial_document) {
        renderPartialDocument(progress.partial_document);
    }
});

// Final document ready
loader.onComplete((document) => {
    console.log("Document fully loaded!");
    renderFullDocument(document);
});
```

### Streaming Example

```typescript
// Start streaming a large document
const client = new OlfStreamingClient();

const stream = await client.startStream("large_doc_123", {
    chunk_strategy: "semantic-based",
    compression: true,
    priority_filter: ["header", "content"] // Only stream essential chunks
});

// Request specific content first
stream.requestBlocks(["intro_text", "summary_chart"]);

// Handle incoming chunks
stream.onChunkReceived((chunk) => {
    console.log(`Received chunk: ${chunk.chunk_id} (${chunk.payload.blocks?.length} blocks)`);
});

// Handle partial document updates
stream.onAssemblyUpdate((partialDoc) => {
    // Update UI with new content as it arrives
    updateDocumentView(partialDoc);
});

// Handle completion
stream.onComplete((fullDoc) => {
    console.log("Streaming complete!");
    finalizeDocumentView(fullDoc);
});
```

This chunking specification enables efficient handling of large OLF documents through flexible splitting strategies, progressive loading, and streaming capabilities while maintaining document integrity and relationships.