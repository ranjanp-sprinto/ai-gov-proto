# Shadow AI + AI Systems Wireframe - Sprinto

## What This Is

High-fidelity interactive wireframe for Sprinto's AI Systems area. It is a single self-contained HTML prototype:

- Main file: `shadow_ai_wireframe.html`
- No build step required
- Preview command: `python3 -m http.server 8000`
- Preview URL: `http://localhost:8000/shadow_ai_wireframe.html`

The prototype currently covers:

- Shadow AI discovery and reconciliation queue
- AI systems register
- AI system detail view
- Evidence tab for assessments and documents
- Model registry and model detail flows
- Events stream for reviewed runtime signals
- Questionnaire intake view
- Configuration page for module setup, integrations, browser extension settings, guardrails, and event routing

## Product Context

Employees use AI tools without IT or compliance knowing. The Shadow AI queue detects that activity and funnels admins toward governed workflows: vendor registration, AI system registration, risk/evidence collection, model tracking, and reviewed runtime events.

Detected Shadow AI object types:

| Type | Badge | Meaning |
| --- | --- | --- |
| `AI vendor` | Green | SaaS product with AI features |
| `Embedded AI` | Purple | AI inside an already approved tool |
| `AI API` | Pink | Direct API calls to AI providers |
| `Model` | Blue | Specific model invoked, for example Bedrock |
| `MCP` | Amber | Model Context Protocol server |
| `Unknown signal` | Grey | Unrecognized endpoint with AI-like traffic |

## UI Structure

```
Shell
|-- Sidebar - Sprinto nav, AI Systems active
|-- Main
    |-- Topbar - title, search, clock, AI icon
    |-- Main tabs
        |-- All AI systems
        |-- Models
        |-- Shadow AI
        |-- Events
        |-- Questionnaire
        |-- Configuration
    |-- Shadow AI view
        |-- Toolbar with View dropdown: Action required, Blocked, Violations, Ignored
        |-- Risk Level and Source filters
        |-- Discovery tables
        |-- Detail drawer with Overview, Network calls, Users, Risk pulse
        |-- Resolve flow: AI vendor -> Add vendor + Add AI system; all other types -> Create AI system or Link to AI system
        |-- Config drawer
    |-- All AI systems view
        |-- AI systems table
        |-- Add AI system menu: manual, CSV, codebase, vendor suggestions
        |-- Detail screen with Overview, Risk, Evidence, Findings, Tasks, Link view
    |-- Evidence tab
        |-- Required evidence table
        |-- Evidence drawer with version tabs, preview center, Details rail
    |-- Models view
        |-- Model table
        |-- Model detail drawer/screen and model library
    |-- Events view
        |-- Reviewed signal stream, not raw low-risk logs
        |-- Event detail drawer with Overview and Integration tabs
    |-- Questionnaire view
        |-- Questionnaire table for current, draft, requested, and suggested intake
    |-- Configuration view
        |-- Configuration cards matching Sprinto's module configuration pattern
        |-- Risk auto-scoring, field ordering, custom fields, document requests
        |-- Shadow AI discovery sources, browser extension settings, guardrails, event routing
```

## Design System

Keep the prototype close to Sprinto's compact admin UI:

```css
--color-background-primary: #ffffff;
--color-background-secondary: #f5f4f0;
--color-text-primary: #1c1917;
--color-text-secondary: #6b6963;
--color-text-tertiary: #a09d98;
--color-border-tertiary: rgba(28,25,23,0.12);
--color-border-secondary: rgba(28,25,23,0.22);
--font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-mono: 'SF Mono', 'Fira Code', monospace;
--border-radius-md: 8px;
--border-radius-lg: 12px;
```

Guidelines:

- Brand accent is `#E05C2A` for active navigation and main tab highlights.
- Icons use Tabler outline classes (`ti ti-*`) from CDN.
- Controls are compact: 11px to 12px text, 0.5px borders, 5px to 8px radii.
- Avoid marketing-style sections. This is an operational compliance surface.
- Do not add cards inside cards. Use cards for repeated items, modals, drawers, and framed work areas only.
- Keep table toolbar filters in one toolbar. Do not add duplicate status tabs.

## Shadow AI Discovery

Primary data array: `rows`.

Each discovery row contains:

```js
{
  name, ico, type, tpC,
  risk, rpC,
  src,
  first, last,
  calls, users,
  desc,
  nc,
  ur,
  sc, slv, ss,
  certs,
  res, trn,
  af,
  conf, confC
}
```

Key functions:

- `renderRows()`
- `openDetail(idx)`
- `openAdd('vendor'|'usecase', name)`
- `selectView(tableId, label, count, el)`
- `toggleViewDrop(event)`
- `openCfgDrawer()`
- `openExtCfg()`
- `backToInt()`
- `showT(msg)`

Resolution flow:

- If `type === 'AI vendor'`: show `Add vendor + AI system`.
- For every other detected type: show only `Add AI system` and `Link to existing AI system`.

The drawer step bar should show step 1 completed before step 2 activates only in the AI vendor flow.

Shadow AI table notes:

- Discovery tables include a `Violations` column between `Discovered on` and `Risk`.
- The Shadow AI View dropdown includes a `Violations` view for discoveries with non-zero violation counts.

## AI System Detail

Primary data array: `aiSystems`.

AI system detail is opened with `openAIDetail(idx)`. The main tab bar hides while the detail screen is open. The current detail tabs are:

- Overview
- Risk
- Evidence
- Findings
- Tasks
- Link view

The old "Due diligence" and "Documents & links" mental model has been replaced by the Evidence tab.

Risk tab behavior:

- If an AI system has an attached vendor, prefill mapped risks from that vendor.
- Prefilled risks should be visibly labeled `Inherited from <vendor>`.
- The `Add risks` action opens a full mapping modal with risk categories, searchable risk list, selected risks, and footer actions.
- Vendor-inherited risks should be preselected and locked in the mapping modal.
- If no vendor is attached, keep the empty state and prompt the user to attach a vendor before inheriting risks.

## Events

Events replaces the old top-level `Violations` surface. It is a reviewed signal stream, not a raw log stream.

Events should include:

- Guardrail violations
- Blocked actions
- Sensitive data events
- High-risk interactions
- Escalations
- Unresolved interactions that need Shadow AI or AI system resolution

Raw low-risk integration activity should not flood Events unless routing configuration says it should appear there.

Key functions:

- `renderEvents()`
- `openEventDrawer(i)`
- `closeEventDrawer()`
- `selectEventTab(el, id)`

## Configuration

Configuration is the module setup surface and should keep the existing Sprinto card pattern. Do not create a separate `Controls`, `Monitor`, or `Settings` tab inside this module.

Configuration cards:

- AI System risk auto-scoring
- AI System field ordering
- Custom fields available for AI Systems
- AI System document request
- Shadow AI discovery sources
- Browser extension settings
- Guardrails
- Event routing

Guardrails are declared in Configuration. Guardrail outcomes appear in Events. Policy/framework declarations stay in existing Sprinto modules outside this prototype; this module can show linked obligations/evidence but should not own the policy/framework source of truth.

## Evidence Tab

Primary data array: `complianceArtifacts`.

Each artifact is an ongoing compliance requirement. It can be an `Assessment` or `Document`.

Artifact-level fields stay stable across versions:

```js
{
  name,
  kind,
  icon,
  trigger,
  req,
  status,
  renewalState,
  monitor,
  dueText,
  owner,
  controlsMapped,
  source,
  tmpl
}
```

Version-level fields change per selected version:

```js
{
  v,
  status,
  date,
  by,
  note,
  fileName,
  fileType,
  answered,
  reviewer,
  submittedBy,
  submittedDate,
  changeType,
  changeReason,
  changeSummary
}
```

Evidence table columns:

```text
Requirement | Type | Recurrence | Status | Actions
```

Evidence table View dropdown:

```text
All
Suggested
Draft
Requested
Current
Due soon
Expired
Special case
```

Current Evidence status mapping:

- Empty `status` -> `Suggested`
- `draft` -> `Draft`
- `requested` -> `Requested`
- `completed` -> `Current`
- `renewalState: 'dueSoon'` -> `Due soon`
- `renewalState: 'expired'` -> `Expired`
- `status: 'specialCase'` -> `Special case`

Key functions:

- `renderCompliance()`
- `cmpStatusKey(a)`
- `cmpActionsHtml(a, i)`
- `openEvidenceDrawer(i, selection)`
- `renderEvidenceDrawer()`
- `evidenceTabsHtml(a)`
- `renderArtifactDetails(a)`
- `renderSelectedVersionDetails(a)`
- `renderSuggestedCenter(a, i)`
- `renderDraftCenter(a, i)`
- `renderRequestedCenter(a, i)`
- `renderVersionCenter(a, i, idx)`

## Evidence Drawer Layout

The drawer is intentionally not a 3-column picker. It uses:

- Top horizontal tabs for versions/states
- Center pane for immediate preview or next action
- Right rail titled `Details`

Center pane rules:

- Assessment evidence shows assessment preview directly.
- Document evidence shows document preview directly.
- Missing evidence shows action state: fill/upload/request/assign.
- Draft evidence shows preview plus continue/edit/submit.
- Requested evidence shows request status plus remind/upload manually.
- Archived versions are read-only.

Details rail rules:

- `Artifact` group: Recurrence, Owner, Controls mapped
- `Selected version` group: Last updated, Submitted by, Change type, Change reason, Summary
- If no version exists: show `Evidence state`
- Keep artifact owner separate from version submitter.

## Versioning Rules

`New version` is a choice modal, not an immediate version mutation.

Options:

- `Renew unchanged`
  - Requires a short comment.
  - Creates a new current version.
  - Archives the previous current version.
  - Sets `changeType: 'Renewed unchanged'`.
  - Sets `changeReason: 'Re-attestation'`.
  - Annual recurrence advances to next year.

- `Draft new version`
  - Creates `Draft vNext`.
  - Current version remains active until submit.
  - If a draft already exists, open the existing draft instead of creating another.
  - Submit requires `Change reason` and `Change summary`.
  - On submit, old current becomes archived and draft becomes current.
  - If submitted before due, use `changeType: 'Updated early'`; otherwise use `Material update`.

- `Request update`
  - Requires a recipient.
  - Creates a `Request sent` tab.
  - Current version remains active.
  - If owner is empty, requested person becomes owner.
  - If owner exists, keep current owner unless `Make requested person the owner` is checked.
  - Can be cancelled from the request tab.

Ownership rules:

- If the current user fills/uploads directly and owner is empty, set owner to current user (`_currentUser`).
- If someone else is requested and owner is empty, set requested person as owner.
- Artifact owner is shown in `Artifact`; version submitter is shown in `Selected version`.

## Special Case Workflow

Product decision:

- Platform language is `Mark as special case`, not `Mark as not applicable`.
- It is artifact-level, not version-level.
- It requires a comment.
- The artifact should move to a `Special case` state.
- `Special case` should be accessible from the Evidence table View dropdown.
- Drawer should show comment, marked by, marked date, and a `Reopen` action.

Current implementation status:

- Implemented in the Evidence drawer Details rail as an artifact-level action.
- `Mark as special case` requires a comment.
- The artifact moves to `Special case` and appears in the Evidence table View dropdown.
- The drawer shows a special-case center state, comment, marked by, and marked date.
- `Reopen` restores the prior artifact status/renewal state.
- `Edit comment` reopens the same modal with the existing comment.
- Do not attach special case to a selected version tab.

## Modal and Overlay Notes

Overlays are moved into `#shell` at the end of the script so they cover sidebar and main consistently. When adding a new modal/overlay, add its ID to the relocation list near the bottom of the file.

Current evidence-related overlays:

- `evidenceDrawerOv`
- `renewModal`
- `newVersionModal`
- `addDocsModal`
- `reqVendorModal`
- `questModal`
- `questFormOv`

## Validation and Testing

Useful checks:

```sh
node - <<'NODE'
const fs = require('fs');
const html = fs.readFileSync('shadow_ai_wireframe.html', 'utf8');
const scripts = [...html.matchAll(/<script[^>]*>([\s\S]*?)<\/script>/gi)].map(m => m[1]);
for (const [i, src] of scripts.entries()) {
  try { new Function(src); }
  catch (e) { console.error('script', i, 'syntax error:', e.message); process.exit(1); }
}
console.log('scripts ok:', scripts.length);
NODE
```

```sh
curl -I --max-time 5 http://localhost:8000/shadow_ai_wireframe.html
```

For browser smoke tests, use the bundled Playwright runtime if available:

```sh
NODE_PATH=/Users/ranjanpai/.cache/codex-runtimes/codex-primary-runtime/dependencies/node/node_modules \
/Users/ranjanpai/.cache/codex-runtimes/codex-primary-runtime/dependencies/node/bin/node <script>
```

Manual path to Evidence:

1. Open `All AI systems`
2. Click `Optimise data storage`
3. Click `Evidence`
4. Open an evidence row, for example `AI Impact Assessment`

## Known Placeholders

- Some Configuration `Manage` actions still toast instead of opening full management screens.
- Some buttons still toast instead of performing real state transitions.
- All data is hardcoded in the HTML file.
- Review/approval flows are intentionally out of scope for V1.

## Editing Guidance

- Keep changes scoped to `shadow_ai_wireframe.html` unless updating this guide.
- Preserve compact Sprinto styling.
- Use Tabler icon classes, not custom SVGs, unless there is no suitable icon.
- Keep Evidence workflows artifact/version scoped: artifact fields in artifact details, version fields in selected version details.
- Do not introduce review flows in V1.
- Do not claim integration-backed compliance proof unless the UI actually has that integration behavior.
- Keep top-level IA as `All AI systems | Models | Shadow AI | Events | Questionnaire | Configuration`.
- Avoid reintroducing top-level `Monitor`, `Controls`, or `Settings` terminology for this module.
