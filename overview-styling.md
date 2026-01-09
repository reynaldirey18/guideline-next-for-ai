# Styling Standards

## 🎨 Styling Standards

### 1. Tailwind CSS

**Rules:**
- Menggunakan utility classes dari Tailwind
- Prefer Tailwind default color classes (`text-`, `bg-`, `border-`)
- Custom colors hanya jika tidak ada yang cocok di Tailwind
- Menggunakan `cn()` function untuk conditional className

### 2. Color System

**CSS Variables**

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

**Tailwind Config**

```typescript
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      primary: {
        DEFAULT: "var(--color-primary)",
        50: "var(--color-primary-50)",
        100: "var(--color-primary-100)",
        // ... up to 900
      },
      secondary: {
        DEFAULT: "var(--color-secondary)",
        // ... similar structure
      },
    },
  },
}
```

### 3. Conditional Styling

**Using cn() Function**

```typescript
import { cn } from "@/lib/utils/common";

// Always use cn() for conditional className
<div
  className={cn("base-classes", condition && "conditional-classes", className)}
/>;
```

**cn() Utility**

```typescript
// lib/utils/common.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### 4. Dark Mode

**Dark Mode Setup**

```typescript
// tailwind.config.ts
module.exports = {
  darkMode: "class", // or "media"
  // ...
};
```

**Dark Mode Provider**

```typescript
// provider/ThemeProvider.tsx (see overview-state.md)
"use client";

import { createContext, useContext, useState, useEffect } from "react";

type Theme = "light" | "dark";

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>("light");

  useEffect(() => {
    const savedTheme = localStorage.getItem("theme") as Theme;
    if (savedTheme) {
      setTheme(savedTheme);
    }
  }, []);

  useEffect(() => {
    document.documentElement.classList.toggle("dark", theme === "dark");
    localStorage.setItem("theme", theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme((prev) => (prev === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

**Dark Mode Classes**

```typescript
// Use dark: prefix untuk dark mode styles
<div className="bg-white dark:bg-gray-900 text-gray-900 dark:text-white">
  Content
</div>
```

### 5. Responsive Design

**Breakpoints**

```typescript
// Tailwind default breakpoints
sm: 640px   // @media (min-width: 640px)
md: 768px   // @media (min-width: 768px)
lg: 1024px  // @media (min-width: 1024px)
xl: 1280px  // @media (min-width: 1280px)
2xl: 1536px // @media (min-width: 1536px)
```

**Responsive Patterns**

```typescript
// Mobile-first approach
<div className="
  text-sm sm:text-base lg:text-lg
  p-4 sm:p-6 lg:p-8
  grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3
">
  Content
</div>
```

**Responsive Utilities**

```typescript
// Hide/show on different screens
<div className="hidden sm:block">Visible on sm and up</div>
<div className="block sm:hidden">Visible only on mobile</div>

// Responsive flex direction
<div className="flex flex-col sm:flex-row">
  Content
</div>
```

### 6. Component Styling Patterns

**Button Variants**

```typescript
// components/atoms/Button.tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-primary text-white hover:bg-primary/90",
        destructive: "bg-red-500 text-white hover:bg-red-600",
        outline: "border border-gray-300 bg-white hover:bg-gray-50",
        ghost: "hover:bg-gray-100",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 px-3",
        lg: "h-11 px-8",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      {...props}
    />
  );
}
```

### 7. CSS Modules (If Needed)

**CSS Module Usage**

```typescript
// components/Module.module.css
.container {
  display: flex;
  gap: 1rem;
}

// components/Module.tsx
import styles from "./Module.module.css";

export function Module() {
  return <div className={styles.container}>Content</div>;
}
```

**Note**: Prefer Tailwind utility classes over CSS Modules unless absolutely necessary.

### 8. Custom Utilities

**Custom Tailwind Utilities**

```typescript
// tailwind.config.ts
module.exports = {
  theme: {
    extend: {
      // Custom spacing
      spacing: {
        "18": "4.5rem",
        "88": "22rem",
      },
      // Custom animations
      animation: {
        "fade-in": "fadeIn 0.5s ease-in-out",
      },
      keyframes: {
        fadeIn: {
          "0%": { opacity: "0" },
          "100%": { opacity: "1" },
        },
      },
    },
  },
};
```

### 9. Styling Best Practices

**Do:**
- ✅ Use Tailwind utility classes
- ✅ Use `cn()` untuk conditional classes
- ✅ Use CSS variables untuk theming
- ✅ Mobile-first responsive design
- ✅ Use semantic HTML

**Don't:**
- ❌ Don't use inline styles
- ❌ Don't create custom CSS unless necessary
- ❌ Don't mix CSS Modules dengan Tailwind utilities
- ❌ Don't use !important (except for utilities)

## Summary Checklist

1. [ ] **Tailwind CSS**: Utility classes digunakan consistently
2. [ ] **Color System**: CSS variables untuk theming
3. [ ] **cn() Function**: Conditional styling dengan cn()
4. [ ] **Dark Mode**: Dark mode implemented dengan provider
5. [ ] **Responsive Design**: Mobile-first dengan breakpoints
6. [ ] **Component Variants**: CVA untuk component variants
7. [ ] **Custom Utilities**: Custom utilities jika diperlukan
8. [ ] **Styling Patterns**: Consistent styling patterns
9. [ ] **Best Practices**: Follow Tailwind best practices
10. [ ] **Semantic HTML**: Use semantic HTML elements
