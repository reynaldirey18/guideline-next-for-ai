# State Management Standards

## 🗂️ State Management Patterns

### 1. Zustand Stores (Client State)

**Basic Zustand Store**

```typescript
// store/useFeatureStore.ts
import { create } from "zustand";

interface FeatureStore {
  data: FeatureData | null;
  isLoading: boolean;
  error: string | null;
  setData: (data: FeatureData) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  reset: () => void;
}

const initialState = {
  data: null,
  isLoading: false,
  error: null,
};

export const useFeatureStore = create<FeatureStore>((set) => ({
  ...initialState,
  setData: (data) => set({ data, error: null }),
  setLoading: (isLoading) => set({ isLoading }),
  setError: (error) => set({ error, isLoading: false }),
  reset: () => set(initialState),
}));
```

**Zustand dengan Persist (LocalStorage)**

```typescript
// store/usePreferencesStore.ts
import { create } from "zustand";
import { persist, createJSONStorage } from "zustand/middleware";

interface PreferencesStore {
  theme: "light" | "dark";
  language: string;
  setTheme: (theme: "light" | "dark") => void;
  setLanguage: (language: string) => void;
}

export const usePreferencesStore = create<PreferencesStore>()(
  persist(
    (set) => ({
      theme: "light",
      language: "en",
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    {
      name: "preferences-storage",
      storage: createJSONStorage(() => localStorage),
    }
  )
);
```

**Zustand dengan DevTools**

```typescript
// store/useAuthStore.ts
import { create } from "zustand";
import { devtools } from "zustand/middleware";

interface AuthStore {
  user: User | null;
  token: string | null;
  setUser: (user: User) => void;
  setToken: (token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>()(
  devtools(
    (set) => ({
      user: null,
      token: null,
      setUser: (user) => set({ user }, false, "setUser"),
      setToken: (token) => set({ token }, false, "setToken"),
      logout: () => set({ user: null, token: null }, false, "logout"),
    }),
    { name: "AuthStore" }
  )
);
```

**Zustand dengan Selectors (Performance)**

```typescript
// store/useUserStore.ts
import { create } from "zustand";
import { shallow } from "zustand/shallow";

interface UserStore {
  user: User | null;
  profile: Profile | null;
  settings: Settings | null;
  updateUser: (user: User) => void;
  updateProfile: (profile: Profile) => void;
  updateSettings: (settings: Settings) => void;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  profile: null,
  settings: null,
  updateUser: (user) => set({ user }),
  updateProfile: (profile) => set({ profile }),
  updateSettings: (settings) => set({ settings }),
}));

// Selector untuk avoid unnecessary re-renders
export const useUser = () => useUserStore((state) => state.user);
export const useProfile = () => useUserStore((state) => state.profile);
export const useSettings = () => useUserStore((state) => state.settings);

// Usage
function UserProfile() {
  // Only re-renders when user changes
  const user = useUser();
  // Not affected by profile or settings changes
  const profile = useProfile();

  return (
    <div>
      {user?.name} - {profile?.bio}
    </div>
  );
}
```

**Zustand dengan Async Actions**

```typescript
// store/useDataStore.ts
import { create } from "zustand";

interface DataStore {
  data: Data[] | null;
  isLoading: boolean;
  error: string | null;
  fetchData: () => Promise<void>;
  createItem: (item: Data) => Promise<void>;
}

export const useDataStore = create<DataStore>((set, get) => ({
  data: null,
  isLoading: false,
  error: null,

  fetchData: async () => {
    set({ isLoading: true, error: null });
    try {
      const response = await fetch("/api/data");
      const data = await response.json();
      set({ data, isLoading: false });
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : "Failed to fetch data",
        isLoading: false,
      });
    }
  },

  createItem: async (item) => {
    try {
      const response = await fetch("/api/data", {
        method: "POST",
        body: JSON.stringify(item),
      });
      const newItem = await response.json();

      // Update local state
      const currentData = get().data || [];
      set({ data: [...currentData, newItem] });
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : "Failed to create item",
      });
    }
  },
}));
```

### 2. React Query (Server State)

**Use React Query untuk Server State** (see `overview-api.md` for details)

- Server state managed by React Query
- Client state managed by Zustand
- Sync Zustand dengan React Query jika perlu

**Combining Zustand dengan React Query**

```typescript
// hooks/useDataWithStore.ts
import { useQuery } from "@tanstack/react-query";
import { useDataStore } from "@/store/useDataStore";

export function useDataWithStore() {
  const { data: storeData, setData } = useDataStore();

  const query = useQuery({
    queryKey: ["data"],
    queryFn: fetchData,
    onSuccess: (data) => {
      // Sync dengan Zustand store
      setData(data);
    },
  });

  // Return React Query state but sync dengan store
  return {
    ...query,
    data: storeData || query.data,
  };
}
```

### 3. Context API (Component-specific State)

**Use Context API untuk:**

- Theme provider
- Modal/Toast state
- Form context
- Local UI state yang perlu shared di subtree

**Theme Context Example**

```typescript
// provider/ThemeProvider.tsx
"use client";

import {
  createContext,
  useContext,
  useState,
  useEffect,
  ReactNode,
} from "react";

type Theme = "light" | "dark";

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>("light");

  useEffect(() => {
    // Load from localStorage
    const savedTheme = localStorage.getItem("theme") as Theme;
    if (savedTheme) {
      setTheme(savedTheme);
    }
  }, []);

  useEffect(() => {
    // Apply theme
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

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error("useTheme must be used within ThemeProvider");
  }
  return context;
}
```

**Form Context Example**

```typescript
// provider/FormProvider.tsx
"use client";

import { createContext, useContext, ReactNode } from "react";
import {
  FormProvider as RHFFormProvider,
  UseFormReturn,
} from "react-hook-form";

interface FormProviderProps<T extends Record<string, any>> {
  methods: UseFormReturn<T>;
  children: ReactNode;
}

export function FormProvider<T extends Record<string, any>>({
  methods,
  children,
}: FormProviderProps<T>) {
  return <RHFFormProvider {...methods}>{children}</RHFFormProvider>;
}

export function useFormContext<T extends Record<string, any>>() {
  return useContext(ReactHookFormContext) as UseFormReturn<T>;
}
```

### 4. State Management Guidelines

**When to Use Each:**

- **Zustand**: Global client state, user preferences, UI state
- **React Query**: Server state, API data, caching
- **Context API**: Component tree-specific state, providers
- **useState**: Component-local state
- **URL State**: Search params, filters (shareable state)

**State Organization**

```typescript
// ✅ Good - Separate stores by domain
store/
  ├── useAuthStore.ts      // Authentication state
  ├── useUserStore.ts      // User data
  ├── usePreferencesStore.ts // User preferences
  └── useUIStore.ts        // UI state (modals, sidebar, etc.)

// ❌ Bad - Single massive store
store/
  └── useAppStore.ts       // Everything in one store
```

### 5. Rematch Models

**Rematch untuk complex state management**

```typescript
// rematch/models/user.ts
export const user = {
  state: {
    data: null,
    isLoading: false,
  },
  reducers: {
    setData: (state: any, payload: User) => ({
      ...state,
      data: payload,
    }),
    setLoading: (state: any, payload: boolean) => ({
      ...state,
      isLoading: payload,
    }),
  },
  effects: {
    async fetchUser(payload: string, rootState: any) {
      this.setLoading(true);
      try {
        const user = await fetchUserById(payload);
        this.setData(user);
      } catch (error) {
        console.error(error);
      } finally {
        this.setLoading(false);
      }
    },
  },
};
```

**Lokasi**: `rematch/models/`

## Summary Checklist

1. [ ] **Zustand**: Global client state management
2. [ ] **Persist**: LocalStorage persistence untuk user preferences
3. [ ] **DevTools**: Zustand DevTools enabled untuk debugging
4. [ ] **Selectors**: Use selectors untuk avoid unnecessary re-renders
5. [ ] **Async Actions**: Handle async operations in Zustand stores
6. [ ] **React Query**: Server state managed by React Query
7. [ ] **Context API**: Use untuk component tree-specific state
8. [ ] **State Organization**: Separate stores by domain
9. [ ] **Rematch**: Use untuk complex state management (if needed)
10. [ ] **State Guidelines**: Follow best practices untuk state selection
