# Context Index

## Overview

Atom is a headless CMS (Content Management System) built specifically for Next.js. It consists of two parts: (1) a Next.js 14 web application (the dashboard) where users create projects, write and manage blog posts in Markdown, and (2) a companion npm package (`atom-nextjs`) that developers install in their own Next.js sites to fetch and render blog content via API. The stack is Next.js 14 (App Router), MongoDB (via Mongoose), Lucia for session-based authentication, Upstash Redis for rate limiting, TailwindCSS with shadcn/ui, and Zod for validation.

## File Tree

```
/source
├── app/                          # Next.js App Router pages and API routes
│   ├── layout.tsx                # Root layout (Montserrat font, Toaster)
│   ├── page.tsx                  # Landing/marketing page
│   ├── globals.css               # Global CSS
│   ├── favicon.ico
│   ├── robots.ts                 # SEO robots.txt config
│   ├── sitemap.ts                # Dynamic sitemap using atom-nextjs SDK
│   ├── signin/page.tsx           # Sign-in page
│   ├── signup/page.tsx           # Sign-up page
│   ├── pricing/page.tsx          # Pricing plans display
│   ├── blog/                     # Public blog (uses atom-nextjs SDK)
│   │   ├── page.tsx              # Blog listing page
│   │   └── [id]/page.tsx         # Individual blog post page
│   ├── app/                      # Protected dashboard area
│   │   ├── layout.tsx            # Wraps children in ProtectedRoute
│   │   ├── page.tsx              # Projects listing (main dashboard)
│   │   ├── projects/[id]/page.tsx # Individual project view
│   │   └── settings/
│   │       ├── page.tsx          # User settings
│   │       └── billing/page.tsx  # Billing (coming soon)
│   └── api/                      # API routes
│       ├── auth/
│       │   ├── signup/route.ts   # POST - User registration
│       │   ├── signin/route.ts   # POST - User login
│       │   ├── signout/route.ts  # POST - User logout
│       │   ├── delete/route.ts   # DELETE - Delete user account
│       │   └── user/
│       │       ├── get/route.ts  # GET - Fetch current user document
│       │       └── update/route.ts # PATCH - Update user name
│       ├── posts/
│       │   ├── create/route.ts   # POST - Create post in project
│       │   ├── delete/route.ts   # DELETE - Delete post from project
│       │   ├── get/single/route.ts # GET - Get single post (by project_key)
│       │   └── update/route.ts   # PATCH - Update post
│       └── projects/
│           ├── create/route.ts   # POST - Create new project
│           ├── delete/route.ts   # DELETE - Delete project
│           └── get/single/
│               ├── route.ts      # GET - Get project (session or key auth)
│               └── client/route.ts # GET - Client-facing project fetch (key auth)
├── components/                   # React components
│   ├── cards/PricingPlanCard.tsx
│   ├── containers/
│   │   ├── AppContainer.tsx      # Dashboard layout wrapper (sidebar + main)
│   │   ├── MainContainer.tsx     # Public page layout (navbar + footer)
│   │   └── ProtectedRoute.tsx    # Server component auth guard
│   ├── forms/
│   │   ├── LoginForm.tsx
│   │   └── SignupForm.tsx
│   ├── misc/
│   │   ├── NpmPackageComponent.tsx
│   │   └── tracing-beam.tsx
│   ├── modals/
│   │   ├── CreatePostModal.tsx
│   │   └── DeleteUserModal.tsx
│   ├── nav/Navbar.tsx            # Public site navbar (auth-aware)
│   ├── pages/
│   │   ├── projects/
│   │   │   ├── ProjectComponent.tsx      # Project editor main component
│   │   │   ├── ProjectFormComponent.tsx  # Post editing form (Zod + react-hook-form)
│   │   │   └── ProjectPage.tsx           # Projects listing with create dialog
│   │   └── settings/SettingsForm.tsx
│   ├── sidebars/
│   │   ├── AppSidebarNav.tsx     # Dashboard sidebar navigation
│   │   └── ProjectComponentSidebar.tsx
│   ├── tables/UserDocumentProjects/
│   │   ├── columns.tsx
│   │   └── table.tsx
│   └── ui/                       # shadcn/ui components
│       ├── alert-dialog.tsx, button.tsx, carousel.tsx, dialog.tsx,
│       │   dropdown-menu.tsx, form.tsx, input.tsx, label.tsx,
│       │   markdown-editor.tsx, popover.tsx, sticky-scroll-reveal.tsx,
│       │   table.tsx, textarea.tsx
├── lib/
│   ├── types.ts                  # Core TypeScript types
│   ├── contants.tsx              # Constants: plans, navOptions, planDetails, baseAPIRoute
│   ├── utils.ts                  # cn() utility (clsx + tailwind-merge)
│   ├── utils/validateEmail.ts    # Email validation regex
│   ├── client/                   # Client-side API wrappers (using axios)
│   │   ├── auth/                 # deleteUser, loginUser, signoutUser, signupUser, updateUser
│   │   ├── posts/                # createPost, deletePost, updatePost
│   │   └── projects/             # createProject, deleteProject
│   └── server/
│       ├── encoding/
│       │   ├── encodePassword.ts # argon2 hashing with salt
│       │   └── isPasswordValid.ts # argon2 verification
│       ├── functions/
│       │   ├── projects/getProject.ts  # Server-side project fetch
│       │   └── user/fetchUser.ts       # Server-side user fetch via cookies
│       ├── lucia/
│       │   ├── init.ts                 # Lucia auth initialization
│       │   └── functions/validate-request.ts # Session validation (cached)
│       ├── mongo/
│       │   ├── init.ts                 # Mongoose connection + model refs
│       │   └── types/                  # Mongoose schemas
│       │       ├── userCredentials.ts
│       │       ├── userDocuments.ts
│       │       ├── userProjects.ts     # Includes Post + Project schemas
│       │       └── userSessions.ts
│       ├── redis/init.ts               # Upstash Redis + rate limiter
│       └── utils/
│           ├── generateProjectKey.ts   # Generates "atom-" prefixed base64 key
│           ├── validateProjectKey.ts   # Validates project key against DB
│           └── validateRequestFetchUser.ts # Validates session + fetches user doc
├── packages/
│   └── atom-nextjs/              # Published npm package (atom-nextjs)
│       ├── package.json          # v0.3.1, built with tsdx
│       ├── src/
│       │   ├── index.tsx         # Package exports
│       │   ├── components/
│       │   │   ├── Atom.tsx              # Single post renderer (server component)
│       │   │   ├── AtomBody.tsx          # MDX body renderer
│       │   │   ├── AtomPage.tsx          # Post listing page (server component)
│       │   │   ├── AtomPostCard.tsx      # Post card component
│       │   │   ├── AtomLoadingSkeleton.tsx
│       │   │   └── AtomArticleSkeleton.tsx
│       │   └── lib/
│       │       ├── types.ts              # Client-facing types (Post, ClientPost, ClientProject, ApiResponse)
│       │       ├── constants.ts          # Base API URL
│       │       └── client/
│       │           ├── getPost.ts        # Fetch single post by key + ID
│       │           ├── getProject.ts     # Fetch project by key
│       │           ├── generatePostMetadata.ts # Next.js metadata generation
│       │           └── generateSitemap.ts      # Sitemap route generation
│       ├── tsconfig.json
│       └── README.md
├── bruno/                        # Bruno API testing collection
│   ├── bruno.json
│   ├── environments/Development.bru
│   └── Routes/                   # API test requests
├── docs/                         # Documentation
│   ├── nav.json                  # Navigation config
│   ├── introduction.md
│   └── omak.md                   # Pipeline test page
├── middleware.ts                  # Rate limiting middleware for /api/* routes
├── next.config.mjs               # Cache-Control: no-store for /, /app/*, /api/*
├── package.json
├── tailwind.config.ts
├── tsconfig.json
├── components.json               # shadcn/ui config
└── postcss.config.js
```

## Architecture

### System Design

Atom follows a **two-part architecture**:

1. **Dashboard Application** — A full Next.js 14 App Router application where users sign up, create projects, and manage blog posts via a Markdown editor. It's both the admin interface and the API server.

2. **Client SDK (`atom-nextjs`)** — An npm package installed in end-user Next.js apps. It provides server components (`Atom`, `AtomPage`) that fetch content from the Atom API using a Bearer project key.

### Data Flow

1. User signs up → credentials stored in MongoDB `credentials` collection, user profile in `documents` collection, session in `sessions` collection (managed by Lucia).
2. User creates a project → stored in `projects` collection with a generated `project_key`. A summary is embedded in the user's `documents.projects` array.
3. User creates/edits posts → posts are stored as an embedded array within the project document.
4. End-user's Next.js app uses `atom-nextjs` SDK → sends `Authorization: Bearer <project_key>` header → Atom API returns project/post data.

### Key Architectural Patterns

- **Server Components for Auth**: The `ProtectedRoute` component and `Navbar` are async server components that validate sessions server-side using Lucia.
- **Two Auth Modes**: Dashboard routes use session cookies (Lucia). Public API routes for the SDK use Bearer token (project_key).
- **Embedded Documents**: Posts are embedded within Project documents (not separate collections).
- **MongoDB Transactions**: Signup, project creation, and project deletion use Mongoose transactions to maintain consistency between collections.
- **Rate Limiting**: All `/api/*` routes are rate limited via middleware using Upstash Redis (30 requests/minute sliding window).

## Key Concepts

- **Project** — A named collection of blog posts with a unique `project_key` for API access.
  - Defined in: `lib/types.ts:L47-L55`
  - Schema: `lib/server/mongo/types/userProjects.ts:L38-L68`
  - Created in: `app/api/projects/create/route.ts`

- **Post** — A blog post with title, author, body (Markdown), image, keywords, and teaser. Stored as embedded array in Project.
  - Defined in: `lib/types.ts:L10-L22`
  - Schema: `lib/server/mongo/types/userProjects.ts:L6-L36`
  - Created in: `app/api/posts/create/route.ts`

- **UserDocument** — User profile with name, email, plan tier, and project references.
  - Defined in: `lib/types.ts:L57-L65`
  - Schema: `lib/server/mongo/types/userDocuments.ts`

- **UserCredentials** — Email + hashed password for authentication.
  - Defined in: `lib/types.ts:L3-L9`
  - Schema: `lib/server/mongo/types/userCredentials.ts`

- **Plan** — Subscription tier ("single", "startup", "business") controlling limits.
  - Defined in: `lib/contants.tsx:L5` and `lib/types.ts:L24`
  - Plan details: `lib/contants.tsx:L36-L83`

- **project_key** — Base64 token prefixed with "atom-", used as Bearer auth for SDK API access.
  - Generated in: `lib/server/utils/generateProjectKey.ts`

- **atom-nextjs SDK** — Published npm package providing React server components for rendering blog content.
  - Entry: `packages/atom-nextjs/src/index.tsx`
  - Exports: `Atom`, `AtomBody`, `AtomPage`, `AtomPostCard`, `AtomLoadingSkeleton`, `AtomArticleSkeleton`, `generatePostMetadata`, `getPost`, `getProject`, `generateSitemap`

## API Surface

### Auth Routes
| Method | Path | File | Description |
|--------|------|------|-------------|
| POST | `/api/auth/signup` | `app/api/auth/signup/route.ts` | Register new user (email, password, first_name, last_name) |
| POST | `/api/auth/signin` | `app/api/auth/signin/route.ts` | Login with email + password |
| POST | `/api/auth/signout` | `app/api/auth/signout/route.ts` | Invalidate session |
| DELETE | `/api/auth/delete` | `app/api/auth/delete/route.ts` | Delete user account |
| GET | `/api/auth/user/get` | `app/api/auth/user/get/route.ts` | Get current user document (session auth) |
| PATCH | `/api/auth/user/update` | `app/api/auth/user/update/route.ts` | Update user first_name/last_name |

### Posts Routes
| Method | Path | File | Description |
|--------|------|------|-------------|
| POST | `/api/posts/create?project_id=` | `app/api/posts/create/route.ts` | Create post in project (session auth) |
| DELETE | `/api/posts/delete?project_id=&post_id=` | `app/api/posts/delete/route.ts` | Delete post (session auth) |
| GET | `/api/posts/get/single?post_id=` | `app/api/posts/get/single/route.ts` | Get single post (Bearer project_key auth) — used by SDK |
| PATCH | `/api/posts/update?project_id=&post_id=` | `app/api/posts/update/route.ts` | Update post fields (session auth) |

### Projects Routes
| Method | Path | File | Description |
|--------|------|------|-------------|
| POST | `/api/projects/create` | `app/api/projects/create/route.ts` | Create project (session auth) |
| DELETE | `/api/projects/delete?project_id=` | `app/api/projects/delete/route.ts` | Delete project (session auth) |
| GET | `/api/projects/get/single?project_id=` | `app/api/projects/get/single/route.ts` | Get project (session auth or Bearer key) |
| GET | `/api/projects/get/single/client` | `app/api/projects/get/single/client/route.ts` | Get project for client display (Bearer key, strips sensitive fields) |

### Page Routes
| Path | File | Description |
|------|------|-------------|
| `/` | `app/page.tsx` | Landing/marketing page |
| `/signin` | `app/signin/page.tsx` | Login page |
| `/signup` | `app/signup/page.tsx` | Registration page |
| `/pricing` | `app/pricing/page.tsx` | Pricing plans |
| `/blog` | `app/blog/page.tsx` | Blog listing (uses atom-nextjs SDK) |
| `/blog/[id]` | `app/blog/[id]/page.tsx` | Blog post (uses atom-nextjs SDK) |
| `/app` | `app/app/page.tsx` | Dashboard: projects listing (protected) |
| `/app/projects/[id]` | `app/app/projects/[id]/page.tsx` | Project editor (protected) |
| `/app/settings` | `app/app/settings/page.tsx` | User settings (protected) |
| `/app/settings/billing` | `app/app/settings/billing/page.tsx` | Billing (coming soon, protected) |

### Standard API Response Shape

All API responses follow the `ApiResponse<T>` type defined in `app/api/auth/signup/route.ts:L17-L21`:
```typescript
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```

## Data Model

### MongoDB Collections

```
credentials  → UserCredentials (email, password_hash, _id, timestamps)
documents    → UserDocument (_id, first_name, last_name, email, plan, projects[], timestamps)
projects     → Project (_id, title, posts[], project_key, creator_uid, timestamps)
sessions     → Session (user_id, expires_at) — managed by Lucia
```

### Type Definitions (lib/types.ts)

- **UserCredentials** (L3-L9): `email`, `password_hash`, `createdAt`, `updatedAt`, `_id`
- **Post** (L10-L22): `createdAt`, `id`, `updatedAt`, `title`, `author`, `body`, `image`, `creator_uid`, `keywords?`, `teaser`
- **Plan** (L24): `"single" | "startup" | "business"`
- **UserDocumentProjects** (L26-L32): `id`, `title`, `createdAt`, `updatedAt`, `creator`
- **UserDocumnetProjectsCreator** (L34-L37): `uid`, `email`
- **Project** (L39-L47): `title`, `_id`, `posts[]`, `project_key`, `creator_uid`, `createdAt`, `updatedAt`
- **UserDocument** (L49-L57): `_id`, `first_name`, `last_name`, `createdAt`, `updatedAt`, `projects[]`, `email`, `plan`
- **Session** (L59-L62): `user_id`, `expires_at`
- **PlanDetailsPlan** (L66-L77): Plan configuration with pricing, limits, features

### SDK Types (packages/atom-nextjs/src/lib/types.ts)

- **Post**: Same structure as main app
- **ClientPost**: Subset: `image?`, `id`, `teaser`, `title`, `createdAt`, `updatedAt`, `author`
- **ClientProject**: `title`, `posts[]`, `id`, `updatedAt`, `createdAt`
- **ApiResponse<T>**: `response`, `success`, `message`

### Relationships

- A **User** has one **UserDocument** and one **UserCredentials** record (shared `_id`).
- A **UserDocument** contains an embedded `projects[]` array (summary references).
- A **Project** contains an embedded `posts[]` array (full post data).
- A **Project** references its creator via `creator_uid`.

## Auth & Middleware

### Authentication
- **Library**: Lucia v3 (`lib/server/lucia/init.ts`)
- **Adapter**: MongoDB adapter connecting to `sessions` and `credentials` collections
- **Session validation**: `lib/server/lucia/functions/validate-request.ts` — cached via React `cache()`, reads session cookie, validates with Lucia, refreshes cookie if needed
- **Password hashing**: Argon2 with environment-provided salt (`lib/server/encoding/encodePassword.ts`)
- **Protected routes**: `components/containers/ProtectedRoute.tsx` — server component that redirects to `/signin` if no valid session
- **Route protection**: All `/app/*` pages are wrapped in `ProtectedRoute` via `app/app/layout.tsx`

### Two Auth Modes
1. **Session-based** (dashboard): Lucia session cookie for all dashboard API routes and pages
2. **Bearer token** (SDK): `Authorization: Bearer <project_key>` header for public post/project GET endpoints

### Middleware
- **File**: `middleware.ts` (root)
- **Scope**: Matches `/api/:path*` only
- **Function**: Rate limiting via Upstash Redis (30 requests/minute per IP)
- **Response on limit**: Returns `ApiResponse` with error message

## Configuration

| Variable | Purpose | Default | File |
|----------|---------|---------|------|
| `HASH_SALT` | Salt for Argon2 password hashing | None (required) | `lib/server/encoding/encodePassword.ts` |
| `MONGO_DB_URI` | MongoDB connection string | None (required) | `lib/contants.tsx`, `lib/server/mongo/init.ts` |
| `ATOM_PROJECT_KEY` | Project key for the app's own blog | None (required) | `app/blog/page.tsx`, `app/blog/[id]/page.tsx`, `app/sitemap.ts` |
| `UPSTASH_REDIS_REST_URL` | Upstash Redis URL for rate limiting | None (required) | `lib/server/redis/init.ts` |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis auth token | None (required) | `lib/server/redis/init.ts` |
| `NEXT_PUBLIC_ENV` | Environment flag (`dev` or `prod`) | Defaults to dev behavior | `lib/contants.tsx` |

### Plan Limits (lib/contants.tsx:L36-L83)

| Plan | Price | Max Projects | Max Posts | Max Body Length | Active |
|------|-------|-------------|-----------|----------------|--------|
| Single | Free | 2 | 100 | 10,000 chars | Yes (disabled=false) |
| Startup | $3.99 | 3 | 1,000 | 100,000 chars | No (disabled=true) |
| Business | $11.99 | 5 | 2,500 | 500,000 chars | No (disabled=true) |

## Dependencies

### Core
- **next** 14.1.0 — React framework (App Router)
- **react** ^18 — UI library
- **mongoose** ^8.1.2 — MongoDB ODM
- **lucia** ^3.1.1 — Session-based authentication
- **@lucia-auth/adapter-mongodb** ^1.0.2 — Lucia MongoDB adapter
- **argon2** ^0.40.1 — Password hashing
- **@upstash/redis** ^1.28.4 + **@upstash/ratelimit** ^1.0.1 — Rate limiting

### UI
- **@radix-ui/** — Headless UI primitives (dialog, dropdown, alert-dialog, popover, label, slot)
- **tailwindcss** ^3.3.0 + **tailwindcss-animate** + **@tailwindcss/typography** — Styling
- **class-variance-authority** + **clsx** + **tailwind-merge** — Utility classes
- **lucide-react** + **react-icons** — Icons
- **framer-motion** — Animations
- **@uiw/react-md-editor** — Markdown editor
- **react-hook-form** + **@hookform/resolvers** + **zod** — Form handling and validation

### Content
- **atom-nextjs** ^0.3.1 — The project's own SDK (used for the app's blog)
- **next-mdx-remote** — MDX rendering
- **react-markdown** + **react-syntax-highlighter** — Markdown rendering
- **rehype-katex** + **rehype-sanitize** + **remark-gfm** + **remark-math** — Markdown plugins
- **sanitize-html** — HTML sanitization

### Other
- **@tanstack/react-query** ^5.27.5 — Data fetching
- **@tanstack/react-table** ^8.13.2 — Table component
- **axios** ^1.6.7 — HTTP client
- **zustand** ^4.5.0 — State management
- **uuid** ^9.0.1 — UUID generation
- **react-hot-toast** — Toast notifications
- **react-tweet** — Twitter embed

## Build & Run

```bash
# Install dependencies
npm install  # or bun install

# Development
npm run dev     # next dev (localhost:3000)

# Production build
npm run build   # next build
npm run start   # next start

# Lint
npm run lint    # next lint
```

### Running atom-nextjs SDK locally
```bash
cd packages/atom-nextjs
npm link
cd ../..
npm link atom-nextjs
```

### Bruno API Testing
API tests are in `bruno/Routes/` using the Bruno API client. Environment variables configured in `bruno/environments/Development.bru`.

## Patterns & Conventions

1. **API Response Pattern**: All API routes return `ApiResponse<T>` with `{ success, message, response }`.
2. **Error Handling**: Try-catch in every API route, errors returned as JSON (never thrown to client).
3. **Auth Check Pattern**: API routes call `connectToDatabase()` then `validateRequest()` to get the session user.
4. **Authorization**: After auth check, routes verify `user.id === resource.creator_uid` for ownership.
5. **File Organization**: Client-side API wrappers in `lib/client/`, server-side utilities in `lib/server/`.
6. **Component Organization**: UI primitives in `components/ui/` (shadcn/ui), feature components in `components/pages/`, layout in `components/containers/`.
7. **Server vs Client Components**: Auth-related and data-fetching components are server components. Interactive components (forms, sidebars) use `"use client"`.
8. **Naming**: File name `contants.tsx` has a typo (missing 's' — should be "constants"). Types file has typo `UserDocumnetProjectsCreator` (should be "Document").
9. **MongoDB Transactions**: Multi-collection writes use `mongoose.startSession()` + `withTransaction()`.
10. **UUID Usage**: User IDs and post IDs generated with `uuid/v4`. Project IDs also use UUID.
11. **Project Key Format**: `"atom-" + randomBytes(32).toString("base64")`.
12. **Watermark**: Single (free) plan users get an Atom watermark appended to post bodies on retrieval.

## Documentation Framework

- **Provider**: `plain` (no framework-specific doc provider)
- **File extension**: `.md`
- **Navigation config**: `docs/nav.json` — JSON with `{ "nav": [{ "title": "...", "path": "..." }] }`
- **Nav strategy**: `manicule-nav-json`
- **Docs directory**: `docs/`
- **Frontmatter fields**: None required
- **MDX components**: Not used

## Open Questions

See `/workspace/context/open-questions.md` for detailed analysis.
