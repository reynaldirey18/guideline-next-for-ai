# Error Handling Standards

## 🚨 Error Handling Patterns

### ⚠️ PRINCIPLE: React Query sebagai Primary Error Handling

**Untuk semua client-side data fetching, SELALU gunakan React Query untuk error handling karena:**

1. ✅ **Built-in Error States**: `isError`, `error`, `isLoading` sudah tersedia
2. ✅ **Retry Logic**: Automatic retry dengan exponential backoff
3. ✅ **Error Handling Hooks**: `onError` callback untuk global atau per-query
4. ✅ **Error Recovery**: Built-in `refetch()` untuk retry manual
5. ✅ **Consistent Pattern**: Sama dengan data fetching pattern di `overview-api.md`

**Kapan TIDAK pakai React Query:**

- ❌ Server Actions (pakai try-catch di Server Action)
- ❌ Server Components (pakai try-catch langsung)
- ❌ Route Handlers (pakai try-catch di route handler)

### 1. Error Boundaries

**Global Error Boundary**

```typescript
// app/error.tsx
"use client";

import { useEffect } from "react";
import { Button } from "@/components/atoms/Button";

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log error to error reporting service
    console.error("Error:", error);
  }, [error]);

  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <Button onClick={reset}>Try again</Button>
    </div>
  );
}
```

**Route-specific Error Boundary**

```typescript
// app/dashboard/error.tsx
"use client";

import { useEffect } from "react";
import { useRouter } from "next/navigation";

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  const router = useRouter();

  useEffect(() => {
    // Log to error reporting service
    console.error("Dashboard error:", error);
  }, [error]);

  return (
    <div className="p-8">
      <h2 className="text-xl font-bold mb-4">Dashboard Error</h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <div className="flex gap-4">
        <button
          onClick={reset}
          className="px-4 py-2 bg-blue-500 text-white rounded"
        >
          Try again
        </button>
        <button
          onClick={() => router.push("/")}
          className="px-4 py-2 bg-gray-500 text-white rounded"
        >
          Go home
        </button>
      </div>
    </div>
  );
}
```

**Custom Error Boundary Component**

```typescript
// components/ErrorBoundary.tsx
"use client";

import React, { Component, ReactNode } from "react";

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log error to error reporting service
    console.error("ErrorBoundary caught an error:", error, errorInfo);

    // Send to error tracking service (e.g., Sentry)
    // Sentry.captureException(error, { contexts: { react: errorInfo } });
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <div className="p-8 text-center">
          <h2 className="text-xl font-bold mb-4">Something went wrong</h2>
          <p className="text-gray-600 mb-4">
            {this.state.error?.message || "An unexpected error occurred"}
          </p>
          <button
            onClick={() => this.setState({ hasError: false, error: null })}
            className="px-4 py-2 bg-blue-500 text-white rounded"
          >
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

**Usage**

```typescript
// app/layout.tsx
import { ErrorBoundary } from "@/components/ErrorBoundary";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <ErrorBoundary>{children}</ErrorBoundary>
      </body>
    </html>
  );
}
```

### 2. Global Error Handler

**Global Error Handler Utility**

```typescript
// lib/errorHandler.ts
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public details?: any
  ) {
    super(message);
    this.name = "AppError";
  }
}

export function handleError(error: unknown): {
  message: string;
  code: string;
  statusCode: number;
} {
  // Log error
  console.error("Error:", error);

  // Handle known error types
  if (error instanceof AppError) {
    return {
      message: error.message,
      code: error.code,
      statusCode: error.statusCode,
    };
  }

  // Handle API errors
  if (error && typeof error === "object" && "response" in error) {
    const apiError = error as any;
    return {
      message: apiError.response?.data?.message || "API request failed",
      code: apiError.response?.data?.code || "API_ERROR",
      statusCode: apiError.response?.status || 500,
    };
  }

  // Handle network errors
  if (error instanceof TypeError && error.message.includes("fetch")) {
    return {
      message: "Network error. Please check your connection.",
      code: "NETWORK_ERROR",
      statusCode: 0,
    };
  }

  // Default error
  return {
    message:
      error instanceof Error ? error.message : "An unexpected error occurred",
    code: "UNKNOWN_ERROR",
    statusCode: 500,
  };
}
```

### 3. API Error Handling

**API Error Response Utility**

```typescript
// lib/apiError.ts
export class APIError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public code?: string,
    public details?: any
  ) {
    super(message);
    this.name = "APIError";
  }
}

export function createErrorResponse(
  message: string,
  statusCode: number = 500,
  code?: string,
  details?: any
) {
  return {
    success: false,
    message,
    code: code || "ERROR",
    ...(details && { details }),
  };
}

export function createSuccessResponse<T>(data: T, message?: string) {
  return {
    success: true,
    data,
    ...(message && { message }),
  };
}
```

**Usage in Route Handlers**

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";
import { createErrorResponse, createSuccessResponse } from "@/lib/apiError";
import { handleError } from "@/lib/errorHandler";

export async function GET(request: NextRequest) {
  try {
    const users = await fetchUsers();
    return NextResponse.json(createSuccessResponse(users));
  } catch (error) {
    const { message, code, statusCode } = handleError(error);
    return NextResponse.json(createErrorResponse(message, statusCode, code), {
      status: statusCode,
    });
  }
}
```

### 4. Server Actions Error Handling

**Server Action dengan Error Handling**

```typescript
// actions/userActions.ts
"use server";

import { revalidatePath } from "next/cache";
import { z } from "zod";

const createUserSchema = z.object({
  name: z.string().min(3),
  email: z.string().email(),
});

export type ActionState = {
  success: boolean;
  message?: string;
  errors?: Record<string, string[]>;
  fieldErrors?: Record<string, string[]>;
};

export async function createUser(
  prevState: ActionState,
  formData: FormData
): Promise<ActionState> {
  try {
    const rawData = {
      name: formData.get("name"),
      email: formData.get("email"),
    };

    // Validation
    const result = createUserSchema.safeParse(rawData);
    if (!result.success) {
      const fieldErrors: Record<string, string[]> = {};
      result.error.errors.forEach((err) => {
        const field = err.path[0] as string;
        if (!fieldErrors[field]) {
          fieldErrors[field] = [];
        }
        fieldErrors[field].push(err.message);
      });

      return {
        success: false,
        errors: { _form: ["Validation failed"] },
        fieldErrors,
      };
    }

    // Database operation
    const user = await createUserInDB(result.data);
    revalidatePath("/users");

    return {
      success: true,
      message: "User created successfully",
    };
  } catch (error) {
    // Log error
    console.error("Create user error:", error);

    // Return user-friendly error
    return {
      success: false,
      message:
        error instanceof Error
          ? error.message
          : "An unexpected error occurred. Please try again.",
    };
  }
}
```

### 5. React Query Error Handling (PRIMARY METHOD)

**⚠️ IMPORTANT: Gunakan React Query untuk semua client-side error handling**

React Query sudah built-in dengan comprehensive error handling, retry logic, dan error states. Ini adalah **standard pattern** untuk error handling di aplikasi Next.js.

**React Query Hook dengan Error Handling**

```typescript
// hooks/useUsers.ts
import { useQuery } from "@tanstack/react-query";
import { fetchUsers } from "@/api/userApi";
import { handleError } from "@/lib/errorHandler";
import { getUserFriendlyMessage } from "@/lib/errorMessages";
import { toast } from "sonner";

export function useUsers() {
  return useQuery({
    queryKey: ["users"],
    queryFn: async () => {
      try {
        return await fetchUsers();
      } catch (error) {
        // Transform error ke format yang konsisten
        const { message, code, statusCode } = handleError(error);
        throw new Error(message); // React Query akan handle ini
      }
    },
    retry: (failureCount, error: any) => {
      // Don't retry on 4xx errors
      if (error?.response?.status >= 400 && error?.response?.status < 500) {
        return false;
      }
      return failureCount < 2; // Retry max 2 times
    },
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    onError: (error: any) => {
      // Handle error dengan utility functions
      const { code } = handleError(error);
      const userMessage = getUserFriendlyMessage(code);

      // Show toast notification
      toast.error(userMessage);

      // Log to error service
      console.error("Fetch users error:", error);
    },
  });
}
```

**Component dengan React Query Error Handling**

```typescript
"use client";

import { useUsers } from "@/hooks/useUsers";
import { Show } from "@/components/atoms/Show";

export default function UsersList() {
  const { data, isLoading, isError, error, refetch } = useUsers();

  if (isLoading) {
    return <div>Loading users...</div>;
  }

  if (isError) {
    return (
      <div className="p-4 border border-red-200 bg-red-50 rounded">
        <h3 className="text-red-800 font-bold mb-2">Error Loading Users</h3>
        <p className="text-red-600 mb-4">
          {error instanceof Error ? error.message : "An error occurred"}
        </p>
        <button
          onClick={() => refetch()}
          className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
        >
          Try Again
        </button>
      </div>
    );
  }

  return (
    <div>
      {data?.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

**Mutation dengan Error Handling**

```typescript
// hooks/useCreateUser.ts
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { createUser } from "@/api/userApi";
import { handleError } from "@/lib/errorHandler";
import { getUserFriendlyMessage } from "@/lib/errorMessages";
import { toast } from "sonner";

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (userData: { name: string; email: string }) => {
      try {
        return await createUser(userData);
      } catch (error) {
        const { message, code } = handleError(error);
        throw new Error(message);
      }
    },
    onSuccess: () => {
      // Invalidate dan refetch users list
      queryClient.invalidateQueries({ queryKey: ["users"] });
      toast.success("User created successfully");
    },
    onError: (error: any) => {
      const { code } = handleError(error);
      const userMessage = getUserFriendlyMessage(code);
      toast.error(userMessage);
    },
  });
}
```

**Global Query Error Handler (Recommended Setup)**

```typescript
// provider/QueryProvider.tsx
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { handleError } from "@/lib/errorHandler";
import { getUserFriendlyMessage } from "@/lib/errorMessages";
import { toast } from "sonner";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error: any) => {
        // Don't retry on 4xx errors (client errors)
        if (error?.response?.status >= 400 && error?.response?.status < 500) {
          return false;
        }
        // Retry max 2 times untuk server errors
        return failureCount < 2;
      },
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      onError: (error: any) => {
        // Global error handler untuk semua queries
        const { code } = handleError(error);
        const userMessage = getUserFriendlyMessage(code);

        // Only show toast untuk critical errors
        if (code !== "NOT_FOUND") {
          toast.error(userMessage);
        }
      },
    },
    mutations: {
      retry: false, // Don't retry mutations by default
      onError: (error: any) => {
        // Global error handler untuk semua mutations
        const { code } = handleError(error);
        const userMessage = getUserFriendlyMessage(code);
        toast.error(userMessage);
      },
    },
  },
});

export function QueryProvider({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}
```

**Error Handling dengan Error Boundaries + React Query**

```typescript
"use client";

import { useQuery } from "@tanstack/react-query";
import { ErrorBoundary } from "@/components/ErrorBoundary";

export function UsersListWithBoundary() {
  return (
    <ErrorBoundary
      fallback={
        <div className="p-4 border border-red-200 bg-red-50 rounded">
          <h3>Something went wrong</h3>
          <p>Please refresh the page or contact support.</p>
        </div>
      }
    >
      <UsersList />
    </ErrorBoundary>
  );
}

function UsersList() {
  const { data, isError, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });

  // React Query handles error state
  if (isError) {
    // Throw error untuk ErrorBoundary catch
    throw error;
  }

  return <div>{/* Render users */}</div>;
}
```

### 6. Not Found Handling

**Not Found Page**

```typescript
// app/not-found.tsx
import Link from "next/link";
import { Button } from "@/components/atoms/Button";

export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-4xl font-bold mb-4">404</h2>
      <p className="text-gray-600 mb-4">Page not found</p>
      <Link href="/">
        <Button>Return Home</Button>
      </Link>
    </div>
  );
}
```

**Dynamic Route Not Found**

```typescript
// app/users/[id]/not-found.tsx
export default function UserNotFound() {
  return (
    <div className="p-8">
      <h2 className="text-xl font-bold mb-4">User Not Found</h2>
      <p className="text-gray-600">
        The user you're looking for doesn't exist or has been removed.
      </p>
    </div>
  );
}
```

**Usage in Server Components**

```typescript
// app/users/[id]/page.tsx
import { notFound } from "next/navigation";

export default async function UserPage({ params }: { params: { id: string } }) {
  const user = await getUserById(params.id);

  if (!user) {
    notFound(); // Triggers not-found.tsx
  }

  return <div>{user.name}</div>;
}
```

### 7. Error Logging & Monitoring

**Error Logging Utility**

```typescript
// lib/logger.ts
type LogLevel = "info" | "warn" | "error";

interface LogData {
  level: LogLevel;
  message: string;
  error?: Error;
  context?: Record<string, any>;
}

export function logError(data: LogData) {
  // Log to console in development
  if (process.env.NODE_ENV === "development") {
    console.error(
      `[${data.level.toUpperCase()}]`,
      data.message,
      data.error,
      data.context
    );
  }

  // Send to error tracking service in production
  if (process.env.NODE_ENV === "production") {
    // Example: Sentry
    // Sentry.captureException(data.error || new Error(data.message), {
    //   level: data.level,
    //   extra: data.context,
    // });

    // Or send to custom logging service
    fetch("/api/logs", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        level: data.level,
        message: data.message,
        error: data.error?.stack,
        context: data.context,
        timestamp: new Date().toISOString(),
      }),
    }).catch(console.error);
  }
}
```

**Usage**

```typescript
import { logError } from "@/lib/logger";

try {
  // Some operation
} catch (error) {
  logError({
    level: "error",
    message: "Failed to create user",
    error: error instanceof Error ? error : new Error(String(error)),
    context: { userId: "123", action: "createUser" },
  });
}
```

### 8. User-Friendly Error Messages

**Error Message Mapping**

```typescript
// lib/errorMessages.ts
const ERROR_MESSAGES: Record<string, string> = {
  NETWORK_ERROR: "Network error. Please check your internet connection.",
  VALIDATION_ERROR: "Please check your input and try again.",
  UNAUTHORIZED: "You are not authorized to perform this action.",
  NOT_FOUND: "The requested resource was not found.",
  SERVER_ERROR: "Server error. Please try again later.",
  TIMEOUT: "Request timed out. Please try again.",
};

export function getUserFriendlyMessage(errorCode: string): string {
  return (
    ERROR_MESSAGES[errorCode] ||
    "An unexpected error occurred. Please try again."
  );
}
```

**Usage**

```typescript
import { getUserFriendlyMessage } from "@/lib/errorMessages";

const error = handleError(someError);
const userMessage = getUserFriendlyMessage(error.code);
toast.error(userMessage);
```

### 9. Retry Logic

**⚠️ NOTE: Gunakan React Query retry mechanism untuk client-side. Manual retry hanya untuk Server Actions atau edge cases.**

**React Query Retry (RECOMMENDED)**

```typescript
// hooks/useData.ts
import { useQuery } from "@tanstack/react-query";

export function useData() {
  return useQuery({
    queryKey: ["data"],
    queryFn: fetchData,
    retry: (failureCount, error: any) => {
      // Custom retry logic
      if (error?.response?.status === 404) return false; // Don't retry 404
      return failureCount < 3; // Retry max 3 times
    },
    retryDelay: (attemptIndex) => {
      // Exponential backoff
      return Math.min(1000 * 2 ** attemptIndex, 30000);
    },
  });
}
```

**Manual Retry Utility (Untuk Server Actions atau Edge Cases)**

```typescript
// lib/retry.ts
// Hanya gunakan jika tidak bisa pakai React Query (e.g., Server Actions)
export async function retry<T>(
  fn: () => Promise<T>,
  maxAttempts: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error instanceof Error ? error : new Error(String(error));

      if (attempt < maxAttempts) {
        // Exponential backoff
        const waitTime = delay * Math.pow(2, attempt - 1);
        await new Promise((resolve) => setTimeout(resolve, waitTime));
      }
    }
  }

  throw lastError!;
}
```

**Usage (Server Actions only)**

```typescript
// actions/dataActions.ts
"use server";

import { retry } from "@/lib/retry";

export async function fetchDataWithRetry() {
  return retry(
    () => fetch("/api/data").then((res) => res.json()),
    3, // 3 attempts
    1000 // 1 second initial delay
  );
}
```

## Summary Checklist

1. [ ] **React Query (PRIMARY)**: Gunakan React Query untuk semua client-side error handling
2. [ ] **Error Boundaries**: Global dan route-specific error boundaries untuk unexpected errors
3. [ ] **Error Types**: Custom error classes untuk different error types
4. [ ] **Error Handling**: Comprehensive error handling di API routes
5. [ ] **Server Actions**: Proper error handling di Server Actions (tidak pakai React Query)
6. [ ] **React Query Hooks**: Custom hooks dengan error handling dan retry logic
7. [ ] **Global Query Handler**: Setup global error handler di QueryProvider
8. [ ] **Not Found**: Custom 404 pages untuk routes
9. [ ] **Error Logging**: Log errors ke monitoring service
10. [ ] **User Messages**: User-friendly error messages dengan mapping
11. [ ] **Retry Logic**: Gunakan React Query retry mechanism (bukan manual retry)
12. [ ] **Error Recovery**: Provide recovery options untuk users (refetch, retry button)
