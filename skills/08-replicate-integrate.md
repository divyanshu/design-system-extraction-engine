# Skill: replicate-integrate

Integrate and theme a bolt-on library for `bolt_on` components.

## When to use

Manifest entry has `mapping_type: bolt_on`. The capability requires a supplementary library (charts, data tables, drag-and-drop, etc.).

## Recommended libraries

| Capability | Library | Notes |
|---|---|---|
| Charts | Recharts | Common in shadcn ecosystem, Tailwind-friendly |
| Data tables (simple) | Native `<table>` with columns API | No library needed for basic column rendering |
| Data tables (advanced) | TanStack Table | Add only if sorting, filtering, or virtualization is required |
| Date picking (simple) | Native `<input type="date">` | Matches many design systems' stable specs without extra dependencies |
| Date picking (custom) | react-day-picker | Use when target shows a custom calendar popup |
| Drag and drop | dnd-kit (`@dnd-kit/core`, `@dnd-kit/sortable`, `@dnd-kit/utilities`) | Headless, accessible |
| Rich text | Tiptap | Headless, extensible |

**Prefer the simplest option that achieves 80-90% fidelity.** Native elements and thin wrappers over heavy libraries.

## Steps

### 1. Check if the library is already installed

Look in `poc/package.json`. If not present, add it:
```bash
cd poc && npm install {library}
```

### 2. Identify target-specific theming

For charts: Check `targets/{target}/tokens/theme.css` for `--chart-*` tokens. These are already mapped to CSS custom properties.

For other libraries: Look for token names matching the component (e.g., `--{target}-table-*`, `--{target}-dnd-*`).

### 3. Write a thin themed wrapper

Do not reimagine the library's API. Create a wrapper that:
1. Passes all props through to the underlying library
2. Applies target tokens via `style` props or className for theming surface points
3. Provides a simplified API for the most common target system use cases

```tsx
"use client"

import { LibraryComponent, ... } from "recharts"

// Map target chart color tokens to an array
const chartColors = [
  "var(--chart-1)",
  "var(--chart-2)",
  "var(--chart-3)",
  "var(--chart-4)",
  "var(--chart-5)",
]

export interface {Target}{Component}Props {
  // Only expose what the target system documents — keep it simple
  data: Record<string, unknown>[]
  lines: { dataKey: string; name?: string; color?: string }[]
  xAxisKey: string
  height?: number
  showGrid?: boolean
  showTooltip?: boolean
}

function {Target}{Component}({ data, lines, xAxisKey, height = 300, showGrid = true, showTooltip = true }) {
  return (
    <div data-slot="chart" style={{ width: "100%", height }}>
      <ResponsiveContainer width="100%" height="100%">
        <LibraryComponent data={data} margin={{ top: 8, right: 16, left: 0, bottom: 0 }}>
          {showGrid && <Grid stroke="var(--border)" opacity={0.5} />}
          <XAxis tick={{ fill: "var(--muted-foreground)" }} tickLine={false} axisLine={{ stroke: "var(--border)" }} />
          <YAxis tick={{ fill: "var(--muted-foreground)" }} tickLine={false} axisLine={{ stroke: "var(--border)" }} />
          {showTooltip && (
            <Tooltip contentStyle={{
              backgroundColor: "var(--popover)",
              border: "1px solid var(--border)",
              borderRadius: "var(--radius)",
              color: "var(--popover-foreground)",
              fontSize: "0.875rem",
            }} />
          )}
          {lines.map((line, index) => (
            <Line key={line.dataKey} stroke={line.color ?? chartColors[index % chartColors.length]} strokeWidth={2} />
          ))}
        </LibraryComponent>
      </ResponsiveContainer>
    </div>
  )
}
```

### 4. Theming approach

**Rule:** All theming through CSS custom properties only. Never use the library's own theme API (e.g., Recharts' `fill` hardcoded values, TanStack's className system). This keeps all tokens in one place and makes dark mode automatic.

**Tooltip/popup elements** that render outside the React tree need inline `style` props (not className) because they can't inherit CSS vars from a parent that doesn't exist in the DOM at that point.

### 5. File placement

- POC first: `poc/src/components/{target}/{component}.tsx`
- Registry after sign-off: `targets/{target}/registry/{target}/integrations/{component}.tsx`

## Dark mode

bolt_on components need dark token coverage just like any other component:
1. Identify every CSS custom property passed as `style` or `stroke`/`fill` prop to the library
2. Confirm dark overrides exist in `globals.css` for each `--{target}-*` token and each shadcn semantic token used (e.g. `--border`, `--muted-foreground`, `--popover`)
3. Library tooltip/popup elements that receive inline `style` props will automatically pick up dark values because they read from CSS custom properties — no extra work needed

## dnd-kit notes

```tsx
import { DndContext, closestCenter, PointerSensor, useSensor, useSensors } from "@dnd-kit/core"
import { SortableContext, verticalListSortingStrategy, useSortable, arrayMove } from "@dnd-kit/sortable"
import { CSS } from "@dnd-kit/utilities"

// Sensor setup — PointerSensor works for mouse + touch
const sensors = useSensors(useSensor(PointerSensor))

// Each item uses useSortable
const { attributes, listeners, setNodeRef, transform, transition, isDragging } = useSortable({ id: item.id })
const style = { transform: CSS.Transform.toString(transform), transition }

// Handle reorder
function handleDragEnd(event: DragEndEvent) {
  const { active, over } = event
  if (over && active.id !== over.id) {
    setItems(items => arrayMove(items, items.findIndex(i => i.id === active.id), items.findIndex(i => i.id === over.id)))
  }
}
```

Items must have stable string or number `id` fields. Use `isDragging` to apply opacity/elevation feedback during drag.

## Recharts theming notes

- Grid lines: `stroke="var(--border)" opacity={0.5}`
- Axis labels: `tick={{ fill: "var(--muted-foreground)" }}`
- Tooltip background: inline `style` with `var(--popover)`, `var(--border)`, `var(--radius)`
- Line colors: `var(--chart-1)` through `var(--chart-5)` (map these in theme.css)
- Active dot: `activeDot={{ r: 4, strokeWidth: 2 }}` — no custom color needed, inherits stroke
