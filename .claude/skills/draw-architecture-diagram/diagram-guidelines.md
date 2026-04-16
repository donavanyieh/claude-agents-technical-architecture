# Draw.io Diagram Guidelines

This document provides detailed specifications for generating high-quality,
properly rendered draw.io XML diagrams from architecture documentation.

---

## Required XML Structure

### Critical Wrapper Elements

**Every draw.io file MUST include proper wrapper structure** to avoid rendering
issues (e.g., blue square icons):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mxfile host="app.diagrams.net" modified="2024-01-01T00:00:00.000Z" agent="Claude" version="22.0.0">
  <diagram name="Architecture" id="architecture-diagram">
    <mxGraphModel dx="1800" dy="1000" grid="0" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        
        <!-- Your components go here -->
        
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

**Without these wrapper elements, icons will render as blue squares.**

---

## Component Sizing

### Icon-based Components

Each component should have adequate size to display icons and text properly:

- **Icon size**: 60x60px (standard for cloud provider icons)
- **Total cell space**: 100x120px minimum
  - 60px for icon
  - 10px gap
  - 50px for multi-line labels
- **Positioning**: Maintain minimum 50-60px from any container edge

### Service Boxes

- **Standard size**: 140x120px (allows icon + multi-line label)
- Increase width for longer names (up to 200px)

### Database Cylinders

- **Standard size**: 120x140px (taller for cylinder shape)

### Containers/Swimlanes

- **Minimum size**: 400x300px (to hold multiple components with proper padding)
- **Larger containers**: Add 200px for each additional row/column of components
- **Padding**: 60-80px on all sides to prevent boundary violations

---

## Label Positioning

### Preventing Text Overlay on Icons

To ensure text appears **below** the icon, not overlaid on top, use these
attributes:

```
verticalLabelPosition=bottom   # Text below the shape
labelPosition=center           # Text centered horizontally
verticalAlign=top              # Icon at top, text below
align=center                   # Horizontal alignment
```

**These attributes are mandatory for all icon-based components.**

### Example Component XML

```xml
<mxCell id="service1" value="API Service" 
  style="shape=mxgraph.aws4.lambda;verticalLabelPosition=bottom;labelPosition=center;verticalAlign=top;align=center;fillColor=#FF9900;strokeColor=#FF9900;" 
  vertex="1" parent="1">
  <mxGeometry x="200" y="150" width="60" height="60" as="geometry"/>
</mxCell>
```

---

## Component Styling

### Color Scheme

Apply consistent colors by component type:

| Component Type | Fill Color | Stroke Color |
|----------------|------------|--------------|
| Frontend | #dae8fc | #6c8ebf |
| Backend services | #d5e8d4 | #82b366 |
| Data stores | #fff2cc | #d6b656 |
| External services | #f5f5f5 | #666666 |
| Message queues | #ffe6cc | #d79b00 |

### Fonts

- **Component labels**: `fontSize=12`
- **Connection labels**: `fontSize=10`
- **Container titles**: `fontSize=14;fontStyle=1` (bold)

---

## Layout & Spacing

### Horizontal Spacing

- **Between components in same row**: 200-250px
- **Between major component groups**: 300px+
  - Example: frontend → backend → data layer

### Vertical Spacing

- **Between rows of components**: 150-200px
- **Between swimlane sections**: 250px+

### Container Padding

**Critical to prevent icons from exceeding boundaries:**

- **Swimlanes/containers**: 60-80px padding on all sides
- **Minimum from edge to first component**: 50-60px
- **Bottom padding for labeled icons**: 100px (to prevent label overflow)

**Rule of thumb**: When icons are 60x60px, position them at least 60px from edges.

### Diagram Canvas

- **Start components at**: (100, 100) to avoid edge clipping
- **Leave margin**: 100px on all diagram edges

---

## Connections

### Arrow Styles

- **Solid arrows** (`endArrow=classic`): Synchronous API calls, direct dependencies
- **Dashed arrows** (`dashed=1`): Asynchronous events, messages, queues

### Connection Labels

- Label arrows with protocol/format: REST, GraphQL, Event, WebSocket, etc.
- Position: `fontSize=10`, `labelBackgroundColor=white` for readability
- Place labels on edges, not at endpoints

### Example Connection XML

```xml
<mxCell id="conn1" value="REST API" 
  style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1;endArrow=classic;strokeWidth=2;fontSize=10;" 
  edge="1" parent="1" source="frontend1" target="backend1">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
```

---

## Cloud Provider Icon Libraries

### AWS (mxgraph.aws4.*)

Common shapes:
- `shape=mxgraph.aws4.lambda` - Lambda functions
- `shape=mxgraph.aws4.s3` - S3 storage
- `shape=mxgraph.aws4.rds` - RDS database
- `shape=mxgraph.aws4.api_gateway` - API Gateway
- `shape=mxgraph.aws4.elastic_load_balancing` - Load balancer
- `shape=mxgraph.aws4.cloudfront` - CloudFront CDN
- `shape=mxgraph.aws4.sqs` - SQS queue
- `shape=mxgraph.aws4.dynamodb` - DynamoDB

### Azure (mxgraph.azure2.*)

**CRITICAL**: Use `azure2` library, NOT `azure`, to avoid blue squares.

Common shapes:
- `shape=mxgraph.azure2.app_service` - App Service
- `shape=mxgraph.azure2.function_app` - Azure Functions
- `shape=mxgraph.azure2.cosmos_db` - Cosmos DB
- `shape=mxgraph.azure2.sql_database` - SQL Database
- `shape=mxgraph.azure2.storage_blob` - Blob Storage
- `shape=mxgraph.azure2.cdn` - CDN
- `shape=mxgraph.azure2.service_bus` - Service Bus
- `shape=mxgraph.azure2.application_gateway` - Application Gateway

### GCP (mxgraph.gcp2.*)

Common shapes:
- `shape=mxgraph.gcp2.compute_engine` - Compute Engine
- `shape=mxgraph.gcp2.cloud_functions` - Cloud Functions
- `shape=mxgraph.gcp2.cloud_storage` - Cloud Storage
- `shape=mxgraph.gcp2.cloud_sql` - Cloud SQL
- `shape=mxgraph.gcp2.bigquery` - BigQuery
- `shape=mxgraph.gcp2.cloud_cdn` - Cloud CDN
- `shape=mxgraph.gcp2.pub_sub` - Pub/Sub

### Generic Shapes (Multi-cloud)

When no specific provider is identified:
- `shape=cylinder` - Databases
- `shape=hexagon` - APIs
- `shape=rectangle` - Services
- `shape=ellipse` - Caches
- `shape=parallelogram` - Queues
- `shape=cloud` - External services

---

## Troubleshooting Common Issues

### Issue 1: Blue Square Icons

**Symptom**: Icons appear as blue squares instead of proper cloud service icons.

**Causes**:
1. Missing `<mxfile>` and `<diagram>` wrapper elements (most common)
2. Incorrect shape library reference (e.g., `mxgraph.azure` instead of `mxgraph.azure2`)
3. Typo in shape name

**Fix**: 
- Ensure proper XML structure with all wrapper elements
- Verify shape library names (double-check `azure2`, not `azure`)
- Validate shape names against documentation

### Issue 2: Icons/Labels Exceeding Container Boundaries

**Symptom**: Icon corners or text labels extend beyond swimlane/container edges.

**Causes**:
1. Insufficient container padding (< 50px)
2. Icon positioned too close to edge
3. Insufficient vertical space for bottom labels
4. Container too small for number of components

**Fix**:
- Increase container size by 100-200px
- Reposition icons to maintain 60px+ margin from edges
- Add 100px bottom padding for labeled components
- Recalculate container size: (num_components × component_size) + padding

### Issue 3: Text Overlaying Icons

**Symptom**: Text appears on top of icon instead of below it.

**Cause**: Missing or incorrect label positioning attributes.

**Fix**: Ensure all icon components have:
```
verticalLabelPosition=bottom
labelPosition=center
verticalAlign=top
align=center
```

### Issue 4: Connections Not Visible

**Symptom**: Arrows between components don't render or are misplaced.

**Causes**:
1. Missing `source` and `target` attributes
2. Incorrect component IDs referenced
3. Connection positioned behind components (z-index issue)

**Fix**:
- Verify source/target IDs match component IDs exactly
- Ensure connections are defined after all components
- Use `edge="1"` attribute in mxCell

### Issue 5: Diagram Too Cramped or Too Spread Out

**Symptom**: Components are too close together or too far apart.

**Cause**: Incorrect spacing calculations.

**Fix**:
- Review spacing guidelines (200-250px horizontal, 150-200px vertical)
- Increase canvas size if diagram extends beyond visible area
- Use consistent spacing multipliers (e.g., 250px × num_columns)

---

## XML Formatting Best Practices

### 1. Use Proper mxCell Structure

Every component needs:
- Unique `id` attribute
- `value` attribute (component label)
- `style` attribute (shape, colors, positioning)
- `vertex="1"` for nodes, `edge="1"` for connections
- `parent="1"` (or parent container ID)
- `<mxGeometry>` element with x, y, width, height

### 2. Include html=1 Attribute

Add `html=1` to style for proper text rendering:
```
style="shape=rectangle;html=1;fontSize=12;"
```

### 3. Set Explicit Dimensions

Always specify width and height in mxGeometry. Don't rely on auto-sizing:
```xml
<mxGeometry x="200" y="150" width="100" height="100" as="geometry"/>
```

### 4. Use Consistent ID Naming

- Frontend: `frontend1`, `frontend2`, etc.
- Backend: `backend1`, `backend2`, etc.
- Databases: `db1`, `db2`, etc.
- Connectors: `conn1`, `conn2`, etc.

### 5. Group Related Components

Use containers/swimlanes to group logically related components:
```xml
<mxCell id="backend-layer" value="Backend Layer" 
  style="swimlane;fontSize=14;fontStyle=1;fillColor=#d5e8d4;strokeColor=#82b366;" 
  vertex="1" parent="1">
  <mxGeometry x="100" y="300" width="800" height="400" as="geometry"/>
</mxCell>

<!-- Components inside container use backend-layer as parent -->
<mxCell id="api1" value="API Service" parent="backend-layer">
  <mxGeometry x="50" y="100" width="100" height="100" as="geometry"/>
</mxCell>
```

---

## Quick Reference Checklist

Before finalizing a diagram, verify:

- [ ] Proper XML wrapper structure (`<mxfile>`, `<diagram>`, `<mxGraphModel>`)
- [ ] All components have unique IDs
- [ ] Cloud provider icons use correct library (aws4, azure2, gcp2)
- [ ] Icon components have proper label positioning attributes
- [ ] Adequate spacing between components (200-250px horizontal, 150-200px vertical)
- [ ] Container padding is sufficient (60-80px on all sides)
- [ ] All connections have valid source/target IDs
- [ ] Consistent color scheme applied by component type
- [ ] Font sizes appropriate (12px labels, 10px connection labels)
- [ ] Diagram starts at (100, 100) with edge margins

---

## Additional Resources

- **draw.io documentation**: https://www.diagrams.net/doc/
- **Shape libraries**: https://www.diagrams.net/blog/shape-libraries
- **AWS shapes**: Search "aws4" in draw.io shape picker
- **Azure shapes**: Search "azure2" in draw.io shape picker
- **GCP shapes**: Search "gcp2" in draw.io shape picker
