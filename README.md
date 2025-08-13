# Open LLM File Format (OLF)

A JSON-based, human-readable, and AI-optimized file format designed for AI-first applications.

## Documentation

- **[OLF.md](OLF.md)** - Complete format specification including schema definitions, block types, relationships, and examples
- **[OLF-Improvements.md](OLF-Improvements.md)** - Complexity reduction strategies and interoperability enhancements including OLF Lite specification

## Overview

OLF prioritizes semantic meaning, relationship preservation, and AI comprehension while maintaining full fidelity for visual rendering. Unlike traditional office formats that focus on visual presentation, OLF is built for the age of AI collaboration.

## Key Features

- **Semantic First** - Content structure reflects logical meaning, not visual layout
- **AI Optimized** - LLM-friendly JSON structure with embedded semantic annotations
- **Version Control Friendly** - Human-readable format for meaningful diffs
- **Extensible** - Plugin-friendly block system with forward/backward compatibility
- **Progressive Enhancement** - From simple OLF Lite to advanced AI-powered documents

## Quick Start

### OLF Lite Example
```json
{
    "format_version": "1.0.0-lite",
    "id": "doc_001",
    "title": "My Document",
    "blocks": [
        {
            "id": "1",
            "type": "text",
            "content": {
                "text": "# Hello OLF!\n\nThis is a simple document."
            }
        }
    ]
}
```

## License

See [LICENSE](LICENSE) file for details.