# SEO & Metadata Standards

## 🔍 Metadata API

Next.js 16 menggunakan Metadata API untuk mendefinisikan meta tags seperti `title`, `description`, dan `Open Graph` images. Jangan gunakan `<head>` manual.

### 1. Static Metadata

Definisikan `export const metadata` di `layout.tsx` atau `page.tsx`.

```typescript
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    template: '%s | NamaAplikasi',
    default: 'NamaAplikasi - Tagline',
  },
  description: 'Deskripsi aplikasi yang menarik dan SEO friendly.',
  openGraph: {
    images: ['/og-image.png'],
  },
}
```

### 2. Dynamic Metadata

Gunakan `generateMetadata` untuk route dinamis yang membutuhkan data fetching.

```typescript
// app/products/[id]/page.tsx
import type { Metadata, ResolvingMetadata } from 'next'

type Props = {
  params: { id: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export async function generateMetadata(
  { params, searchParams }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  // read route params
  const id = params.id

  // fetch data
  const product = await fetch(`https://api.example.com/products/${id}`).then((res) => res.json())

  // optionally access and extend (rather than replace) parent metadata
  const previousImages = (await parent).openGraph?.images || []

  return {
    title: product.title,
    openGraph: {
      images: ['/some-specific-page-image.jpg', ...previousImages],
    },
  }
}
```

### 3. Sitemap & Robots

Gunakan file khusus untuk generate sitemap dan robots.txt.

**app/sitemap.ts**

```typescript
import { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://acme.com',
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 1,
    },
    {
      url: 'https://acme.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    // ...dynamic routes logic
  ]
}
```

**app/robots.ts**

```typescript
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/private/',
    },
    sitemap: 'https://acme.com/sitemap.xml',
  }
}
```

### 4. JSON-LD Structured Data

Untuk SEO tingkat lanjut, render JSON-LD di dalam komponen.

```typescript
export default function Page({ data }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: data.name,
    image: data.image,
    description: data.description,
  }

  return (
    <section>
      {/* Add JSON-LD to your page */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* ... */}
    </section>
  )
}
```
