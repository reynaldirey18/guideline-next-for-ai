# Configuration Files Standards

## ⚙️ Configuration Files

### 1. Next.js Configuration (`next.config.ts`)

**Standard Next.js 16 Configuration**

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  reactStrictMode: true,

  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "example.com",
      },
    ],
  },
};

export default nextConfig;
```

**Extended Configuration (jika diperlukan)**

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  reactStrictMode: true,

  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "example.com",
      },
    ],
  },

  // Security headers (optional)
  async headers() {
    return [
      {
        source: "/:path*",
        headers: [
          {
            key: "X-Frame-Options",
            value: "DENY",
          },
          {
            key: "X-Content-Type-Options",
            value: "nosniff",
          },
        ],
      },
    ];
  },
};

export default nextConfig;
```

### 2. TypeScript Configuration (`tsconfig.json`)

**Recommended TypeScript Configuration**

```json
{
  "compilerOptions": {
    // Language and Environment
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",

    // Modules
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowImportingTsExtensions": true,

    // Interop Constraints
    "isolatedModules": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "allowSyntheticDefaultImports": true,

    // Type Checking
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,

    // Emit
    "noEmit": true,
    "incremental": true,

    // Path Aliases
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    },

    // Next.js specific
    "plugins": [
      {
        "name": "next"
      }
    ]
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 3. ESLint Configuration (`.eslintrc.json`)

**Recommended ESLint Configuration**

```json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module",
    "ecmaFeatures": {
      "jsx": true
    }
  },
  "plugins": ["@typescript-eslint", "react", "react-hooks"],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "react/prop-types": "off",
    "@typescript-eslint/no-unused-vars": [
      "error",
      {
        "argsIgnorePattern": "^_",
        "varsIgnorePattern": "^_"
      }
    ],
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/no-explicit-any": "warn",
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

### 4. Prettier Configuration (`.prettierrc`)

**Recommended Prettier Configuration**

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always",
  "endOfLine": "lf",
  "bracketSpacing": true,
  "jsxSingleQuote": false,
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

**Prettier Ignore (`.prettierignore`)**

```
.next
out
node_modules
build
dist
*.min.js
*.min.css
package-lock.json
yarn.lock
pnpm-lock.yaml
```

### 5. Environment Variables (`.env.example`)

**Environment Variables Template**

```bash
# Public Environment Variables (exposed to browser)
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_ANALYTICS_ID=your-analytics-id

# Private Environment Variables (server-only)
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
JWT_SECRET=your-jwt-secret-key
JWT_REFRESH_SECRET=your-refresh-secret-key
ENCRYPTION_KEY=your-encryption-key

# Third-party Services
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your-email@example.com
SMTP_PASSWORD=your-password

# OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# File Storage
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_S3_BUCKET=your-bucket-name
AWS_REGION=us-east-1

# Feature Flags
ENABLE_FEATURE_X=true
ENABLE_FEATURE_Y=false
```

### 6. Git Configuration (`.gitignore`)

**Recommended `.gitignore`**

```gitignore
# Dependencies
node_modules
.pnp
.pnp.js

# Testing
coverage
.nyc_output

# Next.js
.next
out
build
dist

# Production
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

# Environment variables
.env
.env*.local
.env.production
.env.development

# Vercel
.vercel

# TypeScript
*.tsbuildinfo
next-env.d.ts

# IDEs
.vscode
.idea
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Misc
*.pem
.cache
.parcel-cache
```

### 7. Package.json Scripts

**Recommended Scripts**

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "typecheck": "tsc --noEmit",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "analyze": "ANALYZE=true next build"
  }
}
```

### 8. Path Aliases Configuration

**Ensure `tsconfig.json` has:**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

**Usage in imports:**

```typescript
// ✅ Good - Use path aliases
import { Button } from "@/components/atoms/Button";
import { useAuth } from "@/hooks/useAuth";
import { formatDate } from "@/lib/utils/date";

// ❌ Avoid - Relative paths
import { Button } from "../../../components/atoms/Button";
```

### 9. Tailwind Configuration (`tailwind.config.ts`)

**Recommended Tailwind Config**

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
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
      fontFamily: {
        sans: ["var(--font-inter)", "sans-serif"],
        mono: ["var(--font-roboto-mono)", "monospace"],
      },
    },
  },
  plugins: [],
};

export default config;
```

### 10. Husky Pre-commit Hooks (`.husky/pre-commit`)

**Pre-commit Hook**

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged
```

**Lint-staged Configuration (`.lintstagedrc.json`)**

```json
{
  "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
  "*.{json,md,mdx,css,html,yml,yaml,scss}": ["prettier --write"]
}
```

## Summary Checklist

1. [ ] **next.config.ts**: Configured dengan images, headers, redirects
2. [ ] **tsconfig.json**: Strict mode enabled dengan proper path aliases
3. [ ] **ESLint**: Configured dengan Next.js dan TypeScript rules
4. [ ] **Prettier**: Configured dengan Tailwind plugin
5. [ ] **Environment Variables**: `.env.example` documented
6. [ ] **.gitignore**: All necessary files excluded
7. [ ] **package.json**: All necessary scripts defined
8. [ ] **Path Aliases**: `@/*` configured and used consistently
9. [ ] **Tailwind Config**: Extended dengan custom colors dan fonts
10. [ ] **Pre-commit Hooks**: Husky + lint-staged configured
