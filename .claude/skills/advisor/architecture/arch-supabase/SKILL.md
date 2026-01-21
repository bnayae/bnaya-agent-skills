---
name: arch-supabase
description: Evaluate Supabase architecture decisions including auth patterns, Edge Functions vs database functions, Row Level Security (RLS), Realtime subscriptions, and database design. Use when user needs guidance on Supabase architecture choices.
color: orange
context: fork
agent: Explore
---

# Supabase Architecture Decision Evaluator

Evaluate and recommend Supabase configurations, authentication patterns, and database design for TypeScript applications.

## Standalone Usage

Can be invoked directly:
- "How should I design RLS policies in Supabase?"
- "Edge Functions vs Database functions?"
- "Best auth pattern for my SaaS app?"
- "How to structure multi-tenant Supabase?"

## Before Answering Supabase Architecture Questions

### Step 1: Check Available Tools

**Supabase CLI**:
```bash
# Check Supabase CLI
supabase --version

# Local development
supabase start
supabase status
```

### Step 2: Fallback to Web Search

Use web search to fetch current information from:
- https://supabase.com/docs
- https://supabase.com/blog

## Authentication Patterns

### Auth Methods

| Method | Use Case | Complexity |
|--------|----------|------------|
| Email/Password | Traditional apps | Low |
| Magic Link | Passwordless | Low |
| OAuth (Google, GitHub, etc.) | Social login | Medium |
| Phone/SMS | Mobile apps | Medium |
| Anonymous | Guest users | Low |
| SSO/SAML | Enterprise | High |

### Auth Configuration

```typescript
// Initialize Supabase client
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
)

// Email signup
await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password'
})

// OAuth
await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    redirectTo: 'https://myapp.com/callback'
  }
})

// Magic Link
await supabase.auth.signInWithOtp({
  email: 'user@example.com'
})
```

### Session Management

```typescript
// Get current session
const { data: { session } } = await supabase.auth.getSession()

// Listen to auth changes
supabase.auth.onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') {
    // Handle sign in
  } else if (event === 'SIGNED_OUT') {
    // Handle sign out
  }
})

// Server-side (Next.js App Router)
import { createServerClient } from '@supabase/ssr'

const supabase = createServerClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!,
  { cookies }
)
```

### Custom Claims

```sql
-- Add custom claims to JWT
CREATE OR REPLACE FUNCTION custom_access_token_hook(event jsonb)
RETURNS jsonb AS $$
DECLARE
  claims jsonb;
  user_role text;
BEGIN
  SELECT role INTO user_role FROM user_profiles WHERE user_id = (event->>'user_id')::uuid;

  claims := event->'claims';
  claims := jsonb_set(claims, '{user_role}', to_jsonb(user_role));

  RETURN jsonb_set(event, '{claims}', claims);
END;
$$ LANGUAGE plpgsql;
```

## Row Level Security (RLS)

### RLS Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **User owns data** | Personal data | `auth.uid() = user_id` |
| **Role-based** | Admin access | `auth.jwt() ->> 'role' = 'admin'` |
| **Organization** | Multi-tenant | `org_id IN (SELECT org_id FROM memberships WHERE user_id = auth.uid())` |
| **Public read** | Blog posts | `SELECT: true, INSERT/UPDATE/DELETE: auth.uid() = author_id` |

### Basic RLS Setup

```sql
-- Enable RLS
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Policy: Users can read all posts
CREATE POLICY "Posts are viewable by everyone"
ON posts FOR SELECT
USING (true);

-- Policy: Users can only insert their own posts
CREATE POLICY "Users can insert own posts"
ON posts FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- Policy: Users can only update their own posts
CREATE POLICY "Users can update own posts"
ON posts FOR UPDATE
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);

-- Policy: Users can only delete their own posts
CREATE POLICY "Users can delete own posts"
ON posts FOR DELETE
USING (auth.uid() = user_id);
```

### Multi-Tenant RLS

```sql
-- Organization-based multi-tenancy
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL
);

CREATE TABLE memberships (
  user_id UUID REFERENCES auth.users(id),
  org_id UUID REFERENCES organizations(id),
  role TEXT NOT NULL DEFAULT 'member',
  PRIMARY KEY (user_id, org_id)
);

CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID REFERENCES organizations(id),
  name TEXT NOT NULL
);

-- RLS policy for organization access
CREATE POLICY "Users can access org projects"
ON projects FOR ALL
USING (
  org_id IN (
    SELECT org_id FROM memberships WHERE user_id = auth.uid()
  )
);
```

## Edge Functions vs Database Functions

### Comparison

| Aspect | Edge Functions | Database Functions |
|--------|----------------|-------------------|
| Language | TypeScript/Deno | PL/pgSQL |
| Latency | Higher (HTTP) | Lower (in-DB) |
| Use case | External APIs, complex logic | Data operations |
| Scaling | Global edge | Database scale |
| Debugging | Easier | Harder |

### When to Use Each

| Scenario | Recommendation |
|----------|----------------|
| Call external APIs | Edge Functions |
| Complex business logic | Edge Functions |
| Data validation | Database Functions |
| Triggers | Database Functions |
| Computed columns | Database Functions |
| Webhooks | Edge Functions |

### Edge Function Example

```typescript
// supabase/functions/send-email/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'

serve(async (req) => {
  const { email, subject, body } = await req.json()

  // Call external email service
  const response = await fetch('https://api.sendgrid.com/v3/mail/send', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${Deno.env.get('SENDGRID_API_KEY')}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      personalizations: [{ to: [{ email }] }],
      from: { email: 'noreply@myapp.com' },
      subject,
      content: [{ type: 'text/plain', value: body }]
    })
  })

  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' }
  })
})
```

### Database Function Example

```sql
-- Trigger function for updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER posts_updated_at
  BEFORE UPDATE ON posts
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

-- Function for complex data operation
CREATE OR REPLACE FUNCTION get_user_stats(user_uuid UUID)
RETURNS TABLE (
  total_posts BIGINT,
  total_likes BIGINT,
  total_comments BIGINT
) AS $$
BEGIN
  RETURN QUERY
  SELECT
    (SELECT COUNT(*) FROM posts WHERE user_id = user_uuid),
    (SELECT COUNT(*) FROM likes WHERE user_id = user_uuid),
    (SELECT COUNT(*) FROM comments WHERE user_id = user_uuid);
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

## Realtime

### Realtime Patterns

| Feature | Use Case | Example |
|---------|----------|---------|
| **Postgres Changes** | Database sync | Live updates |
| **Broadcast** | Custom events | Typing indicators |
| **Presence** | Online status | Who's online |

### Postgres Changes

```typescript
// Subscribe to table changes
const channel = supabase
  .channel('posts-changes')
  .on(
    'postgres_changes',
    {
      event: '*', // INSERT, UPDATE, DELETE, or *
      schema: 'public',
      table: 'posts',
      filter: 'user_id=eq.123' // Optional filter
    },
    (payload) => {
      console.log('Change received!', payload)
    }
  )
  .subscribe()

// Cleanup
channel.unsubscribe()
```

### Broadcast

```typescript
// Send custom events
const channel = supabase.channel('room-1')

// Listen
channel.on('broadcast', { event: 'typing' }, (payload) => {
  console.log(`${payload.user} is typing...`)
})

// Send
channel.send({
  type: 'broadcast',
  event: 'typing',
  payload: { user: 'Alice' }
})
```

### Presence

```typescript
// Track online users
const channel = supabase.channel('online-users')

channel.on('presence', { event: 'sync' }, () => {
  const state = channel.presenceState()
  console.log('Online:', Object.keys(state))
})

channel.subscribe(async (status) => {
  if (status === 'SUBSCRIBED') {
    await channel.track({
      user_id: userId,
      online_at: new Date().toISOString()
    })
  }
})
```

## Storage

### Storage Patterns

```typescript
// Upload file
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${userId}/avatar.png`, file)

// Get public URL
const { data: { publicUrl } } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${userId}/avatar.png`)

// Download file
const { data, error } = await supabase.storage
  .from('documents')
  .download('private/doc.pdf')

// Signed URL (temporary access)
const { data } = await supabase.storage
  .from('documents')
  .createSignedUrl('private/doc.pdf', 3600) // 1 hour
```

### Storage RLS

```sql
-- Storage policies
CREATE POLICY "Users can upload own avatar"
ON storage.objects FOR INSERT
WITH CHECK (
  bucket_id = 'avatars' AND
  (storage.foldername(name))[1] = auth.uid()::text
);

CREATE POLICY "Avatars are publicly accessible"
ON storage.objects FOR SELECT
USING (bucket_id = 'avatars');
```

## Database Design

### Schema Patterns

```sql
-- Users profile (extends auth.users)
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  username TEXT UNIQUE,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Auto-create profile on signup
CREATE OR REPLACE FUNCTION create_profile()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO profiles (id) VALUES (NEW.id);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION create_profile();
```

### Multi-Tenancy Patterns

| Pattern | Isolation | Complexity | Use Case |
|---------|-----------|------------|----------|
| Column | Low | Low | Simple SaaS |
| Schema | Medium | Medium | Stronger isolation |
| Database | High | High | Enterprise |

## Output Contract

```yaml
architecture_decision:
  vendor: "supabase"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<auth|rls|functions|realtime|storage>"

  options_evaluated:
    - id: "<option-id>"
      name: "<option name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        security: "<low|medium|high>"
        complexity: "<low|medium|high>"
        performance: "<low|medium|high>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  rls_policies:
    - table: "<table name>"
      policies:
        - name: "<policy name>"
          operation: "<SELECT|INSERT|UPDATE|DELETE|ALL>"
          check: "<policy expression>"

  implementation_guidance:
    getting_started:
      - "<step>"
    migrations:
      - "<migration>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| No RLS | Security vulnerability | Always enable RLS |
| Service key in client | Exposes full access | Use anon key + RLS |
| N+1 queries | Performance | Use joins, views |
| Large Edge Functions | Cold start | Keep functions small |
| No indexes | Slow queries | Index filtered columns |

## Integration Points

- Use **cost-supabase** to estimate costs
- Use **arch-nextjs** or **arch-sveltekit** for frontend
- Feeds into **implementation-plan** for project setup
