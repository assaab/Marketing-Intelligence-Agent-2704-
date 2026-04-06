# 04 Performance Benchmarks

## Benchmark Positioning
This document defines a production-grade benchmark package for submission. It is based on executable benchmark scripts in the repository, measurable API behavior, reliability controls, and auditable evidence artifacts.

## Production Evidence Inputs

| Evidence input | Source |
| --- | --- |
| API benchmark capture | `scripts/benchmark_report.py` |
| API smoke benchmark | `scripts/benchmark_smoke.py` |
| Live API endpoint | `https://marcopsbackedncontainerapp.kindplant-7510bd3c.westeurope.azurecontainerapps.io/docs` |
| Benchmark artifacts output | `artifacts/benchmarks/benchmark-report.md`, `artifacts/benchmarks/captured-benchmark-evidence.json` |
| Readiness and metrics endpoints | `GET /ready`, `GET /metrics` |
| Runtime connector mode verification | `GET /runtime/config` |

## Submission Runbook (What To Do)

### Step 1: Run deterministic baseline benchmark
Execute from repository root:

`python scripts/benchmark_report.py`

Expected output:
- `artifacts/benchmarks/benchmark-report.md`
- `artifacts/benchmarks/captured-benchmark-evidence.json`
- `artifacts/benchmarks/captured-run-latest.json`

### Step 2: Run smoke benchmark for endpoint sanity
Execute:

`python scripts/benchmark_smoke.py`

Use this to prove:
- analyze and plan workflow endpoints return successfully,
- metrics endpoint is reachable,
- baseline latencies are captured.

### Step 3: Capture live connector benchmark run
Use live connectors and rerun benchmark capture:

`USE_REAL_CONNECTORS=true python scripts/benchmark_report.py`

This run should show connector mode from `GET /runtime/config` and include connector error behavior in evidence.

### Step 4: Produce benchmark evidence bundle
Attach these files to submission evidence:
- `artifacts/benchmarks/benchmark-report.md`
- `artifacts/benchmarks/captured-benchmark-evidence.json`
- `artifacts/benchmarks/captured-run-latest.json`
- One screenshot from live Swagger endpoint and one screenshot from production frontend during benchmark window.

## Scenario Benchmark Table

| Scenario | Workload definition | Latency target / evidence | Throughput target / evidence | Reliability target / evidence | Cost metric | Accuracy / quality metric |
| --- | --- | --- | --- | --- | --- | --- |
| Analyze campaign | Single campaign analysis with connector reads, KPI normalization, health scoring, and recommendations | Target `P95 < 30s`; evidence from `captured-benchmark-evidence.json` latency samples | Derived from benchmark rounds and worker configuration in captured run | Non-write workflow completion target `>= 99.5%` with per-run HTTP success rate evidence | Derived from run duration and infrastructure pricing model | False-positive anomaly rate target `< 15%` with review sample log |
| Generate report | Executive or campaign report generation from normalized metrics | Target `P95 < 90s`; measured in report benchmark session | Derived from sustained run profile in benchmark artifacts | Non-write workflow completion target `>= 99.5%` | Derived from compute plus model usage per run | Report factual correctness target `> 95%` via deterministic and human review checks |
| Recommend optimizations | KPI-driven optimization set with recommendation ranking and approval flags | Target `P95 < 30s`; measured from optimization workflow captures | Derived from recommendation throughput per benchmark window | Non-write workflow completion target `>= 99.5%` | Low inference cost when deterministic path is used | Recommendation acceptance target `> 60%` tracked in action lifecycle analytics |
| Generate content | LLM-assisted content variant generation with schema validation and phrase scans | Measured from content workflow benchmark runs and API timing logs | Derived from content job completion counts per benchmark window | Fallback path active on model or JSON failure with continuity preserved | Derived from token usage and generation volume | Must pass deterministic phrase scans and editorial acceptance |
| Segment audience | Segment proposal using CRM quality signals and campaign context | Measured from segmentation workflow benchmark runs | Derived from segmentation job completion counts per window | Non-write workflow completion target `>= 99.5%` | Derived from batch execution profile | Zero blocked bias-scan violations in release candidate |
| Propose, approve, execute low-risk action | Full governed action lifecycle including audit | Simulated execute success target `>= 95%` | Derived from governed action queue throughput | State preconditions enforced and audit completeness target `100%` | Derived from executed action cost profile | Approval trace and execution trace recorded for each action |
| Audit retrieval | Retrieve audit history for workflow or campaign | Measured from audit query timing captures | Derived from query throughput in benchmark window | Audit completeness target `100%` with chain verification | Minimal | Traceability and chain verification pass |

## Operational Outcome Metrics

| Outcome class | Evidence type | Current status |
| --- | --- | --- |
| Campaign efficiency improvement | Before/after KPI deltas from benchmarked optimization cycles | Ready for capture in benchmark evidence pack |
| Time saved in reporting or optimization workflow | Measured workflow completion time before/after benchmarked automation | Ready for capture in benchmark evidence pack |
| Reduction in manual approval handling time | Approval queue timing analytics | Ready for capture in governed-action benchmark run |
| Improved action traceability | Audit completeness and state transition verification | Evidenced by runtime design and benchmark traces |
| Reduced failed workflow rate under dependency instability | Reliability benchmark with induced connector failure modes | Evidenced through retry, fallback, dead-letter, and recovery behavior |

## Known Failure Modes and Mitigations

| Failure mode | Typical symptom | Mitigation | Residual risk |
| --- | --- | --- | --- |
| Connector timeout or external API error | Partial metrics, longer latency, connector error payloads | Retry with bounded exponential backoff; continue with partial result when safe | Incomplete recommendations if source coverage is reduced |
| Invalid OAuth or expired token | Connector returns auth error | Operator rotates token or reauthorizes integration | Live mode reliability depends on external credential hygiene |
| LLM timeout, invalid JSON, or schema mismatch | Content generation fails or degrades | Fall back to templates; preserve workflow continuity | Lower content quality during fallback mode |
| Bias phrase scan hit | Generated text or research output blocked | Substitute template or seeded content; route to human review | Does not replace full fairness audit |
| HITL pipeline not resumed | Workflow remains paused | Resume through operator action or runbook | SLA impact if approvals are not staffed |
| Postgres unavailable in persistence mode | Readiness failure, durable workflows blocked | Restore database, fail readiness, preserve deployment hygiene | Availability tied to data platform recovery |
| Background job retries exhausted | Job reaches dead-letter state | Inspect `last_error`, fix root cause, re-queue | Manual intervention required for some failure classes |
| Invalid action state transition | Execute or rollback rejected | Enforce lifecycle ordering | User training and UX cues still required |

## Stress Testing Approach

### Test tiers

| Tier | Goal | Proposed load profile |
| --- | --- | --- |
| Smoke | Validate core route health and regression | Single-user run across analyze, report, actions, audit |
| Concurrency | Validate API stability under normal team traffic | `25`, `50`, and `100` concurrent workflow requests |
| Soak | Validate resource leaks and queue stability | `8`, `12`, and `24` hour sustained mixed workflow load |
| Failure injection | Validate degradation behavior | Expired token, upstream timeout, LLM error, DB failover drill |
| Approval queue test | Validate HITL behavior under backlog | Burst of proposed actions awaiting approval |

### Stress test success criteria
- No data corruption in workflow, action, or audit records
- Readiness and metrics remain available during recoverable upstream failures
- Non-write workflows remain at or above `99.5%` completion in supported conditions
- Dead-letter and retry behavior are observable and bounded
- No high-risk action bypasses approval or policy enforcement

## Benchmark Governance Notes
- Benchmark evidence should be regenerated after material workflow, connector, or prompt changes.
- Benchmarks should be run in both simulated connector mode and live connector mode.
- Simulated mode should remain deterministic for baseline regression comparison.
- Live mode should include connector error reporting, not only pass / fail scoring.
- Responsible AI release gating should require bias scan pass plus human review sign-off for externally published content.

## Lifecycle Maturity and Version History

| Maturity signal | Evidence in workspace | Submission value |
| --- | --- | --- |
| Production-oriented MVP | Architecture, governance, and deployment docs are present | Shows system is beyond prototype stage |
| Connector hardening | Production connector architecture and rollout docs exist | Demonstrates enterprise integration maturity |
| Optional durable persistence | PostgreSQL mode plus job queue and audit persistence | Supports repeatable operations and recovery |
| Agent runtime evolution | Legacy flow plus optional LangGraph mode | Shows lifecycle progression and extensibility |
| CI and benchmark hooks | Security scan, tests, smoke checks, benchmark smoke | Supports continuous validation |

## Version Register

| Version / release | Maturity note | Evidence | Status |
| --- | --- | --- | --- |
| `v0.x` | Initial governed marketing workflows | Core orchestration and audit design | Established |
| `v0.x` | Connector hardening and reliability controls | Connector architecture and rollout docs | Established |
| `v1.0-candidate` | G42 submission package baseline | Submission package plus benchmark evidence artifacts | Ready |

## Assumptions
- Production benchmark quality requires fresh evidence capture in the target environment immediately before submission.
- Final submission should include the generated benchmark artifacts and live endpoint proof captured in the same benchmark window.
- Document revision: April 2026.
