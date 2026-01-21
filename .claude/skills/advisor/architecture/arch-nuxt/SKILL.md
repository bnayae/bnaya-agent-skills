---
name: arch-nuxt
description: Evaluate Nuxt 3 architecture decisions including rendering modes (SSR, SSG, Hybrid), Nitro server, data fetching patterns, and deployment options. Use when user needs guidance on Nuxt/Vue architecture choices for TypeScript applications.
context: fork
agent: Explore
---

# Nuxt 3 Architecture Decision Evaluator

Evaluate and recommend Nuxt 3 configurations, rendering strategies, and architectural patterns for Vue.js TypeScript applications.

## Standalone Usage

Can be invoked directly:
- "Should I use Nuxt or Next.js for my Vue app?"
- "SSR vs SSG vs Hybrid in Nuxt?"
- "How to structure a large Nuxt application?"
- "Best deployment options for Nuxt 3?"

## Before Answering Nuxt Architecture Questions

### Step 1: Check Project Context

```bash
# Check Nuxt version
npm list nuxt

# Check nuxt.config.ts for configuration
cat nuxt.config.ts
```

### Step 2: Fallback to Web Search

Use web search to fetch current information from:
- https://nuxt.com/docs
- https://nuxt.com/blog

## Nuxt 3 vs Next.js

### Comparison Matrix

| Aspect | Nuxt 3 | Next.js |
|--------|--------|---------|
| Framework | Vue 3 | React |
| Language | TypeScript | TypeScript |
| Rendering | SSR/SSG/Hybrid/SPA | SSR/SSG/ISR/Dynamic |
| Data fetching | useFetch, useAsyncData | Server Components, fetch |
| Server | Nitro (universal) | Node.js, Edge |
| Auto-imports | ✅ Built-in | ❌ Manual |
| File routing | ✅ pages/ | ✅ app/ or pages/ |
| State | Pinia (recommended) | Zustand, Redux |

### When to Choose Nuxt

| Scenario | Nuxt | Next.js |
|----------|------|---------|
| Vue.js team/preference | ✅ | ❌ |
| Simpler learning curve | ✅ | ⚠️ |
| Auto-imports preferred | ✅ | ❌ |
| React ecosystem needed | ❌ | ✅ |
| Largest community | ❌ | ✅ |
| Convention over config | ✅ | ⚠️ |

## Rendering Modes

### Overview

| Mode | Build | Server | Use Case |
|------|-------|--------|----------|
| **SSR** | ❌ | ✅ On every request | Dynamic content |
| **SSG** | ✅ At build | ❌ | Static sites |
| **SPA** | ✅ Client shell | ❌ | App-like experience |
| **Hybrid** | Mixed | Mixed | Best of both |

### Configuration

```typescript
// nuxt.config.ts

// Full SSR (default)
export default defineNuxtConfig({
  ssr: true
})

// Full SSG
export default defineNuxtConfig({
  ssr: true,
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  }
})

// SPA mode
export default defineNuxtConfig({
  ssr: false
})

// Hybrid (per-route)
export default defineNuxtConfig({
  routeRules: {
    '/': { prerender: true },           // SSG
    '/blog/**': { isr: 3600 },          // ISR (1 hour)
    '/dashboard/**': { ssr: false },    // SPA
    '/api/**': { cors: true },          // API routes
  }
})
```

### Route Rules Reference

| Rule | Effect |
|------|--------|
| `prerender: true` | Generate at build time (SSG) |
| `isr: seconds` | Incremental Static Regeneration |
| `ssr: false` | Client-side only (SPA) |
| `redirect: '/new-path'` | Server redirect |
| `headers: {}` | Custom headers |
| `cors: true` | Enable CORS |
| `cache: {}` | Cache control |

## Data Fetching

### useFetch vs useAsyncData

| Feature | useFetch | useAsyncData |
|---------|----------|--------------|
| Auto URL | ✅ | ❌ |
| Any async | ❌ | ✅ |
| Caching | ✅ Built-in | ✅ With key |
| SSR support | ✅ | ✅ |

### useFetch Pattern

```typescript
// pages/posts/[id].vue
<script setup lang="ts">
const route = useRoute()

const { data: post, pending, error } = await useFetch(
  `/api/posts/${route.params.id}`,
  {
    // Options
    key: `post-${route.params.id}`,  // Cache key
    transform: (data) => data.post,   // Transform response
    watch: [() => route.params.id],   // Re-fetch on change
  }
)
</script>

<template>
  <div v-if="pending">Loading...</div>
  <div v-else-if="error">Error: {{ error.message }}</div>
  <article v-else>
    <h1>{{ post.title }}</h1>
    <p>{{ post.content }}</p>
  </article>
</template>
```

### useAsyncData Pattern

```typescript
<script setup lang="ts">
const { data: analytics } = await useAsyncData(
  'dashboard-analytics',
  async () => {
    const [users, revenue, orders] = await Promise.all([
      $fetch('/api/analytics/users'),
      $fetch('/api/analytics/revenue'),
      $fetch('/api/analytics/orders'),
    ])
    return { users, revenue, orders }
  }
)
</script>
```

### Lazy Loading

```typescript
// Don't block navigation
const { data: comments } = await useLazyFetch('/api/comments')

// Or with useAsyncData
const { data } = await useLazyAsyncData('key', () => fetchData())
```

## Nitro Server

### API Routes

```typescript
// server/api/posts/index.get.ts
export default defineEventHandler(async (event) => {
  const posts = await db.post.findMany()
  return posts
})

// server/api/posts/index.post.ts
export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  const post = await db.post.create({ data: body })
  return post
})

// server/api/posts/[id].get.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  const post = await db.post.findUnique({ where: { id } })
  if (!post) {
    throw createError({ statusCode: 404, message: 'Post not found' })
  }
  return post
})
```

### Server Middleware

```typescript
// server/middleware/auth.ts
export default defineEventHandler((event) => {
  const token = getCookie(event, 'auth-token')
  if (event.path.startsWith('/api/protected') && !token) {
    throw createError({ statusCode: 401, message: 'Unauthorized' })
  }
})
```

### Server Utilities

```typescript
// server/utils/db.ts
import { PrismaClient } from '@prisma/client'

let prisma: PrismaClient

export function useDb() {
  if (!prisma) {
    prisma = new PrismaClient()
  }
  return prisma
}
```

## Project Structure

### Recommended Structure

```
├── app.vue                # Root component
├── nuxt.config.ts         # Nuxt configuration
├── pages/                 # File-based routing
│   ├── index.vue          # /
│   ├── about.vue          # /about
│   └── posts/
│       ├── index.vue      # /posts
│       └── [id].vue       # /posts/:id
├── components/            # Auto-imported components
│   ├── ui/
│   │   ├── Button.vue
│   │   └── Card.vue
│   └── features/
│       └── PostCard.vue
├── composables/           # Auto-imported composables
│   ├── useAuth.ts
│   └── usePosts.ts
├── layouts/               # Page layouts
│   ├── default.vue
│   └── dashboard.vue
├── server/                # Nitro server
│   ├── api/               # API routes
│   ├── middleware/        # Server middleware
│   └── utils/             # Server utilities
├── stores/                # Pinia stores
│   └── auth.ts
├── types/                 # TypeScript types
└── utils/                 # Auto-imported utilities
```

### Auto-imports

```typescript
// These are auto-imported - no import needed!

// Composables
const { data } = await useFetch('/api/data')
const route = useRoute()
const router = useRouter()

// Components (from components/)
<UiButton>Click me</UiButton>

// Utils (from utils/)
const formatted = formatDate(date)

// Vue APIs
const count = ref(0)
const doubled = computed(() => count.value * 2)
```

## State Management with Pinia

### Store Setup

```typescript
// stores/auth.ts
export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => !!user.value)

  async function login(credentials: Credentials) {
    const data = await $fetch('/api/auth/login', {
      method: 'POST',
      body: credentials
    })
    user.value = data.user
  }

  function logout() {
    user.value = null
  }

  return { user, isAuthenticated, login, logout }
})
```

### Using Stores

```typescript
<script setup lang="ts">
const auth = useAuthStore()

// Access state
console.log(auth.user)

// Call actions
await auth.login({ email, password })
</script>
```

## Deployment Options

| Platform | Preset | Best For |
|----------|--------|----------|
| **Vercel** | `vercel` | Easy deployment |
| **Netlify** | `netlify` | Netlify users |
| **Cloudflare Pages** | `cloudflare-pages` | Edge, cheap |
| **AWS Lambda** | `aws-lambda` | AWS ecosystem |
| **Node.js** | `node-server` | Self-hosted |
| **Static** | `static` | SSG only |

### Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'vercel' // or 'netlify', 'cloudflare-pages', etc.
  }
})
```

## Modules Ecosystem

### Essential Modules

| Module | Purpose |
|--------|---------|
| `@nuxtjs/tailwindcss` | Tailwind CSS |
| `@pinia/nuxt` | State management |
| `@vueuse/nuxt` | Vue utilities |
| `@nuxt/image` | Image optimization |
| `@nuxtjs/i18n` | Internationalization |
| `@sidebase/nuxt-auth` | Authentication |

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/tailwindcss',
    '@pinia/nuxt',
    '@vueuse/nuxt',
  ]
})
```

## Output Contract

```yaml
architecture_decision:
  vendor: "nuxt"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<rendering|data|server|deployment>"

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
        seo: "<poor|good|excellent>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  rendering_strategy:
    default: "<SSR|SSG|SPA|Hybrid>"
    route_rules:
      - pattern: "<route pattern>"
        rule: "<rule>"

  implementation_guidance:
    getting_started:
      - "<step>"
    modules:
      - "<module>: <purpose>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| useFetch in onMounted | SSR mismatch | Use in setup |
| Direct API calls in components | No caching | useFetch/useAsyncData |
| Large pages directory | Hard to navigate | Group by feature |
| Ignoring hydration | Mismatches | Check SSR compatibility |
| Not using auto-imports | Manual imports | Follow conventions |

## Integration Points

- Use **arch-supabase** or **arch-firebase** for backend
- Consider **Vercel** or **Cloudflare** for deployment
- Feeds into **implementation-plan** for project setup
