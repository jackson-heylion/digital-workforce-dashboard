# AGENTS.md — Digital Workforce Dashboard

## Confirmed user decisions (source of truth)

1. **Audience:** first release prioritizes the group management team (2026-10-09).
2. **UI delivery:** use the uploaded complete HTML prototype to **directly restore the existing page layout and interactions**, not invent a redesigned page (2026-10-10).
3. The exact original is at **`prototype/index.html`** (Git blob `92d6df5d024a665cb88fcff9724fcb8ac1787225`). Treat this file as immutable visual/interaction baseline. **Do not edit or overwrite it.**
4. **Confirmed front-end stack (2026-10-10): Vue 3 + TypeScript + Vite**, implementing a faithful 1:1 reproduction in a new `frontend/` directory. This decision is documented in [ADR-001](docs/10-frontend-adr.md). No working frontend project exists yet.
5. Start with UI-F0 (1:1 reproduction), then UI-F1 (real backend/data), then UI-F2 (optional management-oriented rearrangement after explicit approval). See [UI specification](docs/08-ui-reproduction.md), [frontend project plan](docs/11-frontend-backlog.md), [interaction contract](docs/12-frontend-interactions.md), and [acceptance standards](docs/13-ui-acceptance.md).

## Implementation workflow

Work through GitHub Issues #2–#11 (FE-00–FE-09). FE-00 creates the scaffold; parallelize FE-01 Shell and FE-06 Demo Provider, then page reconstruction, tests and demo build. Do not state that build or tests pass until actually run.

## Scope of UI-F0

Reproduce the original topbar, two first-level tabs, overview, KPI cards, real-time feed, per-department employee cards, all charts, employee detail page, five scenario cases, monitoring page, file upload/preview, return navigation, alert demonstrations, layout, breakpoints, typography and colors.

Refer to the actual DOM/CSS/JS rather than relying solely on written design summaries. Original CSS values and interaction behavior take precedence over prose mockups in `docs/06-management-dashboard.md`; the latter describes **future UI-F2 proposals**, not a replacement reference.

### Acceptance

- Compare screenshots at 1440, 1280, 768, 390 and 375 px widths with the original under the same environment.
- Confirm all 4 page IDs and internal navigation: `p0`, `p11`, `p7`, `p4`.
- Check demo employee cards, five scenario anchors, charts (including offline fallback), upload preview, alert actions, time/live updates and responsive overflow.
- Keep production metrics separated from demo data; never report mock counts, costs, success rates or “real employees” as verified production facts.
- The origin has a remote ECharts CDN dependency; document any local bundling substitutions that preserve visuals.

## Architecture constraints for later phases

- Treat a business Task as the counting unit, separate from Run, Step, LLM and tool calls.
- Final business success requires source-system evidence, not a generated Agent reply.
- Dashboard is an observability/value layer, not a replacement Agent orchestrator or approval engine.
- Prefer a simple modular service and trustworthy metric snapshots over decorative real-time updates; technology choices remain draft, see `docs/03-architecture.md` and `docs/07-executive-data-architecture.md`.
- Enforce authentication, data scope, auditability and sensitive-information handling before production connectivity.

## Repository publicity

This repository is public. The exact uploaded original mentions internal business processes and company roles. Its upload was explicitly requested, but that alone does not establish corporate permission to publish. Do not add sensitive records, private endpoints, credentials, production traces or live staff/customer data. Escalate review of the repository visibility to the owner.
