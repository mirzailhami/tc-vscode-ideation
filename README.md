# Topcoder VSCode Plugin — Ideation & Technical Design

**Challenge ID:** `a5ad9e6f-5e08-4261-ac6e-40b3967fa0d9`  
**Submitter:** Mirza Ilhami  
**Date:** March 2026

---

## Summary

This submission presents a complete ideation and technical design for a **read-only VS Code extension** that brings Topcoder challenge information directly into the editor. The plugin eliminates constant browser-to-IDE context switching by surfacing challenge specs, requirements checklists, forum discussions, phase timelines, and attachments — all within VS Code panels and views.

The design is organized into **three modular tiers** that can be enabled independently:

| Tier | Feature Set | Pain Solved |
|------|------------|-------------|
| **A** | Challenge Explorer + Spec Reader | Browser ↔ IDE switching; missing spec details |
| **B** | Inline Requirement Checker + Checklist | Missing subtle requirements; no code-to-spec traceability |
| **C** | Forum & Timeline Live Feed | Missing forum updates; no phase timer visibility |

All features rely exclusively on **existing Topcoder v5/v6 APIs** — no new endpoints are proposed.

---

## Deliverables

| File | Description | Scoring Area |
|------|-------------|-------------|
| [wireframes.md](wireframes.md) | 7 wireframes (WF1–WF7) with annotations + end-to-end user flow diagram | 50% — Wireframes |
| [glossary.md](glossary.md) | 59-entry developer glossary: every UI element, interaction, data concept | 50% — Glossary |
| [architecture.md](architecture.md) | Component diagrams, 8-endpoint API table with verified sample payloads, field-to-UI mapping, auth flow, security, performance, error handling | 25% — Architecture + API |
| [dev-requirements.md](dev-requirements.md) | Prerequisites, dependencies, file structure, package.json, build, testing, risks, requirement traceability matrix, quality gates | 50% — Dev Requirements |
| [extras.md](extras.md) | 5 bonus enhancement ideas (AI extraction, spec diff, dry-run, dashboard, shortcuts) | 25% — Innovation |

---

## How to Review

### Option A: VS Code (Recommended)
1. Open this folder in VS Code.
2. Open any `.md` file and press `Ctrl+Shift+V` (or `Cmd+Shift+V` on macOS) for Markdown Preview.
3. Mermaid diagrams in `architecture.md` require the **Markdown Preview Mermaid Support** extension (`bierner.markdown-mermaid`).

### Option B: GitHub
All files are standard GitHub-flavored Markdown. View directly in the repository browser. Mermaid diagrams are supported natively.

### Option C: Any Markdown Viewer
All files use standard Markdown with optional Mermaid fenced code blocks and HTML tables. Compatible with any modern Markdown renderer.

### Review Order (Suggested)
1. **This README** — overview and context
2. **wireframes.md** — understand what the plugin looks like
3. **glossary.md** — deep-dive into every element
4. **architecture.md** — API calls, security, performance
5. **dev-requirements.md** — how to build it
6. **extras.md** — bonus ideas

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | TypeScript 5+ |
| Platform | VS Code Extension API ^1.85 |
| HTTP | axios |
| Markdown | markdown-it + sanitize-html |
| Date/Time | date-fns |
| Webview UI | @vscode/webview-ui-toolkit (optional React) |
| Build | esbuild |
| Package | @vscode/vsce |
| Testing | Jest (unit) + @vscode/test-electron (integration) |
| Auth | OAuth2 via Auth0 → SecretStorage |

---

## Three Tiers at a Glance

### Tier A: Challenge Explorer + Spec Reader
- Activity bar icon with tree view of active challenges
- Rich webview rendering full challenge spec as formatted markdown
- Requirements checklist with manual checkboxes (persisted)
- Attachment download links
- Status bar countdown timer with color-coded urgency

### Tier B: Inline Requirement Checker + Checklist
- Hover provider linking code lines to matched requirements via keywords
- Gutter decorations (green = checked, gray = unchecked)
- Problems panel warnings for uncovered requirements
- Dedicated requirements checklist tree view with progress bar
- Workspace scan command with progress indicator

### Tier C: Forum & Timeline Live Feed
- Forum webview with threaded post cards and relative timestamps
- Auto-polling with configurable interval (60–300s)
- Visual timeline bar showing all challenge phases with progress
- External reply links (read-only plugin)

---

## API Coverage

All 8 API calls target **existing** Topcoder endpoints:

| # | Endpoint | Purpose |
|---|----------|---------|
| 1 | `GET /v5/challenges` | List active challenges |
| 2 | `GET /v5/challenges/{id}` | Challenge detail + spec |
| 3 | `GET /v5/challenges/{id}/attachments` | Attachment files |
| 4 | `GET /v5/resources` | Resource roles (registrants) |
| 5 | `GET /v5/challenge-discussions` | Forum posts |
| 6 | `GET /v5/submissions` | Submission history |
| 7 | `GET /v5/members/{handle}` | Member profile |
| 8 | `POST auth0/oauth/token` | Authentication |

Full details with query params, auth headers, response fields, caching strategy, and **verified sample payloads** in [architecture.md](architecture.md#10-api-verification-appendix). Requirement traceability matrix and quality gates in [dev-requirements.md](dev-requirements.md#11-requirement-traceability-matrix).

---

## Confidentiality

This submission is provided for the Topcoder challenge review process. Content is original work by the submitter. No proprietary Topcoder source code was used — only publicly documented API endpoints.

---

## License

This design document is submitted under the terms of the Topcoder challenge agreement.
