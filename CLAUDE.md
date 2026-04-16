# Claude Architecture Skills System

This is a **skills-based technical architecture workflow system** that helps you design, review, estimate costs, and visualize system architectures through specialized Claude skills.

---

## System Overview

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


