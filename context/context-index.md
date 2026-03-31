# Context Index

## Overview

Atom is a headless CMS built specifically for Next.js. It consists of two parts: (1) a Next.js dashboard web application where users create projects and manage blog posts via a markdown editor, and (2) a companion NPM package (`atom-nextjs`) that provides React server components for rendering blog content in any Next.js app.

The core stack is Next.js 14 (App Router), MongoDB via Mongoose, Lucia v3 for session-based authentication, Upstash Redis for API rate limiting, Tailwind CSS with shadcn/ui components, and Argon2 for password hashing. The dashboard is deployed at `cmsatom.netlify.app`.

Users sign up on the dashboard, create a "project" (a named collection of posts), write markdown content, and receive a `project_key`. They then install `atom-nextjs` in their own Next.js site and pass that key to pre-built server components (`AtomPage`, `AtomPost`) which fetch and render blog content directly from the Atom API.

## File Tree

```
/source
├── app/                            # Next.js App Router pages & API routes
│   ├── layout.tsx                  # Root layout (Montserrat font, Toaster)
│   ├── page.tsx                    # Landing/marketing page
│   ├── globals.css                 # Tailwind + CSS variables (shadcn theme)
│   ├── favicon.ico
│   ├── robots.ts                   # SEO robots.txt config
│   ├── sitemap.ts                  # Dynamic sitemap via atom-nextjs
│   ├── signin/page.tsx             # Sign-in page
│   ├── signup/page.tsx             # Sign-up page
│   ├── pricing/page.tsx            # Pricing plans page
│   ├── blog/
│   │   ├── page.tsx                # Public blog listing (uses atom-nextjs AtomPage)
│   │   └── [id]/page.tsx           # Single blog post (uses atom-nextjs AtomPost)
│   ├── app/                        # Authenticated dashboard area
│   │   ├── layout.tsx              # Wraps children in ProtectedRoute
│   │   ├── page.tsx                # Projects list (dashboard home)
│   │   ├── projects/[id]/page.tsx  # Single project view (post editor)
│   │   └── settings/
│   │       ├── page.tsx            # User settings (name, delete account)
│   │       └── billing/page.tsx    # Billing page (coming soon)
│   └── api/                        # REST API routes
│       ├── auth/
│       │   ├── signup/route.ts     # POST — user registration
│       │   ├── signin/route.ts     # POST — user login
│       │   ├── signout/route.ts    # POST — session invalidation
│       │   ├── delete/route.ts     # DELETE — delete user & all data
│       │   └── user/
│       │       ├── get/route.ts    # GET — fetch current user document
│       │       └── update/route.ts # PATCH — update user name
│       ├── posts/
│       │   ├── create/route.ts     # POST — create post in a project
│       │   ├── delete/route.ts     # DELETE — remove post from project
│       │   ├── get/single/route.ts # GET — get single post (by project_key Bearer token)
│       │   └── update/route.ts     # PATCH — update post fields
│       └── projects/
│           ├── create/route.ts     # POST — create a project
│           ├── delete/route.ts     # DELETE — delete project & refs
│           └── get/single/
│               ├── route.ts        # GET — get project (by session or Bearer)
│               └── client/route.ts # GET — get project for client SDK (Bearer, sanitized)
├── components/                     # React components
│   ├── cards/PricingPlanCard.tsx   # Pricing plan display card
│   ├── containers/
│   │   ├── AppContainer.tsx        # Dashboard layout with sidebar
│   │   ├── MainContainer.tsx       # Public page layout with navbar+footer
│   │   └── ProtectedRoute.tsx      # Server component — redirects to /signin if no session
│   ├── forms/
│   │   ├── LoginForm.tsx           # Sign-in form (client component)
│   │   └── SignupForm.tsx          # Sign-up form (client component)
│   ├── misc/
│   │   ├── NpmPackageComponent.tsx # npm install display widget
│   │   └── tracing-beam.tsx        # Decorative animation (framer-motion)
│   ├── modals/
│   │   ├── CreatePostModal.tsx     # Modal for creating a new post
│   │   └── DeleteUserModal.tsx     # Modal to confirm account deletion
│   ├── nav/Navbar.tsx              # Top navigation bar
│   ├── pages/
│   │   ├── projects/
│   │   │   ├── ProjectComponent.tsx     # Project view with sidebar + editor
│   │   │   ├── ProjectFormComponent.tsx # Post edit form (Zod validated)
│   │   │   └── ProjectPage.tsx          # Projects list + create dialog
│   │   └── settings/SettingsForm.tsx    # User settings form
│   ├── sidebars/
│   │   ├── AppSidebarNav.tsx            # Dashboard left nav sidebar
│   │   └── ProjectComponentSidebar.tsx  # Post list sidebar within a project
│   ├── tables/UserDocumentProjects/
│   │   ├── columns.tsx                  # TanStack Table column definitions
│   │   └── table.tsx                    # TanStack Table component
│   └── ui/                              # shadcn/ui primitives
│       ├── alert-dialog.tsx, button.tsx, carousel.tsx, dialog.tsx,
│       │   dropdown-menu.tsx, form.tsx, input.tsx, label.tsx,
│       │   markdown-editor.tsx, popover.tsx, sticky-scroll-reveal.tsx,
│       │   table.tsx, textarea.tsx
├── lib/
│   ├── types.ts                    # Core TypeScript types
│   ├── contants.tsx                # Plans, nav options, constants, base API URL
│   ├── utils.ts                    # cn() utility (clsx + tailwind-merge)
│   ├── utils/validateEmail.ts      # Email validation regex
│   ├── client/                     # Client-side API helpers (axios)
│   │   ├── auth/                   # loginUser, signupUser, signoutUser, deleteUser, updateUser
│   │   ├── posts/                  # createPost, deletePost, updatePost
│   │   └── projects/               # createProject, deleteProject
│   └── server/                     # Server-side utilities
│       ├── encoding/
│       │   ├── encodePassword.ts   # Argon2 hash with env salt
│       │   └── isPasswordValid.ts  # Argon2 verify
│       ├── functions/
│       │   ├── projects/getProject.ts  # Server-side project fetch (passes cookies)
│       │   └── user/fetchUser.ts       # Server-side user fetch (passes cookies)
│       ├── lucia/
│       │   ├── init.ts             # Lucia auth setup with MongoDB adapter
│       │   └── functions/validate-request.ts  # Cached session validation
│       ├── mongo/
│       │   ├── init.ts             # MongoDB connection + model refs
│       │   └── types/              # Mongoose schemas
│       │       ├── userCredentials.ts
│       │       ├── userDocuments.ts
│       │       ├── userProjects.ts
│       │       └── userSessions.ts
│       ├── redis/init.ts           # Upstash Redis + rate limiter (30 req/min)
│       └── utils/
│           ├── generateProjectKey.ts       # Random base64 project key
│           ├── validateProjectKey.ts       # Validate project key exists
│           └── validateRequestFetchUser.ts # Validate session + fetch user doc
├── packages/
│   └── atom-nextjs/                # Published NPM package (atom-nextjs)
│       ├── package.json            # v0.3.1, built with tsdx
│       ├── src/
│       │   ├── index.tsx           # Package exports
│       │   ├── components/
│       │   │   ├── Atom.tsx        # AtomPost — renders single blog post (server component)
│       │   │   ├── AtomBody.tsx    # MDX body renderer (compileMDX with remark-gfm)
│       │   │   ├── AtomPage.tsx    # Blog listing page (server component)
│       │   │   ├── AtomPostCard.tsx # Individual post card (link)
│       │   │   ├── AtomLoadingSkeleton.tsx  # Loading skeleton for blog list
│       │   │   └── AtomArticleSkeleton.tsx  # Loading skeleton for article
│       │   └── lib/
│       │       ├── types.ts        # Post, ClientPost, ClientProject, ApiResponse
│       │       ├── constants.ts    # baseAPIRoute (hardcoded to cmsatom.netlify.app)
│       │       └── client/
│       │           ├── getPost.ts           # Fetch single post via Bearer token
│       │           ├── getProject.ts        # Fetch project listing via Bearer token
│       │           ├── generatePostMetadata.ts  # Next.js metadata generation
│       │           └── generateSitemap.ts   # Sitemap generation helper
│       └── README.md               # Package documentation
├── bruno/                          # Bruno API collection for testing
│   ├── bruno.json
│   ├── environments/Development.bru
│   └── Routes/                     # API test requests
├── docs/                           # Documentation files
│   ├── nav.json                    # Navigation config
│   ├── introduction.md             # Introduction doc
│   └── omak.md                     # Pipeline test page
├── middleware.ts                    # Next.js middleware — rate limits /api/* routes
├── next.config.mjs                 # Cache-Control: no-store headers
├── package.json                    # Root package config
├── tailwind.config.ts              # Tailwind config (shadcn/ui theme)
├── tsconfig.json
├── postcss.config.js
└── components.json                 # shadcn/ui config
```

## Architecture

### System Design

Atom is a **two-part system**:

1. **Dashboard Web App** (this repo root) — A Next.js 14 App Router application where authenticated users manage their CMS projects and posts. Users sign up, create "projects" (each a named collection of blog posts), write content in a markdown editor, and receive a `project_key` they can use to fetch content from their own Next.js sites.

2. **Client SDK** (`packages/atom-nextjs/`) — An NPM package (`atom-nextjs`) that provides async server components (`AtomPage`, `AtomPost`) and utility functions (`generatePostMetadata`, `generateSitemap`) for rendering Atom-hosted blog content in any Next.js app.

### Data Flow

1. User signs up → creates a Project → gets a `project_key` (random base64 token prefixed with `atom-`)
2. User creates Posts within the Project via the dashboard markdown editor
3. In their own Next.js app, user installs `atom-nextjs`, passes `project_key` to `AtomPage`/`AtomPost` components
4. SDK components make server-side `fetch()` calls to Atom's API (Bearer `project_key`) → fetch project/post data → render MDX content

### Authentication Flow

- **Lucia v3** manages sessions with a MongoDB adapter
- Passwords are hashed with **Argon2** + an env-variable salt (`HASH_SALT`)
- Session cookies are set on signup/signin; `validateRequest()` (cached via React `cache()`) reads the cookie and validates against Lucia
- The `/app/*` routes are protected by `ProtectedRoute` server component which redirects to `/signin` if no session
- API routes that modify data call `validateRequest()` and check `user.id === project.creator_uid` for authorization

### API Authentication Modes

- **Session-based**: Dashboard API routes (POST/PATCH/DELETE) use Lucia session cookies
- **Bearer token**: Public read routes (`/api/posts/get/single`, `/api/projects/get/single/client`) accept `Authorization: Bearer <project_key>` header — used by the `atom-nextjs` SDK

### Rate Limiting

- Middleware at `middleware.ts` rate-limits all `/api/*` routes to **30 requests per minute per IP** using Upstash Redis sliding window

### Database

- **MongoDB** via Mongoose with 4 collections:
  - `credentials` — email/password hash (UserCredentials)
  - `documents` — user profile + project references (UserDocument)
  - `projects` — project with embedded posts array (Project)
  - `sessions` — Lucia sessions (Session)
- Multi-document transactions (mongoose sessions) used for signup, project creation/deletion, user deletion

## Key Concepts

- **Project** — A named collection of blog posts, identified by a UUID `_id` and authenticated via a `project_key`
  - Defined in: `lib/types.ts:L48-L56`, schema at `lib/server/mongo/types/userProjects.ts`
  - Created via: `app/api/projects/create/route.ts`

- **Post** — A blog post with markdown body, embedded within a Project's `posts` array
  - Defined in: `lib/types.ts:L12-L24`, schema at `lib/server/mongo/types/userProjects.ts:L7-L47`
  - Created via: `app/api/posts/create/route.ts`

- **UserDocument** — User profile with plan, name, email, and array of project references
  - Defined in: `lib/types.ts:L58-L67`, schema at `lib/server/mongo/types/userDocuments.ts`

- **UserCredentials** — Login credentials (email + argon2 hash)
  - Defined in: `lib/types.ts:L3-L10`, schema at `lib/server/mongo/types/userCredentials.ts`

- **Plan** — Pricing tier (`single` | `startup` | `business`) controlling project/post limits
  - Defined in: `lib/contants.tsx:L5` (plans array), details at `lib/contants.tsx:L33-L88`
  - Only `single` (free) is currently active; `startup` and `business` are `disabled: true`

- **Project Key** — Random base64 token prefixed with `atom-`, used as Bearer token for public API access
  - Generated at: `lib/server/utils/generateProjectKey.ts`

- **ApiResponse<T>** — Standard API response envelope: `{ success, message, response: T }`
  - Defined in: `app/api/auth/signup/route.ts:L16-L20`

## API Surface

All routes are under `/api/` and rate-limited via middleware.

### Auth Routes

| Method | Path | Handler | Purpose |
|--------|------|---------|---------|
| POST | `/api/auth/signup` | `app/api/auth/signup/route.ts` | Register user (email, password, first/last name) |
| POST | `/api/auth/signin` | `app/api/auth/signin/route.ts` | Login with email+password |
| POST | `/api/auth/signout` | `app/api/auth/signout/route.ts` | Invalidate session |
| DELETE | `/api/auth/delete` | `app/api/auth/delete/route.ts` | Delete account (requires password confirmation) |
| GET | `/api/auth/user/get` | `app/api/auth/user/get/route.ts` | Get current user document |
| PATCH | `/api/auth/user/update` | `app/api/auth/user/update/route.ts` | Update user name |

### Project Routes

| Method | Path | Handler | Purpose |
|--------|------|---------|---------|
| POST | `/api/projects/create` | `app/api/projects/create/route.ts` | Create project (session auth) |
| DELETE | `/api/projects/delete?project_id=` | `app/api/projects/delete/route.ts` | Delete project (session auth, owner check) |
| GET | `/api/projects/get/single?project_id=` | `app/api/projects/get/single/route.ts` | Get project (session auth OR Bearer token) |
| GET | `/api/projects/get/single/client` | `app/api/projects/get/single/client/route.ts` | Get project for SDK (Bearer only, sanitized — strips `project_key`, `creator_uid`) |

### Post Routes

| Method | Path | Handler | Purpose |
|--------|------|---------|---------|
| POST | `/api/posts/create?project_id=` | `app/api/posts/create/route.ts` | Create post in project (session auth, owner check) |
| DELETE | `/api/posts/delete?project_id=&post_id=` | `app/api/posts/delete/route.ts` | Delete post (session auth, owner check) |
| GET | `/api/posts/get/single?post_id=` | `app/api/posts/get/single/route.ts` | Get single post (Bearer `project_key`) |
| PATCH | `/api/posts/update?project_id=&post_id=` | `app/api/posts/update/route.ts` | Update post fields (session auth, owner check) |

## Data Model

### UserCredentials (`lib/types.ts:L3-L10`)
```
email: string (unique, lowercase, trimmed)
password_hash: string
_id: string (UUID)
createdAt: Date
updatedAt: Date
```

### UserDocument (`lib/types.ts:L58-L67`)
```
_id: string (UUID, same as credentials)
first_name: string
last_name: string
email: string
plan: "single" | "startup" | "business"
projects: UserDocumentProjects[] (embedded array of project references)
createdAt: Date
updatedAt: Date
```

### UserDocumentProjects (`lib/types.ts:L30-L37`)
```
id: string (project UUID)
title: string
createdAt: Date
updatedAt: Date
creator: { uid: string, email: string }
```

### Project (`lib/types.ts:L48-L56`)
```
_id: string (UUID)
title: string
posts: Post[] (embedded array)
project_key: string (atom-<base64>)
creator_uid: string (user UUID)
createdAt: Date
updatedAt: Date
```

### Post (`lib/types.ts:L12-L24`)
```
id: string (UUID)
title: string
author: string
body: string (markdown)
image: string | null
creator_uid: string
keywords: string[]
teaser: string
createdAt: Date
updatedAt: Date
```

### Session (`lib/types.ts:L69-L72`)
```
user_id: string
expires_at: Date
```

### Relationships
- UserCredentials._id == UserDocument._id (1:1)
- UserDocument.projects[] ← soft references to Project._id
- Project.creator_uid → UserDocument._id (ownership)
- Project.posts[] — embedded Post documents
- Session.user_id → UserCredentials._id

## Auth & Middleware

### Authentication Stack
- **Lucia v3** (`lib/server/lucia/init.ts`) — session-based auth with MongoDB adapter
- **Argon2** (`lib/server/encoding/`) — password hashing with env salt
- **validateRequest** (`lib/server/lucia/functions/validate-request.ts`) — cached session validation via React `cache()`, reads `lucia.sessionCookieName` cookie

### Middleware
- `middleware.ts` — Matches `/api/*` routes only. Rate limits by IP (30/min) using Upstash Redis sliding window. Returns `ApiResponse` JSON on rate limit exceeded.

### Protected Routes
- `components/containers/ProtectedRoute.tsx` — Server component that validates session and redirects to `/signin` if unauthenticated. Used in `app/app/layout.tsx` to protect all `/app/*` pages.

### API Authorization Pattern
Every mutating API route follows this pattern:
1. `connectToDatabase()`
2. `validateRequest()` → get `user`
3. If modifying a resource: verify `user.id === resource.creator_uid`

## Configuration

| Variable | Purpose | Default | File |
|----------|---------|---------|------|
| `HASH_SALT` | Salt appended to passwords before Argon2 hashing | None (required) | `lib/server/encoding/encodePassword.ts` |
| `MONGO_DB_URI` | MongoDB connection string | None (required) | `lib/contants.tsx:L30` |
| `ATOM_PROJECT_KEY` | Project key for the app's own blog (self-hosting) | None (required) | `app/blog/page.tsx`, `app/sitemap.ts` |
| `UPSTASH_REDIS_REST_URL` | Upstash Redis URL for rate limiting | None (required) | `lib/server/redis/init.ts` |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis auth token | None (required) | `lib/server/redis/init.ts` |
| `NEXT_PUBLIC_ENV` | `"dev"` or `"prod"` — controls base API URL | `"dev"` (→ localhost:3000) | `lib/contants.tsx:L28-L30` |
| `NODE_ENV` | Standard Node env — controls session cookie `secure` flag | — | `lib/server/lucia/init.ts` |

## Dependencies

### Key Runtime Dependencies
| Package | Purpose |
|---------|---------|
| `next` 14.1.0 | Framework (App Router) |
| `lucia` 3.1.1 | Session-based authentication |
| `@lucia-auth/adapter-mongodb` | Lucia MongoDB adapter |
| `mongoose` 8.1.2 | MongoDB ODM |
| `argon2` 0.40.1 | Password hashing |
| `@upstash/redis` + `@upstash/ratelimit` | Redis-based rate limiting |
| `atom-nextjs` 0.3.1 | Self-hosted blog rendering (own package) |
| `axios` | HTTP client for server→API calls |
| `zod` | Schema validation (forms) |
| `react-hook-form` + `@hookform/resolvers` | Form management |
| `@tanstack/react-query` | Client data fetching |
| `@tanstack/react-table` | Data tables |
| `@uiw/react-md-editor` | Markdown editor |
| `next-mdx-remote` | MDX rendering (used in atom-nextjs package) |
| `zustand` | State management |
| `framer-motion` | Animations |
| `react-hot-toast` | Toast notifications |
| `react-syntax-highlighter` | Code highlighting |
| `react-markdown` | Markdown rendering |
| `react-icons` | Icon library |
| `uuid` | UUID generation |

### UI Components
- **shadcn/ui** (Radix UI primitives + Tailwind) — configured in `components.json`, components in `components/ui/`
- **Tailwind CSS** with `tailwindcss-animate` and `@tailwindcss/typography`

## Build & Run

```bash
# Development
npm run dev          # or bun dev

# Build
npm run build

# Start production
npm run start

# Lint
npm run lint
```

### atom-nextjs package (local development)
```bash
cd packages/atom-nextjs
npm link              # Host locally
cd ../..
npm link atom-nextjs  # Connect to local version
```

### atom-nextjs package scripts
```bash
cd packages/atom-nextjs
npm run start    # tsdx watch
npm run build    # tsdx build
npm run test     # tsdx test
npm run lint     # tsdx lint
```

## Patterns & Conventions

### API Response Pattern
All API routes return `ApiResponse<T>` envelope:
```ts
{ success: boolean; message: string | null; response: T }
```

### Error Handling
- API routes use try/catch with `err.message || err` fallback
- MongoDB duplicate key (code 11000) gets special handling in signup route
- Client-side helpers throw on `!data.success`

### File Organization
- `lib/client/` — Browser-safe API call helpers (using axios)
- `lib/server/` — Server-only code (auth, DB, encoding)
- `app/api/` — API routes (Next.js App Router convention)
- `app/app/` — Dashboard pages (protected by layout)
- `components/` — Organized by purpose (containers, forms, modals, nav, pages, sidebars, tables, ui)

### Naming Conventions
- Route files: `route.ts` with exported HTTP method handlers (POST, GET, DELETE, PATCH)
- Types exported alongside routes (e.g., `CreatePostRequest`, `SignupRequestParams`)
- MongoDB model refs suffixed with `Ref` (e.g., `UserCredentialsRef`, `ProjectsRef`)
- Constants file is `contants.tsx` (note: typo in filename — "contants" not "constants")

### Component Patterns
- Server components for data fetching (pages, ProtectedRoute)
- Client components (`"use client"`) for interactivity (forms, project editor)
- `AppContainer` layout for dashboard pages (sidebar + main)
- `MainContainer` layout for public pages (navbar + footer)

### State Management
- Server state: Fetched in server components, passed as props
- Client state: `useState` for local UI state, `zustand` available but usage limited
- `react-hook-form` + `zod` for form validation

### Database Pattern
- `connectToDatabase()` called at start of every API route/server function
- Mongoose transactions for multi-collection operations (signup, project CRUD, user deletion)
- Posts are **embedded** within Project documents (not separate collection)

## Documentation Framework

- **Provider**: `plain` (plain markdown files)
- **File extension**: `.md`
- **Navigation config**: `docs/nav.json` (manicule-nav-json strategy)
- **Nav format**: `{ "nav": [{ "title": "...", "path": "..." }] }` — path is filename without extension
- **Frontmatter**: None required
- **Docs directory**: `docs/`
- **Existing docs**: `introduction.md` (comprehensive intro), `omak.md` (pipeline test page)

## Open Questions

See `/workspace/context/open-questions.md` for detailed analysis.
