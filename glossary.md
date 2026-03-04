# Glossary — Topcoder VSCode Plugin

> **Purpose:** Developer-facing reference explaining every UI element, user interaction, and data concept used in the plugin design. Organized by category and cross-referenced to wireframes and tiers.  
> **Convention:** "API" column references the VS Code Extension API class/method. "Tier" indicates which feature tier (A/B/C) the element belongs to. "Wireframe" maps to the wireframe number in `wireframes.md`.

---

## 1. UI Elements

| # | Element | Type | Description | VS Code API | Tier | Wireframe |
|---|---------|------|-------------|-------------|------|-----------|
| 1 | **Topcoder Activity Bar Icon** | Icon | Custom icon ("TC" logo) added to the Activity Bar (left vertical strip). Clicking activates the Topcoder sidebar container. Registered via `viewsContainers.activitybar` in `package.json` with a reference to a 24×24 SVG icon in `media/`. | `package.json` contribution point | A | WF1 |
| 2 | **Challenge Explorer Tree View** | Tree View | Primary sidebar view listing the user's active/upcoming Topcoder challenges. Each root node is a challenge; expanding reveals child items (Spec, Requirements, Attachments, Forum, Timeline). Implemented as a class extending `TreeDataProvider<ChallengeTreeItem>`. | `vscode.window.registerTreeDataProvider()`, `TreeDataProvider`, `TreeItem` | A | WF1 |
| 3 | **Challenge Tree Node** | Tree Item | Root-level node representing one challenge. `label` = challenge name (e.g., "API Microservice"). `description` = status string ("Active" / "Upcoming"). `iconPath` = trophy icon. `collapsibleState` = `Collapsed`. `contextValue` = `'challenge'` (enables right-click context menu commands). | `TreeItem`, `TreeItemCollapsibleState` | A | WF1 |
| 4 | **Status Badge** | Inline Text | Short text appended after the challenge name showing current status. Rendered via `TreeItem.description`. Color is inherited from theme; no custom coloring at tree-item level (VS Code limitation). | `TreeItem.description` | A | WF1 |
| 5 | **Spec & Requirements Node** | Tree Item (Child) | Child node under a challenge. `label` = "Spec & Requirements", `iconPath` = `$(book)`. Clicking triggers `topcoder.openSpec` command which opens the Spec Webview (WF2). `command` property set on the `TreeItem`. | `TreeItem.command` | A | WF1, WF2 |
| 6 | **Requirements Node** | Tree Item (Child) | Child node showing "Requirements (5/8)". Count reflects checked/total from `workspaceState`. Clicking opens the Requirements Checklist view (WF5) or scrolls to the requirements section in the Spec Webview. | `TreeItem`, `TreeItem.description` | A, B | WF1, WF5 |
| 7 | **Submissions Node** | Tree Item (Child) | Child node showing "Submissions". Clicking opens a quick-pick list showing your submission history (status, date). Each selection triggers `vscode.env.openExternal()` with the submission URL. | `TreeItem.command`, `vscode.window.showQuickPick()` | A | WF1 |
| 8 | **Discussions Node** | Tree Item (Child) | Child node showing "Discussions (12)". Count from the discussions API. Clicking opens the Forum & Timeline webview (WF6). Badge updates on new posts (polling). | `TreeItem`, `TreeItem.description` | C | WF1, WF6 |
| 9 | **Timeline Node** | Tree Item (Child) | Child node labeled "Timeline". Clicking opens the Timeline section of WF6. Shows phase summary in tooltip. | `TreeItem.tooltip` | C | WF1, WF6 |
| 10 | **Refresh Button (Tree Header)** | Inline Toolbar Action | Button in the tree view title bar area. Icon: `$(refresh)`. Triggers re-fetch of `GET /v5/challenges` and refreshes the tree data provider via `_onDidChangeTreeData.fire()`. | `view/title` command contribution, `EventEmitter` | A | WF1 |
| 11 | **Login Info Footer** | Tree Item | Bottom node in the explorer showing "Logged in as: {handle}". Non-collapsible, no command. Handle decoded from the JWT stored in `SecretStorage`. If not logged in, shows "Not logged in — click to authenticate" with a command to trigger login flow. | `TreeItem` (contextValue: `'auth'`) | A | WF1 |
| 12 | **Spec Webview Panel** | Webview Panel | Full editor-tab panel rendering the challenge specification as formatted HTML. Created via `createWebviewPanel('topcoder.specView', title, ViewColumn.One, options)`. Options include `enableScripts: true`, `localResourceRoots: [extensionUri]`, strict CSP. Disposed when tab is closed. | `vscode.window.createWebviewPanel()`, `WebviewPanel` | A | WF2 |
| 13 | **Webview Toolbar** | HTML Element | Horizontal button strip at the top of the Spec Webview. Contains: Refresh, Open Attachments, Copy Spec, Open in Browser. Each button sends a `postMessage({ command: '...' })` to the extension host. Styled with `@vscode/webview-ui-toolkit` `<vscode-button>` components. | Webview `postMessage()` / `onDidReceiveMessage` | A | WF2 |
| 14 | **Spec Content Area** | HTML Element | Main body of the Spec Webview. Challenge `description` (markdown) rendered to HTML via `markdown-it` library. Sanitized with `sanitize-html` to strip dangerous tags/attributes. Supports headings, code blocks, tables, images (proxied through webview CSP). | `markdown-it`, `sanitize-html` | A | WF2 |
| 15 | **Requirements Checklist (Webview)** | HTML Element | Collapsible `<details>` section within the Spec Webview. Lists requirements as `<input type="checkbox">` items. Checkbox changes send `postMessage({ command: 'toggleRequirement', id, checked })` to extension host, which persists to `workspaceState`. | Webview messaging, `ExtensionContext.workspaceState` | A, B | WF2, WF5 |
| 16 | **Countdown Status Bar Item** | Status Bar Item | Left-aligned status bar item showing "$(clock) TC: Submission ends in Xh Ym". `backgroundColor` changes by urgency: default (>24h), `warningBackground` (4–24h), `errorBackground` (<4h). Updates every 60s via `setInterval`. Click opens Timeline (WF6). | `vscode.window.createStatusBarItem(StatusBarAlignment.Left, 100)`, `StatusBarItem` | A | WF3 |
| 17 | **Requirements Counter Status Bar Item** | Status Bar Item | Right-aligned status bar item showing "$(checklist) 5/8". Reflects real-time checklist state from `workspaceState`. Click opens requirements panel (WF5). | `vscode.window.createStatusBarItem(StatusBarAlignment.Right, 50)`, `StatusBarItem` | B | WF3 |
| 18 | **Hover Decoration (Requirement Link)** | Hover | Hover tooltip appearing over code lines that match requirement keywords. Shows requirement ID, text, checked status, and two action links: "Toggle Checked" and "Open in Spec". Implemented via `HoverProvider`. | `vscode.languages.registerHoverProvider()`, `Hover`, `MarkdownString` | B | WF4 |
| 19 | **Gutter Decoration (Green Bar)** | Editor Decoration | Green background applied to the gutter region of code lines matching a checked requirement. Created via `createTextEditorDecorationType({ gutterIconPath, overviewRulerColor: 'green' })`. | `vscode.window.createTextEditorDecorationType()`, `TextEditorDecorationType` | B | WF4 |
| 20 | **Gutter Decoration (Gray Bar)** | Editor Decoration | Gray background for lines matching an unchecked requirement. Same mechanism as #19, different color/icon. | `TextEditorDecorationType` | B | WF4 |
| 21 | **Diagnostic Entry** | Diagnostic | Warning-level entry in the Problems panel for requirements with no matching code found in the workspace. Message includes the requirement text and expected keywords. Severity: `DiagnosticSeverity.Warning`. | `vscode.languages.createDiagnosticCollection('topcoder')`, `Diagnostic` | B | WF4 |
| 22 | **Requirements Checklist Tree View** | Tree View | Dedicated sidebar tree view (below the Challenge Explorer) showing requirements as checkable items with progress. Uses `TreeItem.checkboxState` (VS Code 1.79+). | `TreeDataProvider`, `TreeItemCheckboxState` | B | WF5 |
| 23 | **Progress Bar** | Text/HTML | Visual progress indicator showing percentage of checked requirements. In tree view: rendered as description text ("████░░░░ 62%"). In webview: HTML `<progress>` element. | `TreeItem.description` or HTML `<progress>` | B | WF5 |
| 24 | **Code Match Child Node** | Tree Item (Child) | Child node under a requirement in the checklist tree. Shows filename and line number where the requirement's keywords were matched. Clicking navigates to that location. | `TreeItem.command` → `vscode.open` with `TextDocumentShowOptions.selection` | B | WF5 |
| 25 | **Forum & Timeline Webview** | Webview Panel | Split-layout webview showing timeline bar (top) and forum posts (bottom). Can be opened in editor area or secondary sidebar. Strict CSP applied. | `vscode.window.createWebviewPanel()`, `WebviewPanel` | C | WF6 |
| 26 | **Timeline Phase Bar** | HTML Element | Horizontal bar divided into segments per challenge phase. Each segment has: label, date range, status icon (✅/🟡/⬜), proportional width based on duration. Active phase has animated pulse CSS. | HTML/CSS (flexbox), data from `phases[]` | C | WF6 |
| 27 | **Forum Post Card** | HTML Element | Card component displaying one forum post. Contains: author handle, relative timestamp (e.g., "2h ago"), role badge (Copilot/Member), post body text, and a "Reply ↗" link that opens the browser. | HTML/CSS, `date-fns.formatDistanceToNow()` | C | WF6 |
| 28 | **Auto-Poll Indicator** | HTML Element | Toggle badge showing "Auto: ON/OFF" with a sync icon. Displays last-refresh timestamp. Toggling sends `postMessage({ command: 'togglePoll' })` to the extension host. | Webview messaging, `setInterval` / `clearInterval` | C | WF6 |
| 29 | **Load More Button** | HTML Element | Button below the last rendered forum post. Clicking increments the `offset` parameter for the next `GET /v5/challenge-discussions` call and appends results. | Webview messaging, API pagination | C | WF6 |

---

## 2. User Interactions

| # | Interaction | Trigger | Result | Related Element(s) | Tier |
|---|-------------|---------|--------|---------------------|------|
| 30 | **Login** | Command Palette: "Topcoder: Login" | Opens browser for OAuth2 authorization. On callback, stores JWT in `SecretStorage`. Updates login footer in tree. Refreshes challenge list. | Login Info Footer (#11) | A |
| 31 | **Logout** | Command Palette: "Topcoder: Logout" | Deletes JWT from `SecretStorage`. Clears tree view. Hides status bar items. Shows "Not logged in" in footer. | Login Info Footer (#11) | A |
| 32 | **Select Challenge** | Click challenge tree node | Sets the "active challenge" in extension state. Triggers `GET /v5/challenges/{id}` fetch. Updates status bar countdown (WF3) and requirements counter. Expands the node to show children. | Challenge Tree Node (#3), Status Bar (#16, #17) | A |
| 33 | **Open Spec** | Click "Spec & Requirements" child node | Creates or reveals the Spec Webview panel. Populates with rendered markdown from the challenge `description` field. | Spec & Requirements Node (#5), Spec Webview (#12) | A |
| 34 | **Refresh Challenge Data** | Click $(refresh) button in tree header, or Command Palette: "Topcoder: Refresh" | Re-fetches `GET /v5/challenges` (list) and `GET /v5/challenges/{id}` (active challenge). Fires `_onDidChangeTreeData` to update the tree. Updates webview if open. | Refresh Button (#10) | A |
| 35 | **Toggle Requirement Checkbox** | Click checkbox in Spec Webview or Requirements Checklist tree | Sends `postMessage` (webview) or fires `onDidChangeCheckboxState` (tree). Extension host updates `workspaceState` and refreshes the requirements counter status bar item. | Requirements Checklist (#15, #22), Status Bar (#17) | A, B |
| 36 | **Download Attachment** | Click attachment in quick-pick or Attachments section | Fetches attachment metadata from `GET /v5/challenges/{id}/attachments`. Opens download URL via `vscode.env.openExternal()` (browser download) or streams to workspace via `fs.writeFile`. | Attachments Node (#7) | A |
| 37 | **Copy Spec to Clipboard** | Click "Copy Spec" button in webview toolbar | Extension host receives `postMessage({ command: 'copySpec' })`. Reads raw markdown from cached challenge data. Writes to clipboard via `vscode.env.clipboard.writeText()`. Shows confirmation via `showInformationMessage`. | Webview Toolbar (#13) | A |
| 38 | **Open in Browser** | Click "Open in Browser" in webview toolbar | Constructs the Topcoder challenge URL (`https://www.topcoder.com/challenges/{id}`) and opens via `vscode.env.openExternal()`. | Webview Toolbar (#13) | A |
| 39 | **Check Requirements (Scan)** | Command Palette: "Topcoder: Check Requirements" | Scans all workspace files for requirement keyword matches. Updates `DiagnosticCollection` with uncovered requirements (WF4). Updates gutter decorations. Runs inside `withProgress()` with `CancellationToken`. | Diagnostics (#21), Gutter Decorations (#19, #20) | B |
| 40 | **Hover Over Code** | Mouse hover on a code line | `HoverProvider` checks the line against requirement keyword sets. If matched, shows a `Hover` with requirement details, status, and action command links. | Hover Decoration (#18) | B |
| 41 | **Navigate to Code Match** | Click file/line child node in Requirements Checklist | Executes `vscode.commands.executeCommand('vscode.open', uri, { selection: range })`. Opens the file and highlights the matching line. | Code Match Child Node (#24) | B |
| 42 | **Export Checklist** | Click "Export" button in Requirements Checklist header | Generates a markdown checklist string from current state. User chooses: copy to clipboard or save as `CHECKLIST.md` in workspace root. Uses `showSaveDialog` for file option. | Requirements Checklist Tree (#22) | B |
| 43 | **Re-scan Workspace** | Click "Re-scan Workspace" button in checklist | Same as #39 but also refreshes the Requirements Checklist tree view by firing its `_onDidChangeTreeData`. | Requirements Checklist Tree (#22) | B |
| 44 | **View Forum** | Click "Discussions" child node in tree | Opens/reveals the Forum & Timeline webview (WF6). Fetches `GET /v5/challenge-discussions`. Renders posts as cards. Starts auto-poll if enabled. | Discussions Node (#8), Forum Webview (#25) | C |
| 45 | **Toggle Auto-Poll** | Click "Auto: ON/OFF" badge in Forum webview | Sends `postMessage({ command: 'togglePoll' })` to extension host. Toggles `setInterval`/`clearInterval`. Updates badge text and last-refresh timestamp. | Auto-Poll Indicator (#28) | C |
| 46 | **Load Older Posts** | Click "Load older posts..." in Forum webview | Extension host increments `offset` parameter, calls `GET /v5/challenge-discussions` again. New posts appended to DOM via `postMessage` back to webview. | Load More Button (#29), Forum Post Card (#27) | C |
| 47 | **Reply to Post (External)** | Click "Reply ↗" link on a forum card | Opens the Topcoder challenge forum page in browser via `vscode.env.openExternal()`. Plugin is read-only; replies happen in the browser. | Forum Post Card (#27) | C |
| 48 | **Click Countdown Timer** | Click status bar countdown | Opens the Timeline section — either reveals the Forum & Timeline webview (WF6) if Tier C is enabled, or shows a `showInformationMessage` with phase details if only Tier A. | Countdown Status Bar (#16) | A, C |

---

## 3. Data Concepts

| # | Concept | Type | Description | Source | Used By |
|---|---------|------|-------------|--------|---------|
| 49 | **JWT (JSON Web Token)** | Auth Token | Bearer token obtained via OAuth2 device flow or browser redirect from `https://accounts-auth0.topcoder.com`. Stored exclusively in `context.secrets` (VS Code `SecretStorage`). Attached as `Authorization: Bearer {jwt}` header on all API calls. Contains encoded `handle`, `userId`, and expiry (`exp`). Never logged or displayed in output channels. | Auth0 / Topcoder OAuth2 | All API calls |
| 50 | **Challenge Object** | API Response | JSON object from `GET /v5/challenges/{id}`. Key fields: `id`, `name`, `description` (markdown spec), `status` (Active/Upcoming/Completed), `phases[]`, `prizeSets[]`, `tags[]`, `attachments[]`, `numOfRegistrants`, `numOfSubmissions`. Cached in `globalState` with a 5-minute TTL. | `GET /v5/challenges/{id}` | WF1, WF2, WF3 |
| 51 | **Phase Object** | Nested Object | Entry in `challenge.phases[]`. Fields: `name` (Registration/Submission/Review/Appeals), `isOpen` (boolean), `scheduledStartDate`, `scheduledEndDate`, `duration` (ms). Used to build the timeline bar (WF6) and calculate countdown (WF3). | `GET /v5/challenges/{id}` → `phases[]` | WF3, WF6 |
| 52 | **Attachment/Resource** | API Response | File metadata from `GET /v5/challenges/{id}/attachments`. Fields: `id`, `name`, `url` (download link), `size`, `fileType`. Displayed in Attachments section of Spec Webview and as child count on Attachments tree node. | `GET /v5/challenges/{id}/attachments` | WF1, WF2 |
| 53 | **Discussion Thread** | API Response | Forum post object from `GET /v5/challenge-discussions`. Fields: `id`, `challengeId`, `authorHandle`, `authorRole` (Copilot/Member/Manager), `body` (text/HTML), `createdAt`, `updatedAt`. Rendered as Post Cards in WF6. Paginated with `limit`/`offset`. | `GET /v5/challenge-discussions` | WF6 |
| 54 | **Submission Metadata** | API Response | Object from `GET /v5/submissions?challengeId={id}&memberId={handle}`. Fields: `id`, `type`, `url`, `createdAt`, `status` (Active/Completed). Shown optionally in the Spec Webview under "Submission History". | `GET /v5/submissions` | WF2 |
| 55 | **Requirement Item** | Derived Data | Parsed from the challenge spec body. Fields (internal): `id` (sequential, e.g., REQ-1), `text` (requirement sentence), `keywords` (tokenized technical terms for matching), `checked` (boolean, from `workspaceState`). Used across Tier A (display) and Tier B (matching/checking). | Parsed from `challenge.description` | WF2, WF4, WF5 |
| 56 | **Keyword Match Result** | Derived Data | Output of the workspace scan. Fields: `requirementId`, `filePath`, `lineNumber`, `matchedKeywords[]`, `confidence` (number of keyword hits). Used to populate gutter decorations (WF4) and code match child nodes (WF5). | Workspace file scan | WF4, WF5 |
| 57 | **Plugin Configuration** | Settings Object | User-configurable settings accessed via `workspace.getConfiguration('topcoder')`. Keys: `apiBaseUrl` (default: `https://api.topcoder.com`), `pollInterval` (default: 120, range: 60–300), `enableTierB` (boolean), `enableTierC` (boolean), `maxChallenges` (default: 50). Defined in `package.json` under `contributes.configuration`. | `vscode.workspace.getConfiguration()` | All |
| 58 | **Cache Entry** | Internal State | Data stored in `globalState` with a TTL pattern. Key format: `topcoder.cache.{type}.{id}`. Value: `{ data: T, expiresAt: number }`. Types: `challengeList` (TTL 5 min), `challengeDetail` (TTL 5 min), `discussions` (TTL 60s). A helper function `getCachedOrFetch<T>()` wraps fetch calls with cache-check logic. | `ExtensionContext.globalState` | All API calls |
| 59 | **Checklist State** | Workspace State | Persisted per-challenge checkbox data. Key: `topcoder.checklist.{challengeId}`. Value: `string[]` (list of checked requirement IDs). Survives VS Code restarts because `workspaceState` is persistent. | `ExtensionContext.workspaceState` | WF2, WF5, WF3 |

---

## 4. Quick Reference — VS Code API Usage Map

| VS Code API | Elements Using It |
|-------------|-------------------|
| `TreeDataProvider` / `TreeItem` | Challenge Explorer (#2, #3), Requirements Checklist (#22) |
| `WebviewPanel` | Spec Webview (#12), Forum & Timeline (#25) |
| `StatusBarItem` | Countdown (#16), Requirements Counter (#17) |
| `HoverProvider` | Inline Requirement Hover (#18) |
| `DiagnosticCollection` | Uncovered Requirements Warnings (#21) |
| `TextEditorDecorationType` | Gutter Decorations (#19, #20) |
| `SecretStorage` | JWT Storage (#49) |
| `globalState` / `workspaceState` | Cache (#58), Checklist State (#59) |
| `env.clipboard` | Copy Spec (#37) |
| `env.openExternal` | Attachments (#36), Reply (#47), Open in Browser (#38) |
| `workspace.getConfiguration` | Plugin Settings (#57) |
| `window.withProgress` | Requirement Scan (#39) |
| `CancellationToken` | All async API calls |
