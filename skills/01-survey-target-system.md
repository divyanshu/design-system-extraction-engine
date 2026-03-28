# Skill: survey-target-system

Produce or update the manifest for a target design system — every component, its variants, mapping type, and confidence score.

## When to use

- Starting a new target system
- Adding newly discovered components to an existing manifest
- Updating manifest entries after better sources become available

## Steps

### 1. Get the component inventory

**From Storybook (if available):**
Fetch `{storybook-url}/index.json` — many design systems publish a machine-readable index of all components and their stories.

Parse each entry:
- `title` → component name and category (e.g., `"Actions/Button"`)
- Story names → variant enumeration (e.g., `"Primary"`, `"Destructive"`)

**From npm package (if available):**
Read the target system's component npm package exports. Look for TypeScript interfaces, exported component names, and enum types.

**From Figma MCP (if available):**
Use `figma_get_component_details` to inspect component pages. Each page typically maps to one component family.

**Fallback:** Manually browse the design system docs or showcase site and list components by hand.

### 2. For each component, determine the mapping type

Ask: does shadcn/ui have an equivalent?

| Situation | Mapping type |
|---|---|
| shadcn has the same component with the same structure | `direct_match` |
| shadcn has a similar component but different slots/API | `partial_match` |
| shadcn has a conceptually related component but component boundaries differ | `structural_mismatch` |
| No shadcn equivalent — needs to be built from scratch | `target_only` |
| Requires an external library (charts, tables, drag-and-drop) | `bolt_on` |

### 3. Assign confidence

- **High**: Multiple sources confirm the variant matrix. Token names match. Can build without additional input.
- **Medium**: One source available (usually Storybook stories), some gaps. Can build with reasonable inference.
- **Low**: Minimal documentation. Complex interaction model or undocumented internals. Human input required.

### 4. Write the manifest entry

```yaml
- name: "{ComponentName}"
  target_system_name: "{ExactNameInTargetSystem}"
  category: "{Category}"
  variants:
    - {propName}: [{value1}, {value2}]
  mapping_type: "{direct_match|partial_match|structural_mismatch|target_only|bolt_on}"
  shadcn_equivalent: "{ShadcnComponentName | null}"
  confidence: "{high|medium|low}"
  confidence_notes: "{Why this confidence score. What's known and what's missing.}"
  sources_used: [{storybook}, {tokens-npm}, {components-npm}, {figma}]
  sources_missing: [{figma}, {source-code}]
  replication_status: "not_started"
```

### 5. Prioritise the build order

When scaling, suggest building in this order:
1. `direct_match` components first — fast wins, establishes token coverage
2. `partial_match` next — moderate complexity, high value
3. `target_only` — higher effort, build the most-needed ones first
4. `bolt_on` — integrate one library at a time
5. `structural_mismatch` — most complex, usually lower priority for prototyping

Within each group, prioritise by prototype value (how often this component appears in typical target system screens).
