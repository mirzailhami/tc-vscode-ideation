# Extras — Bonus Enhancement Ideas

> **Purpose:** Optional innovations that go beyond the three core tiers. Each idea is described with enough detail for a reviewer to evaluate feasibility and value. These are future-facing — none are required for the current submission.

---

## 1. AI-Powered Requirement Extraction

### Problem
Tier B's keyword matching relies on simple tokenization to extract requirements from spec markdown. This approach misses requirements phrased as paragraphs, embedded in tables, or written in non-standard formats (e.g., "the system must..." vs. a bullet list).

### Proposed Enhancement
Use a lightweight language model to parse the full spec and produce a structured list of actionable requirements automatically.

### Implementation Options

| Option | Approach | Trade-offs |
|--------|----------|------------|
| **A: Copilot Chat API** | Use `vscode.lm.selectChatModels()` (VS Code 1.90+ Language Model API) to send the spec text with a structured prompt: "Extract all functional requirements as a numbered list." | Requires Copilot subscription; depends on API availability. Zero additional cost for users who already have Copilot. |
| **B: Local regex + heuristics** | Enhanced pattern matching: detect "must", "shall", "required", numbered/bulleted lists, table rows with "requirement" headers. | No external dependency; less accurate for freeform text. |
| **C: External API (opt-in)** | Send spec to a user-configured LLM endpoint (OpenAI, Azure, self-hosted). | Maximum flexibility; privacy concern → must be opt-in with clear disclosure. |

### Output
Structured JSON written to `workspaceState`:
```typescript
interface ExtractedRequirement {
  id: string;           // "REQ-1", "REQ-2", ...
  text: string;         // Original requirement sentence
  keywords: string[];   // Auto-extracted searchable terms
  source: string;       // "line 42" or "table row 3" of spec
  confidence: number;   // 0.0–1.0 extraction confidence
}
```

### Value
Eliminates manual requirement identification. Works on any spec format. Dramatically improves Tier B accuracy.

---

## 2. Spec Diff Viewer

### Problem
Challenge specs are sometimes updated mid-phase (copilot clarifications, scope changes). Competitors may miss critical changes if they only read the spec once at registration.

### Proposed Enhancement
Detect spec updates and show a visual diff between the previously cached version and the latest version.

### Implementation
1. On each spec refresh (`GET /v6/challenges/{id}`), compare `description` field against the cached version in `globalState`.
2. If different, show a notification: "Spec updated for {challenge name}. View changes?"
3. On click, open a VS Code diff editor:
   ```typescript
   vscode.commands.executeCommand('vscode.diff',
     Uri.parse('topcoder-cache:spec-old'),  // Virtual document (old)
     Uri.parse('topcoder-cache:spec-new'),  // Virtual document (new)
     'Spec Changes: {challenge name}'
   );
   ```
4. Register a `TextDocumentContentProvider` for the `topcoder-cache` scheme that returns the old/new markdown text.

### UI
- Notification badge on the Spec & Requirements tree node: "$(warning) Updated".
- Standard VS Code side-by-side diff view with additions (green) and deletions (red).
- Timeline entry in WF6 showing when the spec was modified.

### Value
Prevents missed scope changes. Uses native VS Code diff — no custom UI needed. Very low implementation effort (~4-6 hours).

---

## 3. Submission Dry-Run Validator

### Problem
Topcoder submissions often fail due to incorrect ZIP structure, missing files, or wrong naming conventions. Competitors only find out after submitting and waiting for review.

### Proposed Enhancement
A pre-submission checklist that validates the current workspace against known Topcoder submission patterns before the user uploads.

### Validation Rules

| Check | How | Severity |
|-------|-----|----------|
| README.md present | `vscode.workspace.findFiles('README.md')` | Error |
| Source code present | `findFiles('src/**/*.{ts,js,py,java}')` | Error |
| No `node_modules` in zip | Check for `node_modules/` directory | Warning |
| File size reasonable | Sum workspace file sizes, warn if >50MB | Warning |
| Required files from spec | Cross-reference extracted requirements for file expectations (e.g., "Dockerfile", "docker-compose.yml") | Warning |
| No secrets/credentials | Scan for patterns: `password=`, `secret=`, `API_KEY=` | Error |

### UI
- Command: "Topcoder: Validate Submission"
- Output: Webview checklist showing pass/fail for each rule, with action buttons ("Fix: Add README template", "Fix: Add .dockerignore").
- Status bar badge: "$(check) Ready to Submit" or "$(warning) 2 issues".

### Value
Catches common mistakes before submission. Reduces failed submissions. Builds confidence in short-deadline challenges.

---

## 4. Multi-Challenge Dashboard

### Problem
Users working on multiple challenges simultaneously need a higher-level overview than the tree view provides. The tree is sequential — hard to compare deadlines and progress across challenges.

### Proposed Enhancement
A webview-based dashboard showing all active challenges in a card grid layout with visual progress indicators.

### Card Layout

> **Visual wireframe:** [`wireframes/extras-dashboard.excalidraw`](wireframes/extras-dashboard.excalidraw)

### Implementation
- Webview panel using CSS Grid for responsive card layout.
- Each card is a self-contained component with data from `GET /v6/challenges`.
- Cards sorted by deadline urgency (soonest first).
- Click actions route to existing commands (`topcoder.openSpec`, `topcoder.openForum`).
- Command: "Topcoder: Open Dashboard".

### Value
At-a-glance view of all active work. Especially useful during multi-challenge sprints. Natural extension of Tier A.

---

## 5. Keyboard Shortcuts & Quick Navigation

### Problem
Frequent operations (open spec, check requirements, toggle forum) require mouse clicks through tree views or command palette typing. Power users on tight deadlines want faster navigation.

### Proposed Enhancement
Pre-configured keyboard shortcuts and a custom Quick Pick navigation menu.

### Keybinding Defaults

| Shortcut | Command | Description |
|----------|---------|-------------|
| `Ctrl+Shift+T S` | `topcoder.openSpec` | Open spec for active challenge |
| `Ctrl+Shift+T F` | `topcoder.openForum` | Open forum panel |
| `Ctrl+Shift+T R` | `topcoder.checkRequirements` | Run requirement scan |
| `Ctrl+Shift+T D` | `topcoder.openDashboard` | Open multi-challenge dashboard |
| `Ctrl+Shift+T L` | `topcoder.login` | Trigger login |

### Quick Pick Navigation
Command: "Topcoder: Quick Navigate" (`Ctrl+Shift+T T`):

> **Visual wireframe:** [`wireframes/extras-quick-pick.excalidraw`](wireframes/extras-quick-pick.excalidraw)

### Implementation
- `contributes.keybindings` in `package.json` with `when` context conditions.
- Quick Pick via `vscode.window.showQuickPick()` with `QuickPickItem` entries.
- All shortcuts use the `Ctrl+Shift+T` chord prefix to avoid conflicts.

### Value
Saves seconds per action, which compounds over a 48–72h challenge timeline. Accessible to keyboard-first developers.

---

## 6. Summary

| Enhancement | Effort | Dependency | Impact |
|------------|--------|-----------|--------|
| AI Requirement Extraction | 10–15h | Copilot API (optional) | High — accuracy improvement |
| Spec Diff Viewer | 4–6h | None (VS Code built-in diff) | High — prevents missed changes |
| Submission Dry-Run | 8–12h | None | Medium — reduces failed submissions |
| Multi-Challenge Dashboard | 10–15h | None | Medium — better overview |
| Keyboard Shortcuts | 2–3h | None | Low effort, high QoL |
