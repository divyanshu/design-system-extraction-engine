# Skill: extract-anatomy

Extract a component's prop API, variant matrix, slots, and states from available sources.

## Prerequisites

- `manifest.yaml` entry exists for the component
- At least one source is available (Storybook, npm package, or Figma MCP)

## Steps

### 1. Identify the component in available sources

**From Storybook:**
- Fetch `{storybook-url}/index.json`
- Find the component's story entries — these enumerate every variant combination
- Each story title maps to a prop value (e.g., story "Primary" under "Button" → `variant="primary"`)

**From npm package:**
- Locate the TypeScript prop interface in the target system's component package
- The interface name typically follows `{ComponentName}Props`
- Enum values are usually defined as TypeScript union types or enums

**From Figma MCP (co-primary source — use for every component when available):**
- Use `figma_get_component_details` to inspect variants and properties
- Variant names in Figma usually map directly to prop values
- Inspect both the Foundations library (tokens, color scales, spacing) and the Component library (anatomy, slot structure, visual states)
- **STABLE vs PROPOSED:** Figma frames may be tagged with readiness status.
  - `STABLE` = production-ready, safe to replicate as-is
  - `PROPOSED` = design in progress, may change — build it but note confidence as Medium or Low
  - When a component exists only as PROPOSED, infer the API conservatively and flag it in the manifest

### 2. Extract the variant matrix

List every prop that drives visual or structural variation:
- Prop name
- Type (enum | boolean | string | ReactNode)
- Allowed values
- Default value
- Whether it's required

For compound variants (e.g., Chip: `variant` × `color`), enumerate all valid combinations.

### 3. Identify slots

Slots are places in the component where the consumer can inject content:
- Required slots (e.g., `children`)
- Optional icon slots (e.g., `icon`, `iconBefore`, `iconAfter`)
- Action slots (e.g., `actions`, `footer`)
- Named sub-components (e.g., `DialogHeader`, `DialogFooter`)

### 4. Identify states

States are visual conditions the component can be in:
- `default`, `hover`, `active`, `focus`, `disabled`, `loading`, `error`, `selected`
- Check the token naming convention — tokens like `colorButtonBackgroundPrimaryDefault` tell you both the variant (`primary`) and state (`default`)

### 5. Record confidence and gaps

- **High**: Full prop interface available from npm + story matrix confirms all variants
- **Medium**: Partial prop interface, some variants inferred from Storybook stories
- **Low**: Only visual examples available, props must be reverse-engineered

Note any variants that exist in the target system but are unclear or undocumented.

## Output format

```yaml
component: "Button"
props:
  - name: "variant"
    type: "enum"
    values: [primary, secondary, destructive, outline, ghost]
    default: "primary"
    required: false
  - name: "size"
    type: "enum"
    values: [default, sm, lg, icon]
    default: "default"
    required: false
  - name: "disabled"
    type: "boolean"
    default: false
    required: false
  - name: "icon"
    type: "ReactNode"
    required: false
slots:
  - children (label text)
  - icon (leading icon)
states:
  - default
  - hover
  - pressed
  - disabled
confidence: "high"
confidence_gaps: "Loading state observed in docs but no token coverage found."
sources_used: [storybook, tokens-npm, components-npm]
```

## Notes

- **Token naming inference:** Many design systems follow a pattern like `{category}{Component}{Property}{Variant}{State}` — use this to infer props/states when docs are sparse
- **Private packages:** Some components may live in private npm packages with no accessible source. Infer the prop API from Figma anatomy + Storybook story names combined. Note confidence as Medium.
- **Anatomy specs location:** Write per-batch anatomy specs to `targets/{target}/specs/anatomy/` (e.g., `batch2-form-inputs.yaml`). Group related components into one file to keep the spec directory manageable.
- **Batch inspection approach:** When using Figma MCP, inspect pages in batches to manage token cost. Record node IDs in a plan document for resumable inspection.
