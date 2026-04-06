# 02 Technical Architecture

## System Overview
The `Marketing Intelligence Agent` is a cloud-agnostic, enterprise-governed marketing operations platform designed to ingest campaign and CRM signals, compute performance insights, generate recommendations and content, and orchestrate low-risk execution workflows under explicit human approval. The architecture favors deterministic analytics for business-critical decisions and restricts LLM usage to bounded, explainable tasks.

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
                           +------------------+-------------------+
                           | Identity and Policy Layer            |
                           | JWT validation, RBAC, rate limits,   |
                           | security headers, action policy      |
                           +------------------+-------------------+
                                              |
                 +----------------------------+-----------------------------+
                 |                            |                             |
                 v                            v                             v
    +-------------------------+   +-------------------------+   +-------------------------+
    | Workflow Orchestrator   |   | Background Worker       |   | Audit and Metrics       |
    | analyze, plan, report,  |   | scheduled jobs, queue   |   | audit chain, health,    |
    | recommend, segment      |   | consumers, retries      |   | readiness, telemetry    |
    +------------+------------+   +------------+------------+   +------------+------------+
                 |                             |                             |
                 +----------------------------+-----------------------------+
                                              |
                     +------------------------+------------------------+
                     |                                                 |
                     v                                                 v
         +-----------------------------+                 +-----------------------------+
         | Connectors / Tool Adapters  |                 | Persistence Layer           |
         | GA4, Google Ads, HubSpot,   |                 | in-memory store, optional   |
         | Meta, optional external API |                 | PostgreSQL, workflow state  |
         +-------------+---------------+                 +-----------------------------+
                       |
                       v
         +-----------------------------+      +--------------------------------------+
         | LLM Adapter Layer           |----->| OpenAI-compatible or sovereign LLM   |
         | strict schema prompts,      |      | endpoint via secure API               |
         | response validation         |      +--------------------------------------+
         +-----------------------------+
```

## Component Model

| Component | Responsibility | Current evidence |
| --- | --- | --- |
| Frontend | Operator dashboard, approvals, audit visibility, workflow entry points | Next.js application |
| API layer | REST endpoints, auth, RBAC, request controls, orchestration | FastAPI application |
| Workflow orchestrator | Analyze, report, recommend, content, segmentation, research workflows | Service layer and workflow endpoints |
| Background worker | Scheduled and queued campaign processing | Worker loop and optional scheduler |
| Connector layer | CRM, analytics, media, and research integrations | GA4, Google Ads, HubSpot, optional Meta, extensible stubs |
| LLM adapter | Bounded content generation and narrative generation | OpenAI-compatible API integration with schema validation |
| Persistence | Audit, workflow state, actions, recommendations, reports, metrics | In-memory store and optional PostgreSQL |
| Observability | Metrics, readiness, latency, telemetry, logs | Metrics endpoint and Application Insights support |

## Primary Data Flows

### 1. Analyze and recommend
1. User or scheduler invokes analyze workflow.
2. API authenticates principal and enforces `read` access.
3. Orchestrator reads channel and CRM signals through connectors.
4. Deterministic analytics computes KPIs, health, and recommendations.
5. Workflow run, recommendations, reports, and audit events are persisted.
6. Dashboard or downstream API consumes normalized outputs.

### 2. Propose, approve, execute
1. Low-risk recommendation is converted to a proposed action.
2. Policy engine classifies action risk and blocks disallowed actions.
3. Authorized approver accepts or rejects.
4. Authorized executor performs bounded connector action or simulated execution.
5. Action state, provider result, and audit chain are updated.

### 3. LLM-assisted content
1. Operator or workflow requests content generation.
2. System prompt constrains output to one strict JSON object.
3. Objective text is inserted as a bounded string without open-ended tool access.
4. Response is parsed and validated against schema.
5. Deterministic phrase scans run for policy and bias screening.
6. On failure, the workflow falls back to templates rather than accepting unsafe output.

## Guardrails and Control Boundaries

| Boundary | Control |
| --- | --- |
| High-risk business actions | Rejected by policy; not silently executed |
| LLM outputs | Never directly trigger platform writes |
| Tool scope | LLM is not granted unrestricted tool calling for production mutations |
| Authentication | Bearer token validation in enterprise mode; dev bypass disabled in production |
| Authorization | Role-based enforcement at route and action lifecycle boundaries |
| Audit | Append-only chained audit events for workflow and action traceability |
| Tenant isolation | Tenant-scoped reads and writes across repository and persistence paths |
| Data exposure | Workflow contracts favor campaign ids and aggregated metrics over raw PII |

## LLM Strategy

### LLM usage model

| Workload | LLM usage | Rationale |
| --- | --- | --- |
| KPI analysis and recommendations | No or minimal | Deterministic logic improves repeatability and explainability |
| Content variants and creative suggestions | Yes, bounded | Suitable for generative variation with schema controls |
| Research narrative augmentation | Optional | Useful for summarization with fallback path |
| Execution actions | No | Avoids unsafe autonomy in spend-affecting operations |

### Model routing
- Route numeric and policy-sensitive workflows to deterministic services first.
- Route content and narrative tasks to an OpenAI-compatible endpoint only when configured.
- Maintain support for multiple model backends through `OPENAI_BASE_URL` or equivalent secure endpoint settings.
- Use `[PLACEHOLDER: sovereign model id]` for G42-approved sovereign deployment and `[PLACEHOLDER: enterprise fallback model id]` for secondary routing if permitted.

### Prompt strategy
- Fixed system prompts define role, output schema, and safety boundaries.
- User inputs are inserted into bounded prompt fields rather than passed as arbitrary tool directives.
- Responses are required to be valid JSON and are rejected on malformed output.
- Prompt, model, and tool output hashes can be retained for post-hoc audit review.

### Tool-calling strategy
- Current implementation uses service-level connector orchestration, not freeform agent tool autonomy.
- This is favorable for enterprise control because external actions remain in deterministic services with explicit policy checks.
- If sovereign LLM tool use is later enabled, recommended production policy is allowlisted tools only, schema-typed parameters only, and a human approval boundary for any external write.

## Evaluation and Quality Approach

| Evaluation layer | Method |
| --- | --- |
| Functional validation | Workflow smoke tests and API endpoint validation |
| Benchmarking | Scenario harness for analyze, segment, propose, approve, execute, rollback, audit |
| Reliability | Retry logic, dead-letter handling, readiness checks, partial failure handling |
| Responsible AI | Bias scans, human review requirements, prompt constraints, residual risk documentation |
| Security | JWT validation, RBAC, audit logging, deployment secrets hygiene |
| Regression | CI smoke, pytest, OpenAPI generation, Playwright smoke |

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
| Networking | Private ingress, managed identity, firewall rules, private endpoints `[PLACEHOLDER if required]` |

### Sovereign LLM integration pattern
- Abstract model access behind a secure API adapter.
- Use mTLS or signed service-to-service auth `[PLACEHOLDER: exact control]`.
- Avoid direct model vendor coupling in business services.
- Log request metadata, not prompt bodies containing sensitive audience data.
- Preserve deterministic fallback paths if sovereign model is unavailable.

## Security Architecture

### AuthN and AuthZ

| Control | Implementation posture |
| --- | --- |
| Authentication | JWT-based Bearer auth in enterprise mode; OIDC discovery and JWKS validation |
| Authorization | RBAC by route and action lifecycle |
| Privileged execution | Separate roles for read, propose, approve, execute, and admin |
| Dev bypass | Available only for local development; must be disabled in staging and production |

### RBAC summary

| Capability | Roles |
| --- | --- |
| Read workflows and reports | `Admin`, `MarketingManager`, `Analyst`, `Auditor` |
| Propose actions | `Admin`, `MarketingManager` |
| Approve or reject actions | `Admin`, `MarketingManager` |
| Execute or rollback | `Admin`, `MarketingManager`, `Operator` |

### Encryption and secrets
- TLS in transit is required for all external and user-facing APIs.
- Secrets must be injected at runtime through a secret manager, not baked into images.
- At-rest encryption is expected through managed platform controls for PostgreSQL, storage, and secret stores.
- Exact cipher suite, key rotation frequency, and KMS ownership are deployment-specific and therefore `[PLACEHOLDER]`.

### Logging, retention, and PII controls

| Area | Control |
| --- | --- |
| Audit logs | Append-only event trail with chained hashes |
| Application telemetry | Request counts, latencies, readiness, and optional cloud telemetry |
| PII minimization | Use campaign ids and aggregates instead of raw personal records in workflows |
| CRM payload handling | Least-privilege tokens; avoid logging full CRM payloads |
| Retention | `[PLACEHOLDER: enterprise retention schedule]` |
| Data residency | `[PLACEHOLDER: sovereign hosting region and policy]` |

## Responsible AI Controls

### Bias testing plan

| Area | Test approach | Current posture |
| --- | --- | --- |
| Content outputs | Deterministic phrase scanning and human review | Implemented |
| Segmentation outputs | Rule-based screening and policy review | Implemented at MVP level |
| Protected class fairness | Formal fairness assessment | `[PLACEHOLDER: enterprise assessment process]` |
| Brand and regulatory review | Human approval before publication | Required |

### Mitigations
- Prefer deterministic logic for KPI-based recommendations.
- Restrict LLM usage to bounded content or narrative tasks.
- Validate JSON outputs and reject malformed responses.
- Fall back to templates when model output fails quality or policy checks.
- Keep humans in the loop for approvals, execution, and externally published content.

### Human oversight
- Human approvers remain accountable for business-impacting actions.
- Manual review is required for externally published copy.
- Final accountability remains with enterprise operators and leadership, not the model.

### Explainability
- Each recommendation is linked to business rationale, KPI context, confidence, and status.
- Audit events provide sequence traceability for who proposed, approved, executed, or rejected an action.
- Explainability artifacts are operational and reviewable even when generation components degrade.

## Deployment Topology for G42 Review

| Layer | Recommended enterprise pattern |
| --- | --- |
| Frontend | Controlled web client behind enterprise identity |
| API | Stateless container replicas separated from worker replicas |
| Worker | Isolated worker service for scheduled or queued workflows |
| Data | Managed PostgreSQL with tenant partitioning and backup policies |
| Secrets | Key Vault or sovereign secret service |
| Models | Secure sovereign LLM endpoint with enterprise API mediation |
| Telemetry | Centralized logging, metrics, distributed tracing, and alerting |

## Architecture Assessment
- Production-grade posture is supported by governance, observability, connector isolation, failure handling, and deployment artifacts.
- The architecture is intentionally bounded: it trades unrestricted autonomy for enterprise trust and auditability.
- The main submission gaps are deployment-specific evidence items such as named sovereign model ids, live customer references, and retention schedules.

## Assumptions
- This document separates currently evidenced implementation behavior from recommended G42 deployment patterns.
- Exact sovereign networking controls, retention schedules, model ids, and named production environments were not provided and are marked as `[PLACEHOLDER]`.
- Document revision: April 2026.
