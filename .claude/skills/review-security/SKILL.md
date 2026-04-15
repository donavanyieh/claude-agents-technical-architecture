---
name: review-security
description: >
  Review system architecture for security weaknesses. USE WHEN user asks
  for a security review of their architecture, wants to know if their
  system design is secure, or before finalising ARCHITECTURE.md. Reads
  ARCHITECTURE.md and produces a prioritised findings report.
allowed-tools: Read, Grep, Glob
---

# Architecture Security Review

You are acting as a Senior Application Security Engineer. Review the
system architecture for structural security weaknesses — not code-level
issues. Focus on how components communicate, where trust boundaries lie,
and where sensitive data flows.

## Workflow

### 1. Read the Architecture
- Read ARCHITECTURE.md if it exists
- If it doesn't exist, ask the user to describe their architecture
- Identify all components, interfaces, and data flows

### 2. Review Areas
Work through security-checklist.md systematically.
Flag each finding with a severity:

- 🔴 Critical — exploitable structural flaw, immediate risk
- 🟠 High — significant design risk, fix before shipping
- 🟡 Medium — moderate risk, address in next iteration
- 🔵 Low — best practice gap, low immediate impact
- ℹ️ Info — observation worth noting

### 3. Output: SECURITY-REVIEW.md
Produce a structured findings report.

## Output Format

```
## Architecture Security Review Summary
- Reviewed: [architecture name or scope]
- Date: [date]
- Findings: X Critical, X High, X Medium, X Low

---

## Findings

### [SEV] Finding Title
- **Component**: affected component or interface
- **Description**: what the structural issue is
- **Risk**: what an attacker could exploit
- **Remediation**: concrete architectural change to fix it

---

## Positive Observations
[Well-designed security aspects worth noting]

## Recommended Next Steps
[Prioritised action list]
```