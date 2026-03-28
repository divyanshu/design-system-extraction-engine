# Skill: replicate-extend

Build a `partial_match` component — fork a shadcn component and modify its structure, slots, or API to match the target.

## When to use

Manifest entry has `mapping_type: partial_match`. The concept exists in shadcn but the component boundaries, slot structure, or prop API differ meaningfully.

## Steps

### 1. Identify the structural differences

Compare the target component anatomy against the shadcn equivalent:
- What slots does the target have that shadcn doesn't?
- What slots does shadcn have that the target doesn't use?
- Does the target compose multiple sub-components that shadcn expresses as one?
- Are prop names or values different?

**Example (Dialog):** shadcn Dialog has `DialogContent` with children. The target Dialog may require explicit `Header`, `Body`, and `Footer` sub-components with independent scroll and a distinct footer background.

**Example (Chip):** shadcn Badge is a plain `<span>`. The target Chip may add interactive delete button, click handler, compound variant system (style × color), icon slot.

### 2. Decide the fork strategy

**Option A — Sub-component composition:** Export a family of named sub-components (`{Target}DialogHeader`, `{Target}DialogBody`, etc.). Consumer assembles them. Best for layout-heavy components (dialogs, cards, pages).

**Option B — Prop-driven single component:** Export one component, slots are optional props (`icon`, `actions`, `onDelete`). Best for leaf components (chips, badges, inputs).

### 3. Write the component

Start from the shadcn source as a reference, but write from scratch using Base UI primitives. Do not copy-paste shadcn internals.

```tsx
"use client"

import { ShadcnPrimitive } from "@base-ui/react/..."
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

// For compound variants (e.g. variant × color):
const {target}{Component}Variants = cva("base-classes", {
  variants: {
    variant: { filled: "", outlined: "bg-transparent border" },
    color: { primary: "", info: "", success: "" },
  },
  compoundVariants: [
    { variant: "filled", color: "primary", className: "bg-[var(--{target}-chip-filled-primary-bg)] text-[var(--{target}-chip-filled-primary-text)]" },
    // ... all combinations
  ],
  defaultVariants: { variant: "filled", color: "primary" },
})
```

### 4. Preserve shadcn conventions

- Use `data-slot` attributes on all elements
- Export both the component and its variants: `export { {Target}Chip, {target}ChipVariants }`
- Accept `className` and spread `...props` for extensibility
- Sub-components should accept `className` and `...props` too

### 5. Custom dropdown pattern

When the @base-ui/react equivalent has an uncertain or unstable API, prefer a custom implementation using React state + click-outside detection. This gives full control over the Figma-matched design and avoids API churn.

```tsx
// Custom dropdown: React state + useEffect for click-outside
const [open, setOpen] = React.useState(false)
const containerRef = React.useRef<HTMLDivElement>(null)

React.useEffect(() => {
  const handleClickOutside = (e: MouseEvent) => {
    if (containerRef.current && !containerRef.current.contains(e.target as Node)) {
      setOpen(false)
    }
  }
  document.addEventListener("mousedown", handleClickOutside)
  return () => document.removeEventListener("mousedown", handleClickOutside)
}, [])
```

Use `@base-ui/react` for components with stable, well-documented APIs (Tooltip, Popover, Tabs, RadioGroup, Checkbox, NumberField). Use custom state for Select-family components where you need full control.

### 6. Dark mode

Add a dark mode checklist to every partial_match component:
1. Identify every `--{target}-*` token used in the component
2. Confirm a `.dark { }` override exists in `globals.css` for each token
3. If missing, add the dark value derived from Figma dark theme or the raw token package's dark scale

### 7. File placement

- POC first: `poc/src/components/{target}/{component}.tsx`
- Registry after sign-off: `targets/{target}/registry/{target}/extensions/{component}.tsx`

## Compound variant pattern

When a component has two variant dimensions that each affect styling independently:

```tsx
compoundVariants: [
  { variant: "filled", color: "info",    className: "bg-[var(--{target}-chip-filled-info-bg)] text-[var(--{target}-chip-filled-info-text)]" },
  { variant: "outlined", color: "info",  className: "border-[var(--{target}-chip-outlined-info-border)] text-[var(--{target}-chip-outlined-info-text)]" },
  // n × m entries
]
```

Compound variants are verbose but explicit — every combination is intentional.

## Sub-component composition pattern

```tsx
// Root controls open/close state
function {Target}Dialog({ ...props }: DialogPrimitive.Root.Props) {
  return <DialogPrimitive.Root data-slot="dialog" {...props} />
}

// Content wraps the portal + overlay + popup
function {Target}DialogContent({ width = "medium", showDismissButton = true, children, ...props }) {
  return (
    <DialogPrimitive.Portal>
      <{Target}DialogOverlay />
      <DialogPrimitive.Popup className={cn("...", dialogWidths[width])} {...props}>
        {children}
        {showDismissButton && <CloseButton />}
      </DialogPrimitive.Popup>
    </DialogPrimitive.Portal>
  )
}

// Slots are plain divs with data-slot + styling
function {Target}DialogHeader({ className, ...props }) {
  return <div data-slot="dialog-header" className={cn("border-b p-4", className)} {...props} />
}
```
