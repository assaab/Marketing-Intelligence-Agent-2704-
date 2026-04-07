# 04 Performance Benchmarks

## Benchmark Positioning
This benchmark package is designed to prove more than latency or endpoint health. The updated system must demonstrate that it chooses the right commercial action consistently enough, suppresses unsafe actioning when evidence is weak, and preserves reliability, explainability, and governance under real operating conditions.

## Production Evidence Inputs

| Evidence input | Source |
| --- | --- |
| Backend decision-routing regression tests | `backend/tests/test_decision_routing.py` |
| Recommendation tranche-two tests | `backend/tests/test_recommendation_tranche_two.py` |
| API projection and portfolio summary tests | `backend/tests/test_api.py` |
| Pilot proof and decision-card tests | `backend/tests/test_pilot_proof_flow.py` |
| API benchmark capture | `scripts/benchmark_report.py` |
| API smoke benchmark | `scripts/benchmark_smoke.py` |
| Benchmark artifacts output | `artifacts/benchmarks/benchmark-report.md`, `artifacts/benchmarks/captured-benchmark-evidence.json` |
| Runtime health and telemetry | `GET /ready`, `GET /metrics` |
| Public deployment evidence | `https://marcopsbackedncontainerapp.kindplant-7510bd3c.westeurope.azurecontainerapps.io/docs` |

## Submission Runbook (What To Do)

### Step 1: Run decision-engine regression tests
Execute from repository root:

`cd backend && pytest tests/test_decision_routing.py tests/test_recommendation_tranche_two.py tests/test_api.py tests/test_pilot_proof_flow.py`

Use this step to prove that routing, portfolio summaries, decision packets, and API projections remain intact after material changes.

### Step 2: Run deterministic baseline benchmark
Execute:

`python scripts/benchmark_report.py`

Expected outputs:
- `artifacts/benchmarks/benchmark-report.md`
- `artifacts/benchmarks/captured-benchmark-evidence.json`
- `artifacts/benchmarks/captured-run-latest.json`

### Step 3: Run smoke benchmark for endpoint sanity
Execute:

`python scripts/benchmark_smoke.py`

Use this to prove:
- core endpoints return successfully,
- metrics and readiness endpoints are reachable,
- baseline latency samples are captured.

### Step 4: Capture live connector benchmark run
Execute:

`USE_REAL_CONNECTORS=true python scripts/benchmark_report.py`

This run should include connector mode evidence, degraded-behavior evidence, and connector error reporting where applicable.

### Step 5: Produce benchmark evidence bundle
Attach these files to submission evidence:
- `artifacts/benchmarks/benchmark-report.md`
- `artifacts/benchmarks/captured-benchmark-evidence.json`
- `artifacts/benchmarks/captured-run-latest.json`
- one screenshot from the live Swagger endpoint
- one screenshot from the production frontend during the benchmark window

## Decision-Quality Benchmark Table

| Scenario / business state | Expected behavior | What to validate | Evidence source |
| --- | --- | --- | --- |
| Underperforming campaign | Diagnose driver before acting; prefer contained optimization or investigation over blind cuts | Top-ranked action is commercially plausible and supported by evidence | Scenario benchmark set plus recommendation tests |
| Overspending campaign | Contain downside proportionally with explicit approval routing | Material spend moves require approval-ready evidence and downside framing | Routing tests plus scenario replay |
| Limited data or cold start | Prefer observe, recommend-only, or data-collection actions | Sparse-data cases do not become approval-ready commercial actions | Data-quality and routing behavior in regression suite |
| Conflicting signals across sources | Surface conflict instead of averaging it away | Contradictions downgrade actionability or escalate | Business-state and contradiction handling scenarios |
| Attribution uncertainty | Favor measurement hardening, test-first, or escalation | High attribution risk reduces direct action confidence | Scenario replay plus exception-code inspection |
| Budget reallocation opportunity | Compare local optimization against account alternatives | Portfolio pressure and opportunity-gap signals influence ranking | `test_recommendation_tranche_two.py` and API projection tests |
| Lead-quality deterioration | Use downstream quality signals, not only top-of-funnel volume | Recommendation reflects CRM quality decline and state flags | Tranche-two recommendation scenarios |
| Stable high-performing campaign | Prefer protect-value behavior over unnecessary churn | Correct decision may be do-nothing or monitor-only | Business-state benchmark scenarios |
| Compliance-sensitive or sensitive-segment action | Escalate to human review | Unsafe targeting does not become autonomous or approval-ready by default | Policy tests, RAI review, and manual benchmark review |

## Operational Metrics

| Metric class | Why it matters | Evidence status |
| --- | --- | --- |
| Top-ranked action correctness | Measures whether the best action is usually the right one | Ready for scenario and replay capture |
| Correct safe-suppression rate | Measures whether the system chooses observe, test-first, or escalate when evidence is weak | Ready for routing and benchmark capture |
| Positive-ROI rate for accepted actions | Measures economic value after approval and execution | Ready for outcome-linked capture |
| Regret versus best alternative | Measures whether ranking missed a better action in the same scope | Ready for portfolio benchmark analysis |
| Confidence calibration | Measures whether predicted confidence tracks realized success | Ready for outcome and recalibration reporting |
| Decision-packet generation success | Measures whether review artifacts are produced reliably | Supported by API and regression tests |
| Recommendation latency | Measures time to decision-ready output | Captured by benchmark scripts and telemetry endpoints |
| Freshness compliance rate | Measures how often action-eligible decisions meet freshness thresholds | Ready for capture in benchmark evidence |
| Approval and rollback success | Measures operational safety after decision selection | Ready for governed action lifecycle capture |
| Audit completeness | Measures whether evidence, route, and outcome linkage are reconstructable | Supported by runtime design and audit traces |
| Time-to-decision improvement | Measures analyst and manager efficiency | Ready for capture in benchmark evidence bundle |

## Verification Snapshot
- Targeted tranche-two ranking, routing, API, and portfolio tests passed
- Adjacent pipeline and portfolio-parallel tests passed
- Latest recorded backend suite result: `156 passed, 2 skipped`
- Recommendation briefs in portfolio and dashboard outputs now expose enriched decision metadata such as `economic_priority_score`, `decision_route`, and `business_state_flags`

## Known Failure Modes And Mitigations

| Failure mode | Why it happens | Mitigation strategy | Residual risk |
| --- | --- | --- | --- |
| False precision under sparse data | Weak samples are easy to narrate too confidently | Downgrade to observe-only, recommend-only, or data-collection posture | Conservative routing may suppress some true positives |
| Source mismatch or identity drift | CRM, analytics, and ad entities do not reconcile cleanly | Use source-confidence scoring, contradiction handling, and escalation | Some enterprise data conflicts still require human resolution |
| LLM rationalization of weak decisions | Model output can sound stronger than the evidence | Keep final ranking and routing deterministic; use LLMs only for bounded synthesis | Narrative quality can still vary during fallback mode |
| KPI-local but business-negative actions | Local lift may hide downstream profitability harm | Use economic inputs, CRM quality, and portfolio comparisons in ranking | Missing economics still limits commercial precision |
| Wrong autonomy lane | Risk is misclassified or evidence is overstated | Use explicit routes, autonomy tiers, exception codes, and approval boundaries | Tier-3 micro-action expansion must remain benchmark-gated |
| Hidden partial-data decisions | Connector or source failure leaves incomplete proof | Separate degraded insight mode from approval-ready decision mode | Partial data can still reduce useful coverage |
| Compliance or bias failure in segments | Audience logic touches sensitive or restricted cases | Enforce sensitive-segment escalation and human review | Formal fairness review remains an external governance requirement |
| Action duplication or thrash | Repeated actions or repeated proposals cause churn | Use action history, state transitions, and cooldown-aware workflows | Operator training and UX cues still matter |
| Portfolio cannibalization | Local optimization ignores better alternatives elsewhere | Include opportunity-cost logic and reallocation pressure in ranking | Full cross-objective optimization remains future work |
| Outcome-learning contamination | Poor attribution or incomplete observation windows distort learning | Gate learning eligibility and bound recalibration strength | Historical priors remain advisory, not authoritative |
| Audit incompleteness | Evidence and approvals are fragmented | Use one canonical decision packet linked to audit and outcome records | External systems still need consistent attachment of screenshots and reports |

## Stress Testing Approach

### Test tiers

| Tier | Goal | Proposed load profile |
| --- | --- | --- |
| Smoke | Validate core route health and regression | Single-user run across analyze, summary, routing, audit |
| Decision replay | Validate business-state and ranking correctness | Replay benchmark scenarios covering underperformance, sparse data, attribution conflict, and reallocation |
| Concurrency | Validate API stability under normal team traffic | `25`, `50`, and `100` concurrent decision requests |
| Soak | Validate resource leaks and queue stability | `8`, `12`, and `24` hour sustained mixed workflow load |
| Failure injection | Validate degradation behavior | Expired token, upstream timeout, LLM error, DB failover drill |
| Approval backlog test | Validate human-in-the-loop behavior under queue pressure | Burst of approval-ready actions awaiting review |

### Stress test success criteria
- No data corruption in workflow, action, or audit records
- Decision routing remains bounded under sparse data, connector failure, or attribution conflict
- No material action bypasses approval or policy enforcement
- Readiness and metrics remain available during recoverable upstream failures
- Recalibration remains bounded and does not override evidence gates

## Benchmark Governance Notes
- Regenerate benchmark evidence after material changes to recommendation logic, routing, connector contracts, or prompts.
- Run benchmarks in both deterministic simulated mode and live connector mode.
- Evaluate decision correctness and safe suppression, not only recommendation frequency.
- Treat `do nothing`, `test first`, and `escalate` as valid successful outcomes when evidence warrants them.
- Expand autonomy only when low-downside micro-actions are benchmark-proven and rollback is reliable.

## Lifecycle Maturity And Version History

| Maturity signal | Evidence in workspace | Submission value |
| --- | --- | --- |
| Governed marketing workflow baseline | Architecture, audit, approvals, and connector orchestration are present | Shows the system is beyond a prototype-only assistant |
| Decision-grade recommendation layer | Routing, exception codes, decision packets, portfolio context, and recalibration are implemented | Shows movement toward production-grade commercial decisioning |
| Benchmark and verification hooks | Regression tests, benchmark scripts, smoke scripts, and telemetry endpoints exist | Supports empirical review rather than narrative claims only |

## Version Register

| Version / release | Maturity note | Evidence | Status |
| --- | --- | --- | --- |
| `v0.x` | Initial governed marketing workflows | Core orchestration, approvals, and audit design | Established |
| `v1.0` | Decision-grade recommendation upgrades | Routing, exception codes, decision packets, portfolio context, recalibration | Established |
| `v1.1-decision-engine` | Submission package refresh aligned to G42 review | Updated submission pack and benchmark framing | Current submission baseline |

## Assumptions
- Production benchmark quality still requires fresh evidence capture in the target environment immediately before submission.
- Final submission should include regenerated benchmark artifacts and live endpoint proof captured in the same benchmark window.
- Document revision: April 2026, aligned to the `submission` package `v1.1-decision-engine`.
