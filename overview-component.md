# Arsitektur Komponen (Atomic Design)

## Atomic Design Pattern

### Atoms

Komponen terkecil dan tidak dapat dibagi lagi:

- `Typography` - Text components
- `Button` - Button component
- `Input` - Form input components
- `Icon` - Icon components
- `Badge` - Badge component

**Lokasi**: `components/atoms/`

### Molecules

Komponen yang terdiri dari beberapa atoms:

- `Card` - Card component dengan header, content, dan footer
- `FormField` - Form input dengan label, input, dan error message
- `SearchInput` - Input field dengan search icon dan clear button
- `DataTable` - Table component dengan pagination dan filtering
- `ListItem` - List item dengan icon, text, dan action button

**Lokasi**: `components/molecules/`

### Organisms

Komponen kompleks yang terdiri dari molecules dan atoms:

- `Card` - Complex card dengan multiple sections dan interactive elements
- `DashboardLayout` - Layout dengan sidebar, header, dan main content area
- `MediaPlayer` - Video/audio player dengan controls dan metadata
- `DataGrid` - Advanced table dengan sorting, filtering, dan bulk actions

**Lokasi**: `components/organisms/`

### Pages/Templates

Halaman lengkap yang menggunakan organisms:

- `app/[locale]/(dashboardLayout)/[page]/page.tsx`

## 🎯 Component Examples

### Atom Component

```typescript
// components/atoms/Button.tsx
import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button"
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    )
  }
)
Button.displayName = "Button"

export { Button, buttonVariants }
```

### Molecule Component

```typescript
// components/molecules/Card.tsx
import * as React from "react"
import { cn } from "@/lib/utils"

const Card = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(({ className, ...props }, ref) => (
  <div
    ref={ref}
    className={cn("rounded-xl border bg-card text-card-foreground shadow", className)}
    {...props}
  />
))
Card.displayName = "Card"

export { Card }
```
