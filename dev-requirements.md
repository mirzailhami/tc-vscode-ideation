# Development Requirements — Topcoder VSCode Plugin

> **Audience:** Developer implementing the plugin from this design. Covers prerequisites, setup, dependencies, file structure, contribution points, build, testing, risks, and effort estimates.

---

## 1. Prerequisites

| Requirement | Version | Purpose |
|------------|---------|---------|
| Node.js | 18+ (LTS recommended) | Runtime for extension development, build tools, tests |
| VS Code | 1.85+ | Target platform; required for `TreeItemCheckboxState`, secondary sidebar, latest API surface |
| npm or pnpm | Latest | Package management |
| `vsce` CLI | Latest (`@vscode/vsce`) | Packaging the extension into `.vsix` |
| Git | 2.30+ | Version control |
| Topcoder account | — | Required for API access; must be registered for at least one challenge to test |

### Optional
- **Docker** — for containerized development/testing.
- **VS Code Insiders** — for testing against pre-release API features.

---

## 2. Project Scaffolding

### Initialize

```bash
# Option A: Yeoman generator (recommended)
npm install -g yo generator-code
yo code
# → Select "New Extension (TypeScript)"
# → Name: "topcoder-vscode-plugin"
# → Identifier: "topcoder-vscode-plugin"
# → Enable strict TypeScript

# Option B: Manual
mkdir topcoder-vscode-plugin && cd topcoder-vscode-plugin
npm init -y
npm install typescript @types/vscode --save-dev
npx tsc --init
```

### TypeScript Configuration (`tsconfig.json`)

```jsonc
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "ES2022",
    "lib": ["ES2022"],
    "strict": true,
    "noImplicitAny": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "rootDir": "src",
    "outDir": "dist",
    "declaration": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "test"]
}
```

### ESLint Configuration (`.eslintrc.json`)

```jsonc
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "project": "./tsconfig.json"
  },
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "no-console": "error"
  }
}
```

---

## 3. Dependencies

### Runtime Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `axios` | ^1.7 | HTTP client for all Topcoder API calls |
| `markdown-it` | ^14 | Render challenge spec markdown to HTML for webview |
| `sanitize-html` | ^2.13 | Strip dangerous HTML/XSS from rendered spec content |
| `date-fns` | ^3.6 | Format timestamps (relative time for forum posts, countdown formatting) |

### Dev Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `@types/vscode` | ^1.85 | VS Code API type definitions |
| `@types/sanitize-html` | ^2 | Type definitions for sanitize-html |
| `@types/markdown-it` | ^14 | Type definitions for markdown-it |
| `typescript` | ^5.4 | TypeScript compiler |
| `esbuild` | ^0.21 | Fast bundler for extension code |
| `@vscode/vsce` | ^2.26 | CLI for packaging `.vsix` files |
| `@vscode/test-electron` | ^2.4 | Integration test runner for VS Code extensions |
| `jest` | ^29 | Unit test framework |
| `ts-jest` | ^29 | TypeScript preset for Jest |
| `@types/jest` | ^29 | Jest type definitions |
| `eslint` | ^9 | Linting |
| `@typescript-eslint/parser` | ^7 | TypeScript ESLint parser |
| `@typescript-eslint/eslint-plugin` | ^7 | TypeScript-specific lint rules |

### Optional Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `@vscode/webview-ui-toolkit` | ^1.4 | Microsoft's component library for webview UIs (buttons, data grids, etc.) |
| `react` + `react-dom` | ^18 | If webviews use React instead of plain HTML/JS |

---

## 4. File Structure

```
topcoder-vscode-plugin/
├── .vscode/
│   ├── launch.json            # F5 debug configuration
│   ├── tasks.json             # Build tasks (compile, watch)
│   └── settings.json          # Workspace settings (editor, format)
├── media/
│   ├── topcoder-icon.svg      # Activity bar icon (24×24)
│   ├── topcoder-icon-dark.svg # Dark theme variant
│   └── topcoder-icon-light.svg# Light theme variant
├── src/
│   ├── extension.ts           # activate() / deactivate() entry point
│   ├── auth.ts                # AuthService: OAuth2, SecretStorage, token mgmt
│   ├── api-client.ts          # ApiClient: axios wrapper, caching, backoff
│   ├── config.ts              # Configuration reader (getConfiguration)
│   ├── types.ts               # Shared TypeScript interfaces
│   ├── challenge-provider.ts  # TreeDataProvider for challenge explorer
│   ├── webview-manager.ts     # Spec webview panel (create, render, messaging)
│   ├── status-bar.ts          # StatusBarManager: countdown + req counter
│   ├── requirement-checker.ts # HoverProvider, DiagnosticCollection, decorations
│   └── forum-provider.ts      # Forum + Timeline webview panel
├── webview/
│   ├── spec.html              # Spec webview HTML template
│   ├── spec.css               # Spec webview styles (or use toolkit)
│   ├── spec.js                # Spec webview client-side JS (postMessage)
│   ├── forum.html             # Forum webview HTML template
│   ├── forum.css              # Forum webview styles
│   └── forum.js               # Forum webview client-side JS
├── test/
│   ├── unit/
│   │   ├── api-client.test.ts # Mock axios, test caching + retry
│   │   ├── challenge-provider.test.ts  # Test tree node generation
│   │   ├── auth.test.ts       # Test token parsing, expiry detection
│   │   ├── requirement-checker.test.ts # Test keyword extraction + matching
│   │   └── config.test.ts     # Test config defaults + overrides
│   └── integration/
│       └── extension.test.ts  # VS Code integration test (activate, commands)
├── .eslintrc.json             # ESLint configuration
├── .vscodeignore              # Files to exclude from .vsix package
├── .gitignore                 # Git ignores (node_modules, dist, *.vsix)
├── esbuild.config.mjs         # esbuild bundler configuration
├── tsconfig.json              # TypeScript configuration
├── package.json               # Extension manifest + contribution points
├── CHANGELOG.md               # Version history
├── LICENSE                    # License file
└── README.md                 # Extension readme (shown in marketplace)
```

---

## 5. package.json — Key Contribution Points

```jsonc
{
  "name": "topcoder-vscode-plugin",
  "displayName": "Topcoder Challenge Explorer",
  "description": "Browse Topcoder challenges, read specs, track requirements, and follow forum discussions — all inside VS Code.",
  "version": "0.1.0",
  "publisher": "topcoder",
  "engines": { "vscode": "^1.85.0" },
  "categories": ["Other"],
  "keywords": ["topcoder", "challenges", "competitive programming", "spec"],

  "activationEvents": ["onStartupFinished"],

  "main": "./dist/extension.js",

  "contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "topcoder",
          "title": "Topcoder",
          "icon": "media/topcoder-icon.svg"
        }
      ]
    },
    "views": {
      "topcoder": [
        {
          "id": "topcoder.challengeExplorer",
          "name": "My Active Challenges",
          "visibility": "visible"
        },
        {
          "id": "topcoder.requirementsView",
          "name": "Requirements Checklist",
          "visibility": "collapsed",
          "when": "topcoder.tierBEnabled"
        }
      ]
    },
    "commands": [
      { "command": "topcoder.login",             "title": "Topcoder: Login" },
      { "command": "topcoder.logout",            "title": "Topcoder: Logout" },
      { "command": "topcoder.refresh",           "title": "Topcoder: Refresh Challenges",   "icon": "$(refresh)" },
      { "command": "topcoder.openSpec",          "title": "Topcoder: Open Spec" },
      { "command": "topcoder.openForum",         "title": "Topcoder: Open Forum" },
      { "command": "topcoder.openTimeline",      "title": "Topcoder: Open Timeline" },
      { "command": "topcoder.checkRequirements", "title": "Topcoder: Check Requirements" },
      { "command": "topcoder.exportChecklist",   "title": "Topcoder: Export Checklist" },
      { "command": "topcoder.copySpec",          "title": "Topcoder: Copy Spec to Clipboard" },
      { "command": "topcoder.openInBrowser",     "title": "Topcoder: Open in Browser" }
    ],
    "menus": {
      "view/title": [
        {
          "command": "topcoder.refresh",
          "when": "view == topcoder.challengeExplorer",
          "group": "navigation"
        }
      ]
    },
    "configuration": {
      "title": "Topcoder",
      "properties": {
        "topcoder.apiBaseUrl": {
          "type": "string",
          "default": "https://api.topcoder.com",
          "description": "Base URL for the Topcoder API (change for dev environment)."
        },
        "topcoder.pollInterval": {
          "type": "number",
          "default": 120,
          "minimum": 60,
          "maximum": 300,
          "description": "Forum/timeline auto-refresh interval in seconds."
        },
        "topcoder.enableTierB": {
          "type": "boolean",
          "default": false,
          "description": "Enable Tier B features: Inline Requirement Checker + Checklist."
        },
        "topcoder.enableTierC": {
          "type": "boolean",
          "default": false,
          "description": "Enable Tier C features: Forum & Timeline Live Feed."
        },
        "topcoder.maxChallenges": {
          "type": "number",
          "default": 50,
          "minimum": 10,
          "maximum": 200,
          "description": "Maximum number of challenges to fetch in the explorer."
        }
      }
    }
  }
}
```

---

## 6. Build & Package

### esbuild Configuration (`esbuild.config.mjs`)

```javascript
import * as esbuild from 'esbuild';

const isWatch = process.argv.includes('--watch');

const config = {
  entryPoints: ['src/extension.ts'],
  bundle: true,
  outfile: 'dist/extension.js',
  external: ['vscode'],
  format: 'cjs',
  platform: 'node',
  target: 'node18',
  sourcemap: true,
  minify: !isWatch,
};

if (isWatch) {
  const ctx = await esbuild.context(config);
  await ctx.watch();
  console.log('Watching for changes...');
} else {
  await esbuild.build(config);
  console.log('Build complete.');
}
```

### npm Scripts (`package.json`)

```jsonc
{
  "scripts": {
    "compile": "node esbuild.config.mjs",
    "watch": "node esbuild.config.mjs --watch",
    "lint": "eslint src/ --ext .ts",
    "test": "jest --config jest.config.ts",
    "test:integration": "vscode-test",
    "package": "vsce package",
    "publish": "vsce publish"
  }
}
```

### Build Steps

```bash
# 1. Install dependencies
npm install

# 2. Compile TypeScript → bundled JS
npm run compile

# 3. Lint
npm run lint

# 4. Run unit tests
npm test

# 5. Package into .vsix
npm run package
# → Produces: topcoder-vscode-plugin-0.1.0.vsix

# 6. Install locally
code --install-extension topcoder-vscode-plugin-0.1.0.vsix
```

### Debug (F5)

`.vscode/launch.json`:

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "args": ["--extensionDevelopmentPath=${workspaceFolder}"],
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "preLaunchTask": "npm: compile"
    }
  ]
}
```

---

## 7. Testing Plan

### Unit Tests (Jest)

| Test Suite | What It Tests | Mocking Strategy |
|-----------|---------------|-----------------|
| `api-client.test.ts` | API calls return correct data; caching works (second call returns cached); retry logic on 429/5xx; 401 triggers re-login. | Mock `axios` with `jest.mock('axios')`. Mock `globalState` with in-memory `Map`. |
| `challenge-provider.test.ts` | Tree nodes have correct labels, icons, children; empty state shows message node; refresh triggers re-fetch. | Mock `ApiClient`. |
| `auth.test.ts` | JWT decode extracts handle; expiry detection works; logout clears storage. | Mock `context.secrets` with in-memory storage. |
| `requirement-checker.test.ts` | Keyword extraction from requirements; file scanning finds matches; diagnostics generated for uncovered reqs. | Mock `vscode.workspace.findFiles` and `vscode.workspace.openTextDocument`. |
| `config.test.ts` | Default values correct; overrides respected; invalid values clamped. | Mock `vscode.workspace.getConfiguration`. |

### Integration Tests (`@vscode/test-electron`)

| Test | What It Verifies |
|------|-----------------|
| Extension activates | `activate()` runs without error; commands registered. |
| Login command | Command exists and is executable (actual auth not tested). |
| Tree view registered | `topcoder.challengeExplorer` view is registered and visible in the activity bar. |
| Webview opens | `topcoder.openSpec` creates a webview panel with correct title. |
| Status bar items | Two status bar items created on activation. |

### Manual E2E Checklist

- [ ] Install `.vsix` in clean VS Code
- [ ] "Topcoder: Login" → browser opens, token stored
- [ ] Sidebar shows "My Active Challenges" with real data
- [ ] Click challenge → children appear (Spec, Requirements, etc.)
- [ ] Click "Spec & Requirements" → webview opens with formatted spec
- [ ] Checkbox toggle persists across VS Code restart
- [ ] Status bar shows countdown, color changes at thresholds
- [ ] "Topcoder: Check Requirements" → Problems panel shows warnings
- [ ] Forum webview loads posts, auto-refreshes
- [ ] "Topcoder: Logout" → tree clears, status bar hides
- [ ] No token in Output channel logs

---

## 8. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| **API rate limiting** | Medium | Requests blocked temporarily | Exponential backoff (base 2s, max 60s, 3 retries). Respect `Retry-After` header. Conservative polling (120s default). |
| **API deprecation (v5→v6 migration)** | Low | Endpoints may change | Pin API version in `apiBaseUrl` config. Document all endpoints explicitly. Abstract behind `ApiClient` for easy swapping. |
| **Large spec bodies (>500KB)** | Low | Webview performance degradation | Truncate with "Show full spec in browser" link. Lazy-render sections with `<details>` elements. |
| **Token expiry during long sessions** | Medium | 401 errors mid-use | Proactive expiry detection (5 min before `exp`). Auto-prompt re-login. Clear state gracefully. |
| **Forum API unavailability** | Medium | No discussion data | Graceful degradation: show "Discussions not available" message. Hide forum section. Do not crash. |
| **No network connectivity** | Medium | All features broken | Cached data remains accessible (read from `globalState`). Show offline banner. Queue refresh for when connectivity returns. |
| **VS Code API breaking changes** | Low | Build failures | Engine constraint `^1.85`. Test against Stable + Insiders. Pin `@types/vscode` version. |
| **Secret storage unavailable** | Very Low | Cannot store JWT | Fallback: prompt login every session. Never store token in plaintext. |
| **Keyword matching false positives** | Medium | Incorrect requirement links | Allow user to manually override/dismiss. Provide confidence score. Make matching configurable. |

---

## 9. Estimated Effort

| Tier | Scope | Estimated Hours | Priority |
|------|-------|----------------|----------|
| **A: Explorer + Spec** | Auth, tree view, spec webview, status bar, attachments | 40–60 hours | High (core value) |
| **B: Requirement Checker** | Keyword extraction, hover provider, diagnostics, checklist tree, workspace scan | 20–30 hours | Medium (innovative) |
| **C: Forum + Timeline** | Forum webview, timeline bar, auto-poll, post cards, pagination | 20–30 hours | Medium (utility) |
| **Testing + Polish** | Unit tests, integration tests, error handling, README, packaging | 15–20 hours | High |
| **Total** | All tiers + testing | **~100–140 hours** | — |

### Recommended Implementation Order

1. Auth + API client (foundation for everything)
2. Challenge tree view (Tier A core)
3. Spec webview (Tier A primary deliverable)
4. Status bar countdown (Tier A, quick win)
5. Attachments (Tier A, small scope)
6. Requirements checklist (Tier B)
7. Inline checker + diagnostics (Tier B)
8. Forum feed (Tier C)
9. Timeline bar (Tier C)
10. Testing, polish, packaging

---

## 10. `.vscodeignore`

```
.vscode/**
src/**
test/**
node_modules/**
.eslintrc.json
tsconfig.json
esbuild.config.mjs
*.map
.gitignore
CHANGELOG.md
```

Only `dist/`, `media/`, `package.json`, `README.md`, and `LICENSE` are included in the `.vsix` package.

---

## 11. Requirement Traceability Matrix

> **Purpose:** Maps every challenge spec requirement to its implementing feature module, wireframe, API endpoint, and test case — ensuring full coverage and auditability.

| Req ID | Spec Requirement | Tier | Feature Module | Wireframe(s) | API Endpoint(s) | Test Case(s) | Status |
|--------|-----------------|------|---------------|--------------|-----------------|-------------|--------|
| R-01 | Wireframes for all key screens | — | N/A (design) | WF1–WF7 | — | Visual review | ✅ Covered |
| R-02 | Glossary defining UI elements | — | N/A (design) | glossary.md | — | Content review (59 entries) | ✅ Covered |
| R-03 | Architecture with existing API usage | — | `architecture.md` | — | EP-1 through EP-8 | API appendix verification | ✅ Covered |
| R-04 | Browse active challenges in sidebar | A | `challenge-provider.ts` | WF1 | EP-1: `GET /v6/challenges` | `challenge-provider.test.ts`: tree node labels, icons, children | ✅ Covered |
| R-05 | Render challenge spec as markdown | A | `webview-manager.ts` | WF2 | EP-2: `GET /v6/challenges/{id}` | `api-client.test.ts`: detail fetch; manual: webview renders | ✅ Covered |
| R-06 | Requirements checklist with checkboxes | A/B | `webview-manager.ts`, `requirement-checker.ts` | WF2, WF5 | EP-2 (description field) | `requirement-checker.test.ts`: extraction; manual: checkbox toggle | ✅ Covered |
| R-07 | Status bar countdown timer | A | `status-bar.ts` | WF3 | EP-2 (phases field) | Manual: color coding at thresholds | ✅ Covered |
| R-08 | Download/view attachments | A | `webview-manager.ts` | WF2 | EP-3: `GET /v6/challenges/{id}/attachments` | `api-client.test.ts`: attachment list; manual: download | ✅ Covered |
| R-09 | Show registrants | A | `challenge-provider.ts` | WF1 | EP-4: `GET /v6/resources` | `challenge-provider.test.ts`: registrant count | ✅ Covered |
| R-10 | Forum/discussions view | C | `forum-provider.ts` | WF6 | EP-5: `discussions[]` from challenge object | Manual: forum rendering | ✅ Covered |
| R-11 | Submission history | A | `webview-manager.ts` | WF2 | EP-6: `GET /v6/submissions` | `api-client.test.ts`: submission list | ✅ Covered |
| R-12 | Inline requirement checker + diagnostics | B | `requirement-checker.ts` | WF4 | Workspace scan (no API) | `requirement-checker.test.ts`: keyword match, diagnostics | ✅ Covered |
| R-13 | Auth via OAuth2 / JWT | A | `auth.ts` | — | EP-8: `POST /oauth/token` | `auth.test.ts`: token decode, expiry | ✅ Covered |
| R-14 | Secure token storage | A | `auth.ts` | — | — | `auth.test.ts`: SecretStorage mock | ✅ Covered |
| R-15 | Polling for live updates | C | `forum-provider.ts` | WF6 | EP-5 (interval) | Manual: auto-refresh indicator | ✅ Covered |
| R-16 | User profile display | A | `challenge-provider.ts` | WF1 (footer) | EP-7: `GET /v6/members?handle={h}` | Manual: login display | ✅ Covered |
| R-17 | No new API endpoints | — | All modules | — | Verified in EP-1–EP-8 | API appendix audit | ✅ Covered |
| R-18 | Edge states (loading/empty/error/expired) | A/B/C | All providers | WF7 (A–F) | Error handlers | Manual: disconnect network, expire token | ✅ Covered |
| R-19 | Multiple ideas / modular tiers | — | N/A (design) | extras.md | — | Content review | ✅ Covered |

---

## 12. Quality Gates

> **Purpose:** Concrete, measurable criteria that must pass before each milestone. No code merges without satisfying the applicable gate.

### Gate 1: Pre-Commit (Developer Workstation)

| Check | Tool | Threshold | Blocking |
|-------|------|-----------|----------|
| TypeScript compilation | `tsc --noEmit` | Zero errors | Yes |
| ESLint | `eslint src/ --ext .ts` | Zero errors, zero warnings | Yes |
| No `any` types | `@typescript-eslint/no-explicit-any: error` | Zero violations | Yes |
| Unit test pass | `jest --ci` | 100% pass rate | Yes |
| Unit test coverage (overall) | `jest --coverage` | ≥ 80% lines | Yes |

### Gate 2: Module-Level Coverage Targets

| Module | Minimum Line Coverage | Rationale |
|--------|----------------------|-----------|
| `api-client.ts` | ≥ 90% | Critical path; caching + retry logic must be fully tested |
| `auth.ts` | ≥ 90% | Security-sensitive; token handling must be airtight |
| `challenge-provider.ts` | ≥ 85% | Core UI data source; tree node generation must be reliable |
| `requirement-checker.ts` | ≥ 80% | Keyword matching has edge cases; good coverage prevents false positives |
| `config.ts` | ≥ 95% | Simple module; easy to achieve full coverage |
| `webview-manager.ts` | ≥ 60% | Webview creation is hard to unit test; rely on integration tests |
| `forum-provider.ts` | ≥ 60% | Similar to webview; integration tests supplement |
| `status-bar.ts` | ≥ 75% | Timer logic needs coverage; UI rendering tested manually |

### Gate 3: Pre-Release (CI Pipeline)

| Check | Tool | Threshold | Blocking |
|-------|------|-----------|----------|
| Build succeeds | `npm run compile` | Exit code 0 | Yes |
| All tests pass | `npm test` | 100% pass, ≥ 80% coverage | Yes |
| Integration tests | `@vscode/test-electron` | All 5 integration tests pass | Yes |
| Bundle size | `vsce package --out /dev/null` | `.vsix` ≤ 2 MB | Warning (non-blocking) |
| No `console.log` | ESLint `no-console: error` | Zero violations | Yes |
| Dependency audit | `npm audit --production` | Zero high/critical vulnerabilities | Yes |
| Manual E2E checklist | Human tester | All 11 checklist items pass (§7) | Yes |

### Gate 4: Post-Release Monitoring

| Metric | Source | Alert Threshold |
|--------|--------|----------------|
| Extension crash rate | VS Code telemetry (opt-in) | > 1% of activations |
| API error rate (429s) | Extension output channel logs | > 5 rate-limited requests/hour |
| Unhandled rejections | `process.on('unhandledRejection')` log | Any occurrence |
| User-reported issues | GitHub Issues | Triage within 48h |

### Release & Rollback Plan

| Phase | Action |
|-------|--------|
| **Versioning** | Semantic versioning (`MAJOR.MINOR.PATCH`). Pre-release: `0.x.y`. |
| **Release channel** | GitHub Releases + VS Code Marketplace (`vsce publish`). |
| **Rollback trigger** | Crash rate > 1% OR critical security issue OR data loss. |
| **Rollback procedure** | `vsce unpublish` current version → re-publish previous `.vsix` from GitHub Releases. Users auto-update on next VS Code restart. |
| **Issue triage** | Critical (security/crash) → hotfix within 24h. High (feature broken) → patch within 72h. Medium/Low → next minor release. |
