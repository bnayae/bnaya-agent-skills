---
name: arch-firebase
description: Evaluate Firebase architecture decisions including Firestore data modeling, Cloud Functions patterns, authentication strategies, hosting options, and scaling considerations. Use when user needs guidance on Firebase architecture choices.
color: amber
context: fork
agent: Explore
---

# Firebase Architecture Decision Evaluator

Evaluate and recommend Firebase configurations, Firestore modeling, and Cloud Functions patterns for TypeScript applications.

## Standalone Usage

Can be invoked directly:
- "How should I structure my Firestore data?"
- "Cloud Functions Gen1 vs Gen2?"
- "Best auth pattern for my Firebase app?"
- "When to use subcollections vs root collections?"

## Before Answering Firebase Architecture Questions

### Step 1: Check Available Tools

**Firebase CLI**:
```bash
# Check Firebase CLI
firebase --version

# Initialize project
firebase init

# Local emulators
firebase emulators:start
```

### Step 2: Fallback to Web Search

Use web search to fetch current information from:
- https://firebase.google.com/docs
- https://firebase.blog

## Firestore Data Modeling

### Collection Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Root collection** | Independent entities | `/users`, `/posts` |
| **Subcollection** | Parent-child relationship | `/users/{uid}/orders` |
| **Collection group** | Query across subcollections | All orders across users |

### When to Use Each

| Scenario | Pattern | Why |
|----------|---------|-----|
| Users list | Root collection | Independent, queryable |
| User's orders | Subcollection | Scoped to user |
| All orders (admin) | Collection group | Query across users |
| Chat messages | Subcollection | `/chats/{chatId}/messages` |
| Categories | Root collection | Shared across app |

### Document Design

```typescript
// Users collection
interface User {
  id: string;
  email: string;
  displayName: string;
  photoURL: string;
  createdAt: Timestamp;
  // Denormalized for display
  stats: {
    postsCount: number;
    followersCount: number;
  };
}

// Posts collection
interface Post {
  id: string;
  authorId: string;
  // Denormalized author info for display
  author: {
    displayName: string;
    photoURL: string;
  };
  title: string;
  content: string;
  tags: string[];
  likesCount: number;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

### Denormalization Patterns

| Pattern | Trade-off | When to Use |
|---------|-----------|-------------|
| **Embed related data** | Stale data vs fewer reads | Display data changes rarely |
| **Store counts** | Eventual consistency | Avoid counting queries |
| **Duplicate for queries** | Storage vs query simplicity | Compound queries |

### Querying Patterns

```typescript
import { collection, query, where, orderBy, limit, getDocs } from 'firebase/firestore';

// Simple query
const q = query(
  collection(db, 'posts'),
  where('authorId', '==', userId),
  orderBy('createdAt', 'desc'),
  limit(10)
);

// Compound query (requires composite index)
const q = query(
  collection(db, 'posts'),
  where('tags', 'array-contains', 'typescript'),
  where('likesCount', '>=', 10),
  orderBy('likesCount', 'desc')
);

// Collection group query
const q = query(
  collectionGroup(db, 'messages'),
  where('senderId', '==', userId)
);
```

### Index Design

```javascript
// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "posts",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "authorId", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "posts",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "tags", "arrayConfig": "CONTAINS" },
        { "fieldPath": "likesCount", "order": "DESCENDING" }
      ]
    }
  ]
}
```

## Cloud Functions

### Gen1 vs Gen2

| Aspect | Gen1 | Gen2 |
|--------|------|------|
| Runtime | Node.js 14-18 | Node.js 18+ |
| Concurrency | 1 request/instance | Up to 1000/instance |
| Min instances | Yes | Yes |
| Traffic splitting | No | Yes |
| Timeout | 9 minutes | 60 minutes |
| Memory | Up to 8GB | Up to 32GB |

### Function Types

| Type | Trigger | Use Case |
|------|---------|----------|
| **HTTP** | HTTP request | APIs, webhooks |
| **Callable** | Client SDK | Secure client calls |
| **Firestore** | Document changes | Data sync, validation |
| **Auth** | User events | Welcome email, profile |
| **Storage** | File uploads | Processing, thumbnails |
| **Scheduled** | Cron | Cleanup, reports |
| **Pub/Sub** | Messages | Async processing |

### Function Examples

```typescript
// HTTP Function (Gen2)
import { onRequest } from 'firebase-functions/v2/https';

export const api = onRequest(
  { cors: true, region: 'us-central1' },
  async (req, res) => {
    res.json({ message: 'Hello World' });
  }
);

// Callable Function
import { onCall, HttpsError } from 'firebase-functions/v2/https';

export const createPost = onCall(async (request) => {
  if (!request.auth) {
    throw new HttpsError('unauthenticated', 'Must be logged in');
  }

  const { title, content } = request.data;
  // Create post...
  return { postId: 'new-post-id' };
});

// Firestore Trigger
import { onDocumentCreated } from 'firebase-functions/v2/firestore';

export const onPostCreated = onDocumentCreated(
  'posts/{postId}',
  async (event) => {
    const post = event.data?.data();
    const postId = event.params.postId;

    // Update author's post count
    await db.doc(`users/${post.authorId}`).update({
      'stats.postsCount': FieldValue.increment(1)
    });
  }
);

// Auth Trigger
import { onUserCreated } from 'firebase-functions/v2/identity';

export const onNewUser = onUserCreated(async (event) => {
  const user = event.data;

  // Create user profile
  await db.doc(`users/${user.uid}`).set({
    email: user.email,
    displayName: user.displayName || '',
    createdAt: FieldValue.serverTimestamp()
  });
});

// Scheduled Function
import { onSchedule } from 'firebase-functions/v2/scheduler';

export const dailyCleanup = onSchedule(
  { schedule: '0 0 * * *', timeZone: 'America/New_York' },
  async () => {
    // Cleanup old data
  }
);
```

### Cold Start Mitigation

```typescript
// Min instances (costs money but reduces cold starts)
export const api = onRequest(
  { minInstances: 1 },
  async (req, res) => { ... }
);

// Lazy initialization
let db: Firestore;
function getDb() {
  if (!db) {
    db = getFirestore();
  }
  return db;
}
```

## Authentication

### Auth Providers

| Provider | Use Case | Setup |
|----------|----------|-------|
| Email/Password | Traditional | Built-in |
| Google | Social login | Enable in console |
| Apple | iOS requirement | Configure |
| Phone | SMS verification | Verify domain |
| Anonymous | Guest users | Enable |
| Custom token | External auth | Server-side |

### Auth Implementation

```typescript
import { getAuth, signInWithPopup, GoogleAuthProvider } from 'firebase/auth';

const auth = getAuth();

// Google sign-in
const provider = new GoogleAuthProvider();
const result = await signInWithPopup(auth, provider);
const user = result.user;

// Listen to auth state
onAuthStateChanged(auth, (user) => {
  if (user) {
    console.log('Signed in:', user.uid);
  } else {
    console.log('Signed out');
  }
});

// Custom claims (server-side)
import { getAuth } from 'firebase-admin/auth';

await getAuth().setCustomUserClaims(uid, {
  admin: true,
  organization: 'org-123'
});
```

### Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }

    function isOwner(userId) {
      return request.auth.uid == userId;
    }

    function isAdmin() {
      return request.auth.token.admin == true;
    }

    // Users collection
    match /users/{userId} {
      allow read: if isAuthenticated();
      allow write: if isOwner(userId);
    }

    // Posts collection
    match /posts/{postId} {
      allow read: if true;  // Public
      allow create: if isAuthenticated()
        && request.resource.data.authorId == request.auth.uid;
      allow update, delete: if isAuthenticated()
        && resource.data.authorId == request.auth.uid;
    }

    // Admin access
    match /{document=**} {
      allow read, write: if isAdmin();
    }
  }
}
```

## Hosting Options

| Option | Best For | Features |
|--------|----------|----------|
| **Firebase Hosting** | Static, SPAs | CDN, SSL, preview |
| **Cloud Run** | SSR, containers | Full control |
| **App Engine** | Complex apps | Scaling, services |

### Firebase Hosting + Cloud Functions

```javascript
// firebase.json
{
  "hosting": {
    "public": "public",
    "rewrites": [
      {
        "source": "/api/**",
        "function": "api"
      },
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

## Scaling Considerations

### Firestore Limits

| Limit | Value |
|-------|-------|
| Document size | 1 MB |
| Field depth | 20 levels |
| Write rate (single doc) | 1 write/second |
| Index entries per doc | 40,000 |
| Collection name length | 6,144 bytes |

### Scaling Patterns

| Pattern | Problem | Solution |
|---------|---------|----------|
| Hot documents | Write contention | Distributed counters |
| Large arrays | 1MB limit | Subcollection |
| Fanout writes | Slow updates | Cloud Functions async |

### Distributed Counter

```typescript
// Shard counter across multiple documents
async function incrementCounter(db: Firestore, docRef: DocumentReference) {
  const numShards = 10;
  const shardId = Math.floor(Math.random() * numShards);
  const shardRef = docRef.collection('shards').doc(String(shardId));

  await shardRef.update({
    count: FieldValue.increment(1)
  });
}

async function getCount(db: Firestore, docRef: DocumentReference) {
  const shards = await docRef.collection('shards').get();
  let total = 0;
  shards.forEach(doc => {
    total += doc.data().count;
  });
  return total;
}
```

## Output Contract

```yaml
architecture_decision:
  vendor: "firebase"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<firestore|functions|auth|hosting>"

  options_evaluated:
    - id: "<option-id>"
      name: "<option name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        scalability: "<low|medium|high>"
        complexity: "<low|medium|high>"
        cost: "<low|medium|high>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  data_model:
    collections:
      - name: "<collection>"
        type: "<root|subcollection>"
        fields: []
        indexes: []

  implementation_guidance:
    getting_started:
      - "<step>"
    security_rules: "<rule snippet>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Deep nesting | Query limitations | Flatten structure |
| Unbounded arrays | 1MB limit, slow updates | Subcollections |
| No indexes | Slow/failed queries | Plan indexes upfront |
| Reading entire collection | Costs, performance | Pagination, filters |
| Sync processing in triggers | Timeout, retry issues | Use Pub/Sub |

## Integration Points

- Use **cost-firebase** to estimate costs
- Use **arch-nextjs** for Next.js integration
- Feeds into **implementation-plan** for project setup
