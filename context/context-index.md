# Context Index

## Overview

**Atom** is a headless CMS built for Next.js developers. It is a full-stack Next.js 14 (App Router) application that lets users create, manage, and publish blog posts. It exposes a REST API consumed by an NPM SDK (`atom-nextjs`) that blog owners embed in their own Next.js applications. The core stack is: Next.js 14 App Router · MongoDB/Mongoose (data) · Lucia v3 (session auth) · Upstash Redis (rate limiting) · Argon2 (password hashing) · TailwindCSS · shadcn/ui · Zod · React Hook Form · TanStack Table.

---

## File Tree

```
/source
├── app/                          # Next.js App Router pages & API routes
│   ├── api/                      # REST API (Next.js Route Handlers)
│   │   ├── auth/
│   │   │   ├── delete/route.ts   # DELETE /api/auth/delete
│   │   │   ├── signin/route.ts   # POST   /api/auth/signin
│   │   │   ├── signout/route.ts  # POST   /api/auth/signout
│   │   │   ├── signup/route.ts   # POST   /api/auth/signup  (also exports ApiResponse type)
│   │   │   └── user/
│   │   │       ├── get/route.ts  # GET    /api/auth/user/get
│   │   │       └── update/route.ts # PATCH /api/auth/user/update
│   │   ├── posts/
│   │   │   ├── create/route.ts   # POST   /api/posts/create?project_id=
│   │   │   ├── delete/route.ts   # DELETE /api/posts/delete?project_id=&post_id=
│   │   │   ├── update/route.ts   # PATCH  /api/posts/update?project_id=&post_id=
│   │   │   └── get/single/route.ts # GET  /api/posts/get/single?post_id= (public, Bearer auth)
│   │   └── projects/
│   │       ├── create/route.ts   # POST   /api/projects/create
│   │       ├── delete/route.ts   # DELETE /api/projects/delete?project_id=
│   │       └── get/
│   │           └── single/
│   │               ├── route.ts           # GET /api/projects/get/single (session OR Bearer)
│   │               └── client/route.ts    # GET /api/projects/get/single/client (Bearer only, public SDK)
│   ├── app/                      # Protected /app dashboard (ProtectedRoute layout)
│   │   ├── layout.tsx            # Wraps all /app/* in ProtectedRoute
│   │   ├── page.tsx              # Projects list page
│   │   ├── projects/[id]/page.tsx # Single project editor
│   │   └── settings/
│   │       ├── page.tsx          # User settings
│   │       └── billing/page.tsx  # Billing page (placeholder)
│   ├── blog/                     # Marketing blog (uses atom-nextjs SDK)
│   │   ├── page.tsx              # Blog listing page
│   │   └── [id]/page.tsx         # Single blog post
│   ├── pricing/page.tsx          # Pricing page
│   ├── signin/page.tsx           # Sign-in page
│   ├── signup/page.tsx           # Sign-up page
│   ├── layout.tsx                # Root layout (Montserrat font, Toaster)
│   ├── page.tsx                  # Landing/marketing homepage
│   ├── sitemap.ts                # Dynamic sitemap using generateSitemap()
│   └── robots.ts                 # robots.txt
│
├── components/
│   ├── cards/
│   │   └── PricingPlanCard.tsx   # Plan card for pricing page
│   ├── containers/
│   │   ├── AppContainer.tsx      # App shell with sidebar nav
│   │   ├── MainContainer.tsx     # Marketing shell with Navbar + footer
│   │   └── ProtectedRoute.tsx    # Server component guard (redirect to /signin)
│   ├── forms/
│   │   ├── LoginForm.tsx         # Client form (Zod + RHF)
│   │   └── SignupForm.tsx        # Client form (Zod + RHF)
│   ├── misc/
│   │   └── NpmPackageComponent.tsx # Copy-to-clipboard npm install snippet
│   ├── modals/
│   │   ├── CreatePostModal.tsx   # Dialog form to create a new post
│   │   └── DeleteUserModal.tsx   # Confirm-password dialog to delete account
│   ├── nav/
│   │   └── Navbar.tsx            # Marketing navbar (server component, auth-aware)
│   ├── pages/
│   │   ├── projects/
│   │   │   ├── ProjectPage.tsx       # Projects list (client, DataTable + create dialog)
│   │   │   ├── ProjectComponent.tsx  # Project editor shell (post list + form)
│   │   │   └── ProjectFormComponent.tsx # RHF form to edit/delete a post
│   │   └── settings/
│   │       └── SettingsForm.tsx  # First/last name update form
│   ├── sidebars/
│   │   ├── AppSidebarNav.tsx     # Left nav for /app (links, user popover, sign-out)
│   │   └── ProjectComponentSidebar.tsx # Post list sidebar inside a project
│   ├── tables/
│   │   └── UserDocumentProjects/
│   │       ├── columns.tsx       # TanStack Table column defs for projects list
│   │       └── table.tsx         # DataTable component
│   └── ui/                       # shadcn/ui components (button, input, dialog, etc.)
│       └── markdown-editor.tsx   # Wrapped @uiw/react-md-editor
│
├── lib/
│   ├── client/                   # Client-side API call helpers (axios)
│   │   ├── auth/                 # loginUser, signupUser, signoutUser, updateUser, deleteUser
│   │   ├── posts/                # createPost, updatePost, deletePost
│   │   └── projects/             # createProject, deleteProject
│   ├── server/
│   │   ├── encoding/
│   │   │   ├── encodePassword.ts # argon2.hash(password + HASH_SALT)
│   │   │   └── isPasswordValid.ts # argon2.verify(hash, password + HASH_SALT)
│   │   ├── functions/
│   │   │   ├── projects/getProject.ts  # SSR helper: fetches project via internal API
│   │   │   └── user/fetchUser.ts       # SSR helper: fetches user via internal API
│   │   ├── lucia/
│   │   │   ├── init.ts           # Lucia instance (MongodbAdapter, sessionCookie config)
│   │   │   └── functions/validate-request.ts  # Reads session cookie, validates session
│   │   ├── mongo/
│   │   │   ├── init.ts           # connectToDatabase(), Mongoose model refs
│   │   │   └── types/            # Mongoose schemas (userCredentials, userDocuments, userProjects/sessions)
│   │   ├── redis/
│   │   │   └── init.ts           # Upstash Redis + Ratelimit (30 req/min sliding window)
│   │   └── utils/
│   │       ├── generateProjectKey.ts    # 32-byte random base64 → "atom-<base64>"
│   │       ├── validateProjectKey.ts    # Unused/legacy – checks project_key in UserDocuments
│   │       └── validateRequestFetchUser.ts  # validateRequest + fetch UserDocument in one call
│   ├── contants.tsx              # plans, navOptions, planDetails, baseAPIRoute, mongoDBURI, etc.
│   ├── types.ts                  # All shared TypeScript types
│   ├── utils.ts                  # cn() utility (clsx + tailwind-merge)
│   └── utils/
│       └── validateEmail.ts      # Regex email validator
│
├── packages/
│   └── atom-nextjs/              # Published NPM SDK (tsdx build)
│       ├── src/
│       │   ├── index.tsx         # Public exports
│       │   ├── components/
│       │   │   ├── Atom.tsx              # Server component: renders single post
│       │   │   ├── AtomBody.tsx          # Server component: MDX-renders post body
│       │   │   ├── AtomPage.tsx          # Server component: renders post list
│       │   │   ├── AtomPostCard.tsx      # Client post card link
│       │   │   ├── AtomLoadingSkeleton.tsx  # Skeleton for AtomPage
│       │   │   └── AtomArticleSkeleton.tsx  # Skeleton for Atom article
│       │   └── lib/
│       │       ├── client/
│       │       │   ├── getPost.ts            # fetch /api/posts/get/single (Bearer)
│       │       │   ├── getProject.ts         # fetch /api/projects/get/single/client (Bearer)
│       │       │   ├── generatePostMetadata.ts # Builds Next.js Metadata from post
│       │       │   └── generateSitemap.ts    # Builds sitemap entries from project posts
│       │       ├── constants.ts          # baseAPIRoute (points to production or localhost)
│       │       └── types.ts              # Post, ClientPost, ClientProject, ApiResponse
│       └── package.json
│
├── bruno/                        # Bruno API collection for manual testing
├── middleware.ts                 # Rate-limit middleware (30 req/min on /api/*)
├── next.config.mjs               # Cache-Control: no-store on /, /app/*, /api/*
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## Architecture

### Two-layer system

1. **Atom CMS platform** (`/app`, `/components`, `/lib`) — The web app where users register, manage *Projects*, and write *Posts* using a markdown editor. Routes are split between:
   - **Marketing pages** (`/`, `/pricing`, `/blog`, `/signin`, `/signup`) wrapped in `MainContainer` (Navbar + footer)
   - **Dashboard app** (`/app/*`) wrapped in `ProtectedRoute` → `AppContainer` (sidebar nav)

2. **`atom-nextjs` SDK** (`/packages/atom-nextjs`) — Installable NPM package that blog owners add to their own Next.js projects. It calls the Atom API using a *project key* (Bearer token) and renders MDX content using `next-mdx-remote`.

### Request flow

```
User browser
  → Atom Dashboard (Next.js SSR/Client)
    → /lib/client/*  (axios, sends session cookie)
      → Next.js API routes (/app/api/*)
        → validateRequest() — verifies Lucia session cookie
        → MongoDB via Mongoose
```

```
Visitor browser (on customer's blog)
  → Customer's Next.js app
    → atom-nextjs SDK (fetch, Bearer project key)
      → Atom API: /api/projects/get/single/client or /api/posts/get/single
        → MongoDB via Mongoose
```

### Data flow in dashboard SSR

Dashboard pages are Server Components that call **internal** API routes using `axios` with forwarded cookies (see `lib/server/functions/user/fetchUser.ts` and `getProject.ts`). This pattern enables SSR while reusing the same route-handler auth logic.

### Authentication

Session-based via **Lucia v3** + **MongodbAdapter**. On sign-in/sign-up, a session document is created in MongoDB and a cookie (`auth_session`) is set. `validateRequest()` is the single function used by every protected route.

Public SDK routes authenticate via `Authorization: Bearer <project_key>` header — no user session needed.

---

## Key Concepts

- **Project** — A named container for posts, with a generated `project_key` used by the SDK.
  - Defined in: `lib/types.ts:L39-L47`, Mongoose schema: `lib/server/mongo/types/userProjects.ts:L45-L70`
  - Model ref: `lib/server/mongo/init.ts` (`ProjectsRef`)

- **Post** — A blog entry embedded inside a Project document (sub-document array).
  - Defined in: `lib/types.ts:L10-L22`, Mongoose schema: `lib/server/mongo/types/userProjects.ts:L1-L43`
  - Body is stored as Markdown/MDX string; rendered via `next-mdx-remote` in the SDK.

- **UserDocument** — Profile document for each user (separate from credentials).
  - Defined in: `lib/types.ts:L49-L59`, Mongoose schema: `lib/server/mongo/types/userDocuments.ts`
  - Contains `first_name`, `last_name`, `email`, `plan`, and a `projects` summary array.

- **UserCredentials** — Auth document storing `email` + `password_hash` (argon2).
  - Defined in: `lib/types.ts:L3-L10`, Mongoose schema: `lib/server/mongo/types/userCredentials.ts`

- **Plan** — Subscription tier. Currently `"single"` (free), `"startup"`, `"business"` (disabled).
  - Defined in: `lib/types.ts:L25`, plan details in `lib/contants.tsx:L30-L90`
  - Enforced at post creation/update (max `body_length`, `max_docs`) and project creation (`max_projects`).

- **Project Key** — `"atom-" + randomBytes(32).toString("base64")`. Issued per project; used as Bearer token by SDK consumers.
  - Generated in: `lib/server/utils/generateProjectKey.ts`

- **ApiResponse<T>** — Standard JSON response shape: `{ success: boolean; message: string | null; response: T }`.
  - Defined/exported in: `app/api/auth/signup/route.ts:L19-L23` (re-used across all routes)

- **validateRequest()** — Memoized React `cache()` function to validate the session cookie and return `{ user, session }` or `{ null, null }`.
  - Defined in: `lib/server/lucia/functions/validate-request.ts`

- **ProtectedRoute** — Server component that calls `validateRequest()` and redirects to `/signin` if no user.
  - Defined in: `components/containers/ProtectedRoute.tsx`

- **AtomBody** — Server component inside the SDK that compiles MDX via `next-mdx-remote/rsc`.
  - Defined in: `packages/atom-nextjs/src/components/AtomBody.tsx`

---

## API Surface

All routes are under `/app/api/`. Session auth routes check Lucia session cookie. Public (SDK) routes use `Authorization: Bearer <project_key>`.

### Auth

| Method | Path | Auth | Description | File |
|--------|------|------|-------------|------|
| POST | `/api/auth/signup` | Public | Create account + session | `app/api/auth/signup/route.ts` |
| POST | `/api/auth/signin` | Public | Sign in, create session | `app/api/auth/signin/route.ts` |
| POST | `/api/auth/signout` | Session cookie | Invalidate session | `app/api/auth/signout/route.ts` |
| DELETE | `/api/auth/delete` | Session cookie | Delete user + all data | `app/api/auth/delete/route.ts` |
| GET | `/api/auth/user/get` | Session cookie | Fetch UserDocument | `app/api/auth/user/get/route.ts` |
| PATCH | `/api/auth/user/update` | Session cookie | Update first/last name | `app/api/auth/user/update/route.ts` |

**Body types**: `SignupRequestParams` (email, password, first_name, last_name), `SigninRequestParams` (email, password), `DeleteUserRequestBody` (password), `UpdateUserRequestParams` (first_name?, last_name?).

### Projects

| Method | Path | Auth | Description | File |
|--------|------|------|-------------|------|
| POST | `/api/projects/create` | Session | Create project (plan-limited) | `app/api/projects/create/route.ts` |
| DELETE | `/api/projects/delete?project_id=` | Session | Delete project + remove from UserDoc | `app/api/projects/delete/route.ts` |
| GET | `/api/projects/get/single?project_id=` | Session OR Bearer | Fetch full Project | `app/api/projects/get/single/route.ts` |
| GET | `/api/projects/get/single/client` | Bearer only | Fetch ClientProject (SDK use) | `app/api/projects/get/single/client/route.ts` |

### Posts

| Method | Path | Auth | Description | File |
|--------|------|------|-------------|------|
| POST | `/api/posts/create?project_id=` | Session | Add post to project | `app/api/posts/create/route.ts` |
| PATCH | `/api/posts/update?project_id=&post_id=` | Session | Update post fields | `app/api/posts/update/route.ts` |
| DELETE | `/api/posts/delete?project_id=&post_id=` | Session | Remove post from project | `app/api/posts/delete/route.ts` |
| GET | `/api/posts/get/single?post_id=` | Bearer | Fetch single post (SDK use) | `app/api/posts/get/single/route.ts` |

**Note**: The GET `/api/posts/get/single` adds a Markdown watermark (`[Atom](...)`) to `post.body` for users on the free `"single"` plan.

---

## Data Model

All types are in `lib/types.ts`. Mongoose schemas are in `lib/server/mongo/types/`.

### UserCredentials (`credentials` collection)
```
_id: string (UUID)
email: string (unique, lowercase)
password_hash: string (argon2)
createdAt / updatedAt: Date
```
Schema: `lib/server/mongo/types/userCredentials.ts`

### UserDocument (`documents` collection)
```
_id: string (same UUID as credentials)
first_name / last_name: string
email: string
plan: "single" | "startup" | "business"
projects: UserDocumentProjects[]   ← summary (id, title, dates, creator)
createdAt / updatedAt: Date
```
Schema: `lib/server/mongo/types/userDocuments.ts`

### Project (`projects` collection)
```
_id: string (UUID)
title: string
project_key: string ("atom-" + 32 random bytes base64)
creator_uid: string
posts: Post[]    ← embedded sub-documents
createdAt / updatedAt: Date
```
Schema: `lib/server/mongo/types/userProjects.ts` (projectsSchema + postSchema)

### Post (embedded in Project)
```
id: string (UUID, not _id)
title / author / body / teaser: string
image: string | null
keywords: string[]
creator_uid: string
createdAt / updatedAt: Date
```

### Session (`session` collection — Lucia-managed)
```
user_id: string
expires_at: Date
```
Schema: `lib/server/mongo/types/userSessions.ts`

### SDK types (`packages/atom-nextjs/src/lib/types.ts`)
- **ClientPost** — subset of Post without body (for listing): `id, teaser, title, createdAt, updatedAt, author, image?`
- **ClientProject** — `title, posts: ClientPost[], id, updatedAt, createdAt`

---

## Auth & Middleware

### Authentication (Lucia v3)
- **Init**: `lib/server/lucia/init.ts` — Creates `Lucia` instance with `MongodbAdapter` pointing to `sessions` and `credentials` collections.
- **Session cookie**: Named `auth_session`, `expires: false`, `secure` in production.
- **Validation**: `lib/server/lucia/functions/validate-request.ts` — Uses React `cache()` for request deduplication. Reads `auth_session` cookie, calls `lucia.validateSession()`, refreshes fresh sessions, and blanks out invalid sessions.
- **Flow**: Sign-up/sign-in → `lucia.createSession()` → set cookie. Sign-out → `lucia.invalidateSession()` → set blank cookie.

### Route Guard
- `components/containers/ProtectedRoute.tsx` — Server component at `app/app/layout.tsx`. Calls `validateRequest()`, redirects to `/signin` if not authenticated.

### API Middleware (`middleware.ts`)
- Matches `/api/:path*`
- Rate-limits all API requests using **Upstash Redis** sliding window: 30 requests per minute per IP.
- Returns `ApiResponse` with `"Too many requests."` on limit exceeded.

### Password Hashing
- `lib/server/encoding/encodePassword.ts` — `argon2.hash(password + HASH_SALT)`
- `lib/server/encoding/isPasswordValid.ts` — `argon2.verify(hash, password + HASH_SALT)`
- Salt is an environment variable `HASH_SALT`.

---

## Configuration

| Variable | Purpose | Default | Where used |
|----------|---------|---------|------------|
| `HASH_SALT` | Pepper for argon2 password hashing | (required) | `lib/server/encoding/` |
| `MONGO_DB_URI` | MongoDB connection string | (required) | `lib/contants.tsx`, `lib/server/mongo/init.ts` |
| `ATOM_PROJECT_KEY` | Atom project key for the Atom marketing blog | (required) | `app/sitemap.ts`, `app/blog/*` |
| `UPSTASH_REDIS_REST_URL` | Upstash Redis URL for rate limiting | (required) | `lib/server/redis/init.ts` |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis auth token | (required) | `lib/server/redis/init.ts` |
| `NEXT_PUBLIC_ENV` | `"prod"` or `"dev"` — switches `baseAPIRoute` | `"dev"` → `localhost:3000` | `lib/contants.tsx` |
| `NODE_ENV` | Standard Next.js env, used for `secure` session cookie flag | `"development"` | `lib/server/lucia/init.ts` |

**`baseAPIRoute`** in `lib/contants.tsx:L31-L34`:
- `prod` → `https://cmsatom.netlify.app/api`
- `dev` → `http://localhost:3000/api`

**SDK `baseAPIRoute`** in `packages/atom-nextjs/src/lib/constants.ts`:
- Hardcoded to `https://cmsatom.netlify.app/api` (production)

**Plan limits** (`lib/contants.tsx:L30-L90`):
| Plan | Max projects | Max posts | Max body length |
|------|-------------|-----------|-----------------|
| single (free) | 2 | 100 | 10,000 chars |
| startup ($3.99) | 3 | 1,000 | 25,000 chars (disabled) |
| business ($11.99) | 5 | 2,500 | 50,000 chars (disabled) |

---

## Dependencies

| Package | Purpose |
|---------|---------|
| `lucia` + `@lucia-auth/adapter-mongodb` | Session-based authentication |
| `mongoose` | MongoDB ODM, model definitions |
| `@upstash/redis` + `@upstash/ratelimit` | Redis-backed rate limiting |
| `argon2` | Password hashing (argon2id) |
| `uuid` | Generate UUIDs for user IDs, project IDs, post IDs |
| `zod` | Schema validation (forms + API input types) |
| `react-hook-form` + `@hookform/resolvers` | Form management |
| `@tanstack/react-table` | Projects list data table |
| `@uiw/react-md-editor` | In-app Markdown/MDX editor |
| `next-mdx-remote` | Server-side MDX compilation (in SDK) |
| `rehype-sanitize` | Sanitize HTML in MDX output |
| `remark-gfm` | GitHub Flavored Markdown in MDX |
| `axios` | Client-side API calls (dashboard) |
| `react-hot-toast` | Toast notifications |
| `framer-motion` | Animations on marketing pages |
| `@radix-ui/*` | Accessible UI primitives (shadcn base) |
| `react-syntax-highlighter` | Code blocks on landing page |
| `react-tweet` | Embedded tweets (currently commented out) |
| `oslo` | Crypto utilities (dependency of Lucia) |

---

## Build & Run

```bash
# Development
bun dev  # or npm run dev / yarn dev / pnpm dev

# Build
bun run build

# Start production
bun run start

# Lint
bun run lint
```

### Running atom-nextjs SDK locally
```bash
cd /source/packages/atom-nextjs
npm link
cd /source
npm link atom-nextjs
```

SDK is built with `tsdx build`. Entry: `packages/atom-nextjs/src/index.tsx` → `dist/index.js`.

### Bruno API Testing
`/source/bruno/` contains a Bruno collection with pre-built requests for all routes. Environment file at `bruno/environments/Development.bru` sets `base_url: http://localhost:3000`. Secrets (`session_id`, `project_key`, `project_id`, `post_id`) are per-environment.

---

## Patterns & Conventions

1. **ApiResponse<T> pattern**: Every API route returns `{ success: boolean, message: string | null, response: T }`. The type is defined in `app/api/auth/signup/route.ts` and imported by all other routes.

2. **Always connect to DB before queries**: Every route handler calls `await connectToDatabase()` before any Mongoose operations. The `connectToDatabase()` function is idempotent (checks existing mongoose connection state).

3. **validateRequest() for auth**: All protected routes call `const { user } = await validateRequest()` and throw `"Invalid session. Please sign in."` if `!user`.

4. **Plan enforcement at API level**: Create/update post and create project routes look up `planDetails` from constants and enforce `max_body_length` and `max_projects`.

5. **Mongoose transactions**: Multi-document writes (signup, delete user, create project, delete project) use `mongoose.startSession()` + `withTransaction()`.

6. **Client helpers in `/lib/client/`**: All client-side API calls are thin axios wrappers that throw on `!res.success`. They are "use client" context — do not import server modules.

7. **Server SSR helpers in `/lib/server/functions/`**: `fetchUser()` and `getProject()` make internal HTTP calls with forwarded cookies for SSR data loading.

8. **Form pattern**: All forms use `react-hook-form` + `zodResolver` + shadcn Form components. Schemas are defined inline or exported from the form component file.

9. **Naming**: Shared types in `lib/types.ts`. Constants in `lib/contants.tsx` (note: typo — "contants" not "constants"). Utility function `cn()` in `lib/utils.ts`.

10. **No testing**: No test files exist in the repository. `tsdx test --passWithNoTests` is configured for the SDK package only.

11. **No dark mode**: The SDK and app are explicitly light-mode only (noted in SDK README).

12. **Cache opt-out**: `next.config.mjs` sets `Cache-Control: no-store` on `/`, `/app/*`, and `/api/*`. Pages also call `cookies()` (e.g., `const _cookies = cookies()`) to opt out of Next.js caching.

13. **Watermark for free plan**: `app/api/posts/get/single/route.ts` appends a Markdown `[Atom](https://atomcms.vercel.app/)` link to post body when the project owner is on the `"single"` plan.

---

## Open Questions

See `context/open-questions.md`
