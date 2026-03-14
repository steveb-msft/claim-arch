# CLM Feature-to-Repository and Service Mapping

> Generated from `CLM_Enterprise_Features.xlsx` — 71 features across 6 repositories

## Repository Overview

| Repository | Status | Tech Stack | Description |
|------------|--------|------------|-------------|
| `claim-clm` | Existing | .NET 8 API + React 18/TS | Core CLM platform: contracts, workflows, search, notifications, audit, RBAC |
| `claim-clm-storage` | Existing | Python 3.12 / FastAPI | Document storage, retrieval, indexing, extraction, preview, classification |
| `claim-clm-ai` | **New** | Python / FastAPI | AI/ML services: summarization, NER, risk scoring, deviation, drafting, PII, translation |
| `claim-clm-integration` | **New** | .NET 8 / Node.js | External integrations: e-signature, Salesforce, SAP, Teams, M365, Power Automate, GraphQL |
| `claim-clm-portal` | **New** | Next.js 14 / React | External counterparty portal with guest authentication |
| `claim-clm-infra` | **New** | Terraform / Bicep | Infrastructure as Code, GitOps (Flux v2), OPA policies, DR runbooks, monitoring dashboards |

---

## Services Per Repository

### claim-clm (Core Platform)

| Service | Type | K8s Namespace | Description |
|---------|------|---------------|-------------|
| `clm-api` | Microservice | clm-core | Main .NET 8 REST API — contracts, clauses, RBAC, audit, compliance, SCIM |
| `clm-web` | Frontend | clm-core | React 18 + TypeScript SPA — UI for all core CLM operations |
| `clm-template-service` | Microservice | clm-core | Contract template engine with variable substitution (DOCX/PDF) |
| `clm-workflow-engine` | Microservice | clm-core | Multi-stage approval workflows (Temporal.io / Durable Functions) |
| `clm-sla-enforcer` | CronJob | clm-core | SLA monitoring every 5 min, escalation via Service Bus |
| `clm-notification-worker` | Worker | clm-core | Email/SMS dispatcher via Azure Communication Services |
| `clm-realtime-hub` | Microservice | clm-core | Azure SignalR hub for collaborative editing + in-app notifications |
| `clm-webhook-dispatcher` | Worker | clm-core | Outbound webhook delivery with HMAC signing and retry |
| `clm-audit-writer` | Worker | clm-core | Cosmos DB append-only audit log writer |
| `clm-search-service` | Microservice | clm-core | Full-text and semantic search via Azure AI Search |

### claim-clm-storage (Document Storage & Processing)

| Service | Type | K8s Namespace | Description |
|---------|------|---------------|-------------|
| `clm-storage-api` | Microservice | clm-data | Document storage/retrieval FastAPI — blob, metadata, legal hold |
| `clm-doc-indexer` | Worker | clm-data | Async document indexing pipeline (KEDA-scaled on Service Bus) |
| `clm-doc-extractor` | Worker | clm-data | OCR/content extraction via Azure AI Document Intelligence |
| `clm-doc-preview` | Microservice | clm-data | Document preview/conversion (Gotenberg/LibreOffice) |
| `clm-doc-classifier` | Worker | clm-data | Document type classification (NDA, MSA, SOW, PO, etc.) |

### claim-clm-ai (AI/ML Services)

| Service | Type | K8s Namespace | Description |
|---------|------|---------------|-------------|
| `clm-ai-summarizer` | Worker | clm-ai | GPT-4o contract summarization |
| `clm-ai-extractor` | Worker | clm-ai | Key term + obligation extraction (Doc Intelligence + Custom NER) |
| `clm-ai-risk-scorer` | Microservice | clm-ai | ML-based contract risk scoring (0-100) |
| `clm-ai-deviation` | Worker | clm-ai | Clause deviation detection vs approved library |
| `clm-ai-drafter` | Microservice | clm-ai | RAG-based AI drafting assistant (SSE streaming) |
| `clm-ai-pii` | Worker | clm-ai | PII detection and redaction via Azure AI Language |
| `clm-ai-search` | Worker | clm-ai | Vector/semantic search indexer (embeddings → AI Search) |
| `clm-ai-translator` | Worker | clm-ai | Multi-language translation via Azure AI Translator |
| `clm-ai-sentiment` | Worker | clm-ai | Sentiment/tone analysis on negotiation correspondence |
| `clm-ai-prediction` | Microservice | clm-ai | Renewal prediction, benchmarking, predictive analytics (Azure ML) |
| `clm-ai-chatbot` | Microservice | clm-ai | Voice/chat conversational interface (GPT-4o function calling) |
| `clm-ai-digest` | CronJob | clm-ai | Weekly contract intelligence digest generator |

### claim-clm-integration (External Integrations)

| Service | Type | K8s Namespace | Description |
|---------|------|---------------|-------------|
| `clm-esign-adapter` | Microservice | clm-core | DocuSign + Adobe Sign adapter with webhook receiver |
| `clm-salesforce-connector` | Microservice | clm-core | Bi-directional Salesforce CRM sync |
| `clm-sap-connector` | Microservice | clm-core | SAP S/4HANA integration via OData v4 |
| `clm-teams-bot` | Microservice | clm-core | Microsoft Teams bot with Adaptive Cards |
| `clm-m365-addin` | Frontend | clm-ingress | Word/Outlook Office Add-in (Azure Static Web Apps) |
| `clm-power-automate` | API Definition | clm-core | Power Automate custom connector (OpenAPI 2.0) |
| `clm-graphql-api` | Microservice | clm-core | GraphQL API layer (Hot Chocolate / Strawberry) |

### claim-clm-portal (External Counterparty Portal)

| Service | Type | K8s Namespace | Description |
|---------|------|---------------|-------------|
| `clm-counterparty-portal` | Frontend | clm-ingress | Next.js 14 external counterparty portal |
| `clm-guest-auth-service` | Microservice | clm-ingress | Guest user lifecycle (Entra External ID) |

### claim-clm-infra (Infrastructure as Code)

| Module | Type | Description |
|--------|------|-------------|
| `modules/aks` | Terraform/Bicep | AKS cluster, node pools, auto-scaling |
| `modules/networking` | Terraform/Bicep | VNet, subnets, NSGs, private endpoints, Bastion |
| `modules/data` | Terraform/Bicep | PostgreSQL, Cosmos DB, Storage Accounts, Redis |
| `modules/security` | Terraform/Bicep | Key Vault, encryption, Entra ID app registrations |
| `modules/monitoring` | Terraform/Bicep | Azure Monitor, Grafana, App Insights, OTel |
| `modules/messaging` | Terraform/Bicep | Service Bus, SignalR Service |
| `modules/ai` | Terraform/Bicep | Azure OpenAI, AI Search, Doc Intelligence, ML workspace |
| `modules/gateway` | Terraform/Bicep | App Gateway WAF v2, Front Door, API Management |
| `modules/dr` | Terraform/Bicep | DR infrastructure, geo-replication, failover config |
| `gitops/` | Flux v2 | GitOps HelmRelease and Kustomization manifests |
| `policies/` | OPA Gatekeeper | Admission control policies |
| `dashboards/` | Grafana JSON | Monitoring dashboards |
| `runbooks/` | Azure Automation | DR failover and operational runbooks |

---

## Complete Feature Mapping

| FeatureID | Priority | Category | Feature Name | Repository | Service(s) |
|-----------|----------|----------|-------------|------------|------------|
| CLM-001 | P0 | Contract Creation | Contract Template Engine | `claim-clm` | `clm-template-service` |
| CLM-002 | P0 | Contract Creation | Clause Library | `claim-clm` | `clm-api` |
| CLM-003 | P0 | Document Storage | Blob Document Storage | `claim-clm-storage` | `clm-storage-api` |
| CLM-004 | P0 | Document Storage | Document Metadata Store | `claim-clm-storage` | `clm-storage-api` |
| CLM-005 | P0 | Document Storage | Document Retrieval API | `claim-clm-storage` | `clm-storage-api` |
| CLM-006 | P0 | Document Storage | Document Indexing Pipeline | `claim-clm-storage` | `clm-doc-indexer` |
| CLM-007 | P0 | Workflow Engine | Multi-Stage Approval Workflow | `claim-clm` | `clm-workflow-engine` |
| CLM-008 | P0 | Workflow Engine | SLA Enforcement and Escalation | `claim-clm` | `clm-sla-enforcer` |
| CLM-009 | P0 | Search and Discovery | Full-Text and Semantic Search | `claim-clm` | `clm-search-service` |
| CLM-010 | P0 | Security and Compliance | Authentication and SSO | `claim-clm-infra` | `modules/security` (cross-cutting) |
| CLM-011 | P0 | Security and Compliance | Role-Based Access Control | `claim-clm-infra` | `modules/security` + `clm-api` (cross-cutting) |
| CLM-012 | P0 | Security and Compliance | Encryption at Rest and in Transit | `claim-clm-infra` | `modules/security` + `modules/networking` |
| CLM-013 | P0 | Infrastructure | AKS Cluster | `claim-clm-infra` | `modules/aks` |
| CLM-014 | P0 | Infrastructure | Ingress and API Gateway | `claim-clm-infra` | `modules/gateway` |
| CLM-015 | P0 | Infrastructure | GitOps and CI/CD | `claim-clm-infra` | `gitops/` + Azure DevOps pipelines |
| CLM-016 | P0 | Infrastructure | Secrets Management | `claim-clm-infra` | `modules/security` |
| CLM-017 | P0 | Infrastructure | Observability and Monitoring | `claim-clm-infra` | `modules/monitoring` + `dashboards/` |
| CLM-018 | P0 | Identity and Access Mgmt | Entra ID Integration | `claim-clm-infra` | `modules/security` |
| CLM-019 | P0 | Audit and Compliance | Immutable Audit Log | `claim-clm` | `clm-audit-writer` |
| CLM-020 | P1 | Contract Creation | E-Signature Integration | `claim-clm-integration` | `clm-esign-adapter` |
| CLM-021 | P1 | Contract Creation | Collaborative Real-Time Editing | `claim-clm` | `clm-realtime-hub` + `clm-web` |
| CLM-022 | P1 | Contract Creation | Dynamic Form Builder | `claim-clm` | `clm-web` + `clm-api` |
| CLM-023 | P1 | Document Storage | Content Extraction and OCR | `claim-clm-storage` | `clm-doc-extractor` |
| CLM-024 | P1 | Document Storage | Document Preview Service | `claim-clm-storage` | `clm-doc-preview` |
| CLM-025 | P1 | Workflow Engine | Approval Delegation | `claim-clm` | `clm-workflow-engine` |
| CLM-026 | P1 | Workflow Engine | External Party Workflow | `claim-clm-portal` | `clm-counterparty-portal` + `clm-guest-auth-service` |
| CLM-027 | P1 | Contract Negotiation | Redlining and Track Changes | `claim-clm` | `clm-api` + `clm-web` |
| CLM-028 | P1 | Contract Negotiation | Counterparty Portal | `claim-clm-portal` | `clm-counterparty-portal` |
| CLM-029 | P1 | AI and ML | Contract Summarization | `claim-clm-ai` | `clm-ai-summarizer` |
| CLM-030 | P1 | AI and ML | Key Term Extraction | `claim-clm-ai` | `clm-ai-extractor` |
| CLM-031 | P1 | AI and ML | Semantic Contract Search | `claim-clm-ai` | `clm-ai-search` |
| CLM-032 | P1 | AI and ML | Clause Deviation Detection | `claim-clm-ai` | `clm-ai-deviation` |
| CLM-033 | P1 | Notifications and Alerts | Email and SMS Notifications | `claim-clm` | `clm-notification-worker` |
| CLM-034 | P1 | Notifications and Alerts | In-App Real-Time Notifications | `claim-clm` | `clm-realtime-hub` |
| CLM-035 | P1 | Integration and APIs | REST API (Comprehensive) | `claim-clm` | `clm-api` |
| CLM-036 | P1 | Integration and APIs | Webhook Framework | `claim-clm` | `clm-webhook-dispatcher` |
| CLM-037 | P1 | Audit and Compliance | Compliance Reporting | `claim-clm` | `clm-api` + `clm-web` |
| CLM-038 | P1 | Audit and Compliance | Audit Log Search | `claim-clm` | `clm-api` |
| CLM-039 | P1 | Identity and Access Mgmt | Multi-Factor Authentication | `claim-clm-infra` | `modules/security` (Entra Conditional Access) |
| CLM-040 | P1 | Identity and Access Mgmt | SCIM User Provisioning | `claim-clm` | `clm-api` |
| CLM-041 | P2 | AI and ML | Contract Risk Scoring | `claim-clm-ai` | `clm-ai-risk-scorer` |
| CLM-042 | P2 | AI and ML | PII Detection and Redaction | `claim-clm-ai` | `clm-ai-pii` |
| CLM-043 | P2 | AI and ML | Contract Renewal Prediction | `claim-clm-ai` | `clm-ai-prediction` |
| CLM-044 | P2 | AI and ML | AI Contract Drafting Assistant | `claim-clm-ai` | `clm-ai-drafter` |
| CLM-045 | P2 | Integration and APIs | Salesforce Connector | `claim-clm-integration` | `clm-salesforce-connector` |
| CLM-046 | P2 | Integration and APIs | SAP Integration | `claim-clm-integration` | `clm-sap-connector` |
| CLM-047 | P2 | Integration and APIs | Microsoft Teams Integration | `claim-clm-integration` | `clm-teams-bot` |
| CLM-048 | P2 | Search and Discovery | Saved Searches and Alerts | `claim-clm` | `clm-search-service` |
| CLM-049 | P2 | Reporting and Analytics | Executive Dashboard | `claim-clm` | `clm-web` + `clm-api` |
| CLM-050 | P2 | Reporting and Analytics | Custom Report Builder | `claim-clm` | `clm-web` + `clm-api` |
| CLM-051 | P2 | Document Storage | Document Classification | `claim-clm-storage` | `clm-doc-classifier` |
| CLM-052 | P2 | Security and Compliance | Data Loss Prevention | `claim-clm-infra` | `modules/security` (Purview DLP) |
| CLM-053 | P2 | Security and Compliance | Network Isolation and Private Endpoints | `claim-clm-infra` | `modules/networking` |
| CLM-054 | P2 | Obligation Management | Obligation Extraction | `claim-clm-ai` | `clm-ai-extractor` |
| CLM-055 | P2 | Obligation Management | Milestone and Renewal Tracking | `claim-clm` | `clm-api` + `clm-sla-enforcer` |
| CLM-056 | P2 | Identity and Access Mgmt | Guest User Access | `claim-clm-portal` | `clm-guest-auth-service` |
| CLM-057 | P2 | Infrastructure | Multi-Tenancy | `claim-clm-infra` | cross-cutting (RLS, per-tenant isolation) |
| CLM-058 | P2 | Infrastructure | Disaster Recovery | `claim-clm-infra` | `modules/dr` + `runbooks/` |
| CLM-059 | P3 | AI and ML | Contract Benchmarking | `claim-clm-ai` | `clm-ai-prediction` |
| CLM-060 | P3 | AI and ML | Multi-Language Contract Support | `claim-clm-ai` | `clm-ai-translator` |
| CLM-061 | P3 | AI and ML | Sentiment and Tone Analysis | `claim-clm-ai` | `clm-ai-sentiment` |
| CLM-062 | P3 | AI and ML | Voice and Chat Interface | `claim-clm-ai` | `clm-ai-chatbot` |
| CLM-063 | P3 | Integration and APIs | GraphQL API | `claim-clm-integration` | `clm-graphql-api` |
| CLM-064 | P3 | Integration and APIs | Microsoft 365 Add-In | `claim-clm-integration` | `clm-m365-addin` |
| CLM-065 | P3 | Integration and APIs | Power Automate Connector | `claim-clm-integration` | `clm-power-automate` |
| CLM-066 | P3 | Reporting and Analytics | Predictive Analytics | `claim-clm-ai` | `clm-ai-prediction` |
| CLM-067 | P3 | Reporting and Analytics | Contract Intelligence Weekly Digest | `claim-clm-ai` | `clm-ai-digest` |
| CLM-068 | P3 | Document Storage | Legal Hold Management | `claim-clm-storage` | `clm-storage-api` |
| CLM-069 | P3 | Infrastructure | Chaos Engineering | `claim-clm-infra` | Azure Chaos Studio config |
| CLM-070 | P3 | Security and Compliance | Zero Trust Network Access | `claim-clm-infra` | `modules/security` (Entra GSA) |
| CLM-071 | P3 | Infrastructure | FinOps and Cost Optimisation | `claim-clm-infra` | Kubecost + Azure Cost Management config |

---

## Feature Count by Repository

| Repository | P0 | P1 | P2 | P3 | Total |
|------------|----|----|----|----|-------|
| `claim-clm` | 5 | 12 | 4 | 0 | **21** |
| `claim-clm-storage` | 4 | 2 | 1 | 1 | **8** |
| `claim-clm-ai` | 0 | 4 | 4 | 6 | **14** |
| `claim-clm-integration` | 0 | 1 | 3 | 3 | **7** |
| `claim-clm-portal` | 0 | 2 | 1 | 0 | **3** |
| `claim-clm-infra` | 10 | 1 | 4 | 3 | **18** |
| **Total** | **19** | **22** | **17** | **13** | **71** |
