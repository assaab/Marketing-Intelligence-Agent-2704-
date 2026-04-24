# 02 Technical Architecture

## System Overview
The `Marketing Intelligence Agent` is architected as a bounded commercial decision system. The core of the platform is a deterministic decision pipeline that ingests campaign and CRM signals, constructs health and evidence state, ranks bounded actions with economic and policy logic, and routes each recommendation into an explicit autonomy lane. LLMs remain useful in the architecture, but only for bounded synthesis, narrative generation, and content support. They do not control final action eligibility, ranking, or execution authority.

## High-Level System Diagram

```text
                           +--------------------------------------+
                           | Enterprise Users / External Systems  |
                           | marketers, analysts, approvers, APIs |
                           +------------------+-------------------+
                                              |
                                              v
                           +--------------------------------------+
                           | UI / API Edge                        |
                           | Next.js frontend / FastAPI endpoints |
                           +------------------+-------------------+
                                              |
                           +--------------------------------------+
                           | Identity and Policy Layer            |
                           | JWT validation, RBAC, rate limits,   |
                           | tenancy, action policy               |
                           +------------------+-------------------+
                                              |
                    +-------------------------+-------------------------+
                    |                                                   |
                    v                                                   v
      +--------------------------------------+          +----------------------------------+
      | Deterministic Decision Pipeline      |          | Worker / Queue / Scheduling      |
      | discovery, KPI state, evidence,      |          | onboarding, retries, background  |
      | business state, ranking, routing     |          | monitoring, outcome processing   |
      +-------------------+------------------+          +----------------+-----------------+
                          |                                                |
                          +-----------------------+------------------------+
                                                  |
                  +-------------------------------+--------------------------------+
                  |                                                                |
                  v                                                                v
     +-------------------------------+                              +-------------------------------+
     | Connectors and Data Adapters  |                              | Persistence and Audit         |
     | GA4, Google Ads, HubSpot,     |                              | workflow state, actions,      |
     | Meta, optional warehouse      |                              | outcomes, audit events        |
     +----------------------+--------+                              +----------------------+--------+
                            |                                                             ^
                            v                                                             |
                 +-------------------------------+                              +----------+-----------+
                 | Bounded LLM Narrative Layer   |----------------------------->| Telemetry and Health |
                 | content, summaries, readouts  |                              | metrics, readiness,  |
                 | strict schema + fallback      |                              | logs, tracing        |
                 +-------------------------------+                              +----------------------+
```

## Component Model

| Component | Responsibility | Current evidence |
| --- | --- | --- |
| Frontend | Dashboard, approval views, audit visibility, workflow entry points | Next.js application |
| API layer | REST endpoints, auth, RBAC, request controls, orchestration | FastAPI application |
| Decision pipeline services | Health construction, evidence scoring, business-state detection, ranking, routing | Backend service layer |
| Portfolio context services | Account-level alternatives, reallocation pressure, opportunity-cost signals | `portfolio_context.py`, dashboard projection logic |
| Decision packet service | Canonical review artifact, route classification, autonomy tier, exception codes | `decision_packet.py` |
| Worker and queue | Scheduled monitoring, connector onboarding, retries, background processing | Worker loop and queue services |
| Connector layer | CRM, analytics, media, and optional warehouse integrations | GA4, Google Ads, HubSpot, optional Meta, adapter-ready patterns; slice and creative ingestion services support richer evidence inputs |
| Campaign identity and mappings | Tenant-scoped slice and external-id mappings so analytics, ads, and CRM entities reconcile for reporting | `campaign_mapping_service`, `campaign_mappings` router, Alembic migrations for slice and mapping tables |
| Business inputs | Validated runtime inputs that feed scoring and gates | `business_inputs_validation` and related runtime wiring |
| Research subsystem | Evidence-first research with policy and internal context | `research_evidence_gate`, `research_retrieval_policy`, `research_internal_context`, and downstream `research_to_*` transformers |
| Persistence | Workflow state, actions, recommendations, outcomes, audit events, summaries | In-memory store and optional PostgreSQL |
| LLM narrative layer | Content variants, summaries, operator-facing narrative, executive readouts | OpenAI-compatible API integration with schema validation |
| Observability | Metrics, readiness, latency, logs, and cloud telemetry | Metrics endpoint and Application Insights support |

## Decision-Engine Pipeline

| Stage | Primary responsibility | Key outputs |
| --- | --- | --- |
| Connect and discover | Authenticate users, attach connectors, discover account and campaign scope | Workspace context, campaign catalog, monitored scope |
| Analyze and build health state | Normalize metrics and build typed campaign health response | KPI snapshot, source breakdown, economic snapshot |
| Evidence sufficiency and degraded-insight gating | Score freshness, completeness, sample sufficiency, and attribution risk | Confidence penalties, degraded-insight posture, actionability floor |
| Business-state detection and contradiction resolution | Determine health tier, momentum tier, intervention posture, and contradiction notes | Stable state frame plus `business_state_flags` |
| Bounded candidate generation | Emit only allowed action families for the resolved state | Candidate action set |
| Portfolio-aware ranking and recalibration | Apply economic scoring, state adjustments, portfolio pressure, and bounded historical priors | `economic_priority_score`, score breakdown, recalibration context |
| Decision packet assembly | Build recommendation payload plus canonical commercial review artifact | `decision_packet`, trust notes, decision grade, routing metadata |
| Approval and execution routing | Map output into observe, recommend, approval, micro-action, or escalation lanes | `decision_route`, `autonomy_tier`, `exception_codes` |
| Measurement and learning | Record outcomes, validate eligibility, and update bounded priors | Outcome records, action memory, future recalibration inputs |

## Primary Data Flows

### 1. Analyze, rank, and route
1. User, scheduler, or connector event invokes campaign analysis.
2. API authenticates principal and enforces tenant and role boundaries.
3. Decision services read channel and CRM signals through connectors.
4. Deterministic services compute KPI state, evidence sufficiency, and business posture.
5. Candidate actions are ranked with economic, portfolio, and state-aware scoring.
6. Recommendation plus `decision_packet` are persisted and returned through API or dashboard.

### 2. Portfolio overview and dashboard bootstrap
1. Repository and dashboard services read current recommendation state across the account.
2. Top recommendation briefs are sorted using `economic_priority_score` and route priority.
3. Portfolio summaries expose `decision_route`, portfolio pressure, opportunity-gap signals, and business-state flags.

### 3. Approval-bound action lifecycle
1. Only recommendations routed to `propose_for_approval` enter the approval path.
2. Approvers inspect decision packet details, exception codes, rollback framing, and evidence.
3. Approved low-risk actions move to bounded execution or controlled simulation.
4. Lifecycle state and audit chain are updated for every transition.

### 4. Bounded LLM narrative and content assistance
1. Operator or workflow requests content, narrative, or executive summary support.
2. System prompt constrains output to a strict schema.
3. Response is parsed, validated, and screened before use.
4. On failure, the system falls back to deterministic templates or seeded outputs.

### 5. Outcome measurement and recalibration
1. Approved or executed actions generate outcome rows tied to the originating recommendation.
2. Eligibility filters exclude low-quality or incomplete observations from learning.
3. Campaign-scope history is used first, then account-scope history, with bounded weight.
4. Future recommendations receive recalibration context, but safety gates remain dominant.

## Repository navigation and impact analysis

The workspace maintains a **Code Review Graph** under `.code-review-graph/` (gitignored generated index). After route moves, schema changes, or large service refactors, regenerate that graph so dependency and impact queries stay accurate. Operational guidance lives in the root `AGENTS.md` and `docs/repo-agent-guide.md`. This is a developer and reviewer ergonomics tool, not a runtime dependency of the product.

## Optional LangGraph evidence pipeline

When the API runs the campaign worker in LangGraph mode with `AGENT_EVIDENCE_GRAPH=true` (`backend/app/agent/graph_evidence.py`), the pipeline is intentionally **shortened** for latency: deterministic analyze first, then LLM stages for evidence pack, diagnosis, packaging approvals, policy readout, optional human interrupt, measurement framing, and learning, then finalize. A simpler four-node graph exists when `AGENT_EVIDENCE_GRAPH=false`. Deterministic recommendation ranking, `decision_packet` assembly, and policy gates in services remain the **authority** for commercial eligibility; the graph adds narrative and structured stage outputs around that core. For the exact node list and edges, see `docs/agentic-production-flow.md` in the repository.

## Decision Policy And Routing

| Control surface | Purpose | Current posture |
| --- | --- | --- |
| Evidence sufficiency gate | Prevent action-ready commercial moves when proof is weak | Active |
| `decision_route` | Separates observe, recommend, approval, micro-action, and escalation lanes | Active |
| `autonomy_tier` | Binds the route to an explicit authority level | Active |
| `exception_codes` | Makes downgrade, block, or escalation reasons inspectable | Active |
| `decision_packet` | Provides a single review artifact for approval, audit, and downstream APIs | Active |
| `portfolio_opportunity_context` | Adds account-level alternative-use-of-budget reasoning | Active |
| `recalibration_context` | Adds bounded historical outcome learning without bypassing policy | Active |

- Material actions are downgraded or blocked when economics are defaulted, insight quality is degraded, attribution risk is high, or contradictions remain unresolved.
- `execute_micro_action` exists as a bounded lane for reversible, benchmark-proven playbooks, but autonomy expansion remains intentionally narrow.
- The ranking policy prefers the highest eligible action after policy gating, not the highest raw score before guardrails.

## Guardrails and Control Boundaries

| Boundary | Control |
| --- | --- |
| Material budget or audience actions | Require decision-ready evidence and human approval |
| Degraded insight or sparse data | Downgrade to observe-only, recommend-only, or test-first posture |
| Defaulted economics | Prevent approval-ready commercial positioning |
| High attribution risk | Downgrade direct actions and favor measurement hardening or escalation |
| Contradiction bundles | Force observe-only or escalation behavior when signals conflict |
| LLM outputs | Never directly determine final action eligibility or trigger platform writes |
| Tool scope | LLM layer is not granted unrestricted production mutation access |
| Authentication | Bearer token validation in enterprise mode; dev bypass disabled in production |
| Authorization | RBAC enforcement at route and action lifecycle boundaries |
| Audit | Append-only audit events with traceable lifecycle transitions |
| Tenant isolation | Tenant-scoped reads and writes across repository and persistence paths |
| Data exposure | Contracts favor campaign ids and aggregated metrics over raw PII |

## LLM Strategy

### LLM usage model

| Workload | LLM usage | Rationale |
| --- | --- | --- |
| KPI computation, evidence scoring, business-state detection | No | Numeric and policy-sensitive logic is deterministic |
| Recommendation ranking and route selection | No | Final action policy must be repeatable and inspectable |
| Content variants and creative suggestions | Yes, bounded | Suitable for generative variation with schema and review controls |
| Executive summaries and operator-friendly narratives | Yes, bounded | Good fit for synthesis once deterministic evidence exists |
| Execution actions | No | Avoids unsafe autonomy in spend-affecting operations |

### Model routing
- Route numeric and policy-sensitive workflows to deterministic services first.
- Route content and summary tasks to an OpenAI-compatible or sovereign endpoint only when configured.
- Preserve fallback templates when the model is unavailable or returns malformed output.
- Store model and prompt metadata for audit where enterprise policy requires it.

## Cloud and Deployment Compatibility

### Cloud-agnostic design
- Containerized application stack with standard HTTP APIs
- No hard dependency on a single cloud-native AI service contract
- Optional PostgreSQL rather than proprietary database lock-in
- OpenAI-compatible inference interface for model portability
- Environment-variable-driven configuration for deployment portability

### Azure enterprise compatibility

| Azure control plane area | Compatibility pattern |
| --- | --- |
| Identity | Microsoft Entra ID and Azure AD B2C token flows |
| Compute | Container Apps or equivalent container platform |
| Secrets | Key Vault-backed secret injection |
| Telemetry | Application Insights and Log Analytics |
| Data | PostgreSQL Flexible Server or equivalent managed Postgres |
| Networking | Private ingress, managed identity, firewall rules, and private endpoints where required |

### Sovereign LLM integration pattern
- Abstract model access behind a secure API adapter.
- Use allowlisted model routing for narrative and content workloads only.
- Avoid direct model-vendor coupling in business services.
- Log request metadata, not sensitive prompt bodies, when enterprise policy requires audit.
- Preserve deterministic fallback paths if the sovereign model is unavailable.

## Security Architecture

### AuthN and AuthZ

| Control | Implementation posture |
| --- | --- |
| Authentication | JWT-based Bearer auth in enterprise mode; OIDC discovery and JWKS validation |
| Authorization | RBAC by route, workflow state, and action lifecycle |
| Privileged execution | Separate roles for read, propose, approve, execute, and admin |
| Dev bypass | Available only for local development; must be disabled in staging and production |

### RBAC summary

| Capability | Roles |
| --- | --- |
| Read workflows, dashboards, and reports | `Admin`, `MarketingManager`, `Analyst`, `Auditor` |
| Propose actions | `Admin`, `MarketingManager` |
| Approve or reject actions | `Admin`, `MarketingManager` |
| Execute or rollback bounded actions | `Admin`, `MarketingManager`, `Operator` |
| Manage policies, integrations, and secrets | `Admin` |

### Logging, retention, and PII controls

| Area | Control |
| --- | --- |
| Audit logs | Append-only event trail linked to workflow and action lifecycle |
| Application telemetry | Request counts, latencies, readiness, and optional cloud telemetry |
| PII minimization | Use campaign ids and aggregates instead of raw personal records in decision flows |
| CRM payload handling | Least-privilege tokens and no full-payload logging by default |
| Retention | Deployment-specific retention schedule remains an external attachment |
| Data residency | Hosting region and sovereign policy remain deployment-specific attachments |

## Responsible AI Controls

| Control area | Current posture |
| --- | --- |
| Sensitive-segment handling | Policy review and escalation for sensitive or compliance-heavy audience actions |
| Bias and phrase screening | Deterministic scans plus human review for generated content or segment outputs |
| Human oversight | Human approval remains mandatory for material budget, targeting, and externally published content |
| Explainability | Decision packets expose evidence, alternatives, trust notes, exception codes, and measurement framing |
| Safe degradation | Weak evidence, source conflict, or defaulted economics downgrade actionability rather than increase autonomy |
| Bounded learning | Historical outcomes can recalibrate confidence but cannot bypass policy gates |

## Architecture Assessment
- The architecture is intentionally bounded: it optimizes for decision correctness, trust, and auditability rather than unconstrained autonomy.
- The deterministic recommendation layer is now the controlling policy surface for commercial actions.
- Portfolio-aware ranking, business-state flags, decision packets, and bounded recalibration materially improve production readiness over the earlier pilot framing.
- Main remaining submission gaps are external attachments such as named customer references, sovereign model identifiers, and refreshed benchmark artifacts from the target deployment environment.

## Assumptions
- This document separates currently evidenced implementation behavior from deployment-specific items that still require external attachments.
- Exact sovereign networking controls, retention schedules, model identifiers, and named production references were not provided in the workspace and are therefore described as deployment-specific.
- Document revision: April 2026, aligned to the `submission` package `v1.2-repository-refresh` (includes mappings, research evidence gates, exports evidence, public UI `https://www.ai-companion.org/`).
