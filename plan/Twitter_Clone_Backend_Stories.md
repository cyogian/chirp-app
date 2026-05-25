# Twitter Clone — Backend Stories

> **Stack:** Node.js 24 · Next.js 16 Server Actions · Prisma 7 · PostgreSQL 17 / Aurora · Redis 7 / ElastiCache · NextAuth 5 · Microsoft Entra ID · AWS KMS · Custom SDK · Zod (server-side) · CopilotKit v2 Runtime

> **Scope:** All server-side work — project scaffolding, Prisma schema, Redis client, Server Actions, API routes, service/repository layer, authentication, caching, real-time SSE, AI backend route, and security hardening.
> For React components and client-side state see the Frontend file. For tests see the Testing file. For deployment see the DevOps file.

---

## Table of Contents

1. [Epic 1 — Project Setup & Scaffolding](#epic-1--project-setup--scaffolding)
2. [Epic 2 — Authentication & Authorization](#epic-2--authentication--authorization)
3. [Epic 3 — Zod Shared Schemas](#epic-3--zod-shared-schemas)
4. [Epic 4 — Core Tweet Server Actions & Services](#epic-4--core-tweet-server-actions--services)
5. [Epic 5 — User Profiles & Social Graph (Backend)](#epic-5--user-profiles--social-graph-backend)
6. [Epic 6 — Feed Architecture & Fan-out](#epic-6--feed-architecture--fan-out)
7. [Epic 7 — Search Service & API Routes](#epic-7--search-service--api-routes)
8. [Epic 8 — Real-time Notifications (Server Side)](#epic-8--real-time-notifications-server-side)
9. [Epic 9 — AI Backend Route & SDK](#epic-9--ai-backend-route--sdk)
10. [Epic 10 — Analytics Data Layer](#epic-10--analytics-data-layer)
11. [Epic 11 — Caching Strategy (Server Side)](#epic-11--caching-strategy-server-side)
12. [Epic 12 — Security & Rate Limiting](#epic-12--security--rate-limiting)

---

## Epic 1 — Project Setup & Scaffolding

**Goal:** Bootstrap the shared project foundation — Prisma schema, Redis client, service/repository stubs, and the Custom SDK skeleton — so all backend feature work can begin immediately.

---

### Story 1.1 — Folder structure and service stubs

**As a** developer,
**I want** a well-defined backend folder structure with service and repository stubs in place,
**so that** responsibilities are clear and engineers can implement features in isolation.

**Acceptance Criteria:**

- `src/services/` contains: `FeedService`, `NotificationService`, `SearchService`, `RateLimitService`, `EncryptionService`
- `src/repositories/` contains: `TweetRepository`, `UserRepository`, `FollowRepository`
- `src/lib/sdk/` contains: `BaseApiClient`, `TweetClient`, `UserClient`, `FeedClient`, `NotificationClient`, `AIClient`, `index.ts` barrel
- Each stub exports a class with typed method signatures (no implementation yet)

**Tasks:**

- [ ] Create `src/services/FeedService.ts` stub with `getHomeFeed`, `fanOut` method signatures
- [ ] Create `src/services/NotificationService.ts` stub with `create`, `markAllRead`
- [ ] Create `src/services/SearchService.ts` stub with `searchTweets`, `searchUsers`, `getTrending`
- [ ] Create `src/services/RateLimitService.ts` stub with `checkLimit`
- [ ] Create `src/services/EncryptionService.ts` stub wrapping KMS calls
- [ ] Create `src/repositories/TweetRepository.ts`, `UserRepository.ts`, `FollowRepository.ts` stubs
- [ ] Create `src/lib/sdk/BaseApiClient.ts` with `fetch` wrapper, auth header injection, retry logic skeleton
- [ ] Create `src/lib/sdk/TweetClient.ts`, `UserClient.ts`, `FeedClient.ts`, `NotificationClient.ts`, `AIClient.ts` extending `BaseApiClient`
- [ ] Create `src/lib/sdk/index.ts` barrel: `export const api = { tweets, users, feed, notifications, ai }`
- [ ] Document the service/repository pattern in `CONTRIBUTING.md`

---

### Story 1.2 — Prisma 7 schema and migrations

**As a** developer,
**I want** Prisma 7 configured with the full schema and initial migration committed,
**so that** any engineer can run `prisma migrate dev` and have a working database.

**Acceptance Criteria:**

- `prisma/schema.prisma` defines all models: `User`, `Tweet`, `Like`, `Follow`, `Hashtag`, `TweetHashtag`, `Notification`
- All relations, indexes (GIN on `Tweet.content`, B-tree on `authorId`/`createdAt`), and unique constraints defined
- `DATABASE_URL` sourced from `.env.local`
- Prisma Client singleton in `src/lib/prisma.ts` with `globalThis.__prisma` guard
- Seed script at `prisma/seed.ts` with 5 test users and 20 seeded tweets

**Tasks:**

- [ ] Install `prisma@7` and `@prisma/client`
- [ ] Define `User` model with `id`, `handle` (unique), `email` (unique), `displayName`, `avatarUrl`, `bio`, timestamps
- [ ] Define `Tweet` model with `authorId` (FK), `parentId` (nullable FK), `isRetweet`, `retweetOfId` (nullable FK), `content`, `createdAt`
- [ ] Define `Like` model with composite PK (`userId`, `tweetId`)
- [ ] Define `Follow` model with composite PK (`followerId`, `followingId`) and self-follow prevention at app layer
- [ ] Define `Hashtag`, `TweetHashtag` join table
- [ ] Define `Notification` model with `type` enum (`LIKE`, `FOLLOW`, `REPLY`, `RETWEET`), `recipientId`, `actorId`, `referenceId`, `read`
- [ ] Add `@@index([authorId, createdAt])` and `@@index([parentId])` to `Tweet`
- [ ] Add `@db.Text` GIN index via raw migration for full-text search on `Tweet.content`
- [ ] Create `src/lib/prisma.ts` with dev-safe singleton pattern
- [ ] Run `prisma migrate dev --name init` and commit migration files
- [ ] Write `prisma/seed.ts` and add `"prisma": { "seed": "ts-node prisma/seed.ts" }` to `package.json`

---

### Story 1.3 — Redis client singleton

**As a** developer,
**I want** a typed Redis 7 client singleton accessible via `@/lib/redis`,
**so that** all caching and pub/sub features are built on a consistent foundation.

**Acceptance Criteria:**

- `ioredis` client instantiated once; reused across hot-reloads in dev via `globalThis.__redis`
- `REDIS_URL` loaded from `.env.local`
- Typed helper functions exported: `get`, `set`, `del`, `incr`, `zadd`, `zrevrange`, `lrange`, `lpush`, `ltrim`, `mget`, `publish`, `subscribe`
- Separate `createSubscriberClient()` factory for pub/sub (dedicated connection)

**Tasks:**

- [ ] Install `ioredis`
- [ ] Create `src/lib/redis.ts` with `globalThis.__redis` singleton and reconnect strategy (`retryStrategy`)
- [ ] Implement typed helper functions for each Redis command used across the codebase
- [ ] Export `createSubscriberClient()` factory returning a fresh `ioredis` instance for subscriptions
- [ ] Add `REDIS_URL=redis://localhost:6379` to `.env.example`
- [ ] Write unit tests for helpers using `ioredis-mock`

---

### Story 1.4 — Custom SDK — BaseApiClient with circuit breaker

**As a** developer,
**I want** the `BaseApiClient` to include retry logic and a circuit breaker,
**so that** calls to external services fail fast and don't cascade under load.

**Acceptance Criteria:**

- `BaseApiClient.request()` retries up to 3 times with exponential backoff on 5xx errors
- Circuit breaker: after 5 consecutive failures, opens circuit for 30s (returns error immediately without calling upstream)
- All requests attach `Authorization: Bearer <JWT>` header from session
- All responses parsed through a Zod schema via `schema.parse(json)` before returning

**Tasks:**

- [ ] Implement `request<T>(path, options, responseSchema)` in `BaseApiClient`
- [ ] Add exponential backoff retry: `delay = 200 * 2^attempt`
- [ ] Implement circuit breaker state machine (`CLOSED` → `OPEN` → `HALF_OPEN`) using module-level state
- [ ] Inject `Authorization` header from `getServerSession()` inside Server Action context
- [ ] Call `responseSchema.parse(data)` on every response; throw `ZodError` on mismatch
- [ ] Write unit tests for: retry on 503, circuit opens after 5 failures, circuit half-opens after 30s

---

## Epic 2 — Authentication & Authorization

**Goal:** Implement NextAuth 5 with Microsoft Entra ID, AWS KMS-signed JWTs, and middleware route protection.

---

### Story 2.1 — NextAuth 5 with Microsoft Entra ID provider

**As a** user,
**I want** to sign in with my Microsoft account,
**so that** I can access the platform without creating a new password.

**Acceptance Criteria:**

- `next-auth@5` installed with `@auth/prisma-adapter`
- `MicrosoftEntraID` provider configured with `clientId`, `clientSecret`, `tenantId`
- OAuth 2.0 PKCE flow initiated correctly
- On callback: `id_token` validated, user upserted in PostgreSQL via Prisma
- Unique `handle` generated from display name on first sign-in (slugified + random suffix if collision)
- `httpOnly` session cookie set with `Secure` flag in production

**Tasks:**

- [ ] Install `next-auth@5` and `@auth/prisma-adapter`
- [ ] Create `src/auth.ts` with `NextAuth({ providers: [MicrosoftEntraID({...})] })`
- [ ] Create `app/api/auth/[...nextauth]/route.ts` exporting `{ GET, POST }` from `src/auth.ts`
- [ ] Configure `NEXTAUTH_SECRET`, `ENTRA_CLIENT_ID`, `ENTRA_CLIENT_SECRET`, `ENTRA_TENANT_ID` in `.env.example`
- [ ] Implement `signIn` callback: upsert user via `prisma.user.upsert`, generate unique `handle`
- [ ] Add `generateHandle(displayName)` utility: slugify + check uniqueness + append 4-char suffix on collision
- [ ] Write integration test for `signIn` callback using NextAuth mock session

---

### Story 2.2 — JWT signing via AWS KMS

**As a** security engineer,
**I want** session JWTs signed using an AWS KMS asymmetric key (RS256),
**so that** private signing keys never leave the KMS Hardware Security Module.

**Acceptance Criteria:**

- `src/lib/kms.ts` wraps `@aws-sdk/client-kms` with `signJWT(payload)` and `verifyJWT(token)`
- RSA public key cached in module scope; refreshed every 24 hours
- NextAuth `jwt.encode` and `jwt.decode` callbacks replaced with KMS-backed implementations
- JWT payload: `{ sub, handle, displayName, iat, exp }` — no sensitive fields

**Tasks:**

- [ ] Install `@aws-sdk/client-kms`
- [ ] Create `src/lib/kms.ts` with `signJWT` (calls `KMS.sign`) and `verifyJWT` (calls `KMS.verify`)
- [ ] Implement public key cache: `let cachedKey: string | null; let cacheExpiry: number`
- [ ] Override `jwt.encode` in NextAuth config: call `kms.signJWT(payload)`
- [ ] Override `jwt.decode` in NextAuth config: call `kms.verifyJWT(token)`
- [ ] Write unit tests mocking `@aws-sdk/client-kms` for both sign and verify paths
- [ ] Document key rotation procedure and IAM permissions needed in `docs/security.md`

---

### Story 2.3 — Middleware auth guard

**As a** developer,
**I want** Next.js middleware to enforce authentication on all protected routes,
**so that** unauthenticated requests never reach Server Components or Server Actions.

**Acceptance Criteria:**

- `middleware.ts` matches all routes except `/(auth)/**`, `/api/auth/**`, and static assets
- Missing or invalid JWT → `redirect('/login')`
- Valid JWT → request forwarded with `x-user-id` and `x-user-handle` headers injected
- `X-RateLimit-Limit` and `X-RateLimit-Remaining` headers added for API routes

**Tasks:**

- [ ] Create `middleware.ts` with `export const config = { matcher: [...] }`
- [ ] Verify JWT using `kms.verifyJWT(token)` from cookie
- [ ] Redirect to `/login` on verification failure
- [ ] Inject `x-user-id` and `x-user-handle` as request headers for downstream RSCs
- [ ] Call `RateLimitService.checkLimit` for `/api/**` routes and attach response headers
- [ ] Write Vitest integration test: invalid token → assert redirect; valid token → assert headers injected

---

## Epic 3 — Zod Shared Schemas

**Goal:** Define all Zod schemas used for server-side validation in a shared location so they can also be consumed by client forms.

---

### Story 3.1 — Tweet schemas

**As a** developer,
**I want** shared Zod schemas for all tweet mutations,
**so that** the same validation logic runs on both client and server with a single source of truth.

**Acceptance Criteria:**

- `createTweetSchema`: `content` (min 1, max 280), optional `parentId` (cuid2), optional `hashtags` (string array, max 5)
- `updateTweetSchema`: partial of `createTweetSchema`
- `likeTweetSchema`: `tweetId` (cuid2)
- `retweetSchema`: `tweetId` (cuid2)
- Inferred TypeScript types exported alongside each schema

**Tasks:**

- [ ] Create `src/shared/schemas/tweet.schema.ts` with all 4 schemas
- [ ] Add `.transform(val => val.trim())` to `content` field
- [ ] Add `.transform(val => stripHtml(val))` to `content` using a lightweight HTML-strip utility
- [ ] Export `CreateTweetInput`, `LikeTweetInput`, `RetweetInput` inferred types
- [ ] Add barrel export to `src/shared/schemas/index.ts`
- [ ] Write Vitest unit tests: boundary values for `content`, invalid `cuid2` for `tweetId`, too many hashtags

---

### Story 3.2 — User and auth schemas

**As a** developer,
**I want** shared Zod schemas for user mutations and auth flows,
**so that** profile updates and sign-in data are consistently validated.

**Acceptance Criteria:**

- `updateProfileSchema`: `displayName` (min 1, max 50), `bio` (max 160, optional), `avatarUrl` (URL, optional)
- `followSchema`: `targetHandle` (string, min 1)
- `searchQuerySchema`: `q` (string, min 2, max 100), `type` (enum `tweets | users`)

**Tasks:**

- [ ] Create `src/shared/schemas/user.schema.ts` with `updateProfileSchema`, `followSchema`
- [ ] Create `src/shared/schemas/search.schema.ts` with `searchQuerySchema`
- [ ] Export inferred types: `UpdateProfileInput`, `FollowInput`, `SearchQueryInput`
- [ ] Write Vitest unit tests: `avatarUrl` must be valid URL, `bio` max 160, `type` enum rejects unknown values

---

## Epic 4 — Core Tweet Server Actions & Services

**Goal:** Implement all tweet-related Server Actions with full Zod validation, auth checks, rate limiting, and cache invalidation.

---

### Story 4.1 — `createTweetAction`

**As a** developer,
**I want** a `createTweetAction` that validates, rate-limits, and persists a tweet in a single atomic operation,
**so that** tweet creation is secure, consistent, and cache-safe.

**Acceptance Criteria:**

- Zod re-validates with `createTweetSchema.safeParse`; returns `{ errors }` on failure
- `getServerSession()` check; returns `{ error: 'Unauthorized' }` if absent
- `RateLimitService.checkLimit(userId, 'createTweet', 10, 60)` check; returns `{ error: 'Rate limit', retryAfter }` if exceeded
- `AIClient.moderate(content)` called before DB insert; returns `{ error: 'Content policy violation' }` if flagged
- Prisma transaction: `tweet.create()` + `hashtag.upsertMany()` + `TweetHashtag` records
- Redis: `DEL feed:*` pattern + `ZADD hashtag:counts` for each hashtag
- `revalidatePath('/home')` called
- Returns `{ success: true, tweet }`

**Tasks:**

- [ ] Create `src/app/(main)/home/actions.ts` with `'use server'`
- [ ] Implement `createTweetAction(input: unknown)` with all validation steps in sequence
- [ ] Extract hashtags from content with regex `/\#(\w+)/g`; upsert `Hashtag` + `TweetHashtag` in same transaction
- [ ] Implement Redis fan-out trigger: call `FeedService.fanOut(tweet.id, session.user.id)` after commit
- [ ] Call `revalidatePath('/home')` and return `{ success: true, tweet }`
- [ ] Write integration test: mocked Prisma + real Zod; assert valid input → success; invalid → errors returned

---

### Story 4.2 — `likeTweetAction` and `retweetAction`

**As a** developer,
**I want** like and retweet Server Actions with optimistic-update-friendly response shapes,
**so that** the client can reflect changes instantly.

**Acceptance Criteria:**

- `likeTweetAction`: upsert `Like` record (idempotent); `INCR tweet:likes:tweetId` in Redis; publish notification
- `retweetAction`: create `Tweet` with `isRetweet: true`; fan-out to actor's followers
- Both re-validate with `likeTweetSchema`/`retweetSchema`, check auth, check rate limit
- Both return `{ success: true, likeCount }` or `{ success: true, retweetCount }`

**Tasks:**

- [ ] Implement `likeTweetAction` in `src/app/(main)/tweet/actions.ts`
- [ ] Implement `unlikeTweetAction` (delete `Like` record, DECR Redis counter)
- [ ] Implement `retweetAction` creating a tweet with `isRetweet: true` and `retweetOfId`
- [ ] Trigger `NotificationService.create` after like (notify tweet author)
- [ ] Write integration tests for both actions with mocked Prisma and Redis

---

### Story 4.3 — `deleteTweetAction`

**As a** developer,
**I want** a `deleteTweetAction` with ownership verification,
**so that** users can only delete their own tweets.

**Acceptance Criteria:**

- Verifies `tweet.authorId === session.user.id`; returns `{ error: 'Forbidden' }` otherwise
- Soft deletes: sets `deletedAt` timestamp (add `deletedAt DateTime?` to schema)
- Cascades cache invalidation: `DEL feed:*`, `revalidatePath('/home')`
- All queries filter `WHERE deletedAt IS NULL` via Prisma global query extension

**Tasks:**

- [ ] Add `deletedAt DateTime?` to `Tweet` model and run migration
- [ ] Add Prisma client extension (`$extends`) to auto-filter soft-deleted records
- [ ] Implement `deleteTweetAction(tweetId: string)` with ownership check
- [ ] Invalidate Redis feed cache and call `revalidatePath`
- [ ] Write integration test: other user's tweet → `Forbidden`; own tweet → soft deleted

---

## Epic 5 — User Profiles & Social Graph (Backend)

**Goal:** Server Actions and service layer for following users, editing profiles, and managing the social graph in Redis and PostgreSQL.

---

### Story 5.1 — `followUserAction` and `unfollowUserAction`

**As a** developer,
**I want** follow and unfollow Server Actions that maintain both the DB record and the Redis social graph,
**so that** feed fan-out and follower counts are always consistent.

**Acceptance Criteria:**

- `followUserAction` validates `followSchema`, checks auth, prevents self-follow (returns `{ error: 'Cannot follow yourself' }`)
- Creates `Follow` record in Prisma; adds `followingId` to `user:following:userId` Redis Set
- Triggers fan-out: adds followee's recent 20 tweets to `feed:userId` Redis list
- Publishes follow notification
- `unfollowUserAction` deletes `Follow` record; removes from Redis Set; removes followee tweets from feed list

**Tasks:**

- [ ] Implement `followUserAction` in `src/app/(main)/[handle]/actions.ts`
- [ ] Implement `unfollowUserAction`
- [ ] Update `user:following:userId` Redis Set on both actions
- [ ] In `followUserAction`: run `LRANGE tweet:userId 0 19` on followee and `LPUSH feed:followerId` for each
- [ ] Trigger `NotificationService.create` for FOLLOW event
- [ ] Write integration test: self-follow → error; valid follow → DB record + Redis Set updated

---

### Story 5.2 — `updateProfileAction`

**As a** developer,
**I want** an `updateProfileAction` that validates and persists profile changes,
**so that** users can update their bios and display names safely.

**Acceptance Criteria:**

- Validates with `updateProfileSchema.safeParse`
- Checks that `session.user.id` matches the user being updated (ownership)
- Updates `User` record via `prisma.user.update`
- Invalidates `profile:handle` Redis key and calls `revalidatePath('/[handle]')`

**Tasks:**

- [ ] Implement `updateProfileAction(input: unknown)` in `src/app/(main)/[handle]/actions.ts`
- [ ] Add `avatarUrl` validation: must be a URL pointing to an allowlisted domain (CDN only)
- [ ] Invalidate `profile:handle` Redis key: `redis.del(`profile:${handle}`)`
- [ ] Call `revalidatePath(`/[handle]`)` after update
- [ ] Write integration test: mismatched user → Forbidden; valid input → DB updated + cache invalidated

---

## Epic 6 — Feed Architecture & Fan-out

**Goal:** Implement the Redis-backed fan-out-on-write feed with a PostgreSQL fallback, delivering O(1) feed reads.

---

### Story 6.1 — Feed API route with Redis-first reads

**As a** developer,
**I want** `/api/feed` to serve personalized feed pages from Redis first, with a PostgreSQL fallback,
**so that** feed reads are consistently fast.

**Acceptance Criteria:**

- `GET /api/feed?cursor=&limit=20` checks `LRANGE feed:userId {offset} {offset+limit-1}` in Redis
- On Redis hit: hydrate tweet details with `MGET tweet:id:*` pipeline from a tweet detail hash or fetch from DB using `prisma.tweet.findMany({ where: { id: { in: ids } } })`
- On Redis miss: query PostgreSQL (`WHERE authorId IN (followingIds) ORDER BY createdAt DESC LIMIT 20`), populate Redis list
- Returns `{ tweets: TweetWithAuthor[], nextCursor: string | null }`
- Validates `cursor` and `limit` query params with Zod

**Tasks:**

- [ ] Create `app/api/feed/route.ts` with GET handler
- [ ] Validate query params: `feedQuerySchema = z.object({ cursor: z.string().optional(), limit: z.coerce.number().int().min(1).max(50).default(20) })`
- [ ] Implement `FeedService.getHomeFeed(userId, cursor, limit)` with Redis-first logic
- [ ] Compute offset from cursor (find tweet index in Redis list)
- [ ] Hydrate tweet IDs to full `TweetWithAuthor` objects via pipeline `MGET`
- [ ] Fallback: `prisma.tweet.findMany()` joining `author`, `_count` for likes/retweets/replies
- [ ] Return `nextCursor` as the `createdAt` ISO string of the last tweet in the page
- [ ] Write integration test: Redis hit path and Redis miss path (both must return same shape)

---

### Story 6.2 — Fan-out on write service

**As a** developer,
**I want** `FeedService.fanOut(tweetId, authorId)` to push the new tweet to all followers' Redis feed lists,
**so that** feed reads remain O(1) regardless of follower count.

**Acceptance Criteria:**

- Retrieves follower IDs from `user:following:userId` Redis Set (fallback to DB for cold-start)
- For ≤ 10,000 followers: pipeline `LPUSH feed:followerId tweetId` + `LTRIM feed:followerId 0 499` for each follower
- For > 10,000 followers: enqueue fan-out job to a Redis list (`queue:fanout`) for async worker processing
- After fan-out: `PUBLISH channel:feed:updates userId`

**Tasks:**

- [ ] Implement `FeedService.fanOut(tweetId, authorId)` in `src/services/FeedService.ts`
- [ ] Fetch follower IDs: `SMEMBERS user:following:authorId`; fallback `prisma.follow.findMany({ where: { followingId: authorId } })`
- [ ] Batch pipeline `LPUSH` + `LTRIM` for ≤ 10k followers using `ioredis` pipeline
- [ ] For > 10k: `RPUSH queue:fanout JSON.stringify({ tweetId, authorId, followerIds })`
- [ ] Implement `FeedWorker` consuming `queue:fanout` with `BLPOP` (async background process)
- [ ] Write unit tests: assert `LPUSH`/`LTRIM` calls for each follower with mock Redis; assert queue push for > 10k

---

## Epic 7 — Search Service & API Routes

**Goal:** PostgreSQL full-text tweet search, user search, and Redis-backed trending hashtags.

---

### Story 7.1 — Full-text tweet search

**As a** developer,
**I want** `SearchService.searchTweets` using PostgreSQL GIN full-text search,
**so that** tweet search is fast and relevance-ranked.

**Acceptance Criteria:**

- Uses `WHERE to_tsvector('english', content) @@ plainto_tsquery($term)` with GIN index
- Supports cursor pagination
- Returns `TweetWithAuthor[]`
- Input validated by `searchQuerySchema` before hitting the DB

**Tasks:**

- [ ] Verify GIN index exists: `CREATE INDEX tweet_content_fts ON "Tweet" USING GIN (to_tsvector('english', content))` — add raw migration if absent
- [ ] Implement `SearchService.searchTweets(query, cursor, limit)` using `prisma.$queryRaw`
- [ ] Implement `SearchService.searchUsers(query)`: `WHERE handle ILIKE $1 OR displayName ILIKE $1 LIMIT 20`
- [ ] Create `app/api/search/route.ts` validating `searchQuerySchema`, calling appropriate service method
- [ ] Return results with proper `TweetWithAuthor` / `UserPublic` shape
- [ ] Write integration test with seeded DB: search for known content → assert tweet returned

---

### Story 7.2 — Trending hashtags with Redis Sorted Set

**As a** developer,
**I want** a Redis Sorted Set powering real-time trending hashtags,
**so that** `getTrending()` returns the top 10 hashtags of the last 24 hours in O(log n).

**Acceptance Criteria:**

- `createTweetAction` calls `ZADD hashtag:counts <timestamp> <hashtag>` for each hashtag in the tweet
- `getTrending()` uses `ZREVRANGE hashtag:counts 0 9 WITHSCORES` from a sliding 24h sorted set
- Stale entries older than 24h pruned via `ZREMRANGEBYSCORE hashtag:counts 0 <now - 86400000>` before each read
- Result cached in Redis at `trending:top10` with TTL 30s

**Tasks:**

- [ ] Implement `SearchService.getTrending()` with pruning + `ZREVRANGE` + `trending:top10` cache
- [ ] Update `createTweetAction` to call `ZADD hashtag:counts ${Date.now()} ${tag}` for each hashtag
- [ ] Update `deleteTweetAction` to call `ZINCRBY hashtag:counts -1 ${tag}` (adjust score on delete)
- [ ] Create `app/api/trending/route.ts` returning top 10 with `export const revalidate = 30`
- [ ] Write unit test for `getTrending` with mock Redis returning sorted set entries

---

## Epic 8 — Real-time Notifications (Server Side)

**Goal:** SSE stream endpoint, notification creation service, and pub/sub plumbing for real-time delivery.

---

### Story 8.1 — SSE stream endpoint

**As a** developer,
**I want** a `/api/stream` route that holds an open SSE connection per authenticated user,
**so that** real-time events can be pushed to the browser without WebSockets.

**Acceptance Criteria:**

- `GET /api/stream` returns `Content-Type: text/event-stream` with `Cache-Control: no-cache`
- Route verifies session before subscribing; returns 401 on invalid JWT
- Subscribes to `channel:notif:userId` on a dedicated Redis subscriber client
- Forwards each Redis message as `data: <json>\n\n` to the `ReadableStream`
- Unsubscribes and closes stream when `request.signal` fires `abort`

**Tasks:**

- [ ] Create `app/api/stream/route.ts` with `GET` handler
- [ ] Verify session with `getServerSession()`; return `new Response('Unauthorized', { status: 401 })` on failure
- [ ] Open `ReadableStream` with `start(controller)` callback
- [ ] Call `createSubscriberClient().subscribe('channel:notif:userId')`
- [ ] Forward messages: `controller.enqueue(encoder.encode(`data: ${msg}\n\n`))`
- [ ] Listen to `request.signal` `'abort'` event: unsubscribe + `controller.close()`
- [ ] Write integration test with mock `ioredis` subscriber: assert message forwarded to stream; abort closes connection

---

### Story 8.2 — Notification service

**As a** developer,
**I want** `NotificationService.create` to persist a notification and publish to Redis in a single call,
**so that** all notification-triggering Server Actions use a consistent, atomic flow.

**Acceptance Criteria:**

- Creates `Notification` record in PostgreSQL via Prisma
- Publishes `{ type, actorHandle, actorAvatarUrl, tweetId?, createdAt }` to `channel:notif:recipientId`
- Increments `notif:unread:recipientId` Redis counter via `INCR`
- Skips if `actorId === recipientId` (no self-notifications)

**Tasks:**

- [ ] Implement `NotificationService.create(recipientId, actorId, type, referenceId?)` in `src/services/NotificationService.ts`
- [ ] Add self-notification guard: early return if `actorId === recipientId`
- [ ] Run `prisma.notification.create` + `redis.publish` + `redis.incr` (best-effort publish; DB is source of truth)
- [ ] Implement `NotificationService.markAllRead(userId)`: `UPDATE notifications SET read = true WHERE recipientId = ?` + `DEL notif:unread:userId`
- [ ] Create `app/api/notifications/route.ts`: paginated GET returning last 20 notifications for session user
- [ ] Create `markAllReadAction` Server Action calling `NotificationService.markAllRead`
- [ ] Write integration test: create notification → assert DB record + Redis publish called; markAllRead → assert `read=true`

---

## Epic 9 — AI Backend Route & SDK

**Goal:** CopilotKit v2 runtime route, Azure OpenAI adapter, and the `AIClient` SDK for content moderation.

---

### Story 9.1 — CopilotKit runtime route

**As a** developer,
**I want** `/api/copilot` to handle CopilotKit streaming requests via Azure OpenAI,
**so that** all AI features have a single, secured backend endpoint.

**Acceptance Criteria:**

- `POST /api/copilot` validates session before processing
- `CopilotRuntime` configured with `OpenAIAdapter` pointing at Azure OpenAI (GPT-4o)
- Streaming response via SSE to the CopilotKit client
- `AZURE_OPENAI_API_KEY` and `AZURE_OPENAI_ENDPOINT` loaded from env (Secrets Manager in production)
- Route protected by middleware auth guard

**Tasks:**

- [ ] Install `@copilotkit/runtime`
- [ ] Create `app/api/copilot/route.ts` using `CopilotRuntime` + `OpenAIAdapter({ apiKey, endpoint, model: 'gpt-4o' })`
- [ ] Verify session in route handler before passing to CopilotRuntime
- [ ] Add `AZURE_OPENAI_API_KEY` and `AZURE_OPENAI_ENDPOINT` to `.env.example`
- [ ] Write integration test: valid session → mock CopilotRuntime returns canned stream; no session → 401

---

### Story 9.2 — `AIClient` moderation endpoint

**As a** platform operator,
**I want** `AIClient.moderate(content)` to screen tweet content before publish,
**so that** policy-violating content is blocked at the Server Action level.

**Acceptance Criteria:**

- `AIClient.moderate` sends `content` to Azure OpenAI moderation API (or a dedicated moderation endpoint)
- Returns `{ flagged: boolean; reason?: string }`
- Called inside `createTweetAction` after rate-limit check, before DB insert
- On `flagged: true`, action returns `{ error: 'Content policy violation', reason }`
- Moderation failure (network error) is non-blocking: logs warning, allows post (fail-open)

**Tasks:**

- [ ] Implement `AIClient.moderate(content: string): Promise<{ flagged: boolean; reason?: string }>` in `src/lib/sdk/AIClient.ts`
- [ ] Use Azure OpenAI Moderations endpoint or equivalent
- [ ] Add try/catch: on network error log warning via structured logger; return `{ flagged: false }` (fail-open)
- [ ] Integrate into `createTweetAction` between rate-limit and DB insert
- [ ] Write unit test: mock `AIClient.moderate` returning `flagged: true` → assert action returns error

---

### Story 9.3 — AI tweet suggestion endpoint

**As a** developer,
**I want** an API route providing tweet completion suggestions consumed by `CopilotTextarea`,
**so that** the AI suggestion feature is routed through the secured proxy.

**Acceptance Criteria:**

- `AIClient.getSuggestions(partialContent, context)` sends request to `/api/copilot` via the catch-all proxy
- Response schema validated with Zod before returning to the client
- Catch-all proxy route (`/api/[...path]/route.ts`) validates the request body and forwards to the appropriate SDK client

**Tasks:**

- [ ] Create `app/api/[...path]/route.ts` catch-all proxy handler
- [ ] Validate request body with `routePayloadSchema = z.object({ path: z.array(z.string()), body: z.unknown() })`
- [ ] Route to the correct SDK client method based on `path[0]` (e.g., `'copilot'` → `api.ai`)
- [ ] Implement `AIClient.getSuggestions(partial, context)` calling the LLM via proxy
- [ ] Write integration test: POST to `/api/copilot/suggest` → mock SDK returns suggestion → assert response shape

---

## Epic 10 — Analytics Data Layer

**Goal:** Aggregated analytics queries against the Aurora Read Replica powering the D3.js dashboard.

---

### Story 10.1 — Read-replica Prisma client

**As a** developer,
**I want** a separate Prisma client instance pointing at the Aurora Read Replica,
**so that** analytics queries do not compete with write traffic on the primary.

**Acceptance Criteria:**

- `src/lib/prismaReadReplica.ts` exports a singleton `prismaRead` pointing at `DATABASE_URL_REPLICA`
- All `AnalyticsService` methods use `prismaRead`, not `prisma`
- `DATABASE_URL_REPLICA` documented in `.env.example`

**Tasks:**

- [ ] Create `src/lib/prismaReadReplica.ts` with same singleton pattern as `prisma.ts` but using `DATABASE_URL_REPLICA`
- [ ] Add `DATABASE_URL_REPLICA` to `.env.example` (can equal `DATABASE_URL` in dev)
- [ ] Import `prismaRead` in all `AnalyticsService` methods

---

### Story 10.2 — AnalyticsService aggregations

**As a** developer,
**I want** `AnalyticsService` methods returning pre-aggregated data arrays for each chart,
**so that** the analytics RSC can fetch everything in parallel and pass it directly to D3.

**Acceptance Criteria:**

- `getEngagement(userId, days)` → `{ date: string; likes: number; retweets: number; replies: number }[]`
- `getFollowerGrowth(userId, days)` → `{ date: string; count: number }[]` (cumulative)
- `getPostingFrequency(userId, days)` → `{ date: string; count: number }[]`
- `getTopHashtags(userId, days)` → `{ tag: string; count: number }[]` (top 10)
- All queries run on `prismaRead`; all accept `days` parameter (default 30)

**Tasks:**

- [ ] Implement `getEngagement` with `prismaRead.$queryRaw`:
  ```sql
  SELECT DATE(l."createdAt") as date, COUNT(*) as likes FROM "Like" l
  JOIN "Tweet" t ON t.id = l."tweetId" WHERE t."authorId" = $1
  AND l."createdAt" >= NOW() - INTERVAL '$2 days' GROUP BY date
  ```
  (plus similar for retweets and replies; merge by date in TypeScript)
- [ ] Implement `getFollowerGrowth` with daily new-follower counts; compute cumulative sum in TypeScript
- [ ] Implement `getPostingFrequency`: count tweets grouped by `DATE(createdAt)` for the user
- [ ] Implement `getTopHashtags`: join `TweetHashtag` → `Hashtag`, group by tag, count, limit 10
- [ ] Create `app/(main)/analytics/page.tsx` RSC calling all 4 methods in `Promise.all`
- [ ] Write unit tests for each aggregation function with a seeded in-memory DB or mocked `prismaRead`

---

## Epic 11 — Caching Strategy (Server Side)

**Goal:** Implement Redis cache-aside for feed and profiles, and configure Next.js `revalidate` intervals for RSC data fetches.

---

### Story 11.1 — Redis cache-aside in FeedService and UserRepository

**As a** developer,
**I want** `FeedService.getHomeFeed` and `UserRepository.getByHandle` to check Redis before hitting PostgreSQL,
**so that** hot reads are served from memory and database load is minimized.

**Acceptance Criteria:**

- `getHomeFeed`: Redis key `feed:userId` → `LRANGE 0 {limit-1}` → hydrate; miss → DB query → `LPUSH`
- `getByHandle`: Redis key `profile:handle` → `GET` → JSON.parse; miss → DB query → `SET` with TTL 300s
- Both implement cache-aside: read → miss → load from DB → write to cache → return
- Both write unit tests covering hit and miss paths

**Tasks:**

- [ ] Implement cache-aside in `FeedService.getHomeFeed`:
  - [ ] `LRANGE feed:userId 0 {limit-1}` → if non-empty, `MGET` tweet details → return
  - [ ] Miss: `prisma.tweet.findMany()` → `LPUSH` IDs + `LTRIM` 0 499
- [ ] Implement cache-aside in `UserRepository.getByHandle`:
  - [ ] `redis.get(`profile:${handle}`)` → if hit, `JSON.parse` and return
  - [ ] Miss: `prisma.user.findUnique()` → `redis.set(`profile:${handle}`, JSON.stringify(user), 'EX', 300)`
- [ ] Write unit tests for both: mock Redis returning data (hit) and null (miss); assert DB called only on miss

---

### Story 11.2 — Next.js route segment revalidation

**As a** developer,
**I want** RSC route segments to declare `revalidate` intervals aligned with data freshness requirements,
**so that** Next.js data caching is consistent and predictable.

**Acceptance Criteria:**

- `home/page.tsx`: `export const revalidate = 60`
- `explore/page.tsx` (trending): `export const revalidate = 30`
- `[handle]/page.tsx`: `export const revalidate = 300`
- `analytics/page.tsx`: `export const revalidate = 300`
- All mutating Server Actions call appropriate `revalidatePath`/`revalidateTag`

**Tasks:**

- [ ] Add `export const revalidate = 60` to `app/(main)/home/page.tsx`
- [ ] Add `export const revalidate = 30` to `app/(main)/explore/page.tsx`
- [ ] Add `export const revalidate = 300` to `app/(main)/[handle]/page.tsx`
- [ ] Add `export const revalidate = 300` to `app/(main)/analytics/page.tsx`
- [ ] Audit all Server Actions: ensure `revalidatePath` is called after each mutation
- [ ] Add `revalidateTag('feed')` pattern to allow granular invalidation in future

---

## Epic 12 — Security & Rate Limiting

**Goal:** Harden all server surfaces against abuse, injection, information leakage, and edge-level attacks.

---

### Story 12.1 — Redis sliding-window rate limiter

**As a** developer,
**I want** a `RateLimitService` implementing a Redis sliding-window algorithm,
**so that** all Server Actions are protected against per-user abuse with precise limits.

**Acceptance Criteria:**

- `checkLimit(userId, action, limit, windowSecs)` → `{ allowed: boolean; retryAfter?: number }`
- Uses Redis `MULTI`/`EXEC` pipeline: `ZREMRANGEBYSCORE` (prune old), `ZADD` (add now), `ZCARD` (count), `EXPIRE`
- Per-action config: `createTweet: 10/min`, `follow: 30/min`, `search: 60/min`, `like: 100/min`
- Returns `retryAfter` (seconds until window clears) when `count > limit`

**Tasks:**

- [ ] Implement `RateLimitService.checkLimit` in `src/services/RateLimitService.ts`
- [ ] Define `RATE_LIMIT_CONFIG` object with per-action limits
- [ ] Integrate into: `createTweetAction`, `followUserAction`, `unfollowUserAction`, `likeTweetAction`, API search route
- [ ] Expose `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` headers in `middleware.ts` for API routes
- [ ] Write Vitest unit tests: under limit → allowed; at limit → allowed; over limit → `{ allowed: false, retryAfter }`; sliding window prunes old entries

---

### Story 12.2 — Input sanitisation and Zod enforcement

**As a** security engineer,
**I want** every Server Action to re-validate its input with Zod regardless of client validation,
**so that** the server never trusts client-provided data.

**Acceptance Criteria:**

- Every Server Action begins with `schema.safeParse(input)` — no exceptions
- `ZodError` returns `{ errors: result.error.flatten() }` — no stack traces leaked to the client
- `content` field sanitised via `.transform(stripHtml)` before any DB write
- No `console.error(internalError)` in production paths (use structured logger)
- Prisma's parameterised queries prevent SQL injection by design

**Tasks:**

- [ ] Implement `stripHtml(input: string): string` utility using `striptags` or regex-based approach
- [ ] Apply `.transform(stripHtml)` to `content` in `createTweetSchema`
- [ ] Audit all Server Actions (use `grep -rn "safeParse"` to verify coverage)
- [ ] Replace all `console.error` in action/service files with structured logger (`src/lib/logger.ts`)
- [ ] Create `src/lib/logger.ts` wrapping `pino` with production-safe log levels
- [ ] Write security-focused Vitest tests: XSS payload `<script>alert(1)</script>` → stripped from content; SQL meta-chars `'; DROP TABLE` → parameterised safely

---

### Story 12.3 — Security headers and CSP

**As a** security engineer,
**I want** HTTP security headers and a Content Security Policy configured in Next.js,
**so that** the application is protected against XSS, clickjacking, and MIME sniffing.

**Acceptance Criteria:**

- `next.config.ts` `headers()` function sets:
  - `Content-Security-Policy` (script-src, connect-src, img-src)
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY`
  - `Strict-Transport-Security: max-age=63072000; includeSubDomains`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy: camera=(), microphone=(), geolocation=()`
- CSP allows: own domain, Azure OpenAI endpoint, CopilotKit CDN
- Verified via Playwright asserting response headers on `/home`

**Tasks:**

- [ ] Add `headers()` export to `next.config.ts` with all 6 security headers
- [ ] Define CSP string with `'self'`, Azure OpenAI endpoint, CopilotKit endpoint in allow-lists
- [ ] Use `nonce`-based CSP for inline scripts if required by Next.js runtime
- [ ] Run CSP validator (`npx csp-evaluator` or similar) to confirm no unsafe directives
- [ ] Write Playwright test asserting `Content-Security-Policy` and `X-Frame-Options` present on authenticated pages

---

## Summary

| Epic                        | Stories        | Primary Tech                                    |
| --------------------------- | -------------- | ----------------------------------------------- |
| 1 — Setup & Scaffolding     | 4              | Prisma 7, ioredis, Custom SDK                   |
| 2 — Authentication          | 3              | NextAuth 5, Entra ID, KMS, Middleware           |
| 3 — Zod Schemas             | 2              | Zod, shared schemas                             |
| 4 — Core Tweet Actions      | 3              | Server Actions, Prisma, Redis                   |
| 5 — Profiles & Social Graph | 2              | Server Actions, Redis Sets                      |
| 6 — Feed Architecture       | 2              | FeedService, Redis Lists, fan-out               |
| 7 — Search & API Routes     | 2              | SearchService, PostgreSQL FTS, Redis Sorted Set |
| 8 — Real-time Notifications | 2              | SSE, Redis Pub/Sub, NotificationService         |
| 9 — AI Backend & SDK        | 3              | CopilotKit Runtime, Azure OpenAI, AIClient      |
| 10 — Analytics Data Layer   | 2              | AnalyticsService, Aurora Read Replica           |
| 11 — Caching (Server)       | 2              | Redis cache-aside, Next.js revalidate           |
| 12 — Security               | 3              | RateLimitService, Zod, CSP headers              |
| **Total**                   | **30 stories** |                                                 |
