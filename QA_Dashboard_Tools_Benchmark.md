# QA Dashboard Tools — Benchmark for Consolidated View

> **Purpose**: Evaluate commercial and open-source tools against the scope of our built-in Product Maturity Assessment & QA Dashboard project. Help decide whether to keep building our own consolidated dashboard or adopt an existing tool.

> **What "consolidated QA dashboard" means here**: A single pane of glass aggregating test management (qTest/XRay), test execution (ADO/Tekton), deployment & change management (ServiceNow), and production telemetry (Datadog/Grafana/etc.) into stakeholder-friendly views — with maturity assessment as our specific add-on capability.

## TL;DR

No single off-the-shelf tool ticks every box. Each category-leader excels in its native domain but compromises on the others:

- **Test management tools** (qTest, XRay, PractiTest) — strong on test-side metrics, weak on production telemetry.
- **DevOps platforms** (Azure DevOps) — strong on pipeline metrics, weak on consolidated test management and production observability.
- **ITSM platforms** (ServiceNow DCV) — strong on change governance and aggregation, weak on engineering-grade test dashboards.
- **Observability platforms** (Datadog, Grafana, New Relic) — strong on production telemetry, weak or absent on test management.

A custom-built dashboard remains the only practical option to **combine all four perspectives plus organisation-specific maturity assessment** into a single coherent view. Commercial tools are credible alternatives only if you accept partial coverage and the limitations are listed below.

---

## Comparison Matrix

| Tool | Test Management | Test Execution | CI/CD Integration | Deployment / Change | Production Telemetry | Cross-Tool Dashboards | Maturity Assessment | Customisation | Cost Model | Best For |
|---|---|---|---|---|---|---|---|---|---|---|
| **ServiceNow DevOps Change Velocity** | ◐ via integration | ◐ pulls results | ✓ ADO, Jenkins, GitHub | ✓ native | ✗ | ✓ DevOps Insights | ✗ | ◐ via apps | High (ITSM Pro license + per-user) | Governance / compliance dashboards |
| **Azure DevOps (Test Plans + Analytics)** | ◐ basic | ✓ native | ✓ native | ◐ release pipelines | ✗ | ◐ ADO-only | ✗ | ✓ widgets | Medium (per-user) | Teams already all-in on ADO |
| **Tricentis qTest** | ✓ native | ✓ native + orchestration | ✓ ADO, Jenkins, GitLab | ✗ | ✗ | ◐ test-focused | ✗ | ◐ dashboards | Medium-High | Enterprise test management |
| **XRay for Jira** | ✓ native | ◐ via CI plugins | ✓ Jenkins, ADO, Bamboo | ✗ | ✗ | ◐ inside Jira | ✗ | ✓ Jira gadgets | Medium (per-user, Jira-tied) | Jira-centric teams |
| **PractiTest** | ✓ native | ◐ links external | ✓ Jenkins, ADO, GitHub | ◐ via ServiceNow link | ✗ | ✓ cross-tool aware | ✗ | ✓ dashboards | Medium-High | Multi-issue-tracker teams |
| **TestCollab** | ✓ native | ◐ links external | ✓ bi-directional, many | ◐ ServiceNow incidents | ✗ | ◐ test-focused | ✗ | ✓ via API | Medium | Multi-tool QA orchestration |
| **Datadog** | ✗ | ◐ CI Visibility | ✓ 700+ integrations | ◐ deployment tracking | ✓ best-in-class | ✓ unified observability | ✗ | ✓ extensive | Very high (per-host + per-GB) | Production-side observability |
| **Grafana (+ LGTM stack)** | ✗ | ◐ via plugins | ✓ data source plugins | ◐ via plugins | ✓ via Loki/Mimir/Tempo | ✓ open dashboards | ✗ | ✓ unlimited | Free (OSS) + Cloud tier | Cross-source custom dashboards |
| **New Relic** | ✗ | ◐ CI insights | ✓ many | ◐ deployment markers | ✓ NRDB unified | ✓ unified | ✗ | ✓ NRQL | Medium (100GB free + per-user) | Production telemetry with free tier |
| **Custom Built-in Dashboard** | ✓ via qTest API | ✓ via qTest API | ✓ via ADO API | ✓ via ServiceNow API | ✓ via Prom/Datadog API | ✓ all-in-one | ✓ native | ✓ unlimited | Engineering time | Org-specific, fully tailored view |

**Legend**: ✓ = native / strong support · ◐ = partial / via integration · ✗ = not supported

---

## Capability Definitions

Before reading the per-tool summaries, here's what each column in the matrix means:

**Test Management** — Storing, organising, and maintaining test cases, test plans, test cycles, traceability matrices. Where the "what we test" lives.

**Test Execution** — Running tests (manual or automated) and capturing pass/fail/blocked/incomplete results, runtime data, and screenshots. Where the "what happened" lives.

**CI/CD Integration** — Native or plugin-based hooks into build pipelines (ADO, Jenkins, GitLab CI, Tekton) to trigger tests and capture results.

**Deployment / Change** — Visibility into what's been deployed where, with which artefact version, governed by which change request.

**Production Telemetry** — Real-time metrics, logs, traces, error rates, SLOs from production systems.

**Cross-Tool Dashboards** — The ability to render data from multiple external systems on a single dashboard, not just from the tool's own data store.

**Maturity Assessment** — Our specific capability: questionnaire-driven maturity scoring across multiple dimensions, weighted, with historical tracking and roadmap.

**Customisation** — How freely you can build dashboards, add fields, define widgets, customise scoring algorithms.

**Cost Model** — License model and how cost scales with usage.

---

## Tool-by-Tool Assessment

### ServiceNow DevOps Change Velocity (DCV)

ServiceNow DCV is the **governance-and-aggregation layer** of the QA dashboard story. It pulls data from CI/CD pipelines, test management tools, code quality scanners, security tools, and artefact repositories into ServiceNow's data model, then uses that data to automate change request creation and approval.

For a consolidated QA dashboard, DCV is most valuable when the audience is **change managers, compliance officers, and stakeholders who think in terms of risk and approvals**. Its DevOps Insights dashboards surface DORA metrics (deployment frequency, lead time, change failure rate, MTTR) computed from the integrated tool data. It can ingest test results, security scan outcomes, code quality metrics, and deployment logs to inform automated risk-based approvals.

Strengths: industry-standard ITSM integration, automated change creation from pipelines, cross-tool data aggregation in a single platform, strong compliance/audit story. Integrates natively with Azure DevOps, Jenkins, GitHub, GitLab, plus JFrog (artefacts), SonarQube (quality), Veracode (security).

Limitations: not built for engineering-grade test management dashboards. The data model is change-centric, not test-centric. Stakeholders who want to drill into "show me the test pass rate for the payment service this sprint, broken down by test suite" will find DCV awkward. It surfaces test data as evidence supporting change decisions, not as a quality narrative. Production telemetry integration is also weak — DCV doesn't speak metrics-and-logs natively. Requires an ITSM Pro license, which is a significant commercial commitment.

**Verdict for our use case**: Excellent complement to a custom dashboard, especially for the deployment/change management slice and DORA metrics. Poor fit as the sole consolidated QA dashboard.

### Azure DevOps (Test Plans + Analytics)

ADO is the **most natural fit if your organisation is already deeply invested in the Microsoft stack**. It includes Boards (planning), Repos (source), Pipelines (CI/CD), Artifacts, and Test Plans under one roof, with built-in Analytics views and customisable dashboards via widgets.

For a consolidated QA dashboard, ADO works well when test management is happening inside ADO Test Plans and the only data you want to surface is pipeline-and-test-related. The dashboards are widget-based and you can pin queries, charts, and pipeline status to a single view. Native integration with qTest exists via Planview Hub (bi-directional sync of test cases, defects, requirements).

Strengths: zero integration friction inside the Microsoft ecosystem, decent test execution dashboards, fully customisable widget-based dashboards, native CI/CD integration (because it is CI/CD), Boards-to-pipeline traceability.

Limitations: Test Plans is widely considered the weak point of ADO — most QE teams using ADO Pipelines still prefer a dedicated test management tool like qTest or XRay. Production telemetry is absent (you'd need to integrate Application Insights separately). Change management is light — there's nothing comparable to ServiceNow's change request workflow. Cross-tool dashboarding is limited to what you can extract via the ADO Analytics OData feed.

**Verdict for our use case**: Good for ADO-native teams, but won't consolidate ServiceNow change data or production telemetry without additional tooling. The Test Plans weakness is a real concern if test management is a priority.

### Tricentis qTest

qTest is the **enterprise test management leader** and the tool we currently integrate with. It provides centralised test case management, execution orchestration, manual and automated test results, requirements traceability, and reporting. The recent positioning includes "test orchestration" — coordinating execution across multiple frameworks and pipelines.

For a consolidated QA dashboard, qTest's strength is the **test perspective** — pass rates, coverage, traceability, release readiness. It integrates with ADO Boards and Pipelines bi-directionally, with Jenkins, with GitLab CI, and supports automation result ingestion from frameworks like Selenium, Cypress, Playwright via the qTest Automation API.

Strengths: industry-standard test management, strong reporting on test-related metrics, native ADO integration via Planview Hub (model-based sync of test cases, defects, requirements), support for both manual and automated workflows, qTest Insights for cross-project reporting.

Limitations: dashboards are test-centric — qTest won't natively surface production incidents from ServiceNow or telemetry from Datadog. Building a true cross-source view requires custom work outside qTest. The "release readiness" widget is good for test pass rate but doesn't incorporate deployment state or production behaviour.

**Verdict for our use case**: Essential as the test management data source (which is how we use it), but not designed to be the consolidated dashboard itself. The custom dashboard layer above it is the right architecture.

### XRay for Jira

XRay is **qTest's main competitor**, embedded inside Jira. It manages test cases as Jira issues, supports manual and automated tests (including BDD with Gherkin), integrates with CI/CD pipelines through plugins (Jenkins, ADO, Bamboo, GitLab, CircleCI, etc.), and surfaces results as Jira gadgets on Jira dashboards.

For a consolidated QA dashboard, XRay's strength is the **tight Jira coupling** — if your team lives in Jira for everything, XRay test data appears alongside user stories, defects, and epics in the same UI. Results from CI pipelines appear in Jira immediately after execution.

Strengths: deepest Jira integration of any test management tool, BDD support, automated test result ingestion from CI, Jira-native dashboarding via gadgets.

Limitations: dashboards are bounded by Jira's gadget model — fine for Jira-internal data, awkward for ServiceNow, Datadog, or external test result sources. XRay's reach beyond Jira is weaker than qTest's. No native maturity assessment, no native production telemetry. If your organisation doesn't standardise on Jira, XRay's value proposition collapses.

**Verdict for our use case**: A strong alternative to qTest if Jira is the central organising tool. Same overall limitation though — not a consolidated cross-domain dashboard.

### PractiTest

PractiTest positions itself as the **test management tool for teams with diverse tool stacks**. It explicitly emphasises cross-tool integration: Jira, Azure DevOps, GitHub Issues, ServiceNow, plus CI/CD platforms. Real-time dashboards show test coverage, pass rates, and risk areas.

For a consolidated QA dashboard, PractiTest is interesting because it deliberately targets the "teams using multiple issue trackers" use case. Product managers can see quality metrics without digging through individual testing tools.

Strengths: explicit cross-tool design (links to ServiceNow exists, not just Jira), real-time dashboards aimed at non-tester stakeholders, traceability across test cases, runs, and defects in a single view.

Limitations: still primarily a test management tool — production telemetry isn't its game. Less established in large enterprise rollouts than qTest or XRay. Dashboard customisation is constrained compared to a custom build.

**Verdict for our use case**: Worth knowing about, but the cross-tool story doesn't extend to production observability. Niche fit for teams who want a TM tool with built-in cross-tracker awareness.

### TestCollab

TestCollab is a similar profile to PractiTest — test management with strong multi-tool integration. Bi-directional sync with Jira, GitHub, GitLab, Azure DevOps, ServiceNow, Asana, Linear, ClickUp. Can create ServiceNow incidents from failed test steps (interesting bridge for the change/incident side). CI/CD reporter plugins for Playwright, Cypress, Selenium, Katalon. REST API and SDK for custom integration.

Strengths: broad integration footprint, including the unusual capability of writing back to ServiceNow as incidents (most test tools only push defects to Jira/ADO), MCP support for AI agents.

Limitations: less mature commercial presence than qTest or XRay, no production telemetry story, dashboards are test-centric.

**Verdict for our use case**: Niche player. Worth a closer look if multi-tool reach is the deciding factor, but doesn't change the fundamental gap on production telemetry and custom maturity assessment.

### Datadog

Datadog is the **observability-platform leader** with 700+ integrations spanning infrastructure metrics, APM, logs, RUM, synthetic monitoring, CI Visibility, security, and recently LLM observability. Its CI Visibility module specifically tracks test execution results and pipeline performance.

For a consolidated QA dashboard, Datadog's strength is **production-side everything** — what's happening in real time, what failed, who's affected, how fast we recovered. CI Visibility extends that into pipeline metrics. With its breadth of integrations, you can stitch in deployment events, alerts, and even synthetic test results.

Strengths: most complete observability platform on the market, single-pane visibility across infrastructure-to-applications-to-tests, Watchdog AI for anomaly detection, Bits AI SRE agent for autonomous incident investigation.

Limitations: no test management — Datadog doesn't store test cases, requirements, or traceability matrices. CI Visibility shows you that tests ran and pass rates over time, but it's not where you'd manage test design. Pricing is the well-known concern: per-host plus per-GB plus per-feature, with cardinality explosions that surprise CFOs. Vendor lock-in is real because Datadog converts everything to its proprietary format under the hood.

**Verdict for our use case**: An obvious choice for the production telemetry slice. Could be the source of the "production behaviour" axis in our maturity model. Not viable as the consolidated dashboard for QA-specific needs (test management, maturity, change governance).

### Grafana (with LGTM stack)

Grafana is **the visualization layer of choice for cross-source dashboards** in the open-source world. The Grafana stack (Loki for logs, Tempo for traces, Mimir for metrics, plus Grafana itself) covers observability, but Grafana's strength is connecting to *any* data source via plugins — Prometheus, Elasticsearch, PostgreSQL, MySQL, Snowflake, BigQuery, even Google Sheets, plus 100+ others.

For a consolidated QA dashboard, Grafana is the **closest match to our custom dashboard's data philosophy**. You could in principle build a Grafana dashboard that queries qTest's PostgreSQL backend (if exposed), ServiceNow's API (via a custom data source plugin), ADO's Analytics OData, and Datadog or Prometheus for production. All on one screen.

Strengths: open-source, no vendor lock-in, unlimited data sources, mature dashboard editor, alerting integrated, free if self-hosted, Grafana Cloud option available. Highly customisable visualisation.

Limitations: Grafana is a *visualisation* tool, not a data store. You're responsible for the backends (Prometheus, Loki, Tempo, or whatever else). It doesn't natively understand test management semantics — qTest's "Target Release/Build" property, requirements traceability, maturity scoring — so you'd write panels with raw SQL or API queries to extract that meaning. The DIY operational overhead of the LGTM stack is non-trivial.

**Verdict for our use case**: A serious alternative if engineering time is the constraint, but you'd build essentially the same dashboard work that we're already building, with the bonus of Grafana's polished UI and minus the freedom to model maturity assessment as a first-class concept. The maturity scoring engine is custom to our domain and doesn't map cleanly to Grafana panel types.

### New Relic

New Relic is **Datadog's closest commercial competitor on observability**, with a notably different pricing model: 100GB/month free ingest including the full platform, no per-host charges (hosts/agents/containers are unmetered), and per-user pricing for full platform access. NRDB (their unified telemetry database) stores metrics, events, logs, and traces with NRQL for cross-signal queries. Workflow Automation supports no-code rollbacks when error rates spike post-deployment.

For a consolidated QA dashboard, New Relic shares Datadog's strengths and limitations: excellent at production observability, no native test management, integrates with deployment events and CI tools but isn't a test platform.

Strengths: best free tier in the observability category (100GB/month is genuinely useful), unified telemetry model, no per-host cost trap, automated deployment rollback workflows.

Limitations: same as Datadog on the test-management front — none. Per-user pricing scales fast for large engineering orgs. Dashboards are good but bounded to the data inside NRDB.

**Verdict for our use case**: A more cost-friendly observability alternative than Datadog. Same fit/non-fit pattern as Datadog for the consolidated dashboard question.

### Custom Built-in Dashboard (our current project)

Our custom dashboard is built specifically around the **four perspectives we identified**: test management (qTest), test execution & CI (ADO/Tekton), deployment & change (ServiceNow), and production telemetry — plus our **organisation-specific maturity assessment** capability with weighted scoring across 9 dimensions, 22 questions, retake-with-pre-fill, and SQLite-backed persistence.

What it does that no commercial tool does:
- Combines all four data domains in a single coherent view per product team
- First-class **Product Maturity Assessment** with custom dimensions, weights, questions, and roadmap recommendations
- "Test Executions in the Last Month" widget driven by live qTest API calls with pagination and release-grouping
- "Requirements Coverage" widget computed from qTest's Traceability Matrix recursive walk
- Per-team historical maturity tracking
- Configurable per-team metadata (product team, tech lead, qTest project, ADO project, ServiceNow CI)

Strengths: every aspect is tailored to our organisation's specific architecture (Public API → Integration API → MongoDB → BFF), naming conventions, and quality framework. Zero per-user license cost. No vendor lock-in. Engineering team owns the roadmap.

Limitations: engineering time is the cost — features that arrive out-of-the-box in commercial tools (auth, RBAC, multi-tenancy, mobile support, etc.) take real effort to build. Maintenance is on us. Adoption depends on us building the right UX. The "single pane of glass" promise is only as good as the integrations we maintain.

**Verdict**: This is currently the only option that delivers a true consolidated QA + maturity dashboard. The question is whether the maintenance cost is justified by the unique capabilities — particularly the maturity assessment, which is the most genuinely differentiated piece.

---

## Decision Framework

The question isn't "which tool should we use?" but "**which combination of tools makes sense for our use case?**" Here's a framework to evaluate.

### Build the custom dashboard if...

- The **Product Maturity Assessment** capability matters strategically (this is the most differentiated piece — no commercial tool offers a configurable, organisation-specific maturity framework)
- You want full control over per-team view layouts and metrics
- The four-domain consolidation (test + CI + change + production) is genuinely required in one place for stakeholders
- You have engineering bandwidth for the (relatively modest) ongoing maintenance
- You expect the metrics and dimensions to evolve as the organisation's quality philosophy matures

### Consider replacing the custom dashboard with commercial tools if...

- The maturity assessment can be dropped or replaced with periodic surveys outside the tool
- Stakeholders are willing to use 2-3 tools (e.g., qTest for test data + ServiceNow for change + Datadog for production) instead of one
- Engineering bandwidth is severely constrained and ongoing dashboard maintenance is unsustainable

### Hybrid approach (likely the pragmatic answer)

Keep the custom dashboard for the parts where it adds unique value, and let specialist tools own their native domains:

- **Custom dashboard owns**: maturity assessment, per-team consolidated overview, requirement coverage widget, qTest test executions widget
- **ServiceNow DCV** owns: change governance, automated change approval, DORA metrics from pipelines
- **qTest** owns: test case design, traceability matrices, manual test execution, test orchestration
- **Datadog or Grafana** owns: production telemetry, SLO dashboards, incident response

The custom dashboard then becomes the "executive view" that links to each specialist tool for deep dives, rather than trying to replicate everything each tool does natively.

---

## Suggested Next Steps

1. **Validate the maturity assessment hypothesis** — confirm with stakeholders that this is genuinely valuable enough to justify a custom build. If yes, the custom dashboard has a clear reason to exist.
2. **Quantify the engineering cost** — track time spent on the custom dashboard over the next quarter to compare against commercial license costs.
3. **Define the data freshness contract** — for each integrated data source (qTest, ADO, ServiceNow, telemetry), document what "near real-time" means and whether commercial alternatives would meet the same bar.
4. **Run a pilot of one commercial alternative** for one specific domain (e.g., put Grafana in front of qTest data for the test execution widget) and compare developer experience and stakeholder satisfaction.
5. **Document the integration architecture as a stable contract** — so that if the dashboard is later replaced or supplemented, the data flow can be re-pointed without re-architecting source systems.
