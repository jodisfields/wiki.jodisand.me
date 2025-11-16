# Next.js

Next.js is a React framework for building full-stack web applications with server-side rendering and static site generation.

## Quick Start

```bash
# Create new Next.js app
npx create-next-app@latest my-app
cd my-app
npm run dev
```

## App Router (Next.js 13+)

### File Structure

```
app/
├── layout.js       # Root layout
├── page.js         # Home page
├── about/
│   └── page.js     # /about route
└── blog/
    ├── [slug]/
    │   └── page.js # /blog/[slug] dynamic route
    └── page.js     # /blog route
```

### Root Layout

```javascript
// app/layout.js
export const metadata = {
  title: 'My App',
  description: 'My awesome app',
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

### Page Component

```javascript
// app/page.js
export default function HomePage() {
  return (
    <main>
      <h1>Welcome to Next.js</h1>
    </main>
  );
}
```

## Server vs Client Components

### Server Component (Default)

```javascript
// app/blog/page.js
async function getBlogPosts() {
  const res = await fetch('https://api.example.com/posts');
  return res.json();
}

export default async function BlogPage() {
  const posts = await getBlogPosts();

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  );
}
```

### Client Component

```javascript
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

## Data Fetching

### Server-side Fetch

```javascript
// Default: cache enabled
async function getData() {
  const res = await fetch('https://api.example.com/data');
  return res.json();
}

// Revalidate every 60 seconds
async function getDataRevalidate() {
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }
  });
  return res.json();
}

// No caching
async function getDataNoCache() {
  const res = await fetch('https://api.example.com/data', {
    cache: 'no-store'
  });
  return res.json();
}
```

### Dynamic Routes

```javascript
// app/blog/[slug]/page.js
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(res => res.json());

  return posts.map(post => ({
    slug: post.slug,
  }));
}

export default async function BlogPost({ params }) {
  const { slug } = params;
  const post = await fetch(`https://api.example.com/posts/${slug}`).then(res => res.json());

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

## API Routes

```javascript
// app/api/hello/route.js
export async function GET(request) {
  return Response.json({ message: 'Hello World' });
}

export async function POST(request) {
  const body = await request.json();

  return Response.json({
    message: 'Data received',
    data: body
  });
}
```

### Dynamic API Routes

```javascript
// app/api/posts/[id]/route.js
export async function GET(request, { params }) {
  const { id } = params;

  return Response.json({
    postId: id
  });
}
```

## Navigation

### Link Component

```javascript
import Link from 'next/link';

export default function Nav() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog/hello-world">Blog Post</Link>
    </nav>
  );
}
```

### Programmatic Navigation

```javascript
'use client';

import { useRouter } from 'next/navigation';

export default function Page() {
  const router = useRouter();

  return (
    <button onClick={() => router.push('/dashboard')}>
      Go to Dashboard
    </button>
  );
}
```

## Image Optimization

```javascript
import Image from 'next/image';

export default function Avatar() {
  return (
    <Image
      src="/profile.jpg"
      alt="Profile"
      width={500}
      height={500}
      priority
    />
  );
}
```

## Metadata

```javascript
// Static metadata
export const metadata = {
  title: 'My Page',
  description: 'Page description',
};

// Dynamic metadata
export async function generateMetadata({ params }) {
  const post = await getPost(params.id);

  return {
    title: post.title,
    description: post.excerpt,
  };
}
```

## Loading States

```javascript
// app/dashboard/loading.js
export default function Loading() {
  return <div>Loading dashboard...</div>;
}
```

## Error Handling

```javascript
// app/dashboard/error.js
'use client';

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

## Middleware

```javascript
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const response = NextResponse.next();

  // Add custom header
  response.headers.set('x-custom-header', 'value');

  return response;
}

export const config = {
  matcher: '/api/:path*',
};
```

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_API_URL=https://api.example.com
DATABASE_URL=postgresql://...
```

```javascript
// Client-side (NEXT_PUBLIC_ prefix required)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Server-side only
const dbUrl = process.env.DATABASE_URL;
```

## Configuration

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    domains: ['example.com'],
  },
  async redirects() {
    return [
      {
        source: '/old-route',
        destination: '/new-route',
        permanent: true,
      },
    ];
  },
};

module.exports = nextConfig;
```

## Resources

- [Official Documentation](https://nextjs.org/docs)
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [Learn Next.js](https://nextjs.org/learn)
