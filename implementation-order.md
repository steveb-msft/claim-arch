# CLM Implementation Order

> Phased implementation plan respecting feature priorities (P0→P3) and technical dependencies.

## Overview

The CLM platform is implemented in 5 phases. Each phase builds on the previous, with infrastructure foundations laid first, followed by core capabilities, then progressively more advanced features. Within each phase, features are ordered by technical dependency — features that other features depend on are implemented first.

---

## Phase 0 — Infrastructure Foundation

**Goal:** Establish the production-grade Azure environment, networking, security baseline, and deployment pipeline before any application code is deployed.

**Repos active:** `claim-clm-infra`

| Order | FeatureID | Feature Name | Rationale |
|-------|-----------|-------------|-----------|
| 0.1 | CLM-013 | AKS Cluster | All services deploy here — must exist first |
| 0.2 | CLM-053 | Network Isolation and Private Endpoints | Elevated from P2 — foundational security requirement for all PaaS; must be in place before provisioning data services |
| 0.3 | CLM-016 | Secrets Management | Key Vault required before any service can store credentials |
| 0.4 | CLM-012 | Encryption at Rest and in Transit | CMK and Istio mTLS must be configured before data flows |
| 0.5 | CLM-014 | Ingress and API Gateway | App Gateway WAF + APIM needed before exposing any API |
| 0.6 | CLM-010 | Authentication and SSO | Entra ID app registration + OIDC required for all authenticated services |
| 0.7 | CLM-018 | Entra ID Integration | Deep Entra integration (Conditional Access, PIM, SCIM client) |
| 0.8 | CLM-011 | Role-Based Access Control | RBAC roles + OPA Gatekeeper policies before app deployment |
| 0.9 | CLM-015 | GitOps and CI/CD | Flux v2 + Azure DevOps pipelines — deployment mechanism for all subsequent phases |
| 0.10 | CLM-017 | Observability and Monitoring | Azure Monitor + Grafana + App Insights — must observe from day one |

**Exit criteria:** AKS cluster running, private networking configured, Entra ID integrated, GitOps deploying to dev namespace, monitoring active.

---

## Phase 1 — Core Platform (P0 Application Features)

**Goal:** Deliver the minimum viable CLM: create contracts from templates, store documents, run approval workflows, search contracts, and maintain an audit trail.

**Repos active:** `claim-clm`, `claim-clm-storage`, `claim-clm-infra`

| Order | FeatureID | Feature Name | Dependencies | Rationale |
|-------|-----------|-------------|--------------|-----------|
| 1.1 | CLM-003 | Blob Document Storage | Phase 0 | Storage foundation — all documents land here |
| 1.2 | CLM-004 | Document Metadata Store | Phase 0 | PostgreSQL metadata — contracts, parties, dates, status |
| 1.3 | CLM-005 | Document Retrieval API | CLM-003, CLM-004 | REST API for fetching documents + metadata; CDN + Redis cache |
| 1.4 | CLM-002 | Clause Library | Phase 0 | Clause repository needed before template engine |
| 1.5 | CLM-001 | Contract Template Engine | CLM-002 | Template rendering depends on clause library |
| 1.6 | CLM-035 | REST API (Comprehensive) | CLM-001–005 | Elevated from P1 — full OpenAPI 3.1 REST API fronting all core operations |
| 1.7 | CLM-019 | Immutable Audit Log | CLM-035 | Audit trail must capture all API actions from this point forward |
| 1.8 | CLM-007 | Multi-Stage Approval Workflow | CLM-035, CLM-019 | Workflow engine depends on API and audit log |
| 1.9 | CLM-008 | SLA Enforcement and Escalation | CLM-007 | SLA CronJob monitors workflows |
| 1.10 | CLM-006 | Document Indexing Pipeline | CLM-003, CLM-004 | Async indexing feeds search |
| 1.11 | CLM-009 | Full-Text and Semantic Search | CLM-006 | Search depends on indexed content |

**Exit criteria:** End-to-end contract lifecycle: create from template → store → approve → search. Audit trail active. All P0 application features functional.

---

## Phase 2 — Enhanced Capabilities (P1 Features)

**Goal:** Add collaboration, notifications, AI-powered analysis, e-signatures, counterparty portal, and compliance reporting.

**Repos active:** All 6 repositories

### Sub-phase 2A — Notifications & Real-Time (sprints 1–2)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 2A.1 | CLM-033 | Email and SMS Notifications | Phase 1 |
| 2A.2 | CLM-034 | In-App Real-Time Notifications | Phase 1 |
| 2A.3 | CLM-021 | Collaborative Real-Time Editing | CLM-034 (SignalR hub) |

### Sub-phase 2B — Document Processing (sprints 1–2, parallel with 2A)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 2B.1 | CLM-023 | Content Extraction and OCR | CLM-003 |
| 2B.2 | CLM-024 | Document Preview Service | CLM-003, CLM-005 |

### Sub-phase 2C — AI/ML Foundation (sprints 2–4)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 2C.1 | CLM-030 | Key Term Extraction | CLM-023 (extracted text) |
| 2C.2 | CLM-029 | Contract Summarization | CLM-023 |
| 2C.3 | CLM-031 | Semantic Contract Search | CLM-006 (indexed content) |
| 2C.4 | CLM-032 | Clause Deviation Detection | CLM-002 (clause library), CLM-030 |

### Sub-phase 2D — Contracts & Negotiation (sprints 2–4, parallel with 2C)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 2D.1 | CLM-022 | Dynamic Form Builder | CLM-035 (API) |
| 2D.2 | CLM-027 | Redlining and Track Changes | CLM-021 (real-time editing) |
| 2D.3 | CLM-025 | Approval Delegation | CLM-007 (workflow engine) |
| 2D.4 | CLM-020 | E-Signature Integration | CLM-007 |
| 2D.5 | CLM-036 | Webhook Framework | CLM-035 |

### Sub-phase 2E — External Access (sprints 3–5)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 2E.1 | CLM-028 | Counterparty Portal | CLM-005 (doc retrieval), CLM-027 (redlining) |
| 2E.2 | CLM-026 | External Party Workflow | CLM-028, CLM-007 |
| 2E.3 | CLM-039 | Multi-Factor Authentication | Phase 0 (Entra) |
| 2E.4 | CLM-040 | SCIM User Provisioning | CLM-010 (Entra) |

### Sub-phase 2F — Audit & Compliance (sprints 4–5)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 2F.1 | CLM-038 | Audit Log Search | CLM-019 (audit log) |
| 2F.2 | CLM-037 | Compliance Reporting | CLM-038, CLM-019 |

**Exit criteria:** Real-time collaboration working, AI analysis pipeline operational, e-signatures integrated, counterparty portal live, compliance reports available.

---

## Phase 3 — Advanced Features (P2)

**Goal:** Add advanced AI (risk scoring, PII, drafting assistant), CRM integrations, dashboards, obligation management, multi-tenancy, and DR.

**Repos active:** All 6 repositories

### Sub-phase 3A — Advanced AI (sprints 1–3)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 3A.1 | CLM-041 | Contract Risk Scoring | CLM-030 (key terms) |
| 3A.2 | CLM-042 | PII Detection and Redaction | CLM-023 (extraction) |
| 3A.3 | CLM-054 | Obligation Extraction | CLM-030 |
| 3A.4 | CLM-044 | AI Contract Drafting Assistant | CLM-002 (clauses), CLM-031 (semantic search) |
| 3A.5 | CLM-043 | Contract Renewal Prediction | CLM-041 (risk data) |

### Sub-phase 3B — Integrations (sprints 1–3, parallel with 3A)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 3B.1 | CLM-045 | Salesforce Connector | CLM-035 (REST API) |
| 3B.2 | CLM-046 | SAP Integration | CLM-035 |
| 3B.3 | CLM-047 | Microsoft Teams Integration | CLM-007 (workflows) |

### Sub-phase 3C — Reporting & Search (sprints 2–4)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 3C.1 | CLM-048 | Saved Searches and Alerts | CLM-009 (search) |
| 3C.2 | CLM-049 | Executive Dashboard | CLM-041 (risk scores), CLM-004 (metadata) |
| 3C.3 | CLM-050 | Custom Report Builder | CLM-049 |
| 3C.4 | CLM-051 | Document Classification | CLM-023 (extraction) |

### Sub-phase 3D — Platform Hardening (sprints 3–5)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 3D.1 | CLM-055 | Milestone and Renewal Tracking | CLM-054 (obligations) |
| 3D.2 | CLM-056 | Guest User Access | CLM-028 (portal) |
| 3D.3 | CLM-052 | Data Loss Prevention | Phase 0 (security baseline) |
| 3D.4 | CLM-057 | Multi-Tenancy | Phase 1 (data layer) |
| 3D.5 | CLM-058 | Disaster Recovery | Phase 0 (infra), CLM-057 |

**Exit criteria:** Full AI pipeline (risk + PII + obligations + drafting), CRM integrations live, executive dashboards, multi-tenant isolation, DR tested.

---

## Phase 4 — Innovation & Optimization (P3)

**Goal:** Add cutting-edge AI features, additional integrations, and operational excellence tooling.

**Repos active:** `claim-clm-ai`, `claim-clm-integration`, `claim-clm-storage`, `claim-clm-infra`

### Sub-phase 4A — Advanced AI (sprints 1–3)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 4A.1 | CLM-060 | Multi-Language Contract Support | CLM-029 (summarization) |
| 4A.2 | CLM-059 | Contract Benchmarking | CLM-041 (risk scoring) |
| 4A.3 | CLM-061 | Sentiment and Tone Analysis | Phase 2 (notifications) |
| 4A.4 | CLM-066 | Predictive Analytics | CLM-043 (renewal prediction) |
| 4A.5 | CLM-067 | Contract Intelligence Weekly Digest | CLM-049 (dashboards) |
| 4A.6 | CLM-062 | Voice and Chat Interface | CLM-009 (search), CLM-029 (summarization) |

### Sub-phase 4B — Extended Integrations (sprints 1–3, parallel with 4A)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 4B.1 | CLM-063 | GraphQL API | CLM-035 (REST API) |
| 4B.2 | CLM-064 | Microsoft 365 Add-In | CLM-002 (clauses), CLM-027 (redlining) |
| 4B.3 | CLM-065 | Power Automate Connector | CLM-035 |

### Sub-phase 4C — Operational Excellence (sprints 2–4)

| Order | FeatureID | Feature Name | Dependencies |
|-------|-----------|-------------|--------------|
| 4C.1 | CLM-068 | Legal Hold Management | CLM-003 (blob storage) |
| 4C.2 | CLM-069 | Chaos Engineering | Phase 3D.5 (DR) |
| 4C.3 | CLM-070 | Zero Trust Network Access | CLM-053 (network isolation) |
| 4C.4 | CLM-071 | FinOps and Cost Optimisation | All previous phases |

**Exit criteria:** All 71 features delivered. Multi-language support, predictive analytics, M365 integration, chaos resilience, and FinOps in place.

---

## Critical Path

The critical path through the implementation is:

```
Phase 0 (Infra) → CLM-003/004 (Storage) → CLM-005 (Retrieval API) → CLM-001/002 (Templates/Clauses)
  → CLM-035 (REST API) → CLM-007 (Workflows) → CLM-023 (OCR) → CLM-030 (Key Terms)
  → CLM-041 (Risk Scoring) → CLM-049 (Dashboard) → CLM-066 (Predictive Analytics)
```

Infrastructure and data layer are the primary bottlenecks. Parallelizing across repos (AI, integration, portal) significantly reduces total delivery time.

---

## Recommended Team Allocation

| Team | Repos | Phase Focus |
|------|-------|-------------|
| Platform/Infra | `claim-clm-infra` | Phase 0, then continuous |
| Core Services | `claim-clm` | Phases 1–3 |
| Storage & Processing | `claim-clm-storage` | Phases 1–3 |
| AI/ML | `claim-clm-ai` | Phases 2–4 |
| Integrations | `claim-clm-integration` | Phases 2–4 |
| External Portal | `claim-clm-portal` | Phases 2–3 |
