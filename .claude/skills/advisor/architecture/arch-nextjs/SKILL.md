---
name: arch-nextjs
description: Evaluate Next.js architecture decisions including App Router vs Pages Router, rendering strategies (SSR vs SSG vs ISR vs Dynamic), Server Components, data fetching patterns, and deployment options. Use when user needs guidance on Next.js architecture choices.
context: fork
agent: Explore
---

# Next.js Architecture Decision Evaluator

Evaluate and recommend Next.js configurations, rendering strategies, and architectural patterns for TypeScript applications.

## Standalone Usage

Can be invoked directly:
- "Should I use App Router or Pages Router?"
- "When to use Server Components vs Client Components?"
- "SSR vs SSG vs ISR for my e-commerce site?"
- "How to structure a large Next.js application?"

## Before Answering Next.js Architecture Questions

### Step 1: Check Project Context

```bash
# Check Next.js version
npm list next

# Check if App Router or Pages Router
ls -la app/    # App Router
ls -la pages/  # Pages Router
```

### Step 2: Fallback to Web Search

Use web search to fetch current information from:
- https://nextjs.org/docs
- https://nextjs.org/blog

## App Router vs Pages Router

### Decision Matrix

| Aspect | App Router | Pages Router |
|--------|------------|--------------|
| Server Components | ✅ Native | ❌ Not supported |
| Streaming | ✅ Yes | ❌ No |
| Layouts | ✅ Nested layouts | ⚠️ Manual |
| Data fetching | fetch() + cache | getServerSideProps |
| Stability | Stable (Next 14+) | Stable (mature) |
| Learning curve | Higher | Lower |
| Migration | N/A | Incremental possible |

### When to Use Each

| Scenario | Recommendation | Why |
|----------|----------------|-----|
| New projects | App Router | Future-proof, Server Components |
| Existing Pages Router | Stay or migrate gradually | Don't rewrite unnecessarily |
| Need streaming | App Router | Native support |
| Simple static site | Either | Pages Router simpler |
| Complex layouts | App Router | Nested layouts |
| Team new to React | Pages Router | Simpler mental model |

## Rendering Strategies

### Overview

| Strategy | Build Time | Request Time | Revalidation | Use Case |
|----------|------------|--------------|--------------|----------|
| **SSG** | ✅ | ❌ | Manual rebuild | Static content |
| **ISR** | ✅ | ⚠️ Stale | Time-based | Semi-static |
| **SSR** | ❌ | ✅ | Every request | Dynamic |
| **Dynamic** | ❌ | ✅ | Every request | Personalized |
| **PPR** | Partial | Partial | Hybrid | Best of both |

### When to Use Each

| Content Type | Strategy | Example |
|--------------|----------|---------|
| Marketing pages | SSG | Homepage, About |
| Blog posts | ISR (60s-1h) | Blog, docs |
| Product pages | ISR (60s) | E-commerce catalog |
| User dashboard | Dynamic | Personalized data |
| Search results | SSR | Dynamic queries |
| Real-time data | Client-side | Stock prices, chat |

### App Router Implementation

```typescript
// Static (SSG) - default
export default async function Page() {
  const data = await fetch('https://api.example.com/data');
  return <div>{data}</div>;
}

// ISR - revalidate every 60 seconds
export const revalidate = 60;

export default async function Page() {
  const data = await fetch('https://api.example.com/data');
  return <div>{data}</div>;
}

// Dynamic - opt out of caching
export const dynamic = 'force-dynamic';

export default async function Page() {
  const data = await fetch('https://api.example.com/data', {
    cache: 'no-store'
  });
  return <div>{data}</div>;
}
```

## Server Components vs Client Components

### Decision Guide

| Need | Server Component | Client Component |
|------|------------------|------------------|
| Fetch data | ✅ | ⚠️ Use hooks |
| Access backend | ✅ | ❌ |
| Reduce JS bundle | ✅ | ❌ |
| Interactivity | ❌ | ✅ |
| Event handlers | ❌ | ✅ |
| useState/useEffect | ❌ | ✅ |
| Browser APIs | ❌ | ✅ |

### Component Composition Pattern

```typescript
// Server Component (default in App Router)
// app/products/page.tsx
import { ProductList } from './product-list';
import { AddToCartButton } from './add-to-cart-button';

export default async function ProductsPage() {
  const products = await fetchProducts(); // Server-side fetch

  return (
    <div>
      <ProductList products={products} />
      {/* Client Component for interactivity */}
      <AddToCartButton />
    </div>
  );
}

// Client Component
// app/products/add-to-cart-button.tsx
'use client';

import { useState } from 'react';

export function AddToCartButton() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>Add ({count})</button>;
}
```

### Best Practice: Push Client Boundary Down

```typescript
// ❌ Bad: Entire page is client
'use client';
export default function Page() { ... }

// ✅ Good: Only interactive parts are client
export default function Page() {
  return (
    <ServerContent />
    <InteractiveWidget /> {/* 'use client' */}
  );
}
```

## Data Fetching Patterns

### App Router Patterns

| Pattern | When to Use | Example |
|---------|-------------|---------|
| Server Component fetch | Default for data | Page, layouts |
| Server Actions | Mutations | Forms, updates |
| Route Handlers | API endpoints | External access |
| Client fetch (SWR/TanStack) | Real-time, polling | Dashboards |

### Server Actions

```typescript
// app/actions.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title');
  await db.post.create({ data: { title } });
  revalidatePath('/posts');
}

// app/posts/new/page.tsx
import { createPost } from '../actions';

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

### Caching Strategies

| Cache | Scope | Duration | Invalidation |
|-------|-------|----------|--------------|
| Request memoization | Single request | Request lifetime | Automatic |
| Data Cache | Server | Persistent | revalidatePath/Tag |
| Full Route Cache | Server | Persistent | revalidatePath |
| Router Cache | Client | Session | refresh/revalidate |

```typescript
// Cache with tags for targeted invalidation
const posts = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] }
});

// Invalidate specific tag
import { revalidateTag } from 'next/cache';
revalidateTag('posts');
```

## Project Structure

### Recommended Structure (App Router)

```
app/
├── (marketing)/          # Route group (no URL segment)
│   ├── page.tsx          # /
│   ├── about/page.tsx    # /about
│   └── layout.tsx        # Shared marketing layout
├── (dashboard)/          # Another route group
│   ├── dashboard/
│   │   ├── page.tsx      # /dashboard
│   │   └── settings/
│   │       └── page.tsx  # /dashboard/settings
│   └── layout.tsx        # Dashboard layout with nav
├── api/                  # Route Handlers
│   └── webhooks/
│       └── route.ts
├── components/           # Shared components
│   ├── ui/              # UI primitives
│   └── features/        # Feature components
├── lib/                 # Utilities, types
│   ├── db.ts
│   └── utils.ts
└── layout.tsx           # Root layout
```

### Colocation Pattern

```
app/
└── posts/
    ├── page.tsx           # Route component
    ├── loading.tsx        # Loading UI
    ├── error.tsx          # Error boundary
    ├── actions.ts         # Server actions
    ├── components/        # Route-specific components
    │   ├── post-card.tsx
    │   └── post-list.tsx
    └── [slug]/
        └── page.tsx       # Dynamic route
```

## Deployment Options

| Platform | Best For | Features | Cost |
|----------|----------|----------|------|
| **Vercel** | Full features | Native, preview, analytics | $20+/mo |
| **Netlify** | Jamstack | Good integration | $19+/mo |
| **AWS Amplify** | AWS ecosystem | Full AWS access | Variable |
| **Cloudflare** | Edge | Fast, cheap | Low |
| **Self-hosted** | Full control | Docker/Node | Infrastructure |

### Deployment Considerations

| Feature | Vercel | Self-hosted |
|---------|--------|-------------|
| ISR | ✅ Native | ⚠️ Complex |
| Image Optimization | ✅ Built-in | ⚠️ Configure |
| Edge Functions | ✅ Native | ⚠️ Limited |
| Preview Deployments | ✅ Automatic | ❌ Manual |
| Serverless | ✅ Default | ⚠️ Configure |

## Performance Patterns

### Optimization Checklist

| Optimization | Implementation |
|--------------|----------------|
| Image optimization | Use `next/image` |
| Font optimization | Use `next/font` |
| Code splitting | Dynamic imports |
| Bundle analysis | `@next/bundle-analyzer` |
| Prefetching | Default for `<Link>` |

### Dynamic Imports

```typescript
import dynamic from 'next/dynamic';

// Load component only when needed
const HeavyChart = dynamic(() => import('./heavy-chart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false // Client-only
});
```

## Output Contract

```yaml
architecture_decision:
  vendor: "nextjs"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<routing|rendering|data|deployment>"

  options_evaluated:
    - id: "<option-id>"
      name: "<option name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        performance: "<low|medium|high>"
        complexity: "<low|medium|high>"
        flexibility: "<low|medium|high>"
        bundle_size: "<small|medium|large>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  rendering_strategy:
    default: "<SSG|ISR|SSR|Dynamic>"
    by_route:
      - route: "<route pattern>"
        strategy: "<strategy>"
        revalidate: <seconds|null>

  implementation_guidance:
    getting_started:
      - "<step>"
    file_structure:
      - "<path>: <purpose>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| 'use client' on everything | Large bundles | Push client boundary down |
| Fetching in client when server works | Waterfalls | Server Components |
| Not using ISR | Unnecessary rebuilds | Set revalidate |
| Giant page components | Hard to maintain | Extract components |
| Ignoring loading states | Bad UX | loading.tsx, Suspense |
| Over-fetching in layouts | Wasted data | Fetch where used |

## Integration Points

- Use **arch-vercel** for Vercel-specific deployment patterns
- Use **arch-supabase** or **arch-firebase** for backend
- Feeds into **implementation-plan** for project setup
