# AGENTS.md — Digital Workforce Dashboard

## Confirmed user decisions (source of truth)

1. **Audience:** first release prioritizes the group management team (2026-10-09).
1a. **LATEST CONFIRMED SCOPE (2026-10-10): Agent registration + Agent Run/log collection + dashboard display ONLY.** See [docs/17-agent-log-dashboard-mvp.md](docs/17-agent-log-dashboard-mvp.md). More complex business outcome tracking, Capability/Binding administration, approval workflows, financial baseline/ROI and generic Connector management are deferred.
1b. **Confirmed ingestion choice:** Add a simple reporting POST at each Agent invocation boundary; see [Agent Run reporting standard v1](docs/18-agent-run-reporting-standard.md) and [OpenAPI](openapi/agent-run-reporting-v1.json). Do not default to building log-polling connectors.
2. **UI delivery:** use the uploaded complete HTML prototype to **directly restore the existing page layout and interactions**, not invent a redesigned page (2026-10-10).
3. The exact original is at **`prototype/index.html`** (Git blob `92d6df5d024a665cb88fcff9724fcb8ac1787225`). Treat this file as immutable visual/interaction baseline. **Do not edit or overwrite it.**
4. **Confirmed front-end stack (2026-10-10): Vue 3 + TypeScript + Vite**, implementing a faithful 1:1 reproduction in a new `frontend/` directory. This decision is documented in [ADR-001](docs/10-frontend-adr.md). No working frontend project exists yet.
5. Start with UI-F0 (1:1 reproduction), then UI-F1 (real backend/data), then UI-F2 (optional management-oriented rearrangement after explicit approval). See [UI specification](docs/08-ui-reproduction.md), [frontend project plan](docs/11-frontend-backlog.md), [interaction contract](docs/12-frontend-interactions.md), and [acceptance standards](docs/13-ui-acceptance.md).

## Implementation workflow

UI-F0 prototype faithful reproduction remains FE-00–FE-08 (GitHub Issues #2–#10). UI-F1 FE-09 (#11) now only reads Agent registration and Run technical metrics from the API. Minimal back-end tasks are #12 (Agent CRUD), #13 (Agent Run ingestion), #14 (dashboard aggregates). Do not implement retired business-Task/Outcome, complex governance and ROI specs from old docs. Do not state build or tests pass until actually run.

## Scope of UI-F0

Reproduce the original topbar, two first-level tabs, overview, KPI cards, real-time feed, per-department employee cards, all charts, employee detail page, five scenario cases, monitoring page, file upload/preview, return navigation, alert demonstrations, layout, breakpoints, typography and colors.

Refer to the actual DOM/CSS/JS rather than relying solely on written design summaries. Original CSS values and interaction behavior take precedence over prose mockups in `docs/06-management-dashboard.md`; the latter describes **future UI-F2 proposals**, not a replacement reference.

### Acceptance

- Compare screenshots at 1440, 1280, 768, 390 and 375 px widths with the original under the same environment.
- Confirm all 4 page IDs and internal navigation: `p0`, `p11`, `p7`, `p4`.
- Check demo employee cards, five scenario anchors, charts (including offline fallback), upload preview, alert actions, time/live updates and responsive overflow.
- Keep production metrics separated from demo data; never report mock counts, costs, success rates or “real employees” as verified production facts.
- The origin has a remote ECharts CDN dependency; document any local bundling substitutions that preserve visuals.

## REQUIRED: metric-by-metric product confirmation (2026-10-10)

- Read [docs/21-metrics-field-confirmation-register.md](docs/21-metrics-field-confirmation-register.md) before changing metric/UI/event fields. The user explicitly requested a **discussion and explicit approval for EACH metric/field**: displayed name/unit/precision/empty-state; computation/time window/denominator; data producer/source; semantics/validation/quality standards.
- **Every row in register 21 is UNCONFIRMED** until the user replies with a decision. Previous assistant proposals in docs/04, 17, 18, 19, 20 and OpenAPI drafts are not product approval. Do not mark confirmed or promote schema changes without user input.
- Discuss sequentially in chat and write each accepted decision to the register with date and implications. A01 is confirmed as all non-archived registered Agents (including paused/disabled). A02 is confirmed as explicitly owner/admin-marked 'in use' from registry metadata, not auto-derived from recent runs. Their remaining edge cases are unconfirmed. A03 is now also confirmed: each non-archived Agent has exactly one primary owning department for homepage card grouping and counts. Secondary use across multiple departments does not cause double counting. **A04 is now confirmed as option C:** separate registered administrative state primary badge, A02 human-marked in-use badge, and an optional live RUNNING execution badge only when reliable start/terminal Run events exist; show last execution separately. Do NOT infer online/offline, mandate RUNNING (v1 terminal-only remains valid), fabricate a heartbeat, or choose timeout/enum/colors without approval. **A05 confirmed option A (2026-10-10):** display name, icon and description are maintained manually by owner/admin in Agent registration; no automatic dependency on external Agent platform sync. Validations, icon upload/default, duplication, requiredness, length, history and presentation remain UNAPPROVED. **A06 confirmed option A (2026-10-10):** owner/admin manually maintains the current Agent version string in registration; cards/details display current value. No external platform sync, no required per-Run agent_version snapshot, and never retrospectively label past runs as the current version. Naming, requiredness, default, upgrade history, environments, validation and historical analysis remain UNAPPROVED. **A07 confirmed option A (2026-10-10):** two distinct, manually maintained Agent registration fields: a structured technology platform code and a business/scenario provenance note. Do NOT auto-discover/sync them or treat prototype `emps[].src` as a platform key. Platform enums, requiredness, description length, migration history and exact UI positions remain UNAPPROVED. **A08 confirmed option A (2026-10-10):** one human-maintained, display-only responsible person-or-team label per Agent, compatible with individual names, name+role, or team names. No mandatory IAM/HR link and NEVER authorize users from a free-text owner label. Requiredness, identifiers, multi-owner, authorization, handoff, privacy and history remain UNAPPROVED. **A09 confirmed option A (2026-10-10):** server generates and retains Agent registry `created_at` on record insertion, and employee cards label it "登记" (registered), not actual Agent creation or production launch. Precision, timezone, missing value, write permissions, migration and environment rules remain UNAPPROVED. Never fill it from prototype `emps[].created` demo dates. **Next discuss A10 scenario demo entry and five predefined scenario bindings.** Original `emps[].scene` maps the five demo real-labelled agents to one static `sc-1..5` section; `real` controls demonstration link, not proof of actual live activity. Decide whether each Agent optionally binds one scene, many, or none. Do not infer from Run, free-text provenance or manually marked in-use, and do not allow missing codes to fall back to `sc-1` in production. Never confuse agent version with API schema_version/model version or silently add OpenAPI fields. Organization source, unassigned group, historical reassignment, authorization and A01/A02 unresolved corners remain open. No other metric or schema is approved.

## Protocol review and nonfunctional design (2026-10-10)

- Original v1.0 API remains at `openapi/agent-run-reporting-v1.json`, immutable in meaning, deployed nowhere so far.
- Read [docs/19-prototype-metrics-coverage-audit.md](docs/19-prototype-metrics-coverage-audit.md) before claiming any metric can be computed. The original has synthetic costs, savings, intervention and alert recovery.
- [docs/20-performance-extensibility-portability.md](docs/20-performance-extensibility-portability.md) is a recommended performance/scalability/portability architecture, **not benchmark results**.
- `openapi/agent-run-reporting-v1.1-draft.json` is a **draft** extension only, NOT an approved deployed schema. It adds optional operation_type, sanitized summary, manually observed interventions, cost with quality/source, correlation ID and bounded registered measurements. Keep existing v1 clients supported through strict schema version dispatch.
- For data quality, never infer business result from technical SUCCEEDED; never infer saved hours from Run duration; never use missing count/cost/intervention as zero. Costs require currency, decimal, provenance and billing vs estimate separation.
- Start with authenticated durable commit to MySQL, transaction-guarded `(agent_id,run_id)` idempotency, indexed pagination and query aggregation. Add daily snapshots/queues/OLAP only on measured bottlenecks; support migrations and platform-neutral fields.

## Minimal MVP data and architecture

- **Registered Agent = one digital employee card.** One source Agent ID + platform + environment maps to a unique internal Agent; one Agent has many Runs.
- **Run is a technical execution**, not a unique business Task and not proof of final business success. Count unique `(agent_id, source_run_id)` and calculate technical success rate on completed SUCCEEDED/FAILED Runs only.
- **Required v1 contract:** `POST /api/v1/ingest/agent-runs`, registered `agent_code`, stable `run_id`, `schema_version: 1.0`, `status`, `started_at`, `finished_at` for terminal, optional duration/usage/sanitized error; service bearer auth scoped to Agent/environment; 200 only after durable commit, no duplicate runs; no business outcome ledger. See docs/18 and OpenAPI as source of truth.
- Recommend a minimal Agent CRUD, Run ingestion/log query, dashboard aggregation API in one service with MySQL; these back-end implementation choices remain proposals.
- Keep the source HTML immutable, UI-F0 visually faithful; in UI-F1 replace fake business costs/saved hours with meaningful Agent Run/latency/error statistics **in the same visual positions**.
- Production never runs demo tickers or backfills mock figures. Authenticate ingestion and reads, redact errors; do not persist raw prompts, personally identifying information or credentials.
- Source of truth for MVP scope: [docs/17-agent-log-dashboard-mvp.md](docs/17-agent-log-dashboard-mvp.md); for Run metrics: [docs/04-metrics-and-contracts.md](docs/04-metrics-and-contracts.md).

## Repository publicity

This repository is public. The exact uploaded original mentions internal business processes and company roles. Its upload was explicitly requested, but that alone does not establish corporate permission to publish. Do not add sensitive records, private endpoints, credentials, production traces or live staff/customer data. Escalate review of the repository visibility to the owner.
