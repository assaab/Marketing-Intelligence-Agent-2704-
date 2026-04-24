# 01 Agent Resume

## Agent Identity

| Field | Value |
| --- | --- |
| Agent name | `Marketing Intelligence Agent` |
| Submission role alignment | G42 `Marketing Intelligence Agent`, Abu Dhabi, UAE, `reqId: 2704` |
| Agent category | Bounded marketing decision engine with governed execution support |
| Current maturity | Governed production-oriented backend with deterministic commercial ranking |
| Primary operating model | Connect -> discover -> analyze -> diagnose -> rank -> route -> approve or execute -> measure -> recalibrate -> learn |
| Execution posture | Controlled autonomy, approval-bound for material actions |
| Canonical decision artifacts | `decision_packet`, `decision_route`, `autonomy_tier`, `exception_codes`, `economic_priority_score` |
| Latest recorded verification snapshot | DB-verified workflow and campaign insight exports under `backend/exports/` (see `workflow_insights_bundle.md`, `campaign_insights_latest.md`); refresh after material pipeline changes |
| Public production evidence | API docs: `https://marcopsbackedncontainerapp.kindplant-7510bd3c.westeurope.azurecontainerapps.io/docs`; Production UI: `https://www.ai-companion.org/` |

## Executive Profile
`Marketing Intelligence Agent` is no longer best described as a campaign-analysis assistant that emits generic recommendations. In its current form it is a bounded marketing decision engine: it ingests campaign and CRM signals, builds typed health and evidence state, ranks bounded action families with deterministic economic logic, routes each recommendation into the correct autonomy lane, and records outcome data for later recalibration. The design goal is not unconstrained autonomy. The goal is commercially correct action under uncertainty with auditability, trust, and explicit human control for material changes.

## What The Agent Does Well Today
- Analyzes campaign health with typed KPI, source-level, and economic context
- Scores evidence sufficiency before suggesting action-ready commercial moves
- Detects business state, momentum, contradictions, and operating posture
- Generates bounded action families instead of open-ended suggestions
- Applies portfolio-aware opportunity-cost logic to recommendation ranking
- Produces canonical decision packets for approval and audit
- Distinguishes advisory outputs from approval-ready outputs through deterministic routing
- Learns from valid historical outcomes through bounded recalibration

## What The Agent Should Not Do
- Auto-promote material actions when economics are defaulted or incomplete
- Hide weak evidence behind confident language
- Let historical wins override current attribution or evidence failures
- Treat campaign-local growth as automatically better than portfolio reallocation
- Collapse contradictory signals into one optimistic action
- Expand autonomy beyond reversible, benchmark-proven low-risk actions

## Functional Coverage By User Need

| Capability area | Current behavior | Role relevance |
| --- | --- | --- |
| Research and planning | Builds research context, planning artifacts, state summaries, and test-first recommendations from campaign and portfolio context | Supports the strategy and planning expectations of the G42 role |
| Creative and media strategy | Emits bounded action families such as `budget_shift`, `scale_winner`, `performance_cutback`, `creative_refresh`, `targeting_refine`, and `data_collection` | Turns diagnosis into commercially relevant next steps |
| Content production | Supports bounded LLM-assisted copy and narrative generation with schema validation and fallback | Covers content support without giving the model policy authority |
| Media management | Routes action proposals through recommendation, approval, execution, rollback, and audit paths | Supports governed operational execution rather than raw recommendation output |
| Measurement and analytics | Produces KPI snapshots, source breakdowns, confidence penalties, and economic context | Gives operators a typed and explainable view of current performance |
| Targeted marketing and audience segmentation | Uses CRM quality and audience signals to support targeting and segment refinement; sensitive cases escalate | Addresses targeted marketing while preserving safety controls |
| Governance and decision support | Exposes `decision_packet`, `decision_route`, `autonomy_tier`, `exception_codes`, trust notes, and portfolio context | Gives managers and auditors a reviewable commercial decision artifact |

## Current End-to-End Flow

| Stage | What the agent does | Primary outputs |
| --- | --- | --- |
| Connect and discover | Accepts authenticated users and connector events, then discovers account and campaign scope | Workspace context, account catalog, monitored campaign set |
| Analyze and build health state | Normalizes campaign metrics and constructs typed health state | KPI snapshot, source breakdown, economic snapshot, connector error context |
| Score data quality and evidence sufficiency | Computes sample sufficiency, attribution risk, evidence completeness, and degraded-insight posture | Confidence penalties, evidence status, safe-action posture |
| Detect business state and resolve contradictions | Determines absolute health tier, momentum tier, intervention posture, and contradiction notes | Business-state flags, posture, contradiction resolution notes |
| Generate bounded candidate actions | Produces only allowed action families for the resolved state | Candidate action set with rationale and guardrail alignment |
| Apply portfolio and state-aware ranking | Scores each candidate using deterministic economic and policy inputs | `economic_priority_score`, score breakdown, portfolio context |
| Produce recommendation and decision packet | Builds commercial recommendation plus audit-ready review artifact | Recommendation body, trust notes, `decision_packet`, `decision_grade`, routing metadata |
| Approval and execution routing | Routes each outcome into observe, recommend, approval, micro-action, or escalation lanes | `decision_route`, `autonomy_tier`, `exception_codes`, actionability status |
| Outcome measurement and learning | Records recommendation outcomes and feeds bounded history into future ranking | Outcome memory, recalibration context, improved future priors |

## Core Decision Inputs And Outputs

| Category | Current inputs or outputs |
| --- | --- |
| Core decision inputs | Campaign KPI snapshot, source-level and slice-level evidence, sample sufficiency, attribution risk, customer economics resolution, account portfolio alternatives, business-state flags, historical priors, validated outcome history |
| Core decision outputs | Ranked recommendations, canonical decision packets, actionability and routing status, approval-ready vs advisory differentiation, portfolio recommendation briefs, measured outcome linkage |
| Business-state outputs | `absolute_health_tier`, `momentum_tier`, `intervention_posture`, `business_state_flags`, contradiction notes |
| Portfolio outputs | `portfolio_opportunity_context`, `reallocation_pressure`, opportunity-gap signals, enriched top-recommendation briefs |
| Learning outputs | `recalibration_context`, bounded confidence adjustments, valid-outcome counts, campaign- and account-scope priors |

## Decision Routing And Autonomy

| Decision route | Implemented autonomy tier | Meaning | Current posture |
| --- | --- | --- | --- |
| `observe_only` | `tier_0_observe` | Informational only, usually due to weak evidence or contradiction | Active |
| `recommend_only` | `tier_1_recommend` | Useful recommendation but not safe enough for approval routing | Active |
| `propose_for_approval` | `tier_2_approval` | Decision-ready commercial move routed to a human approver | Active |
| `execute_micro_action` | `tier_3_micro_action` | Reserved for low-risk, reversible, benchmark-proven playbooks | Intentionally narrow |
| `escalate` | `tier_0_observe` | Blocked or outside authority due to policy, data, or business constraints | Active |

- Deterministic downgrades and escalations are triggered by conditions such as defaulted economics, degraded insight, attribution risk, missing slice evidence, contradiction bundles, consent or compliance sensitivity, and high downside.
- Final routing happens after data-quality scoring and contradiction resolution, not before.
- Historical recalibration can influence score and confidence, but it does not override evidence gates or approval policy.

## Production Deployment Evidence

| Area | Evidence |
| --- | --- |
| Application architecture | Next.js frontend, FastAPI API, optional background worker, optional PostgreSQL |
| Decision-engine implementation | Deterministic recommendation, scoring, portfolio, calibration, and decision-packet services are present in the backend |
| Azure enterprise baseline | Bicep assets for Container Apps, ACR, Key Vault, Log Analytics, Application Insights, and PostgreSQL Flexible Server |
| Public production API evidence | Live FastAPI Swagger/OpenAPI endpoint at `https://marcopsbackedncontainerapp.kindplant-7510bd3c.westeurope.azurecontainerapps.io/docs` |
| Public production UI evidence | Live frontend at `https://www.ai-companion.org/` |
| Identity | Microsoft Entra ID and Azure AD B2C compatible token validation paths |
| Monitoring | `GET /metrics`, `GET /ready`, request latency metrics, and Application Insights integration |
| Reliability controls | Retry logic, partial-failure handling, queue processing, approval gating, and degraded-mode behavior |
| Persistent operations | Workflow runs, actions, audit events, reports, and outcome records can be stored durably |

## Key Backend Components

| Component | Purpose |
| --- | --- |
| `backend/app/services/recommendations.py` | Candidate generation, scoring inputs, ranking flow, and recommendation assembly |
| `backend/app/services/portfolio_context.py` | Account-level opportunity-cost and reallocation context |
| `backend/app/services/campaign_business_state.py` | Business-state resolution and flag derivation |
| `backend/app/services/recommendation_scoring.py` | Deterministic economic and confidence-aware scoring primitives |
| `backend/app/services/calibration.py` | Historical outcome-based recalibration context |
| `backend/app/services/decision_packet.py` | Decision routing, autonomy classification, exception codes, and canonical packet assembly |
| `backend/app/repository.py` | Persistence and portfolio summary projection |
| `backend/app/services/dashboard_bootstrap.py` | Dashboard-level recommendation summary and portfolio brief output |
| `backend/app/services/campaign_mapping_service.py` | Campaign slice and external-id alias mapping for cross-system reconciliation |
| `backend/app/routers/campaign_mappings.py` | REST surface for mapping CRUD aligned to tenant RBAC |
| `backend/app/services/business_inputs_validation.py` | Hardened validation for business inputs used in runtime scoring |
| `backend/app/services/research_evidence_gate.py`, `research_retrieval_policy.py`, `research_internal_context.py` | Evidence gating, retrieval policy, and internal context assembly for research flows |
| `backend/app/services/slice_ingestion.py`, `creative_ingestion.py` | Ingestion paths for performance slices and creative assets feeding analysis |

## Verification Snapshot
- Authoritative packaged evidence for reviewers: `backend/exports/workflow_insights_bundle.md` and `backend/exports/campaign_insights_latest.md` (generated from latest `analyze_campaign` rows; 10/10 seeded campaigns verified in the current export window).
- Supplementary bundles: `full_campaign_analysis_bundle.*`, `campaign_agentic_debug.*` in the same folder.
- For structural or API contract changes, refresh the repo-local Code Review Graph at `.code-review-graph` (see root `AGENTS.md` and `docs/repo-agent-guide.md`).
- Benchmark scripts for fresh capture remain in `scripts/benchmark_report.py` and `scripts/benchmark_smoke.py`.

## Why This Agent Fits The Role
- It covers the functional span expected by G42: planning, media strategy, content support, analytics, segmentation, and governed media operations.
- It is built around commercial decision correctness under uncertainty rather than free-form AI output.
- It provides reviewable, auditable decision packets instead of opaque recommendations.
- It supports cloud-agnostic deployment, Azure enterprise compatibility, and secure sovereign model integration patterns.
- It is explicitly bounded: the system is designed to be trusted in enterprise environments, not to maximize autonomy at the expense of control.

## Document revision
- April 2026, aligned to the `submission` package `v1.2-repository-refresh` (public UI `https://www.ai-companion.org/`, exports under `backend/exports/`).
