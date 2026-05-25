# Twitter Clone — Testing Stories

> **Stack:** Vitest 4 · MSW 2 · Playwright · React Testing Library · @testing-library/user-event · @vitest/coverage-v8

> **Scope:** Full testing strategy across all three layers — unit tests (Vitest 4), integration tests (Vitest 4 + MSW 2), and end-to-end tests (Playwright). Each story maps to a testing concern that cuts across one or more features.
> Feature-level acceptance criteria are in the Frontend and Backend files. This file owns the test infrastructure and cross-cutting test coverage.

---

## Table of Contents

1. [Epic 1 — Test Infrastructure Setup](#epic-1--test-infrastructure-setup)
2. [Epic 2 — Unit Tests: Shared Schemas & Utilities](#epic-2--unit-tests-shared-schemas--utilities)
3. [Epic 3 — Unit Tests: Service Layer](#epic-3--unit-tests-service-layer)
4. [Epic 4 — Unit Tests: React Components (RTL)](#epic-4--unit-tests-react-components-rtl)
5. [Epic 5 — Unit Tests: D3 Data Transforms](#epic-5--unit-tests-d3-data-transforms)
6. [Epic 6 — Integration Tests: MSW 2 Handler Suite](#epic-6--integration-tests-msw-2-handler-suite)
7. [Epic 7 — Integration Tests: TanStack Query Hooks](#epic-7--integration-tests-tanstack-query-hooks)
8. [Epic 8 — Integration Tests: Server Actions](#epic-8--integration-tests-server-actions)
9. [Epic 9 — E2E Tests: Playwright Suite](#epic-9--e2e-tests-playwright-suite)
10. [Epic 10 — Coverage, CI Gates & Reporting](#epic-10--coverage-ci-gates--reporting)

---

## Epic 1 — Test Infrastructure Setup

**Goal:** Install and configure all testing tooling so the team can write tests immediately on a stable, repeatable foundation.

---

### Story 1.1 — Vitest 4 workspace configuration

**As a** developer,
**I want** Vitest 4 configured with separate environments for node code and browser code,
**so that** service/schema tests and component tests can both run correctly in CI.

**Acceptance Criteria:**

- `vitest.config.ts` defines two workspace projects: `node` (for services, schemas, SDK) and `jsdom` (for React components)
- `@vitest/coverage-v8` provider configured
- Global setup file registers MSW server lifecycle
- TypeScript path aliases (`@/`) resolved in tests
- `npm run test` runs all suites; `npm run test:unit` and `npm run test:integration` filter by project

**Tasks:**

- [ ] Install `vitest@4`, `@vitest/coverage-v8`, `happy-dom` (or `jsdom`)
- [ ] Create `vitest.config.ts` with `defineWorkspace` containing `node` and `jsdom` projects
- [ ] Configure `resolve.alias` to honour `@/` → `src/` path alias
- [ ] Create `src/test/setup.ts`: import `@testing-library/jest-dom/extend-expect` matchers
- [ ] Create `src/test/setup-msw.ts`: `beforeAll(server.listen)`, `afterEach(server.resetHandlers)`, `afterAll(server.close)`
- [ ] Add `test`, `test:unit`, `test:integration`, `test:coverage` scripts to `package.json`
- [ ] Verify a placeholder test runs in both environments: `expect(1 + 1).toBe(2)`

---

### Story 1.2 — MSW 2 mock handler infrastructure

**As a** developer,
**I want** MSW 2 handlers organized by domain and available for both Node (Vitest) and browser (dev/Storybook),
**so that** all integration tests intercept network calls consistently.

**Acceptance Criteria:**

- `src/mocks/handlers/` contains: `tweet.handlers.ts`, `user.handlers.ts`, `feed.handlers.ts`, `auth.handlers.ts`, `notifications.handlers.ts`, `ai.handlers.ts`
- `src/mocks/node.ts` exports `server = setupServer(...allHandlers)`
- `src/mocks/browser.ts` exports `worker = setupWorker(...allHandlers)`
- Each handler file exports typed MSW 2 `http.*` handlers
- Handler responses match the exact shape returned by the real API routes

**Tasks:**

- [ ] Install `msw@2`
- [ ] Create `src/mocks/handlers/tweet.handlers.ts`:
  - `http.post('/api/tweets', ...)` → `HttpResponse.json(mockTweet)`
  - `http.delete('/api/tweets/:id', ...)` → `HttpResponse.json({ success: true })`
  - `http.post('/api/tweets/:id/like', ...)` → `HttpResponse.json({ likeCount: 1 })`
- [ ] Create `src/mocks/handlers/feed.handlers.ts`:
  - `http.get('/api/feed', ...)` → paginated `HttpResponse.json({ tweets: mockTweets, nextCursor })`
- [ ] Create `src/mocks/handlers/user.handlers.ts`:
  - `http.get('/api/users/:handle', ...)`, `http.post('/api/users/:handle/follow', ...)`
- [ ] Create `src/mocks/handlers/auth.handlers.ts`:
  - `http.get('/api/auth/session', ...)` → `HttpResponse.json(mockSession)`
- [ ] Create `src/mocks/handlers/notifications.handlers.ts`, `ai.handlers.ts`
- [ ] Create `src/mocks/node.ts` and `src/mocks/browser.ts`
- [ ] Create `src/mocks/data/` with factory functions: `mockTweet()`, `mockUser()`, `mockNotification()`

---

### Story 1.3 — Playwright configuration

**As a** QA engineer,
**I want** Playwright configured with three browser projects and a global setup that seeds test data,
**so that** E2E tests run reliably across browsers with consistent starting state.

**Acceptance Criteria:**

- `playwright.config.ts` defines projects: Chromium, Firefox, WebKit
- `baseURL` points to `http://localhost:3000` (or `PLAYWRIGHT_BASE_URL` env override)
- `globalSetup` script seeds test DB and saves auth state (`storageState`)
- `globalTeardown` script cleans test DB
- `npm run test:e2e` runs all Playwright specs; `npm run test:e2e:chromium` runs single-browser

**Tasks:**

- [ ] Install `@playwright/test` and run `playwright install --with-deps`
- [ ] Create `playwright.config.ts` with 3 browser projects and `webServer` config launching `next dev`
- [ ] Create `tests/global-setup.ts`:
  - Seed DB: `prisma.user.createMany(testUsers)`, `prisma.tweet.createMany(testTweets)`
  - Mock OAuth callback in test env to skip real Entra ID
  - Save session cookie to `tests/.auth/user.json` via `browser.newContext().storageState()`
- [ ] Create `tests/global-teardown.ts`: `prisma.tweet.deleteMany()`, `prisma.user.deleteMany()` for test data
- [ ] Create `tests/fixtures.ts` with `test.extend` providing `authenticatedPage` fixture using `storageState`
- [ ] Add `test:e2e`, `test:e2e:chromium`, `test:e2e:headed` scripts to `package.json`

---

### Story 1.4 — Mock factories and shared test utilities

**As a** developer,
**I want** a library of typed mock data factories,
**so that** every test can create realistic test data in one line without duplicating object shapes.

**Acceptance Criteria:**

- `mockTweet(overrides?)` returns a valid `TweetWithAuthor` object
- `mockUser(overrides?)` returns a valid `UserPublic` object
- `mockNotification(overrides?)` returns a valid `NotificationPayload` object
- `mockSession(overrides?)` returns a valid NextAuth `Session` object
- All factories use `faker` for realistic data; all fields match Prisma model types

**Tasks:**

- [ ] Install `@faker-js/faker`
- [ ] Create `src/test/factories/tweet.factory.ts` with `mockTweet(overrides?)`
- [ ] Create `src/test/factories/user.factory.ts` with `mockUser(overrides?)`
- [ ] Create `src/test/factories/notification.factory.ts` with `mockNotification(overrides?)`
- [ ] Create `src/test/factories/session.factory.ts` with `mockSession(overrides?)`
- [ ] Export all from `src/test/factories/index.ts`
- [ ] Write a sanity test asserting factories produce valid Zod-parseable output

---

## Epic 2 — Unit Tests: Shared Schemas & Utilities

**Goal:** Achieve 100% line coverage on all Zod schemas and utility functions — these are the most critical unit tests.

---

### Story 2.1 — `createTweetSchema` unit tests

**As a** developer,
**I want** comprehensive unit tests for `createTweetSchema`,
**so that** every validation rule is verified and regressions are caught immediately.

**Acceptance Criteria:**

- Valid input passes without errors
- `content` empty string → error
- `content` > 280 characters → error with correct message
- `content` exactly 280 characters → passes
- `content` with HTML tags → tags stripped by `.transform`
- `parentId` not a valid cuid2 → error
- `hashtags` array with 6 items → error; 5 items → passes

**Tasks:**

- [ ] Create `src/shared/schemas/__tests__/tweet.schema.test.ts`
- [ ] Write test cases for all rules above using `Vitest` `it.each` for boundary values
- [ ] Assert `.safeParse(invalid).success === false` and `issues` contains expected error code
- [ ] Assert `.safeParse(validInput).data.content` has no HTML tags after transform
- [ ] Assert exactly 280-char string passes (fence-post test)

---

### Story 2.2 — `updateProfileSchema` and `searchQuerySchema` unit tests

**As a** developer,
**I want** unit tests for user and search schemas,
**so that** all profile update and search input rules are verified.

**Acceptance Criteria:**

- `displayName` empty → error; exactly 50 chars → passes; 51 chars → error
- `bio` > 160 chars → error; exactly 160 → passes
- `avatarUrl` not a URL → error; valid HTTPS URL → passes
- `searchQuerySchema` with `q` of 1 char → error; 2 chars → passes; `type` unknown value → error

**Tasks:**

- [ ] Create `src/shared/schemas/__tests__/user.schema.test.ts` covering all `updateProfileSchema` and `followSchema` rules
- [ ] Create `src/shared/schemas/__tests__/search.schema.test.ts` covering `searchQuerySchema` rules
- [ ] Use `it.each` for boundary value analysis on every field with min/max

---

### Story 2.3 — Utility function unit tests

**As a** developer,
**I want** unit tests for all utility functions (`stripHtml`, `generateHandle`, `formatRelativeTime`),
**so that** shared helpers are provably correct.

**Acceptance Criteria:**

- `stripHtml('<script>alert(1)</script>text')` → `'text'`
- `stripHtml('<b>bold</b>')` → `'bold'`
- `generateHandle('John Doe')` → `'johndoe'` or `'johndoe-a1b2'` on collision
- `formatRelativeTime(date)` returns `'just now'`, `'2m ago'`, `'3h ago'`, `'Jan 5'` correctly

**Tasks:**

- [ ] Create `src/lib/__tests__/stripHtml.test.ts`: XSS payloads, nested tags, plain text (no change)
- [ ] Create `src/lib/__tests__/generateHandle.test.ts`: spaces, accents, duplicates, max length
- [ ] Create `src/lib/__tests__/formatRelativeTime.test.ts`: < 60s, 60s–59m, 1h–23h, ≥ 1 day
- [ ] Use `vi.useFakeTimers()` for deterministic time in `formatRelativeTime` tests

---

## Epic 3 — Unit Tests: Service Layer

**Goal:** Unit-test all service methods with mocked Redis and Prisma, verifying business logic in isolation.

---

### Story 3.1 — `FeedService` unit tests

**As a** developer,
**I want** unit tests for `FeedService.getHomeFeed` and `FeedService.fanOut`,
**so that** the feed's Redis-first caching logic and fan-out algorithm are provably correct.

**Acceptance Criteria:**

- `getHomeFeed` — Redis hit: DB not called; returns deserialized tweets
- `getHomeFeed` — Redis miss: DB called; Redis `LPUSH` + `LTRIM` called to warm cache
- `fanOut` — ≤ 10k followers: `LPUSH` + `LTRIM` called for each follower in pipeline
- `fanOut` — > 10k followers: job enqueued to `queue:fanout`; individual `LPUSH` not called

**Tasks:**

- [ ] Create `src/services/__tests__/FeedService.test.ts`
- [ ] Mock `ioredis` using `vi.mock('@/lib/redis')` + `vi.fn()` for each command
- [ ] Mock `@/lib/prisma` using `vi.mock` with `prisma.tweet.findMany` returning `mockTweets`
- [ ] Test `getHomeFeed` hit path: `redis.lrange` returns IDs → assert `prisma.findMany` not called
- [ ] Test `getHomeFeed` miss path: `redis.lrange` returns `[]` → assert `prisma.findMany` called + `redis.lpush` called
- [ ] Test `fanOut` with 5 followers: assert pipeline called 5× with correct keys
- [ ] Test `fanOut` with 10,001 followers: assert `redis.rpush('queue:fanout', ...)` called; `lpush` not called

---

### Story 3.2 — `RateLimitService` unit tests

**As a** developer,
**I want** unit tests for the sliding-window rate limiter,
**so that** the algorithm's allow/deny logic and window sliding are provably correct.

**Acceptance Criteria:**

- First call in window → `{ allowed: true }`
- Calls up to limit → all `{ allowed: true }`
- Call exceeding limit → `{ allowed: false, retryAfter: number }`
- After window expires (old entries pruned) → `{ allowed: true }` again
- Redis pipeline calls made in correct order: `ZREMRANGEBYSCORE`, `ZADD`, `ZCARD`, `EXPIRE`

**Tasks:**

- [ ] Create `src/services/__tests__/RateLimitService.test.ts`
- [ ] Mock `ioredis` `multi().exec()` return values for each scenario
- [ ] Use `vi.useFakeTimers()` to control current time for window sliding tests
- [ ] Write 5 test cases covering: under limit, at limit, over limit, window resets, pipeline order

---

### Story 3.3 — `NotificationService` unit tests

**As a** developer,
**I want** unit tests for `NotificationService.create` and `markAllRead`,
**so that** notification creation and read-state logic are verified independently of the DB.

**Acceptance Criteria:**

- `create` with `actorId === recipientId` → no DB insert, no Redis publish (self-notification guard)
- `create` with different actor/recipient → Prisma `create` called + Redis `publish` called + `incr` called
- `markAllRead` → Prisma `updateMany` called with `{ where: { recipientId }, data: { read: true } }` + `DEL notif:unread:userId`

**Tasks:**

- [ ] Create `src/services/__tests__/NotificationService.test.ts`
- [ ] Mock `prisma.notification.create` and `prisma.notification.updateMany`
- [ ] Mock `redis.publish` and `redis.incr` and `redis.del`
- [ ] Test self-notification guard: assert no mocked functions called
- [ ] Test valid notification: assert all 3 side effects called with correct arguments
- [ ] Test `markAllRead`: assert `updateMany` + `del` called; `publish` not called

---

### Story 3.4 — `SearchService` unit tests

**As a** developer,
**I want** unit tests for `SearchService` tweet search, user search, and trending,
**so that** search business logic is verified without a real database.

**Acceptance Criteria:**

- `searchTweets` → calls `prisma.$queryRaw` with correct SQL template and parameters
- `searchUsers` → calls `prisma.user.findMany` with `ILIKE` conditions
- `getTrending` — Redis hit: returns cached `trending:top10`; DB not called
- `getTrending` — Redis miss: calls `ZREVRANGE`, prunes stale entries, sets `trending:top10` cache

**Tasks:**

- [ ] Create `src/services/__tests__/SearchService.test.ts`
- [ ] Mock `prisma.$queryRaw` returning a typed array of tweets
- [ ] Assert SQL template string contains `to_tsvector` and `plainto_tsquery`
- [ ] Test `getTrending` Redis hit: mock `redis.get('trending:top10')` returning JSON → assert `ZREVRANGE` not called
- [ ] Test `getTrending` Redis miss: mock `redis.get` returning null → assert `ZREVRANGE` called → cache SET called

---

## Epic 4 — Unit Tests: React Components (RTL)

**Goal:** Unit-test all key React components using React Testing Library, verifying rendering, user interactions, and accessibility.

---

### Story 4.1 — `TweetComposer` component tests

**As a** developer,
**I want** RTL tests for `TweetComposer`,
**so that** form validation, character counter, and submission flow are verified.

**Acceptance Criteria:**

- Empty content → submit button disabled
- Content > 280 chars → character counter turns red, submit button disabled, error message visible
- Exactly 280 chars → counter green, button enabled
- Valid content submit → `createTweetAction` called with correct payload
- Server Action error returned → inline error message rendered
- Loading spinner visible during pending action

**Tasks:**

- [ ] Create `src/components/__tests__/TweetComposer.test.tsx`
- [ ] Mock `createTweetAction` with `vi.mock('@/app/(main)/home/actions')`
- [ ] Wrap component in `QueryClientProvider` and `SessionProvider` test wrappers
- [ ] Use `userEvent.type(textarea, '...')` to simulate input
- [ ] Assert `screen.getByRole('button', { name: /tweet/i })` disabled/enabled states
- [ ] Assert `screen.getByText(/280/)` character counter colour class changes
- [ ] Assert action called with `content` after valid submit
- [ ] Simulate action returning `{ error: 'Rate limit' }` → assert error message visible

---

### Story 4.2 — `TweetCard` and `TweetActions` tests

**As a** developer,
**I want** RTL tests for `TweetCard` and `TweetActions`,
**so that** tweet display and optimistic like/retweet interactions are verified.

**Acceptance Criteria:**

- `TweetCard` renders handle, display name, content, and relative timestamp
- `TweetActions` like button click → optimistic count +1; re-renders with new count
- `TweetActions` on action error → count rolled back to original
- Delete option visible only for own tweets; hidden for others

**Tasks:**

- [ ] Create `src/components/__tests__/TweetCard.test.tsx` rendering with `mockTweet()`
- [ ] Create `src/components/__tests__/TweetActions.test.tsx` with MSW handler for like endpoint
- [ ] Mock `useSession` to return `mockSession({ user: { id: mockTweet().authorId } })` for own-tweet test
- [ ] Assert like count increments on click before server response (optimistic)
- [ ] Mock MSW to return 500 → assert count reverts to original

---

### Story 4.3 — `NotificationBell` and `useSSE` hook tests

**As a** developer,
**I want** RTL tests for `NotificationBell` and a Vitest test for `useSSE`,
**so that** real-time notification delivery to the UI is verified.

**Acceptance Criteria:**

- `useSSE` test: mock `EventSource` class → assert `onMessage` called on `message` event; `close()` called on unmount
- `NotificationBell` test: simulate SSE message → assert badge count increments
- `NotificationBell` test: click "Mark all read" → assert badge cleared

**Tasks:**

- [ ] Create `src/hooks/__tests__/useSSE.test.ts` (node environment, no DOM)
- [ ] Mock global `EventSource` constructor with `vi.fn()` returning mock event emitter
- [ ] Create `src/components/__tests__/NotificationBell.test.tsx`
- [ ] Simulate SSE by directly calling the `onMessage` callback with a mock notification payload
- [ ] Assert badge `aria-label` or text content increments correctly
- [ ] Click "Mark all read" → assert `markAllReadAction` called + badge text is `0` or hidden

---

### Story 4.4 — `FollowButton` and `EditProfileModal` tests

**As a** developer,
**I want** RTL tests for follow interaction and edit profile form,
**so that** social graph mutations and form validation are verified in the UI layer.

**Acceptance Criteria:**

- `FollowButton` starts as "Follow"; click → optimistic "Following" state
- `FollowButton` on server error → reverts to "Follow"
- `EditProfileModal` renders with pre-filled current values
- `bio` > 160 chars → error shown, submit button disabled
- Valid submit → `updateProfileAction` called with correct payload

**Tasks:**

- [ ] Create `src/components/__tests__/FollowButton.test.tsx`
- [ ] Mock MSW follow handler: success path + 429 error path
- [ ] Create `src/components/__tests__/EditProfileModal.test.tsx`
- [ ] Use `userEvent.type` to simulate editing bio field
- [ ] Assert inline error visible for > 160 chars
- [ ] Assert action called with trimmed, valid payload on valid submit

---

## Epic 5 — Unit Tests: D3 Data Transforms

**Goal:** Test all D3 data transformation helpers in the Node environment — no DOM, no SVG rendering.

---

### Story 5.1 — Engagement and follower growth data transforms

**As a** developer,
**I want** unit tests for `prepareEngagementData` and `prepareGrowthData`,
**so that** the raw database rows are correctly shaped for D3 charts.

**Acceptance Criteria:**

- `prepareEngagementData(rows)` merges likes/retweets/replies by date into a single array sorted by date
- Missing dates filled with zero values (no gaps in chart)
- `prepareGrowthData(rows)` returns cumulative sum correctly
- Edge cases: empty array → `[]`; single row → returns that row

**Tasks:**

- [ ] Create `src/components/analytics/__tests__/prepareEngagementData.test.ts`
- [ ] Test merge logic: rows on same date summed correctly
- [ ] Test date gap filling: 3-day window with middle day missing → 3 entries, middle has zeros
- [ ] Create `src/components/analytics/__tests__/prepareGrowthData.test.ts`
- [ ] Test cumulative sum: `[1, 2, 3]` → `[1, 3, 6]`
- [ ] Test empty input → `[]`

---

### Story 5.2 — Bubble chart and heatmap data transforms

**As a** developer,
**I want** unit tests for `packData` and `heatmapData`,
**so that** D3 pack and heatmap layouts receive correctly shaped input.

**Acceptance Criteria:**

- `packData(hashtags)` returns `{ name: string; value: number }[]` sorted by value descending
- `heatmapData(postings)` returns a 52×7 grid with `count` and `date` per cell; missing dates = 0
- Edge cases: empty input → empty / zero-filled grid

**Tasks:**

- [ ] Create `src/components/analytics/__tests__/packData.test.ts`
- [ ] Assert output sorted by `value` descending
- [ ] Assert each item has `name` and `value` fields
- [ ] Create `src/components/analytics/__tests__/heatmapData.test.ts`
- [ ] Assert grid dimensions: `result.length === 52`, `result[0].length === 7`
- [ ] Assert a seeded posting on a specific date appears in the correct grid cell

---

## Epic 6 — Integration Tests: MSW 2 Handler Suite

**Goal:** Verify that all API route handlers return the correct shapes and status codes, intercepted by MSW.

---

### Story 6.1 — Feed API integration tests

**As a** developer,
**I want** integration tests for the `/api/feed` route,
**so that** pagination, cursor handling, and response shape are verified end-to-end.

**Acceptance Criteria:**

- First page request (no cursor) → returns 20 tweets + `nextCursor`
- Second page request (with cursor) → returns next 20 tweets (different from first page)
- `limit` query param respected
- Invalid `cursor` → returns 400 with Zod error
- Unauthenticated request → returns 401

**Tasks:**

- [ ] Create `src/app/api/feed/__tests__/feed.route.test.ts` (Vitest, node env)
- [ ] Use MSW `feed.handlers.ts` to intercept Redis/DB calls
- [ ] Test first page: assert 20 items, `nextCursor` present
- [ ] Test pagination: cursor from page 1 → page 2 items different from page 1
- [ ] Test invalid params: `limit=999` → 400; no auth → 401
- [ ] Assert response body matches `FeedResponseSchema` (Zod parse)

---

### Story 6.2 — Search API integration tests

**As a** developer,
**I want** integration tests for the `/api/search` route,
**so that** search query validation and result shapes are verified.

**Acceptance Criteria:**

- `?q=hello&type=tweets` → returns tweet results array
- `?q=h&type=tweets` → 400 (q too short, min 2)
- `?q=hello&type=unknown` → 400 (invalid enum)
- `?q=hello&type=users` → returns user results array
- Unauthenticated → 401

**Tasks:**

- [ ] Create `src/app/api/search/__tests__/search.route.test.ts`
- [ ] Mock `SearchService.searchTweets` and `SearchService.searchUsers` with `vi.mock`
- [ ] Test each validation failure scenario with MSW capturing the request
- [ ] Assert correct service method called based on `type` param
- [ ] Assert result shapes match `TweetWithAuthor[]` and `UserPublic[]` types

---

### Story 6.3 — Notifications API integration tests

**As a** developer,
**I want** integration tests for `/api/notifications` and the SSE `/api/stream` route,
**so that** notification delivery plumbing is verified without real Redis connections.

**Acceptance Criteria:**

- `GET /api/notifications` → returns last 20 notifications for session user
- `GET /api/notifications` unauthenticated → 401
- `GET /api/stream` → opens SSE connection; sends `data: {...}\n\n` on Redis message
- `GET /api/stream` unauthenticated → 401
- Abort signal on SSE route → Redis unsubscribed (mock assert)

**Tasks:**

- [ ] Create `src/app/api/notifications/__tests__/notifications.route.test.ts`
- [ ] Mock `prisma.notification.findMany` returning 20 mocked notifications
- [ ] Create `src/app/api/stream/__tests__/stream.route.test.ts`
- [ ] Mock `createSubscriberClient()` with a mock EventEmitter
- [ ] Assert SSE response headers: `Content-Type: text/event-stream`
- [ ] Emit mock Redis message → read stream and assert `data:` line received
- [ ] Abort `AbortController` → assert `unsubscribe()` called on mock Redis client

---

## Epic 7 — Integration Tests: TanStack Query Hooks

**Goal:** Test custom hooks and components that use TanStack Query, verifying fetch, cache, and optimistic update behaviour.

---

### Story 7.1 — `useInfiniteQuery` feed hook tests

**As a** developer,
**I want** integration tests for the infinite feed query,
**so that** pagination, loading states, and end-of-feed detection are verified.

**Acceptance Criteria:**

- First render → `isLoading: true` → MSW responds → tweets rendered
- Trigger `fetchNextPage` → MSW responds with page 2 → all tweets from both pages rendered
- MSW returns `nextCursor: null` on page 2 → `hasNextPage: false`
- Network error → `isError: true`; existing cache preserved

**Tasks:**

- [ ] Create `src/hooks/__tests__/useFeed.test.tsx` using `renderHook` inside `QueryClientProvider`
- [ ] Use MSW `feed.handlers.ts` for mock responses
- [ ] Call `result.current.fetchNextPage()` and await re-render
- [ ] Assert `result.current.data.pages.length === 2`
- [ ] Override MSW handler to return 500 → assert `result.current.isError === true`

---

### Story 7.2 — Optimistic mutation tests (like and follow)

**As a** developer,
**I want** integration tests for optimistic like and follow mutations,
**so that** the optimistic update and rollback logic are verified in the full query-cache context.

**Acceptance Criteria:**

- Like: before server responds → like count in cache is `original + 1`
- Like: server returns 500 → like count rolled back to `original`
- Follow: before server responds → `isFollowing: true` in cache
- Follow: server returns 500 → `isFollowing: false` rolled back

**Tasks:**

- [ ] Create `src/hooks/__tests__/useLikeTweet.test.tsx`
- [ ] Seed `queryClient` cache with `mockTweet({ likeCount: 5 })`
- [ ] Call `mutate(tweetId)` — before MSW responds → assert cache shows `likeCount: 6`
- [ ] MSW handler returns 500 → await error → assert cache shows `likeCount: 5`
- [ ] Create `src/hooks/__tests__/useFollowUser.test.tsx` with same pattern for `isFollowing`

---

## Epic 8 — Integration Tests: Server Actions

**Goal:** Test each Server Action with real Zod validation, mocked Prisma, and mocked Redis, verifying the full action lifecycle.

---

### Story 8.1 — `createTweetAction` integration tests

**As a** developer,
**I want** integration tests for `createTweetAction`,
**so that** the full validation → auth → rate-limit → moderation → persist pipeline is verified.

**Acceptance Criteria:**

- Valid input + valid session → Prisma `tweet.create` called; Redis cache invalidated; returns `{ success: true }`
- Content > 280 chars → returns `{ errors: { content: [...] } }` without calling Prisma
- No session → returns `{ error: 'Unauthorized' }` without calling Prisma
- Rate limit exceeded (mock) → returns `{ error: 'Rate limit', retryAfter: 30 }` without calling Prisma
- `AIClient.moderate` returns `flagged: true` → returns `{ error: 'Content policy violation' }` without calling Prisma

**Tasks:**

- [ ] Create `src/app/(main)/home/__tests__/actions.test.ts` (Vitest, node env)
- [ ] Mock `@/lib/prisma` with `vi.mock`; mock `@/lib/redis` with `vi.mock`
- [ ] Mock `next-auth` `getServerSession` returning `mockSession()`
- [ ] Mock `RateLimitService.checkLimit`: return `{ allowed: true }` (default) or `{ allowed: false, retryAfter: 30 }`
- [ ] Mock `AIClient.moderate`: return `{ flagged: false }` (default) or `{ flagged: true, reason: '...' }`
- [ ] Assert Prisma `create` call args include `content`, `authorId`, hashtag records
- [ ] Assert `redis.del` called with `feed:*` pattern after successful create

---

### Story 8.2 — `followUserAction` and `updateProfileAction` integration tests

**As a** developer,
**I want** integration tests for follow and profile update actions,
**so that** ownership checks, self-follow guards, and Redis side effects are verified.

**Acceptance Criteria:**

- `followUserAction` — self-follow → `{ error: 'Cannot follow yourself' }`
- `followUserAction` — valid → Prisma `create` called + Redis `sadd` called + notification published
- `updateProfileAction` — wrong user → `{ error: 'Forbidden' }`
- `updateProfileAction` — valid → Prisma `update` called + Redis `del profile:handle` called

**Tasks:**

- [ ] Create `src/app/(main)/[handle]/__tests__/actions.test.ts`
- [ ] Test self-follow: mock session with `userId = targetUserId` → assert Prisma not called
- [ ] Test valid follow: assert `prisma.follow.create` + `redis.sadd` + `redis.publish` all called
- [ ] Test wrong-user profile update: mock session with different userId → assert `{ error: 'Forbidden' }`
- [ ] Test valid profile update: assert `prisma.user.update` + `redis.del` called

---

## Epic 9 — E2E Tests: Playwright Suite

**Goal:** Verify critical user journeys across the full stack in real browsers, using seeded data and mocked OAuth.

---

### Story 9.1 — Authentication E2E tests

**As a** QA engineer,
**I want** Playwright E2E tests for the authentication flow,
**so that** sign-in, session persistence, and sign-out are verified in a real browser.

**Acceptance Criteria:**

- Unauthenticated visit to `/home` → redirected to `/login`
- "Sign in with Microsoft" triggers OAuth (mocked callback in test env) → redirected to `/home`
- Authenticated user's handle visible in sidebar nav
- Sign-out → redirected to `/login`; session cookie cleared

**Tasks:**

- [ ] Create `tests/auth.spec.ts`
- [ ] Test 1: `page.goto('/home')` → assert URL is `/login`
- [ ] Test 2: `page.click('[data-testid="signin-button"]')` → mock OAuth callback → assert URL is `/home`
- [ ] Test 3: assert `page.getByText('@testuser')` visible in nav
- [ ] Test 4: click sign-out → assert URL is `/login` → `page.goto('/home')` → assert redirect back to `/login`
- [ ] Run across all 3 browser projects

---

### Story 9.2 — Tweet CRUD E2E tests

**As a** QA engineer,
**I want** Playwright E2E tests for composing, liking, retweeting, and deleting tweets,
**so that** the core tweet lifecycle is verified end-to-end.

**Acceptance Criteria:**

- Compose tweet: type content → click "Tweet" → assert tweet appears at top of feed
- Like tweet: click like button → assert count increments; click again → assert count decrements
- Retweet: click retweet → assert retweet indicator shown on tweet
- Delete own tweet: click kebab → "Delete" → confirm → assert tweet removed from feed
- Cannot delete other user's tweet (delete option not shown)

**Tasks:**

- [ ] Create `tests/tweet.spec.ts` using `authenticatedPage` fixture
- [ ] Compose: `fill('[data-testid="tweet-composer"]', 'E2E test tweet')` → `click('[data-testid="submit-tweet"]')` → assert tweet visible
- [ ] Like: `click('[data-testid="like-button"]')` → assert `like-count` text changed to `1`
- [ ] Retweet: `click('[data-testid="retweet-button"]')` → assert `data-retweeted="true"` on button
- [ ] Delete: assert delete option only visible on seeded own-user tweet; click → confirm → assert tweet gone
- [ ] Assert with seeded other-user tweet: delete option not present

---

### Story 9.3 — Feed infinite scroll E2E tests

**As a** QA engineer,
**I want** Playwright E2E tests for infinite scroll pagination,
**so that** the `IntersectionObserver`-driven load-more mechanism is verified in a real browser.

**Acceptance Criteria:**

- Initial load: first 20 tweets visible
- Scroll to bottom → loading spinner appears → 20 more tweets appended
- Total visible tweet count = 40
- When no more pages: "You're all caught up" message visible, spinner not shown

**Tasks:**

- [ ] Create `tests/feed.spec.ts` using `authenticatedPage` fixture
- [ ] Seed 45 tweets for test user's followees in `global-setup.ts`
- [ ] Assert `page.locator('[data-testid="tweet-card"]').count()` equals 20 on initial load
- [ ] `page.evaluate(() => window.scrollTo(0, document.body.scrollHeight))` → wait for network idle
- [ ] Assert tweet card count equals 40
- [ ] Scroll again (only 5 remaining) → assert count equals 45 + "You're all caught up" visible

---

### Story 9.4 — Search and profile E2E tests

**As a** QA engineer,
**I want** Playwright E2E tests for the search and profile journeys,
**so that** search results, user navigation, and follow interaction are verified.

**Acceptance Criteria:**

- Type ≥ 2 chars in search bar → results appear within 1 second
- Click tweet result → navigated to tweet thread page
- Switch to "Users" tab → user results appear
- Click user result → navigated to profile page
- Click "Follow" → button changes to "Following"; refresh page → still "Following"

**Tasks:**

- [ ] Create `tests/search.spec.ts` using `authenticatedPage` fixture
- [ ] `page.fill('[data-testid="search-input"]', 'E2E')` → wait for results → assert `tweet-card` visible
- [ ] Click tweet card → assert URL matches `/tweet/[id]`
- [ ] Tab to "Users" → assert `user-card` visible
- [ ] Click user card → assert URL matches `/[handle]`
- [ ] Click Follow → assert button text "Following" → `page.reload()` → assert button still "Following"
- [ ] Create `tests/profile.spec.ts`: visit seeded user's profile → assert tweet list, follower count, edit profile unavailable

---

### Story 9.5 — Notification real-time E2E tests

**As a** QA engineer,
**I want** Playwright E2E tests for notification delivery,
**so that** the SSE → badge update → dropdown flow is verified in a real browser.

**Acceptance Criteria:**

- User A likes User B's tweet → User B's notification bell badge increments without page refresh
- User B opens notification dropdown → sees the like notification
- User B clicks "Mark all read" → badge disappears

**Tasks:**

- [ ] Create `tests/notifications.spec.ts`
- [ ] Open two browser contexts: User A (`authenticatedPage`) and User B (`page2 = await context2.newPage()`)
- [ ] User B navigates to `/home`; User A likes User B's tweet via API call or UI interaction
- [ ] Assert User B's `[data-testid="notif-badge"]` shows `1` (poll with `waitFor`)
- [ ] User B clicks bell → assert notification item visible containing User A's handle
- [ ] User B clicks "Mark all read" → assert badge hidden/`0`

---

## Epic 10 — Coverage, CI Gates & Reporting

**Goal:** Enforce coverage thresholds and integrate test results into CI as mandatory quality gates.

---

### Story 10.1 — Coverage thresholds and reporting

**As a** tech lead,
**I want** minimum coverage thresholds enforced by Vitest,
**so that** untested code cannot be merged to `main`.

**Acceptance Criteria:**

- `src/services/` → 80% lines, 80% branches
- `src/shared/schemas/` → 100% lines
- `src/lib/` → 80% lines
- `src/components/` → 70% lines
- Coverage report generated in both `text` (CI) and `html` (artifact) formats
- `npm run test:coverage` exits with non-zero code if any threshold is violated

**Tasks:**

- [ ] Configure `coverage` in `vitest.config.ts`: `provider: 'v8'`, `reporter: ['text', 'html', 'json-summary']`, `thresholds: { ... }`
- [ ] Set per-folder thresholds using `include` globs
- [ ] Add `test:coverage` script running `vitest run --coverage`
- [ ] Output HTML report to `coverage/` directory (gitignored)
- [ ] Verify: remove a test → coverage drops below threshold → command exits with code 1

---

### Story 10.2 — CI quality gates

**As a** developer,
**I want** GitHub Actions to enforce test and coverage gates on every PR,
**so that** no failing or uncovered code can be merged.

**Acceptance Criteria:**

- PR branch protection requires `lint-typecheck` and `unit-tests` jobs to pass
- `unit-tests` job runs `test:coverage` and uploads `coverage/` as a GitHub Actions artifact
- E2E tests run in CI against a staging deployment (not blocking PRs, but blocking production deploys)
- Test results posted as PR comment via `github-actions-test-reporter` or similar

**Tasks:**

- [ ] Add `unit-tests` job to `.github/workflows/ci.yml`: `npm run test:coverage`
- [ ] Add `upload-artifact` step saving `coverage/html` and `coverage/json-summary`
- [ ] Add `download-artifact` + coverage comment step on PR using `coverage/json-summary`
- [ ] Configure GitHub branch protection rules: require `lint-typecheck` + `unit-tests` for `main`
- [ ] Add `test:e2e` job to CI workflow that runs after staging deploy (separate job, not blocking PR merge)

---

## Summary

| Epic                            | Stories        | Test Layer                                                  |
| ------------------------------- | -------------- | ----------------------------------------------------------- |
| 1 — Infrastructure Setup        | 4              | Vitest config, MSW setup, Playwright config, mock factories |
| 2 — Schema & Utility Unit Tests | 3              | Vitest unit (node)                                          |
| 3 — Service Layer Unit Tests    | 4              | Vitest unit (node), mocked Redis + Prisma                   |
| 4 — React Component Unit Tests  | 4              | Vitest + RTL (jsdom)                                        |
| 5 — D3 Transform Unit Tests     | 2              | Vitest unit (node)                                          |
| 6 — MSW Integration Tests       | 3              | Vitest integration, MSW 2                                   |
| 7 — TanStack Query Integration  | 2              | Vitest + RTL + MSW                                          |
| 8 — Server Action Integration   | 2              | Vitest integration (node)                                   |
| 9 — Playwright E2E Suite        | 5              | Playwright (Chromium + FF + WebKit)                         |
| 10 — Coverage & CI Gates        | 2              | @vitest/coverage-v8, GitHub Actions                         |
| **Total**                       | **31 stories** |                                                             |

---

## Test Coverage Target Map

| Area                        | Target Coverage         | Test Layer             |
| --------------------------- | ----------------------- | ---------------------- |
| `src/shared/schemas/`       | 100% lines              | Unit                   |
| `src/services/`             | 80% lines, 80% branches | Unit                   |
| `src/lib/` (utilities, SDK) | 80% lines               | Unit                   |
| `src/components/`           | 70% lines               | RTL unit + integration |
| Server Actions              | 80% lines               | Integration (Vitest)   |
| Critical journeys           | 100% paths              | E2E (Playwright)       |
