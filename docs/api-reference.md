# API Reference

Atom exposes a REST API built on Next.js Route Handlers. All endpoints live under `/api` and return a consistent JSON envelope. There are two distinct authentication models depending on who's calling:

- **Session auth** (cookie-based, Lucia v3): used by the Atom dashboard and its internal client helpers in `lib/client/`.
- **Bearer auth** (project key): used by the `atom-nextjs` SDK and any public-facing code that needs to read posts or project data.

Every request to `/api/*` is subject to a **rate limit of 30 requests per minute** per IP address, enforced by Upstash Redis using a sliding window algorithm (`middleware.ts`).

---

## Contents

- [Response envelope](#response-envelope)
- [Authentication](#authentication)
- [Data shapes](#data-shapes)
- [Auth endpoints](#auth-endpoints)
  - [POST /api/auth/signup](#post-apiauthsignup)
  - [POST /api/auth/signin](#post-apiauthsignin)
  - [POST /api/auth/signout](#post-apiauthsignout)
  - [GET /api/auth/user/get](#get-apiauthUserget)
  - [PATCH /api/auth/user/update](#patch-apiauthUserupdate)
  - [DELETE /api/auth/delete](#delete-apiauthdelete)
- [Project endpoints](#project-endpoints)
  - [POST /api/projects/create](#post-apiprojectscreate)
  - [DELETE /api/projects/delete](#delete-apiprojectsdelete)
  - [GET /api/projects/get/single](#get-apiprojectsgetsingle)
  - [GET /api/projects/get/single/client](#get-apiprojectsgetsingleclient)
- [Post endpoints](#post-endpoints)
  - [POST /api/posts/create](#post-apipostscreate)
  - [PATCH /api/posts/update](#patch-apipostsupdate)
  - [DELETE /api/posts/delete](#delete-apipostsdelete)
  - [GET /api/posts/get/single](#get-apipostsgetsingle)
- [The atom-nextjs SDK](#the-atom-nextjs-sdk)
- [Plan limits summary](#plan-limits-summary)
- [Error handling](#error-handling)

---

## Response envelope

Every endpoint returns the same shape:

```ts
// Defined in app/api/auth/signup/route.ts
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```

When a request succeeds, `success` is `true` and `response` contains the data. When it fails, `success` is `false`, `response` is `null`, and `message` describes what went wrong. You never need to check HTTP status codes for business logic — check `success` instead.

---

## Authentication

### Session-based (dashboard routes)

On successful sign-in or sign-up, Lucia creates a session document in MongoDB and sets an `auth_session` cookie. Every protected route calls `validateRequest()` internally:

```ts
// lib/server/lucia/functions/validate-request.ts
const { user, session } = await validateRequest();
if (!user) throw new Error("Invalid session. Please sign in.");
```

You don't call this directly. The dashboard's client helpers in `lib/client/` use `axios` and the browser automatically forwards the cookie.

### Bearer token (SDK / public routes)

Routes meant to be called from a customer's blog accept `Authorization: Bearer <project_key>`. The project key looks like `atom-<32-byte base64 string>` and is generated when you create a project. Find it in the Atom dashboard under your project settings.

```http
Authorization: Bearer atom-abc123...
```

---

## Data shapes

These TypeScript types are defined in `lib/types.ts` and are used throughout all route responses.

### `Post`

```ts
type Post = {
  id: string;           // UUID
  title: string;
  author: string;
  body: string;         // Raw Markdown/MDX string
  teaser: string;       // Short summary, max 100 characters
  image: string | null; // URL to cover image
  keywords?: string[];  // Parsed from comma-separated input
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
};
```

`body` is stored as raw Markdown. The `atom-nextjs` SDK renders it via `next-mdx-remote` with `remark-gfm` and `rehype-sanitize` applied by default.

### `Project`

```ts
type Project = {
  _id: string;          // UUID
  title: string;
  project_key: string;  // Bearer token used by the SDK
  posts: Post[];
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
};
```

### `ClientProject` (SDK-safe subset)

The public SDK endpoint strips sensitive fields before returning:

```ts
type ClientProject = {
  id: string;
  title: string;
  posts: ClientPost[];
  createdAt: Date;
  updatedAt: Date;
};

type ClientPost = {
  id: string;
  title: string;
  author: string;
  teaser: string;
  image?: string | null;
  createdAt: Date;
  updatedAt: Date;
  // body is intentionally absent — fetch the full post separately
};
```

### `UserDocument`

```ts
type UserDocument = {
  _id: string;
  email: string;
  first_name: string;
  last_name: string;
  plan: "single" | "startup" | "business";
  projects: UserDocumentProjects[];
  createdAt: Date;
  updatedAt: Date;
};
```

`projects` here is a lightweight summary array (id, title, timestamps, creator). It does not contain posts. To get posts, fetch the full project.

---

## Auth endpoints

### `POST /api/auth/signup`

Creates a new user account, starts a session, and sets the `auth_session` cookie.

**Request body:**

```ts
{
  email: string;       // Must be a valid email address
  password: string;    // Minimum 8 characters
  first_name: string;  // Max 30 characters
  last_name: string;   // Max 30 characters
}
```

**Response:** `ApiResponse<UserDocument>`

```json
{
  "success": true,
  "message": "Created account successfuly.",
  "response": {
    "_id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "jane@example.com",
    "first_name": "Jane",
    "last_name": "Doe",
    "plan": "single",
    "projects": [],
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Errors:**
- `"Email not valid."` — email failed regex validation
- `"Please enter a password."` — password field is empty
- `"Password must be at least 8 characters long."` — password is present but fewer than 8 characters
- `"Email already in use."` — MongoDB duplicate key (code 11000)
- `"Invalid first name."` / `"Invalid last name."` — empty or exceeds 30 characters

New accounts are created with the `"single"` plan. Passwords are hashed with Argon2 + a server-side salt (`HASH_SALT` env var) before storage.

---

### `POST /api/auth/signin`

Signs in an existing user and sets the `auth_session` cookie.

**Request body:**

```ts
{
  email: string;
  password: string;
}
```

**Response:** `ApiResponse<UserDocument>`

**Errors:**
- `"Account does not exist."` — no credentials document with that email
- `"Password is incorrect."`

---

### `POST /api/auth/signout`

Invalidates the current session and clears the `auth_session` cookie. Requires a valid session cookie.

**Request body:** none

**Response:** `ApiResponse<null>`

```json
{
  "success": true,
  "message": "Successfuly signed out user.",
  "response": null
}
```

---

### `GET /api/auth/user/get`

Returns the current user's `UserDocument`. Requires a valid session cookie.

**Response:** `ApiResponse<UserDocument>`

This is the endpoint used by the dashboard SSR layer to hydrate the user on every page load (`lib/server/functions/user/fetchUser.ts`).

---

### `PATCH /api/auth/user/update`

Updates the current user's first or last name. Both fields are optional — only send what you want to change. Requires a valid session cookie.

**Request body:**

```ts
{
  first_name?: string;  // Max 30 characters
  last_name?: string;   // Max 30 characters
}
```

**Response:** `ApiResponse<null>`

Empty-string values are silently ignored. Only non-empty strings are written to the database.

---

### `DELETE /api/auth/delete`

Permanently deletes the account, all of its projects, all sessions, and the credentials document. This runs inside a MongoDB transaction so either everything is deleted or nothing is. Requires a valid session cookie.

**Request body:**

```ts
{
  password: string;  // Current password, re-verified before deletion
}
```

**Response:** `ApiResponse<null>`

**Errors:**
- `"Invalid password."` — password confirmation failed

---

## Project endpoints

### `POST /api/projects/create`

Creates a new project for the authenticated user. Generates a unique `project_key` (32 random bytes, base64-encoded, prefixed with `atom-`). Requires a valid session cookie.

**Request body:**

```ts
{
  title: string;  // Max 50 characters, must not be empty
}
```

**Response:** `ApiResponse<Project>`

```json
{
  "success": true,
  "message": "Successfuly created project.",
  "response": {
    "_id": "...",
    "title": "My Blog",
    "project_key": "atom-abc123...",
    "posts": [],
    "creator_uid": "...",
    "createdAt": "...",
    "updatedAt": "..."
  }
}
```

**Plan limits:**
- `single` plan: max 2 projects
- `startup` plan: max 3 projects
- `business` plan: max 5 projects

Attempting to create beyond the plan limit returns `"Cannot create more than N projects."`. The project key returned here is the Bearer token you pass to the SDK.

---

### `DELETE /api/projects/delete?project_id=`

Deletes a project and removes it from the user's `UserDocument.projects` array. Both operations run in a MongoDB transaction. Requires a valid session cookie. You must be the project's `creator_uid`.

**Query parameter:** `project_id` (UUID string)

**Response:** `ApiResponse<null>`

---

### `GET /api/projects/get/single`

Fetches a single project. Accepts either a session cookie (dashboard) or a Bearer token (programmatic). The auth method determines which path is taken.

**When called with a session cookie:** Query parameter `project_id` is required. Ownership is verified — the request will fail with `"Not authorized."` if the authenticated user is not the project's `creator_uid`.

**When called with a Bearer token:** No query parameters needed. The project is looked up by `project_key` directly.

**Response:** `ApiResponse<Project>` (full project including all post bodies)

This endpoint is used internally by the SSR layer (`lib/server/functions/projects/getProject.ts`).

---

### `GET /api/projects/get/single/client`

The public endpoint used by the `atom-nextjs` SDK. Requires `Authorization: Bearer <project_key>`. Returns a `ClientProject` with a stripped-down post list — no post bodies are included. Use this to render a post listing page.

**Response:** `ApiResponse<ClientProject>`

```json
{
  "success": true,
  "message": "Successfuly fetched posts.",
  "response": {
    "id": "...",
    "title": "My Blog",
    "posts": [
      {
        "id": "...",
        "title": "Hello World",
        "author": "Jane Doe",
        "teaser": "A short description of this post.",
        "image": "https://example.com/cover.jpg",
        "createdAt": "...",
        "updatedAt": "..."
      }
    ],
    "createdAt": "...",
    "updatedAt": "..."
  }
}
```

`body`, `keywords`, and `creator_uid` are intentionally absent from `ClientPost`. To render a post's content, call `GET /api/posts/get/single`.

---

## Post endpoints

All post write operations require a valid session cookie and ownership of the parent project.

### `POST /api/posts/create?project_id=`

Creates a new post inside an existing project.

**Query parameter:** `project_id` (UUID)

**Request body:**

```ts
{
  title: string;      // Required
  author: string;     // Required
  body: string;       // Required. Markdown/MDX. Max length depends on plan.
  teaser: string;     // Required. Max 100 characters.
  image?: string;     // Optional. URL string.
  keywords?: string;  // Optional. Comma-separated, e.g. "react,nextjs,cms"
}
```

**Response:** `ApiResponse<Post>`

The `keywords` string is split on commas and stored as a `string[]`. The returned `Post` object already has the parsed array.

**Plan body limits:**
- `single`: 10,000 characters
- `startup`: 100,000 characters
- `business`: 500,000 characters

---

### `PATCH /api/posts/update?project_id=&post_id=`

Updates one or more fields of an existing post. All fields are optional — only provided non-empty values are applied.

**Query parameters:** `project_id`, `post_id`

**Request body:**

```ts
{
  title?: string;     // Max 50 characters
  author?: string;    // Max 30 characters
  body?: string;      // Subject to plan body length limit
  teaser?: string;    // Max 100 characters
  keywords?: string;  // Max 30 characters
  image?: string;     // URL string
}
```

**Response:** `ApiResponse<null>`

Empty strings are filtered out and not written. Partial updates are safe.

---

### `DELETE /api/posts/delete?project_id=&post_id=`

Removes a post from its parent project using a `$pull` on the posts sub-document array.

**Query parameters:** `project_id`, `post_id`

**Response:** `ApiResponse<null>`

---

### `GET /api/posts/get/single?post_id=`

Fetches a single post by ID. Requires `Authorization: Bearer <project_key>` — this is a public endpoint with no session requirement.

**Query parameter:** `post_id` (UUID)

**Headers:** `Authorization: Bearer <project_key>`

**Response:** `ApiResponse<Post>`

One plan-specific behavior: if the project owner is on the `single` plan, the string `\n\nThis post was created using [Atom](https://atomcms.vercel.app/)` is appended to `body` before returning. This watermark is removed on paid plans.

---

## The `atom-nextjs` SDK

The SDK (`packages/atom-nextjs`) wraps the two public endpoints above into React Server Components and helper functions. Install it with:

```bash
npm i atom-nextjs@latest
```

All SDK exports are in `packages/atom-nextjs/src/index.tsx`. Everything requires your `project_key`, which you get from the Atom dashboard after creating a project.

### `getProject(projectKey)`

Raw fetch wrapper around `GET /api/projects/get/single/client`. Returns `ApiResponse<ClientProject>`.

```ts
import { getProject } from 'atom-nextjs';

const res = await getProject(process.env.ATOM_PROJECT_KEY!);
if (res.success) {
  console.log(res.response.posts); // ClientPost[]
}
```

### `getPost(projectKey, postId)`

Raw fetch wrapper around `GET /api/posts/get/single`. Returns `ApiResponse<Post>` with the full body.

```ts
import { getPost } from 'atom-nextjs';

const res = await getPost(process.env.ATOM_PROJECT_KEY!, postId);
if (res.success) {
  console.log(res.response.body); // raw MDX string
}
```

### `<AtomPage>` component

A Next.js Server Component that fetches your project and renders a grid of post cards. Use it on your blog's index page.

```tsx
import { AtomPage } from 'atom-nextjs';

export default function BlogPage() {
  return (
    <AtomPage
      projectKey={process.env.ATOM_PROJECT_KEY!}
      baseRoute="/blog"   // cards link to /blog/<post_id>
      title={true}        // renders the project title as an h1 (default: true)
    />
  );
}
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | required | Your `atom-` prefixed project key |
| `baseRoute` | `string` | required | URL prefix for post links, e.g. `/blog` |
| `title` | `boolean` | `true` | Whether to render the project title |

On error, renders `res.message` as plain text rather than throwing.

### `<Atom>` component

A Server Component that fetches a single post and renders the full article: title, cover image (if present), author, publish date, and MDX body. Use it on individual post pages.

```tsx
import { Atom } from 'atom-nextjs';

export default function PostPage({ params }: { params: { id: string } }) {
  return (
    <Atom
      projectKey={process.env.ATOM_PROJECT_KEY!}
      postId={params.id}
    />
  );
}
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | required | Your project key |
| `postId` | `string` | required | The post's UUID (use as the dynamic route segment) |
| `remarkPlugins` | `any[]` | `[]` | Additional remark plugins passed to `compileMDX` |
| `rehypePlugins` | `any[]` | `[]` | Additional rehype plugins passed to `compileMDX` |

`remark-gfm` and `rehype-sanitize` are always applied and don't need to be passed in. The component renders inside a `prose` / `lg:prose-xl` Tailwind container — you need `@tailwindcss/typography` in your project for the styles to take effect.

### `<AtomBody>` component

If you want the MDX rendering without the article chrome (title, image, author header), use `AtomBody` directly. It compiles and renders the body string with the same plugin defaults.

```tsx
import { AtomBody } from 'atom-nextjs';

<AtomBody
  body={post.body}
  className="my-prose-class"
  remarkPlugins={[myPlugin]}
/>
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `body` | `string` | required | Raw Markdown/MDX string |
| `className` | `string` | `undefined` | CSS class applied to the wrapper `div` |
| `remarkPlugins` | `any[]` | `[]` | Extra remark plugins |
| `rehypePlugins` | `any[]` | `[]` | Extra rehype plugins |

### `generatePostMetadata(apiKey, postId)`

Builds a Next.js `Metadata` object from a post. Designed for use inside `generateMetadata()`.

```ts
import { generatePostMetadata } from 'atom-nextjs';

export async function generateMetadata({ params }: { params: { id: string } }) {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
}
```

Returns `{ title, description, keywords, authors }`. If the fetch fails, returns `{ title: "Couldn't find post.", authors: { name: 'Atom' } }` rather than throwing.

### `generateSitemap(projectKey, blogRoute)`

Builds a sitemap entries array compatible with Next.js `sitemap.ts`. Each post gets an entry at `<blogRoute>/<post.id>`. The blog index route (`blogRoute`) is also included.

```ts
// app/sitemap.ts
import { generateSitemap } from 'atom-nextjs';

export default async function sitemap() {
  return generateSitemap(process.env.ATOM_PROJECT_KEY!, 'https://example.com/blog');
}
```

Each entry has `url`, `lastModified` (from `post.updatedAt`), and `priority: 0.5`. The index route gets `priority: 0.6`.

### Loading skeletons

Two skeleton components are provided for use with `<Suspense>`:

```tsx
import { AtomLoadingSkeleton, AtomArticleSkeleton } from 'atom-nextjs';

// For post listing pages (shows 4 card-shaped skeletons)
<Suspense fallback={<AtomLoadingSkeleton />}>
  <AtomPage projectKey={...} baseRoute="/blog" />
</Suspense>
```

```tsx
// For individual post pages (shows a banner + text skeletons)
<Suspense fallback={<AtomArticleSkeleton />}>
  <Atom projectKey={...} postId={...} />
</Suspense>
```

Both use `react-loading-skeleton`. `AtomLoadingSkeleton` imports `react-loading-skeleton/dist/skeleton.css` automatically. `AtomArticleSkeleton` does **not** import the CSS — you must import it yourself (`import 'react-loading-skeleton/dist/skeleton.css'`) if you use it. Neither component accepts props.

---

## Plan limits summary

| | Single (free) | Startup ($3.99/mo) | Business ($11.99/mo) |
|---|---|---|---|
| Max projects | 2 | 3 | 5 |
| Max posts | 100 | 1,000 | 2,500 |
| Max body length | 10,000 chars | 100,000 chars | 500,000 chars |
| Watermark | Yes | No | No |

Plan is stored on the `UserDocument` as `"single"`, `"startup"`, or `"business"`. Startup and Business plans are currently disabled in the source (`disabled: true` in `lib/contants.tsx`).

> **Note:** The "Max posts" limit is defined in `planDetails` (`max_docs`) but is **not enforced** by any API route. The API will not reject post creation based on post count.

---

## Error handling

All endpoints follow the same pattern: unexpected errors are caught, logged to the server console, and returned as `{ success: false, message: "...", response: null }`. The HTTP status code is always 200 even for errors. Check `data.success` in your code, not the status code.

Rate limit rejections also return a 200-status JSON body:

```json
{
  "success": false,
  "message": "Too many requests.",
  "response": null
}
```

The base API URL is `https://cmsatom.netlify.app/api` in production, or `http://localhost:3000/api` when `NEXT_PUBLIC_ENV` is not set to `"prod"` (controlled in `lib/contants.tsx`).
