# 05 Integration Capabilities

## Supported Platform Coverage

| Category | Platform | Current support | Primary methods | Decision-engine relevance |
| --- | --- | --- | --- | --- |
| CRM | HubSpot | Implemented | REST API, token-based auth | Supplies downstream quality, lifecycle, and lead-signal context |
| CRM | Salesforce | Adapter-ready pattern | REST API, OAuth2 | Can provide funnel-quality and revenue context without redesigning the decision flow |
| CRM | Microsoft Dynamics 365 | Adapter-ready pattern | REST API, OAuth2 | Supports enterprise CRM enrichment through the normalized connector contract |
| Marketing automation | HubSpot marketing workflows | Implemented through HubSpot integration | REST API, token-based auth | Supports lifecycle and campaign-context enrichment |
| Marketing automation | Marketo / Pardot / Braze / Iterable | Adapter-ready pattern | REST API, webhook, ETL | Extends campaign and audience context into the same decision layer |
| Analytics | Google Analytics 4 | Implemented | Data API, OAuth2 | Primary campaign performance and website-conversion signal source |
| Media | Google Ads | Implemented | GAQL, OAuth2, developer token | Supports both read-heavy decisioning and bounded governed action paths |
| Media | Meta Marketing API | Implemented optional | REST API, access token | Adds cross-channel media and audience evidence where enabled |
| Data warehouse / BI | Internal BI or warehouse | Adapter-ready pattern | API, ETL, batch import | Supports canonical metrics, historical replay, and enterprise data-plane alignment |
| CMS / content | Headless CMS platforms | Adapter-ready export pattern | REST API, webhook | Accepts bounded content or narrative outputs after review |
| Identity | Microsoft Entra ID | Implemented | OIDC, JWT, JWKS | Workforce identity path for enterprise deployment |
| Identity | Azure AD B2C | Implemented | MSAL, OIDC, JWT | External or consumer identity path where needed |
| Model endpoint | OpenAI-compatible or sovereign endpoint | Implemented integration pattern | Secure API adapter | Supports bounded narrative and content workloads without changing business services |

## Integration Methods

| Method | Usage |
| --- | --- |
| REST APIs | Primary integration mode for CRM, analytics, media, and content systems |
| OAuth2 and token-based auth | Used for Google and other external providers |
| JWT and OIDC | Used for enterprise identity and API protection |
| Webhooks and event triggers | Used for alerts, connector events, and workflow kickoff patterns |
| ETL / batch ingestion | Recommended for warehouses, historical backfills, and benchmark replay |
| OpenAPI contract | Primary machine-readable internal integration contract |
| Secure API mediation | Used to abstract sovereign or enterprise model endpoints from business services |

## Example Integration Patterns

### Pattern 1: Analytics + CRM + portfolio decisioning
1. Read campaign metrics from GA4 and media signals from Google Ads or Meta.
2. Read lifecycle or lead-quality signals from HubSpot or another CRM.
3. Normalize to account and campaign scope.
4. Compute health state, evidence sufficiency, business-state flags, and portfolio opportunity context.
5. Return ranked recommendations and decision packets through dashboard or API.

### Pattern 2: Governed budget reallocation and approval routing
1. Decision engine identifies `budget_shift`, `performance_cutback`, or `scale_winner` as the top eligible action.
2. Recommendation receives `decision_route`, `autonomy_tier`, and exception metadata.
3. If approval-ready, an approver reviews the canonical decision packet.
4. Authorized operator executes through the bounded connector path or controlled simulation.
5. Audit chain records rationale, route, approval, execution, and later outcome state.

### Pattern 3: Outcome measurement and recalibration loop
1. Recommendation and action lifecycle state are persisted after approval or execution.
2. Outcome records are linked back to the originating recommendation family.
3. Eligible history is summarized at campaign scope first, then account scope.
4. Future recommendations receive `recalibration_context` without bypassing evidence gates.

### Pattern 4: Bounded content and narrative assistance
1. Operator requests message variants, content support, or an executive readout.
2. LLM adapter generates strict-schema output with fallback controls.
3. Deterministic scans and human review apply where required.
4. Approved output is exported to downstream CMS, campaign, or reporting systems through standard API or webhook contracts.

## Enterprise Dependencies

| Dependency area | Requirement |
| --- | --- |
| Identity | Enterprise IdP issuing JWTs compatible with OIDC and JWKS |
| Secrets | Secret manager such as Key Vault or equivalent |
| Compute | Container runtime for API and optional worker separation |
| Persistence | PostgreSQL for durable workflows, recommendations, actions, outcomes, and audit events |
| Network | Secure outbound access to marketing platforms and inbound access to API/UI |
| Monitoring | Centralized logs, metrics, alerts, and trace storage |
| Canonical mapping and data quality | Entity mapping, source-confidence handling, and freshness controls across systems |
| Model endpoint | OpenAI-compatible or sovereign endpoint for bounded narrative and content workloads |

## Deployment Options

| Deployment option | Fit | Dependencies |
| --- | --- | --- |
| Azure Container Apps + managed Postgres | Best fit for Azure enterprise landing zone | Entra, Key Vault, Application Insights, Log Analytics, PostgreSQL |
| Kubernetes on sovereign infrastructure | Best fit for strict control or private hosting | Ingress, secret store, Postgres, observability stack, secure model gateway |
| VM or app-service style container hosting | Transitional option | Reverse proxy, secret injection, monitoring, database connectivity |
| Hybrid mode with external model endpoint | Best fit where data plane stays local and model plane is abstracted | Secure API mediation, outbound policy controls, audit logging |

## Integration Readiness by Evaluation Lens

| Evaluation lens | Readiness statement |
| --- | --- |
| CRM compatibility | Ready through HubSpot plus adapter-ready patterns for additional CRM systems |
| Marketing automation compatibility | Ready through HubSpot workflows and standardized automation integration patterns |
| Analytics compatibility | Ready through GA4, Google Ads, and optional Meta |
| CMS compatibility | Ready through structured export contracts and webhook or API integration patterns |
| Media buying compatibility | Ready through governed Google Ads path and policy-controlled multi-platform connector model |
| Enterprise identity compatibility | Ready for Entra and B2C-backed JWT flows |
| Sovereign model compatibility | Ready through secure API mediation and bounded model usage |

## Assumptions
- Integration statements distinguish implemented versus adapter-ready patterns as noted per platform.
- Customer-specific connector credentials, enterprise contracts, and sovereign networking approvals remain outside repository scope.
- Document revision: April 2026, aligned to the `submission` package `v1.2-repository-refresh` (campaign slice and mapping APIs complement connector data for cross-platform identity alignment).
