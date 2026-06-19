# Product Maturity — Four Axes

> Level 1 is the **minimum viable baseline** — what must be in place to move from chaos to a measurable starting point.
> Teams with none of these in place are at **Level 0 (below the floor)** and their first investments should target the L1 baseline.

## The Four Axes

The maturity framework evaluates a product team across four axes. Three are about what the team **does** (inputs and processes); the fourth measures what the team **achieves** (outcomes in production).

| Axis | What it measures | Type |
|---|---|---|
| **Testing** | Test types, automation, management, pipelines | Input / process |
| **Performance** | Load, stress, resilience, security testing | Input / process |
| **Observability** | Logs, metrics, traces, alerts | Input / process |
| **Production Behaviour** | Real outcomes — incidents, recovery, user impact | **Outcome** |

The Production Behaviour axis is what reveals whether the other three axes are actually working. A team can score high on Testing, Performance, and Observability but still score low on Production Behaviour — that gap is the most important signal in the framework. It means the team has invested in machinery but the machinery isn't preventing production issues.

## The Five Levels

| Level | Name | Gist |
|---|---|---|
| **0** | Below the Floor | Nothing in place. First target the L1 baseline. |
| **1** | Initial | Foundation established |
| **2** | Managed | Visible & repeatable |
| **3** | Defined | Measured & governed |
| **4** | Quantitative | Data-driven |
| **5** | Optimizing | Predictive & self-healing |

---

## Level 1 — Foundation Established

Minimum viable baseline. Cheap, fast to set up (days to weeks). Once in place, the team is measurable and has something to build on.

### Testing
- Unit tests exist for core logic
- CI pipeline runs unit tests on every PR
- Static code analysis runs on commit
- Test cases tracked (even informally — wiki, spreadsheet, or test tool)

### Performance
- SAST scan runs on every PR
- Dependency vulnerability scan runs weekly (e.g. Snyk, Dependabot)
- Known critical CVEs tracked in a backlog (not necessarily fixed yet, but visible)

### Observability
- Services emit structured logs (JSON format)
- Health check endpoint exists per service (`GET /health` returns 200)
- Logs are accessible (even via `kubectl logs` is acceptable at this level)

### Production Behaviour
- Incidents are logged consistently — one ticket per incident
- Severity classified at logging time (P1/P2/P3)
- Post-incident review held for P1 incidents (lightweight; even a Slack thread counts)

---

## Level 2 — Visible & Repeatable

The practices from L1 are now centralised, automated, and enforced. Quality gates start blocking bad changes from reaching production.

### Testing
- Unit + component tests in CI with mocked dependencies
- Code coverage measured and gated in CI (e.g. ≥ 40%)
- Static analysis blocks merge on quality gate failure
- Test cases catalogued in qTest (or equivalent test management tool)

### Performance
- Performance smoke tests on key endpoints (latency thresholds)
- Dependency vulnerability scan blocks merge on critical CVEs in CI
- DAST scan scheduled in TEST environment

### Observability
- Centralised log aggregation (ELK, Datadog, Splunk, etc.)
- Per-service metric dashboards (CPU, memory, request rate, error rate)
- Error rate alerts routed to on-call channel

### Production Behaviour
- Incidents tracked in ServiceNow (or equivalent)
- Manual MTTR and change failure rate calculated per quarter
- Hotfixes logged separately from regular releases

---

## Level 3 — Measured & Governed

Industry-standard metrics in use. Test coverage extends beyond single services to cross-service contracts and end-to-end journeys.

### Testing
- Contract tests in CI: HTTP (Pact) + Schema (MongoDB document compatibility)
- Application integration tests in TEST environment (CT pipeline)
- System tests (E2E) in TEST + UAT
- Req → Test → Defect traceability in qTest with full coverage matrix

### Performance
- Scheduled load tests in TEST environment
- Performance baselines documented per service
- Penetration test before major releases

### Observability
- Distributed tracing across services (OpenTelemetry, Jaeger)
- SLO dashboards with error budgets
- Synthetic monitoring of critical user journeys

### Production Behaviour
- DORA metrics measured monthly
- Defect Escape Rate (DER) < 25%
- Mean Time to Recovery (MTTR) < 4 hours
- Change Failure Rate (CFR) < 30%

> These thresholds align with DORA's published benchmarks for **"Medium" performers** in the State of DevOps Report.

---

## Level 4 — Data-Driven

Metrics drive decisions. Quality gates enforce performance regression prevention, not just functional correctness. Release readiness is computed, not estimated.

### Testing
- Full CI / CD / CT pipeline discipline
- Test results auto-sync to qTest (no manual updates)
- Release readiness scored from coverage + pass rate + open defects
- Predictability metrics: cycle time + pass-rate trends per release

### Performance
- Performance regression gates in CT (build fails on > X% degradation)
- Continuous DAST + secret scanning
- Threat modelling per feature

### Observability
- Anomaly detection on key metrics (ML or statistical)
- Alerts tied to user-facing SLOs (alert when user impact crosses threshold, not when a single host blips)
- Production telemetry informs test gaps ("we have no test for this code path that errors most in prod")

### Production Behaviour
- DORA metrics live in dashboard (continuously updated)
- DER < 10%, MTTR < 1 hour, CFR < 15%
- SLO error budget tracked weekly with stakeholders

> These thresholds align with DORA's **"High performer"** benchmarks.

---

## Level 5 — Predictive & Self-Healing

The system anticipates failures rather than reacting to them. Human intervention shifts from incident response to strategic improvement.

### Testing
- AI-assisted test selection (run only the tests relevant to the changed code)
- Risk-based test prioritisation driven by code change blast-radius + production signals
- Predictive release readiness analytics

### Performance
- Chaos engineering and fault injection in TEST
- Resilience scoring per service
- Runtime application self-protection (RASP) deployed

### Observability
- Full production debug telemetry (eBPF or equivalent)
- Canary analysis with automatic rollback on error rate / latency / business KPI degradation
- Self-healing on detected anomalies (auto-restart, auto-scale, auto-failover)

### Production Behaviour
- DER < 5%, MTTR < 15 minutes, CFR < 5%
- Predictive incident detection (ML-based forecasting of upcoming failures)
- Automatic rollback on SLO breach

> These thresholds align with DORA's **"Elite performer"** benchmarks.

---

## Production Behaviour KPI Definitions

| KPI | Definition | Source |
|---|---|---|
| **DER** (Defect Escape Rate) | Bugs found in PROD ÷ total bugs found across all environments | qTest + ServiceNow |
| **MTTR** (Mean Time to Recovery) | Average time from incident detection to resolution | ServiceNow |
| **CFR** (Change Failure Rate) | % of deployments that cause an incident or require a hotfix | ADO + ServiceNow |
| **Deployment Frequency** | How often the team deploys to PROD | ADO + Tekton |
| **Lead Time for Changes** | Time from commit merged to deployed in PROD | ADO + Tekton |
| **SLO Error Budget Consumption** | % of allowed error budget used in the current window | Observability platform |

The first four are the **DORA metrics**, an industry standard for measuring software delivery performance. The thresholds in L3, L4, and L5 above use DORA's published "Medium / High / Elite" performer benchmarks.

## Important: How Production Behaviour Differs From the Other Axes

The Testing, Performance, and Observability axes can be **self-assessed by the team** — they describe practices the team puts in place. The Production Behaviour axis cannot be self-assessed. Its scores are **computed from real data** (ServiceNow incident records, ADO deployment data, monitoring platform metrics).

This asymmetry is intentional. A team can claim Level 4 on Testing by listing the practices they have; they cannot claim Level 4 on Production Behaviour without the data to back it up. Production Behaviour is the integrity check on the rest of the framework.
