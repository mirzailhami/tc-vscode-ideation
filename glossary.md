# Glossary — Topcoder VSCode Plugin

> **Purpose:** Developer-facing reference explaining every UI element, user interaction, and data concept used in the plugin design. Organized by category and cross-referenced to wireframes and tiers.  
> **Convention:** "API" column references the VS Code Extension API class/method. "Tier" indicates which feature tier (A/B) the element belongs to. "Wireframe" maps to the wireframe number in `wireframes.md`.

---

## 1. UI Elements

| # | Element | Type | Description | VS Code API | Tier | Wireframe |
|---|---------|------|-------------|-------------|------|-----------|
| 1 | **Topcoder Activity Bar Icon** | Icon | Custom icon ("TC" logo) added to the Activity Bar (left vertical strip). Clicking activates the Topcoder sidebar container. Registered via `viewsContainers.activitybar` in `package.json` with a reference to a 24×24 SVG icon in `media/`. | `package.json` contribution point | A | WF1 |
| 2 | **Challenge Explorer Tree View** | Tree View | Primary sidebar view listing the user's active/upcoming Topcoder challenges. Each root node is a challenge; expanding reveals child items (Spec, Requirements, Attachments, Forum, Timeline). Implemented as a class extending `TreeDataProvider<ChallengeTreeItem>`. | `vscode.window.registerTreeDataProvider()`, `TreeDataProvider`, `TreeItem` | A | WF1 |
| 3 | **Challenge Tree Node** | Tree Item | Root-level node representing one challenge. `label` = challenge name (e.g., "API Microservice"). `description` = status string ("ACTIVE" / "DRAFT"). `iconPath` = trophy icon. `collapsibleState` = `Collapsed`. `contextValue` = `'challenge'` (enables right-click context menu commands). | `TreeItem`, `TreeItemCollapsibleState` | A | WF1 |
| 4 | **Status Badge** | Inline Text | Short text appended after the challenge name showing current status. Rendered via `TreeItem.description`. Color is inherited from theme; no custom coloring at tree-item level (VS Code limitation). | `TreeItem.description` | A | WF1 |
| 5 | **Spec & Requirements Node** | Tree Item (Child) | Child node under a challenge. `label` = "Spec & Requirements", `iconPath` = `$(book)`. Clicking triggers `topcoder.openSpec` command which opens the Spec Webview (WF2). `command` property set on the `TreeItem`. | `TreeItem.command` | A | WF1, WF2 |
| 6 | **Requirements Node** | Tree Item (Child) | Child node showing "Requirements (5/8)". Count reflects checked/total from `workspaceState`. Clicking scrolls to the requirements section in the Spec Webview (WF2). | `TreeItem`, `TreeItem.description` | A | WF1, WF2 |
| 7 | **Submissions Node** | Tree Item (Child) | Child node showing "Submissions". Clicking opens a quick-pick list showing your submission history (status, date). Each selection triggers `vscode.env.openExternal()` with the submission URL. | `TreeItem.command`, `vscode.window.showQuickPick()` | A | WF1 |
| 8 | **Discussions Node** | Tree Item (Child) | Child node showing "Discussions (12)". Count from the discussions API. Clicking opens the Forum & Timeline webview (WF6). Badge updates on new posts (polling). | `TreeItem`, `TreeItem.description` | B | WF1, WF6 |
| 9 | **Timeline Node** | Tree Item (Child) | Child node labeled "Timeline". Clicking opens the Timeline section of WF6. Shows phase summary in tooltip. | `TreeItem.tooltip` | B | WF1, WF6 |
| 10 | **Refresh Button (Tree Header)** | Inline Toolbar Action | Button in the tree view title bar area. Icon: `$(refresh)`. Triggers re-fetch of `GET /v6/challenges` and refreshes the tree data provider via `_onDidChangeTreeData.fire()`. | `view/title` command contribution, `EventEmitter` | A | WF1 |
| 11 | **Login Info Footer** | Tree Item | Bottom node in the explorer showing "Logged in as: {handle}". Non-collapsible, no command. Handle decoded from the JWT stored in `SecretStorage`. If not logged in, shows "Not logged in — click to authenticate" with a command to trigger login flow. | `TreeItem` (contextValue: `'auth'`) | A | WF1 |
| 12 | **Spec Webview Panel** | Webview Panel | Full editor-tab panel rendering the challenge specification as formatted HTML. Created via `createWebviewPanel('topcoder.specView', title, ViewColumn.One, options)`. Options include `enableScripts: true`, `localResourceRoots: [extensionUri]`, strict CSP. Disposed when tab is closed. | `vscode.window.createWebviewPanel()`, `WebviewPanel` | A | WF2 |
| 13 | **Webview Toolbar** | HTML Element | Horizontal button strip at the top of the Spec Webview. Contains: Refresh, Open Attachments, Copy Spec, Open in Browser. Each button sends a `postMessage({ command: '...' })` to the extension host. Styled with `@vscode/webview-ui-toolkit` `<vscode-button>` components. | Webview `postMessage()` / `onDidReceiveMessage` | A | WF2 |
| 14 | **Spec Content Area** | HTML Element | Main body of the Spec Webview. Challenge `description` (markdown) rendered to HTML via `markdown-it` library. Sanitized with `sanitize-html` to strip dangerous tags/attributes. Supports headings, code blocks, tables, images (proxied through webview CSP). | `markdown-it`, `sanitize-html` | A | WF2 |
| 15 | **Requirements Checklist (Webview)** | HTML Element | Collapsible `<details>` section within the Spec Webview. Lists requirements as `<input type="checkbox">` items for personal progress tracking. Checkbox changes send `postMessage({ command: 'toggleRequirement', id, checked })` to extension host, which persists to `workspaceState`. Not an automated assessment — a personal productivity aid. | Webview messaging, `ExtensionContext.workspaceState` | A | WF2 |
| 16 | **Countdown Status Bar Item** | Status Bar Item | Left-aligned status bar item showing "$(clock) TC: Submission ends in Xh Ym". `backgroundColor` changes by urgency: default (>24h), `warningBackground` (4–24h), `errorBackground` (<4h). Updates every 60s via `setInterval`. Click opens Timeline (WF6). | `vscode.window.createStatusBarItem(StatusBarAlignment.Left, 100)`, `StatusBarItem` | A | WF3 |
| 17 | **Requirements Counter Status Bar Item** | Status Bar Item | Right-aligned status bar item showing "$(checklist) 5/8". Reflects real-time checklist state from `workspaceState`. Click opens requirements section in the Spec Webview (WF2). | `vscode.window.createStatusBarItem(StatusBarAlignment.Right, 50)`, `StatusBarItem` | A | WF3 |
| 25 | **Forum & Timeline Webview** | Webview Panel | Split-layout webview showing timeline bar (top) and forum posts (bottom). Can be opened in editor area or secondary sidebar. Strict CSP applied. | `vscode.window.createWebviewPanel()`, `WebviewPanel` | B | WF6 |
| 26 | **Timeline Phase Bar** | HTML Element | Horizontal bar divided into segments per challenge phase. Each segment has: label, date range, status icon (✅/🟡/⬜), proportional width based on duration. Active phase has animated pulse CSS. | HTML/CSS (flexbox), data from `phases[]` | B | WF6 |
| 27 | **Forum Post Card** | HTML Element | Card component displaying one forum post. Contains: author handle, relative timestamp (e.g., "2h ago"), role badge (Copilot/Member), post body text, and a "Reply ↗" link that opens the browser. | HTML/CSS, `date-fns.formatDistanceToNow()` | B | WF6 |
| 28 | **Auto-Poll Indicator** | HTML Element | Toggle badge showing "Auto: ON/OFF" with a sync icon. Displays last-refresh timestamp. Toggling sends `postMessage({ command: 'togglePoll' })` to the extension host. | Webview messaging, `setInterval` / `clearInterval` | B | WF6 |
| 29 | **Load More Button** | HTML Element | Button below the last rendered forum post. Clicking loads additional forum content from the Vanilla forum provider URL in the challenge's `discussions[]` array. | Webview messaging, Vanilla forum | B | WF6 |

---

## 2. User Interactions

| # | Interaction | Trigger | Result | Related Element(s) | Tier |
|---|-------------|---------|--------|---------------------|------|
| 30 | **Login** | Command Palette: "Topcoder: Login" | Opens browser for OAuth2 authorization. On callback, stores JWT in `SecretStorage`. Updates login footer in tree. Refreshes challenge list. | Login Info Footer (#11) | A |
| 31 | **Logout** | Command Palette: "Topcoder: Logout" | Deletes JWT from `SecretStorage`. Clears tree view. Hides status bar items. Shows "Not logged in" in footer. | Login Info Footer (#11) | A |
| 32 | **Select Challenge** | Click challenge tree node | Sets the "active challenge" in extension state. Triggers `GET /v6/challenges/{id}` fetch. Updates status bar countdown (WF3) and requirements counter. Expands the node to show children. | Challenge Tree Node (#3), Status Bar (#16, #17) | A |
| 33 | **Open Spec** | Click "Spec & Requirements" child node | Creates or reveals the Spec Webview panel. Populates with rendered markdown from the challenge `description` field. | Spec & Requirements Node (#5), Spec Webview (#12) | A |
| 34 | **Refresh Challenge Data** | Click $(refresh) button in tree header, or Command Palette: "Topcoder: Refresh" | Re-fetches `GET /v6/challenges` (list) and `GET /v6/challenges/{id}` (active challenge). Fires `_onDidChangeTreeData` to update the tree. Updates webview if open. | Refresh Button (#10) | A |
| 35 | **Toggle Requirement Checkbox** | Click checkbox in Spec Webview | Sends `postMessage` (webview). Extension host updates `workspaceState` and refreshes the requirements counter status bar item. | Requirements Checklist (#15), Status Bar (#17) | A |
| 36 | **Download Attachment** | Click attachment in quick-pick or Attachments section | Fetches attachment metadata from `GET /v6/challenges/{id}/attachments`. Opens download URL via `vscode.env.openExternal()` (browser download) or streams to workspace via `fs.writeFile`. | Attachments Node (#7) | A |
| 37 | **Copy Spec to Clipboard** | Click "Copy Spec" button in webview toolbar | Extension host receives `postMessage({ command: 'copySpec' })`. Reads raw markdown from cached challenge data. Writes to clipboard via `vscode.env.clipboard.writeText()`. Shows confirmation via `showInformationMessage`. | Webview Toolbar (#13) | A |
| 38 | **Open in Browser** | Click "Open in Browser" in webview toolbar | Constructs the Topcoder challenge URL (`https://www.topcoder.com/challenges/{id}`) and opens via `vscode.env.openExternal()`. | Webview Toolbar (#13) | A |
| 44 | **View Forum** | Click "Discussions" child node in tree | Opens/reveals the Forum & Timeline webview (WF6). Uses `discussions[]` from the challenge object (Vanilla forum provider). Renders forum link or embedded content. Starts auto-poll if enabled. | Discussions Node (#8), Forum Webview (#25) | B |
| 45 | **Toggle Auto-Poll** | Click "Auto: ON/OFF" badge in Forum webview | Sends `postMessage({ command: 'togglePoll' })` to extension host. Toggles `setInterval`/`clearInterval`. Updates badge text and last-refresh timestamp. | Auto-Poll Indicator (#28) | B |
| 46 | **Load Older Posts** | Click "Load older posts..." in Forum webview | Extension host loads additional forum content from the Vanilla forum provider. New posts appended to DOM via `postMessage` back to webview. | Load More Button (#29), Forum Post Card (#27) | B |
| 47 | **Reply to Post (External)** | Click "Reply ↗" link on a forum card | Opens the Topcoder challenge forum page in browser via `vscode.env.openExternal()`. Plugin is read-only; replies happen in the browser. | Forum Post Card (#27) | B |
| 48 | **Click Countdown Timer** | Click status bar countdown | Opens the Timeline section — either reveals the Forum & Timeline webview (WF6) if Tier B is enabled, or shows a `showInformationMessage` with phase details if only Tier A. | Countdown Status Bar (#16) | A, B |

---

## 3. Data Concepts

| # | Concept | Type | Description | Source | Used By |
|---|---------|------|-------------|--------|---------|
| 49 | **JWT (JSON Web Token)** | Auth Token | Bearer token obtained via OAuth2 device flow or browser redirect from `https://accounts-auth0.topcoder.com`. Stored exclusively in `context.secrets` (VS Code `SecretStorage`). Attached as `Authorization: Bearer {jwt}` header on all API calls. Contains encoded `handle`, `userId`, and expiry (`exp`). Never logged or displayed in output channels. | Auth0 / Topcoder OAuth2 | All API calls |
| 50 | **Challenge Object** | API Response | JSON object from `GET /v6/challenges/{id}`. Key fields: `id`, `name`, `description` (markdown spec), `status` (ACTIVE/DRAFT/COMPLETED), `phases[]`, `prizeSets[]`, `tags[]`, `discussions[]`, `numOfRegistrants`, `numOfSubmissions`, `overview`. Cached in `globalState` with a 5-minute TTL. | `GET /v6/challenges/{id}` | WF1, WF2, WF3 |
| 51 | **Phase Object** | Nested Object | Entry in `challenge.phases[]`. Fields: `name` (Registration/Submission/Review/Appeals), `isOpen` (boolean), `scheduledStartDate`, `scheduledEndDate`, `duration` (ms). Used to build the timeline bar (WF6) and calculate countdown (WF3). | `GET /v6/challenges/{id}` → `phases[]` | WF3, WF6 |
| 52 | **Attachment/Resource** | API Response | File metadata from `GET /v6/challenges/{id}/attachments`. Fields: `id`, `name`, `url` (download link), `size`, `fileType`. Displayed in Attachments section of Spec Webview and as child count on Attachments tree node. | `GET /v6/challenges/{id}/attachments` | WF1, WF2 |
| 53 | **Discussion Thread** | API Response | Discussion metadata embedded in the challenge object's `discussions[]` array. Fields: `id`, `name`, `type`, `provider` ("vanilla"), `url` (Vanilla forum link at `discussions.topcoder.com`). Used to launch the Forum webview or open externally. | `GET /v6/challenges/{id}` → `discussions[]` | WF6 |
| 54 | **Submission Metadata** | API Response | Object from `GET /v6/submissions?challengeId={id}&memberId={handle}`. Fields: `id`, `type`, `url`, `createdAt`, `status` (Active/Completed). Shown optionally in the Spec Webview under "Submission History". | `GET /v6/submissions` | WF2 |
| 55 | **Requirement Item** | Derived Data | Parsed from the challenge spec body. Fields (internal): `id` (sequential, e.g., REQ-1), `text` (requirement sentence), `checked` (boolean, from `workspaceState`). Displayed in the Spec Webview (WF2) as a checklist for personal progress tracking. | Parsed from `challenge.description` | WF2 |
| 57 | **Plugin Configuration** | Settings Object | User-configurable settings accessed via `workspace.getConfiguration('topcoder')`. Keys: `apiBaseUrl` (default: `https://api.topcoder.com`), `pollInterval` (default: 120, range: 60–300), `enableTierB` (boolean), `maxChallenges` (default: 50). Defined in `package.json` under `contributes.configuration`. | `vscode.workspace.getConfiguration()` | All |
| 58 | **Cache Entry** | Internal State | Data stored in `globalState` with a TTL pattern. Key format: `topcoder.cache.{type}.{id}`. Value: `{ data: T, expiresAt: number }`. Types: `challengeList` (TTL 5 min), `challengeDetail` (TTL 5 min), `discussions` (TTL 60s). A helper function `getCachedOrFetch<T>()` wraps fetch calls with cache-check logic. | `ExtensionContext.globalState` | All API calls |
| 59 | **Checklist State** | Workspace State | Persisted per-challenge checkbox data. Key: `topcoder.checklist.{challengeId}`. Value: `string[]` (list of checked requirement IDs). Survives VS Code restarts because `workspaceState` is persistent. | `ExtensionContext.workspaceState` | WF2, WF3 |

---

## 4. Quick Reference — VS Code API Usage Map

| VS Code API | Elements Using It |
|-------------|-------------------|
| `TreeDataProvider` / `TreeItem` | Challenge Explorer (#2, #3) |
| `WebviewPanel` | Spec Webview (#12), Forum & Timeline (#25) |
| `StatusBarItem` | Countdown (#16), Requirements Counter (#17) |
| `SecretStorage` | JWT Storage (#49) |
| `globalState` / `workspaceState` | Cache (#58), Checklist State (#59) |
| `env.clipboard` | Copy Spec (#37) |
| `env.openExternal` | Attachments (#36), Reply (#47), Open in Browser (#38) |
| `workspace.getConfiguration` | Plugin Settings (#57) |
| `window.withProgress` | Requirement Scan (#39) |
| `CancellationToken` | All async API calls |
