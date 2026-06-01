# Shadow AI Discovery Wireframe — Sprinto

## What this is
A high-fidelity interactive wireframe for the **Shadow AI discovery module** inside Sprinto's compliance platform. It lives under **AI Systems > Shadow AI** in the nav. The module is a discovery and reconciliation queue: it detects ungoverned AI activity across the org and guides admins to resolve it through a governed workflow.

Single file: `shadow_ai_wireframe.html` — fully self-contained, no build step needed. Open in any browser.

---

## Product context

### The problem it solves
Employees use AI tools (ChatGPT, Notion AI, Copilot, direct API calls, MCPs, etc.) without IT/compliance knowing. This creates data risk, audit gaps, and policy violations. The module surfaces that activity and funnels it into Sprinto's existing vendor and AI system register.

### Object types detected
| Type | Badge colour | Meaning |
|------|-------------|---------|
| `AI vendor` | Green | A SaaS product with AI features |
| `Embedded AI` | Purple | AI baked into an existing approved tool |
| `AI API` | Pink | Direct REST calls to AI provider APIs |
| `Model` | Blue | A specific model invoked (e.g. via Bedrock) |
| `MCP` | Amber | Model Context Protocol server |
| `Unknown signal` | Grey | Unrecognised endpoint with AI-like pattern |

### Resolution flow (two steps)
1. **Add vendor** → registers the vendor in Sprinto's vendor register (name, website, category)
2. **Add AI system** → registers the use case (owner, business objective, type: internal/third-party, models used)

The step bar in the drawer shows progress: Step 1 filled dark → Step 2 outlined muted → on save step 1 gets a checkmark and step 2 activates.

---

## UI structure

```
Shell (590px height)
├── Sidebar (196px) — Sprinto nav, "AI Systems" active
└── Main
    ├── Topbar — "AI Systems" + search/clock/AI icons
    ├── Main tabs — All AI systems | Shadow AI (active) | Violations | Questionnaire
    └── Shadow AI view
        ├── Sub-bar — Action required (7) | Blocked (2) | Ignored (5) | [Configure →]
        ├── Toolbar — filters + search/export/columns
        ├── Table — rows of detected AI activity
        ├── Context menu — Ignore / Block (three-dot)
        ├── Detail drawer (360px) — Overview / Network calls / Users / Risk pulse tabs
        │   └── Add flow — Step 1: Add vendor, Step 2: Add AI system
        └── Config drawer (420px)
            ├── Tab: Integrations — Browser Extension (expanded) + AWS Bedrock + ChatGPT Ent + available integrations
            ├── Tab: Preferences — single toggle: "Show AI usage for approved vendors as well"
            └── Panel: Browser Extension configure (replaces tabs, back button to return)
                ├── Detection scope — per-vendor domain toggles + custom domains
                ├── Data capture — capture level + privacy toggles + user identity
                ├── Detection sensitivity — threshold + new vendor alert + personal key alert
                └── Exclusions — approved domain chip list
```

---

## Design system / CSS variables

```css
--color-background-primary: #ffffff
--color-background-secondary: #f5f4f0
--color-text-primary: #1c1917
--color-text-secondary: #6b6963
--color-text-tertiary: #a09d98
--color-border-tertiary: rgba(28,25,23,0.12)
--color-border-secondary: rgba(28,25,23,0.22)
--font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
--font-mono: 'SF Mono', 'Fira Code', monospace
--border-radius-md: 8px
--border-radius-lg: 12px
```

Brand accent: `#E05C2A` (orange) — used for active nav item indicator and active main tab.
Icons: Tabler Icons outline webfont via CDN (`ti ti-*` classes). No filled variants.

Toggle component: `.tog.on` = `background:#1C1917`, `.tog.off` = `background:var(--color-border-secondary)`. Knob via `::after` pseudo-element.

---

## Data model (JS `rows` array)

Each row in the table has:
```js
{
  name, ico, type, tpC,       // display name, tabler icon class, type label, badge class
  risk, rpC,                  // risk level label, risk pill class
  src,                        // detection source (Browser Extension / AWS Bedrock / etc.)
  first, last,                // first/last detected date strings
  calls, users,               // total call count, affected user count (null for agents)
  desc,                       // description paragraph for Overview tab
  nc: [                       // network calls: [method, endpoint, timestamp, user]
    ['POST', '/api/v1/...', '25 May 14:32', 'AR']
  ],
  ur: [                       // users: [initials, name, email, av-class, calls, last-date]
    ['AR', 'Arjun Rao', 'arjun@...', 'av-a', 88, '25 May']
  ],
  sc, slv, ss,                // risk score (0–100), risk level label, risk summary
  certs: [['name', 'y/n']],  // certification list for Risk pulse tab
  res, trn,                   // data residency, training policy
  af: 'vendor' | 'usecase'   // which Add flow to open
}
```

---

## Key interactions to know

- **Row click** → opens detail drawer (`openDetail(idx)`)
- **Add button** → opens Add flow directly (`openAdd('vendor'|'usecase', name)`)
- **Three-dot menu** → context menu positioned relative to `#vShadow` using `getBoundingClientRect()`
- **Configure button** (sub-bar) → opens config drawer (`openCfgDrawer()`)
- **Browser Extension → Configure** → hides the cfg-tabs bar, shows `#cExtCfg` panel (`openExtCfg()`)
- **← Integrations back button** → restores tabs, shows `#cInt` (`backToInt()`)
- **Toast** → `showT(msg)` — appears at bottom, auto-dismisses after 3s

---

## Things that are intentionally placeholder
- "All AI systems", "Violations", "Questionnaire" tabs → show a placeholder div, not wired up
- Blocked tab has only one row (DeepSeek), Ignored has one row (Grammarly)
- "Send install reminder" and "View installed devices" buttons → toast only
- No real API calls; all data is hardcoded in the `rows` array

---

## Possible next areas to work on
- Flesh out the Blocked and Ignored tab rows
- "All AI systems" tab content (the full governed register view)
- Violations tab
- More detail in the Add vendor / Add AI system forms (e.g. field validation, inline errors)
- Risk pulse tab enhancements (trend chart, historical data)
- Responsive behaviour or layout at larger widths
- Dark mode pass
- Real component extraction (break out into HTML/CSS/JS files or a React component structure)
