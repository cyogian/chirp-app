# Twitter Clone — Frontend Stories

> **Stack:** React 19 · Next.js 16 App Router · Radix UI · Tailwind CSS 4 · TanStack Query · react-hook-form · D3.js · CopilotKit v2 · Zod (client-side)

> **Scope:** All UI-facing work — React Server Components, Client Components, design system, form handling, optimistic updates, D3 charts, and AI-assisted composition.
> For Server Actions, API routes, services, and infrastructure see the Backend, Testing, and DevOps files.

---

## Table of Contents

1. [Epic 1 — Project Setup (Frontend Concerns)](#epic-1--project-setup-frontend-concerns)
2. [Epic 2 — Design System](#epic-2--design-system)
3. [Epic 3 — Authentication UI](#epic-3--authentication-ui)
4. [Epic 4 — Core Tweet UI](#epic-4--core-tweet-ui)
5. [Epic 5 — User Profiles UI](#epic-5--user-profiles-ui)
6. [Epic 6 — Feed UI & Infinite Scroll](#epic-6--feed-ui--infinite-scroll)
7. [Epic 7 — Search & Explore UI](#epic-7--search--explore-ui)
8. [Epic 8 — Notifications UI](#epic-8--notifications-ui)
9. [Epic 9 — AI Integration (Client Side)](#epic-9--ai-integration-client-side)
10. [Epic 10 — Analytics Dashboard (D3.js)](#epic-10--analytics-dashboard-d3js)
11. [Epic 11 — Client-Side Caching (TanStack Query)](#epic-11--client-side-caching-tanstack-query)

---

## Epic 1 — Project Setup (Frontend Concerns)

**Goal:** Bootstrap the Next.js 16 App Router project with TypeScript, linting, and Tailwind CSS 4 so all frontend engineers share a consistent, type-safe foundation.

---

### Story 1.1 — Initialize Next.js 16 App Router project

**As a** developer,
**I want** the Next.js 16 project initialized with TypeScript strict mode and the App Router,
**so that** the team has a consistent, type-safe starting point.

**Acceptance Criteria:**

- `next@16` installed with `app/` directory structure
- `tsconfig.json` set to `strict: true`, path aliases configured (`@/` → `src/`)
- ESLint 9 flat config (`eslint.config.mjs`) with Next.js + TypeScript rules
- Prettier configured with shared `.prettierrc`
- Stylelint configured for CSS/Tailwind rules
- `package.json` scripts: `dev`, `build`, `lint`, `format`, `typecheck`

**Tasks:**

- [ ] `npx create-next-app@latest` with App Router and TypeScript flags
- [ ] Set `strict: true` and add path aliases to `tsconfig.json`
- [ ] Install and configure ESLint 9 flat config with `@eslint/js`, `typescript-eslint`, `eslint-config-next`
- [ ] Install Prettier and create `.prettierrc` + `.prettierignore`
- [ ] Install Stylelint and create `stylelint.config.mjs` for Tailwind CSS rules
- [ ] Add `lint-staged` + `husky` pre-commit hooks
- [ ] Verify `next build` succeeds on a clean clone

---

### Story 1.2 — Configure Tailwind CSS 4

**As a** developer,
**I want** Tailwind CSS 4 installed with a CSS-first `@theme` config,
**so that** the team can use design tokens without a JavaScript config file.

**Acceptance Criteria:**

- Tailwind CSS 4 imported via `@import 'tailwindcss'` in `globals.css`
- Design token layer defined with `@theme { ... }` block
- Dark mode enabled via `.dark` class variant
- `@layer components` utility classes scaffolded

**Tasks:**

- [ ] Install `tailwindcss@4` and remove legacy `tailwind.config.js`
- [ ] Create `app/globals.css` with `@import 'tailwindcss'` and `@theme {}` block
- [ ] Define brand colour palette (`--color-brand-50` → `--color-brand-950`)
- [ ] Define typography, spacing, radius, and shadow design tokens
- [ ] Configure dark mode tokens under `@theme dark {}`
- [ ] Add `@layer components {}` for shared utility class patterns (`.tweet-card`, `.btn-primary`)
- [ ] Validate purge is working in production build — no unused styles shipped

---

### Story 1.3 — Set up shared folder structure (frontend directories)

**As a** developer,
**I want** a well-defined frontend folder structure enforced from the start,
**so that** components, hooks, and contexts are discoverable and responsibilities are clear.

**Acceptance Criteria:**

- `src/components/` organized by: `ui/` (primitives), `composed/` (higher-level), `layout/` (shell)
- `src/hooks/` for custom React hooks (`useSSE`, `useDebounce`, `useInfiniteScroll`)
- `src/contexts/` for React contexts (`ThemeContext`, `AuthContext`, `UIContext`)
- `src/types/` for shared TypeScript interfaces
- Path alias `@/` maps to `src/`

**Tasks:**

- [ ] Create `src/components/ui/`, `src/components/composed/`, `src/components/layout/` with `index.ts` barrel exports
- [ ] Create `src/hooks/` with placeholder `index.ts`
- [ ] Create `src/contexts/` with placeholder `index.ts`
- [ ] Create `src/types/index.ts` with shared interfaces (`TweetWithAuthor`, `UserPublic`, `NotificationPayload`)
- [ ] Document folder conventions in `CONTRIBUTING.md`

---

## Epic 2 — Design System

**Goal:** Build the Radix UI + Tailwind CSS 4 component library that all feature UIs consume.

---

### Story 2.1 — Install and configure Radix UI primitives

**As a** developer,
**I want** all required Radix UI packages installed with a consistent styled wrapper pattern,
**so that** accessible, unstyled primitives are ready for composition.

**Acceptance Criteria:**

- All primitives installed: `Dialog`, `Sheet`, `Popover`, `DropdownMenu`, `Avatar`, `Tooltip`, `HoverCard`, `Switch`, `Checkbox`, `RadioGroup`, `ScrollArea`, `Tabs`, `Separator`
- Each primitive has a thin wrapper in `src/components/ui/` with Tailwind classes applied
- All exported from `src/components/ui/index.ts`

**Tasks:**

- [ ] Install all `@radix-ui/react-*` packages
- [ ] Create wrapper components: `Dialog`, `Sheet`, `Popover`, `DropdownMenu`, `Avatar`, `Tooltip`, `HoverCard`, `Switch`, `Checkbox`, `RadioGroup`, `ScrollArea`, `Tabs`, `Separator`
- [ ] Apply `data-[state=open]:`, `data-[disabled]:`, `data-[highlighted]:` Tailwind variant styles on each wrapper
- [ ] Add `animate-in` / `animate-out` keyframe animations via Tailwind for overlays
- [ ] Export all wrappers from `src/components/ui/index.ts`
- [ ] Write Vitest + React Testing Library snapshot tests for each primitive wrapper

---

### Story 2.2 — Build composed components

**As a** designer/developer,
**I want** higher-level components built from Radix primitives and design tokens,
**so that** feature teams can assemble UIs without reimplementing atoms.

**Acceptance Criteria:**

- `TweetCard`: avatar, handle, timestamp, content, action bar slot
- `UserCard`: avatar, display name, handle, bio, follow-button slot
- `TrendingItem`: hashtag text and count badge
- `NotificationItem`: type icon, body text, relative timestamp
- `SearchInput`: icon, controlled input, clear button, keyboard accessibility

**Tasks:**

- [ ] Build `TweetCard` RSC with `TweetWithAuthor` props interface and action bar `children` slot
- [ ] Build `UserCard` RSC with follow-button `children` slot
- [ ] Build `TrendingItem` functional component
- [ ] Build `NotificationItem` with icon map: `LIKE` → heart, `FOLLOW` → user-plus, `REPLY` → message, `RETWEET` → repeat
- [ ] Build `SearchInput` Client Component with `useRef` + clear handler
- [ ] Write RTL tests: render each component with mock data, assert key content and ARIA attributes

---

### Story 2.3 — Implement dark mode theming

**As a** user,
**I want** to toggle between light and dark mode,
**so that** I can use the app comfortably in any lighting condition.

**Acceptance Criteria:**

- Theme persists across page reloads via `localStorage`
- `ThemeContext` exposes `theme: 'light' | 'dark'` and `toggleTheme()`
- `.dark` class applied to `<html>` element on toggle
- All design tokens respond correctly in both modes (no hardcoded colours)

**Tasks:**

- [ ] Create `src/contexts/ThemeContext.tsx` with `useLocalStorage` hook reading `'theme'` key
- [ ] Apply `.dark` class to `document.documentElement` reactively on state change
- [ ] Wire `ThemeContext` into root layout via `<ThemeProvider>` wrapper component
- [ ] Build `ThemeToggle` button using `Switch` Radix primitive with sun/moon icons
- [ ] Define all dark-mode token overrides under `@theme dark {}` in `globals.css`
- [ ] Write RTL test: click toggle → assert `.dark` on `<html>` and `localStorage` updated

---

## Epic 3 — Authentication UI

**Goal:** Provide a clean sign-in page, session indicator, and auth-aware navigation shell.

---

### Story 3.1 — Login page and OAuth trigger

**As a** user,
**I want** a clean login page with a "Sign in with Microsoft" button,
**so that** I can authenticate without creating a separate password.

**Acceptance Criteria:**

- `/login` page renders with brand logo, tagline, and single CTA button
- Clicking the button calls `signIn('microsoft-entra-id')` from NextAuth
- OAuth errors (e.g., `access_denied`) render a friendly inline message
- Page redirects to `/home` after successful sign-in

**Tasks:**

- [ ] Create `app/(auth)/login/page.tsx` as an RSC
- [ ] Build `SignInButton` Client Component that calls `signIn()` with loading state
- [ ] Add error message rendering from `searchParams.error`
- [ ] Style page with centered card layout, brand colours, and responsive design
- [ ] Write RTL test: click button → assert `signIn` called with correct provider

---

### Story 3.2 — Session-aware navigation shell

**As a** user,
**I want** to see my avatar and handle in the nav bar,
**so that** I always know which account I'm using.

**Acceptance Criteria:**

- `AuthContext` exposes current session `user` (`id`, `handle`, `displayName`, `avatarUrl`) to all client components
- `UserBadge` in sidebar nav shows avatar (Radix `Avatar`) + display name + handle
- Dropdown menu (Radix `DropdownMenu`) offers "Sign out" option
- Clicking "Sign out" calls `signOut()` and redirects to `/login`

**Tasks:**

- [ ] Create `src/contexts/AuthContext.tsx` wrapping NextAuth `useSession`
- [ ] Build `UserBadge` Client Component using `Avatar` + `DropdownMenu` primitives
- [ ] Wire `signOut({ callbackUrl: '/login' })` to the dropdown item
- [ ] Add `AuthContext` provider to `(main)` layout
- [ ] Write RTL test: render `UserBadge` with mock session → assert handle and sign-out trigger

---

## Epic 4 — Core Tweet UI

**Goal:** Deliver the tweet composer, tweet card rendering, optimistic interactions, and reply threads.

---

### Story 4.1 — Tweet Composer component

**As a** user,
**I want** to compose and post a tweet from the home feed,
**so that** I can share content with my followers.

**Acceptance Criteria:**

- `TweetComposer` is a Client Component using `react-hook-form` + `zodResolver(createTweetSchema)`
- Character counter visible (max 280); counter turns red and submit button disables beyond limit
- Inline field errors rendered below the textarea in real time
- On valid submit, calls `createTweetAction` Server Action
- Optimistic insert adds the new tweet to the top of the feed cache immediately
- Loading spinner shown on submit button during pending Server Action

**Tasks:**

- [ ] Install `react-hook-form` and `@hookform/resolvers`
- [ ] Build `TweetComposer` with `<textarea>` controlled by `react-hook-form`
- [ ] Add character counter: `{length}/280` with red styling beyond limit
- [ ] Integrate `zodResolver(createTweetSchema)` for real-time field validation
- [ ] Call `createTweetAction(formData)` on valid submit
- [ ] Implement TanStack Query `onMutate` optimistic insert + `onError` rollback
- [ ] Show `<Spinner>` on submit button while `isPending`
- [ ] Write RTL test: content > 280 chars → button disabled + counter red; valid content → action called

---

### Story 4.2 — Tweet card display

**As a** user,
**I want** to see tweets rendered with author info, content, and engagement counts,
**so that** I can understand what was posted and how it performed.

**Acceptance Criteria:**

- `Tweet` RSC renders: avatar, display name, `@handle`, relative timestamp (`2m ago`), tweet content
- `TweetActions` Client Component shows Like, Retweet, Reply, Share buttons with counts
- Like and Retweet counts update optimistically on click
- Own tweets show a kebab `DropdownMenu` with "Delete" option

**Tasks:**

- [ ] Create `Tweet` RSC accepting `TweetWithAuthor` prop type
- [ ] Use `Intl.RelativeTimeFormat` for human-readable timestamps
- [ ] Create `TweetActions` CC with `useMutation` for like and retweet actions
- [ ] Implement optimistic like count update in TanStack Query cache (`setQueryData`)
- [ ] Add `DropdownMenu` with "Delete" item — visible only when `authorId === session.user.id`
- [ ] Write RTL test: like button click → optimistic count +1; error → count rolled back

---

### Story 4.3 — Reply thread view

**As a** user,
**I want** to view a tweet's full reply thread,
**so that** I can follow and participate in conversations.

**Acceptance Criteria:**

- `/tweet/[id]` page RSC renders the original tweet followed by all replies in chronological order
- `TweetComposer` rendered above the thread with `parentId` pre-set
- Replies visually indented (max 3 levels via margin classes)
- 404 page shown for non-existent tweet IDs

**Tasks:**

- [ ] Create `app/(main)/tweet/[id]/page.tsx` RSC
- [ ] Render original tweet with `<Tweet>` component
- [ ] Map replies array recursively, applying `ml-8` indentation per nesting level (capped at 3)
- [ ] Pass `defaultValues={{ parentId: id }}` to `TweetComposer`
- [ ] Show Next.js `notFound()` when tweet ID is not found
- [ ] Write RTL test: renders original tweet and nested reply with correct indentation class

---

## Epic 5 — User Profiles UI

**Goal:** Profile pages, follow/unfollow interaction, and edit profile modal.

---

### Story 5.1 — User profile page

**As a** user,
**I want** to visit a profile at `/[handle]` and see their tweets, bio, and social counts,
**so that** I can learn about other users and decide whether to follow them.

**Acceptance Criteria:**

- `ProfileHeader` RSC shows avatar, display name, handle, bio, follower count, following count
- Tab bar (Radix `Tabs`): "Tweets" | "Replies" | "Media" | "Likes"
- `FollowButton` CC visible to authenticated visitors; hidden on own profile
- Tweet list rendered inside a `<Suspense>` boundary with skeleton fallback
- 404 shown for unknown handles

**Tasks:**

- [ ] Create `app/(main)/[handle]/page.tsx` RSC
- [ ] Build `ProfileHeader` RSC accepting `UserPublic` + counts props
- [ ] Build `FollowButton` CC with optimistic toggle using `useMutation`
- [ ] Implement tab navigation with `Tabs` primitive — each tab renders a different tweet list RSC
- [ ] Add `<Suspense fallback={<TweetListSkeleton />}>` around each tab's tweet list
- [ ] Show skeleton loaders during data fetches
- [ ] Write RTL test: `FollowButton` hidden when `handle === session.user.handle`

---

### Story 5.2 — Edit profile modal

**As a** user,
**I want** to edit my display name and bio via a modal,
**so that** I can keep my profile up to date.

**Acceptance Criteria:**

- "Edit Profile" button opens a Radix `Sheet` (or `Dialog`) with a form
- `updateProfileSchema` validated via `zodResolver`: `displayName` (max 50), `bio` (max 160), optional `avatarUrl`
- Inline errors shown per field in real time
- On successful submit, profile page updates immediately (cache invalidated)
- Sheet closes after successful save

**Tasks:**

- [ ] Build `EditProfileModal` CC using `Sheet` primitive
- [ ] Wire `react-hook-form` + `zodResolver(updateProfileSchema)` to form fields
- [ ] Show character counters for `displayName` and `bio` fields
- [ ] Call `updateProfileAction` on submit; close sheet on `{ success: true }`
- [ ] Invalidate `['profile', handle]` TanStack Query key on success
- [ ] Write RTL test: bio > 160 chars → inline error shown; valid submit → action called

---

## Epic 6 — Feed UI & Infinite Scroll

**Goal:** Render the personalized home feed with client-side infinite scroll powered by TanStack Query.

---

### Story 6.1 — Home feed layout

**As a** user,
**I want** to see my personalized feed on `/home`,
**so that** I can read tweets from people I follow as soon as I land on the page.

**Acceptance Criteria:**

- `/home` RSC streams the first 20 tweets inside a `<Suspense>` boundary
- `TweetComposer` is pinned above the feed list
- `TweetList` RSC renders each tweet as a `<Tweet>` + `<TweetActions>` pair
- Skeleton placeholders shown while tweets are loading

**Tasks:**

- [ ] Create `app/(main)/home/page.tsx` RSC with `<Suspense fallback={<FeedSkeleton />}>`
- [ ] Build `TweetList` RSC that accepts `Tweet[]` initial data
- [ ] Build `FeedSkeleton` component with 5 animated placeholder cards
- [ ] Position `TweetComposer` above `TweetList` with a sticky top style
- [ ] Write RTL test: renders first-page tweets from props; `TweetComposer` present in document

---

### Story 6.2 — Infinite scroll pagination

**As a** user,
**I want** new tweets to load automatically as I scroll down,
**so that** I never have to click a "Load more" button.

**Acceptance Criteria:**

- `InfiniteScroll` Client Component wraps `TweetList` and uses `useInfiniteQuery`
- `IntersectionObserver` on a sentinel `<div>` at the bottom triggers `fetchNextPage()`
- Cursor-based pagination: each page response includes `nextCursor` (`createdAt` of last tweet)
- Loading spinner shown while fetching the next page
- "You're all caught up" message displayed when `hasNextPage === false`

**Tasks:**

- [ ] Build `InfiniteScroll` CC using `useInfiniteQuery` querying `/api/feed?cursor=&limit=20`
- [ ] Attach `IntersectionObserver` to sentinel `<div>` with `ref` callback
- [ ] Call `fetchNextPage()` when sentinel enters viewport and `hasNextPage` is true
- [ ] Show `<Spinner>` when `isFetchingNextPage`
- [ ] Render "You're all caught up" end-of-feed message
- [ ] Write RTL test: mock `useInfiniteQuery` returning 2 pages → assert both page tweets rendered

---

## Epic 7 — Search & Explore UI

**Goal:** Build the Explore page with a debounced search bar for tweets and users, plus a trending sidebar.

---

### Story 7.1 — Search bar with debounced query

**As a** user,
**I want** to type in a search bar and see results appear automatically,
**so that** I can find content without pressing Enter.

**Acceptance Criteria:**

- `SearchBar` Client Component debounces input by 300ms before triggering query
- Minimum 2 characters before any network request fires
- Results section switches between "Tweets" and "Users" via tab buttons
- Empty state shows a friendly message ("No tweets found for …")
- Loading spinner shown during fetch

**Tasks:**

- [ ] Build `SearchBar` CC with `useDebounce(value, 300)` custom hook
- [ ] Use `useQuery({ queryKey: ['search', debouncedTerm, type], enabled: debouncedTerm.length >= 2 })`
- [ ] Build `useDebounce` hook in `src/hooks/useDebounce.ts`
- [ ] Render tweet results as `<TweetCard>` list; user results as `<UserCard>` list with `FollowButton`
- [ ] Show skeleton loaders while `isLoading`, empty state when `data.length === 0`
- [ ] Write RTL test: type 1 char → no fetch; type 2+ chars → query enabled after debounce

---

### Story 7.2 — Explore page layout with trending sidebar

**As a** user,
**I want** to see trending hashtags alongside search results,
**so that** I can discover popular conversations.

**Acceptance Criteria:**

- `/explore` page uses a 2-column layout: search results (left) + trending sidebar (right)
- `TrendingList` RSC renders top 10 hashtags as `<TrendingItem>` components
- Clicking a hashtag pre-fills the `SearchBar` with `#hashtag`
- Trending list revalidates every 30 seconds server-side

**Tasks:**

- [ ] Create `app/(main)/explore/page.tsx` RSC with 2-column CSS Grid layout
- [ ] Build `TrendingList` RSC fetching from `SearchService.getTrending()`
- [ ] Add `export const revalidate = 30` to the explore page segment
- [ ] Wire hashtag click to update `searchParams` via `useRouter().push`
- [ ] Pre-fill `SearchBar` from `searchParams.q` on mount
- [ ] Write RTL test: clicking `TrendingItem` updates `SearchBar` value

---

## Epic 8 — Notifications UI

**Goal:** Real-time notification bell with badge, dropdown list, and SSE-driven live updates.

---

### Story 8.1 — `useSSE` custom hook

**As a** developer,
**I want** a `useSSE(url)` hook wrapping the browser's `EventSource` API,
**so that** any component can subscribe to the server-sent event stream cleanly.

**Acceptance Criteria:**

- Hook opens `EventSource` on mount and closes it on unmount
- Accepts `onMessage` callback called with parsed JSON payload
- Reconnects automatically on connection error (browser `EventSource` built-in)
- Returns `{ status: 'connecting' | 'open' | 'error' }`

**Tasks:**

- [ ] Create `src/hooks/useSSE.ts` with `useEffect` wrapping `EventSource`
- [ ] Call `onMessage(JSON.parse(event.data))` on each message event
- [ ] Set `readyState`-derived status in state
- [ ] Clean up `eventSource.close()` in `useEffect` return
- [ ] Write Vitest test mocking `EventSource` class: assert open → message → close lifecycle

---

### Story 8.2 — Notification bell and dropdown

**As a** user,
**I want** a notification bell icon in the nav with a live unread badge,
**so that** I can see and act on interactions without refreshing the page.

**Acceptance Criteria:**

- `NotificationBell` CC connects to `/api/stream` via `useSSE`
- Incoming SSE events trigger `queryClient.invalidateQueries(['notifications'])` and badge increment
- Badge shows unread count (red); disappears when all read
- Clicking the bell opens a `Popover` with a `ScrollArea` of the last 20 `<NotificationItem>` components
- "Mark all as read" button calls `markAllReadAction` and resets badge

**Tasks:**

- [ ] Build `NotificationBell` CC using `useSSE('/api/stream', { onMessage })`
- [ ] On message receipt: increment local unread count state + invalidate notifications query
- [ ] Use `Popover` + `ScrollArea` primitives for the dropdown panel
- [ ] Fetch notifications via `useQuery(['notifications'])` pointing at `/api/notifications`
- [ ] Build `MarkAllReadButton` CC calling `markAllReadAction` on click, resetting count to 0
- [ ] Write RTL test: mock SSE message → assert badge increments; click mark-read → badge clears

---

## Epic 9 — AI Integration (Client Side)

**Goal:** Integrate CopilotKit v2 on the client for inline tweet suggestions and an AI sidebar chat.

---

### Story 9.1 — CopilotKit provider setup

**As a** developer,
**I want** `<CopilotKit>` wrapping the authenticated layout,
**so that** all AI features share a single runtime connection.

**Acceptance Criteria:**

- `<CopilotKit runtimeUrl="/api/copilot">` wraps `(main)` layout
- Provider renders without errors in both light and dark mode
- `useCopilotReadable` can be called from any descendant component

**Tasks:**

- [ ] Install `@copilotkit/react-core` and `@copilotkit/react-ui`
- [ ] Add `<CopilotKit runtimeUrl="/api/copilot">` to `app/(main)/layout.tsx`
- [ ] Verify provider renders without console errors in dev mode
- [ ] Write RTL test: render layout with provider → children accessible

---

### Story 9.2 — AI-powered `CopilotTextarea` in composer

**As a** user,
**I want** inline AI ghost-text suggestions as I type a tweet,
**so that** I can compose more engaging content faster.

**Acceptance Criteria:**

- `<CopilotTextarea>` replaces plain `<textarea>` in `TweetComposer`
- Suggestions debounced 300ms; ghost text rendered in muted colour
- Pressing Tab accepts the suggestion and populates the full textarea value
- `useCopilotReadable` exposes: last 5 tweets by the current user, top 3 trending hashtags
- Character counter still functions correctly with AI-filled content

**Tasks:**

- [ ] Replace `<textarea>` with `<CopilotTextarea>` from `@copilotkit/react-ui`
- [ ] Implement `useCopilotReadable({ description: 'recentTweets', value: [...] })` in `TweetComposer`
- [ ] Implement `useCopilotReadable({ description: 'trendingHashtags', value: [...] })` in `TweetComposer`
- [ ] Configure `CopilotTextarea` suggestion prompt: `"Complete this tweet in the user's style"`
- [ ] Verify character counter `watch('content').length` updates with AI-populated value
- [ ] Write RTL test with mocked CopilotKit: assert ghost text element rendered

---

### Story 9.3 — AI sidebar chat

**As a** user,
**I want** a toggleable AI sidebar I can open to brainstorm tweet ideas conversationally,
**so that** I can refine my thoughts before posting.

**Acceptance Criteria:**

- `<CopilotSidebar>` toggled from a nav button; open/closed state in `UIContext`
- Sidebar context-aware: current page URL injected as readable context
- Custom `useCopilotAction("draftTweet")` populates `TweetComposer` via shared `UIContext` draft state
- Sidebar labels customized: title "Tweet Assistant", placeholder "How can I help you tweet today?"

**Tasks:**

- [ ] Add `<CopilotSidebar>` to `(main)` layout with customized label props
- [ ] Add `isSidebarOpen` and `toggleSidebar` to `UIContext`
- [ ] Add sidebar toggle button in `NavBar` CC using `UIContext.toggleSidebar`
- [ ] Implement `useCopilotAction("draftTweet", { handler: ({ content }) => setDraft(content) })`
- [ ] Expose `draft` and `setDraft` from `UIContext` so `TweetComposer` can read the draft value
- [ ] Write RTL test: fire `draftTweet` action → assert `TweetComposer` textarea value updated

---

## Epic 10 — Analytics Dashboard (D3.js)

**Goal:** Interactive, animated analytics charts built with D3.js inside React Client Components, fed with data from the analytics RSC.

---

### Story 10.1 — Analytics page layout

**As a** user,
**I want** to visit `/analytics` and see my performance metrics at a glance,
**so that** I can understand how my content is performing.

**Acceptance Criteria:**

- `/analytics` RSC fetches aggregated data and passes it as serializable props to CC boundary
- Page uses a 2×2 CSS Grid layout for four chart panels
- Each chart panel has a title, subtitle, and loading skeleton
- Print-friendly `@media print` styles applied

**Tasks:**

- [ ] Create `app/(main)/analytics/page.tsx` RSC with data fetch and CC boundary
- [ ] Build `AnalyticsLayout` CC with CSS Grid: `grid-cols-2 gap-6`
- [ ] Build `ChartPanel` wrapper component with title, subtitle, and `children` slot
- [ ] Build `ChartSkeleton` animated placeholder (pulsing grey rectangle)
- [ ] Add `@media print { ... }` styles to `globals.css` hiding nav, preserving charts
- [ ] Write RTL test: renders 4 `ChartPanel` components with correct titles

---

### Story 10.2 — Engagement line chart

**As a** user,
**I want** to see a multi-line chart of my likes, retweets, and replies over the past 30 days,
**so that** I can identify which content drove the most engagement.

**Acceptance Criteria:**

- `EngagementChart` CC renders a D3 SVG with three lines: likes (blue), retweets (green), replies (orange)
- X-axis: dates using `d3.scaleTime`; Y-axis: counts using `d3.scaleLinear`
- Legend checkboxes toggle individual lines on/off
- Animated axes and paths on mount via `d3.transition().duration(750)`
- Responsive to container width via `ResizeObserver`

**Tasks:**

- [ ] Create `EngagementChart` CC with `useRef(svgRef)` and `useEffect` D3 render
- [ ] Implement `d3.scaleTime` for X and `d3.scaleLinear` for Y
- [ ] Draw three `d3.line` paths with distinct colours using `data` join
- [ ] Add `d3.axisBottom` and `d3.axisLeft` with formatted tick labels
- [ ] Add enter animation: stroke-dashoffset trick with `d3.transition`
- [ ] Build legend with three toggle checkboxes that filter line visibility
- [ ] Add `ResizeObserver` to re-render SVG on container resize
- [ ] Write Vitest test for `prepareEngagementData(rawRows)` data transform helper

---

### Story 10.3 — Follower growth area chart

**As a** user,
**I want** to see an area chart of my cumulative follower count over time,
**so that** I can track my audience growth momentum.

**Acceptance Criteria:**

- `FollowerGrowth` CC renders a D3 area chart with filled region (brand colour 20% opacity)
- Line stroke in brand colour on top of fill
- Hover tooltip shows exact date and follower count via `d3.bisector`
- Tooltip positioned relative to SVG, not fixed to cursor

**Tasks:**

- [ ] Create `FollowerGrowth` CC using `d3.area` for fill and `d3.line` for stroke
- [ ] Compute cumulative sum in `prepareGrowthData(rows)` helper
- [ ] Implement `mousemove` handler: `d3.bisector(d => d.date).left` to find nearest data point
- [ ] Build `ChartTooltip` component positioned via inline `transform: translate(x, y)`
- [ ] Write Vitest test for `prepareGrowthData` cumulative sum calculation

---

### Story 10.4 — Hashtag bubble chart and posting heatmap

**As a** user,
**I want** to see a bubble chart of my top hashtags and a calendar heatmap of posting frequency,
**so that** I can understand what topics I post about and when I'm most active.

**Acceptance Criteria:**

- `TrendingViz` CC: `d3.pack` layout where bubble radius = hashtag count; hover shows tag + count
- `EngagementHeatmap` CC: calendar grid (weeks × days) with `d3.scaleSequential` fill
- Both charts animate on mount
- Both charts are responsive via `ResizeObserver`

**Tasks:**

- [ ] Create `TrendingViz` CC with `d3.pack` hierarchy and circle rendering
- [ ] Add `mouseover`/`mouseout` tooltip for each bubble
- [ ] Create `EngagementHeatmap` CC: render 52-week × 7-day grid as `rect` elements
- [ ] Apply `d3.scaleSequential(d3.interpolateBlues)` colour scale by post count
- [ ] Animate both charts with `d3.transition().duration(600)` on mount
- [ ] Write Vitest tests for `packData` and `heatmapData` transform helpers

---

## Epic 11 — Client-Side Caching (TanStack Query)

**Goal:** Configure TanStack Query for optimal client-side data freshness and cache lifetime per data type.

---

### Story 11.1 — TanStack Query client configuration

**As a** developer,
**I want** a centrally configured `QueryClient` with appropriate defaults,
**so that** the client avoids unnecessary refetches while keeping data fresh.

**Acceptance Criteria:**

- Global defaults: `staleTime: 30_000ms`, `gcTime: 300_000ms`
- Feed queries override: `staleTime: 30_000ms`
- Notification queries override: `staleTime: 0` (real-time from SSE invalidation)
- Profile queries override: `staleTime: 60_000ms`
- `<QueryClientProvider>` wraps `(main)` layout
- React Query Devtools visible in `development` mode only

**Tasks:**

- [ ] Install `@tanstack/react-query` and `@tanstack/react-query-devtools`
- [ ] Create `src/lib/queryClient.ts` exporting a typed `QueryClient` with global defaults
- [ ] Wrap `app/(main)/layout.tsx` with `<QueryClientProvider client={queryClient}>`
- [ ] Add `<ReactQueryDevtools initialIsOpen={false}>` inside `process.env.NODE_ENV === 'development'` guard
- [ ] Document per-key `staleTime` overrides pattern in `CONTRIBUTING.md`
- [ ] Write unit test: `queryClient` instance has correct `defaultOptions.queries.staleTime`

---

### Story 11.2 — Optimistic mutation pattern

**As a** developer,
**I want** a consistent optimistic update pattern used across all mutations (like, retweet, follow),
**so that** the UI feels instant and rolls back gracefully on errors.

**Acceptance Criteria:**

- `onMutate`: snapshot previous cache, apply optimistic change, return snapshot as context
- `onError`: restore snapshot from context
- `onSettled`: always invalidate the affected query key to sync with server truth
- Pattern documented and used in `likeTweetAction`, `retweetAction`, `followUserAction`

**Tasks:**

- [ ] Create `src/lib/optimisticMutation.ts` with a generic `buildOptimisticMutation<T>` helper
- [ ] Apply pattern in `useMutation` for `likeTweet` (increment count in `['feed']` cache)
- [ ] Apply pattern in `useMutation` for `followUser` (toggle `isFollowing` in `['profile', handle]` cache)
- [ ] Write Vitest test: simulate error → assert cache rolled back to snapshot

---

## Summary

| Epic                        | Stories        | Primary Tech                           |
| --------------------------- | -------------- | -------------------------------------- |
| 1 — Project Setup (FE)      | 3              | Next.js 16, Tailwind CSS 4, ESLint 9   |
| 2 — Design System           | 3              | Radix UI, Tailwind CSS 4               |
| 3 — Auth UI                 | 2              | NextAuth `useSession`, AuthContext     |
| 4 — Core Tweet UI           | 3              | react-hook-form, Zod, TanStack Query   |
| 5 — User Profiles UI        | 2              | Radix Tabs, TanStack Query mutations   |
| 6 — Feed UI                 | 2              | useInfiniteQuery, IntersectionObserver |
| 7 — Search & Explore UI     | 2              | useDebounce, useQuery, searchParams    |
| 8 — Notifications UI        | 2              | EventSource, Popover, ScrollArea       |
| 9 — AI Integration (Client) | 3              | CopilotKit v2, useCopilotReadable      |
| 10 — Analytics (D3.js)      | 4              | D3.js, ResizeObserver, SVG             |
| 11 — Client Caching         | 2              | TanStack Query, optimistic mutations   |
| **Total**                   | **28 stories** |                                        |
