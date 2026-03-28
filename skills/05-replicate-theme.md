# Skill: replicate-theme

Build a `direct_match` component — apply token overrides and extend variants on an existing shadcn component.

## When to use

Manifest entry has `mapping_type: direct_match`. The shadcn equivalent is structurally close enough that the main work is styling and variant extension.

## Steps

### 1. Review the anatomy spec

Read the extracted anatomy for this component. Note:
- Which props drive variants
- What the default values are
- What slots exist

### 2. Check the token file for relevant tokens

Scan `targets/{target}/tokens/theme.css` for tokens matching this component.
Token naming pattern: `--{target}-{component}-{property}-{variant}-{state}`

Examples:
- `--{target}-button-primary-bg`
- `--{target}-button-primary-bg-hover`
- `--{target}-button-disabled-text`

If component tokens are missing, they may use semantic shadcn tokens (`--primary`, `--foreground`) which are already mapped.

### 3. Write the component using cva

Follow this pattern:

```tsx
"use client"  // only if needed

import { ShadcnPrimitive } from "@base-ui/react/..."
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

const {target}{Component}Variants = cva(
  // Base classes shared across all variants
  "...",
  {
    variants: {
      // Each visual prop becomes a variant key
      {propName}: {
        {value}: "...", // Tailwind classes using var(--{target}-...) tokens
      }
    },
    defaultVariants: {
      {propName}: "{defaultValue}",
    },
  }
)

export interface {Target}{Component}Props
  extends ShadcnPrimitive.Props,
    VariantProps<typeof {target}{Component}Variants> {}

function {Target}{Component}({ className, {variantProps}, ...props }: {Target}{Component}Props) {
  return (
    <ShadcnPrimitive
      data-slot="{component}"
      className={cn({target}{Component}Variants({ {variantProps} }), className)}
      {...props}
    />
  )
}

export { {Target}{Component}, {target}{Component}Variants }
```

### 4. Handle disabled state

If the component has a disabled state with distinct tokens, apply them inline via `cn()` conditional:

```tsx
className={cn(
  {target}ButtonVariants({ variant, size }),
  disabled && "bg-[var(--{target}-button-disabled-bg)] text-[var(--{target}-button-disabled-text)]",
  className
)}
```

### 5. File placement

- POC: `poc/src/components/{target}/{component}.tsx`
- Registry: `targets/{target}/registry/{target}/ui/{component}.tsx`

Always build in POC first for visual verification, then copy to registry after sign-off.

### 6. Add to POC showcase

Add the component to the showcase page with examples of each key variant.

## Common patterns

- Base UI primitives (`@base-ui/react`) can be used instead of Radix — same API surface, different import
- `data-slot` attributes on all elements enable composability and CSS targeting
- `focus-visible:ring-2 focus-visible:ring-[var(--ring)]` for focus styling
- `[&_svg]:size-4 [&_svg]:pointer-events-none` for icon sizing in base classes
- Disabled state applied via `disabled` prop + `cn()` conditional (not as a cva variant) when the disabled token is shared across all variants

## Base UI data attributes

`@base-ui/react` uses **data attributes** differently from Radix:
- Radix: `data-state="checked"` / `data-state="unchecked"`
- Base UI: `data-checked` / `data-unchecked` (presence-based, no value)

Always use the Base UI form when writing Tailwind selectors:
```tsx
// Correct (Base UI)
"data-checked:bg-[var(--{target}-switch-on-bg)]"

// Wrong (Radix pattern — will not work with @base-ui/react)
"data-[state=checked]:bg-[var(--{target}-switch-on-bg)]"
```

## Border radius

Do not assume all components use the global `--radius` token. Check for component-specific radius tokens in the anatomy spec and `theme.css`. For example, form inputs may use a smaller radius than cards or dialogs.

## Dark mode

Every new component needs dark token coverage. Checklist:
1. Scan `poc/src/app/globals.css` for the component's token group — confirm a `.dark { }` block exists for each `--{target}-*` token used
2. If dark overrides are missing, add them to the `.dark { }` block in `globals.css` and mirror to `targets/{target}/tokens/theme.css`
3. Test by toggling the dark mode button in the showcase
4. Intentionally-same tokens (e.g. shadows using `rgba`, always-dark surfaces like toast notifications) can be omitted from the dark block — add a comment noting the intent
