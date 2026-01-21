---
name: arch-remix
description: Evaluate Remix architecture decisions including loaders/actions, nested routes, form handling, deployment options, and comparison with Next.js. Use when user needs guidance on Remix architecture choices for TypeScript applications.
color: cyan
context: fork
agent: Explore
---

# Remix Architecture Decision Evaluator

Evaluate and recommend Remix configurations, data loading patterns, and architectural decisions for web standards-focused TypeScript applications.

## Standalone Usage

Can be invoked directly:
- "Should I use Remix or Next.js for my app?"
- "How do loaders and actions work in Remix?"
- "Best deployment option for Remix?"
- "How to handle forms in Remix?"

## Before Answering Remix Architecture Questions

### Step 1: Check Project Context

```bash
# Check Remix version
npm list @remix-run/react

# Check which adapter
npm list @remix-run/node @remix-run/cloudflare @remix-run/deno
```

### Step 2: Fallback to Web Search

Use web search to fetch current information from:
- https://remix.run/docs
- https://remix.run/blog

## Remix vs Next.js Decision

### Comparison Matrix

| Aspect | Remix | Next.js |
|--------|-------|---------|
| Philosophy | Web standards, progressive enhancement | React-first, many rendering modes |
| Data loading | Loaders (server) | Server Components, getServerSideProps |
| Mutations | Actions (forms) | Server Actions, API routes |
| Routing | Nested routes with parallel loading | File-based, layouts |
| Forms | Native HTML forms | Client-side or Server Actions |
| Streaming | Yes | Yes |
| Static generation | Limited | Full support (SSG, ISR) |
| Edge deployment | Excellent | Good (Vercel) |

### When to Choose Remix

| Scenario | Remix | Next.js |
|----------|-------|---------|
| Form-heavy applications | ✅ | ⚠️ |
| Progressive enhancement | ✅ | ❌ |
| Works without JS | ✅ | ❌ |
| Static site generation | ❌ | ✅ |
| Large ecosystem needed | ⚠️ | ✅ |
| Edge-first deployment | ✅ | ⚠️ |
| Simpler mental model | ✅ | ❌ |

## Data Loading with Loaders

### Basic Loader Pattern

```typescript
// app/routes/posts.tsx
import type { LoaderFunctionArgs } from '@remix-run/node';
import { json } from '@remix-run/node';
import { useLoaderData } from '@remix-run/react';

export async function loader({ request }: LoaderFunctionArgs) {
  const posts = await db.post.findMany();
  return json({ posts });
}

export default function Posts() {
  const { posts } = useLoaderData<typeof loader>();
  return (
    <ul>
      {posts.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

### Parallel Data Loading with Nested Routes

```typescript
// app/routes/dashboard.tsx - Parent route
export async function loader() {
  const user = await getUser();
  return json({ user });
}

// app/routes/dashboard.analytics.tsx - Child route
export async function loader() {
  const analytics = await getAnalytics(); // Loads in parallel!
  return json({ analytics });
}
```

### Loader Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| Simple fetch | Basic data | `await db.posts.findMany()` |
| With params | Dynamic routes | `params.postId` |
| With search params | Filtering | `new URL(request.url).searchParams` |
| With headers | Auth, cookies | `request.headers.get('Cookie')` |
| Throw responses | Errors, redirects | `throw redirect('/login')` |

## Mutations with Actions

### Basic Action Pattern

```typescript
// app/routes/posts.new.tsx
import type { ActionFunctionArgs } from '@remix-run/node';
import { redirect } from '@remix-run/node';
import { Form } from '@remix-run/react';

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData();
  const title = formData.get('title');

  await db.post.create({ data: { title } });
  return redirect('/posts');
}

export default function NewPost() {
  return (
    <Form method="post">
      <input name="title" required />
      <button type="submit">Create</button>
    </Form>
  );
}
```

### Multiple Actions Pattern

```typescript
export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData();
  const intent = formData.get('intent');

  switch (intent) {
    case 'create':
      return createPost(formData);
    case 'delete':
      return deletePost(formData);
    default:
      throw new Error('Unknown intent');
  }
}

export default function Posts() {
  return (
    <>
      <Form method="post">
        <input name="title" />
        <button name="intent" value="create">Create</button>
      </Form>

      <Form method="post">
        <input type="hidden" name="id" value={postId} />
        <button name="intent" value="delete">Delete</button>
      </Form>
    </>
  );
}
```

## Form Patterns

### Progressive Enhancement

```typescript
// Works without JavaScript, enhanced with JS
import { Form, useNavigation } from '@remix-run/react';

export default function ContactForm() {
  const navigation = useNavigation();
  const isSubmitting = navigation.state === 'submitting';

  return (
    <Form method="post">
      <input name="email" type="email" required />
      <button disabled={isSubmitting}>
        {isSubmitting ? 'Sending...' : 'Send'}
      </button>
    </Form>
  );
}
```

### Optimistic UI

```typescript
import { useFetcher } from '@remix-run/react';

export function LikeButton({ postId, liked }: Props) {
  const fetcher = useFetcher();

  // Optimistic: show new state immediately
  const optimisticLiked = fetcher.formData
    ? fetcher.formData.get('liked') === 'true'
    : liked;

  return (
    <fetcher.Form method="post" action="/api/like">
      <input type="hidden" name="postId" value={postId} />
      <button name="liked" value={String(!optimisticLiked)}>
        {optimisticLiked ? '❤️' : '🤍'}
      </button>
    </fetcher.Form>
  );
}
```

### Form Validation

```typescript
import { json } from '@remix-run/node';
import { useActionData } from '@remix-run/react';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData();
  const result = schema.safeParse(Object.fromEntries(formData));

  if (!result.success) {
    return json({ errors: result.error.flatten() }, { status: 400 });
  }

  // Process valid data
  return redirect('/dashboard');
}

export default function Signup() {
  const actionData = useActionData<typeof action>();

  return (
    <Form method="post">
      <input name="email" />
      {actionData?.errors?.fieldErrors?.email && (
        <span>{actionData.errors.fieldErrors.email}</span>
      )}
      {/* ... */}
    </Form>
  );
}
```

## Routing Patterns

### File-based Routing

```
app/routes/
├── _index.tsx           # /
├── about.tsx            # /about
├── posts._index.tsx     # /posts
├── posts.$postId.tsx    # /posts/:postId
├── posts.$postId_.edit.tsx  # /posts/:postId/edit (no layout)
├── dashboard.tsx        # /dashboard (layout)
├── dashboard._index.tsx # /dashboard (index)
├── dashboard.settings.tsx   # /dashboard/settings
└── $.tsx                # Catch-all (404)
```

### Route Module API

```typescript
// Every route can export these
export const loader = async () => {};      // GET data
export const action = async () => {};      // POST/PUT/DELETE
export const headers = () => {};           // HTTP headers
export const meta = () => {};              // <head> metadata
export const links = () => {};             // <link> elements
export const handle = {};                  // Custom route data
export default function Component() {}      // UI
export function ErrorBoundary() {}         // Error UI
```

## Project Structure

### Recommended Structure

```
app/
├── routes/              # All routes
│   ├── _index.tsx
│   ├── dashboard.tsx
│   └── dashboard.settings.tsx
├── components/          # Shared UI components
│   ├── ui/
│   └── forms/
├── lib/                 # Utilities
│   ├── db.server.ts     # .server = server-only
│   ├── auth.server.ts
│   └── utils.ts
├── models/              # Data models/queries
│   ├── user.server.ts
│   └── post.server.ts
├── services/            # External services
├── styles/              # CSS
├── entry.client.tsx     # Client entry
├── entry.server.tsx     # Server entry
└── root.tsx             # Root layout
```

### Server-only Code

```typescript
// lib/db.server.ts - .server.ts ensures server-only
import { PrismaClient } from '@prisma/client';

export const db = new PrismaClient();

// This file is NEVER bundled for the client
```

## Deployment Options

| Platform | Adapter | Best For |
|----------|---------|----------|
| **Vercel** | `@vercel/remix` | Easy deployment |
| **Fly.io** | `@remix-run/node` | Global edge, Docker |
| **Cloudflare Pages** | `@remix-run/cloudflare` | Edge, cheap |
| **Cloudflare Workers** | `@remix-run/cloudflare` | Full edge |
| **AWS Lambda** | `@remix-run/architect` | AWS ecosystem |
| **Netlify** | `@netlify/remix-adapter` | Netlify users |
| **Node.js** | `@remix-run/node` | Self-hosted |

### Deployment Comparison

| Platform | Cold Start | Global | Cost | Complexity |
|----------|------------|--------|------|------------|
| Fly.io | Low | ✅ | $$ | Low |
| Cloudflare | None (edge) | ✅ | $ | Low |
| Vercel | Low | ✅ | $$$ | Low |
| AWS Lambda | Medium | ⚠️ | $$ | High |
| Self-hosted | None | ❌ | $ | High |

## Error Handling

### Error Boundaries

```typescript
// app/routes/posts.$postId.tsx
import { isRouteErrorResponse, useRouteError } from '@remix-run/react';

export function ErrorBoundary() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>{error.status}</h1>
        <p>{error.statusText}</p>
      </div>
    );
  }

  return <div>Something went wrong</div>;
}
```

### Throwing Responses

```typescript
export async function loader({ params }: LoaderFunctionArgs) {
  const post = await db.post.findUnique({ where: { id: params.postId } });

  if (!post) {
    throw new Response('Not Found', { status: 404 });
  }

  return json({ post });
}
```

## Output Contract

```yaml
architecture_decision:
  vendor: "remix"
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
        progressive_enhancement: "<yes|no>"
        complexity: "<low|medium|high>"
        performance: "<low|medium|high>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  route_structure:
    - path: "<route path>"
      file: "<file path>"
      loader: "<yes|no>"
      action: "<yes|no>"

  implementation_guidance:
    getting_started:
      - "<step>"
    deployment:
      platform: "<platform>"
      adapter: "<adapter package>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Client-side fetching for initial data | Waterfalls | Use loaders |
| useEffect for form submission | No progressive enhancement | Use actions |
| Giant route files | Hard to maintain | Extract to models/services |
| Ignoring pending states | Poor UX | useNavigation |
| Not using nested routes | Missed parallel loading | Nest routes properly |

## Integration Points

- Use **arch-supabase** or **arch-firebase** for backend
- Consider **Fly.io** or **Cloudflare** for deployment
- Feeds into **implementation-plan** for project setup
