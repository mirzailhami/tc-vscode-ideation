# Architecture — Topcoder VSCode Plugin

> **Scope:** Read-only VS Code extension fetching Topcoder challenge data via existing public v5/v6 APIs. No new endpoints proposed. No write/submit operations (view-only).

---

## 1. High-Level Flow

```mermaid
graph TD
    A[VS Code Extension Activation] --> B[Auth: Login Command → SecretStorage JWT]
    B --> C[Tree Provider: GET /v6/challenges → My Challenges]
    C --> D[On Select: GET /v6/challenges/id]
    D --> E[Webview: Render spec markdown + checklist]
    D --> F[Attachments: GET /v5/challenges/id/attachments → Download/Open]
    D --> G[Status Bar: Countdown from phases]
    E --> H[Polling: 60–300s → refresh timeline/forum]
    I[Commands] --> J[Check Requirements → keyword match code vs spec]
    J --> K[DiagnosticCollection: uncovered warnings]
    H --> L[Forum: GET /v5/challenge-discussions]
```

---

## 2. Component Architecture

```mermaid
graph LR
    subgraph Extension Host
        EXT[extension.ts<br/>activate / deactivate]
        AUTH[auth.ts<br/>SecretStorage + OAuth2]
        API[api-client.ts<br/>axios + cache + backoff]
        CFG[config.ts<br/>getConfiguration]
        TYPES[types.ts<br/>interfaces]
    end

    subgraph Providers
        TREE[challenge-provider.ts<br/>TreeDataProvider]
        REQS[requirement-checker.ts<br/>HoverProvider + Diagnostics]
        SB[status-bar.ts<br/>StatusBarItem x 2]
    end

    subgraph Webviews
        SPEC[webview-manager.ts<br/>Spec + Requirements panel]
        FORUM[forum-provider.ts<br/>Forum + Timeline panel]
    end

    EXT -->|registers| AUTH
    EXT -->|registers| TREE
    EXT -->|registers| REQS
    EXT -->|registers| SB
    EXT -->|registers| SPEC
    EXT -->|registers| FORUM
    AUTH -->|JWT| API
    API -->|cached data| TREE
    API -->|cached data| SPEC
    API -->|cached data| FORUM
    API -->|phases| SB
    CFG -->|settings| API
    CFG -->|settings| FORUM
    TYPES -.->|shared| API
    TYPES -.->|shared| TREE
    TYPES -.->|shared| SPEC
```

---

## 3. Module Breakdown

| Module | Responsibility | Key Exports |
|--------|---------------|-------------|
| `extension.ts` | Entry point. Registers all commands, providers, and views in `activate()`. Pushes all disposables to `context.subscriptions`. | `activate()`, `deactivate()` |
| `auth.ts` | OAuth2 device/browser flow. Stores JWT in `context.secrets`. Decodes token for handle/userId. Detects expiry. | `AuthService` class |
| `api-client.ts` | Centralized HTTP client (axios). Attaches `Authorization: Bearer` header. Implements caching (globalState + TTL), retry logic (3 retries, exponential backoff), and `CancellationToken` support. | `ApiClient` class |
| `config.ts` | Reads `workspace.getConfiguration('topcoder')`. Validates and exposes typed settings. | `getConfig()` helper |
| `types.ts` | TypeScript interfaces: `Challenge`, `Phase`, `Attachment`, `Discussion`, `Submission`, `Requirement`, `CacheEntry<T>`. | Type-only exports |
| `challenge-provider.ts` | `TreeDataProvider<ChallengeTreeItem>`. Fetches challenge list, builds tree nodes with children (Spec, Requirements, Attachments, Forum, Timeline). | `ChallengeProvider` class |
| `webview-manager.ts` | Creates/manages the Spec Webview panel. Renders markdown via `markdown-it`, sanitizes HTML, handles `postMessage` commands (refresh, copy, toggle checkbox). | `WebviewManager` class |
| `status-bar.ts` | Creates two `StatusBarItem` instances (countdown + requirements counter). Updates countdown every 60s. Color-codes by urgency. | `StatusBarManager` class |
| `requirement-checker.ts` | Parses spec into requirement items. Keyword extraction. Workspace file scanning. Registers `HoverProvider`. Manages `DiagnosticCollection` and `TextEditorDecorationType`. | `RequirementChecker` class |
| `forum-provider.ts` | Creates/manages the Forum & Timeline Webview. Fetches discussions. Renders post cards. Manages auto-poll interval. Handles pagination. | `ForumProvider` class |

---

## 4. API Specification Table

All endpoints are existing public Topcoder APIs. **No new endpoints required.**

| # | Purpose | Method | Endpoint | Query Params | Auth | Response Fields Used | Caching |
|---|---------|--------|----------|-------------|------|---------------------|---------|
| 1 | **List active challenges** | `GET` | `/v6/challenges` | `status=Active`, `memberId={handle}`, `sortBy=updatedAt`, `sortOrder=desc`, `perPage=50` | `Bearer {JWT}` | `id`, `name`, `status`, `numOfRegistrants`, `numOfSubmissions`, `tags[]`, `prizeSets[]` | `globalState`, TTL: 5 min |
| 2 | **Get challenge details** | `GET` | `/v6/challenges/{challengeId}` | — | `Bearer {JWT}` | `id`, `name`, `description` (markdown spec), `status`, `phases[]`, `prizeSets[]`, `tags[]`, `legacy.track`, `metadata` | `globalState`, TTL: 5 min |
| 3 | **List attachments** | `GET` | `/v5/challenges/{challengeId}/attachments` | — | `Bearer {JWT}` | `id`, `name`, `url`, `fileType`, `size` | `globalState`, TTL: 10 min |
| 4 | **List resources (roles)** | `GET` | `/v5/resources` | `challengeId={id}` | `Bearer {JWT}` | `memberId`, `memberHandle`, `roleId` (to verify registration) | `globalState`, TTL: 10 min |
| 5 | **Get forum discussions** | `GET` | `/v5/challenge-discussions` | `challengeId={id}`, `limit=20`, `offset=0`, `sort=createdAt desc` | `Bearer {JWT}` | `id`, `challengeId`, `body`, `authorHandle`, `authorRole`, `createdAt` | `globalState`, TTL: 60s |
| 6 | **Get submission history** | `GET` | `/v5/submissions` | `challengeId={id}`, `memberId={handle}`, `perPage=10` | `Bearer {JWT}` | `id`, `type`, `url`, `createdAt`, `status` | `globalState`, TTL: 5 min |
| 7 | **Get member profile** | `GET` | `/v5/members/{handle}` | — | `Bearer {JWT}` | `handle`, `photoURL`, `skills[]` (for login info display) | `globalState`, TTL: 30 min |
| 8 | **Authenticate (token)** | `POST` | `https://accounts-auth0.topcoder.com/oauth/token` | Body: `grant_type`, `client_id`, `scope`, `audience` (or device code flow params) | None (generates token) | `access_token`, `id_token`, `expires_in`, `token_type` | `SecretStorage` (persistent) |

### Base URLs

| Environment | Base URL |
|-------------|----------|
| Production | `https://api.topcoder.com` |
| Development | `https://api.topcoder-dev.com` |
| Auth (Production) | `https://accounts-auth0.topcoder.com` |

> **All endpoints are existing and documented.** The extension makes no mutations — all calls are `GET` (read-only) except the initial `POST` for authentication.

---

## 5. Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant VSCode as VS Code Extension
    participant Browser
    participant Auth0 as accounts-auth0.topcoder.com
    participant API as api.topcoder.com

    User->>VSCode: Command: Topcoder Login
    VSCode->>Browser: Open Auth0 authorize URL
    Browser->>Auth0: User enters credentials
    Auth0->>VSCode: Redirect with auth code via URI handler
    VSCode->>Auth0: POST /oauth/token exchange code
    Auth0->>VSCode: access_token + id_token + expires_in
    VSCode->>VSCode: Store JWT in SecretStorage
    VSCode->>API: GET /v6/challenges with Bearer token
    API->>VSCode: Challenge list JSON
    VSCode->>User: Tree view populated
```

### Token Management
- **Storage:** `context.secrets.store('topcoder.jwt', token)` — encrypted, OS-level keychain.
- **Retrieval:** `context.secrets.get('topcoder.jwt')` — called by `ApiClient` before each request.
- **Expiry Detection:** Decode JWT payload, check `exp` claim against `Date.now()`. If expired or within 5 min of expiry, prompt re-login via `showWarningMessage`.
- **401 Handling:** `ApiClient` intercepts 401 responses → clears stored JWT → shows "Session expired. Please log in again." → triggers login command.
- **Logout:** `context.secrets.delete('topcoder.jwt')` → clear all cached data → reset tree view.

---

## 6. Security

### Webview Content Security Policy
Every webview includes this CSP in the `<meta>` tag:

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'none'; 
               style-src ${webview.cspSource}; 
               script-src 'nonce-${nonce}'; 
               img-src ${webview.cspSource} https:; 
               font-src ${webview.cspSource};">
```

### Security Checklist

| Concern | Mitigation |
|---------|-----------|
| Token exposure | Store in `SecretStorage` only. Never in `globalState`, settings, or logs. |
| XSS in webviews | Sanitize all HTML with `sanitize-html`. Strict CSP. Nonce-based `<script>` tags only. |
| Resource access | `localResourceRoots` restricted to extension's `media/` and `dist/` directories. |
| Logging | Custom `OutputChannel` for debug info. JWT values are redacted: `Bearer ***` in logs. |
| API abuse | Rate limit detection (HTTP 429) → exponential backoff (base 2s, max 60s, 3 retries). |
| Man-in-middle | All API calls over HTTPS. No HTTP fallback. |
| Extension permissions | Minimal activation events. No file system write access required. |
| Cancellation | All async API calls accept `CancellationToken`. Aborted on user action or timeout. |

---

## 7. Performance

### Caching Strategy

```typescript
interface CacheEntry<T> {
  data: T;
  expiresAt: number; // Date.now() + ttlMs
}

async function getCachedOrFetch<T>(
  key: string,
  fetcher: (token: CancellationToken) => Promise<T>,
  ttlMs: number,
  globalState: vscode.Memento,
  token: CancellationToken
): Promise<T> {
  const cached = globalState.get<CacheEntry<T>>(key);
  if (cached && cached.expiresAt > Date.now()) {
    return cached.data;
  }
  const data = await fetcher(token);
  await globalState.update(key, { data, expiresAt: Date.now() + ttlMs });
  return data;
}
```

### TTL Configuration

| Data Type | Cache Key Pattern | Default TTL | Configurable |
|-----------|-------------------|-------------|-------------|
| Challenge list | `tc.cache.list` | 5 min | No |
| Challenge detail | `tc.cache.detail.{id}` | 5 min | No |
| Attachments | `tc.cache.attach.{id}` | 10 min | No |
| Forum posts | `tc.cache.forum.{id}` | 60s | Via `pollInterval` |
| Member profile | `tc.cache.member.{handle}` | 30 min | No |

### Polling

- Forum/timeline auto-refresh: configurable interval (`topcoder.pollInterval`), default 120s, range 60–300s.
- Status bar countdown: UI-only timer (`setInterval` 60s), no API call — uses cached phase dates.
- Tree view: manual refresh only (no auto-poll) to minimize API calls.
- All intervals stored as disposable references, cleared in `deactivate()` and on logout.

### Lazy Activation

```jsonc
// package.json
{
  "activationEvents": [
    "onStartupFinished"
  ]
}
```

The extension activates after VS Code finishes loading. No blocking `onStartup` or `*` activation. Tree data is fetched only when the Topcoder sidebar is first opened (lazy on view visibility).

---

## 8. Error Handling

| Scenario | Detection | Response |
|----------|-----------|----------|
| Network failure | `axios` error `ERR_NETWORK` / timeout | `showErrorMessage("Cannot reach Topcoder API. Check your connection.")`. Queue retry. |
| HTTP 401 Unauthorized | Response status 401 | Clear stored JWT. `showWarningMessage("Session expired...")`. Trigger `topcoder.login` command. |
| HTTP 403 Forbidden | Response status 403 | `showErrorMessage("Access denied. You may not be registered for this challenge.")` |
| HTTP 429 Rate Limited | Response status 429 + `Retry-After` header | Exponential backoff: wait `min(2^attempt * 2000, 60000)` ms. Max 3 retries. Show progress notification during wait. |
| HTTP 5xx Server Error | Response status 500–599 | `showErrorMessage("Topcoder API error. Try again later.")`. Log status + URL to output channel. |
| Invalid/corrupt JWT | JWT decode fails or `exp` < now | Clear stored JWT. Prompt re-login. |
| Empty challenge list | 200 response with empty array | Show tree view message: "No active challenges found. Make sure you are registered." |
| Forum API unavailable | 404 or specific error code | Graceful degradation: hide forum section, show "Discussions not available for this challenge." |
| Large spec body | Description > 500 KB | Truncate with "Show full spec in browser" link. Prevents webview performance issues. |
| Extension crash | Unhandled rejection | Global `process.on('unhandledRejection')` logs to output channel. No user-facing crash. |

---

## 9. Data Flow Summary

| User Action | Extension Host | API Call | UI Update |
|------------|---------------|----------|-----------|
| Login | `AuthService.login()` | `POST auth0/oauth/token` | Tree: show challenges |
| Select challenge | `ApiClient.getDetail()` | `GET /v6/challenges/{id}` | Status bar, expand tree |
| Open spec | `WebviewManager.show()` | (cached from above) | Webview: rendered spec |
| Toggle checkbox | `workspaceState.update()` | (none — local only) | Counter: 6/8 |
| Download attachment | `env.openExternal()` | `GET /v5/.../attachments` | Browser opens download |
| Refresh | `ApiClient.getList()` | `GET /v6/challenges` | Tree: updated list |
| Check requirements | `RequirementChecker` | (none — workspace scan) | Diagnostics, decorations |
| View forum | `ForumProvider.show()` | `GET /v5/challenge-discussions` | Webview: post cards |
| Auto-poll tick | `ForumProvider.poll()` | `GET /v5/challenge-discussions` | Badge count, new posts |
