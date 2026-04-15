# Architecture Design Review Checklist

Run through this before finalising any architecture output. Every item
should be explicitly addressed — if it's out of scope, note why.

---

## Requirements
- [ ] Functional requirements are clearly listed
- [ ] Non-functional requirements are defined (latency, uptime, throughput)
- [ ] Scale expectations are confirmed (users, requests/sec, data volume)
- [ ] Constraints are documented (budget, existing stack, team size)

---

## Architecture
- [ ] Pattern choice is justified against alternatives
- [ ] All components are named with clear responsibilities
- [ ] No component has more than one primary responsibility
- [ ] Component boundaries and interfaces are defined
- [ ] Data flows are documented end-to-end
- [ ] There are no single points of failure (or they are acknowledged)

---

## Data
- [ ] Core entities and relationships are defined
- [ ] Storage technology is chosen and justified
- [ ] Read vs write access patterns are considered
- [ ] Data migration strategy is noted
- [ ] Sensitive data is identified and handling is specified

---

## API & Interfaces
- [ ] Internal interfaces between components are defined
- [ ] External APIs are specified (endpoints, contracts)
- [ ] Versioning strategy is noted
- [ ] Error responses are considered

---

## Security
- [ ] Authentication mechanism is chosen
- [ ] Authorisation model is defined
- [ ] Sensitive data is encrypted at rest and in transit
- [ ] Input validation strategy is noted
- [ ] Known attack surfaces are identified (injection, SSRF, etc.)

---

## Scalability & Performance
- [ ] Bottlenecks are identified
- [ ] Caching strategy is defined
- [ ] Horizontal vs vertical scaling approach is noted
- [ ] Expected latency targets are documented

---

## Operability
- [ ] Logging strategy is defined
- [ ] Metrics and alerting approach is noted
- [ ] Deployment strategy is outlined (CI/CD, environments)
- [ ] Rollback strategy is considered
- [ ] On-call / incident response is noted if relevant

---

## Risks & Open Questions
- [ ] Top 3 technical risks are listed
- [ ] Any assumptions made during design are documented
- [ ] Open questions requiring further investigation are listed
- [ ] Any out-of-scope items are explicitly noted

---

## Final Sanity Check
- [ ] Would a new engineer understand this document without a walkthrough?
- [ ] Does the design avoid over-engineering for the current stage?
- [ ] Is the MVP scope clearly distinguished from future work?