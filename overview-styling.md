# Styling Standards

## 🎨 Styling Standards

### Tailwind CSS

- Menggunakan utility classes dari Tailwind
- Prefer Tailwind default color classes (`text-`, `bg-`, `border-`)
- Custom colors hanya jika tidak ada yang cocok di Tailwind
- Menggunakan `cn()` function untuk conditional className

### Color System

```typescript
// Primary colors (CSS variables)
primary-50 hingga primary-900

// Secondary colors (CSS variables)
secondary-50 hingga secondary-900

// Text colors
text-primary: #3E424B
text-secondary: #868D9D
text-dark: #1E1E1E
```

### Conditional Styling

```typescript
import { cn } from "@/lib/utils/common";

// Always use cn() for conditional className
<div
  className={cn("base-classes", condition && "conditional-classes", className)}
/>;
```
