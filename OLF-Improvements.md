# OLF Format Improvements

## 1. Complexity Reduction

### OLF Lite Specification

The OLF Lite format is a minimal subset of the full OLF specification, designed for quick adoption and simple use cases. It maintains compatibility with the full format while drastically reducing complexity.

#### OLF Lite Schema (Minimal Required Fields)

```typescript
interface OlfLiteDocument {
    // Required metadata (minimal)
    format_version: "1.0.0-lite";
    id: string;                          // UUID format recommended
    title: string;
    
    // Simple content structure
    blocks: OlfLiteBlock[];
    
    // Optional metadata
    created?: ISO8601DateTime;
    modified?: ISO8601DateTime;
    description?: string;
    tags?: string[];
}

interface OlfLiteBlock {
    // Required fields only
    id: string;                          // Unique within document
    type: "text" | "data" | "visual";    // Simplified types
    content: any;                        // Type-specific content
    
    // Optional fields
    title?: string;                      // Block title/heading
    metadata?: Record<string, any>;      // Extensible metadata
}

// Minimal content types
interface OlfLiteTextContent {
    text: string;                        // Plain text or markdown
    format?: "plain" | "markdown";       // Default: "markdown"
}

interface OlfLiteDataContent {
    headers: string[];                   // Column headers
    rows: any[][];                       // Data rows
    types?: string[];                    // Optional column types
}

interface OlfLiteVisualContent {
    type: "image" | "chart";
    url?: string;                        // External resource
    data?: any;                          // Embedded data
    alt?: string;                        // Accessibility text
}
```

#### OLF Lite Example

```json
{
    "format_version": "1.0.0-lite",
    "id": "doc_001",
    "title": "Sales Report",
    "blocks": [
        {
            "id": "1",
            "type": "text",
            "content": {
                "text": "# Q4 Sales Summary\n\nTotal revenue increased by 20%."
            }
        },
        {
            "id": "2",
            "type": "data",
            "content": {
                "headers": ["Month", "Revenue"],
                "rows": [
                    ["October", 100000],
                    ["November", 120000],
                    ["December", 150000]
                ]
            }
        }
    ]
}
```

### Progressive Enhancement Levels

#### Level 1: Basic (OLF Lite)
- Minimal required fields
- Simple block types (text, data, visual)
- No relationships or AI features
- Direct mapping to common formats
- **Use cases**: Simple documents, notes, basic reports

#### Level 2: Standard
- All Basic features plus:
- Full block type support
- Basic relationships (reference, dependency)
- Simple metadata and semantic tags
- Version tracking
- Basic collaboration features
- **Use cases**: Business documents, collaborative editing, structured reports

#### Level 3: Advanced
- All Standard features plus:
- AI reasoning blocks
- Complex relationships
- Semantic indexing
- Real-time collaboration
- Advanced analytics
- Full presentation modes
- **Use cases**: AI-powered documents, knowledge bases, interactive presentations

### Migration Path

```typescript
// Utility to upgrade documents between levels
interface OlfMigrator {
    // Detect current document level
    detectLevel(doc: any): "lite" | "standard" | "advanced";
    
    // Upgrade functions
    liteToStandard(doc: OlfLiteDocument): OlfStandardDocument;
    standardToAdvanced(doc: OlfStandardDocument): OlfDocument;
    
    // Downgrade functions (with potential data loss)
    advancedToStandard(doc: OlfDocument): OlfStandardDocument;
    standardToLite(doc: OlfStandardDocument): OlfLiteDocument;
    
    // Validation
    validate(doc: any, level: string): ValidationResult;
}

// Example upgrade from Lite to Standard
function liteToStandard(lite: OlfLiteDocument): OlfStandardDocument {
    return {
        ...lite,
        format_version: "1.0.0",
        authors: [],
        relationships: [],
        layout: {
            type: "linear",
            blocks: lite.blocks.map(b => b.id)
        },
        blocks: lite.blocks.map(block => ({
            ...block,
            created: new Date().toISOString(),
            modified: new Date().toISOString(),
            semantic_tags: [],
            summary: "",
            keywords: [],
            dependencies: [],
            version: 1
        }))
    };
}
```

## 5. Interoperability Improvements

### Standard OLF Manipulation API

#### Core API Specification

```typescript
// OLF Document API
interface OlfDocumentAPI {
    // Document operations
    create(options?: CreateOptions): Promise<OlfDocument>;
    load(source: string | File | URL): Promise<OlfDocument>;
    save(doc: OlfDocument, destination: string): Promise<void>;
    validate(doc: any): ValidationResult;
    
    // Block operations
    blocks: {
        add(doc: OlfDocument, block: Block): OlfDocument;
        remove(doc: OlfDocument, blockId: string): OlfDocument;
        update(doc: OlfDocument, blockId: string, updates: Partial<Block>): OlfDocument;
        find(doc: OlfDocument, query: BlockQuery): Block[];
        move(doc: OlfDocument, blockId: string, position: number): OlfDocument;
    };
    
    // Relationship operations
    relationships: {
        add(doc: OlfDocument, rel: Relationship): OlfDocument;
        remove(doc: OlfDocument, relId: string): OlfDocument;
        findByBlock(doc: OlfDocument, blockId: string): Relationship[];
        validate(doc: OlfDocument): RelationshipValidation[];
    };
    
    // Export/Import
    export: {
        toJSON(doc: OlfDocument, pretty?: boolean): string;
        toMarkdown(doc: OlfDocument): string;
        toHTML(doc: OlfDocument, template?: string): string;
        toDocx(doc: OlfDocument): Promise<Blob>;
        toPDF(doc: OlfDocument): Promise<Blob>;
    };
    
    import: {
        fromMarkdown(content: string): OlfDocument;
        fromHTML(content: string): OlfDocument;
        fromDocx(file: File): Promise<OlfDocument>;
        fromJSON(json: string): OlfDocument;
    };
}

// RESTful API Specification
interface OlfRestAPI {
    // Document endpoints
    "GET /api/olf/documents": {
        response: { documents: OlfDocumentSummary[] };
    };
    
    "POST /api/olf/documents": {
        body: CreateDocumentRequest;
        response: { document: OlfDocument };
    };
    
    "GET /api/olf/documents/:id": {
        response: { document: OlfDocument };
    };
    
    "PUT /api/olf/documents/:id": {
        body: OlfDocument;
        response: { document: OlfDocument };
    };
    
    "DELETE /api/olf/documents/:id": {
        response: { success: boolean };
    };
    
    // Block operations
    "POST /api/olf/documents/:id/blocks": {
        body: Block;
        response: { block: Block };
    };
    
    "PATCH /api/olf/documents/:id/blocks/:blockId": {
        body: Partial<Block>;
        response: { block: Block };
    };
    
    // Export endpoints
    "GET /api/olf/documents/:id/export/:format": {
        params: { format: "json" | "markdown" | "html" | "docx" | "pdf" };
        response: Blob | string;
    };
    
    // Import endpoints
    "POST /api/olf/import": {
        body: { content: string; format: string };
        response: { document: OlfDocument };
    };
}

// WebSocket API for real-time collaboration
interface OlfWebSocketAPI {
    // Connection
    connect(documentId: string, token: string): WebSocket;
    
    // Events
    events: {
        // Outgoing
        "block.update": { blockId: string; changes: any };
        "block.add": { block: Block };
        "block.remove": { blockId: string };
        "cursor.move": { blockId: string; position: number };
        
        // Incoming
        "document.updated": { document: OlfDocument };
        "collaborator.joined": { user: User };
        "collaborator.left": { userId: string };
        "collaborator.cursor": { userId: string; blockId: string; position: number };
    };
}
```

### Transformation Pipelines

#### Pipeline Architecture

```typescript
// Transformation pipeline system
interface TransformationPipeline {
    // Pipeline definition
    name: string;
    description: string;
    stages: TransformationStage[];
    
    // Execution
    execute(input: any, options?: PipelineOptions): Promise<any>;
    
    // Validation
    validate(input: any): ValidationResult;
}

interface TransformationStage {
    name: string;
    transformer: Transformer;
    errorHandling: "skip" | "abort" | "fallback";
    fallback?: any;
}

interface Transformer {
    // Core transformation function
    transform(input: any, context: TransformContext): Promise<any>;
    
    // Metadata
    inputSchema?: Schema;
    outputSchema?: Schema;
    
    // Capabilities
    capabilities: {
        streaming: boolean;
        parallel: boolean;
        reversible: boolean;
    };
}

// Common transformation pipelines
const COMMON_PIPELINES = {
    // Markdown to OLF
    markdownToOlf: {
        name: "markdown-to-olf",
        stages: [
            { name: "parse", transformer: new MarkdownParser() },
            { name: "extract-structure", transformer: new StructureExtractor() },
            { name: "create-blocks", transformer: new BlockCreator() },
            { name: "detect-relationships", transformer: new RelationshipDetector() },
            { name: "build-document", transformer: new OlfBuilder() }
        ]
    },
    
    // OLF to Word
    olfToDocx: {
        name: "olf-to-docx",
        stages: [
            { name: "validate", transformer: new OlfValidator() },
            { name: "flatten-structure", transformer: new StructureFlattener() },
            { name: "convert-blocks", transformer: new BlockToDocxConverter() },
            { name: "apply-styles", transformer: new DocxStyler() },
            { name: "generate-file", transformer: new DocxGenerator() }
        ]
    },
    
    // Excel to OLF
    excelToOlf: {
        name: "excel-to-olf",
        stages: [
            { name: "parse", transformer: new ExcelParser() },
            { name: "detect-tables", transformer: new TableDetector() },
            { name: "extract-charts", transformer: new ChartExtractor() },
            { name: "infer-relationships", transformer: new DataRelationshipInferrer() },
            { name: "create-document", transformer: new OlfDocumentCreator() }
        ]
    }
};

// Bidirectional transformation support
interface BidirectionalTransformer extends Transformer {
    // Reverse transformation
    reverse(output: any, context: TransformContext): Promise<any>;
    
    // Round-trip fidelity score (0-1)
    fidelityScore(): number;
}

// Example: Markdown ↔ OLF transformer
class MarkdownOlfTransformer implements BidirectionalTransformer {
    async transform(markdown: string, context: TransformContext): Promise<OlfDocument> {
        // Parse markdown
        const ast = parseMarkdown(markdown);
        
        // Create blocks from AST
        const blocks: Block[] = [];
        let blockId = 1;
        
        for (const node of ast.children) {
            switch (node.type) {
                case 'heading':
                case 'paragraph':
                    blocks.push({
                        id: String(blockId++),
                        type: 'text',
                        content: {
                            content: node.value,
                            format: 'markdown'
                        }
                    });
                    break;
                    
                case 'table':
                    blocks.push({
                        id: String(blockId++),
                        type: 'data',
                        content: this.tableToDataBlock(node)
                    });
                    break;
                    
                case 'code':
                    if (node.lang === 'chart') {
                        blocks.push({
                            id: String(blockId++),
                            type: 'visual',
                            content: this.parseChartDefinition(node.value)
                        });
                    }
                    break;
            }
        }
        
        return {
            format_version: "1.0.0",
            id: generateId(),
            title: this.extractTitle(ast),
            blocks,
            relationships: this.inferRelationships(blocks)
        };
    }
    
    async reverse(doc: OlfDocument, context: TransformContext): Promise<string> {
        const lines: string[] = [];
        
        for (const block of doc.blocks) {
            switch (block.type) {
                case 'text':
                    lines.push(block.content.content);
                    break;
                    
                case 'data':
                    lines.push(this.dataBlockToMarkdownTable(block));
                    break;
                    
                case 'visual':
                    if (block.content.type === 'chart') {
                        lines.push(`\`\`\`chart\n${JSON.stringify(block.content, null, 2)}\n\`\`\``);
                    }
                    break;
            }
            
            lines.push(''); // Empty line between blocks
        }
        
        return lines.join('\n');
    }
    
    fidelityScore(): number {
        return 0.85; // High fidelity for text, some loss for complex features
    }
}
```

### Plugin Architecture

```typescript
// OLF Plugin System
interface OlfPlugin {
    // Plugin metadata
    name: string;
    version: string;
    description: string;
    author: string;
    
    // Lifecycle hooks
    onInstall?(api: OlfAPI): Promise<void>;
    onUninstall?(): Promise<void>;
    onDocumentOpen?(doc: OlfDocument): void;
    onDocumentClose?(doc: OlfDocument): void;
    
    // Extension points
    blockTypes?: CustomBlockType[];
    transformers?: Transformer[];
    validators?: Validator[];
    exportFormats?: ExportFormat[];
    
    // UI extensions (if applicable)
    uiComponents?: {
        blockRenderers?: Record<string, ComponentType>;
        toolbarItems?: ToolbarItem[];
        sidebarPanels?: SidebarPanel[];
    };
}

// Example plugin: LaTeX support
const latexPlugin: OlfPlugin = {
    name: "olf-latex",
    version: "1.0.0",
    description: "LaTeX equation support for OLF documents",
    author: "Example Corp",
    
    blockTypes: [{
        type: "latex",
        schema: {
            equation: { type: "string", required: true },
            displayMode: { type: "boolean", default: false }
        },
        validator: (content) => {
            try {
                validateLatex(content.equation);
                return { valid: true };
            } catch (e) {
                return { valid: false, error: e.message };
            }
        }
    }],
    
    transformers: [{
        transform: async (input, context) => {
            if (input.type === "latex") {
                return {
                    ...input,
                    rendered: await renderLatex(input.content.equation)
                };
            }
            return input;
        }
    }],
    
    exportFormats: [{
        format: "tex",
        name: "LaTeX Document",
        exporter: async (doc) => {
            return convertOlfToLatex(doc);
        }
    }]
};
```

### Standard Integration Patterns

```typescript
// Integration with popular frameworks
// React Hook
function useOlfDocument(documentId: string) {
    const [doc, setDoc] = useState<OlfDocument | null>(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<Error | null>(null);
    
    useEffect(() => {
        OlfAPI.load(documentId)
            .then(setDoc)
            .catch(setError)
            .finally(() => setLoading(false));
    }, [documentId]);
    
    const updateBlock = useCallback((blockId: string, updates: any) => {
        if (doc) {
            const updated = OlfAPI.blocks.update(doc, blockId, updates);
            setDoc(updated);
        }
    }, [doc]);
    
    return { doc, loading, error, updateBlock };
}

// Vue Composable
function useOlfDocument(documentId: Ref<string>) {
    const doc = ref<OlfDocument | null>(null);
    const loading = ref(true);
    const error = ref<Error | null>(null);
    
    watchEffect(async () => {
        try {
            loading.value = true;
            doc.value = await OlfAPI.load(documentId.value);
        } catch (e) {
            error.value = e;
        } finally {
            loading.value = false;
        }
    });
    
    return { doc, loading, error };
}

// Express.js middleware
function olfMiddleware(options: OlfMiddlewareOptions = {}) {
    return async (req: Request, res: Response, next: NextFunction) => {
        // Parse OLF documents in request body
        if (req.is('application/olf+json')) {
            try {
                req.body = await OlfAPI.import.fromJSON(req.body);
            } catch (e) {
                return res.status(400).json({ error: 'Invalid OLF document' });
            }
        }
        
        // Add OLF response helpers
        res.olf = (doc: OlfDocument) => {
            res.type('application/olf+json');
            res.json(doc);
        };
        
        res.olfExport = async (doc: OlfDocument, format: string) => {
            const exported = await OlfAPI.export[format](doc);
            res.type(getContentType(format));
            res.send(exported);
        };
        
        next();
    };
}
```

## Summary

These improvements address the two key areas requested:

### 1. Complexity Reduction
- **OLF Lite**: A minimal 20-line schema that covers 80% of use cases
- **Progressive Enhancement**: Clear upgrade path from simple to advanced
- **Migration Tools**: Automated conversion between levels

### 2. Interoperability
- **Standard APIs**: RESTful, WebSocket, and programmatic APIs
- **Transformation Pipelines**: Bidirectional conversion with popular formats
- **Plugin System**: Extensible architecture for custom features
- **Framework Integration**: Ready-to-use patterns for React, Vue, Express, etc.

These changes make OLF more approachable for developers while maintaining its powerful capabilities for advanced use cases.