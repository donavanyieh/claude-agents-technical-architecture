# Architecture Security Checklist

Focused on structural and design-level security concerns only.
Not for code-level review.

---

## Trust Boundaries
- [ ] Trust boundaries between components are explicitly defined
- [ ] External-facing components are isolated from internal services
- [ ] No internal services are directly exposed to the public internet
- [ ] Admin interfaces are on a separate network or access layer

---

## Authentication & Authorisation Design
- [ ] Authentication is handled at a consistent, centralised layer
- [ ] Authorisation model is defined (RBAC, ABAC, policy-based)
- [ ] Service-to-service authentication is specified

---

## API & Interface Design
- [ ] All external interfaces are explicitly documented
- [ ] Public APIs are behind a gateway or load balancer
- [ ] Rate limiting is assigned to a specific architectural layer
- [ ] CORS and access control policy ownership is clear
- [ ] Deprecated or internal APIs are not publicly reachable

---

## Infrastructure & Network
- [ ] Network segmentation is defined (public, private, data tiers)
- [ ] Databases and internal services are not in the public subnet
- [ ] Firewall rules and security groups are referenced
- [ ] Secrets management approach is specified (vault, env, KMS)