# Open LLM File Format Specification (.olf)

## Format Overview

The `.olf` format is a JSON-based, human-readable, and AI-optimized file format designed specifically for the AI first applications. Unlike traditional office formats that prioritize visual presentation, the `.olf` format prioritizes semantic meaning, relationship preservation, and AI comprehension while maintaining full fidelity for visual rendering.

## Design Principles

### 1. Semantic First
- Content structure reflects logical meaning, not visual layout
- Metadata preserves intent and context
- Relationships are explicit rather than implied

### 2. AI Optimized
- LLM-friendly JSON structure for direct processing
- Embedded semantic annotations and tags
- Complete context preservation for AI reasoning

### 3. Version Control Friendly
- Human-readable JSON for meaningful diffs
- Incremental updates generate minimal changes
- Merge conflict resolution through semantic understanding

### 4. Extensible Architecture
- Plugin-friendly block type system
- Custom metadata and properties
- Forward/backward compatibility mechanisms

## File Structure

### Root Document Schema
```typescript
interface OlfDocument {
    // Format metadata
    format_version: string;              // "1.0.0"
    created: ISO8601DateTime;            // Creation timestamp
    modified: ISO8601DateTime;           // Last modification timestamp
    
    // Document identity and metadata
    id: DocumentId;                      // Unique document identifier
    title: string;                       // Human-readable title
    description?: string;                // Optional description
    tags: string[];                      // Searchable tags
    
    // Author and collaboration info
    authors: Author[];                   // Document creators/contributors
    collaboration: CollaborationInfo;   // Sharing and permissions
    
    // Content structure
    blocks: Block[];                     // All content blocks
    layout: LayoutInfo;                  // Visual arrangement
    relationships: Relationship[];       // Inter-block connections
    
    // AI integration
    ai_context: AIGlobalContext;         // Document-level AI state
    semantic_index: SemanticIndex;       // Content understanding
    
    // Export and compatibility
    export_metadata: ExportMetadata;     // Format conversion info
    compatibility: CompatibilityInfo;    // Version compatibility
    
    // Presentation modes
    presentation_modes: PresentationModes; // Edit vs presentation configurations
}
```

### Block Structure
```typescript
interface Block {
    // Identity
    id: BlockId;                         // Unique block identifier
    type: BlockType;                     // Block content type
    created: ISO8601DateTime;            // Creation timestamp
    modified: ISO8601DateTime;           // Last modification
    
    // Core content
    content: BlockContent;               // Type-specific content data
    metadata: BlockMetadata;             // Block-specific metadata
    
    // Semantic information
    semantic_tags: string[];             // AI-generated content tags
    summary: string;                     // AI-generated summary
    keywords: string[];                  // Extracted key terms
    
    // Relationships and dependencies
    dependencies: BlockId[];             // Blocks this one depends on
    references: Reference[];             // External references
    internal_links: InternalLink[];      // Links to other blocks
    
    // Visual presentation
    styling: BlockStyling;               // Visual appearance
    position: BlockPosition;             // Layout coordinates
    
    // AI integration
    ai_context: AIBlockContext;          // Block-level AI state
    reasoning_history: ReasoningEvent[]; // AI thought processes
    
    // Collaboration
    version: number;                     // Block version number
    last_author: AuthorId;               // Last modifier
    comments: Comment[];                 // Collaborative annotations
    
    // Validation and integrity
    checksum: string;                    // Content integrity hash
    schema_version: string;              // Block schema version
}
```

## Block Type Specifications

### Text Block
```typescript
interface TextBlockContent {
    // Rich text content
    content: SemanticMarkdown;           // Enhanced markdown with semantics
    
    // Text-specific properties
    word_count: number;                  // Automatic word count
    reading_time: number;                // Estimated reading time in minutes
    language: LanguageCode;              // Primary language
    
    // Semantic analysis
    entities: NamedEntity[];             // Extracted entities (people, places, etc.)
    topics: Topic[];                     // Main topics discussed
    sentiment: SentimentAnalysis;        // Emotional tone analysis
    
    // Cross-references
    variable_references: VariableRef[];  // References to data blocks
    citation_references: Citation[];     // Academic/source citations
    
    // AI assistance
    writing_suggestions: Suggestion[];   // AI improvement recommendations
    fact_checks: FactCheck[];           // Verification status of claims
}

// Enhanced markdown with semantic annotations
type SemanticMarkdown = string; // Markdown with special syntax:
// {{blockId.property}} - Dynamic references
// @[entity](type:person) - Entity annotations  
// #[concept](domain:finance) - Concept tags
// ^[source](url:...) - Source citations
```

### Data Block
```typescript
interface DataBlockContent {
    // Tabular data
    schema: DataSchema;                  // Column definitions
    data: DataRow[];                     // Actual data rows
    
    // Computed columns
    formulas: Formula[];                 // Spreadsheet-like formulas
    calculated_columns: CalculatedColumn[]; // Derived data
    
    // Data quality
    validation_rules: ValidationRule[];  // Data integrity rules
    quality_metrics: QualityMetric[];    // Data quality assessment
    
    // Semantic understanding
    column_semantics: ColumnSemantic[];  // Meaning of each column
    data_lineage: DataLineage[];        // Source and transformation history
    
    // Analysis results
    statistical_summary: StatSummary;    // Automatic statistics
    correlations: Correlation[];         // Detected relationships
    anomalies: Anomaly[];               // Unusual data points
    
    // Export configurations
    pivot_configs: PivotConfig[];        // Saved pivot table configurations
    filter_presets: FilterPreset[];      // Commonly used filters
}

interface DataSchema {
    columns: DataColumn[];
    primary_key?: string[];              // Unique identifier columns
    foreign_keys: ForeignKey[];          // Relationships to other data blocks
    indexes: Index[];                    // Performance optimization hints
}

interface DataColumn {
    name: string;                        // Column identifier
    display_name: string;               // Human-readable name
    type: DataType;                     // Data type
    constraints: Constraint[];          // Validation constraints
    description?: string;               // Column description
    semantic_type?: SemanticType;       // Meaning (e.g., 'currency', 'date')
    unit?: string;                      // Measurement unit
}

type DataType = 'string' | 'number' | 'boolean' | 'date' | 'datetime' | 'currency' | 'percentage' | 'url' | 'email';
type SemanticType = 'person_name' | 'company' | 'location' | 'currency' | 'percentage' | 'count' | 'measurement';
```

### Visual Block (Charts and Images)
```typescript
interface VisualBlockContent {
    // Visual type
    visual_type: VisualType;             // Chart, image, diagram, etc.
    
    // Data connection
    data_sources: DataSource[];          // Connected data blocks
    
    // Visual configuration
    chart_config?: ChartConfig;          // Chart-specific settings
    image_config?: ImageConfig;          // Image-specific settings
    diagram_config?: DiagramConfig;      // Diagram-specific settings
    
    // Generated assets
    rendered_assets: RenderedAsset[];    // Generated visual files
    
    // Accessibility
    alt_text: string;                    // Alternative text description
    data_table: DataTable;              // Tabular representation
    
    // AI analysis
    visual_analysis: VisualAnalysis;     // AI interpretation of visual
    insights: Insight[];                 // Automatically detected patterns
    
    // Interactivity
    interactive_config: InteractivityConfig; // User interaction settings
    drill_down_config: DrillDownConfig;  // Hierarchical exploration
}

type VisualType = 'bar_chart' | 'line_chart' | 'pie_chart' | 'scatter_plot' | 'heatmap' | 
                 'image' | 'diagram' | 'flowchart' | 'mind_map' | 'timeline' | 'gantt' | 
                 'network_graph' | 'treemap' | 'sankey' | 'histogram' | 'box_plot';

interface ChartConfig {
    title: string;
    subtitle?: string;
    x_axis: AxisConfig;
    y_axis: AxisConfig;
    series: SeriesConfig[];
    theme: ThemeConfig;
    annotations: Annotation[];
    responsive: ResponsiveConfig;
}
```

### AI Reasoning Block
```typescript
interface AIReasoningBlockContent {
    // Reasoning process
    reasoning_chain: ReasoningStep[];    // Step-by-step thought process
    
    // Input and output
    initial_prompt: string;              // Starting question/request
    final_conclusion: string;            // Final answer/recommendation
    confidence_score: number;            // 0-1 confidence rating
    
    // Evidence and sources
    evidence: EvidenceItem[];           // Supporting information
    assumptions: Assumption[];          // Stated assumptions
    limitations: Limitation[];          // Acknowledged constraints
    
    // Alternative perspectives
    alternative_views: AlternativeView[]; // Different interpretations
    counterarguments: Counterargument[]; // Opposing viewpoints
    
    // Validation
    fact_checks: FactCheckResult[];     // Verification of claims
    logical_validation: LogicalCheck[]; // Reasoning validity
    
    // Interaction history
    clarification_requests: Clarification[]; // Questions asked by AI
    user_feedback: UserFeedback[];      // Human input and corrections
    
    // Meta-reasoning
    reasoning_quality: ReasoningQuality; // Self-assessment
    improvement_suggestions: string[];   // How reasoning could be better
}

interface ReasoningStep {
    id: string;
    type: ReasoningType;
    input: any;                         // Step input data
    process: string;                    // Description of reasoning
    output: any;                        // Step output
    confidence: number;                 // Step confidence 0-1
    duration_ms: number;                // Processing time
    dependencies: string[];             // Previous steps used
}

type ReasoningType = 'analysis' | 'synthesis' | 'deduction' | 'induction' | 'abduction' | 
                    'comparison' | 'evaluation' | 'planning' | 'verification' | 'generation';
```

### Canvas Block (Freeform Design)
```typescript
interface CanvasBlockContent {
    // Canvas properties
    canvas_size: {
        width: number;
        height: number;
        unit: 'px' | 'cm' | 'in';
    };
    
    // Design elements
    elements: CanvasElement[];           // All design elements
    
    // Layout and grouping
    groups: ElementGroup[];              // Logical groupings
    layers: Layer[];                     // Visual layers
    
    // Design system
    style_guide: StyleGuide;            // Colors, fonts, spacing
    component_library: Component[];      // Reusable elements
    
    // Interaction and animation
    interactions: Interaction[];         // User interactions
    animations: Animation[];             // Motion graphics
    
    // Collaboration
    design_comments: DesignComment[];    // Visual feedback
    version_history: CanvasVersion[];    // Design iterations
    
    // Export settings
    export_settings: ExportSetting[];   // Output configurations
}

interface CanvasElement {
    id: string;
    type: ElementType;
    position: Position;
    size: Size;
    styling: ElementStyling;
    content: ElementContent;
    constraints: LayoutConstraint[];     // Responsive behavior
    interactions: ElementInteraction[];  // User interactions
}

type ElementType = 'text' | 'shape' | 'image' | 'line' | 'arrow' | 'chart_embed' | 
                  'data_embed' | 'component' | 'group' | 'frame';
```

## Relationship System

### Relationship Types
```typescript
interface Relationship {
    id: string;
    type: RelationshipType;
    source_block: BlockId;
    target_block: BlockId;
    
    // Relationship metadata
    strength: number;                    // 0-1 relationship strength
    confidence: number;                  // 0-1 AI confidence in relationship
    created: ISO8601DateTime;            // When relationship was identified
    
    // Semantic information
    description: string;                 // Human-readable description
    semantic_type: SemanticRelationType; // Meaning of relationship
    
    // Validation
    verified: boolean;                   // Human-verified relationship
    validation_notes: string;           // Verification comments
}

type RelationshipType = 
    | 'dependency'        // B requires A to be computed
    | 'reference'         // B references data from A
    | 'elaboration'       // B provides detail about A
    | 'contradiction'     // B contradicts A
    | 'support'          // B supports argument in A
    | 'causation'        // A causes B
    | 'correlation'      // A and B are correlated
    | 'similarity'       // A and B are similar
    | 'sequence'         // B follows A in time/logic
    | 'hierarchy'        // B is part of A
    | 'comparison'       // A and B are being compared
    | 'translation';     // B translates/transforms A

type SemanticRelationType = 
    | 'explains' | 'proves' | 'disproves' | 'questions' | 'assumes' 
    | 'implies' | 'requires' | 'enables' | 'prevents' | 'measures'
    | 'describes' | 'categorizes' | 'exemplifies' | 'generalizes';
```

## AI Context and Semantic Index

### Global AI Context
```typescript
interface AIGlobalContext {
    // Document understanding
    purpose: string;                     // Overall document goal
    audience: string;                    // Intended readers
    domain: string;                      // Subject area (finance, science, etc.)
    
    // Content analysis
    main_themes: Theme[];                // Primary topics
    argument_structure: ArgumentMap;     // Logical flow
    knowledge_gaps: KnowledgeGap[];     // Missing information
    
    // Quality metrics
    coherence_score: number;            // 0-1 logical consistency
    completeness_score: number;         // 0-1 information completeness
    clarity_score: number;              // 0-1 readability/understandability
    
    // Collaborative intelligence
    ai_contributions: AIContribution[];  // AI-generated content
    human_ai_interactions: Interaction[]; // Collaboration history
    
    // Learning and adaptation
    user_preferences: UserPreference[];  // Learned user patterns
    style_guidelines: StyleGuideline[];  // Consistency rules
}
```

### Semantic Index
```typescript
interface SemanticIndex {
    // Content embeddings
    block_embeddings: BlockEmbedding[];  // Vector representations
    
    // Concept mapping
    concepts: Concept[];                 // Identified concepts
    concept_relations: ConceptRelation[]; // How concepts relate
    
    // Entity recognition
    entities: Entity[];                  // Named entities
    entity_mentions: EntityMention[];    // Where entities appear
    
    // Topic modeling
    topics: Topic[];                     // Discovered topics
    topic_distributions: TopicDistribution[]; // Topic weights per block
    
    // Search optimization
    search_keywords: SearchKeyword[];    // Optimized search terms
    similarity_clusters: SimilarityCluster[]; // Related content groups
}
```

## Dual-Mode System: Edit vs Presentation

### Presentation Modes Configuration
```typescript
interface PresentationModes {
    // Default mode settings
    default_mode: 'edit' | 'presentation';           // Default when opening document
    
    // Edit mode configuration
    edit_mode: {
        enabled: boolean;                             // Can document be edited
        permissions: EditPermissions;                // Who can edit what
        collaboration: CollaborationSettings;        // Real-time collaboration settings
        ai_assistance: AIAssistanceLevel;            // Level of AI help available
        interface: EditInterfaceConfig;              // Edit-specific UI configuration
    };
    
    // Presentation mode configuration  
    presentation_mode: {
        enabled: boolean;                             // Can document be presented
        layout: PresentationLayout;                  // How blocks are arranged for viewing
        navigation: NavigationConfig;                // How users navigate through content
        interactivity: InteractivityLevel;           // What users can interact with
        branding: PresentationBranding;              // Visual branding for presentations
        export_options: PresentationExportOptions;   // Export formats for presentations
    };
    
    // Mode switching
    mode_switching: {
        allowed: boolean;                             // Can users switch between modes
        permissions: ModeSwitchPermissions;          // Who can switch modes
        transition_effects: TransitionConfig;        // Animations between modes
        state_preservation: StatePreservationConfig; // What state to maintain when switching
    };
    
    // Responsive behavior
    responsive_modes: {
        mobile: 'presentation' | 'edit' | 'auto';    // Mobile device default
        tablet: 'presentation' | 'edit' | 'auto';    // Tablet device default
        desktop: 'presentation' | 'edit' | 'auto';   // Desktop device default
        auto_detection: ResponsiveDetectionConfig;   // How to automatically choose
    };
}

interface EditPermissions {
    block_creation: boolean;                         // Can create new blocks
    block_deletion: boolean;                         // Can delete blocks
    content_editing: boolean;                        // Can edit block content
    structure_modification: boolean;                 // Can reorder/restructure
    ai_interaction: boolean;                         // Can interact with AI
    collaboration: boolean;                          // Can see other editors
}

interface PresentationLayout {
    type: 'linear' | 'slideshow' | 'dashboard' | 'story' | 'custom';
    
    // Linear layout (document-like)
    linear: {
        block_spacing: number;                       // Space between blocks
        max_width: number;                          // Maximum content width
        margins: MarginConfig;                       // Page margins
        typography: TypographyConfig;               // Fonts and sizing
    };
    
    // Slideshow layout (presentation-like)
    slideshow: {
        slides_per_view: number;                    // Blocks per slide
        slide_transitions: TransitionType[];        // Available transitions
        auto_advance: AutoAdvanceConfig;            // Automatic progression
        controls: SlideshowControlConfig;           // Navigation controls
    };
    
    // Dashboard layout (overview-like)  
    dashboard: {
        grid_system: GridConfig;                    // How blocks are arranged
        responsive_breakpoints: BreakpointConfig;   // Responsive behavior
        widget_sizing: WidgetSizingConfig;          // How blocks are sized
        refresh_intervals: RefreshConfig;           // Auto-refresh for data blocks
    };
    
    // Story layout (narrative flow)
    story: {
        narrative_flow: NarrativeFlowConfig;       // Story progression logic
        chapter_breaks: ChapterConfig;              // How to divide content
        multimedia_integration: MultimediaConfig;  // Images, videos, etc.
        pacing_control: PacingConfig;              // Control reading/viewing pace
    };
}
```

### Edit Mode Specifications
```typescript
interface EditModeConfiguration {
    // Editing capabilities
    editing_features: {
        real_time_collaboration: boolean;           // Multi-user editing
        version_control: boolean;                   // Track changes
        comment_threads: boolean;                   // Collaborative comments  
        suggestion_mode: boolean;                   // Suggest rather than edit
        ai_writing_assistance: boolean;             // AI help with content
        block_templates: boolean;                   // Quick block creation
        cross_references: boolean;                  // Link between blocks
        formula_editing: boolean;                   // Data calculations
    };
    
    // Interface elements
    interface_elements: {
        toolbar: 'full' | 'minimal' | 'contextual'; // Editing toolbar
        sidebar: 'always' | 'auto' | 'hidden';      // Sidebar visibility
        block_handles: boolean;                      // Drag/drop handles
        type_indicators: boolean;                    // Block type icons
        dependency_lines: boolean;                   // Visual dependency connections
        ai_suggestions: 'inline' | 'sidebar' | 'popup'; // How AI suggestions appear
    };
    
    // Performance settings
    performance: {
        auto_save_interval: number;                 // Seconds between auto-saves
        lazy_loading: boolean;                      // Load content on demand
        collaborative_throttling: number;          // Collaboration update frequency
        ai_response_streaming: boolean;             // Stream AI responses
    };
}
```

### Presentation Mode Specifications  
```typescript
interface PresentationModeConfiguration {
    // Viewing experience
    viewing_experience: {
        immersive_mode: boolean;                    // Full-screen presentation
        distraction_free: boolean;                  // Hide editing elements
        focus_highlighting: boolean;                // Highlight active content
        smooth_scrolling: boolean;                  // Smooth navigation
        zoom_controls: boolean;                     // Allow zooming
        search_overlay: boolean;                    // Search within presentation
    };
    
    // Navigation options
    navigation: {
        type: 'scroll' | 'click' | 'keyboard' | 'gesture'; // Navigation method
        progress_indicator: boolean;                // Show progress through document
        table_of_contents: boolean;                 // Show document outline
        breadcrumbs: boolean;                       // Show current location
        minimap: boolean;                          // Small overview of document
        jump_to_section: boolean;                   // Quick section navigation
    };
    
    // Interactivity levels
    interactivity: {
        level: 'none' | 'limited' | 'full';       // How much users can interact
        
        // Limited interactivity
        limited: {
            data_exploration: boolean;              // Can explore charts/data
            comment_viewing: boolean;               // Can see comments
            link_following: boolean;                // Can follow external links
            content_selection: boolean;             // Can select/copy text
        };
        
        // Full interactivity  
        full: {
            ai_chat: boolean;                      // Can chat with AI about content
            annotation: boolean;                    // Can add personal annotations
            bookmarking: boolean;                   // Can bookmark sections
            sharing: boolean;                       // Can share specific parts
            export: boolean;                        // Can export sections
        };
    };
    
    // Presentation-specific features
    presentation_features: {
        speaker_notes: boolean;                     // Show speaker notes
        presenter_mode: boolean;                    // Presenter view with notes
        audience_interaction: boolean;              // Audience can ask questions
        live_polling: boolean;                      // Interactive polls
        screen_sharing: boolean;                    // Share screen during presentation
        recording: boolean;                         // Record presentation
    };
}
```

### Mode-Specific Block Rendering
```typescript
interface ModeSpecificRendering {
    // Block appearance per mode
    block_rendering: {
        edit_mode: {
            show_metadata: boolean;                 // Show creation/modification info
            show_dependencies: boolean;             // Show block relationships
            show_ai_context: boolean;              // Show AI reasoning
            show_comments: boolean;                 // Show collaborative comments
            interactive_elements: 'all' | 'essential'; // Which UI elements to show
        };
        
        presentation_mode: {
            hide_technical_details: boolean;       // Hide implementation details
            emphasize_content: boolean;             // Focus on core content
            optimize_readability: boolean;          // Optimize for reading
            auto_play_media: boolean;              // Auto-play videos/animations
            simplified_interactions: boolean;       // Simplify interactive elements
        };
    };
    
    // Content adaptation
    content_adaptation: {
        edit_mode: {
            show_formulas: 'expanded' | 'collapsed'; // How to show formulas
            show_code: 'syntax_highlighted' | 'plain'; // Code display
            show_ai_reasoning: 'expanded' | 'summary'; // AI thought processes
            data_views: 'table' | 'chart' | 'both'; // How to show data
        };
        
        presentation_mode: {
            formula_rendering: 'results_only' | 'formatted'; // Show results vs formulas
            code_rendering: 'hidden' | 'output_only' | 'formatted'; // Code visibility
            ai_integration: 'invisible' | 'summary' | 'interactive'; // AI visibility
            data_emphasis: 'visualizations_first' | 'key_metrics'; // Data presentation priority
        };
    };
}
```

### Document State Management
```typescript
interface DocumentStateManagement {
    // State preservation between modes
    state_preservation: {
        cursor_position: boolean;                   // Remember where user was
        scroll_position: boolean;                   // Remember scroll position
        selected_blocks: boolean;                   // Remember selected content
        expanded_sections: boolean;                 // Remember collapsed/expanded state
        user_annotations: boolean;                  // Preserve personal notes
        view_preferences: boolean;                  // Remember zoom, layout preferences
    };
    
    // Mode-specific state
    mode_state: {
        edit_mode: {
            active_tool: string;                    // Currently selected tool
            clipboard_content: ClipboardContent;    // Copied content
            undo_stack: UndoStack;                 // Edit history
            ai_conversation: AIConversationState;   // AI chat history
            collaboration_awareness: CollabState;   // Other users' activity
        };
        
        presentation_mode: {
            presentation_progress: number;          // Progress through presentation
            bookmarks: Bookmark[];                  // User bookmarks
            personal_notes: Note[];                 // Private notes
            viewing_history: ViewingHistory;        // What user has seen
            interaction_log: InteractionLog;        // What user has interacted with
        };
    };
    
    // Synchronization between modes
    cross_mode_sync: {
        content_updates: 'immediate' | 'on_switch' | 'manual'; // When to sync content changes
        preference_sync: boolean;                   // Sync user preferences
        state_migration: StateMigrationConfig;     // How to migrate state between modes
    };
}
```

## Export and Compatibility

### Export Metadata
```typescript
interface ExportMetadata {
    // Traditional format mappings
    docx_mapping: DocxMapping;           // Word document conversion
    xlsx_mapping: XlsxMapping;          // Excel spreadsheet conversion
    pptx_mapping: PptxMapping;          // PowerPoint presentation conversion
    pdf_mapping: PdfMapping;            // PDF document conversion
    
    // Web formats
    html_config: HtmlExportConfig;       // Web page export
    markdown_config: MarkdownExportConfig; // Markdown export
    
    // Data formats
    csv_configs: CsvExportConfig[];      // Data table exports
    json_config: JsonExportConfig;       // Structured data export
    
    // Print and publication
    print_layouts: PrintLayout[];        // Physical printing layouts
    publication_formats: PublicationFormat[]; // Academic/professional formats
}
```

### Version Compatibility
```typescript
interface CompatibilityInfo {
    min_reader_version: string;          // Minimum version that can read
    feature_flags: FeatureFlag[];        // Optional features used
    migration_notes: MigrationNote[];    // Upgrade instructions
    deprecated_features: DeprecatedFeature[]; // Legacy compatibility
}
```

## File Format Examples

### Minimal Document
```json
{
    "format_version": "1.0.0",
    "created": "2024-12-07T10:00:00Z",
    "modified": "2024-12-07T10:30:00Z",
    "id": "doc_example_001",
    "title": "Simple Document",
    "description": "A basic olf document example",
    "tags": ["example", "basic"],
    "authors": [{
        "id": "user_001",
        "name": "John Doe",
        "email": "john@example.com"
    }],
    "blocks": [{
        "id": "block_001",
        "type": "text",
        "created": "2024-12-07T10:00:00Z",
        "modified": "2024-12-07T10:00:00Z",
        "content": {
            "content": "# Hello OLF!\n\nThis is a simple text block.",
            "word_count": 8,
            "reading_time": 0.1,
            "language": "en"
        },
        "semantic_tags": ["greeting", "introduction"],
        "summary": "A greeting and introduction to OLF"
    }],
    "layout": {
        "type": "linear",
        "blocks": ["block_001"]
    },
    "relationships": [],
    "ai_context": {
        "purpose": "demonstration",
        "audience": "developers",
        "domain": "documentation"
    }
}
```

### Complex Multi-Block Document
```json
{
    "format_version": "1.0.0",
    "id": "doc_sales_analysis",
    "title": "Q4 Sales Analysis",
    "blocks": [
        {
            "id": "intro_text",
            "type": "text",
            "content": {
                "content": "# Q4 Sales Analysis\n\nOur revenue grew {{sales_data.total_growth}}% this quarter.",
                "variable_references": [{
                    "variable": "{{sales_data.total_growth}}",
                    "target_block": "sales_data",
                    "target_property": "total_growth"
                }]
            }
        },
        {
            "id": "sales_data",
            "type": "data",
            "content": {
                "schema": {
                    "columns": [
                        {"name": "month", "type": "string", "display_name": "Month"},
                        {"name": "revenue", "type": "currency", "display_name": "Revenue"},
                        {"name": "growth", "type": "percentage", "display_name": "Growth %"}
                    ]
                },
                "data": [
                    {"month": "October", "revenue": 100000, "growth": 0.15},
                    {"month": "November", "revenue": 120000, "growth": 0.20},
                    {"month": "December", "revenue": 150000, "growth": 0.25}
                ],
                "calculated_columns": [{
                    "name": "total_growth",
                    "formula": "AVERAGE(growth)",
                    "value": 0.20
                }]
            }
        },
        {
            "id": "revenue_chart",
            "type": "chart",
            "content": {
                "visual_type": "line_chart",
                "data_sources": [{"block_id": "sales_data", "columns": ["month", "revenue"]}],
                "chart_config": {
                    "title": "Monthly Revenue Growth",
                    "x_axis": {"column": "month"},
                    "y_axis": {"column": "revenue", "format": "currency"}
                }
            }
        },
        {
            "id": "ai_analysis",
            "type": "ai_reasoning",
            "content": {
                "initial_prompt": "Analyze the sales trends and provide insights",
                "reasoning_chain": [
                    {
                        "id": "step_1",
                        "type": "analysis",
                        "process": "Examining monthly revenue progression",
                        "output": "Revenue shows consistent growth: Oct $100K → Nov $120K → Dec $150K"
                    },
                    {
                        "id": "step_2", 
                        "type": "synthesis",
                        "process": "Calculating growth acceleration",
                        "output": "Growth rate is accelerating: 20% (Oct-Nov) vs 25% (Nov-Dec)"
                    }
                ],
                "final_conclusion": "Sales show strong momentum with accelerating growth. December's 25% growth suggests successful holiday campaigns.",
                "confidence_score": 0.85
            }
        }
    ],
    "relationships": [
        {
            "type": "reference",
            "source_block": "intro_text",
            "target_block": "sales_data",
            "description": "Text references calculated growth rate from data"
        },
        {
            "type": "dependency",
            "source_block": "revenue_chart", 
            "target_block": "sales_data",
            "description": "Chart visualizes data from sales_data block"
        },
        {
            "type": "elaboration",
            "source_block": "ai_analysis",
            "target_block": "sales_data", 
            "description": "AI provides insights about the sales data trends"
        }
    ]
}
```

This comprehensive file format provides the foundation for OLF's revolutionary approach to document storage, ensuring that content remains meaningful, connected, and AI-accessible while maintaining full fidelity for traditional export formats.