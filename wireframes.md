# Wireframes — Topcoder VSCode Plugin

> **Format:** Each wireframe is provided as an **Excalidraw sketch** (visual) alongside a **text-based layout** (accessible reference). Open the `.excalidraw` files in VS Code with the [Excalidraw extension](https://marketplace.visualstudio.com/items?itemName=pomdtr.excalidraw-editor) for the best visual experience.  
> **Convention:** `[ ]` = button/action, `[x]` = checkbox checked, `$(icon)` = VS Code Codicon reference, `───` = separator.

### Visual Wireframe Files

| # | Wireframe | Excalidraw File |
|---|-----------|-----------------|
| WF1 | Activity Bar & Challenge Explorer | [`wireframes/WF1-challenge-explorer.excalidraw`](wireframes/WF1-challenge-explorer.excalidraw) |
| WF2 | Spec & Requirements Webview | [`wireframes/WF2-spec-webview.excalidraw`](wireframes/WF2-spec-webview.excalidraw) |
| WF3 | Status Bar Countdown | [`wireframes/WF3-status-bar.excalidraw`](wireframes/WF3-status-bar.excalidraw) |
| WF4 | Inline Requirement Checker | [`wireframes/WF4-inline-checker.excalidraw`](wireframes/WF4-inline-checker.excalidraw) |
| WF5 | Requirements Checklist Panel | [`wireframes/WF5-checklist-panel.excalidraw`](wireframes/WF5-checklist-panel.excalidraw) |
| WF6 | Forum & Timeline Sidebar | [`wireframes/WF6-forum-timeline.excalidraw`](wireframes/WF6-forum-timeline.excalidraw) |

---

## WF1: Activity Bar & Challenge Explorer (Tier A)

This wireframe shows the primary entry point — a new "Topcoder" icon in the Activity Bar that opens a tree-based sidebar for browsing active challenges.

```
┌─────────────────────────────────────────────────────────────────────┐
│  ACTIVITY BAR          │  SIDEBAR (Topcoder)                       │
│  ┌──────┐              │                                           │
│  │ 📁   │ Explorer     │  MY ACTIVE CHALLENGES          [$(refresh)]│
│  │ 🔍   │ Search       │  ─────────────────────────────────────────│
│  │ 🔀   │ Source Ctrl  │  ▶ $(trophy) TC VSCode Plugin     [Active]│
│  │ 🐛   │ Run/Debug    │  ▼ $(trophy) RFP Proposal BE     [Active]│
│  │ 🧩   │ Extensions   │  │  ├── $(book)    Spec & Requirements    │
│  │      │              │  │  ├── $(comment) Discussions (12)       │
│  │ ┌──┐ │              │  │  ├── $(cloud-upload) Submissions       │
│  │ │TC│ │ ◀ TOPCODER   │  │  ├── $(person)  Registrants (24)      │
│  │ └──┘ │   (active)   │  │  └── $(clock)  Timeline                │
│  │      │              │  ▶ $(trophy) RFP Proposal UI   [Upcoming]│
│  └──────┘              │                                           │
│                        │  ─────────────────────────────────────────│
│                        │  $(info) Logged in as: mirzailhami       │
├────────────────────────┴───────────────────────────────────────────┤
│  STATUS BAR                                                        │
│  $(clock) TC: Submission ends in 4h 12m  │  $(check) 5/8 reqs     │
└─────────────────────────────────────────────────────────────────────┘
```

### Annotations
- **Activity Bar Icon ("TC"):** Custom Topcoder icon registered via `viewsContainers.activitybar` in `package.json`. Activates the Topcoder sidebar on click.
- **Tree Nodes:** Each challenge is a collapsible `TreeItem` with `label` = challenge name and `description` = status badge (`Active`, `Upcoming`). Status badge uses `TreeItemLabel` with highlight color.
- **Child Nodes:** Five fixed children per challenge: Spec & Requirements, Discussions, Submissions, Registrants, Timeline. Each has a contextual icon (`$(book)`, `$(comment)`, `$(cloud-upload)`, `$(person)`, `$(clock)`) and shows a count where applicable.
- **Refresh Button:** Inline toolbar action on the tree view header. Triggers `GET /v6/challenges` re-fetch.
- **Login Info:** Footer section in sidebar showing current handle, sourced from the decoded JWT stored in `SecretStorage`.
- **Status Bar Items:** Two items — countdown timer (left-aligned, priority 100) and requirements progress (right-aligned, priority 50). Both are `StatusBarItem` instances disposed on deactivation.

---

## WF2: Spec & Requirements Webview Panel (Tier A)

Opens when the user clicks "Spec & Requirements" in the tree. A full-width editor-tab webview rendering the challenge specification with interactive requirements.

```
┌─────────────────────────────────────────────────────────────────────┐
│  TAB BAR                                                            │
│  [extension.ts]  [index.html]  [$(book) API Microservice Spec ✕]   │
├─────────────────────────────────────────────────────────────────────┤
│  TOOLBAR                                                            │
│  [$(refresh) Refresh]  [$(files) Attachments]  [$(clippy) Copy Spec]│
│                                                         [$(link-external) Open in Browser]│
├─────────────────────────────────────────────────────────────────────┤
│  SPEC CONTENT (Webview — markdown-rendered)                         │
│                                                                     │
│  # API Microservice Challenge                                       │
│  **Prize:** $1,500  |  **Tech:** Node.js, PostgreSQL               │
│  **Phase:** Submission  |  **Ends:** Mar 6, 2026 14:00 UTC         │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  ## Description                                                     │
│  Build a RESTful API microservice that handles user                 │
│  authentication and profile management...                           │
│  [full markdown content rendered here]                              │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│  ▼ REQUIREMENTS (5 of 8 checked)                    [$(collapse-all)]│
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ [x] REQ-1: Implement JWT authentication endpoint            │    │
│  │ [x] REQ-2: User registration with email validation          │    │
│  │ [x] REQ-3: Profile CRUD operations                          │    │
│  │ [ ] REQ-4: Rate limiting (100 req/min per user)             │    │
│  │ [x] REQ-5: PostgreSQL schema with migrations                │    │
│  │ [ ] REQ-6: Unit test coverage ≥ 80%                         │    │
│  │ [x] REQ-7: Docker Compose setup                             │    │
│  │ [ ] REQ-8: API documentation (Swagger/OpenAPI)              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
│  ▶ ATTACHMENTS (3 files)                                            │
│  ▶ SUBMISSION HISTORY                                               │
└─────────────────────────────────────────────────────────────────────┘
```

### Annotations
- **Webview Panel:** Created via `vscode.window.createWebviewPanel()` with `viewType: 'topcoder.specView'`. Opened in the editor area (column `ViewColumn.One`).
- **Toolbar:** Implemented as HTML buttons inside the webview. Communication via `postMessage()` → extension host processes commands (refresh fetches `GET /v6/challenges/{id}` again; attachments triggers download flow; copy writes spec markdown to clipboard via `vscode.env.clipboard`).
- **Spec Rendering:** Raw `description` field from challenge API is rendered with `markdown-it`. All HTML is sanitized with `sanitize-html` before injection.
- **Requirements Checklist:** Extracted from the spec body by parsing bullet lists / numbered items containing keywords like "must", "should", "required". Checkbox state persisted in `workspaceState` keyed by `challengeId`.
- **Collapsible Sections:** Pure HTML `<details>/<summary>` elements. Attachments section lists files from `GET /v5/challenges/{id}/attachments`; clicking downloads via `vscode.env.openExternal`.
- **Content Security Policy:** Webview HTML includes strict CSP: `default-src 'none'; style-src ${webview.cspSource}; script-src 'nonce-${nonce}'; img-src ${webview.cspSource} https:;`. No inline styles or scripts without nonce.

---

## WF3: Status Bar Countdown (Tier A)

Persistent status bar items showing phase countdown and requirements progress. Always visible when a challenge is selected.

```
┌─────────────────────────────────────────────────────────────────────┐
│  ... editor content ...                                             │
├─────────────────────────────────────────────────────────────────────┤
│  STATUS BAR                                                         │
│                                                                     │
│  $(clock) TC: Submission ends in 4h 12m          $(checklist) 5/8   │
│  ▲                                                ▲                 │
│  │ LEFT-ALIGNED (priority 100)                    │ RIGHT-ALIGNED   │
│  │ Click → opens Timeline webview                 │ Click → opens   │
│  │                                                │ Requirements    │
│  │                                                │ checklist       │
│  │  ┌─── TOOLTIP (on hover) ───────────────┐      │                 │
│  │  │ API Microservice Challenge            │      │                 │
│  │  │ ──────────────────────────────        │      │                 │
│  │  │ Registration: ✅ Closed               │      │                 │
│  │  │ Submission:   🟡 4h 12m remaining     │      │                 │
│  │  │ Review:       ⬜ Starts Mar 7         │      │                 │
│  │  │ Appeals:      ⬜ Starts Mar 9         │      │                 │
│  │  └──────────────────────────────────────┘      │                 │
│  │                                                │                 │
│  │  COLOR CODING:                                 │                 │
│  │  Green  = > 24h remaining                      │                 │
│  │  Yellow = 4–24h remaining                      │                 │
│  │  Red    = < 4h remaining                       │                 │
└──┴────────────────────────────────────────────────┴─────────────────┘
```

### Annotations
- **Countdown Timer:** `StatusBarItem` with `alignment: StatusBarAlignment.Left` and `priority: 100`. Text updates every 60 seconds via `setInterval` (cleared on dispose). Time remaining calculated from the current phase's `scheduledEndDate` from `GET /v6/challenges/{id}` response.
- **Color Coding:** `backgroundColor` uses `ThemeColor` — `statusBarItem.warningBackground` for yellow (4–24h), `statusBarItem.errorBackground` for red (<4h), default for green (>24h).
- **Tooltip:** Multi-line tooltip string listing all phases with status icons. Built from the `phases[]` array in the challenge detail response.
- **Click Action:** `command` property set to `topcoder.openTimeline` which opens the Timeline webview (WF6, Tier C) or scrolls to the timeline section in the spec webview.
- **Requirements Counter:** Separate `StatusBarItem` at `StatusBarAlignment.Right`. Shows `checked/total` from `workspaceState`. Click opens the requirements checklist panel.
- **Lifecycle:** Both items created in `activate()`, stored in `ExtensionContext.subscriptions` for automatic disposal.

---

## WF4: Inline Requirement Checker (Tier B)

Decorations and diagnostics overlaid on the code editor, connecting source code to spec requirements via keyword matching.

```
┌─────────────────────────────────────────────────────────────────────┐
│  TAB BAR                                                            │
│  [$(book) API Spec]  [auth.ts ●]  [server.ts]                      │
├─────────────────────────────────────────────────────────────────────┤
│  CODE EDITOR: auth.ts                                               │
│                                                                     │
│   1 │ import jwt from 'jsonwebtoken';                               │
│   2 │                                                               │
│   3 │ export async function authenticate(                           │
│   4 │   req: Request, res: Response                                 │
│   5 │ ) {                                                           │
│   6 │   const token = req.headers.authorization;                    │
│   7 │   // ... JWT verification logic                               │
│   8 │ }  ◀── HOVER ─────────────────────────────────────────┐      │
│   9 │                │  $(info) Linked Requirement           │      │
│  10 │ export async   │  ─────────────────────────────────    │      │
│  11 │   function     │  REQ-1: Implement JWT authentication  │      │
│  12 │   register(    │  endpoint with token generation and   │      │
│  13 │   ...          │  validation.                          │      │
│  14 │                │  Status: ✅ Checked                   │      │
│  15 │                │  [Toggle Checked] [Open in Spec]      │      │
│  16 │                └───────────────────────────────────────┘      │
│                                                                     │
│  GUTTER DECORATIONS:                                                │
│  Lines 3-8:  $(pass) green bar  (req matched & checked)             │
│  Lines 10-20: $(circle-outline) gray bar (req matched, unchecked)   │
│  Lines 25+:  (no decoration — no requirement linked)                │
├─────────────────────────────────────────────────────────────────────┤
│  PROBLEMS PANEL                                          [Filter ▾] │
│  ─────────────────────────────────────────────────────────────────  │
│  ⚠ Topcoder Requirements (3 uncovered)                              │
│    $(warning) REQ-4: Rate limiting — no matching code found         │
│               → Expected keywords: rate, limit, throttle            │
│    $(warning) REQ-6: Unit test coverage — no test files detected    │
│               → Expected: *.test.ts, *.spec.ts files                │
│    $(warning) REQ-8: API documentation — no swagger/openapi found   │
│               → Expected: swagger, openapi, @ApiProperty            │
└─────────────────────────────────────────────────────────────────────┘
```

### Annotations
- **Hover Provider:** Registered via `vscode.languages.registerHoverProvider('*', ...)`. When triggered, scans the hovered line's text for keywords extracted from requirements. If a match is found, returns a `Hover` with requirement text, status, and action links (command URIs).
- **Gutter Decorations:** `TextEditorDecorationType` instances for two states: matched+checked (green) and matched+unchecked (gray). Applied via `editor.setDecorations()` after scanning the document.
- **Keyword Matching:** Each requirement is pre-processed into a keyword set (nouns + technical terms extracted via simple tokenization). A code line matches if it contains ≥2 keywords from any requirement. Matching is case-insensitive.
- **Problems Panel:** `DiagnosticCollection` created via `vscode.languages.createDiagnosticCollection('topcoder')`. Uncovered requirements appear as `DiagnosticSeverity.Warning`. Diagnostics point to a synthetic range (line 1, col 1 of the active file) since they are project-level, not line-level.
- **Command: "Topcoder: Check Requirements":** Scans all workspace files (respecting `.gitignore`) for keyword matches, then updates the `DiagnosticCollection` with uncovered requirements.
- **Performance:** File scanning is debounced (500ms after last edit) and runs in a `withProgress` wrapper to show progress in the notification area. Uses `CancellationToken` to abort if the user triggers another scan.

---

## WF5: Requirements Checklist Panel (Tier B)

A dedicated tree view or webview showing all extracted requirements as a tracked checklist with progress visualization.

```
┌─────────────────────────────────────────────────────────────────────┐
│  SIDEBAR: TOPCODER                                                  │
├─────────────────────────────────────────────────────────────────────┤
│  MY ACTIVE CHALLENGES                                [$(refresh)]   │
│  ─────────────────────────────────────────────────────────────────  │
│  [collapsed tree — see WF1]                                         │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│  REQUIREMENTS CHECKLIST                    [$(export) Export] [$(refresh)]│
│  ─────────────────────────────────────────────────────────────────  │
│  API Microservice Challenge                                         │
│                                                                     │
│  Progress: ████████░░░░░░░░ 62% (5/8)                              │
│                                                                     │
│  [x] REQ-1  JWT authentication endpoint          $(pass-filled)     │
│      Matched: auth.ts:3, auth.ts:22                                 │
│  [x] REQ-2  User registration + email validation $(pass-filled)     │
│      Matched: register.ts:15                                        │
│  [x] REQ-3  Profile CRUD operations              $(pass-filled)     │
│      Matched: profile.controller.ts:8                               │
│  [ ] REQ-4  Rate limiting (100 req/min)           $(circle-outline) │
│      ⚠ No matching code found                                       │
│  [x] REQ-5  PostgreSQL schema + migrations        $(pass-filled)    │
│      Matched: migrations/*.sql (4 files)                            │
│  [ ] REQ-6  Unit test coverage ≥ 80%              $(circle-outline) │
│      ⚠ No test files detected                                       │
│  [x] REQ-7  Docker Compose setup                  $(pass-filled)    │
│      Matched: docker-compose.yml                                    │
│  [ ] REQ-8  API documentation (Swagger)           $(circle-outline) │
│      ⚠ No swagger/openapi files found                               │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│  [$(run-all) Re-scan Workspace]  [$(clear-all) Reset Checks]       │
└─────────────────────────────────────────────────────────────────────┘
```

### Annotations
- **View Registration:** Registered as a second view (`topcoder.requirementsView`) under the same `viewsContainers.activitybar` entry as the challenge explorer. Uses `TreeDataProvider` with `TreeItem` nodes.
- **Progress Bar:** Rendered as a text-based progress indicator in the tree view's header description, or as HTML in a webview variant. Percentage calculated from checked/total requirements.
- **Checkbox Toggle:** `TreeItem.checkboxState` (VS Code 1.79+) for native checkbox support. State changes fire `onDidChangeCheckboxState` event → persist to `workspaceState`.
- **Code Match Lines:** Child `TreeItem` nodes under each requirement showing file paths where keywords matched. Clicking navigates to the file and line (`vscode.commands.executeCommand('vscode.open', uri, { selection })`).
- **Export Command:** `topcoder.exportChecklist` writes a markdown checklist to clipboard or to a `CHECKLIST.md` file in the workspace root.
- **Re-scan:** Triggers the full workspace keyword scan (same as WF4's "Check Requirements" command), then refreshes the tree.
- **Persistence:** Checkbox states stored in `context.workspaceState.update('topcoder.checklist.' + challengeId, checkedIds[])`. Survives VS Code restarts.

---

## WF6: Forum & Timeline Sidebar (Tier C)

A split webview panel combining threaded forum posts and a visual timeline bar for challenge phases.

```
┌─────────────────────────────────────────────────────────────────────┐
│  TAB BAR (Webview — secondary sidebar or editor tab)                │
│  [$(comment-discussion) Forum & Timeline — API Microservice  ✕]    │
├─────────────────────────────────────────────────────────────────────┤
│  TIMELINE                                              [$(refresh)] │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  Registration    Submission       Review        Appeals             │
│  ✅ ━━━━━━━━━━━  🟡 ━━━━━━━━━━━  ⬜ ─────────  ⬜ ─────────       │
│  Feb 28 – Mar 2  Mar 2 – Mar 6   Mar 7 – 10    Mar 10 – 12        │
│                  ▲ YOU ARE HERE                                      │
│                  4h 12m remaining                                    │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│  FORUM POSTS (12 posts)          [$(refresh)] [Auto: ON $(sync)]    │
│  Sort: [Latest First ▾]          Filter: [All ▾]                    │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  ┌─── Post #12 ─────────────────────────────────────────────────┐  │
│  │ $(person) copilot_sarah  •  2h ago  •  $(star) Copilot       │  │
│  │ ─────────────────────────────────────────────────────         │  │
│  │ Clarification: Rate limiting should be per-IP, not per-user. │  │
│  │ Please update REQ-4 accordingly.                              │  │
│  │                                           [$(reply) Reply ↗]  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌─── Post #11 ─────────────────────────────────────────────────┐  │
│  │ $(person) dev_mike  •  5h ago  •  $(account) Member          │  │
│  │ ─────────────────────────────────────────────────────         │  │
│  │ Q: Should the JWT tokens use RS256 or HS256?                  │  │
│  │                                           [$(reply) Reply ↗]  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌─── Post #10 ─────────────────────────────────────────────────┐  │
│  │ $(person) copilot_sarah  •  6h ago  •  $(star) Copilot       │  │
│  │ ─────────────────────────────────────────────────────         │  │
│  │ Welcome! Please read the full spec carefully. Key points:     │  │
│  │ 1. PostgreSQL required (not MySQL)                            │  │
│  │ 2. Docker Compose must include all services                   │  │
│  │ 3. Submit via Topcoder CLI or web                             │  │
│  │                                           [$(reply) Reply ↗]  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  [$(chevron-down) Load older posts...]                              │
│                                                                     │
│  ─────────────────────────────────────────────────────────────────  │
│  $(info) Auto-refresh: every 120s  •  Last updated: 2 min ago      │
└─────────────────────────────────────────────────────────────────────┘
```

### Annotations
- **Webview Panel:** Created via `createWebviewPanel()` with `viewType: 'topcoder.forumTimeline'`. Can be opened in the editor area or the secondary sidebar (VS Code 1.85+ `ViewColumn.Beside`).
- **Timeline Bar:** Horizontal progress bar rendered in HTML/CSS. Each phase is a `<div>` segment with width proportional to its duration. Colors: green (completed), yellow/animated (active), gray (upcoming). Data sourced from `phases[]` in `GET /v6/challenges/{id}`.
- **Forum Posts:** Fetched via `GET /v5/challenge-discussions?challengeId={id}` (or polled from the challenge object's discussion metadata). Each post rendered as a card with author avatar placeholder, handle, timestamp (relative via `date-fns`), role badge, and body text.
- **Auto-Poll:** Configurable interval (default 120s, range 60–300s) via `topcoder.pollInterval` setting. A `setInterval` re-fetches the discussion endpoint; new posts trigger a badge count update on the tree node and an optional `showInformationMessage` notification.
- **Reply Link:** Opens the challenge forum in the browser via `vscode.env.openExternal(forumUrl)` — the plugin is read-only, no in-IDE posting.
- **Sort/Filter:** Client-side controls in the webview. Sort by date (latest/oldest), filter by author role (Copilot/Member/All). State managed in webview JavaScript.
- **Load More:** Pagination via `offset` query param on the discussions API. "Load older" appends posts to the DOM.
- **Polling Indicator:** "Auto: ON" badge with sync icon. Shows last-refreshed timestamp. Togglable via a webview button that sends a `postMessage` to the extension host to start/stop the interval.

---

## Wireframe Summary Matrix

| Wireframe | Tier | VS Code API Used | Primary API Call |
|-----------|------|-------------------|-----------------|
| WF1: Challenge Explorer | A | `TreeDataProvider`, `StatusBarItem` | `GET /v6/challenges` |
| WF2: Spec Webview | A | `WebviewPanel`, `env.clipboard` | `GET /v6/challenges/{id}` |
| WF3: Status Bar | A | `StatusBarItem`, `ThemeColor` | `GET /v6/challenges/{id}` (phases) |
| WF4: Inline Checker | B | `HoverProvider`, `DiagnosticCollection`, `TextEditorDecorationType` | Workspace file scan |
| WF5: Checklist Panel | B | `TreeDataProvider` (checkboxState), `workspaceState` | Workspace file scan |
| WF6: Forum & Timeline | C | `WebviewPanel`, `setInterval` | `GET /v5/challenge-discussions` |
