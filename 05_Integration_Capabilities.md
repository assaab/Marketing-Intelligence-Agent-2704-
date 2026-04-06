# 05 Integration Capabilities

## Supported Platform Coverage

| Category | Platform | Current support | Primary methods | Notes |
| --- | --- | --- | --- | --- |
| CRM | HubSpot | Implemented | REST API, token-based auth | Used for segment quality and CRM signals |
| CRM | Salesforce | Production-ready adapter pattern | API extension pattern | Connector contract allows enterprise adapter onboarding without workflow redesign |
| CRM | Microsoft Dynamics 365 | Production-ready adapter pattern | REST API, OAuth2 | Connector abstraction supports standards-based integration |
| Marketing automation | HubSpot marketing workflows | Implemented through HubSpot integration | REST API, token-based auth | Supports campaign and lifecycle data enrichment flows |
| Marketing automation | Marketo / Pardot / Braze / Iterable | Production-ready adapter pattern | REST API, webhook, ETL | Supported through normalized connector interface and event ingestion pattern |
| Analytics | Google Analytics 4 | Implemented | Data API, OAuth2 | Primary campaign metrics source |
| Analytics | Google Ads | Implemented | GAQL, OAuth2, developer token | Real reads supported; writes remain bounded |
| Analytics | Meta Marketing API | Implemented optional | REST API, access token | Enabled by flag |
| Analytics | Internal BI / warehouse | Production-ready adapter pattern | API, ETL, batch import | Uses normalized payload adapter pattern for enterprise data plane alignment |
| CMS / content | Headless CMS platforms | Production-ready export integration pattern | REST API, webhook | Structured content output is compatible with downstream CMS ingestion contracts |
| Media buying | Google Ads | Implemented | API read and governed write path | Live mutation support depends on deployment mode and approval |
| Media buying | Meta Ads | Implemented optional | API | Read path is active and can be governed under approval-controlled execution policy |
| Identity | Microsoft Entra ID | Implemented | OIDC, JWT, JWKS | Workforce identity path |
| Identity | Azure AD B2C | Implemented | MSAL, OIDC, JWT | External or consumer identity path |

## Integration Methods

| Method | Usage |
| --- | --- |
| REST APIs | Primary integration mode for CRM, analytics, media, and content systems |
| OAuth2 and token-based auth | Used for Google and other external providers |
| JWT and OIDC | Used for enterprise identity and API protection |
| Webhooks | Available for alerts and event-driven integration patterns across enterprise systems |
| ETL / batch ingestion | Recommended for warehouses, historical backfills, and bulk campaign normalization |
| OpenAPI contract | Primary machine-readable internal integration contract |

## Example Integration Patterns

### Pattern 1: Analytics + CRM enrichment
1. Read campaign metrics from GA4 and media spend from Google Ads or Meta.
2. Read segment quality or lead outcome signals from HubSpot.
3. Normalize to account and campaign scope.
4. Compute health, recommendations, and audience refinement signals.
5. Return results through dashboard or API.

### Pattern 2: Governed media optimization
1. Recommendation engine identifies a low-risk action.
2. Action is proposed and persisted.
3. Human approver accepts or rejects.
4. Authorized operator executes through bounded connector path.
5. Audit chain records rationale, decision, and execution state.

### Pattern 3: Content and launch preparation
1. Operator requests messaging variants for a campaign objective.
2. LLM adapter generates structured copy variants with fallback controls.
3. Segment and channel recommendations are paired with content options.
4. Approved output is exported to downstream content or campaign systems using standard API or webhook contracts.

### Pattern 4: Enterprise warehouse or CDP augmentation
1. Historical campaign performance is loaded through ETL or secure APIs.
2. Agent consumes normalized metrics rather than direct platform calls where required.
3. Output is written back to analytics dashboards, planning tools, or approval work queues.

## Enterprise Dependencies

| Dependency area | Requirement |
| --- | --- |
| Identity | Enterprise IdP issuing JWTs compatible with OIDC and JWKS |
| Secrets | Secret manager such as Key Vault or equivalent |
| Compute | Container runtime for API and optional worker separation |
| Persistence | PostgreSQL for durable workflows, actions, recommendations, and audit events |
| Network | Secure outbound access to marketing platforms and inbound access to API/UI |
| Monitoring | Centralized logs, metrics, alerts, and trace storage |
| Model endpoint | OpenAI-compatible or sovereign LLM endpoint for generative workloads |

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
| CRM compatibility | Ready through HubSpot and enterprise adapter framework for additional CRM systems |
| Marketing automation compatibility | Ready through HubSpot workflows and standardized automation integration pattern |
| Analytics compatibility | Ready through GA4, Google Ads, and optional Meta |
| CMS compatibility | Ready through structured content export contracts and webhook/API integration model |
| Media buying compatibility | Ready through governed Google Ads path and policy-controlled multi-platform connector model |
| Enterprise identity compatibility | Ready for Entra and B2C-backed JWT flows |

## Assumptions
- Integration statements describe implemented or adapter-ready patterns as noted per platform; customer-specific connector credentials and contracts are outside repository scope.
- Document revision: April 2026.
