# Performance & Optimization

## ⚡ Optimization Guidelines

### 1. Image Optimization (`next/image`)

**WAJIB** menggunakan `next/image` untuk semua gambar statis dan dinamis. Jangan gunakan `<img>` tag biasa kecuali untuk SVG sederhana.

```typescript
import Image from 'next/image'
import heroImage from './hero.png' // Static import

export default function Hero() {
  return (
    <div className="relative h-64 w-full">
      {/* Static Image */}
      <Image
        src={heroImage}
        alt="Hero Image"
        placeholder="blur" // Blur effect otomatis untuk static import
      />
      
      {/* Remote Image */}
      <Image 
        src="https://example.com/image.jpg"
        alt="Description"
        width={500}
        height={300}
        className="object-cover"
        priority={true} // Gunakan priority untuk gambar above-the-fold (LCP)
      />
    </div>
  )
}
```

### 2. Font Optimization (`next/font`)

Gunakan `next/font` untuk semua font. Ini menghilangkan Layout Shift (CLS) dan otomatis host font file.

```typescript
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google'
import { cn } from '@/lib/utils'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
})

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  variable: '--font-roboto-mono',
  display: 'swap',
})

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={cn(inter.variable, robotoMono.variable)}>
      <body>{children}</body>
    </html>
  )
}
```

### 3. Script Loading (`next/script`)

Gunakan `next/script` untuk third-party scripts (Analytics, Ads, Chat Widget).

```typescript
import Script from 'next/script'

export default function Dashboard() {
  return (
    <>
      <Script 
        src="https://third-party.com/script.js" 
        strategy="lazyOnload" // afterInteractive, beforeInteractive, lazyOnload, worker
        onLoad={() => console.log('Script loaded')}
      />
    </>
  )
}
```

### 4. Lazy Loading Components

Gunakan `next/dynamic` untuk lazy load Client Components yang berat (Charts, Maps, Rich Text Editors).

```typescript
import dynamic from 'next/dynamic'
import { Skeleton } from '@/components/ui/skeleton'

// Lazy load component
const HeavyChart = dynamic(() => import('@/components/charts/HeavyChart'), {
  loading: () => <Skeleton className="h-[400px] w-full" />,
  ssr: false // Set false jika component butuh browser API (window/document)
})

export default function Page() {
  return (
    <div>
      <h1>Analytics</h1>
      <HeavyChart />
    </div>
  )
}
```

### 5. Bundle Analyzer

Gunakan `@next/bundle-analyzer` secara berkala untuk memantau ukuran bundle JavaScript.

```bash
ANALYZE=true npm run build
```
