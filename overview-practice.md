# Coding Standards & Best Practices

## 🔧 Coding Standards

### Component Structure

```typescript
"use client"; // If needed for client component

import React from "react";
import Typography from "@/components/atoms/Typography";
import Show from "@/components/atoms/Show";
import { cn } from "@/lib/utils/common";

interface ComponentProps {
  // Props definition
}

export default function Component({ props }: ComponentProps) {
  // Component logic

  return (
    <div className={cn("base-classes", className)}>
      <Show when={condition}>{/* Conditional rendering */}</Show>
    </div>
  );
}
```

### Conditional Rendering

- **Always use `<Show>` component** untuk conditional rendering (kecuali text ternary)
- **Text ternary**: Boleh menggunakan ternary operator langsung

```typescript
// ✅ Correct - Use Show component
<Show when={isVisible}>
  <Component />
</Show>

// ✅ Correct - Text ternary
<Typography>{isActive ? "Active" : "Inactive"}</Typography>

// ❌ Avoid - Component ternary
{isVisible ? <Component /> : null}
```

### TypeScript

- Strict mode enabled
- Interface untuk props dan data structures
- Type untuk union types dan primitives
- Export types/interfaces yang digunakan di luar file

### Imports

- Absolute imports menggunakan `@/` alias
- Group imports: React → External libraries → Internal components → Utils → Types
- Use named imports untuk utilities, default imports untuk components

## 📋 Best Practices

1. **Component Composition**: Gunakan composition over inheritance
2. **Type Safety**: Selalu define types untuk props dan API responses
3. **Error Handling**: Handle errors dengan proper error boundaries
4. **Loading States**: Selalu provide loading states untuk async operations
5. **Accessibility**: Gunakan semantic HTML dan ARIA attributes
6. **Performance**:
   - Use React.memo untuk expensive components
   - Lazy load heavy components
   - Optimize images dengan Next.js Image component
7. **Code Organization**:
   - Keep components small and focused
   - Extract reusable logic ke custom hooks
   - Separate concerns (UI, logic, data)
