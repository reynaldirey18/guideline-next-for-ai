# Routing Patterns (App Router)

## 🚏 App Router & Navigation

### 1. File-System Routing Fundamentals

Next.js 16 menggunakan App Router berbasis file-system.

- **Page**: `page.tsx` - UI unik untuk route tersebut.
- **Layout**: `layout.tsx` - UI yang dibagi antar route (tidak re-render saat navigasi).
- **Template**: `template.tsx` - Mirip layout tapi re-render saat navigasi (state reset).
- **Loading**: `loading.tsx` - Loading UI (Suspense fallback) otomatis.
- **Error**: `error.tsx` - Error Boundary untuk handling runtime error.
- **Not Found**: `not-found.tsx` - UI untuk 404.

### 2. Route Groups

Gunakan tanda kurung `(folderName)` untuk mengelompokkan route tanpa mempengaruhi struktur URL.

```
app/
├── (auth)/             # Group "auth" (tidak ada di URL)
│   ├── login/          # /login
│   └── register/       # /register
├── (dashboard)/        # Group "dashboard"
│   ├── layout.tsx      # Sidebar layout hanya untuk dashboard
│   └── dashboard/      # /dashboard
```

### 3. Dynamic Routes

Gunakan kurung siku `[param]` untuk dynamic segments.

```typescript
// app/users/[id]/page.tsx
export default function UserDetail({ params }: { params: { id: string } }) {
  return <div>User ID: {params.id}</div>;
}
```

### 4. Parallel Routes

Gunakan slot `@slotName` untuk merender multiple page secara conditional atau simultan dalam satu layout.

```
app/
├── layout.tsx
├── @analytics/page.tsx
└── @team/page.tsx
```

```typescript
// app/layout.tsx
export default function Layout({
  children,
  analytics,
  team
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <>
      {children}
      {analytics}
      {team}
    </>
  )
}
```

### 5. Intercepting Routes

Gunakan `(..)` untuk menangkap route dan menampilkan konten dalam context route saat ini (biasanya untuk modal).

```
app/
├── feed/
│   └── (..)photo/[id]/  # Intercept /photo/[id] saat diakses dari feed
│       └── page.tsx     # Render sebagai Modal
└── photo/
    └── [id]/
        └── page.tsx     # Full page saat direfresh/akses langsung
```

### 6. Navigation & SearchParams

**PENTING**: Di Server Components, `searchParams` adalah props. Di Client Components, gunakan `useSearchParams`.

**Server Component:**

```typescript
// app/users/page.tsx
export default function UsersPage({
  searchParams,
}: {
  searchParams: { page?: string; q?: string };
}) {
  const page = searchParams.page || "1";
  return <UserList page={page} />;
}
```

**Client Component (Navigation):**

```typescript
"use client";
import { useRouter, usePathname, useSearchParams } from "next/navigation";

export default function Search() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();

  const handleSearch = (term: string) => {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set("q", term);
    } else {
      params.delete("q");
    }
    router.replace(`${pathname}?${params.toString()}`);
  };

  return <input onChange={(e) => handleSearch(e.target.value)} />;
}
```
