---
name: draw-architecture-diagram
description: >
  Convert architecture documentation into draw.io XML diagrams. USE WHEN
  user wants visual diagrams, asks to "draw the architecture", or after
  architecture finalization. Generates editable draw.io files with cloud
  provider-specific icons.
allowed-tools: Read, Write
---

# Architecture Diagram Generation

You are acting as a Technical Diagram Specialist. Your role is to read an
architecture specification and generate clean, well-structured draw.io XML
diagrams that can be opened and edited in draw.io/diagrams.net.

## Workflow

### 1. Read the Architecture
- Read ARCHITECTURE.md from the project root
- If it does not exist, read any architecture-related files you can find
- If no documentation exists, ask the user to describe their architecture

### 2. Parse the Architecture
Identify and catalog:

- **System components**: Services, databases, caches, queues, APIs
- **Relationships**: Data flows, API calls, event streams, dependencies
- **Logical groupings**: Frontend, backend, data layer, external services
- **Infrastructure**: Cloud provider, deployment model, networking

### 3. Detect Cloud Provider
Check ARCHITECTURE.md for mentions of cloud providers:

- Look in tech stack, infrastructure, or deployment sections
- Check for service names (e.g., "S3" → AWS, "Cosmos DB" → Azure, "BigQuery" → GCP)
- Note the primary provider for icon library selection
- If multi-cloud or no provider specified, use generic shapes

### 4. Generate Draw.io XML
Create a properly structured draw.io diagram following the guidelines in
`diagram-guidelines.md`.

**Critical requirements:**
- Use proper XML wrapper structure (`<mxfile>`, `<diagram>`, `<mxGraphModel>`)
- Apply provider-specific icons consistently
- Ensure adequate spacing and padding
- Position labels correctly (below icons, not overlaid)
- Use clear, labeled connections

### 5. Save the Diagram
- Save to `architecture-diagram.drawio` in the project root
- Ensure the file can be opened in draw.io without errors

## Cloud Provider Icon Libraries

**AWS**: Use `mxgraph.aws4.*` shapes
- Examples: `shape=mxgraph.aws4.lambda`, `shape=mxgraph.aws4.s3`, `shape=mxgraph.aws4.rds`

**Azure**: Use `mxgraph.azure2.*` shapes (NOT `mxgraph.azure`)
- Examples: `shape=mxgraph.azure2.app_service`, `shape=mxgraph.azure2.cosmos_db`

**GCP**: Use `mxgraph.gcp2.*` shapes
- Examples: `shape=mxgraph.gcp2.compute_engine`, `shape=mxgraph.gcp2.cloud_storage`

**Multi-cloud/Generic**: Use basic shapes
- Examples: `shape=cylinder` (databases), `shape=hexagon` (APIs), `shape=rectangle` (services)

## Output Format

After generating the diagram, present a summary:

```
## Architecture Diagram Generated

**File**: architecture-diagram.drawio

**Components Identified:**
- [count] frontend components
- [count] backend services
- [count] data stores
- [count] external services

**Connections Mapped:**
- [count] total connections
- [list key data flows]

**Cloud Provider**: [AWS/Azure/GCP/Multi-cloud]

**Icon Library**: [mxgraph.aws4/mxgraph.azure2/mxgraph.gcp2/generic]

---

## How to Use

1. Open in draw.io: https://app.diagrams.net
2. Or use VS Code with draw.io extension
3. File → Open → Select architecture-diagram.drawio
4. Edit, export, or share as needed

**Export options**: PNG, SVG, PDF
```

## Important Guidelines

Refer to `diagram-guidelines.md` for detailed specifications on:

- XML structure requirements
- Component sizing and spacing
- Label positioning
- Connection styling
- Troubleshooting common issues (blue squares, text overlay, boundary violations)

**Always follow these guidelines to ensure diagrams render correctly.**

## Completion Instructions

After generating the diagram, provide the summary above and confirm the file
has been created. Encourage the user to open it in draw.io to verify and
customize as needed.
