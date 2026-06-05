# Test Strategy by Environment

> **Purpose**: Define what testing is executed in each environment, with what data, and via which pipeline mechanism — covering the journey from local development through to production.

## Overview

Our test strategy operates on three intersecting dimensions:

**Test Types** — Static Code Analysis, Unit Tests, Contract Tests, Component Tests, Application Integration Tests, System Tests, and Non-Functional Tests (Performance, Security, Resilience).

**Execution Strategies** — Continuous Integration (CI), Continuous Deployment (CD), and Continuous Testing (CT) — orchestrated through Azure DevOps (CI) and Tekton on OpenShift (CD/CT).

**Execution Contexts** — Local, Build Pipeline, Sandbox, DEV, TEST, UAT, and PROD — each with its own purpose, audience, data profile, and acceptable level of risk.

This document defines what happens in each context.

---

## Test Type Definitions

Brief refresher on the test types referenced throughout this document.

**Static Code Analysis** — Automated inspection of source code without executing it. Detects code smells, security vulnerabilities, bugs, and enforces coding standards. Tooling: SonarQube, ESLint, language-specific linters.

**Unit Tests** — Verify the smallest testable units (functions, classes, methods) in isolation. Fast, deterministic, run in seconds. Tooling: Jest, JUnit, pytest.

**Contract Tests** — Verify that the agreement between two parties (HTTP API consumer-provider, message schema, database schema) holds. Three sub-types in our architecture: HTTP Pact contracts (Angular ↔ BFF, REST clients ↔ Public API), Schema contracts (MongoDB document compatibility between write-side and read-side), and Message contracts (Change Stream / Kafka notification payload schema). Tooling: Pact, JSON Schema validation, Schema Registry.

**Component Tests** — Verify a single deployable service in isolation with all external dependencies mocked. The service starts for real (REST endpoints, Kafka consumers, in-process logic) but databases, message brokers, and external APIs are stubbed. Tooling: Testcontainers (in-memory MongoDB), WireMock (HTTP), embedded-kafka, MSW (frontend).

**Application Integration Tests** — Verify multiple services within a single application boundary working together with real shared infrastructure. Real MongoDB, real Kafka, real services. Tests cross-service interactions like write-then-read flows.

**System Tests (E2E)** — Verify the entire system end-to-end across all applications. REST/Kafka → Public API + Integration API → MongoDB → notification → BFF → Angular. Verifies cross-application consistency and complete user journeys.

**Non-Functional Tests** — Performance (load, stress, soak), Security (SAST, DAST, dependency vulnerability scanning, penetration testing), and Resilience (chaos engineering, fault injection).

---

## Environment-by-Environment Strategy

### Local Development Workstation

| Aspect | Detail |
|---|---|
| **Purpose** | Fast feedback while writing code. The developer's primary safety net. |
| **Audience** | Individual developer |
| **Risk** | None — fully isolated |
| **Data** | Mocked. In-memory databases (Testcontainers, Fongo). No real credentials. |
| **Pipeline** | None — runs via developer's IDE or local CLI |

**Tests executed locally:**
- Static code analysis (IDE plugins + pre-commit hooks)
- Unit tests on save / before commit
- Component tests against in-memory dependencies
- Contract tests (consumer side) against locally generated mock providers

**What NOT to do locally:**
- Connect to shared DEV/TEST databases — pollutes shared data, breaks reproducibility
- Run System or Application Integration Tests — they require real shared infrastructure
- Test against PROD-like data — privacy and compliance risk

---

### Build Pipeline (CI — Azure DevOps)

| Aspect | Detail |
|---|---|
| **Purpose** | Validate every code change before merge. The merge gate. |
| **Audience** | Pull request reviewer + CI system |
| **Risk** | None — isolated build agents |
| **Data** | Mocked. Same as local: Testcontainers, embedded brokers, no real backends. |
| **Pipeline** | Azure DevOps — triggered on PR open + commit |

**Tests executed in CI (must pass to merge):**
- Static code analysis (SonarQube — quality gate enforced)
- Unit tests (coverage gate enforced, e.g. ≥ 40%)
- Component tests with mocked dependencies
- Contract tests — HTTP (Pact), Schema (MongoDB document compat), Message (notification payload)
- Dependency vulnerability scan (block merge on critical CVEs)
- Build & containerise the deployable artefact

**Key principle**: CI is the only place component tests run. Once the service is deployed somewhere, tests against it are integration/system tests by definition.

---

### Sandbox (Optional — Developer Self-Service)

| Aspect | Detail |
|---|---|
| **Purpose** | Ad-hoc experimentation, debugging, prototyping. Disposable. |
| **Audience** | Individual developer or small team |
| **Risk** | None — sandboxed and short-lived |
| **Data** | Synthetic or mocked. Generated on demand. |
| **Pipeline** | Manual or self-service spin-up (Helm chart, Terraform module) |

**Tests executed in Sandbox:**
- Manual exploratory testing of new features
- Performance prototyping
- Spike testing for technical investigations
- Optional: subset of System Tests during development of cross-service features

Sandboxes are torn down when no longer needed. They are not gated by formal test runs.

---

### DEV Environment

The DEV environment is the **first shared environment** and represents the integration point where the team's work converges. It deserves dedicated attention — see the [Detailed DEV Strategy](#detailed-dev-environment-strategy) section below.

| Aspect | Detail |
|---|---|
| **Purpose** | First integrated, deployed runtime. Validate the deployment + early E2E behaviour. |
| **Audience** | Engineering team |
| **Risk** | Low — resettable; no real users |
| **Data** | Synthetic. Seeded test data + masked production samples for realistic edge cases. |
| **Pipeline** | CD (Tekton) post-publish to Artifactory; CT triggered manually or on schedule |

**Tests executed in DEV:**
- **Tier 1 Smoke Tests** (CD, automatic post-deploy) — health, version match, read-only API checks
- **Tier 2 Smoke Tests** (CD, automatic post-deploy) — write-verify-cleanup cycles
- **Tier 3 Smoke Tests** (CD, automatic post-deploy) — full Kafka → MongoDB → notify → BFF infrastructure chain
- **Application Integration Tests** (CT, manual or scheduled) — write-path and read-path verification
- **System Tests** (CT, manual or scheduled) — full end-to-end journeys
- **Performance Tests** (CT, manual) — ad-hoc performance investigation before TEST runs scheduled ones
- **Exploratory manual testing** — developers validating their own changes

---

### TEST Environment

| Aspect | Detail |
|---|---|
| **Purpose** | The primary validation environment. Where the full quality bar is enforced. |
| **Audience** | QE team + engineering |
| **Risk** | Low — no real users, but treated as production-shaped |
| **Data** | Synthetic + masked production copy. Refreshed regularly. |
| **Pipeline** | CD (Tekton) on successful DEV; CT triggered automatically post-CD |

**Tests executed in TEST (automatic post-deploy):**
- All three tiers of Smoke Tests
- Full Application Integration Test suite (every test, no exclusions)
- Full System Test suite (every E2E journey)
- Performance Tests (scheduled — load, stress, soak)
- DAST security scan against the deployed environment
- Visual regression / cross-browser tests (if applicable to Angular)

TEST is the proving ground. If something passes in TEST, it earns the right to be promoted to UAT.

---

### UAT Environment

| Aspect | Detail |
|---|---|
| **Purpose** | Business validation and pre-production rehearsal. Last chance before PROD. |
| **Audience** | Business stakeholders + UAT testers + selected end users |
| **Risk** | Medium — manual testers use it; lighter-touch automation |
| **Data** | Masked production copy. Realistic data volumes and shapes. |
| **Pipeline** | CD (Tekton) on TEST sign-off; CT runs a subset of tests |

**Tests executed in UAT (post-deploy):**
- Tier 1 + Tier 2 Smoke Tests (Tier 3 infra-chain is blocked — UAT is data-stable)
- Application Integration Tests — **safe subset only** (tests tagged `@safety:uat-safe` that clean up after themselves)
- System Tests — read-mostly E2E (heavy writes blocked)
- Manual UAT scripts executed by business testers
- Acceptance criteria verification against user stories
- Performance Tests — **blocked** (no synthetic load on a UAT shared with humans)

---

### PROD Environment

| Aspect | Detail |
|---|---|
| **Purpose** | Live production. Real users, real money, real consequences. |
| **Audience** | End users + operations team |
| **Risk** | High — anything destructive is blocked |
| **Data** | Real production data. Never copied out. |
| **Pipeline** | CD (Tekton, ServiceNow-gated); only read-only CT |

**Tests executed in PROD (post-deploy):**
- Tier 1 Smoke Tests only (health, version, read-only system reads — fully non-destructive)
- Read-only system tests verifying critical user journeys end-to-end
- Synthetic monitoring (continuous, not just post-deploy)
- Canary analysis on rolling deploys (error rate, latency, business KPIs)
- **Blocked**: Tier 2 (writes), Tier 3 (infra writes), Application Integration, System Tests with writes, Performance Tests

The principle is unambiguous: **PROD is observed, not exercised**. Anything that mutates state in PROD is a real user action or a deliberately rehearsed operations procedure, never an automated test.

---

## Detailed DEV Environment Strategy

This section provides extended guidance for what DEV should look like in its ideal state.

### Purpose of DEV

DEV serves three roles in the testing strategy:

1. **First deployed runtime** — services run as containers in OpenShift, with real Kafka, real MongoDB, real network policies, real DNS. The first time anything resembles production.
2. **Integration point** — multiple services from multiple teams converge here. Inter-service issues that mocks couldn't catch surface for the first time.
3. **Engineering playground** — engineers debug, experiment, validate their assumptions. DEV must be permissive enough that experimentation is safe but disciplined enough that results are meaningful.

### What Runs in DEV

| Category | What | Trigger | Pipeline |
|---|---|---|---|
| Smoke Tier 1 | Health, version, read-only API | Post-deploy (auto) | CD — Tekton |
| Smoke Tier 2 | Write → verify → cleanup | Post-deploy (auto) | CD — Tekton |
| Smoke Tier 3 | Kafka E2E → Mongo → notify → BFF | Post-deploy (auto) | CD — Tekton |
| Application Integration | Write path, Read path with notification | Manual / scheduled | CT — ADO + Tekton |
| System Tests | Full E2E user journeys | Manual / scheduled | CT — ADO + Tekton |
| Performance | Ad-hoc load tests on new endpoints | Manual | CT — ADO + Tekton |
| Manual Exploratory | Developer-driven feature validation | Ad-hoc | None |

Note that **component tests do not run in DEV**. Component tests run in CI only with mocked dependencies. Once a service is deployed to DEV, tests against it are by definition integration tests.

### Data Strategy for DEV

DEV uses a **hybrid data approach**:

**Synthetic data** — programmatically generated, deterministic, version-controlled. Seeded into MongoDB at environment provisioning and refreshed regularly. Covers happy-path scenarios and known edge cases.

**Masked production copy** — a periodically refreshed subset of PROD data with PII and sensitive fields anonymised. Provides realistic shapes, volumes, and edge cases that synthetic data won't catch. Critical for testing migration scripts, performance characteristics under realistic data distributions, and rare scenarios.

**Test data builders / factories** — code-based utilities that generate test entities on-demand within test runs. Each test creates the data it needs, with a deterministic prefix (e.g. `_test_TIMESTAMP_`) for easy cleanup.

What DEV **must not** contain:

- Raw production data (PII, payment details, real user identifiers) — privacy and compliance violation
- Production credentials, API keys, or secrets — should use DEV-specific tokens with limited scope

### Configuration Strategy for DEV

DEV configuration mirrors PROD structure but with safer defaults:

**External integrations** — use sandbox / test endpoints of third-party services (Stripe test mode, sandbox payment gateways, mocked email/SMS providers that don't actually send). Real third-party calls are reserved for TEST/UAT/PROD.

**Feature flags** — DEV is the **first place feature flags become real**. All flags default to "off" but can be toggled per service for testing. Engineers use DEV to validate the on/off behaviour of new flags before they reach TEST.

**Logging level** — verbose (DEBUG) by default. Engineers need full visibility for debugging. PROD is INFO/WARN only.

**Resource limits** — relaxed compared to PROD. Don't replicate PROD's tight CPU/memory caps in DEV; engineers shouldn't fight resource constraints while debugging.

**Network policies** — same shape as PROD (services can/cannot talk to each other based on the same rules) but with permissive egress to allow developer tools to reach the environment.

**Secrets** — managed via Vault. DEV uses DEV-scoped secrets; never the same credentials as TEST/UAT/PROD.

### What Makes a "Good" DEV Environment

A well-functioning DEV environment has these properties:

**Fast feedback** — deployment takes minutes, not hours. Smoke tests complete in under 2 minutes post-deploy. Engineers see results before context-switching.

**Resettable** — there's a one-click (or one-command) way to wipe and re-seed. If DEV gets into a bad state, recovery should take minutes.

**Observable** — full logs, metrics, traces available to all engineers. Same observability stack as PROD (Datadog, Grafana, etc.), so engineers learn the tools they'll need in PROD.

**Cheap** — DEV is the most-used environment by volume of deploys. Right-size it; don't over-provision.

**Honest about scale** — DEV will never match PROD's data volume or traffic. Don't pretend it does. Use TEST for realistic-scale validation; use DEV for correctness validation.

### Common Anti-Patterns to Avoid

**DEV as a parking lot** — services accumulate that no one cleans up. Solution: auto-expire services that haven't received a deploy in 30 days.

**Engineers testing against TEST/UAT** — happens when DEV is slow, broken, or hard to use. Symptom of bigger problems. Fix DEV; don't tolerate the workaround.

**Production data leaking in** — engineers copying a row "just to debug" creates compliance debt. Solution: tooling for masked refreshes + clear policy + audit.

**Brittle test data assumptions** — tests that depend on specific records existing in the DEV DB. Solution: each test creates its own data; reset between test runs.

---

## Summary Matrix

| Test Type | Local | CI | Sandbox | DEV | TEST | UAT | PROD |
|---|---|---|---|---|---|---|---|
| Static Analysis | ✓ (IDE) | ✓ (gate) | — | — | — | — | — |
| Unit Tests | ✓ | ✓ (gate) | — | — | — | — | — |
| Contract Tests | ✓ | ✓ (gate) | — | can-i-deploy | can-i-deploy | can-i-deploy | can-i-deploy |
| Component Tests | ✓ | ✓ (gate) | — | — | — | — | — |
| Smoke Tier 1 | — | — | — | ✓ | ✓ | ✓ | ✓ |
| Smoke Tier 2 | — | — | — | ✓ | ✓ | ✓ | ✗ blocked |
| Smoke Tier 3 | — | — | — | ✓ | ✓ | ✗ blocked | ✗ blocked |
| App Integration | — | — | — | manual/sched | ✓ full | ✓ safe subset | ✗ blocked |
| System Tests | — | — | optional | manual/sched | ✓ full | ✓ read-mostly | ✓ read-only only |
| Performance | — | — | proto | manual | ✓ scheduled | ✗ blocked | ✗ blocked |
| DAST / Pen Test | — | — | — | — | ✓ | — | — |
| Synthetic Monitoring | — | — | — | — | — | — | ✓ continuous |

**Legend**: ✓ = runs; — = not applicable; ✗ blocked = explicitly prevented by safety guards; sched = scheduled; proto = prototyping only.
