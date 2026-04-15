# Architecture Design & Review Agent System

A comprehensive Claude-powered agent and skill system for technical architecture design, security review, cost estimation, and diagram generation.

Inspired to do this to semi-automate/ produce a first draft of technical architecture for work. I have little experience creating architectures, and wanted to use GenAI to help brainstorm, make sense of ambiguity, and have a first architecture draft for me to edit

**From my experiments, this is useful to someone who can still make sense of technical jargon, ask the right questions, and challenge the outputs of the agents. Otherwise, it's cool to see it in action and create zero-shot architecture diagrams**

---

## 📋 Overview

This repository contains a specialized **Claude agent ecosystem** for architecture workflows. The system automates:

- 🏗️ **Architecture Design** — From requirements to comprehensive ARCHITECTURE.md
- 🔒 **Security Review** — Structural security analysis with severity-rated findings
- 💰 **Cost Estimation** — Infrastructure cost projections across usage scenarios
- 📊 **Diagram Generation** — Automated draw.io XML from architecture documentation

---

## 🤖 Agent System

### Architecture

The system uses 4 specialized agents

```
┌──────────────────────┐
│ architecture-reviewer │  Evaluates architecture quality
└──────────┬───────────┘
           ↓ manual delegation
┌──────────────────────┐
│  security-reviewer    │  Analyzes security weaknesses
└──────────┬───────────┘
           ↓ manual delegation
┌──────────────────────┐
│   cost-estimator      │  Estimates infrastructure costs
└──────────┬───────────┘
           ↓ manual delegation
┌──────────────────────┐
│ architecture-drawer   │  Generates draw.io diagrams
└──────────────────────┘
```

### Available Agents

| Agent | File | Purpose |
|-------|------|---------|
| **architecture-reviewer** | [agents/architecture-reviewer.md](.claude/agents/architecture-reviewer.md) | Reviews architecture for pattern fit, component design, scalability, maintainability, and completeness |
| **security-reviewer** | [agents/security-reviewer.md](.claude/agents/security-reviewer.md) | Applies security checklist to identify exploitable flaws, design risks, and best practice gaps |
| **cost-estimator** | [agents/cost-estimator.md](.claude/agents/cost-estimator.md) | Produces monthly cost estimates at low/mid/high usage with optimization recommendations |
| **architecture-drawer** | [agents/architecture-drawer.md](.claude/agents/architecture-drawer.md) | Converts ARCHITECTURE.md into draw.io XML with cloud provider-specific icons |

### Explicit Delegation Pattern

Each agent includes a mandatory **"Next Action"** section:

This ensures:
- ✅ Clear workflow progression
- ✅ No ambiguity about next steps
- ✅ Transparent handoffs between specialists

---

## 🎯 Skills

Skills provide specialized instructions for specific architecture tasks.

| Skill | File | When to Use |
|-------|------|-------------|
| **design-tech-architecture** | [skills/design-tech-architecture/SKILL.md](.claude/skills/design-tech-architecture/SKILL.md) | Creating a new system architecture from requirements |
| **review-security** | [skills/review-security/SKILL.md](.claude/skills/review-security/SKILL.md) | Security-focused review before finalizing architecture |

---

## 🚀 Usage

### Designing a New Architecture

Ask Claude:
```
"Help me design a system for [your requirement]"
```

Claude will:
1. Activate the `design-tech-architecture` skill
2. Ask clarifying questions about scale, constraints, team size
3. Choose an appropriate architecture pattern
4. Generate a complete ARCHITECTURE.md with:
   - Component diagrams (ASCII)
   - Tech stack with justifications
   - Data model
   - API design
   - Risks and open questions

**Example output**: See [ARCHITECTURE.md](./ARCHITECTURE.md) for a complete example.

---

### Reviewing an Existing Architecture

Ask Claude:
```
"Review my ARCHITECTURE.md"
```

Claude will automatically:

**1. architecture-reviewer** analyzes:
- Pattern fit (appropriate for scale and team size)
- Component design (single responsibility, clear boundaries)
- Scalability (bottlenecks, independent scaling)
- Maintainability (understandability, coupling)
- Completeness (auth, logging, caching, gaps)

**2. security-reviewer** examines:
- Authentication and authorization mechanisms
- Data protection and encryption
- Network security and trust boundaries
- Input validation and injection risks
- Secrets management
- SSRF and other attack vectors

**3. cost-estimator** calculates:
- Per-component monthly costs (low/mid/high usage)
- Total estimates across scales
- Cost risks (non-linear scaling, hidden costs)
- Optimization opportunities (reserved instances, alternatives)

**Output**: Cost summary table with recommendations

**4. architecture-drawer** generates:
- draw.io XML diagram from ARCHITECTURE.md
- Auto-detects cloud provider (AWS, Azure, GCP)
- Uses provider-specific icons (e.g., `mxgraph.aws4.*`)
- Falls back to generic shapes for multi-cloud

**Output**: `architecture.drawio` — editable in [draw.io](https://app.diagrams.net)

---

## 🔄 Workflow Guide

### Manual Delegation Pattern

The agent system uses a **manual delegation pattern** where you explicitly request each agent in the review chain. This gives you control to review outputs at each stage before proceeding.

#### Full Review Chain

**Step 1: Architecture Review**
```
You: "Review my ARCHITECTURE.md"
Claude: [Performs review using architecture-reviewer agent]
Claude: "✅ Architecture review complete. Next: delegate to security-reviewer"
```

**Step 2: Security Review**
```
You: "Delegate to security-reviewer"
Claude: [Performs security analysis]
Claude: "✅ Security review complete. Next: delegate to cost-estimator"
```

**Step 3: Cost Estimation**
```
You: "Delegate to cost-estimator"
Claude: [Calculates infrastructure costs]
Claude: "✅ Cost estimation complete. Next: delegate to architecture-drawer"
```

**Step 4: Diagram Generation**
```
You: "Delegate to architecture-drawer"
Claude: [Creates draw.io XML]
Claude: "✅ Workflow complete"
```

### Why Manual Delegation?

The system uses **explicit delegation** rather than automatic chaining because:
- ✅ You can review and approve each stage before proceeding
- ✅ You can jump to specific agents as needed
- ✅ You can iterate on findings before moving forward
- ✅ You maintain full control over the workflow
- ✅ You can stop at any stage (e.g., just get security review, skip cost estimation)

---

### Security-Only Review

Ask Claude:
```
"Do a security review of my architecture"
```

Claude will:
1. Activate the `review-security` skill
2. Work through [security-checklist.md](.claude/skills/review-security/security-checklist.md)
3. Produce [SECURITY-REVIEW.md](./SECURITY-REVIEW.md) with prioritized findings

---

## 📁 Project Structure

```
.
├── README.md                              # This file
├── ARCHITECTURE.md                        # Example architecture (demo output)
├── SECURITY-REVIEW.md                     # Example security review (demo output)
├── architecture.drawio                    # Example diagram (demo output)
│
└── .claude/                               # Agent & skill definitions
    │
    ├── agents/                            # Specialized review agents
    │   ├── architecture-reviewer.md       # Architecture quality reviewer
    │   ├── security-reviewer.md           # Security analysis specialist
    │   ├── cost-estimator.md              # Infrastructure cost estimator
    │   └── architecture-drawer.md         # Diagram generator
    │
    └── skills/                            # Task-specific skills
        │
        ├── design-tech-architecture/      # Architecture design skill
        │   ├── SKILL.md                   # Main skill definition
        │   └── checklist.md               # Pre-finalization checklist
        │
        └── review-security/               # Security review skill
            ├── SKILL.md                   # Main skill definition
            └── security-checklist.md      # Security checklist
```

---

## 💡 Example Scenarios

### Scenario 1: Greenfield Project

**Task**: "Design a microservices architecture for a B2B SaaS platform with 10k users that has project management module, and requires object detection on photo upload"

**Claude activates**: `design-tech-architecture` skill
- Asks about team size, budget, existing infrastructure
- Recommends event-driven microservices pattern
- Generates ARCHITECTURE.md with Kubernetes, Kafka, PostgreSQL
- Auto-triggers review chain → security → cost → diagram

### Scenario 2: Draw Architecture Diagram

**Pre-requisite**: Have an ARCHITECTURE.md file in project root directory

- cd to project root directory
- go into claude terminal i.e. type 'claude' in terminal
- type 'claude --agent architecture-drawer'
- Draws architecture diagram and saves it to architecture.drawio

---

## 🛠️ Best Practices

1. **Start with constraints** — Team size, budget, scale expectations
2. **Choose simple patterns first** — Avoid over-engineering for small teams
3. **Document assumptions** — Make implicit decisions explicit
4. **Run the full review chain** — Don't skip security or cost estimation

---

## TODO
1. **High priority: Delegation patterns**
    - I haven't quite cracked the delegation patterns. Stating it in the agents MD file doesn't help as much as I would like.
    - It's not a problem if you manually invoke, e.g. say "Do a security review next", or "modify the tech architecture for xxx" but i would love for the agents to be more autonomous. Will look into it soon
2. **Low priority: Improve DrawIO**
    - Beautify the technical architecture. I wonder where is the limitation of representing beautiful diagrams in XML. 
    - Icons need to be rendered properly
