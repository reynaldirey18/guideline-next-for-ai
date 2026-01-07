# API & Data Fetching

## 🔌 API Layer Pattern

### Structure

```typescript
// api/[feature]/[feature]Api.ts
import { axiosInstance } from "@/https/core";

export interface FeatureResponse {
  // Response type
}

export async function getFeature(id: string): Promise<FeatureResponse> {
  const response = await axiosInstance.get(`/feature/${id}`);
  return response.data;
}
```

### HTTP Client

- Centralized axios instance di `https/core.ts`
- Request/response interceptors untuk auth dan error handling
- Type-safe API responses

### Server Components vs Client Components

**Server Components (Default)**

- Tidak perlu `"use client"` directive
- Render di server, tidak ada JavaScript di bundle
- Dapat langsung fetch data dan access backend resources
- Tidak bisa menggunakan hooks, event handlers, atau browser APIs

```typescript
// app/[locale]/[route]/page.tsx (Server Component)
import { getFeature } from "@/api/feature/featureApi";

export default async function FeaturePage({
  params,
}: {
  params: { id: string };
}) {
  // Direct data fetching in Server Component
  const data = await getFeature(params.id);

  return (
    <div>
      <h1>{data.title}</h1>
      {/* Client Component untuk interaktif */}
      <ClientFeatureDetails data={data} />
    </div>
  );
}
```

**Client Components**

- Perlu `"use client"` directive di top of file
- Render di browser, memiliki JavaScript bundle
- Dapat menggunakan hooks, event handlers, browser APIs
- Gunakan untuk interaktif components (forms, buttons, modals)

```typescript
// components/feature/FeatureForm.tsx (Client Component)
"use client";

import { useActionState } from "react";
import { createFeature } from "@/app/[locale]/[route]/actions";

const initialState = {
  message: '',
  errors: {}
};

export default function FeatureForm() {
  // useActionState (Next.js 15 / React 19) handle form state & errors
  const [state, formAction, isPending] = useActionState(createFeature, initialState);

  return (
    <form action={formAction}>
      <input name="title" />
      {state?.errors?.title && <p className="text-red-500">{state.errors.title}</p>}
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
      
      {state?.message && <p>{state.message}</p>}
    </form>
  );
}
```

### Server Actions

Server Actions untuk form submissions dan mutations:

```typescript
// app/[locale]/[route]/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { z } from "zod";

const schema = z.object({
  title: z.string().min(1),
});

export async function createFeature(prevState: any, formData: FormData) {
  const validatedFields = schema.safeParse({
    title: formData.get("title"),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Validation Error',
    };
  }

  try {
    // Process data
    // await db.feature.create(...)

    revalidatePath("/features");
    return { message: 'Success!' };
  } catch (e) {
    return { message: 'Failed to create feature' };
  }
}
```

### Route Handlers (API Routes)

Untuk API endpoints di Next.js:

```typescript
// app/api/[feature]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { axiosInstance } from "@/https/core";

// Next.js 15: Fetch requests are not cached by default (no-store)
// To cache: fetch('...', { cache: 'force-cache' })

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const id = searchParams.get("id");

    const data = await getFeature(id!);

    return NextResponse.json({ data });
  } catch (error) {
    return NextResponse.json(
      { error: "Internal Server Error" },
      { status: 500 }
    );
  }
}
```

### Data Fetching Patterns

**1. Server Component Data Fetching**

```typescript
// Direct fetch in Server Component
// Next.js 15 default: Dynamic (no caching) unless specified
export default async function Page({ params }: { params: { id: string } }) {
  const data = await getFeature(params.id);
  return <div>{data.title}</div>;
}
```

**2. Streaming Data with `use` Hook (Client Component)**

```typescript
// Server Component
import { Suspense } from 'react';
import FeatureClient from './FeatureClient';

export default function Page({ params }: { params: { id: string } }) {
  // Start fetching, don't await
  const featurePromise = getFeature(params.id);

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <FeatureClient featurePromise={featurePromise} />
    </Suspense>
  );
}

// Client Component
"use client";
import { use } from "react";

export default function FeatureClient({ featurePromise }: { featurePromise: Promise<any> }) {
  const data = use(featurePromise); // Unwraps promise
  return <div>{data.title}</div>;
}
```

**2. React Query (TanStack Query) untuk Client Components**

```typescript
"use client";

import { useQuery } from "@tanstack/react-query";
import { getFeature } from "@/api/feature/featureApi";

export default function FeatureComponent({ id }: { id: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ["feature", id],
    queryFn: () => getFeature(id),
  });

  if (isLoading) return <Loading />;
  if (error) return <Error />;

  return <div>{data.title}</div>;
}
```

**3. Streaming dengan Suspense**

```typescript
import { Suspense } from "react";
import Loading from "@/components/atoms/Loading";

export default function Page() {
  return (
    <Suspense fallback={<Loading />}>
      <FeatureList />
    </Suspense>
  );
}

async function FeatureList() {
  const data = await getFeatures();
  return <div>{/* render data */}</div>;
}
```

### Caching Strategies

Next.js 15 caching configuration:

```typescript
// Default in Next.js 16 is 'no-store' (dynamic) for fetch

// Force specific page to be static
export const dynamic = "force-static";

// Revalidate every 60 seconds
export const revalidate = 60;

// On-demand revalidation
import { revalidatePath, revalidateTag } from "next/cache";

export async function updateAction() {
    'use server'
    revalidatePath("/features");
    revalidateTag("features");
}
```

## 🌐 gRPC Integration (Reference)

**Note**: gRPC belum digunakan di proyek ini, tapi berikut pattern untuk referensi:

```typescript
// lib/grpc/client.ts
import * as grpc from "@grpc/grpc-js";
import * as protoLoader from "@grpc/proto-loader";

const PROTO_PATH = "./path/to/service.proto";

const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
const service = protoDescriptor.serviceName as any;

const client = new service.ServiceName(
  "localhost:50051",
  grpc.credentials.createInsecure()
);

export default client;
```
