# API & Data Fetching Standards

This document establishes the end-to-end standard for data fetching in the application, from the raw API call to the React component consumption.

## 1. Type Definitions & API Call

**Rule**: Always define a Base Response type, then wrap it in `CoreHttpResponse<T>` when making the Axios call.

### A. Define the Types
Typically located in `types/` (e.g., `types/auth.ts` or `types/chat.ts`).

```typescript
// types/chat.ts
export interface ChatTokenResponse {
  access_token: string;
  user: {
    id: string;
    username: string;
  };
}
```

### B. Implement the API Function
Typically located in `api/` (e.g., `api/chatApi.ts`).

```typescript
import { CoreHttpResponse } from "@/types/api/common";
import axios from "axios";
import { ChatTokenResponse } from "@/types/chat";

export const getChatToken = async (): Promise<ChatTokenResponse> => {
  // 1. Explicitly type the Axios response with CoreHttpResponse
  const res: CoreHttpResponse<ChatTokenResponse> = await axios.get(
    "/api/chat/get-token"
  );
  
  // 2. Return the data payload (unwrapped from status/message if needed, or keeping structure based on requirement)
  // Common pattern: Return the data part directly
  return res.data; 
};
```

## 2. React Query Hooks

**Rule**: Encapsulate `useQuery` and `useMutation` logic inside custom hooks. Do not use `useQuery` directly in components for complex logic.

### A. Data Fetching (`useQuery`)
Create a hook file, e.g., `hooks/chat/useChatToken.ts`.

```typescript
import { useQuery } from "@tanstack/react-query";
import { getChatToken } from "@/api/chatApi";

export const useChatToken = () => {
  return useQuery({
    queryKey: ["chat", "token"], // Use consistent key namespacing
    queryFn: async () => {
      const data = await getChatToken();
      return data;
    },
    // Optional: Add default options usually needed
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};
```

### B. Data Mutation (`useMutation`)
Create a hook file for actions, e.g., `hooks/chat/useSendMessage.ts`.

```typescript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { sendMessage } from "@/api/chatApi";
import { toast } from "sonner"; // or your toast library

export const useSendMessage = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: sendMessage,
    onSuccess: () => {
      // Invalidate queries to refresh data
      queryClient.invalidateQueries({ queryKey: ["chat", "messages"] });
      toast.success("Message sent successfully");
    },
    onError: (error: any) => {
      toast.error(error?.message || "Failed to send message");
    },
  });
};
```

## 3. Component Consumption

**Rule**: Consume the custom hooks in your UI components. Handle `isLoading` and `isError` states gracefully.

```tsx
"use client";

import { useChatToken } from "@/hooks/chat/useChatToken";
import { useSendMessage } from "@/hooks/chat/useSendMessage";
import { useState } from "react";

export default function ChatComponent() {
  // 1. Consumption of Query
  const { data: tokenData, isLoading, isError } = useChatToken();
  
  // 2. Consumption of Mutation
  const { mutate: sendMessage, isPending: isSending } = useSendMessage();

  const [input, setInput] = useState("");

  if (isLoading) return <div>Loading chat...</div>;
  if (isError) return <div>Error loading chat token.</div>;

  const handleSend = () => {
    if (!input) return;
    
    sendMessage(
      { message: input, token: tokenData?.access_token }, // Arguments matching your API function
      {
        onSuccess: () => setInput(""), // Component-specific side effect
      }
    );
  };

  return (
    <div className="p-4">
      <h1>Logged in as: {tokenData?.user.username}</h1>
      
      <div className="flex gap-2">
        <input 
          value={input}
          onChange={(e) => setInput(e.target.value)}
          disabled={isSending}
          className="border p-2 rounded"
        />
        <button 
          onClick={handleSend} 
          disabled={isSending}
          className="bg-blue-500 text-white p-2 rounded"
        >
          {isSending ? "Sending..." : "Send"}
        </button>
      </div>
    </div>
  );
}
```

## 4. Next.js Server Patterns (Server Components & Actions)

While the above standard applies to Client Component data fetching, Next.js Server Components have distinct patterns.

### A. Server Component Fetching
Directly call internal logic or `fetch` without `useEffect` or `useQuery`.

```typescript
// app/users/page.tsx
export default async function Page() {
  const data = await getFeature("123"); // Direct async call
  return <div>{data.title}</div>;
}
```

### B. Server Actions (Form Mutations)
Use `useActionState` for handling form submissions that don't need rich interaction feedback or complex state.

```typescript
// actions.ts
"use server";
import { revalidatePath } from "next/cache";

export async function createFeature(prevState: any, formData: FormData) {
  // Logic...
  revalidatePath("/features");
  return { message: "Success" };
}
```

## 5. Caching Strategies

Next.js 16 caching configuration:

```typescript
// Default in Next.js 16 is 'no-store' (dynamic) for fetch

// Force specific page to be static
export const dynamic = "force-static";

// Revalidate every 60 seconds
export const revalidate = 60;
```

## Summary Checklist

1.  [ ] **Type**: Base Response defined in `types/`.
2.  [ ] **API**: `CoreHttpResponse` used in Axios call.
3.  [ ] **Hook**: `useQuery` / `useMutation` wrapped in a custom hook file.
4.  [ ] **Component**: Hook consumed with proper loading/error states.
5.  [ ] **Server**: Use Server Components for initial page loads where possible.
