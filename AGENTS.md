# AGENTS.md — Digital Workforce Dashboard

## Confirmed user decisions (source of truth)

1. **Audience:** first release prioritizes the group management team (2026-10-09).
1a. **LATEST CONFIRMED SCOPE (2026-10-10): Agent registration + Agent Run/log collection + dashboard display ONLY.** See [docs/17-agent-log-dashboard-mvp.md](docs/17-agent-log-dashboard-mvp.md). More complex business outcome tracking, Capability/Binding administration, approval workflows, financial baseline/ROI and generic Connector management are deferred.
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

## Minimal MVP data and architecture

- **Registered Agent = one digital employee card.** One source Agent ID + platform + environment maps to a unique internal Agent; one Agent has many Runs.
- **Run is a technical execution**, not a unique business Task and not proof of final business success. Count unique `(agent_id, source_run_id)` and calculate technical success rate on completed SUCCEEDED/FAILED Runs only.
- Prefer authenticated HTTP post of Run summaries plus an optional simple read-only adapter when needed; do not build a generic connector platform or business Outcome ledger.
- Recommend a minimal Agent CRUD, Run ingestion/log query, dashboard aggregation API in one service with MySQL; these back-end implementation choices remain proposals.
- Keep the source HTML immutable, UI-F0 visually faithful; in UI-F1 replace fake business costs/saved hours with meaningful Agent Run/latency/error statistics **in the same visual positions**.
- Production never runs demo tickers or backfills mock figures. Authenticate ingestion and reads, redact errors; do not persist raw prompts, personally identifying information or credentials.
- Source of truth for MVP scope: [docs/17-agent-log-dashboard-mvp.md](docs/17-agent-log-dashboard-mvp.md); for Run metrics: [docs/04-metrics-and-contracts.md](docs/04-metrics-and-contracts.md).

## Repository publicity

This repository is public. The exact uploaded original mentions internal business processes and company roles. Its upload was explicitly requested, but that alone does not establish corporate permission to publish. Do not add sensitive records, private endpoints, credentials, production traces or live staff/customer data. Escalate review of the repository visibility to the owner.
