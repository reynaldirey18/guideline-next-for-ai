# Testing & Quality Standards

## 🧪 Testing & Quality

### 1. Testing Setup

**Dependencies**

```json
{
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.1.0",
    "@testing-library/user-event": "^14.5.0",
    "vitest": "^1.0.0",
    "jsdom": "^23.0.0",
    "@vitest/ui": "^1.0.0",
    "msw": "^2.0.0"
  }
}
```

**Vitest Configuration (`vitest.config.ts`)**

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./test/setup.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
    },
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./"),
    },
  },
});
```

**Test Setup File (`test/setup.ts`)**

```typescript
import "@testing-library/jest-dom";
import { cleanup } from "@testing-library/react";
import { afterEach } from "vitest";

// Cleanup after each test
afterEach(() => {
  cleanup();
});
```

### 2. Component Testing

**Basic Component Test**

```typescript
// components/atoms/Button.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi } from "vitest";
import { Button } from "./Button";

describe("Button", () => {
  it("renders button with text", () => {
    render(<Button>Click me</Button>);
    expect(
      screen.getByRole("button", { name: /click me/i })
    ).toBeInTheDocument();
  });

  it("calls onClick handler when clicked", async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();

    render(<Button onClick={handleClick}>Click me</Button>);

    const button = screen.getByRole("button", { name: /click me/i });
    await user.click(button);

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("is disabled when disabled prop is true", () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole("button")).toBeDisabled();
  });

  it("applies custom className", () => {
    render(<Button className="custom-class">Click me</Button>);
    expect(screen.getByRole("button")).toHaveClass("custom-class");
  });
});
```

**Form Component Testing**

```typescript
// components/forms/LoginForm.test.tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi, beforeEach } from "vitest";
import LoginForm from "./LoginForm";
import * as authActions from "@/actions/authActions";

vi.mock("@/actions/authActions", () => ({
  login: vi.fn(),
}));

describe("LoginForm", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("renders email and password fields", () => {
    render(<LoginForm />);

    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole("button", { name: /login/i })).toBeInTheDocument();
  });

  it("shows validation errors for empty fields", async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    const submitButton = screen.getByRole("button", { name: /login/i });
    await user.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/email wajib diisi/i)).toBeInTheDocument();
      expect(screen.getByText(/password wajib diisi/i)).toBeInTheDocument();
    });
  });

  it("submits form with valid data", async () => {
    const user = userEvent.setup();
    const mockLogin = vi.mocked(authActions.login);
    mockLogin.mockResolvedValue({ success: true });

    render(<LoginForm />);

    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole("button", { name: /login/i });

    await user.type(emailInput, "test@example.com");
    await user.type(passwordInput, "password123");
    await user.click(submitButton);

    await waitFor(() => {
      expect(mockLogin).toHaveBeenCalledWith(
        expect.anything(),
        expect.objectContaining({
          get: expect.any(Function),
        })
      );
    });
  });
});
```

### 3. Hook Testing

**Custom Hook Test**

```typescript
// hooks/useAuth.test.ts
import { renderHook, act, waitFor } from "@testing-library/react";
import { describe, it, expect, vi, beforeEach } from "vitest";
import { useAuth } from "./useAuth";
import { CookiesProvider } from "react-cookie";

// Mock next/navigation
vi.mock("next/navigation", () => ({
  useRouter: () => ({
    push: vi.fn(),
  }),
}));

describe("useAuth", () => {
  it("returns null user when not authenticated", () => {
    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <CookiesProvider>{children}</CookiesProvider>
    );

    const { result } = renderHook(() => useAuth(), { wrapper });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
  });

  it("returns user when authenticated", () => {
    const mockUser = { id: "1", email: "test@example.com", role: "user" };
    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <CookiesProvider defaultSetCookies={[["user", JSON.stringify(mockUser)]]}>
        {children}
      </CookiesProvider>
    );

    const { result } = renderHook(() => useAuth(), { wrapper });

    expect(result.current.user).toEqual(mockUser);
    expect(result.current.isAuthenticated).toBe(true);
  });

  it("checks user role correctly", () => {
    const mockUser = { id: "1", email: "test@example.com", role: "admin" };
    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <CookiesProvider defaultSetCookies={[["user", JSON.stringify(mockUser)]]}>
        {children}
      </CookiesProvider>
    );

    const { result } = renderHook(() => useAuth(), { wrapper });

    expect(result.current.hasRole("admin")).toBe(true);
    expect(result.current.hasRole("user")).toBe(false);
    expect(result.current.hasRole(["admin", "moderator"])).toBe(true);
  });
});
```

### 4. API Testing dengan MSW

**Mock Service Worker Setup (`test/msw-handlers.ts`)**

```typescript
import { http, HttpResponse } from "msw";

export const handlers = [
  // Mock GET request
  http.get("/api/users", () => {
    return HttpResponse.json([
      { id: "1", name: "John Doe", email: "john@example.com" },
      { id: "2", name: "Jane Doe", email: "jane@example.com" },
    ]);
  }),

  // Mock POST request
  http.post("/api/auth/login", async ({ request }) => {
    const body = (await request.json()) as { email: string; password: string };

    if (body.email === "test@example.com" && body.password === "password") {
      return HttpResponse.json({
        access_token: "mock-token",
        refresh_token: "mock-refresh-token",
        user: { id: "1", email: body.email, role: "user" },
      });
    }

    return HttpResponse.json(
      { message: "Invalid credentials" },
      { status: 401 }
    );
  }),

  // Mock error response
  http.get("/api/error", () => {
    return HttpResponse.json({ message: "Server error" }, { status: 500 });
  }),
];
```

**Test dengan MSW**

```typescript
// hooks/useChatToken.test.ts
import { renderHook, waitFor } from "@testing-library/react";
import { describe, it, expect, beforeEach } from "vitest";
import { setupServer } from "msw/node";
import { handlers } from "@/test/msw-handlers";
import { useChatToken } from "./useChatToken";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe("useChatToken", () => {
  it("fetches chat token successfully", async () => {
    const queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } },
    });

    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
    );

    const { result } = renderHook(() => useChatToken(), { wrapper });

    await waitFor(() => expect(result.current.isSuccess).toBe(true));

    expect(result.current.data).toBeDefined();
    expect(result.current.data?.access_token).toBe("mock-token");
  });

  it("handles error correctly", async () => {
    server.use(
      http.get("/api/chat/get-token", () => {
        return HttpResponse.json({ message: "Unauthorized" }, { status: 401 });
      })
    );

    const queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } },
    });

    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
    );

    const { result } = renderHook(() => useChatToken(), { wrapper });

    await waitFor(() => expect(result.current.isError).toBe(true));

    expect(result.current.error).toBeDefined();
  });
});
```

### 5. Server Component Testing

**Server Component Test**

```typescript
// app/dashboard/page.test.tsx
import { describe, it, expect, vi, beforeEach } from "vitest";
import { render } from "@testing-library/react";
import DashboardPage from "./page";
import * as authLib from "@/lib/auth";

vi.mock("@/lib/auth", () => ({
  requireServerAuth: vi.fn(),
}));

describe("DashboardPage", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("renders dashboard for authenticated user", async () => {
    const mockUser = { id: "1", email: "test@example.com", role: "user" };
    vi.mocked(authLib.requireServerAuth).mockResolvedValue(mockUser);

    const component = await DashboardPage();
    const { container } = render(component);

    expect(container.textContent).toContain("Welcome");
    expect(container.textContent).toContain("test@example.com");
  });

  it("redirects unauthenticated user", async () => {
    vi.mocked(authLib.requireServerAuth).mockRejectedValue(
      new Error("Unauthorized")
    );

    // In actual test, you'd check redirect behavior
    // This requires Next.js test setup
    await expect(DashboardPage()).rejects.toThrow("Unauthorized");
  });
});
```

### 6. Server Actions Testing

**Server Action Test**

```typescript
// actions/userActions.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { createUser } from "./userActions";
import * as nextCache from "next/cache";

vi.mock("next/cache", () => ({
  revalidatePath: vi.fn(),
}));

describe("createUser", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("creates user successfully", async () => {
    // Mock fetch
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ id: "1", name: "John", email: "john@example.com" }),
    });

    const formData = new FormData();
    formData.append("name", "John");
    formData.append("email", "john@example.com");

    const result = await createUser({ success: false }, formData);

    expect(result.success).toBe(true);
    expect(result.message).toBe("User created successfully");
    expect(nextCache.revalidatePath).toHaveBeenCalledWith("/users");
  });

  it("returns validation errors", async () => {
    const formData = new FormData();
    formData.append("name", "Jo"); // Too short

    const result = await createUser({ success: false }, formData);

    expect(result.success).toBe(false);
    expect(result.errors?.name).toBeDefined();
  });
});
```

### 7. E2E Testing dengan Playwright

**Playwright Configuration (`playwright.config.ts`)**

```typescript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
  ],
  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

**E2E Test Example (`e2e/login.spec.ts`)**

```typescript
import { test, expect } from "@playwright/test";

test.describe("Login Flow", () => {
  test("should login successfully with valid credentials", async ({ page }) => {
    await page.goto("/login");

    await page.fill('input[type="email"]', "test@example.com");
    await page.fill('input[type="password"]', "password123");
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL("/dashboard");
    await expect(page.locator("text=Welcome")).toBeVisible();
  });

  test("should show error with invalid credentials", async ({ page }) => {
    await page.goto("/login");

    await page.fill('input[type="email"]', "wrong@example.com");
    await page.fill('input[type="password"]', "wrongpassword");
    await page.click('button[type="submit"]');

    await expect(page.locator("text=Invalid credentials")).toBeVisible();
    await expect(page).toHaveURL("/login");
  });
});
```

### 8. Type Checking & Linting

**Type Checking**

```bash
# TypeScript type checking
pnpm typecheck

# Watch mode
pnpm typecheck:watch
```

**Linting**

```bash
# ESLint
pnpm lint

# Fix automatically
pnpm lint:fix
```

**Pre-commit Hooks (`package.json`)**

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "typecheck": "tsc --noEmit",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "prepare": "husky install"
  }
}
```

## Summary Checklist

1. [ ] **Setup**: Vitest configured dengan React Testing Library
2. [ ] **Component Tests**: Unit tests untuk semua components
3. [ ] **Hook Tests**: Test custom hooks dengan renderHook
4. [ ] **API Tests**: MSW untuk mock API calls
5. [ ] **Server Components**: Test Server Components dengan mocks
6. [ ] **Server Actions**: Test Server Actions dengan FormData mocks
7. [ ] **E2E Tests**: Playwright untuk end-to-end testing
8. [ ] **Type Checking**: TypeScript strict mode enabled
9. [ ] **Linting**: ESLint configured dengan Next.js rules
10. [ ] **CI/CD**: Test scripts integrated in CI pipeline
