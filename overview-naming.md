# Konvensi Penamaan

## 📝 Konvensi Penamaan

### File Naming

- **Components**: PascalCase (e.g., `Card.tsx`, `ListItem.tsx`)
- **Hooks**: camelCase dengan prefix `use` (e.g., `useAuthStore.ts`, `useClientPagination.ts`)
- **Utils**: camelCase (e.g., `formatUtils.ts`, `helper.ts`)
- **Types**: camelCase (e.g., `course.ts`, `api/login.ts`)
- **Constants**: camelCase atau UPPER_SNAKE_CASE (e.g., `api.ts`, `menuData.ts`)
- **API files**: camelCase dengan suffix `Api` (e.g., `courseApi.ts`, `authApi.ts`)

### Component Naming

- Default export untuk main component
- Named export untuk sub-components atau types
- Interface/Type: PascalCase dengan prefix `I` atau tanpa prefix

### Folder Naming

- **Route groups**: `(groupName)` - menggunakan parentheses
- **Dynamic routes**: `[param]` - menggunakan brackets
- **Private folders**: `_folderName` - menggunakan underscore prefix
- **Feature folders**: kebab-case atau camelCase

## 📚 Path Aliases

```typescript
// tsconfig.json paths
"@/*": ["./*"]

// Usage
import Component from "@/components/atoms/Component";
import { util } from "@/lib/utils/util";
import { useHook } from "@/hooks/useHook";
import type { Type } from "@/types/type";
```
