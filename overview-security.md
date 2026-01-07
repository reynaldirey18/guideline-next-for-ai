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
  const isProtectedRoute = protectedRoutes.some((route) => path.startsWith(route));
  const isPublicRoute = publicRoutes.includes(path);

  // 3. Decrypt the session from the cookie (Mock example)
  const token = request.cookies.get("access_token")?.value;
  
  // 4. Redirect to /login if the user is not authenticated on a protected route
  if (isProtectedRoute && !token) {
    return NextResponse.redirect(new URL("/login", request.nextUrl));
  }

  // 5. Redirect to /dashboard if the user is authenticated and on a public route (like login)
  if (isPublicRoute && token && !request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/dashboard", request.nextUrl));
  }

  return NextResponse.next();
}

export const config = {
  // Matcher ignoring internal Next.js files and static assets
  matcher: ["/((?!api|_next/static|_next/image|.*\\.png$).*)"],
};
```
