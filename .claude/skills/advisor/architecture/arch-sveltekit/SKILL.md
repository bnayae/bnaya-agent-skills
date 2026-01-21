---
name: arch-sveltekit
description: Evaluate SvelteKit architecture decisions including load functions, form actions, rendering strategies, adapters, and deployment options. Use when user needs guidance on SvelteKit/Svelte architecture choices for TypeScript applications.
context: fork
agent: Explore
---

# SvelteKit Architecture Decision Evaluator

Evaluate and recommend SvelteKit configurations, data loading patterns, and architectural decisions for Svelte TypeScript applications.

## Standalone Usage

Can be invoked directly:
- "Should I use SvelteKit or Next.js?"
- "How do load functions work in SvelteKit?"
- "Best deployment option for SvelteKit?"
- "How to handle forms in SvelteKit?"

## Before Answering SvelteKit Architecture Questions

### Step 1: Check Project Context

```bash
# Check SvelteKit version
npm list @sveltejs/kit

# Check adapter
npm list @sveltejs/adapter-*
```

### Step 2: Fallback to Web Search

Use web search to fetch current information from:
- https://kit.svelte.dev/docs
- https://svelte.dev/docs

## SvelteKit vs Next.js vs Remix

### Comparison Matrix

| Aspect | SvelteKit | Next.js | Remix |
|--------|-----------|---------|-------|
| Framework | Svelte | React | React |
| Bundle size | Smallest | Medium | Medium |
| Reactivity | Compile-time | Runtime | Runtime |
| Data loading | load functions | Server Components | loaders |
| Forms | Form actions | Server Actions | actions |
| Learning curve | Low | Medium-High | Medium |

### When to Choose SvelteKit

| Scenario | SvelteKit | Next.js/Remix |
|----------|-----------|---------------|
| Smallest bundle size | ✅ | ❌ |
| Simpler reactivity model | ✅ | ❌ |
| Less boilerplate | ✅ | ❌ |
| Largest ecosystem | ❌ | ✅ |
| Most job opportunities | ❌ | ✅ |
| Team knows React | ❌ | ✅ |

## Load Functions

### Page Load

```typescript
// src/routes/posts/+page.server.ts
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ params, fetch }) => {
  const posts = await fetch('/api/posts').then(r => r.json());
  return { posts };
};

// src/routes/posts/+page.svelte
<script lang="ts">
  import type { PageData } from './$types';
  export let data: PageData;
</script>

<ul>
  {#each data.posts as post}
    <li>{post.title}</li>
  {/each}
</ul>
```

### Server vs Universal Load

| Type | File | Runs On | Use Case |
|------|------|---------|----------|
| Server | `+page.server.ts` | Server only | DB access, secrets |
| Universal | `+page.ts` | Server & client | Public APIs |

```typescript
// Server load - access database directly
// +page.server.ts
export const load = async () => {
  const posts = await db.post.findMany();
  return { posts };
};

// Universal load - runs on both
// +page.ts
export const load = async ({ fetch }) => {
  const posts = await fetch('/api/posts').then(r => r.json());
  return { posts };
};
```

### Layout Load

```typescript
// src/routes/+layout.server.ts
export const load = async ({ locals }) => {
  return {
    user: locals.user
  };
};

// Accessible in all child routes
// +page.svelte
<script>
  export let data;
  // data.user available from parent layout
</script>
```

### Parallel Loading

```typescript
// Parent and child load functions run in parallel!

// src/routes/dashboard/+layout.server.ts
export const load = async () => {
  const user = await getUser(); // Starts immediately
  return { user };
};

// src/routes/dashboard/analytics/+page.server.ts
export const load = async () => {
  const analytics = await getAnalytics(); // Also starts immediately!
  return { analytics };
};
```

## Form Actions

### Basic Action

```typescript
// src/routes/posts/new/+page.server.ts
import type { Actions } from './$types';
import { fail, redirect } from '@sveltejs/kit';

export const actions: Actions = {
  default: async ({ request }) => {
    const data = await request.formData();
    const title = data.get('title');

    if (!title) {
      return fail(400, { error: 'Title required' });
    }

    await db.post.create({ data: { title: title.toString() } });
    throw redirect(303, '/posts');
  }
};
```

```svelte
<!-- src/routes/posts/new/+page.svelte -->
<script lang="ts">
  import type { ActionData } from './$types';
  export let form: ActionData;
</script>

<form method="POST">
  <input name="title" />
  {#if form?.error}
    <p class="error">{form.error}</p>
  {/if}
  <button>Create</button>
</form>
```

### Multiple Actions

```typescript
// +page.server.ts
export const actions: Actions = {
  create: async ({ request }) => {
    // Handle create
  },
  delete: async ({ request }) => {
    // Handle delete
  }
};
```

```svelte
<!-- Use formaction to specify which action -->
<form method="POST" action="?/create">
  <input name="title" />
  <button>Create</button>
</form>

<form method="POST" action="?/delete">
  <input type="hidden" name="id" value={post.id} />
  <button>Delete</button>
</form>
```

### Progressive Enhancement

```svelte
<script>
  import { enhance } from '$app/forms';
</script>

<!-- Works without JS, enhanced with JS -->
<form method="POST" use:enhance>
  <input name="title" />
  <button>Create</button>
</form>

<!-- Custom enhance behavior -->
<form method="POST" use:enhance={({ formData, cancel }) => {
  // Before submit
  return async ({ result, update }) => {
    // After submit
    if (result.type === 'success') {
      // Custom handling
    }
    await update(); // Apply default behavior
  };
}}>
```

## Rendering Strategies

### Configuration

```typescript
// +page.server.ts or +page.ts
export const prerender = true;  // SSG
export const ssr = false;       // SPA (client-only)
export const csr = false;       // No client JS
```

### Per-Route Options

| Option | Effect |
|--------|--------|
| `prerender = true` | Generate at build time |
| `prerender = 'auto'` | Prerender if no dynamic data |
| `ssr = false` | Client-side only |
| `csr = false` | No JavaScript |

### Prerendering

```typescript
// svelte.config.js
const config = {
  kit: {
    prerender: {
      entries: ['/', '/about', '/posts'],
      crawl: true
    }
  }
};

// Or per-page
// +page.ts
export const prerender = true;

// Dynamic prerender entries
// +page.server.ts
export const entries = async () => {
  const posts = await db.post.findMany();
  return posts.map(post => ({ slug: post.slug }));
};
```

## Project Structure

### Recommended Structure

```
src/
├── routes/               # File-based routing
│   ├── +layout.svelte    # Root layout
│   ├── +layout.server.ts # Root layout data
│   ├── +page.svelte      # Home page
│   ├── +error.svelte     # Error page
│   ├── about/
│   │   └── +page.svelte  # /about
│   ├── posts/
│   │   ├── +page.svelte  # /posts
│   │   ├── +page.server.ts
│   │   ├── [slug]/
│   │   │   ├── +page.svelte    # /posts/:slug
│   │   │   └── +page.server.ts
│   │   └── new/
│   │       ├── +page.svelte    # /posts/new
│   │       └── +page.server.ts
│   └── api/
│       └── posts/
│           └── +server.ts # API endpoint
├── lib/                  # Shared code ($lib alias)
│   ├── components/       # Reusable components
│   ├── server/           # Server-only ($lib/server)
│   │   └── db.ts
│   └── utils.ts
├── app.html              # HTML template
├── app.d.ts              # Type declarations
└── hooks.server.ts       # Server hooks
```

### API Routes

```typescript
// src/routes/api/posts/+server.ts
import type { RequestHandler } from './$types';
import { json } from '@sveltejs/kit';

export const GET: RequestHandler = async () => {
  const posts = await db.post.findMany();
  return json(posts);
};

export const POST: RequestHandler = async ({ request }) => {
  const body = await request.json();
  const post = await db.post.create({ data: body });
  return json(post, { status: 201 });
};
```

## Hooks

### Server Hooks

```typescript
// src/hooks.server.ts
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
  // Before route handlers
  const session = await getSession(event.cookies);
  event.locals.user = session?.user;

  // Continue to route
  const response = await resolve(event);

  // After route handlers
  return response;
};
```

### Auth Example

```typescript
// src/hooks.server.ts
export const handle: Handle = async ({ event, resolve }) => {
  const token = event.cookies.get('session');

  if (token) {
    const user = await verifyToken(token);
    event.locals.user = user;
  }

  // Protect routes
  if (event.url.pathname.startsWith('/dashboard') && !event.locals.user) {
    throw redirect(303, '/login');
  }

  return resolve(event);
};
```

## Adapters

| Adapter | Use Case | Command |
|---------|----------|---------|
| `adapter-auto` | Auto-detect platform | Default |
| `adapter-vercel` | Vercel | `npm i -D @sveltejs/adapter-vercel` |
| `adapter-netlify` | Netlify | `npm i -D @sveltejs/adapter-netlify` |
| `adapter-cloudflare` | Cloudflare Pages | `npm i -D @sveltejs/adapter-cloudflare` |
| `adapter-node` | Node.js server | `npm i -D @sveltejs/adapter-node` |
| `adapter-static` | Static site | `npm i -D @sveltejs/adapter-static` |

### Configuration

```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-vercel';

const config = {
  kit: {
    adapter: adapter({
      runtime: 'edge' // or 'nodejs18.x'
    })
  }
};

export default config;
```

## State Management

### Svelte Stores

```typescript
// src/lib/stores/auth.ts
import { writable, derived } from 'svelte/store';

export const user = writable<User | null>(null);
export const isAuthenticated = derived(user, $user => !!$user);
```

```svelte
<script>
  import { user, isAuthenticated } from '$lib/stores/auth';
</script>

{#if $isAuthenticated}
  <p>Welcome, {$user.name}</p>
{/if}
```

### Context API

```svelte
<!-- Parent.svelte -->
<script>
  import { setContext } from 'svelte';
  setContext('theme', { color: 'dark' });
</script>

<!-- Child.svelte -->
<script>
  import { getContext } from 'svelte';
  const theme = getContext('theme');
</script>
```

## Output Contract

```yaml
architecture_decision:
  vendor: "sveltekit"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<routing|data|forms|deployment>"

  options_evaluated:
    - id: "<option-id>"
      name: "<option name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        bundle_size: "<small|medium|large>"
        complexity: "<low|medium|high>"
        performance: "<low|medium|high>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  rendering_config:
    default: "<SSR|SSG|SPA>"
    routes:
      - path: "<route>"
        prerender: <boolean>
        ssr: <boolean>

  implementation_guidance:
    getting_started:
      - "<step>"
    adapter: "<adapter>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Fetch in onMount | SSR mismatch | Use load functions |
| Direct DB in +page.ts | Exposes to client | Use +page.server.ts |
| Not using enhance | No progressive enhancement | Add use:enhance |
| Huge +page.svelte | Hard to maintain | Extract components |
| Ignoring types | Type errors | Use $types imports |

## Integration Points

- Use **arch-supabase** or **arch-firebase** for backend
- Consider **Vercel** or **Cloudflare** for deployment
- Feeds into **implementation-plan** for project setup
