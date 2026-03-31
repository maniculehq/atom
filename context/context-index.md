# Context Index

## Overview

Atom is a headless CMS built for Next.js. It lets users create blog projects, write markdown posts in a dashboard, and render them in their own Next.js sites via a companion SDK package (`atom-nextjs`). The main app is a Next.js 14 App Router project using MongoDB (Mongoose), Lucia for session-based auth, Upstash Redis for rate limiting, Tailwind CSS, shadcn/ui components, and Argon2 password hashing. The codebase also contains the `atom-nextjs` npm package as a sub-package.

The repository is structured as a monorepo: the root is a Next.js 14 App Router application (the Atom dashboard and API at `app/`), with a publishable npm package at `packages/atom-nextjs/` that provides React server components (`Atom`, `AtomPage`, `AtomBody`) and helper functions (`getPost`, `getProject`, `generatePostMetadata`, `generateSitemap`) for rendering blog content. The dashboard exposes 14 REST API endpoints under `/api/` for auth, post, and project management. Data is stored in four MongoDB collections (`credentials`, `documents`, `projects`, `sessions`) accessed via Mongoose, with Lucia v3 handling session-cookie authentication for dashboard users and Bearer `project_key` tokens authenticating SDK consumers. Rate limiting is enforced on all API routes through Upstash Redis middleware (30 requests/minute per IP).

## File Tree

```
/source
├── app/                          # Next.js App Router pages & API routes
│   ├── layout.tsx                # Root layout (Montserrat font, Toaster)
│   ├── page.tsx                  # Landing/marketing page
│   ├── globals.css               # Tailwind + CSS variables (shadcn theme)
│   ├── favicon.ico
│   ├── robots.ts                 # SEO robots.txt generation
│   ├── sitemap.ts                # Dynamic sitemap using atom-nextjs SDK
│   ├── signin/page.tsx           # Sign-in page
│   ├── signup/page.tsx           # Sign-up page
│   ├── pricing/page.tsx          # Pricing plans page
│   ├── blog/
│   │   ├── page.tsx              # Blog listing (uses AtomPage from SDK)
│   │   └── [id]/page.tsx         # Single blog post (uses Atom from SDK)
│   ├── app/                      # Authenticated dashboard area
│   │   ├── layout.tsx            # ProtectedRoute wrapper
│   │   ├── page.tsx              # Projects list page
│   │   ├── projects/[id]/page.tsx # Single project editor
│   │   └── settings/
│   │       ├── page.tsx          # User settings
│   │       └── billing/page.tsx  # Billing (placeholder)
│   └── api/                      # API route handlers
│       ├── auth/
│       │   ├── signup/route.ts   # POST — register user
│       │   ├── signin/route.ts   # POST — login user
│       │   ├── signout/route.ts  # POST — logout user
│       │   ├── delete/route.ts   # DELETE — delete user account
│       │   └── user/
│       │       ├── get/route.ts  # GET — fetch current user
│       │       └── update/route.ts # PATCH — update user profile
│       ├── posts/
│       │   ├── create/route.ts   # POST — create post in project
│       │   ├── delete/route.ts   # DELETE — delete post from project
│       │   ├── get/single/route.ts # GET — get single post (Bearer auth)
│       │   └── update/route.ts   # PATCH — update post
│       └── projects/
│           ├── create/route.ts   # POST — create project
│           ├── delete/route.ts   # DELETE — delete project
│           └── get/single/
│               ├── route.ts      # GET — get project (session or Bearer)
│               └── client/route.ts # GET — get project for client SDK (Bearer)
├── components/                   # React components
│   ├── cards/PricingPlanCard.tsx
│   ├── containers/
│   │   ├── AppContainer.tsx      # Dashboard layout with sidebar
│   │   ├── MainContainer.tsx     # Public pages layout with navbar + footer
│   │   └── ProtectedRoute.tsx    # Server-side auth gate (redirects to /signin)
│   ├── forms/
│   │   ├── LoginForm.tsx         # Zod-validated login form
│   │   └── SignupForm.tsx        # Zod-validated signup form
│   ├── misc/
│   │   ├── NpmPackageComponent.tsx
│   │   └── tracing-beam.tsx
│   ├── modals/
│   │   ├── CreatePostModal.tsx
│   │   └── DeleteUserModal.tsx
│   ├── nav/Navbar.tsx            # Main navbar (server component, auth-aware)
│   ├── pages/
│   │   ├── projects/
│   │   │   ├── ProjectComponent.tsx  # Project editor with sidebar + form
│   │   │   ├── ProjectFormComponent.tsx # Post edit form
│   │   │   └── ProjectPage.tsx       # Projects listing page
│   │   └── settings/SettingsForm.tsx
│   ├── sidebars/
│   │   ├── AppSidebarNav.tsx
│   │   └── ProjectComponentSidebar.tsx
│   ├── tables/UserDocumentProjects/
│   │   ├── columns.tsx
│   │   └── table.tsx
│   └── ui/                       # shadcn/ui primitives
│       ├── alert-dialog.tsx, button.tsx, carousel.tsx, dialog.tsx,
│       │   dropdown-menu.tsx, form.tsx, input.tsx, label.tsx,
│       │   markdown-editor.tsx, popover.tsx, sticky-scroll-reveal.tsx,
│       │   table.tsx, textarea.tsx
├── lib/                          # Shared libraries
│   ├── types.ts                  # Core TypeScript types
│   ├── contants.tsx              # Constants (plans, nav, API base URL)  [sic: "contants"]
│   ├── utils.ts                  # cn() utility (clsx + tailwind-merge)
│   ├── utils/validateEmail.ts    # Email regex validator
│   ├── client/                   # Client-side API call helpers
│   │   ├── auth/                 # deleteUser, loginUser, signoutUser, signupUser, updateUser
│   │   ├── posts/                # createPost, deletePost, updatePost
│   │   └── projects/             # createProject, deleteProject
│   └── server/                   # Server-side utilities
│       ├── encoding/
│       │   ├── encodePassword.ts  # Argon2 hashing with salt
│       │   └── isPasswordValid.ts # Argon2 verification
│       ├── functions/
│       │   ├── projects/getProject.ts  # Server-side project fetch
│       │   └── user/fetchUser.ts       # Server-side user fetch
│       ├── lucia/
│       │   ├── init.ts            # Lucia auth initialization
│       │   └── functions/validate-request.ts # Session validation (cached)
│       ├── mongo/
│       │   ├── init.ts            # Mongoose connection + model refs
│       │   └── types/             # Mongoose schemas
│       │       ├── userCredentials.ts
│       │       ├── userDocuments.ts
│       │       ├── userProjects.ts
│       │       └── userSessions.ts
│       ├── redis/init.ts          # Upstash Redis + Ratelimit setup
│       └── utils/
│           ├── generateProjectKey.ts
│           ├── validateProjectKey.ts
│           └── validateRequestFetchUser.ts
├── middleware.ts                  # Rate limiting on /api/* routes
├── packages/
│   └── atom-nextjs/              # NPM SDK package (atom-nextjs)
│       ├── package.json          # v0.3.1, built with tsdx
│       ├── README.md
│       ├── src/
│       │   ├── index.tsx          # Package exports
│       │   ├── components/
│       │   │   ├── Atom.tsx       # Single post renderer (SSR)
│       │   │   ├── AtomBody.tsx   # MDX body compiler
│       │   │   ├── AtomPage.tsx   # Post listing page
│       │   │   ├── AtomPostCard.tsx # Post card link
│       │   │   ├── AtomLoadingSkeleton.tsx
│       │   │   └── AtomArticleSkeleton.tsx
│       │   └── lib/
│       │       ├── client/
│       │       │   ├── generatePostMetadata.ts
│       │       │   ├── generateSitemap.ts
│       │       │   ├── getPost.ts
│       │       │   └── getProject.ts
│       │       ├── constants.ts   # baseAPIRoute (cmsatom.netlify.app)
│       │       └── types.ts       # Post, ClientPost, ClientProject, ApiResponse
│       └── tsconfig.json
├── bruno/                        # Bruno API collection for testing
│   ├── bruno.json
│   ├── environments/Development.bru
│   └── Routes/                   # API test files
├── docs/                         # Documentation (plain markdown)
│   ├── nav.json                  # Navigation config
│   ├── introduction.md
│   └── omak.md                   # Pipeline test page
├── package.json                  # Main app package
├── next.config.mjs               # Cache-Control headers config
├── tailwind.config.ts
├── tsconfig.json
├── components.json               # shadcn/ui config
├── postcss.config.js
└── middleware.ts                  # Rate limiter middleware
```

## Architecture

### System Overview

Atom is a two-part system:

1. **Atom Dashboard (this repo's main app)** — A Next.js 14 App Router application where users sign up, create "projects" (blog containers), and write/edit posts via a markdown editor. The dashboard exposes a REST API that both the dashboard itself and the SDK consume.

2. **`atom-nextjs` SDK (sub-package)** — An npm package that end-users install in their own Next.js apps. It provides server components (`Atom`, `AtomPage`) that fetch content from the Atom API using a `project_key` as a Bearer token.

### Data Flow

```
User's Next.js App          Atom Dashboard/API              MongoDB
      |                           |                           |
      |-- Bearer project_key ---->|                           |
      |                           |--- Mongoose query ------->|
      |<-- JSON (posts/project) --|<-- Document --------------|
      |                           |                           |
```

Dashboard users authenticate via session cookies (Lucia). SDK consumers authenticate via Bearer `project_key` tokens.

### Key Architectural Patterns

- **Server Components**: The Navbar, ProtectedRoute, and page-level data fetching are all async server components.
- **Client Components**: Forms, modals, project editors are `"use client"` with `react-hook-form` + `zod` for validation.
- **API Route Pattern**: All API routes follow a consistent `ApiResponse<T>` response shape (`{ success, message, response }`).
- **Dual Auth**: API routes support both session-cookie auth (dashboard) and Bearer token auth (SDK clients).
- **MongoDB Transactions**: User signup, project creation, and deletion use MongoDB sessions/transactions for atomicity.
- **Rate Limiting**: Middleware applies Upstash Redis sliding window rate limiting (30 req/min) on all `/api/*` routes.

## Key Concepts

- **Project** — A named collection of blog posts, identified by `_id` (UUID) and `project_key` (random base64 token). Created by authenticated users.
  - Defined in: `lib/types.ts:L46-L54`
  - Schema: `lib/server/mongo/types/userProjects.ts`
  - API: `app/api/projects/create/route.ts`, `app/api/projects/delete/route.ts`, `app/api/projects/get/single/route.ts`

- **Post** — A blog post embedded within a Project document as an array element. Contains title, author, body (markdown), image, keywords, teaser.
  - Defined in: `lib/types.ts:L10-L22`
  - Schema: `lib/server/mongo/types/userProjects.ts` (postSchema)
  - API: `app/api/posts/create/route.ts`, `app/api/posts/update/route.ts`, `app/api/posts/delete/route.ts`, `app/api/posts/get/single/route.ts`

- **UserDocument** — User profile data (name, email, plan, projects list).
  - Defined in: `lib/types.ts:L56-L64`
  - Schema: `lib/server/mongo/types/userDocuments.ts`

- **UserCredentials** — Authentication credentials (email, password_hash).
  - Defined in: `lib/types.ts:L3-L8`
  - Schema: `lib/server/mongo/types/userCredentials.ts`

- **Plan** — Subscription tier (`"single" | "startup" | "business"`). Controls max projects, posts, and body length.
  - Defined in: `lib/types.ts:L24`, `lib/contants.tsx:L3` (plans array)
  - Plan details: `lib/contants.tsx:L33-L86` (planDetails array)

- **Session** — Lucia auth session stored in MongoDB.
  - Defined in: `lib/types.ts:L66-L69`
  - Schema: `lib/server/mongo/types/userSessions.ts`

- **ApiResponse<T>** — Standard API response type used across all routes.
  - Defined in: `app/api/auth/signup/route.ts:L17-L21`

## API Surface

All API routes are under `/api/` and return `ApiResponse<T>` JSON.

### Auth Routes

| Method | Path | Purpose | Auth | File |
|--------|------|---------|------|------|
| POST | `/api/auth/signup` | Register new user | None | `app/api/auth/signup/route.ts` |
| POST | `/api/auth/signin` | Login user | None | `app/api/auth/signin/route.ts` |
| POST | `/api/auth/signout` | Logout user | Session | `app/api/auth/signout/route.ts` |
| DELETE | `/api/auth/delete` | Delete user account | Session + password | `app/api/auth/delete/route.ts` |
| GET | `/api/auth/user/get` | Get current user document | Session | `app/api/auth/user/get/route.ts` |
| PATCH | `/api/auth/user/update` | Update user name | Session | `app/api/auth/user/update/route.ts` |

### Post Routes

| Method | Path | Purpose | Auth | File |
|--------|------|---------|------|------|
| POST | `/api/posts/create?project_id=` | Create post | Session | `app/api/posts/create/route.ts` |
| DELETE | `/api/posts/delete?project_id=&post_id=` | Delete post | Session | `app/api/posts/delete/route.ts` |
| PATCH | `/api/posts/update?project_id=&post_id=` | Update post | Session | `app/api/posts/update/route.ts` |
| GET | `/api/posts/get/single?post_id=` | Get single post | Bearer token | `app/api/posts/get/single/route.ts` |

### Project Routes

| Method | Path | Purpose | Auth | File |
|--------|------|---------|------|------|
| POST | `/api/projects/create` | Create project | Session | `app/api/projects/create/route.ts` |
| DELETE | `/api/projects/delete?project_id=` | Delete project | Session | `app/api/projects/delete/route.ts` |
| GET | `/api/projects/get/single?project_id=` | Get project (dashboard) | Session or Bearer | `app/api/projects/get/single/route.ts` |
| GET | `/api/projects/get/single/client` | Get project (SDK client) | Bearer token | `app/api/projects/get/single/client/route.ts` |

### Page Routes

| Path | Purpose | File |
|------|---------|------|
| `/` | Landing/marketing page | `app/page.tsx` |
| `/signin` | Login page | `app/signin/page.tsx` |
| `/signup` | Registration page | `app/signup/page.tsx` |
| `/pricing` | Pricing plans | `app/pricing/page.tsx` |
| `/blog` | Blog listing (uses own SDK) | `app/blog/page.tsx` |
| `/blog/[id]` | Single blog post | `app/blog/[id]/page.tsx` |
| `/app` | Dashboard: projects list | `app/app/page.tsx` |
| `/app/projects/[id]` | Dashboard: project editor | `app/app/projects/[id]/page.tsx` |
| `/app/settings` | Dashboard: user settings | `app/app/settings/page.tsx` |
| `/app/settings/billing` | Dashboard: billing (placeholder) | `app/app/settings/billing/page.tsx` |

## Data Model

### MongoDB Collections

| Collection | Mongoose Model | TypeScript Type | Schema File |
|------------|---------------|-----------------|-------------|
| `credentials` | `UserCredentialsRef` | `UserCredentials` | `lib/server/mongo/types/userCredentials.ts` |
| `documents` | `UserDocumentsRef` | `UserDocument` | `lib/server/mongo/types/userDocuments.ts` |
| `projects` | `ProjectsRef` | `Project` | `lib/server/mongo/types/userProjects.ts` |
| `sessions` | `SessionRef` | `Session` | `lib/server/mongo/types/userSessions.ts` |

### Type Definitions (lib/types.ts)

- **UserCredentials**: `{ email, password_hash, createdAt, updatedAt, _id }`
- **Post**: `{ createdAt, id, updatedAt, title, author, body, image, creator_uid, keywords?, teaser }`
- **UserDocumentProjects**: `{ id, title, createdAt, updatedAt, creator: { uid, email } }`
- **Project**: `{ title, _id, posts: Post[], project_key, creator_uid, createdAt, updatedAt }`
- **UserDocument**: `{ _id, first_name, last_name, createdAt, updatedAt, projects: UserDocumentProjects[], email, plan }`
- **Session**: `{ user_id, expires_at }`
- **PlanDetailsPlan**: `{ title, id, price, description, max_docs, max_body_length, features, max_projects, active, disabled }`

### Relationships
- A **User** has one `UserCredentials` doc and one `UserDocument` doc (same `_id`).
- A **UserDocument** embeds an array of `UserDocumentProjects` (denormalized project references).
- A **Project** embeds an array of `Post` documents (posts are stored inside the project).
- Projects reference their creator via `creator_uid`.

## Auth & Middleware

### Authentication (Lucia v3)
- **Library**: Lucia v3 with MongoDB adapter
- **Init**: `lib/server/lucia/init.ts` — Creates Lucia instance with MongoDB session/credential collections
- **Session validation**: `lib/server/lucia/functions/validate-request.ts` — Cached function that reads session cookie, validates via Lucia, refreshes if needed
- **Password hashing**: Argon2 with configurable `HASH_SALT` env var
  - Encode: `lib/server/encoding/encodePassword.ts`
  - Verify: `lib/server/encoding/isPasswordValid.ts`

### Middleware
- **File**: `middleware.ts`
- **Scope**: Matches `/api/:path*` only
- **Function**: Rate limits API requests using Upstash Redis (30 requests per minute per IP, sliding window)
- **Rate limiter**: `lib/server/redis/init.ts`

### Route Protection
- **ProtectedRoute**: `components/containers/ProtectedRoute.tsx` — Server component that validates session and redirects to `/signin` if unauthenticated. Used as layout wrapper for `/app/*` routes.
- **API route auth**: Each API route calls `validateRequest()` individually to check sessions.

## Configuration

| Variable | Purpose | Default | File |
|----------|---------|---------|------|
| `HASH_SALT` | Salt appended to passwords before Argon2 hashing | Required | `lib/server/encoding/encodePassword.ts` |
| `MONGO_DB_URI` | MongoDB connection string | Required | `lib/contants.tsx`, `lib/server/mongo/init.ts` |
| `ATOM_PROJECT_KEY` | Project key for Atom's own blog | Required | `app/blog/page.tsx`, `app/sitemap.ts` |
| `UPSTASH_REDIS_REST_URL` | Upstash Redis REST API URL | Required | `lib/server/redis/init.ts` |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis auth token | Required | `lib/server/redis/init.ts` |
| `NEXT_PUBLIC_ENV` | Environment flag (`"dev"` or `"prod"`) | `"dev"` | `lib/contants.tsx` |
| `ENV` | Environment flag (mentioned in README) | `"dev"` | README.md |

### Base API URL Logic (`lib/contants.tsx:L28-L31`)
- `prod`: `https://cmsatom.netlify.app/api`
- `dev`: `http://localhost:3000/api`

### SDK Base API URL (`packages/atom-nextjs/src/lib/constants.ts`)
- Hardcoded to: `https://cmsatom.netlify.app/api`

## Dependencies

### Main App (Notable)
| Package | Purpose |
|---------|---------|
| `next` 14.1.0 | Framework |
| `lucia` 3.1.1 | Session-based authentication |
| `@lucia-auth/adapter-mongodb` | Lucia MongoDB adapter |
| `mongoose` 8.1.2 | MongoDB ODM |
| `argon2` | Password hashing |
| `@upstash/redis` + `@upstash/ratelimit` | Rate limiting |
| `zod` + `@hookform/resolvers` + `react-hook-form` | Form validation |
| `@tanstack/react-query` | Client-side data fetching |
| `@tanstack/react-table` | Data tables |
| `@uiw/react-md-editor` | Markdown editor |
| `react-markdown` + `remark-gfm` + `rehype-*` | Markdown rendering |
| `zustand` | Client-side state management |
| `axios` | HTTP client |
| `atom-nextjs` | Own SDK package (also used by the app itself for its blog) |
| `framer-motion` | Animations |
| `tailwindcss` + shadcn/ui (`@radix-ui/*`) | UI framework |

### SDK Package (`atom-nextjs`)
| Package | Purpose |
|---------|---------|
| `next` | Peer dependency |
| `next-mdx-remote` | MDX compilation for post bodies |
| `react-loading-skeleton` | Loading skeletons |
| `remark-gfm` + `rehype-sanitize` | Markdown processing |

## Build & Run

### Main App
```bash
npm run dev      # Start dev server (localhost:3000)
npm run build    # Production build
npm run start    # Start production server
npm run lint     # ESLint
```

### SDK Package (`packages/atom-nextjs`)
```bash
npm run start    # tsdx watch mode
npm run build    # tsdx build
npm run test     # tsdx test
npm link         # Link locally for development
```

To develop the SDK locally with the main app:
1. `cd packages/atom-nextjs && npm link`
2. `cd ../.. && npm link atom-nextjs`

## Patterns & Conventions

### API Response Pattern
All API routes return `ApiResponse<T>`:
```typescript
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```

### Error Handling
- API routes use try/catch blocks, returning `{ success: false, message: err.message }` on error.
- Client-side helpers throw errors from API responses (`if (!data.success) throw new Error(data.message)`).
- Toast notifications via `react-hot-toast` on client-side operations.

### Auth Pattern
- Dashboard API routes: Call `validateRequest()` then check `if (!user) throw`.
- Owner verification: Check `project.creator_uid === user.id` before mutations.
- SDK-facing routes: Parse `Authorization: Bearer <project_key>` header.

### File Naming
- API routes: `app/api/<resource>/<action>/route.ts`
- Client helpers: `lib/client/<resource>/<action>.ts`
- Server functions: `lib/server/functions/<resource>/<function>.ts`
- MongoDB schemas: `lib/server/mongo/types/<collection>.ts`
- UI components: `components/ui/<name>.tsx` (shadcn)
- Page components: `components/pages/<section>/<Component>.tsx`
- Note: Constants file has typo in name: `lib/contants.tsx` (missing "s")

### Component Patterns
- Server components for data fetching and auth checks (Navbar, ProtectedRoute, page components)
- Client components for interactive UI (`"use client"` directive)
- Form validation with Zod schemas and react-hook-form
- shadcn/ui for base UI components (Radix primitives + Tailwind)

### Database Pattern
- Posts are embedded in Project documents (not separate collection)
- User data is split across `credentials` (auth) and `documents` (profile) collections
- Project references are denormalized into UserDocument.projects array
- MongoDB transactions used for operations spanning multiple collections

## Documentation Framework

- **Provider**: `plain` (no framework, plain markdown)
- **File extension**: `.md`
- **Docs directory**: `docs/`
- **Navigation config**: `docs/nav.json` — JSON file with `{ "nav": [{ "title": "...", "path": "..." }] }` structure
- **Navigation strategy**: `manicule-nav-json`
- **Frontmatter**: None required
- **MDX components**: Not used

## Open Questions

See `/workspace/context/open-questions.md` for detailed analysis.

1. Where is the app currently deployed — Netlify or Vercel? (Code references both `cmsatom.netlify.app` and `atomcms.vercel.app`)
2. Is billing/payments actually implemented? (Billing page shows "Coming soon..." and paid plans are `disabled: true`)
