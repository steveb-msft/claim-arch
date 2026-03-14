# Azure Cost Estimates for CLM at 25,000 Contracts/Month

> **Planning estimate only.** Prices are approximate USD/month, rounded, and assume a production-grade Azure deployment in a mainstream US region on pay-as-you-go pricing. Actual spend will vary by region, reservation terms, data transfer, storage transactions, and support plan. This estimate intentionally separates **core Azure runtime** from **enterprise licensing / embedded analytics add-ons** so the cost model is easier to budget.

## 1. Executive Summary

For a CLM platform processing **25,000 new contracts per month**, the **core Azure production runtime** is approximately **$29,254/month** (**$351,048/year**) when excluding user licensing and Power BI Embedded capacity. A **fully loaded production estimate** including **Microsoft Entra ID P2, Purview, and Power BI Embedded** is approximately **$38,354/month** (**$460,248/year**).

With the cost optimizations listed later (reserved capacity, scale-to-zero, lifecycle policies, and right-sizing of shared services), a realistic **optimized fully loaded target** is **about $30,050/month** (**$360,600/year**), which is a **~21.7% reduction** from the baseline fully loaded estimate and lands in the requested **$28K-$35K/month** planning range.

### Summary by Category

| Category | Monthly Cost | Annual Cost |
|---|---:|---:|
| Compute | $2,480 | $29,760 |
| Data | $5,310 | $63,720 |
| AI & Search | $6,350 | $76,200 |
| Messaging & Real-Time | $3,100 | $37,200 |
| Networking & Security | $9,694 | $116,328 |
| Monitoring | $750 | $9,000 |
| Analytics & Reporting | $4,300 | $51,600 |
| Integration | $1,000 | $12,000 |
| Identity (Licensing) | $5,000 | $60,000 |
| DevOps & Container | $370 | $4,440 |
| **Fully Loaded Total** | **$38,354** | **$460,248** |

### Budget View

| Budget View | Monthly | Annual | Notes |
|---|---:|---:|---|
| Core Azure runtime | $29,254 | $351,048 | Excludes Entra ID P2, Purview, and Power BI Embedded |
| Fully loaded production | $38,354 | $460,248 | Includes all listed services |
| Optimized fully loaded target | $30,050 | $360,600 | Assumes RI/Savings Plan, scale-to-zero, and rightsizing |

## 2. Volume Assumptions

- **25,000** new contracts per month (~830/day, ~35/hour)
- Average contract: **5 pages**, **~250KB** total document size
- Each contract lifecycle: **~100 API calls** (create, edit, review, approve, sign, search)
- **2.5M total API calls/month**
- AI processing per contract: summarization + key term extraction + risk scoring = **75,000 AI operations/month**
- **500 internal users**, ~**50 concurrent** at peak
- **200 external counterparty users**
- Document storage growth: **~6.25GB/month** new content
- Steady-state active documents after 12 months: **~75GB**
- Search index: **~50GB**
- Audit log: **~500K events/month**

### Workload Math Used for Sizing

- **Document pages per month:** 25,000 contracts x 5 pages = **125,000 pages/month**
- **Document storage growth:** 25,000 x 250KB = **~6.25GB/month**
- **Steady-state active storage:** ~6.25GB x 12 months = **~75GB**
- **API traffic:** 25,000 x 100 = **2.5M API calls/month**
- **AI workload:** 25,000 x 3 AI steps = **75,000 AI operations/month**
- **Peak user concurrency:** sized for **~50 internal active users**, workflow bursts, OCR/indexing queues, and approval spikes

## 3. Detailed Cost Breakdown

## Compute

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| AKS System Node Pool | 3x Standard_D4s_v5 | One node per zone (1/2/3) for control-plane add-ons, ingress, metrics, and critical system pods; 128GB Premium SSD per node for OS + daemonset overhead | $420 |
| AKS App Node Pool | 3-10x Standard_D8s_v5 (avg 5) | Main API, workflow, background workers, and integration services. Average 5 nodes supports ~2.5M API calls/month, 50 concurrent users, and bursty indexing/approval traffic with HPA + cluster autoscaler | $1,400 |
| AKS AI Node Pool | 0-4x Standard_NC6s_v3 (avg 1) | Dedicated GPU capacity for ML inference or heavier model-serving workloads; KEDA allows scale-to-zero when idle, so average monthly utilization is modeled at 1 node | $660 |
| **Compute Total** |  |  | **$2,480** |

## Data

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| PostgreSQL Flexible Server (Primary) | Business Critical D8s_v3, zone-redundant HA | Main transactional metadata store for contracts, parties, approvals, clause relationships, and workflow state. Sized for high availability, low-latency writes, and 35-day backup retention with geo-redundant backup | $2,400 |
| PostgreSQL Flexible Server (Secondary) | General Purpose D4s_v3, zone-redundant HA | Separate operational database for clause library, form schemas, and workflow/configuration services to isolate noisy workloads from core transaction processing | $800 |
| Azure Cosmos DB NoSQL | 10,000 RU/s provisioned, multi-region writes | Append-only audit/event store. 500K audit events/month is not massive in raw volume, but multi-region writes, predictable latency, and burst absorption justify provisioned throughput | $1,160 |
| Azure Blob Storage ADLS Gen2 | Hot ~75GB, Cool ~50GB, Archive ~20GB, GRS | Stores active contracts, signed documents, generated PDFs, and extracted artifacts. Includes steady-state hot storage plus lifecycle-managed cooler/archive tiers and GRS protection | $150 |
| Azure Cache for Redis | Premium P2, 13GB, zone-redundant | Used for metadata cache, session cache, document render cache, and rate-limited lookup acceleration under review/search bursts | $800 |
| **Data Total** |  |  | **$5,310** |

## AI & Search

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Azure AI Search | Standard S2, 6 replicas | Supports full-text, semantic, and vector search across contracts and audit data. Six replicas/search units provide headroom for query concurrency, indexing throughput, and HA for ~50GB index footprint | $4,500 |
| Azure OpenAI Service | GPT-4o 100K TPM + text-embedding-ada-002 | Covers summarization, drafting help, deviation detection, obligation extraction, and embeddings. 75K AI operations/month with moderate prompt sizes typically lands in the mid-hundreds/month range | $650 |
| Azure AI Document Intelligence | S0, 20 TPS | OCR + contract extraction for ~125K pages/month. Includes prebuilt contract model usage and some custom classification / extraction passes | $500 |
| Azure AI Language | S tier | Used for custom NER, PII detection, and selective language analytics on contract text, comments, and negotiation metadata | $200 |
| Azure Machine Learning | Standard workspace | Supports periodic risk scoring model training plus a managed inference endpoint for production scoring workloads | $500 |
| **AI & Search Total** |  |  | **$6,350** |

## Messaging & Real-Time

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Azure Service Bus | Premium, 4 messaging units | Backbone for asynchronous OCR, indexing, notifications, workflow transitions, retry handling, and audit fan-out. Premium isolates workloads and absorbs bursts during import/review windows | $2,700 |
| Azure SignalR Service | Premium, 10 units | Real-time collaborative editing presence, live notifications, approval status updates, and in-app event pushes for internal/external users | $300 |
| Azure Communication Services | Email + SMS | Invitation flows, escalations, reminders, approval notifications, and OTP-style communication (~25K emails/month, ~5K SMS/month) | $100 |
| **Messaging & Real-Time Total** |  |  | **$3,100** |

## Networking & Security

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Azure Application Gateway WAF v2 | Prevention mode, OWASP 3.2 | TLS termination, WAF inspection, path routing, and ingress protection for internet-facing CLM apps and portals | $350 |
| Azure Front Door Premium | CDN + geo-routing + health probes | Global edge entry point for low-latency access, CDN offload for static content/documents, and regional failover/DR routing | $400 |
| Azure API Management | Premium, 2 units, zone-redundant | Central API gateway for token validation, rate limiting, partner exposure, policy enforcement, and developer portal with enterprise-grade HA | $5,600 |
| Azure Key Vault Premium | HSM-backed, soft-delete, purge protection | Manages CMK, TLS certs, secrets, API keys, and signing keys used by integrations and document-signing flows | $150 |
| Azure DDoS Protection Standard | Per VNet | Enterprise internet-facing production VNet protection. This line item is expensive but often required by policy for regulated external applications | $2,944 |
| Azure Bastion | Standard | Secure admin access to private AKS and private-network resources without exposing jump boxes | $140 |
| Azure Private Endpoints | ~15 endpoints | Private access to PostgreSQL, Cosmos DB, Blob, Redis, Key Vault, Service Bus, OpenAI, Search, and other PaaS services | $110 |
| **Networking & Security Total** |  |  | **$9,694** |

## Monitoring

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Azure Monitor + Log Analytics | Per-GB ingestion | Container Insights, platform logs, AKS diagnostics, PostgreSQL diagnostics, and security logs at ~50GB/month ingestion | $500 |
| Azure Application Insights | Per-GB ingestion | Distributed tracing for APIs, workflow services, queues, background workers, and UI telemetry at ~20GB/month | $200 |
| Azure Managed Grafana | Standard | Shared dashboards for platform SRE/operations teams across AKS, PostgreSQL, Redis, and Service Bus | $50 |
| **Monitoring Total** |  |  | **$750** |

## Analytics & Reporting

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Azure Synapse Analytics | Serverless SQL pool | Ad hoc reporting/query layer over curated exports, audit data, and reporting views without a permanently running warehouse | $200 |
| Power BI Embedded | A4 SKU | Executive dashboards, compliance reports, and embedded customer/ops reporting experiences sized for a moderate but always-available audience | $4,100 |
| **Analytics & Reporting Total** |  |  | **$4,300** |

## Integration

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Azure Bot Service | S1 | Teams bot and web chat experiences for contract status, guided intake, and self-service actions | $500 |
| Azure Logic Apps | Standard | SAP and enterprise orchestration, document movement, approval triggers, and adapter-based integrations | $200 |
| Azure AI Translator | S2, 100M chars/month (Phase 4 only) | Optional multilingual support for global legal/commercial teams; included here as a full-feature deployment assumption | $300 |
| **Integration Total** |  |  | **$1,000** |

## Identity (Licensing)

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Microsoft Entra ID P2 | Per user | 500 internal users with Conditional Access, PIM, MFA, identity governance, and higher-assurance access controls | $4,500 |
| Microsoft Purview | Information Protection | DLP / labeling / information protection overlays for documents and policy enforcement | $500 |
| **Identity Total** |  |  | **$5,000** |

## DevOps & Container

| Service | SKU | Sizing rationale | Monthly Cost |
|---|---|---|---:|
| Azure Container Registry | Premium, geo-replicated | Stores Docker images, Helm charts OCI artifacts, and release images for HA deployment pipelines | $170 |
| Azure DevOps | Basic plan, 5 parallel jobs | CI/CD pipelines, repo/boards usage, and baseline delivery automation for the CLM platform | $200 |
| **DevOps & Container Total** |  |  | **$370** |

## Baseline Roll-Up

| Roll-Up | Monthly Cost |
|---|---:|
| Fully loaded production total | $38,354 |
| Less Entra ID P2 + Purview | -$5,000 |
| Less Power BI Embedded | -$4,100 |
| **Core Azure runtime subtotal** | **$29,254** |

That **$29.3K/month** core runtime subtotal is the closest representation of the Azure platform cost for the CLM workload itself. The additional **$9.1K/month** reflects enterprise identity licensing and always-on embedded reporting capacity.

## 4. Cost Optimization Recommendations

1. **Reserved Instances / Reserved Capacity (1-year and 3-year)**
   - Apply to the steady-state AKS system/app nodes, PostgreSQL Flexible Server, and Redis.
   - Typical savings: **20-45%** depending on term and service.
   - Best targets: AKS system/app pools, PostgreSQL primary/secondary, Redis Premium.

2. **KEDA scale-to-zero for AI node pool and non-prod**
   - Keep the GPU pool at zero when there is no inference queue backlog.
   - Use the same pattern for dev/test worker pools and batch processors.
   - Likely savings: **$300-$700/month in prod**, much more in non-prod.

3. **Spot instances for ML training jobs**
   - Use Azure ML compute clusters on Spot for retraining and experimentation.
   - Keep production inference on regular capacity, but move non-critical training to Spot.
   - Savings can reach **60-80%** for intermittent training jobs.

4. **Blob Storage lifecycle policies (Hot -> Cool -> Archive)**
   - Auto-tier older signed contracts, prior versions, extracted OCR artifacts, and dormant negotiation packages.
   - This is low-risk because access frequency drops sharply after execution.
   - Savings are modest at current volume, but compound materially as data accumulates.

5. **DDoS Protection evaluation**
   - Re-check whether CLM needs a dedicated DDoS Protection Standard charge on its own VNet or whether a shared enterprise pattern / alternate network protection model is acceptable.
   - This is one of the largest individual optimization levers.

6. **API Management tiering**
   - Keep **Premium** in production if private networking, multi-region patterns, and advanced policy isolation are required.
   - For **non-prod**, use **Consumption** or **Developer** tier rather than duplicating Premium.

7. **Azure Savings Plan for compute**
   - Use Savings Plan for bursty compute not well-covered by Reservations, especially autoscaled AKS app nodes.
   - This complements Reservations for mixed or changing workloads.

8. **Additional practical optimizations**
   - Reduce Application Insights and Log Analytics ingestion via sampling and shorter hot retention.
   - Right-size Search replicas/partitions after real query telemetry is available.
   - Schedule Power BI Embedded capacity around business hours if 24x7 concurrency is unnecessary.
   - Treat Translator as truly optional until Phase 4 adoption justifies always-on budget allocation.

## 5. Optimized Monthly Estimate

The table below shows a realistic optimized target after applying the recommendations above. This assumes:
- 1-year reserved capacity or Savings Plan on stable compute/data layers
- AI GPU scale-to-zero working as designed
- non-critical AML training moved to Spot
- Blob lifecycle policies enabled
- logging/telemetry sampling tuned
- Power BI Embedded scheduled/autoscaled instead of fully saturated 24x7
- DDoS / shared enterprise security costs partially amortized or right-sized

| Category | Baseline | Optimized | Main Optimization Lever |
|---|---:|---:|---|
| Compute | $2,480 | $1,700 | RI/Savings Plan + AI scale-to-zero |
| Data | $5,310 | $4,000 | PostgreSQL/Redis reservations + storage tiering |
| AI & Search | $6,350 | $5,200 | Prompt efficiency, AML Spot, Search right-sizing |
| Messaging & Real-Time | $3,100 | $2,700 | Rightsize Service Bus / SignalR after telemetry |
| Networking & Security | $9,694 | $7,300 | Shared/right-sized DDoS and tighter private endpoint/APIM allocation |
| Monitoring | $750 | $500 | Sampling + lower retention |
| Analytics & Reporting | $4,300 | $2,500 | Power BI scheduling/autoscale |
| Integration | $1,000 | $850 | Translator phased / lower Logic Apps throughput |
| Identity (Licensing) | $5,000 | $5,000 | Usually fixed unless license scope changes |
| DevOps & Container | $370 | $300 | Small savings only |
| **Optimized Fully Loaded Total** | **$38,354** | **$30,050** | **~21.7% reduction** |

### Optimized Budget View

| Budget View | Monthly | Annual |
|---|---:|---:|
| Optimized core Azure runtime | $20,950 | $251,400 |
| Optimized fully loaded production | $30,050 | $360,600 |

## 6. Scaling Projections

Not every service scales linearly. Identity, ACR, Bastion, and some gateway/security costs are relatively fixed, while AKS app nodes, PostgreSQL, Search, Cosmos DB, Service Bus, OCR, and AI usage scale more directly with contract volume.

| Contract Volume | Estimated Monthly Cost | Planning Note |
|---|---:|---|
| 25K/month | $30K-$38K | Baseline described in this document; optimized target is near $30K fully loaded |
| 50K/month | $43K-$48K | Search, API, messaging, DB, and OCR/AI usage begin to dominate |
| 100K/month | $65K-$75K | Expect larger PostgreSQL tiers, more Search capacity, higher Service Bus/Cosmos throughput, and more app nodes |
| 250K/month | $135K-$155K | At this scale, Search, AI, messaging, audit storage, and database partitioning/scale-out become major design constraints |

### Scaling Interpretation

- **2x volume does not equal 2x cost** because gateway, identity, and several platform services are semi-fixed.
- The most elastic cost drivers are **AKS app capacity, PostgreSQL, Search, Service Bus, OpenAI/Document Intelligence, and audit/event storage**.
- At **100K+ contracts/month**, redesign choices (event partitioning, index sharding, data retention, and caching strategy) matter more than list-price arithmetic.

## 7. Non-Prod Environment Costs

A reasonable pattern is to size **Dev + Staging** together at **30-40% of production** by using:
- smaller PostgreSQL / Redis SKUs
- fewer AKS nodes
- no dedicated GPU except on-demand
- non-prod APIM on Consumption/Developer tier
- Search with fewer replicas
- scale-to-zero worker pools and batch services
- lighter monitoring retention

| Environment Set | Estimated Monthly Cost | Notes |
|---|---:|---|
| Dev | $3,500-$4,500 | Small AKS footprint, smaller DB/cache, scale-to-zero AI |
| Staging | $5,500-$7,500 | Closer to prod topology but fewer replicas and smaller SKUs |
| **Dev + Staging Total** | **$9,000-$12,000** | Roughly 30-40% of optimized production |

## Final Recommendation

For budgeting purposes, use the following numbers:

- **Core Azure runtime budget:** **~$29.3K/month** baseline
- **Fully loaded production budget:** **~$38.4K/month** baseline
- **Target after optimization:** **~$30.1K/month** fully loaded
- **Dev + Staging:** **~$9K-$12K/month combined**

If leadership wants a single planning number for a production-ready, full-feature CLM rollout at **25,000 contracts/month**, the most defensible figure is **~$30K-$32K/month after standard Azure cost controls are applied**.