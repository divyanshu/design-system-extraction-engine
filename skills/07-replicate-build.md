# Skill: replicate-build

Build a `target_only` component from scratch using shadcn/Base UI primitives.

## When to use

Manifest entry has `mapping_type: target_only`. The component has no shadcn equivalent — it must be built from Radix/Base UI primitives following shadcn conventions.

## Steps

### 1. Identify the closest primitive(s)

Even if shadcn doesn't have the component, Radix/Base UI likely has a primitive that covers the interaction model:

| Component type | Likely primitive |
|---|---|
| Notification/alert | None needed — plain `<div role="alert">` |
| Tooltip | `@base-ui/react/tooltip` |
| Popover | `@base-ui/react/popover` |
| Dropdown | `@base-ui/react/menu` |
| Tabs | `@base-ui/react/tabs` |
| Accordion | `@base-ui/react/collapsible` |
| Progress | `@base-ui/react/progress` |
| Slider | `@base-ui/react/slider` |

For purely visual/layout components (e.g. Callout, SectionHeader, EmptyState), no primitive is needed.

### 2. Design the component API

Model the prop API to match the target system as closely as practical:
- Use the same prop names where possible (aids developer muscle memory)
- Map target system naming conventions to React conventions (`isDisabled` → `disabled`, `canClose` → `canClose` is fine)
- Prefer union type string enums over boolean flags for state variants

### 3. Write the component

Pattern for visual-only components:

```tsx
import * as React from "react"
import { cva, type VariantProps } from "class-variance-authority"
import { IconName } from "lucide-react"
import { cn } from "@/lib/utils"

// 1. cva for variant-driven classes
const {component}Variants = cva("base-classes", {
  variants: {
    type: {
      information: "bg-[var(--{target}-{c}-info-bg)] border-l-[var(--{target}-{c}-info-border)]",
      // ...
    }
  }
})

// 2. Lookup maps for non-class data (icons, colors per variant)
const iconColorMap = {
  information: "text-[var(--{target}-{c}-info-icon)]",
  // ...
} as const

const defaultIcons = {
  information: InfoIcon,
  // ...
} as const

// 3. Props interface
export interface {Target}{Component}Props
  extends Omit<React.HTMLAttributes<HTMLDivElement>, "conflicting-props">,
    VariantProps<typeof {component}Variants> {
  // Target-system-specific props
  type: NonNullable<VariantProps<typeof {component}Variants>["type"]>
  title?: React.ReactNode
  canClose?: boolean
  onClose?: () => void
  actions?: React.ReactNode
}

// 4. Component implementation
function {Target}{Component}({ type, title, canClose, onClose, actions, children, className, ...props }) {
  const [visible, setVisible] = React.useState(true)
  const IconComponent = defaultIcons[type]
  if (!visible) return null

  return (
    <div data-slot="{component}" role="alert" className={cn({component}Variants({ type }), className)} {...props}>
      <IconComponent className={cn("size-5 shrink-0", iconColorMap[type])} aria-hidden />
      <div className="flex-1 space-y-1">
        {title && <div data-slot="{component}-title" className={cn("font-semibold", titleColorMap[type])}>{title}</div>}
        {children && <div data-slot="{component}-body">{children}</div>}
        {actions && <div data-slot="{component}-actions" className="flex gap-2 pt-2">{actions}</div>}
      </div>
      {canClose && <CloseButton onClick={() => { setVisible(false); onClose?.() }} />}
    </div>
  )
}
```

### 4. Key patterns

**Auto-icons:** Use a `const defaultIcons = { ... } as const` lookup map. Keeps the cva clean and allows icon overrides if needed later.

**Dismissible state:** Local `useState(true)` — simple and self-contained. For controlled dismiss, accept `open` + `onOpenChange` props.

**Close button:** Plain `<button>` with `aria-label="Dismiss"` and SVG X icon, or import `XIcon` from lucide-react.

**data-slot attributes:** Always add `data-slot="{component}"` on the root and `data-slot="{component}-{part}"` on named parts. This enables CSS targeting and test selection.

### 5. Export helper functions when needed

Some compound components require helper utilities that consumers call directly. Export these alongside the component:

```tsx
// Example: ProgressTracker needs a state helper
export function getProgressTrackerItemState(
  currentStep: number,
  itemIndex: number
): "completed" | "current" | "upcoming" {
  if (itemIndex < currentStep) return "completed"
  if (itemIndex === currentStep) return "current"
  return "upcoming"
}
```

Export helpers from the same file as the component — keeps the API surface co-located.

### 6. Compound component pattern

When the target component is a family of related sub-components, export them as a compound set:

```tsx
// Root container
function {Target}List({ className, ...props }: React.HTMLAttributes<HTMLUListElement>) {
  return <ul data-slot="list" className={cn("...", className)} {...props} />
}

// Item (accepts sub-components as children)
function {Target}ListItem({ className, ...props }: React.HTMLAttributes<HTMLLIElement>) {
  return <li data-slot="list-item" className={cn("...", className)} {...props} />
}

// Named content slots
function {Target}ListTitle({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div data-slot="list-title" className={cn("...", className)} {...props} />
}

// Export the whole family
export { {Target}List, {Target}ListItem, {Target}ListTitle, {Target}ListDescription, {Target}ListItemContent, {Target}ListItemLink }
```

### 7. createPortal for floating components

Components that render outside the normal DOM tree (e.g. toast notifications) must use `createPortal` to avoid z-index and overflow clipping issues:

```tsx
import { createPortal } from "react-dom"

function {Target}Snackbar({ show, children, ... }) {
  if (!show) return null
  return createPortal(
    <div data-slot="snackbar" className="fixed bottom-6 left-1/2 -translate-x-1/2 z-50 ...">
      {children}
    </div>,
    document.body
  )
}
```

Use `createPortal` any time the component needs to escape a `overflow: hidden` or stacking context ancestor.

### 8. File placement

- POC first: `poc/src/components/{target}/{component}.tsx`
- Registry after sign-off: `targets/{target}/registry/{target}/custom/{component}.tsx`

## Dark mode

Every target_only component needs dark token coverage:
1. List every `--{target}-*` token the component uses
2. Confirm `.dark { }` overrides exist in `globals.css` for each
3. Purely structural tokens (layout spacing, border radius) typically don't need dark variants — only color tokens do
4. Test by toggling the dark mode button in the showcase
