# Design System Replication Workflow

## Overview

A portable, agent-friendly system for taking **shadcn/ui** as a baseline reference system and systematically transforming it to faithfully reproduce any target design system (e.g., Atlassian Design System, Salesforce Lightning, IBM Carbon) at **80–90% visual and structural fidelity** — sufficient for high-quality prototyping and proof-of-concept work.

The system lives as a **repo** containing a themed shadcn/ui baseline plus a set of codified **skills** (structured instructions any capable AI agent can follow). Skills are compatible with Claude and Gemini as primary agents.

---

## Design Principles

1. **Prototyping-first fidelity.** Target 80–90% accuracy. Prioritise visual and structural fidelity over pixel-perfect behavioral fidelity (animation curves, keyboard edge-cases). Good enough to demo, not expected to ship as-is.
2. **Human-in-the-loop, always.** The agent extracts what it can, reports confidence, and the human fills gaps. No phase runs fully autonomously.
3. **Confidence transparency.** Every extracted or replicated artefact carries a confidence score. The human always sees where the agent is uncertain and can choose to invest effort (and tokens) selectively.
4. **Progressive enrichment.** Start with publicly available sources (docs, Storybook, published tokens). Layer in richer sources (Figma via MCP, source code) when the human provides access and when confidence needs to improve.
5. **Agent portability.** Skills are written as structured markdown instructions that any sufficiently capable agent (Claude, Gemini) can follow. No agent-specific tool calls baked into the core skill definitions.

---

## Terminology

| Term | Definition |
|------|-----------|
| **Reference system** | The baseline headless component library we build on top of. Default: shadcn/ui. Headless primitives: `@base-ui/react` (Radix UI successor — shadcn's stated direction). Uses `data-checked`/`data-unchecked` data attributes, not Radix-style `data-state="checked"`. |
| **Target system** | The design system we are replicating. e.g., Atlassian Design System (ADS). |
| **Skill** | A structured, step-by-step instruction document an AI agent follows to perform a specific task. |
| **Manifest** | A structured inventory of every component in the target system, its variants, tokens, mapping classification, and current replication status. |
| **Confidence score** | A per-component rating (High / Medium / Low) indicating how complete and accurate the extraction or replication is, based on source quality and mapping complexity. |
| **Bolt-on library** | A supplementary headless library (e.g., Recharts for charts, TanStack Table for data grids) integrated and themed to fill gaps the reference system doesn't cover. |

---

## Workflow Phases

The workflow is **iterative, not linear**. The core loop is:

```
Survey → Extract → Replicate → Validate → Human Review → Refine
   ↑                                                         |
   └─────────────────────────────────────────────────────────┘
```

Each phase can feed back into earlier phases as confidence improves.

---

### Phase 0: Input & Source Gathering

**Purpose:** The agent and human collaboratively establish what sources are available and accessible for the target design system.

**How it works:** The agent asks the human to provide sources incrementally. Not everything needs to be available upfront — the workflow is designed to start with whatever is available and enrich over time.

**Source types (in order of typical availability):**

| Source | What it provides | Access method |
|--------|-----------------|---------------|
| Design system website/docs | Component list, usage guidelines, visual examples, token documentation | Agent scrapes/reads public URLs |
| Storybook instance | Live component demos, variant matrices, prop documentation | Agent reads public Storybook URL |
| Published token packages | Design tokens as JSON/CSS | Agent reads npm package or CDN |
| Figma libraries | Component anatomy, variants, spacing, visual specs | Human provides via Figma MCP or exports |
| Source code (GitHub/internal) | Component implementation, prop interfaces, behavior logic | Human provides repo access or pastes relevant code |
| Brand guidelines PDF/page | Logo usage, color palettes, typography scales | Human uploads or provides URL |

**Output:** A `sources.md` file listing every available source, its access method, and what it covers.

**Human role:** Provide URLs, grant access, upload files. Flag any sources that are internal-only and need manual extraction.

---

### Phase 1: Survey (Target System Audit)

**Purpose:** Produce a structured manifest of the target design system — every component, its variants, and a preliminary mapping against shadcn/ui.

**Skill: `survey-target-system`**

The agent works through available sources and produces a draft manifest. The human reviews, corrects, and annotates it.

**Manifest structure (per component):**

```yaml
- name: "Button"
  target_system_name: "Button"  # name in target system
  category: "Actions"
  variants:
    - appearance: [primary, subtle, warning, danger, link]
    - spacing: [compact, default]
    - icon_position: [before, after, icon-only]
    - loading: [true, false]
  mapping_type: "direct_match"  # see taxonomy below
  shadcn_equivalent: "Button"
  confidence: "high"
  confidence_notes: "All variants documented in public Storybook"
  sources_used: ["docs", "storybook"]
  sources_missing: ["figma"]
```

**Component Mapping Taxonomy:**

| Mapping Type | Description | Example | Replication Approach |
|-------------|-------------|---------|---------------------|
| `direct_match` | Target component has a clear shadcn equivalent with similar structure | ADS Button → shadcn Button | Theme + extend variants |
| `partial_match` | Structurally similar but different API surface, slots, or behavior | ADS Modal → shadcn Dialog | Fork shadcn component, modify structure |
| `structural_mismatch` | Concept exists in both but component boundaries differ | ADS "Inline Dialog" vs shadcn Popover + Dialog | Decompose target concept, recombine from shadcn primitives |
| `target_only` | Component exists only in target system | ADS Inline Edit | Build from scratch using shadcn primitives + Radix |
| `bolt_on` | Capability gap that requires a supplementary library | Charts, data grids, rich text | Integrate external headless library, theme to match |

**Confidence scoring criteria:**

| Score | Meaning |
|-------|---------|
| **High** | Full variant matrix documented, visual examples available, tokens published. Agent can replicate with minimal human input. |
| **Medium** | Partial documentation. Some variants or states unclear. Agent can make reasonable inferences but human should review. |
| **Low** | Minimal documentation. Component exists in target but spec is incomplete. Human input required before replication. |

**Human role:** Review draft manifest. Add components the agent missed (especially internal-only ones). Correct mapping types. Flag components that are out of scope for the current prototype.

**Output:** `manifest.yaml` — the work order for all subsequent phases.

---

### Phase 2: Extract

**Purpose:** For each component in the manifest, extract the detailed specification the agent needs to replicate it.

Three sub-skills, each producing a different type of spec:

#### Sub-skill: `extract-tokens`

**What:** Pull design tokens from the target system.

**Sources (in priority order):**
1. Published token package (npm, CDN)
2. Design system docs token reference page
3. Figma styles (via MCP or export)
4. Manually inspected from Storybook/component examples

**Output:** A structured token file mapping target system tokens → CSS custom properties compatible with shadcn/ui's theming layer.

```css
/* Example: ADS tokens mapped to shadcn CSS custom properties */
:root {
  --background: 255 255 255;        /* N0 */
  --foreground: 23 43 77;           /* N800 */
  --primary: 0 82 204;              /* B400 */
  --primary-foreground: 255 255 255;
  --destructive: 222 53 11;         /* R400 */
  --muted: 244 245 247;             /* N20 */
  --radius: 0.1875rem;              /* 3px - ADS default border radius */
  /* ... */
}
```

**Confidence reporting:** Agent flags any tokens that were inferred rather than directly extracted, and which source they came from.

#### Sub-skill: `extract-anatomy`

**What:** For each component, document its props/API, variant matrix, slot structure, and composition patterns.

**Output per component:** A structured component spec.

```yaml
component: "Button"
props:
  - name: "appearance"
    type: "enum"
    values: ["primary", "default", "subtle", "link", "warning", "danger"]
    default: "default"
  - name: "spacing"
    type: "enum"
    values: ["compact", "default"]
    default: "default"
  - name: "isDisabled"
    type: "boolean"
    default: false
  - name: "isLoading"
    type: "boolean"
    default: false
  - name: "iconBefore"
    type: "ReactNode"
    required: false
  - name: "iconAfter"
    type: "ReactNode"
    required: false
slots:
  - "children (label text)"
  - "iconBefore"
  - "iconAfter"
  - "loadingSpinner (internal)"
states:
  - "default"
  - "hover"
  - "active"
  - "focus"
  - "disabled"
  - "loading"
confidence: "high"
confidence_gaps: "Loading spinner animation timing not documented"
```

**Human role:** Review specs. For `medium` or `low` confidence components, provide additional input — paste Figma specs, share code snippets, or clarify variant behavior.

#### Sub-skill: `extract-behavior`

**What:** Document interaction patterns — keyboard handling, focus management, state transitions, animation specs.

**Note for prototyping scope:** This sub-skill is **lower priority** at 80–90% fidelity. The agent should extract what's easily available (documented keyboard shortcuts, visible transitions) but not spend excessive effort reverse-engineering undocumented behavior.

**Output per component:** A lightweight behavioral spec.

```yaml
component: "Dropdown Menu"
keyboard:
  - "Enter/Space: open menu, activate focused item"
  - "Arrow Down/Up: navigate items"
  - "Escape: close menu"
  - "Tab: close menu and move focus"
focus_management: "Focus trapped within menu when open. Returns to trigger on close."
animations:
  - "Open: fade + scale, ~150ms ease-out (estimated)"
  - "Close: fade, ~100ms ease-in (estimated)"
confidence: "medium"
confidence_gaps: "Animation durations estimated from visual inspection. Focus behavior inferred from Radix defaults."
```

**Human role:** Only engage here if specific behavioral fidelity matters for the prototype (e.g., a demo that specifically showcases interaction design).

---

### Phase 3: Replicate

**Purpose:** Build the themed and customised components in the reference system.

Four sub-skills based on the mapping taxonomy:

#### Sub-skill: `replicate-theme`

**For:** `direct_match` components.

**What:** Apply token overrides and add variant extensions to existing shadcn components without forking them.

**Approach:**
1. Apply extracted tokens via CSS custom properties (shadcn's native theming)
2. Extend variant definitions in the component's `cva` (class-variance-authority) config
3. Add any missing variant combinations
4. Update default props to match target system defaults

**Output:** Modified shadcn component files + updated theme CSS.

#### Sub-skill: `replicate-extend`

**For:** `partial_match` and `structural_mismatch` components.

**What:** Fork the shadcn component and modify its structure, slots, or composition to match the target spec.

**Approach:**
1. Copy the shadcn component source
2. Modify the component anatomy (add/remove slots, change composition)
3. Update the prop interface to match the target system's API
4. Apply visual styling from extracted tokens + anatomy spec
5. Document what was changed and why

**Output:** New component files in the repo, clearly marked as forked/extended.

#### Sub-skill: `replicate-build`

**For:** `target_only` components.

**What:** Build new components from scratch using shadcn primitives and Radix UI building blocks.

**Approach:**
1. Review the extracted anatomy and behavioral specs
2. Identify which Radix primitives can serve as a foundation
3. Build the component following shadcn's authoring conventions (cva, cn utility, Tailwind)
4. Apply tokens and styling
5. Implement core interaction behavior (prioritise what's visible in the prototype)

**Output:** New component files following shadcn conventions.

#### Sub-skill: `replicate-integrate`

**For:** `bolt_on` components (charts, data grids, rich text editors, etc.)

**What:** Identify, integrate, and theme supplementary headless libraries.

**Recommended bolt-on libraries:**

| Capability Gap | Recommended Library | Rationale |
|---------------|-------------------|-----------|
| Charts & data viz | Recharts or Tremor | Already common in shadcn ecosystem, Tailwind-friendly |
| Data tables (advanced) | TanStack Table | Headless, composable, pairs well with shadcn Table |
| Rich text editing | Tiptap | Headless, extensible, good React integration |
| Date/time picking | react-day-picker | Already used by shadcn's Calendar component |
| Drag and drop | dnd-kit | Headless, accessible, framework-agnostic |

**Approach:**
1. Identify the gap from the manifest
2. Select the best bolt-on library
3. Create a themed wrapper that applies the target system's tokens
4. Build any target-system-specific compositions on top

**Output:** Integrated library with themed wrapper components.

---

### Phase 4: Validate

**Purpose:** Verify that replicated components match the target system at the required fidelity level.

**Guiding principle:** Validation is **human-driven and cost-conscious**. The agent provides automated baseline checks, and the human decides which components warrant deeper inspection.

#### Sub-skill: `validate-visual-diff`

**What:** Automated screenshot comparison between replicated components and the target system's reference images.

**Approach:**
1. Render each replicated component in its key states/variants using a local Storybook or test harness
2. Capture screenshots via Playwright
3. Compare against reference screenshots (from target system's Storybook, docs, or human-provided)
4. Generate a visual diff report with percentage-match scores

**Output:** A diff report per component with side-by-side comparisons and match scores.

```yaml
component: "Button"
variants_tested: 12
visual_match_average: 87%
issues:
  - variant: "appearance=warning"
    match: 72%
    note: "Border color slightly off — likely token mapping issue"
  - variant: "spacing=compact, isLoading=true"
    match: 65%
    note: "Loading spinner size and position differ"
```

**Human role:** Review the diff report. Decide which issues matter for the prototype and which to accept. For low-match variants, provide additional reference material or accept the deviation.

#### Sub-skill: `validate-human-review`

**What:** A structured checklist the human walks through per component, guided by the agent.

**The agent presents each component one at a time and asks:**
1. Does the default state look right?
2. Do the key variants look right?
3. Are there any obvious interaction issues?
4. Is this good enough for prototyping, or does it need refinement?

**Output:** Updated manifest with per-component sign-off status (approved / needs-refinement / blocked).

**Cost note:** This is the primary validation method. Visual diffing is a useful accelerator, but human review is the source of truth for "good enough." The human can skip visual diffing entirely and go straight to manual review if they want to conserve tokens.

#### Sub-skill: `validate-token-audit`

**What:** Automated scan that verifies token application across all components.

**Approach:**
1. Render components and inspect computed styles
2. Compare against the extracted token file
3. Flag any hardcoded values that should be using tokens
4. Flag any tokens that are defined but unused

**Output:** Token coverage report.

---

## Repo Structure

The repo separates **core machinery** (skills, templates, shared tooling) from **per-target work** (extraction output, replicated components, validation). Each target system lives in its own directory under `targets/`, making it easy to spin up new replication experiments, archive stale ones, or compare across systems.

```
design-system-replica/
├── README.md                              # Setup, usage, how to run with an agent
│
├── skills/                                # Core agent-readable skill files (target-agnostic)
│   ├── 00-input-source-gathering.md
│   ├── 01-survey-target-system.md
│   ├── 02-extract-tokens.md
│   ├── 03-extract-anatomy.md
│   ├── 04-extract-behavior.md
│   ├── 05-replicate-theme.md
│   ├── 06-replicate-extend.md
│   ├── 07-replicate-build.md
│   ├── 08-replicate-integrate.md
│   ├── 09-validate-visual-diff.md
│   ├── 10-validate-human-review.md
│   └── 11-validate-token-audit.md
│
├── templates/                             # Starter templates for new targets
│   ├── manifest.yaml                      # Empty manifest template
│   ├── sources.md                         # Empty sources template
│   └── registry.json                      # Starter registry.json for output
│
├── shared/                                # Shared utilities and config
│   ├── lib/                               # cn utility, shared helpers
│   ├── storybook-config/                  # Base Storybook config (reused per target)
│   └── scripts/                           # Build, validation, and publish scripts
│
└── targets/                               # All target system replications live here
    │
    ├── atlassian-ds/                       # Example: Atlassian Design System
    │   ├── README.md                       # Target-specific notes, status, decisions
    │   ├── manifest.yaml                   # Component inventory + mapping + status
    │   ├── sources.md                      # Available sources for this target
    │   ├── preset.txt                      # shadcn preset code (if generated)
    │   │
    │   ├── specs/                          # Extracted component specifications
    │   │   ├── anatomy/                    # Per-component anatomy specs
    │   │   └── behavior/                   # Per-component behavior specs
    │   │
    │   ├── tokens/                         # Extracted and mapped design tokens
    │   │   ├── target-raw/                 # Raw tokens from target system
    │   │   └── theme.css                   # Mapped tokens for shadcn theming
    │   │
    │   ├── registry/                       # shadcn registry structure (for output)
    │   │   ├── registry.json               # Registry definition for this target
    │   │   └── ads/                        # Replicated components as registry items
    │   │       ├── ui/                     # Themed/extended shadcn components
    │   │       ├── custom/                 # Target-only components (built from scratch)
    │   │       └── integrations/           # Bolt-on library wrappers
    │   │
    │   ├── validation/                     # Validation outputs
    │   │   ├── screenshots/                # Component screenshots
    │   │   ├── references/                 # Target system reference screenshots
    │   │   └── reports/                    # Diff reports, token audits
    │   │
    │   └── .storybook/                     # Target-specific Storybook config/overrides
    │
    ├── salesforce-lightning/                # Example: another target
    │   ├── README.md
    │   ├── manifest.yaml
    │   ├── sources.md
    │   └── ...                             # Same internal structure as above
    │
    └── _archive/                           # Stale or abandoned experiments
        └── ibm-carbon-v1/                  # Moved here when no longer active
```

**Key design decisions:**

- **`skills/` is target-agnostic.** The same set of skills works for any target system. Target-specific context comes from the manifest, sources, and specs within each target directory.
- **`targets/` groups all replication work.** Each target is self-contained — you can zip up `targets/atlassian-ds/` and hand it to someone, or delete it entirely without affecting other targets.
- **`registry/` inside each target** is structured for `shadcn build` compatibility from the start. When you're ready to publish, you run the build from within the target directory and the output is a deployable registry.
- **`templates/`** provides empty starter files so spinning up a new target is fast — copy the templates into a new `targets/<name>/` directory and start the Survey phase.
- **`_archive/`** is a convention for parking old experiments. No special tooling — just move the directory. Git history is preserved.
- **`shared/`** contains anything that's reused across targets but isn't a skill — utilities, base Storybook config, build scripts. Targets reference these rather than duplicating them.

---

## Friction Points & Known Risks

### Technical Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| **Target system docs are incomplete or inconsistent** | Extraction confidence is low, replication guesses wrong | Lean on multiple sources. Agent flags low confidence. Human provides Figma/code when needed. |
| **shadcn component updates break customisations** | Forked or extended components drift from upstream | Pin shadcn versions. Document all modifications. Re-validate after upgrades. |
| **Bolt-on libraries have conflicting styling approaches** | Themed wrappers are fragile or inconsistent | Standardise all bolt-on theming through CSS custom properties. Avoid library-specific theme APIs. |
| **Screenshot diffing is noisy** | Font rendering, anti-aliasing, and subpixel differences generate false positives | Set tolerance thresholds. Use structural comparison (layout, color values) over pixel-perfect matching. |

### Process Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| **Scope creep toward production fidelity** | Time and token cost explodes | Manifest tracks fidelity tier per component. "Good enough for prototype" is an explicit sign-off state. |
| **Target system versioning** | Target system ships updates, replica drifts | Track target system version in manifest. Re-run Survey periodically to detect changes. |
| **Agent token cost on large systems** | Full extraction + replication of 100+ components is expensive | Prioritise components by prototype need. Use manifest to track what's in scope vs. deferred. |
| **Agent capability variance** | Claude and Gemini have different strengths (vision, code execution, context length) | Skills are written as structured markdown any agent can follow. Agent-specific automation is optional. |

### Fidelity Gaps to Accept at 80–90%

These are things that will likely **not** be accurate in a prototype-grade replica, and that's fine:

- Animation timing and easing curves (unless specifically important to the prototype)
- Keyboard navigation edge cases in complex components
- Exact font rendering (system font stacks differ across OSes)
- Responsive breakpoint behavior of individual components
- Accessibility attributes beyond what Radix provides by default
- State management patterns that are internal to the target system's implementation

---

## Workflow Execution Sequence

For a typical engagement (replicating a target system for prototyping):

### Sprint 0: Setup (1–2 hours, human + agent)
1. Human creates repo from template (or uses existing repo if already set up)
2. Create a new target directory: copy `templates/` into `targets/<target-name>/`
3. Run `00-input-source-gathering` skill — human provides available sources
4. Run `01-survey-target-system` skill — agent produces draft manifest
5. Human reviews and annotates manifest, marks in-scope components

### Sprint 1: Foundation (2–4 hours, agent-heavy)
1. Run `02-extract-tokens` for the full target system
2. Apply tokens to shadcn base theme
3. Human reviews themed baseline components (just visually scan — do they feel right?)

### Sprint 2: Component Replication (iterative, per-component or per-batch)
1. For each in-scope component, run the appropriate extract sub-skill
2. Agent reports confidence; human supplements where needed
3. Run the appropriate replicate sub-skill based on mapping type
4. Agent shows result; human gives quick pass/fail

### Sprint 3: Validation & Refinement (human-driven)
1. Run `09-validate-visual-diff` on completed components (optional, for components where visual precision matters)
2. Run `10-validate-human-review` as the primary sign-off
3. For any "needs refinement" components, loop back to extract/replicate with additional human input
4. Run `11-validate-token-audit` as a final consistency check

---

## Impact of shadcn CLI v4 (March 2026)

The release of shadcn CLI v4 significantly simplifies several parts of this workflow. Four features are directly relevant:

### 1. `registry:base` — Solves the Output Distribution Problem

shadcn registries can now distribute an entire design system as a single payload using the `registry:base` type — components, dependencies, CSS vars, fonts, and config in one install. This is essentially a first-party solution to our "how do we pull the replicated system into prototype projects" problem.

**What this means for us:** Instead of building custom publish tooling, the output of our Replicate phase can be structured as a shadcn registry. Prototype projects install individual components via `npx shadcn add @yourorg/ads-button` or the full base in one shot. This is Option D from our original thinking, except shadcn has already built the tooling.

### 2. Presets — Simplifies Token Application

A preset encodes an entire design system config (colors, theme, icon library, fonts, radius) into a short code string. Projects can switch presets with a single command: `npx shadcn init --preset <code>`. The `shadcn/create` tool lets you preview how a preset applies to real components before committing.

**What this means for us:** Our `extract-tokens` skill can output a shadcn preset in addition to (or instead of) raw CSS custom properties. The `replicate-theme` skill for `direct_match` components becomes significantly simpler — apply the preset, then extend variants. Preset codes are also portable across agents and team members.

### 3. `shadcn/skills` — Agent Foundation Layer

shadcn now ships an official skill that teaches AI agents how to work with shadcn components and the CLI correctly. It covers Radix and Base UI primitives, component patterns, registry workflows, and knows when to invoke the CLI and which flags to pass. It runs `shadcn info --json` to detect project configuration.

**What this means for us:** Our skills can build on top of `shadcn/skills` as a foundation rather than re-teaching agents how shadcn works. The agent already knows how to add components, use registries, and follow shadcn conventions. Our skills only need to focus on the delta — the target-system-specific extraction and replication logic.

### 4. `--dry-run`, `--diff`, and `--view` — Validation Tooling

These flags let you inspect what a registry will add to your project before anything is written. `--diff` shows changes against existing files. `--view` displays the full payload.

**What this means for us:** For the Validate phase, `--diff` can serve as an additional check when updating replicated components. It also supports the human-in-the-loop review process — the human can preview exactly what will change before accepting an update.

### How This Changes the Plan

The core workflow (Survey → Extract → Replicate → Validate) remains the same. What changes is the infrastructure around it:

- **Replicate phase** gets simpler for `direct_match` components — presets handle the baseline, we extend from there.
- **Output distribution** is now a solved problem — use shadcn's registry system natively.
- **Agent skills** can reference `shadcn/skills` as a dependency, reducing what we need to teach agents from scratch.
- **Repo structure** should be designed from the start to be registry-compatible, with a `registry.json` at the root and components structured to support `shadcn build`.

The features that **don't** change are the hard parts: extraction from target systems, handling component mapping mismatches, building `target_only` components, and behavioral fidelity. Those are still our core problem space.

---

## Output Strategy: Internal Git Repo (Chosen Approach)

### The Problem

The extractor repo contains everything — skills, specs, validation infrastructure, and the replicated components. When a prototype developer wants to use the design system, they shouldn't need to clone the whole extractor. They want just the clean, usable component files.

A hosted shadcn registry (served over HTTP) is the ideal long-term solution but requires infrastructure — a server that's accessible to all developers, which is a constraint in internal/corporate environments.

### Chosen Approach: Output Git Repo

Two repos with a one-way relationship. The extractor runs a simple export script to populate the output repo.

```
┌──────────────────────────────┐        ┌──────────────────────────────┐
│  EXTRACTOR REPO              │        │  OUTPUT REPO                 │
│  (this repo — the machine)   │ export │  {target}-components         │
│                              │ ─────► │  (the product)               │
│  skills/                     │        │                              │
│  specs/                      │        │  components/                 │
│  manifest.yaml               │        │    ├── button.tsx            │
│  tokens/                     │        │    ├── chip.tsx              │
│  registry/{target}/          │        │    ├── callout.tsx           │
│    ├── ui/                   │        │    └── ...                   │
│    ├── extensions/           │        │  tokens/                     │
│    ├── custom/               │        │    └── theme.css             │
│    └── integrations/         │        │  README.md                   │
└──────────────────────────────┘        └──────────────────────────────┘
  You and the agent work here             Prototype devs clone this
```

### How Prototype Devs Use the Output Repo

```bash
# 1. Clone the output repo (once)
git clone git@github.com:yourorg/{target}-components.git

# 2. Copy the files they need into their Next.js project
cp {target}-components/components/button.tsx src/components/{target}/
cp {target}-components/tokens/theme.css src/app/

# 3. That's it — components are plain .tsx files, no magic
```

Or, for a full setup, copy everything at once:
```bash
cp -r {target}-components/components/ src/components/{target}/
cp {target}-components/tokens/theme.css src/app/globals.css  # or import it
```

### Why This Beats a Hosted Registry for Internal Use

| | Hosted Registry | Output Git Repo |
|---|---|---|
| Needs a server | Yes | No |
| Works offline / behind firewall | No | Yes |
| Version control | Manual | Free via git tags |
| Devs can inspect source | Only post-install | Always visible |
| Diff/review changes | Hard | Native `git diff` |
| Setup complexity | Medium | Zero |
| Works for internal corporate use | Needs internal hosting | Yes |

The main trade-off is no `npx shadcn add` magic — devs copy files manually. For prototyping (not production shipping), this is completely acceptable.

### Export Script

The extractor repo will have a script at `shared/scripts/export.sh` that:
1. Reads all `signed_off` components from `manifest.yaml`
2. Copies their files from `targets/{target}/registry/{target}/` into a clean output directory
3. Copies `targets/{target}/tokens/theme.css`
4. Writes a `README.md` with setup instructions

This runs whenever a new batch of components is signed off. The output repo is updated by pushing the result.

### Output Repo Structure

```
{target}-components/
├── README.md                    ← Setup instructions for prototype devs
├── components/                  ← All signed-off components as plain .tsx
│   ├── button.tsx
│   ├── chip.tsx
│   ├── callout.tsx
│   └── ...
├── tokens/
│   └── theme.css                ← All target tokens as CSS custom properties
└── showcase/                    ← Browsable component reference (Next.js app)
    ├── src/app/
    │   ├── page.tsx             ← Thin layout wrapper
    │   ├── globals.css          ← All tokens loaded
    │   ├── _components/
    │   │   ├── showcase-sidebar.tsx    ← Sticky sidebar with scroll-spy
    │   │   ├── section-wrapper.tsx     ← Section shell + mapping-type badge
    │   │   └── sections/               ← One file per category
    │   │       ├── foundations.tsx     ← Color swatches, type scale
    │   │       ├── actions.tsx
    │   │       └── ...
    └── src/components/{target}/ ← Symlinked or copied from components/
```

The showcase replaces Storybook for output repo consumers — no build tooling required beyond `npm run dev`.

### When to Create the Output Repo

Create the output repo once you have ~20-25 components signed off — enough coverage for typical prototype work (buttons, inputs, layout, messaging, navigation).

### Deferred Decisions

- **Versioning:** Git tags (`v1.0`, `v1.1`) are sufficient for now. Semver package versioning is overkill until teams depend on it.
- **Partial updates:** Re-export everything on each release — simpler than delta tracking. Git diff shows what changed.
- **Storybook in output repo:** Replaced by the custom showcase app. Storybook adds weight and setup cost; the showcase is purpose-built for this system.

