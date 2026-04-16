# Claude Architecture Skills System

This is a **skills-based technical architecture workflow system** that helps you design, review, estimate costs, and visualize system architectures through specialized Claude skills.

---

## 🎯 System Overview

This repository contains 4 specialized skills for comprehensive architecture work:

1. **design-tech-architecture** - Design architecture from requirements
2. **review-security** - Security-focused architecture review
3. **estimate-infrastructure-costs** - Cloud infrastructure cost estimation
4. **draw-architecture-diagram** - Visual diagram generation

Each skill is automatically activated based on user requests and produces structured outputs (ARCHITECTURE.md, SECURITY-REVIEW.md, COST-ESTIMATE.md, architecture-diagram.drawio).

---

## 🧭 Skill Routing Logic

### When to Use Each Skill

| User Request Pattern | Activate Skill | Prerequisites | Output |
|---------------------|----------------|---------------|---------|
| "Design a system for..." | `design-tech-architecture` | None | ARCHITECTURE.md |
| "Help me architect..." | `design-tech-architecture` | None | ARCHITECTURE.md |
| "Create architecture for..." | `design-tech-architecture` | None | ARCHITECTURE.md |
| "Is this architecture secure?" | `review-security` | ARCHITECTURE.md | SECURITY-REVIEW.md |
| "Do a security review" | `review-security` | ARCHITECTURE.md | SECURITY-REVIEW.md |
| "Review security of..." | `review-security` | ARCHITECTURE.md | SECURITY-REVIEW.md |
| "How much will this cost?" | `estimate-infrastructure-costs` | ARCHITECTURE.md | COST-ESTIMATE.md |
| "Estimate costs for..." | `estimate-infrastructure-costs` | ARCHITECTURE.md | COST-ESTIMATE.md |
| "What's the monthly cost?" | `estimate-infrastructure-costs` | ARCHITECTURE.md | COST-ESTIMATE.md |
| "Draw the architecture" | `draw-architecture-diagram` | ARCHITECTURE.md | architecture-diagram.drawio |
| "Generate a diagram" | `draw-architecture-diagram` | ARCHITECTURE.md | architecture-diagram.drawio |
| "Visualize the system" | `draw-architecture-diagram` | ARCHITECTURE.md | architecture-diagram.drawio |

### Trigger Keywords by Skill

**design-tech-architecture**
- "design", "architect", "create architecture", "build system", "tech stack", "system design", "microservices", "monolith"

**review-security**
- "security", "secure", "vulnerabilities", "security review", "security analysis", "threats", "attack vectors"

**estimate-infrastructure-costs**
- "cost", "costs", "budget", "pricing", "monthly", "expenses", "estimate", "how much"

**draw-architecture-diagram**
- "draw", "diagram", "visualize", "draw.io", "visual", "generate diagram"

---

## 📚 Skills Reference

### 1. design-tech-architecture

**Location**: `.claude/skills/design-tech-architecture/SKILL.md`

**Purpose**: Design a complete system architecture from requirements through to a structured technical specification.

**When to use**:
- User wants to design a new system or feature
- Need help choosing appropriate tech stack
- Planning scalability and component design
- Creating ARCHITECTURE.md from scratch

**Output**: `ARCHITECTURE.md` with component diagrams, tech stack, data model, API design, risks

---

### 2. review-security

**Location**: `.claude/skills/review-security/SKILL.md`

**Purpose**: Review system architecture for structural security weaknesses and produce a prioritized findings report.

**When to use**:
- User asks for security review of architecture
- Before finalizing ARCHITECTURE.md
- Want to identify security gaps in design
- Need to understand security risks

**Prerequisites**: Requires existing `ARCHITECTURE.md`

**Output**: `SECURITY-REVIEW.md` with severity-rated findings, risks, and remediation steps

---

### 3. estimate-infrastructure-costs

**Location**: `.claude/skills/estimate-infrastructure-costs/SKILL.md`

**Purpose**: Estimate infrastructure and operational costs across different usage scenarios (low/mid/high).

**When to use**:
- User asks about infrastructure costs
- Budget planning needed
- Want to understand monthly expenses
- Need cost optimization recommendations

**Prerequisites**: Requires existing `ARCHITECTURE.md`

**Output**: `COST-ESTIMATE.md` with cost breakdown, total estimates, risks, optimization strategies

---

### 4. draw-architecture-diagram

**Location**: `.claude/skills/draw-architecture-diagram/SKILL.md`

**Purpose**: Convert architecture documentation into editable draw.io XML diagrams with cloud provider-specific icons.

**When to use**:
- User wants visual representation of architecture
- Need diagrams for documentation or presentations
- After finalizing ARCHITECTURE.md
- Want editable diagram for stakeholder communication

**Prerequisites**: Requires existing `ARCHITECTURE.md`

**Output**: `architecture-diagram.drawio`


## 🔄 Recommended Workflows

### Full Architecture Workflow (New System)

**Use when**: Designing a new system from scratch

**Sequence**:
1. **design-tech-architecture** → Creates ARCHITECTURE.md
2. **review-security** → Creates SECURITY-REVIEW.md
3. *(Iterate on architecture based on security findings)*
4. **estimate-infrastructure-costs** → Creates COST-ESTIMATE.md
5. *(Adjust architecture for budget constraints)*
6. **draw-architecture-diagram** → Creates architecture-diagram.drawio

**Outputs**: Complete architecture package (design + security + costs + diagram)

---

### Security Review Only

**Use when**: You already have ARCHITECTURE.md and want security analysis

**Sequence**:
1. **review-security** → Creates SECURITY-REVIEW.md
2. *(Modify ARCHITECTURE.md based on findings)*

**Prerequisites**: Existing ARCHITECTURE.md

---

### Cost Analysis Only

**Use when**: You want to understand financial implications of existing architecture

**Sequence**:
1. **estimate-infrastructure-costs** → Creates COST-ESTIMATE.md
2. *(Optimize architecture for cost efficiency)*

**Prerequisites**: Existing ARCHITECTURE.md

---

### Visualization Only

**Use when**: You need diagrams for existing architecture documentation

**Sequence**:
1. **draw-architecture-diagram** → Creates architecture-diagram.drawio

**Prerequisites**: Existing ARCHITECTURE.md

---

## 📄 File Outputs & Dependencies

### Source of Truth Files

| File | Created By | Used By | Purpose |
|------|-----------|---------|---------|
| `ARCHITECTURE.md` | design-tech-architecture | All other skills | Complete architecture specification |
| `SECURITY-REVIEW.md` | review-security | - | Security findings and recommendations |
| `COST-ESTIMATE.md` | estimate-infrastructure-costs | - | Cost projections and optimization |
| `architecture-diagram.drawio` | draw-architecture-diagram | - | Visual diagram (editable) |

### File Reading Dependencies

- **review-security** reads → ARCHITECTURE.md
- **estimate-infrastructure-costs** reads → ARCHITECTURE.md
- **draw-architecture-diagram** reads → ARCHITECTURE.md

**Important**: ARCHITECTURE.md is the single source of truth. All skills reference it.

---

## 🎓 Best Practices

### 1. Start with Requirements
Always use **design-tech-architecture** first if you don't have ARCHITECTURE.md. The skill includes requirements gathering to clarify:
- Scale expectations (users, requests/sec)
- Team size and capabilities
- Budget constraints
- Existing infrastructure

### 2. Run Security Review Early
Use **review-security** before finalizing architecture. It's easier to fix structural security issues in design than after implementation.

### 3. Consider Costs Throughout
Run **estimate-infrastructure-costs** to understand financial implications. Cost constraints might influence architectural decisions.

### 4. Iterate Based on Findings
- Security findings → Update ARCHITECTURE.md → Re-run security review
- Cost concerns → Adjust tech stack → Re-estimate costs
- Architecture changes → Regenerate diagram

### 5. Use Diagrams for Communication
Run **draw-architecture-diagram** to create visual representations for:
- Team alignment
- Stakeholder presentations
- Documentation
- Design reviews

---

## 🔧 Skill Activation Mechanics

### How Skills Are Activated

Claude automatically activates skills based on:
1. **Trigger phrases** in user requests (see Routing Logic above)
2. **Intent detection** from conversation context
3. **File prerequisites** (e.g., won't suggest review-security without ARCHITECTURE.md)

### Manual Skill Activation

You can explicitly request a skill by saying:
- "Use the design-tech-architecture skill"
- "Activate review-security skill"
- "Run estimate-infrastructure-costs"

---

## 📖 Supporting Documentation

Each skill has detailed documentation in `.claude/skills/`:

```
.claude/skills/
├── design-tech-architecture/
│   ├── SKILL.md           # Main skill definition
│   └── checklist.md       # Pre-finalization checklist
│
├── review-security/
│   ├── SKILL.md           # Main skill definition
│   └── security-checklist.md  # Security review checklist
│
├── estimate-infrastructure-costs/
│   └── SKILL.md           # Main skill definition
│
└── draw-architecture-diagram/
    ├── SKILL.md           # Main skill definition
    └── diagram-guidelines.md  # Detailed diagram specs
```

---

## 🚀 Quick Start Examples

### Example 1: New E-commerce Platform

**User**: "Design a microservices architecture for an e-commerce platform with 50K users"

**Claude activates**: `design-tech-architecture`

**Output**: ARCHITECTURE.md with microservices pattern, tech stack, data model, API design

**Next steps**: Run security review → estimate costs → generate diagram

---

### Example 2: Security Check

**User**: "Is my architecture secure?"

**Claude activates**: `review-security` (requires ARCHITECTURE.md)

**Output**: SECURITY-REVIEW.md with severity-rated findings

**Next steps**: Address critical/high findings → update ARCHITECTURE.md

---

### Example 3: Budget Planning

**User**: "How much will this infrastructure cost per month?"

**Claude activates**: `estimate-infrastructure-costs` (requires ARCHITECTURE.md)

**Output**: COST-ESTIMATE.md with low/mid/high cost scenarios

**Next steps**: Optimize high-cost components → re-estimate

---

### Example 4: Visual Diagram

**User**: "Create a diagram of the architecture"

**Claude activates**: `draw-architecture-diagram` (requires ARCHITECTURE.md)

**Output**: architecture-diagram.drawio (open in draw.io or VS Code)

**Next steps**: Export to PNG/SVG for presentations

---

## ⚠️ Important Notes

1. **ARCHITECTURE.md is required** for all skills except design-tech-architecture
2. **Skills are sequential** - design must come before review/cost/diagram
3. **Iterate as needed** - Skills can be re-run after updating ARCHITECTURE.md
4. **Single source of truth** - Keep one ARCHITECTURE.md, don't create duplicates
5. **Cloud provider detection** - draw-architecture-diagram auto-detects AWS/Azure/GCP from ARCHITECTURE.md

---

## 🛠️ Troubleshooting

**Problem**: Skill not activating automatically

**Solution**: Use explicit activation ("Use the [skill-name] skill") or check trigger phrases

---

**Problem**: "ARCHITECTURE.md not found" error

**Solution**: Run design-tech-architecture first to create ARCHITECTURE.md

---

**Problem**: Diagram shows blue squares instead of cloud icons

**Solution**: Check diagram-guidelines.md for proper XML structure and icon library usage

---

**Problem**: Cost estimates seem inaccurate

**Solution**: Update ARCHITECTURE.md with more specific component details (instance types, storage sizes)

---

## 📞 Support

For issues or improvements to the skills system, refer to the skill-specific SKILL.md files in `.claude/skills/` for detailed workflows and output formats.
