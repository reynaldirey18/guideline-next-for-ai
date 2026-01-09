# Security & Environment

## 🔐 Security & Environment

### Environment Variables

- Prefix dengan `NEXT_PUBLIC_` untuk client-side variables
- Never commit `.env` files
- Use `.env.example` untuk documentation

### API Security

- Tokens stored securely (cookies/httpOnly)
- API keys never exposed in client code
- Validate all user inputs

### Middleware Pattern

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// 1. Specify protected and public routes
const protectedRoutes = ["/dashboard", "/profile"];
const publicRoutes = ["/login", "/register", "/"];

export function middleware(request: NextRequest) {
  // 2. Check if the current route is protected or public
  const path = request.nextUrl.pathname;
  const isProtectedRoute = protectedRoutes.some((route) =>
    path.startsWith(route)
  );
  const isPublicRoute = publicRoutes.includes(path);

  // 3. Decrypt the session from the cookie (Mock example)
  const token = request.cookies.get("access_token")?.value;

  // 4. Redirect to /login if the user is not authenticated on a protected route
  if (isProtectedRoute && !token) {
    return NextResponse.redirect(new URL("/login", request.nextUrl));
  }

  // 5. Redirect to /dashboard if the user is authenticated and on a public route (like login)
  if (
    isPublicRoute &&
    token &&
    !request.nextUrl.pathname.startsWith("/dashboard")
  ) {
    return NextResponse.redirect(new URL("/dashboard", request.nextUrl));
  }

  return NextResponse.next();
}

export const config = {
  // Matcher ignoring internal Next.js files and static assets
  matcher: ["/((?!api|_next/static|_next/image|.*\\.png$).*)"],
};
```

### 2. Content Security Policy (CSP)

**CSP Headers di `next.config.ts`**

```typescript
// next.config.ts
const securityHeaders = [
  {
    key: "Content-Security-Policy",
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-eval' 'unsafe-inline'",
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https:",
      "font-src 'self' data:",
      "connect-src 'self' https://api.example.com",
      "frame-ancestors 'none'",
    ].join("; "),
  },
  {
    key: "X-Frame-Options",
    value: "DENY",
  },
  {
    key: "X-Content-Type-Options",
    value: "nosniff",
  },
  {
    key: "X-XSS-Protection",
    value: "1; mode=block",
  },
  {
    key: "Referrer-Policy",
    value: "strict-origin-when-cross-origin",
  },
];
```

### 3. XSS Prevention

**Input Sanitization**

```typescript
// lib/sanitize.ts
import DOMPurify from "isomorphic-dompurify";

export function sanitizeHtml(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ["b", "i", "em", "strong", "a", "p", "br"],
    ALLOWED_ATTR: ["href"],
  });
}
```

### 4. CSRF Protection

**CSRF Token Generation**

```typescript
// lib/csrf.ts
import { randomBytes } from "crypto";

export function generateCSRFToken(): string {
  return randomBytes(32).toString("hex");
}

// Verify CSRF di Server Actions
export async function createPost(formData: FormData) {
  const cookieStore = await cookies();
  const sessionToken = cookieStore.get("csrf-token")?.value;
  const requestToken = formData.get("csrf-token") as string;

  if (!sessionToken || requestToken !== sessionToken) {
    return { success: false, message: "Invalid CSRF token" };
  }
  // Process form...
}
```

### 5. Input Validation

**Always Validate User Input**

```typescript
// Use Zod untuk validation
import { z } from "zod";

const userInputSchema = z.object({
  name: z.string().min(3).max(100),
  email: z.string().email(),
  age: z.number().int().min(18).max(120),
});

// In Server Actions
const result = userInputSchema.safeParse(rawData);
if (!result.success) {
  return { success: false, errors: result.error.errors };
}
```

### 6. SQL Injection Prevention

**Use Parameterized Queries atau ORM**

```typescript
// ✅ Good - Parameterized query
const user = await db.query("SELECT * FROM users WHERE id = $1", [userId]);

// ✅ Good - ORM (Prisma automatically handles parameterization)
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### 7. Rate Limiting

See `overview-route-handlers.md` for rate limiting implementation details.

### 8. Secure File Upload

**File Upload Validation**

```typescript
const ALLOWED_TYPES = ["image/jpeg", "image/png", "image/webp"];
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB

export function validateFile(file: File): { valid: boolean; error?: string } {
  if (!ALLOWED_TYPES.includes(file.type)) {
    return { valid: false, error: "Invalid file type" };
  }
  if (file.size > MAX_FILE_SIZE) {
    return { valid: false, error: "File too large (max 5MB)" };
  }
  return { valid: true };
}
```

## Summary Checklist

1. [ ] **Environment Variables**: Validated with Zod schema
2. [ ] **CSP Headers**: Content Security Policy configured
3. [ ] **XSS Prevention**: Input sanitization implemented
4. [ ] **CSRF Protection**: CSRF tokens generated and verified
5. [ ] **Input Validation**: All inputs validated with Zod
6. [ ] **SQL Injection**: Parameterized queries or ORM used
7. [ ] **Rate Limiting**: Rate limiting on API routes
8. [ ] **Security Headers**: X-Frame-Options, X-Content-Type-Options, etc.
9. [ ] **File Upload**: Secure file upload with validation
10. [ ] **Token Storage**: httpOnly cookies untuk authentication tokens
