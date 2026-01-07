# Struktur Folder & Arsitektur

## 📁 Struktur Folder

```
project-root/
├── api/                    # API service layer
│   ├── [feature]/          # Feature-based API modules
│   └── [feature]Api.ts     # API functions per feature
├── app/                    # Next.js App Router
│   └── [locale]/           # Internationalization routes
│       ├── (auth)/         # Route groups for auth pages
│       ├── (dashboardLayout)/  # Route groups with layout
│       ├── (withoutLayout)/    # Route groups without layout
│       └── [dynamic]/      # Dynamic routes
│           └── _components/    # Page-specific components
├── components/             # Reusable components (Atomic Design)
│   ├── atoms/              # Basic building blocks
│   ├── molecules/          # Composite components
│   ├── organisms/          # Complex components
│   ├── ui/                 # shadcn/ui components
│   └── [feature]/          # Feature-specific components
├── constants/              # Application constants
│   ├── api.ts             # API endpoints
│   ├── enum/              # Enum definitions
│   └── [feature].ts       # Feature constants
├── hooks/                  # Custom React hooks
├── https/                  # HTTP client configuration
├── lib/                    # Utility libraries
│   ├── schemas/           # Validation schemas (Yup/Zod)
│   └── utils/             # Utility functions
├── locales/                # i18n translation files
├── provider/               # React context providers
├── public/                 # Static assets
├── rematch/                # State management (Rematch)
├── store/                  # Zustand stores
├── styles/                 # Global styles
└── types/                  # TypeScript type definitions
    └── api/                # API response types
```
