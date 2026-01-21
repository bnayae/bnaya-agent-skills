---
name: arch-vercel
description: Evaluate Vercel architecture decisions including Edge vs Serverless functions, caching strategies, platform integrations (Postgres, KV, Blob), deployment patterns, and preview deployments. Use when user needs guidance on Vercel platform architecture choices.
context: fork
agent: Explore
---

# Vercel Platform Architecture Decision Evaluator

Evaluate and recommend Vercel platform configurations, function types, caching strategies, and deployment patterns for TypeScript applications.

## Standalone Usage

Can be invoked directly:
- "Should I use Edge or Serverless functions?"
- "How to optimize caching on Vercel?"
- "Vercel Postgres vs external database?"
- "Best practices for preview deployments?"

## Before Answering Vercel Architecture Questions

### Step 1: Check Available Tools

**Vercel CLI**:
```bash
# Check Vercel CLI
vercel --version

# Link project
vercel link

# Environment variables
vercel env ls
```

### Step 2: Fallback to Web Search

Use web search to fetch current information from:
- https://vercel.com/docs
- https://vercel.com/blog

## Edge vs Serverless Functions

### Comparison

| Aspect | Edge Functions | Serverless Functions |
|--------|----------------|---------------------|
| Runtime | V8 (limited Node APIs) | Full Node.js |
| Cold start | ~0ms | 250ms-1s |
| Location | Global (all regions) | Single region |
| Max duration | 30s (Hobby), 5min (Pro) | 10s-900s |
| Max size | 1MB | 50MB (compressed) |
| Streaming | ✅ Yes | ✅ Yes |
| Node.js APIs | Limited | Full |
| npm packages | Limited | Full |

### When to Use Each

| Use Case | Edge | Serverless |
|----------|------|------------|
| Auth/middleware | ✅ | ❌ |
| A/B testing | ✅ | ❌ |
| Geolocation routing | ✅ | ❌ |
| API with npm packages | ❌ | ✅ |
| Database connections | ⚠️ Limited | ✅ |
| Heavy computation | ❌ | ✅ |
| File processing | ❌ | ✅ |
| Low latency globally | ✅ | ❌ |

### Edge Function Example

```typescript
// app/api/geo/route.ts
import { NextRequest, NextResponse } from 'next/server';

export const runtime = 'edge';

export async function GET(request: NextRequest) {
  const country = request.geo?.country || 'Unknown';
  const city = request.geo?.city || 'Unknown';

  return NextResponse.json({
    country,
    city,
    message: `Hello from ${city}, ${country}!`
  });
}
```

### Edge Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Auth check
  const token = request.cookies.get('session');
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // A/B testing
  const bucket = request.cookies.get('ab-bucket')?.value ||
    (Math.random() > 0.5 ? 'a' : 'b');

  const response = NextResponse.next();
  response.cookies.set('ab-bucket', bucket);

  return response;
}

export const config = {
  matcher: ['/dashboard/:path*', '/experiments/:path*']
};
```

### Serverless Function Example

```typescript
// app/api/process/route.ts
import { NextRequest, NextResponse } from 'next/server';
import sharp from 'sharp'; // Full npm package support

export async function POST(request: NextRequest) {
  const formData = await request.formData();
  const file = formData.get('image') as File;

  const buffer = Buffer.from(await file.arrayBuffer());
  const optimized = await sharp(buffer)
    .resize(800, 600)
    .webp()
    .toBuffer();

  return new NextResponse(optimized, {
    headers: { 'Content-Type': 'image/webp' }
  });
}
```

## Caching Strategies

### Cache Layers

| Layer | Scope | Control |
|-------|-------|---------|
| **CDN Cache** | Global edge | Cache-Control headers |
| **Data Cache** | Server | fetch options, unstable_cache |
| **Full Route Cache** | Server | Route segment config |
| **Router Cache** | Client | Navigation |

### Cache-Control Headers

```typescript
// app/api/data/route.ts
export async function GET() {
  const data = await fetchData();

  return NextResponse.json(data, {
    headers: {
      // Cache for 1 hour, stale-while-revalidate for 1 day
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400',
    },
  });
}
```

### Data Cache with Tags

```typescript
// Fetch with cache tag
const posts = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'], revalidate: 3600 }
});

// Revalidate by tag
import { revalidateTag } from 'next/cache';

export async function POST() {
  await createPost();
  revalidateTag('posts'); // Invalidate all caches with this tag
  return NextResponse.json({ success: true });
}
```

### ISR (Incremental Static Regeneration)

```typescript
// app/posts/[slug]/page.tsx

// Generate static pages at build
export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((post) => ({ slug: post.slug }));
}

// Revalidate every 60 seconds
export const revalidate = 60;

export default async function Post({ params }) {
  const post = await getPost(params.slug);
  return <article>{post.content}</article>;
}
```

### On-Demand Revalidation

```typescript
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request: NextRequest) {
  const { path, tag, secret } = await request.json();

  // Verify secret
  if (secret !== process.env.REVALIDATION_SECRET) {
    return NextResponse.json({ error: 'Invalid secret' }, { status: 401 });
  }

  if (path) {
    revalidatePath(path);
  }
  if (tag) {
    revalidateTag(tag);
  }

  return NextResponse.json({ revalidated: true });
}
```

## Vercel Storage

### Vercel Postgres

```typescript
import { sql } from '@vercel/postgres';

// Query
const { rows } = await sql`SELECT * FROM users WHERE id = ${userId}`;

// With connection pooling (recommended)
import { createPool } from '@vercel/postgres';

const pool = createPool();
const { rows } = await pool.query('SELECT * FROM users WHERE id = $1', [userId]);
```

### Vercel KV (Redis)

```typescript
import { kv } from '@vercel/kv';

// Set value
await kv.set('user:123', { name: 'Alice' });

// Get value
const user = await kv.get('user:123');

// With expiration
await kv.set('session:abc', data, { ex: 3600 }); // 1 hour

// Increment
await kv.incr('page:views');

// Hash operations
await kv.hset('user:123', { name: 'Alice', email: 'alice@example.com' });
const name = await kv.hget('user:123', 'name');
```

### Vercel Blob

```typescript
import { put, del, list } from '@vercel/blob';

// Upload
const blob = await put('images/avatar.png', file, {
  access: 'public',
  contentType: 'image/png'
});
console.log(blob.url); // CDN URL

// Delete
await del('images/avatar.png');

// List
const { blobs } = await list({ prefix: 'images/' });
```

### Edge Config

```typescript
import { get, getAll, has } from '@vercel/edge-config';

// Feature flags
const isFeatureEnabled = await get('feature-new-ui');

// Get all config
const config = await getAll();

// Check existence
const exists = await has('maintenance-mode');
```

## Deployment Patterns

### Preview Deployments

```yaml
# vercel.json
{
  "git": {
    "deploymentEnabled": {
      "main": true,
      "staging": true,
      "feature/*": true
    }
  }
}
```

### Environment Variables

| Type | Scope | Use Case |
|------|-------|----------|
| Production | main branch only | Secrets, prod config |
| Preview | All preview branches | Test config |
| Development | Local only | Dev secrets |

### Protected Deployments

```yaml
# vercel.json
{
  "passwordProtection": true,
  "trustedIps": {
    "addresses": ["1.2.3.4/32"],
    "protectionMode": "only"
  }
}
```

### Monorepo Setup

```yaml
# vercel.json (root)
{
  "projects": [
    { "name": "web", "root": "apps/web" },
    { "name": "api", "root": "apps/api" }
  ]
}

# With Turborepo
{
  "buildCommand": "turbo run build --filter=web",
  "installCommand": "pnpm install",
  "framework": "nextjs"
}
```

## Performance Optimization

### Skew Protection

```typescript
// Prevents version mismatch between static and dynamic content
// Enabled by default on Pro/Enterprise
```

### Speed Insights

```typescript
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### Analytics

```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

## Integration Patterns

### External Database

```typescript
// Use connection pooling for serverless
// PlanetScale
import { connect } from '@planetscale/database';
const conn = connect({ url: process.env.DATABASE_URL });

// Neon
import { Pool } from '@neondatabase/serverless';
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// Supabase
import { createClient } from '@supabase/supabase-js';
const supabase = createClient(url, key);
```

### Auth Providers

| Provider | Integration |
|----------|-------------|
| NextAuth.js | Native support |
| Clerk | Middleware, Edge |
| Auth0 | SDK, API routes |
| Supabase Auth | SDK |

## Output Contract

```yaml
architecture_decision:
  vendor: "vercel"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<functions|caching|storage|deployment>"

  options_evaluated:
    - id: "<option-id>"
      name: "<option name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        latency: "<sub-ms|low|medium|high>"
        global: "<yes|no>"
        cost: "<low|medium|high>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  caching_strategy:
    cdn: "<cache-control header>"
    data_cache: "<revalidate value>"
    tags: []

  implementation_guidance:
    getting_started:
      - "<step>"
    vercel_json: "<config snippet>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Edge for heavy compute | Timeout, limits | Serverless |
| No caching strategy | Slow, expensive | Set Cache-Control |
| Serverless for auth | Latency | Edge middleware |
| Direct DB from Edge | Connection limits | Use Vercel Postgres or API |
| Ignoring preview URLs | Security leaks | Password protect |

## Integration Points

- Use **arch-nextjs** for Next.js-specific patterns
- Use **cost-vercel** for cost estimation
- Feeds into **implementation-plan** for deployment
