# Authentication & Authorization Standards

## 🔐 Authentication & Authorization Patterns

### 1. Authentication Flow

**Login Flow dengan Server Actions**

```typescript
// actions/authActions.ts
"use server";

import { cookies } from "next/headers";
import { redirect } from "next/navigation";
import { CoreHttpResponse } from "@/types/api/common";
import axios from "axios";

export type AuthActionState = {
  success: boolean;
  message?: string;
  errors?: Record<string, string[]>;
};

export async function login(
  prevState: AuthActionState,
  formData: FormData
): Promise<AuthActionState> {
  try {
    const email = formData.get("email") as string;
    const password = formData.get("password") as string;

    // API call untuk login
    const res: CoreHttpResponse<{
      access_token: string;
      refresh_token: string;
      user: { id: string; email: string; role: string };
    }> = await axios.post("/api/auth/login", {
      email,
      password,
    });

    const { access_token, refresh_token, user } = res.data;

    // Set cookies dengan httpOnly flag untuk security
    const cookieStore = await cookies();
    cookieStore.set("access_token", access_token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24, // 24 hours
      path: "/",
    });

    cookieStore.set("refresh_token", refresh_token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24 * 7, // 7 days
      path: "/",
    });

    // Store user data in cookie (non-sensitive)
    cookieStore.set("user", JSON.stringify(user), {
      httpOnly: false, // Allow client-side access
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24,
      path: "/",
    });

    return { success: true };
  } catch (error: any) {
    return {
      success: false,
      message: error.response?.data?.message || "Login failed",
      errors: error.response?.data?.errors,
    };
  }
}

export async function logout() {
  const cookieStore = await cookies();
  cookieStore.delete("access_token");
  cookieStore.delete("refresh_token");
  cookieStore.delete("user");
  redirect("/login");
}
```

**Login Form Component**

```typescript
"use client";

import { useActionState } from "react";
import { login, AuthActionState } from "@/actions/authActions";
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import * as yup from "yup";
import { useRouter } from "next/navigation";
import { useEffect } from "react";
import { toast } from "sonner";

const loginSchema = yup.object({
  email: yup.string().email().required("Email wajib diisi"),
  password: yup.string().required("Password wajib diisi"),
});

type LoginFormData = yup.InferType<typeof loginSchema>;

export default function LoginForm() {
  const router = useRouter();
  const [state, formAction, isPending] = useActionState<
    AuthActionState,
    FormData
  >(login, { success: false });

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginFormData>({
    resolver: yupResolver(loginSchema),
  });

  useEffect(() => {
    if (state.success) {
      toast.success("Login berhasil");
      router.push("/dashboard");
    } else if (state.message) {
      toast.error(state.message);
    }
  }, [state, router]);

  const onSubmit = (data: LoginFormData) => {
    const formData = new FormData();
    formData.append("email", data.email);
    formData.append("password", data.password);
    formAction(formData);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} type="email" placeholder="Email" />
      {errors.email && <p className="text-red-500">{errors.email.message}</p>}

      <input {...register("password")} type="password" placeholder="Password" />
      {errors.password && (
        <p className="text-red-500">{errors.password.message}</p>
      )}

      <button type="submit" disabled={isPending}>
        {isPending ? "Logging in..." : "Login"}
      </button>
    </form>
  );
}
```

### 2. Protected Routes (Middleware)

**Advanced Middleware Pattern**

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// Define route patterns
const publicRoutes = ["/login", "/register", "/", "/forgot-password"];
const protectedRoutes = ["/dashboard", "/profile", "/settings"];
const adminRoutes = ["/admin"];

// Check if route matches pattern
const isPublicRoute = (pathname: string) => {
  return publicRoutes.some(
    (route) => pathname === route || pathname.startsWith(`${route}/`)
  );
};

const isProtectedRoute = (pathname: string) => {
  return protectedRoutes.some((route) => pathname.startsWith(route));
};

const isAdminRoute = (pathname: string) => {
  return adminRoutes.some((route) => pathname.startsWith(route));
};

// Verify JWT token
async function verifyToken(
  token: string
): Promise<{ valid: boolean; user?: any }> {
  try {
    // Call your auth API to verify token
    const response = await fetch(
      `${process.env.NEXT_PUBLIC_API_URL}/api/auth/verify`,
      {
        headers: { Authorization: `Bearer ${token}` },
      }
    );

    if (!response.ok) {
      return { valid: false };
    }

    const user = await response.json();
    return { valid: true, user };
  } catch {
    return { valid: false };
  }
}

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const token = request.cookies.get("access_token")?.value;

  // Handle public routes
  if (isPublicRoute(pathname)) {
    // Redirect authenticated users away from login/register
    if (token && (pathname === "/login" || pathname === "/register")) {
      return NextResponse.redirect(new URL("/dashboard", request.url));
    }
    return NextResponse.next();
  }

  // Handle protected routes
  if (isProtectedRoute(pathname) || isAdminRoute(pathname)) {
    if (!token) {
      // Redirect to login with return URL
      const loginUrl = new URL("/login", request.url);
      loginUrl.searchParams.set("redirect", pathname);
      return NextResponse.redirect(loginUrl);
    }

    // Verify token
    const { valid, user } = await verifyToken(token);

    if (!valid) {
      // Invalid token, redirect to login
      const loginUrl = new URL("/login", request.url);
      loginUrl.searchParams.set("redirect", pathname);
      return NextResponse.redirect(loginUrl);
    }

    // Check admin routes
    if (isAdminRoute(pathname) && user.role !== "admin") {
      return NextResponse.redirect(new URL("/dashboard", request.url));
    }

    // Add user info to request headers for use in Server Components
    const requestHeaders = new Headers(request.headers);
    requestHeaders.set("x-user-id", user.id);
    requestHeaders.set("x-user-role", user.role);

    return NextResponse.next({
      request: {
        headers: requestHeaders,
      },
    });
  }

  return NextResponse.next();
}

export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     * - public files (public folder)
     */
    "/((?!api|_next/static|_next/image|favicon.ico|.*\\..*|public).*)",
  ],
};
```

### 3. Auth State Management (Zustand Store)

**Auth Store dengan Zustand**

```typescript
// store/useAuthStore.ts
import { create } from "zustand";
import { persist, createJSONStorage } from "zustand/middleware";
import { devtools } from "zustand/middleware";

interface User {
  id: string;
  email: string;
  role: string;
  name?: string;
  permissions?: string[];
}

interface AuthStore {
  user: User | null;
  isAuthenticated: boolean;
  setAuth: (user: User) => void;
  setUser: (user: User) => void;
  logout: () => void;
}

const initialState = {
  user: null,
  isAuthenticated: false,
};

export const useAuthStore = create<AuthStore>()(
  devtools(
    persist(
      (set) => ({
        ...initialState,
        setAuth: (user) =>
          set({
            user,
            isAuthenticated: true,
          }),
        setUser: (user) => set({ user }),
        logout: () => set(initialState),
      }),
      {
        name: "auth-storage",
        storage: createJSONStorage(() => localStorage),
        partialize: (state) => ({
          // Only persist user data, not tokens (tokens in httpOnly cookies)
          user: state.user,
        }),
      }
    ),
    { name: "AuthStore" }
  )
);
```

**Auth Hook dengan Zustand Store**

```typescript
// hooks/useAuth.ts
"use client";

import { useAuthStore } from "@/store/useAuthStore";
import { useRouter } from "next/navigation";
import { useEffect, useMemo } from "react";
import { useCookies } from "react-cookie";

interface User {
  id: string;
  email: string;
  role: string;
  permissions?: string[];
}

export function useAuth() {
  const router = useRouter();
  const [cookies] = useCookies(["user", "access_token"]);

  const {
    user: storeUser,
    isAuthenticated: storeIsAuthenticated,
    setAuth,
    setUser,
    logout: storeLogout,
  } = useAuthStore();

  // Sync dengan cookies saat mount
  useEffect(() => {
    if (cookies.user && cookies.access_token && !storeUser) {
      try {
        const userData = JSON.parse(cookies.user);
        setUser(userData);
      } catch {
        // Ignore parse error
      }
    }
  }, [cookies.user, cookies.access_token, storeUser, setUser]);

  // Use store user atau fallback ke cookies
  const user: User | null =
    storeUser ||
    useMemo(() => {
      if (!cookies.user) return null;
      try {
        return JSON.parse(cookies.user);
      } catch {
        return null;
      }
    }, [cookies.user]);

  const isAuthenticated = storeIsAuthenticated || !!cookies.access_token;

  const hasRole = (roles: string | string[]): boolean => {
    if (!user) return false;
    const roleArray = Array.isArray(roles) ? roles : [roles];
    return roleArray.includes(user.role);
  };

  const hasPermission = (permission: string): boolean => {
    if (!user?.permissions) return false;
    return user.permissions.includes(permission);
  };

  const requireAuth = () => {
    if (!isAuthenticated) {
      router.push("/login");
      return false;
    }
    return true;
  };

  const requireRole = (roles: string | string[]) => {
    if (!requireAuth()) return false;
    if (!hasRole(roles)) {
      router.push("/unauthorized");
      return false;
    }
    return true;
  };

  const logout = () => {
    storeLogout();
    // Cookies akan dihapus oleh Server Action logout
    router.push("/login");
  };

  return {
    user,
    isAuthenticated,
    hasRole,
    hasPermission,
    requireAuth,
    requireRole,
    logout,
    setAuth: (user: User) => {
      // Update store setelah login
      setAuth(user);
    },
  };
}
```

**Update Login Server Action untuk Sync dengan Store**

```typescript
// actions/authActions.ts
"use server";

import { cookies } from "next/headers";
import { redirect } from "next/navigation";
import { CoreHttpResponse } from "@/types/api/common";
import axios from "axios";

export type AuthActionState = {
  success: boolean;
  message?: string;
  errors?: Record<string, string[]>;
  user?: {
    id: string;
    email: string;
    role: string;
    name?: string;
  };
};

export async function login(
  prevState: AuthActionState,
  formData: FormData
): Promise<AuthActionState> {
  try {
    const email = formData.get("email") as string;
    const password = formData.get("password") as string;

    // API call untuk login
    const res: CoreHttpResponse<{
      access_token: string;
      refresh_token: string;
      user: { id: string; email: string; role: string; name?: string };
    }> = await axios.post("/api/auth/login", {
      email,
      password,
    });

    const { access_token, refresh_token, user } = res.data;

    // Set cookies dengan httpOnly flag untuk security
    const cookieStore = await cookies();
    cookieStore.set("access_token", access_token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24, // 24 hours
      path: "/",
    });

    cookieStore.set("refresh_token", refresh_token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24 * 7, // 7 days
      path: "/",
    });

    // Store user data in cookie (non-sensitive)
    cookieStore.set("user", JSON.stringify(user), {
      httpOnly: false, // Allow client-side access untuk Zustand sync
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24,
      path: "/",
    });

    // Return user data untuk client-side state update
    return {
      success: true,
      user,
    };
  } catch (error: any) {
    return {
      success: false,
      message: error.response?.data?.message || "Login failed",
      errors: error.response?.data?.errors,
    };
  }
}
```

**Update Login Form untuk Sync dengan Store**

```typescript
"use client";

import { useActionState } from "react";
import { login, AuthActionState } from "@/actions/authActions";
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import * as yup from "yup";
import { useRouter } from "next/navigation";
import { useEffect } from "react";
import { toast } from "sonner";
import { useAuth } from "@/hooks/useAuth";

const loginSchema = yup.object({
  email: yup.string().email().required("Email wajib diisi"),
  password: yup.string().required("Password wajib diisi"),
});

type LoginFormData = yup.InferType<typeof loginSchema>;

export default function LoginForm() {
  const router = useRouter();
  const { setAuth } = useAuth();
  const [state, formAction, isPending] = useActionState<
    AuthActionState,
    FormData
  >(login, { success: false });

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginFormData>({
    resolver: yupResolver(loginSchema),
  });

  useEffect(() => {
    if (state.success && state.user) {
      // Update Zustand store dengan user data
      // Tokens sudah di cookies (httpOnly), tidak perlu di store
      setAuth(state.user);

      toast.success("Login berhasil");
      router.push("/dashboard");
    } else if (state.message) {
      toast.error(state.message);
    }
  }, [state, router, setAuth]);

  const onSubmit = (data: LoginFormData) => {
    const formData = new FormData();
    formData.append("email", data.email);
    formData.append("password", data.password);
    formAction(formData);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} type="email" placeholder="Email" />
      {errors.email && <p className="text-red-500">{errors.email.message}</p>}

      <input {...register("password")} type="password" placeholder="Password" />
      {errors.password && (
        <p className="text-red-500">{errors.password.message}</p>
      )}

      <button type="submit" disabled={isPending}>
        {isPending ? "Logging in..." : "Login"}
      </button>
    </form>
  );
}
```

**React Query Hook untuk Fetch User Data (Optional)**

```typescript
// hooks/useCurrentUser.ts
import { useQuery } from "@tanstack/react-query";
import { getCurrentUser } from "@/api/authApi";
import { useAuthStore } from "@/store/useAuthStore";
import { useEffect } from "react";

export function useCurrentUser() {
  const { setUser } = useAuthStore();

  const query = useQuery({
    queryKey: ["currentUser"],
    queryFn: getCurrentUser,
    enabled:
      typeof window !== "undefined" &&
      !!document.cookie.includes("access_token"),
    staleTime: 5 * 60 * 1000, // 5 minutes
    retry: 1,
  });

  // Sync dengan Zustand store
  useEffect(() => {
    if (query.data) {
      setUser(query.data);
    }
  }, [query.data, setUser]);

  return query;
}
```

### 4. Role-Based Access Control (RBAC)

**RBAC Hook (Updated dengan Zustand)**

```typescript
// hooks/useAuth.ts sudah di atas dengan RBAC methods
// hasRole, hasPermission, requireAuth, requireRole sudah included
```

**RBAC Component Wrapper**

```typescript
// components/RequireAuth.tsx
"use client";

import { useAuth } from "@/hooks/useAuth";
import { useRouter } from "next/navigation";
import { useEffect } from "react";

interface RequireAuthProps {
  children: React.ReactNode;
  roles?: string | string[];
  permissions?: string | string[];
  fallback?: React.ReactNode;
}

export function RequireAuth({
  children,
  roles,
  permissions,
  fallback,
}: RequireAuthProps) {
  const { isAuthenticated, hasRole, hasPermission } = useAuth();
  const router = useRouter();

  useEffect(() => {
    if (!isAuthenticated) {
      router.push("/login");
    }
  }, [isAuthenticated, router]);

  if (!isAuthenticated) {
    return fallback || <div>Loading...</div>;
  }

  if (roles && !hasRole(roles)) {
    return fallback || <div>Unauthorized. Insufficient role.</div>;
  }

  if (permissions) {
    const permissionArray = Array.isArray(permissions)
      ? permissions
      : [permissions];
    const hasAllPermissions = permissionArray.every((p) => hasPermission(p));

    if (!hasAllPermissions) {
      return fallback || <div>Unauthorized. Insufficient permissions.</div>;
    }
  }

  return <>{children}</>;
}
```

**Usage**

```typescript
"use client";

import { RequireAuth } from "@/components/RequireAuth";

export default function AdminDashboard() {
  return (
    <RequireAuth roles="admin">
      <div>Admin Dashboard Content</div>
    </RequireAuth>
  );
}
```

### 5. JWT Token Refresh

**Token Refresh Hook**

```typescript
// hooks/useTokenRefresh.ts
"use client";

import { useCookies } from "react-cookie";
import { useRouter } from "next/navigation";
import { useCallback, useEffect } from "react";
import axios from "axios";

export function useTokenRefresh() {
  const [cookies, setCookie, removeCookie] = useCookies([
    "access_token",
    "refresh_token",
  ]);
  const router = useRouter();

  const refreshToken = useCallback(async () => {
    try {
      const refresh_token = cookies.refresh_token;

      if (!refresh_token) {
        throw new Error("No refresh token");
      }

      const res = await axios.post("/api/auth/refresh", {
        refresh_token,
      });

      const { access_token } = res.data;

      // Update access token cookie
      setCookie("access_token", access_token, {
        httpOnly: true,
        secure: process.env.NODE_ENV === "production",
        sameSite: "lax",
        maxAge: 60 * 60 * 24,
        path: "/",
      });

      return access_token;
    } catch (error) {
      // Refresh failed, logout user
      removeCookie("access_token");
      removeCookie("refresh_token");
      removeCookie("user");
      router.push("/login");
      throw error;
    }
  }, [cookies.refresh_token, setCookie, removeCookie, router]);

  // Auto-refresh token before expiry
  useEffect(() => {
    const interval = setInterval(() => {
      if (cookies.access_token) {
        refreshToken().catch(console.error);
      }
    }, 50 * 60 * 1000); // Refresh every 50 minutes (if token expires in 60 min)

    return () => clearInterval(interval);
  }, [cookies.access_token, refreshToken]);

  return { refreshToken };
}
```

**Axios Interceptor untuk Auto Refresh**

```typescript
// lib/axios.ts
import axios, { AxiosError, InternalAxiosRequestConfig } from "axios";

const axiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

// Request interceptor untuk attach token
axiosInstance.interceptors.request.use((config: InternalAxiosRequestConfig) => {
  if (typeof window !== "undefined") {
    const token = document.cookie
      .split("; ")
      .find((row) => row.startsWith("access_token="))
      ?.split("=")[1];

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
  }
  return config;
});

// Response interceptor untuk handle 401 dan refresh token
axiosInstance.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & {
      _retry?: boolean;
    };

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Refresh token
        const refreshToken = document.cookie
          .split("; ")
          .find((row) => row.startsWith("refresh_token="))
          ?.split("=")[1];

        if (!refreshToken) {
          throw new Error("No refresh token");
        }

        const res = await axios.post("/api/auth/refresh", {
          refresh_token: refreshToken,
        });

        const { access_token } = res.data;

        // Update cookie
        document.cookie = `access_token=${access_token}; path=/; max-age=${
          60 * 60 * 24
        }; SameSite=Lax`;

        // Retry original request with new token
        if (originalRequest.headers) {
          originalRequest.headers.Authorization = `Bearer ${access_token}`;
        }

        return axiosInstance(originalRequest);
      } catch (refreshError) {
        // Refresh failed, redirect to login
        window.location.href = "/login";
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);

export default axiosInstance;
```

### 6. Server Component Authentication

**Get User in Server Component**

```typescript
// lib/auth.ts
import { cookies, headers } from "next/headers";

export async function getServerUser() {
  const cookieStore = await cookies();
  const token = cookieStore.get("access_token")?.value;

  if (!token) {
    return null;
  }

  // Verify token and get user
  try {
    const response = await fetch(
      `${process.env.NEXT_PUBLIC_API_URL}/api/auth/me`,
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
        cache: "no-store",
      }
    );

    if (!response.ok) {
      return null;
    }

    return await response.json();
  } catch {
    return null;
  }
}

export async function requireServerAuth() {
  const user = await getServerUser();

  if (!user) {
    throw new Error("Unauthorized");
  }

  return user;
}
```

**Usage di Server Component**

```typescript
// app/dashboard/page.tsx
import { requireServerAuth } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const user = await requireServerAuth();

  if (!user) {
    redirect("/login");
  }

  return <div>Welcome, {user.email}</div>;
}
```

### 7. Password Reset Flow

**Password Reset Server Actions**

```typescript
// actions/authActions.ts
"use server";

import { cookies } from "next/headers";
import { redirect } from "next/navigation";

export async function requestPasswordReset(email: string) {
  try {
    // Send reset email
    await axios.post("/api/auth/forgot-password", { email });
    return { success: true, message: "Reset link sent to your email" };
  } catch (error: any) {
    return {
      success: false,
      message: error.response?.data?.message || "Failed to send reset link",
    };
  }
}

export async function resetPassword(
  token: string,
  password: string,
  confirmPassword: string
) {
  if (password !== confirmPassword) {
    return { success: false, message: "Passwords do not match" };
  }

  try {
    await axios.post("/api/auth/reset-password", {
      token,
      password,
    });

    return { success: true, message: "Password reset successful" };
  } catch (error: any) {
    return {
      success: false,
      message: error.response?.data?.message || "Failed to reset password",
    };
  }
}
```

## Summary Checklist

1. [ ] **Auth Store**: Zustand store untuk menyimpan user data di client
2. [ ] **Login Flow**: Server Action untuk handle login dengan cookie management + Zustand sync
3. [ ] **Auth Hook**: useAuth hook dengan Zustand store integration
4. [ ] **Logout Flow**: Clear Zustand store, cookies, dan redirect
5. [ ] **Middleware**: Protect routes dengan token verification
6. [ ] **RBAC**: Role-based access control dengan hooks dan components
7. [ ] **Token Refresh**: Auto-refresh JWT tokens before expiry
8. [ ] **Axios Interceptor**: Handle 401 errors dengan auto token refresh
9. [ ] **Server Auth**: Get user in Server Components
10. [ ] **Protected Components**: RequireAuth wrapper component
11. [ ] **Password Reset**: Forgot password dan reset password flow
12. [ ] **Security**: httpOnly cookies untuk tokens, Zustand untuk user data (non-sensitive)
13. [ ] **State Sync**: Sync Zustand store dengan cookies saat mount
